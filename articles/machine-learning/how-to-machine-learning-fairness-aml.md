---
title: Evaluación de la equidad de los modelos de ML en Python (versión preliminar)
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo evaluar y mitigar la equidad de los modelos de aprendizaje automático mediante Fairlearn y el SDK de Azure Machine Learning para Python.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: mesameki
author: mesameki
ms.reviewer: luquinta
ms.date: 11/16/2020
ms.topic: how-to
ms.custom: devx-track-python, responsible-ml
ms.openlocfilehash: 3b71347f9375ebb24befe665c031af9cd7ad7cc3
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107884847"
---
# <a name="use-azure-machine-learning-with-the-fairlearn-open-source-package-to-assess-the-fairness-of-ml-models-preview"></a>Uso de Azure Machine Learning con el paquete de código abierto Fairlearn para evaluar la equidad de los modelos de ML (versión preliminar)

En esta guía paso a paso, aprenderá a usar el paquete Python de código abierto [Fairlearn](https://fairlearn.github.io/) con Azure Machine Learning para realizar las siguientes tareas:

* Evalúe la equidad de las predicciones del modelo. Para más información sobre la equidad en el aprendizaje automático, consulte la [equidad en el artículo del aprendizaje automático](concept-fairness-ml.md).
* Cargue, enumere y descargue información de evaluación de equidad en Azure Machine Learning Studio.
* Consulte un panel de evaluación de equidad en Azure Machine Learning Studio para interactuar con la información de equidad de los modelos.

>[!NOTE]
> La evaluación de equidad no es un ejercicio puramente técnico. **Este paquete puede ayudarle a evaluar la equidad de un modelo de aprendizaje automático, pero solo puede configurar y tomar decisiones sobre cómo funciona el modelo.**  Aunque este paquete ayuda a identificar métricas cuantitativas para evaluar la equidad, los desarrolladores de modelos de aprendizaje automático también deben realizar un análisis cualitativo para evaluar la equidad de sus propios modelos.

## <a name="azure-machine-learning-fairness-sdk"></a>SDK de equidad Azure Machine Learning 

El SDK de equidad Azure Machine Learning, `azureml-contrib-fairness`, integra el paquete de código abierto de Python, [Fairlearn](http://fairlearn.github.io), dentro de Azure Machine Learning. Para más información acerca de la integración de Fairlearn en Azure Machine Learning, consulte estos [cuadernos de muestra](https://github.com/Azure/MachineLearningNotebooks/tree/master/contrib/fairness). Para más información sobre Fairlearn, consulte la [guía de ejemplo](https://fairlearn.org/v0.6.0/auto_examples/) y los [cuadernos de muestra](https://github.com/fairlearn/fairlearn/tree/master/notebooks). 

Use los siguientes comandos para instalar los paquetes `azureml-contrib-fairness` y `fairlearn`:
```bash
pip install azureml-contrib-fairness
pip install fairlearn==0.4.6
```
Las versiones posteriores de Fairlearn también deberían funcionar en el siguiente código de ejemplo.



## <a name="upload-fairness-insights-for-a-single-model"></a>Cargar información de equidad para un único modelo

En el ejemplo siguiente se muestra cómo usar el paquete de equidad. Cargaremos información de equidad del modelo en Azure Machine Learning y veremos el panel de evaluación de equidad en Azure Machine Learning Studio.

1. Entrene un modelo de ejemplo en Jupyter Notebook. 

    Para el conjunto de datos, usaremos el conocido conjunto de datos del censo de adultos, que obtendremos de OpenML. Imaginemos que tenemos un problema con la decisión de un préstamo que cuenta con una etiqueta que indica si una persona pagó un préstamo anterior. Entrenaremos un modelo para predecir si aquellas personas que no hayamos visto antes devolverán un préstamo. Este modelo podría usarse para tomar decisiones en los préstamos.

    ```python
    import copy
    import numpy as np
    import pandas as pd

    from sklearn.compose import ColumnTransformer
    from sklearn.datasets import fetch_openml
    from sklearn.impute import SimpleImputer
    from sklearn.linear_model import LogisticRegression
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler, OneHotEncoder
    from sklearn.compose import make_column_selector as selector
    from sklearn.pipeline import Pipeline
    
    from fairlearn.widget import FairlearnDashboard

    # Load the census dataset
    data = fetch_openml(data_id=1590, as_frame=True)
    X_raw = data.data
    y = (data.target == ">50K") * 1
    
    # (Optional) Separate the "sex" and "race" sensitive features out and drop them from the main data prior to training your model
    X_raw = data.data
    y = (data.target == ">50K") * 1
    A = X_raw[["race", "sex"]]
    X = X_raw.drop(labels=['sex', 'race'],axis = 1)
    
    # Split the data in "train" and "test" sets
    (X_train, X_test, y_train, y_test, A_train, A_test) = train_test_split(
        X_raw, y, A, test_size=0.3, random_state=12345, stratify=y
    )

    # Ensure indices are aligned between X, y and A,
    # after all the slicing and splitting of DataFrames
    # and Series
    X_train = X_train.reset_index(drop=True)
    X_test = X_test.reset_index(drop=True)
    y_train = y_train.reset_index(drop=True)
    y_test = y_test.reset_index(drop=True)
    A_train = A_train.reset_index(drop=True)
    A_test = A_test.reset_index(drop=True)

    # Define a processing pipeline. This happens after the split to avoid data leakage
    numeric_transformer = Pipeline(
        steps=[
            ("impute", SimpleImputer()),
            ("scaler", StandardScaler()),
        ]
    )
    categorical_transformer = Pipeline(
        [
            ("impute", SimpleImputer(strategy="most_frequent")),
            ("ohe", OneHotEncoder(handle_unknown="ignore")),
        ]
    )
    preprocessor = ColumnTransformer(
        transformers=[
            ("num", numeric_transformer, selector(dtype_exclude="category")),
            ("cat", categorical_transformer, selector(dtype_include="category")),
        ]
    )

    # Put an estimator onto the end of the pipeline
    lr_predictor = Pipeline(
        steps=[
            ("preprocessor", copy.deepcopy(preprocessor)),
            (
                "classifier",
                LogisticRegression(solver="liblinear", fit_intercept=True),
            ),
        ]
    )

    # Train the model on the test data
    lr_predictor.fit(X_train, y_train)

    # (Optional) View this model in Fairlearn's fairness dashboard, and see the disparities which appear:
    from fairlearn.widget import FairlearnDashboard
    FairlearnDashboard(sensitive_features=A_test, 
                       sensitive_feature_names=['Race', 'Sex'],
                       y_true=y_test,
                       y_pred={"lr_model": lr_predictor.predict(X_test)})
    ```

2. Inicie sesión en Azure Machine Learning y registre el modelo.
   
    El panel de equidad puede integrarse con modelos registrados o no registrados. Registre el modelo en Azure Machine Learning con los siguientes pasos:
    ```python
    from azureml.core import Workspace, Experiment, Model
    import joblib
    import os
    
    ws = Workspace.from_config()
    ws.get_details()

    os.makedirs('models', exist_ok=True)
    
    # Function to register models into Azure Machine Learning
    def register_model(name, model):
        print("Registering ", name)
        model_path = "models/{0}.pkl".format(name)
        joblib.dump(value=model, filename=model_path)
        registered_model = Model.register(model_path=model_path,
                                        model_name=name,
                                        workspace=ws)
        print("Registered ", registered_model.id)
        return registered_model.id

    # Call the register_model function 
    lr_reg_id = register_model("fairness_logistic_regression", lr_predictor)
    ```

3. Calcular previamente las métricas de equidad.

    Cree un diccionario de panel con el paquete `metrics` de Fairlearn. El método `_create_group_metric_set` tiene argumentos similares al constructor del panel, salvo que las características confidenciales se pasan como un diccionario (para asegurarse de que los nombres están disponibles). También se debe especificar el tipo de predicción (clasificación binaria en este caso) cuando se llama a este método.

    ```python
    #  Create a dictionary of model(s) you want to assess for fairness 
    sf = { 'Race': A_test.race, 'Sex': A_test.sex}
    ys_pred = { lr_reg_id:lr_predictor.predict(X_test) }
    from fairlearn.metrics._group_metric_set import _create_group_metric_set

    dash_dict = _create_group_metric_set(y_true=y_test,
                                        predictions=ys_pred,
                                        sensitive_features=sf,
                                        prediction_type='binary_classification')
    ```
4. Cargue las métricas de equidad precalculadas.
    
    Ahora, importe el paquete `azureml.contrib.fairness` para realizar la carga:

    ```python
    from azureml.contrib.fairness import upload_dashboard_dictionary, download_dashboard_by_upload_id
    ```
    Cree un Experimento, después una Ejecución y cargue el panel en él:
    ```python
    exp = Experiment(ws, "Test_Fairness_Census_Demo")
    print(exp)

    run = exp.start_logging()

    # Upload the dashboard to Azure Machine Learning
    try:
        dashboard_title = "Fairness insights of Logistic Regression Classifier"
        # Set validate_model_ids parameter of upload_dashboard_dictionary to False if you have not registered your model(s)
        upload_id = upload_dashboard_dictionary(run,
                                                dash_dict,
                                                dashboard_name=dashboard_title)
        print("\nUploaded to id: {0}\n".format(upload_id))
        
        # To test the dashboard, you can download it back and ensure it contains the right information
        downloaded_dict = download_dashboard_by_upload_id(run, upload_id)
    finally:
        run.complete()
    ```
5. Comprobación del panel de equidad desde Azure Machine Learning Studio

    Si completa los pasos anteriores (carga de información de equidad generada en Azure Machine Learning), puede ver el panel de equidad en [Azure Machine Learning Studio](https://ml.azure.com). Este panel es el mismo panel de visualización proporcionado en Fairlearn, lo que le permite analizar las diferencias entre los subgrupos de la característica confidencial (por ejemplo, macho frente a hembra).
    Siga una de estas rutas de acceso para llegar al panel de visualización de Azure Machine Learning Studio:

    * **Panel de experimentos (versión preliminar)**
    1. Al hacer clic en **Experimentos** en el panel izquierdo, verá una lista de experimentos que se han ejecutado en Azure Machine Learning.
    1. Seleccione un experimento determinado para ver todas las ejecuciones de ese experimento.
    1. Seleccione una ejecución y luego, la pestaña **Equidad** en el panel de visualización de la explicación.
    1. Una vez que se abra la pestaña **Equidad**, haga clic en un **id. de equidad** en el menú de la derecha.
    1. Para configurar el panel, seleccione el atributo de confidencialidad, la métrica de rendimiento y la métrica de equidad de interés en la página de evaluación de equidad.
    1. Cambie el tipo de gráfico de uno a otro para observar los daños en la **asignación** y en la **calidad del servicio**.



    [![Asignación del panel de equidad](./media/how-to-machine-learning-fairness-aml/dashboard-1.png)](./media/how-to-machine-learning-fairness-aml/dashboard-1.png#lightbox)
    
    [![Calidad de servicio del panel de equidad](./media/how-to-machine-learning-fairness-aml/dashboard-2.png)](./media/how-to-machine-learning-fairness-aml/dashboard-2.png#lightbox)
    * **Panel de modelos**
    1. Si registró el modelo original siguiendo los pasos anteriores, puede seleccionar **Modelos** en el panel izquierdo para verlo.
    1. Seleccione un modelo y luego, la pestaña **Equidad** para ver el panel de visualización de la explicación.

    Para obtener más información sobre el panel de visualización y lo que contiene, consulte la [guía del usuario](https://fairlearn.org/v0.6.0/user_guide/assessment.html#fairlearn-dashboard) de Fairlearn.

## <a name="upload-fairness-insights-for-multiple-models"></a>Cargar información de equidad para varios modelos

Para comparar varios modelos y ver cómo difieren sus evaluaciones de equidad, puede pasar más de un modelo al panel de visualización y comparar sus ventajas e inconvenientes en el rendimiento.

1. Entrenar los modelos:
    
    ahora vamos a crear un segundo clasificador, basado en un estimador de máquina de vectores de soporte, y cargaremos un diccionario de panel de equidad mediante el paquete `metrics` de Fairlearn. Damos por hecho que el modelo entrenado anteriormente todavía está disponible.


    ```python
    # Put an SVM predictor onto the preprocessing pipeline
    from sklearn import svm
    svm_predictor = Pipeline(
        steps=[
            ("preprocessor", copy.deepcopy(preprocessor)),
            (
                "classifier",
                svm.SVC(),
            ),
        ]
    )

    # Train your second classification model
    svm_predictor.fit(X_train, y_train)
    ```

2. Registrar los modelos

    Después, registre ambos modelos dentro de Azure Machine Learning. Para mayor comodidad, almacene los resultados en un diccionario que asigne el `id` del modelo registrado (una cadena con formato `name:version`) al mismo predictor:

    ```python
    model_dict = {}

    lr_reg_id = register_model("fairness_logistic_regression", lr_predictor)
    model_dict[lr_reg_id] = lr_predictor

    svm_reg_id = register_model("fairness_svm", svm_predictor)
    model_dict[svm_reg_id] = svm_predictor
    ```

3. Cargar el panel de Fairlearn localmente

    Antes de cargar la información de equidad en Azure Machine Learning, puede examinar estas predicciones en un panel de Fairlearn invocado localmente. 



    ```python
    #  Generate models' predictions and load the fairness dashboard locally 
    ys_pred = {}
    for n, p in model_dict.items():
        ys_pred[n] = p.predict(X_test)

    from fairlearn.widget import FairlearnDashboard

    FairlearnDashboard(sensitive_features=A_test, 
                    sensitive_feature_names=['Race', 'Sex'],
                    y_true=y_test.tolist(),
                    y_pred=ys_pred)
    ```

3. Calcular previamente las métricas de equidad.

    Cree un diccionario de panel con el paquete `metrics` de Fairlearn.

    ```python
    sf = { 'Race': A_test.race, 'Sex': A_test.sex }

    from fairlearn.metrics._group_metric_set import _create_group_metric_set

    dash_dict = _create_group_metric_set(y_true=Y_test,
                                        predictions=ys_pred,
                                        sensitive_features=sf,
                                        prediction_type='binary_classification')
    ```
4. Cargue las métricas de equidad precalculadas.
    
    Ahora, importe el paquete `azureml.contrib.fairness` para realizar la carga:

    ```python
    from azureml.contrib.fairness import upload_dashboard_dictionary, download_dashboard_by_upload_id
    ```
    Cree un Experimento, después una Ejecución y cargue el panel en él:
    ```python
    exp = Experiment(ws, "Compare_Two_Models_Fairness_Census_Demo")
    print(exp)

    run = exp.start_logging()

    # Upload the dashboard to Azure Machine Learning
    try:
        dashboard_title = "Fairness Assessment of Logistic Regression and SVM Classifiers"
        # Set validate_model_ids parameter of upload_dashboard_dictionary to False if you have not registered your model(s)
        upload_id = upload_dashboard_dictionary(run,
                                                dash_dict,
                                                dashboard_name=dashboard_title)
        print("\nUploaded to id: {0}\n".format(upload_id))
        
        # To test the dashboard, you can download it back and ensure it contains the right information
        downloaded_dict = download_dashboard_by_upload_id(run, upload_id)
    finally:
        run.complete()
    ```


    Al igual que en la sección anterior, puede seguir una de las rutas de acceso descritas anteriormente (a través de **Experimentos** o **Modelos**) en Azure Machine Learning Studio para acceder al panel de visualización y comparar los dos modelos en cuanto a equidad y rendimiento.


## <a name="upload-unmitigated-and-mitigated-fairness-insights"></a>Cargar información de equidad no mitigada y mitigada

Puede usar los [algoritmos de mitigación](https://fairlearn.org/v0.6.0/user_guide/mitigation.html) de Fairlearn, comparar sus modelos mitigados generados con el modelo no mitigado original y navegar por los equilibrios de rendimiento/equidad entre los modelos comparados.

Para ver un ejemplo que demuestre el uso del algoritmo de mitigación de la [Búsqueda de cuadrícula](https://fairlearn.org/v0.6.0/user_guide/mitigation.html#grid-search) (que crea una colección de modelos mitigados con diferentes elementos de equidad y rendimiento), consulte este [cuaderno de muestra](https://github.com/Azure/MachineLearningNotebooks/blob/master/contrib/fairness/fairlearn-azureml-mitigation.ipynb). 

La carga de la información de equidad de varios modelos en una sola ejecución le permitirá comparar los modelos con respecto a la equidad y el rendimiento. Puede hacer clic en cualquiera de los modelos mostrados en el gráfico de comparación de modelos para ver la información detallada sobre la equidad de un modelo determinado.


[![Panel de comparación de modelos de Fairlearn](./media/how-to-machine-learning-fairness-aml/multi-model-dashboard.png)](./media/how-to-machine-learning-fairness-aml/multi-model-dashboard.png#lightbox)
    

## <a name="next-steps"></a>Pasos siguientes

[Más información sobre la imparcialidad de modelos](concept-fairness-ml.md)

[Consulte los cuadernos de muestra de equidad de Azure Machine Learning](https://github.com/Azure/MachineLearningNotebooks/tree/master/contrib/fairness)
