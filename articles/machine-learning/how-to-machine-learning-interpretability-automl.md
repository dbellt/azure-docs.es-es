---
title: Capacidad de explicación en aprendizaje automático automatizado (versión preliminar)
titleSuffix: Azure Machine Learning
description: Aprenda a obtener explicaciones sobre cómo el modelo de aprendizaje automático automatizado determina la importancia de las características y realiza predicciones al usar el SDK de Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.custom: automl, responsible-ml
ms.author: mithigpe
author: minthigpen
ms.date: 07/09/2020
ms.openlocfilehash: 716ed2920b1e731a8515b7f099e77853955d9943
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107884811"
---
# <a name="interpretability-model-explanations-in-automated-machine-learning-preview"></a>Capacidad de interpretación: explicaciones de los modelos en el aprendizaje automático automatizado (versión preliminar)


En este artículo, aprenderá a obtener explicaciones del aprendizaje automático automatizado en Azure Machine Learning con el SDK de Python. El aprendizaje automático automatizado ayuda a comprender la importancia de las características de los modelos que se generan. 

De forma predeterminada, todas las versiones del SDK posteriores a 1.0.85 establecen `model_explainability=True`. En la versión 1.0.85 del SDK y anteriores, los usuarios deben establecer `model_explainability=True` en el objeto `AutoMLConfig` para poder usar la interpretación de modelos. 


En este artículo aprenderá a:

- Interpretar durante el entrenamiento para el mejor o cualquier modelo.
- Habilitar visualizaciones para ayudarle a ver patrones en datos y explicaciones.
- Implementar la interpretación durante la inferencia o puntuación.

## <a name="prerequisites"></a>Prerrequisitos

- Características de interoperabilidad. Ejecute `pip install azureml-interpret` para obtener el paquete necesario.
- Conocimiento de la compilación de experimentos de aprendizaje automático automatizado. Para más información sobre cómo usar el SDK de Azure Machine Learning, complete este [tutorial de modelo de regresión](tutorial-auto-train-models.md) o consulte cómo [configurar experimentos de aprendizaje automático automatizado](how-to-configure-auto-train.md).

## <a name="interpretability-during-training-for-the-best-model"></a>Interpretación durante el entrenamiento del mejor modelo

Recupere la explicación de `best_run`, que incluye explicaciones de las características diseñadas y sin formato.

> [!NOTE]
> La interpretabilidad, la mejor explicación del modelo, no está disponible para los experimentos de previsión de ML automático que recomiendan los algoritmos siguientes como el mejor modelo: 
> * TCNForecaster
> * AutoArima
> * ExponentialSmoothing
> * Prophet
> * Media 
> * Naive
> * Media estacional 
> * Naive estacional

### <a name="download-the-engineered-feature-importances-from-the-best-run"></a>Descargue las importancias de las características diseñadas de la mejor ejecución

Puede usar `ExplanationClient` para descargar las explicaciones de las características diseñadas del almacén de artefactos de `best_run`. 

```python
from azureml.interpret import ExplanationClient

client = ExplanationClient.from_run(best_run)
engineered_explanations = client.download_model_explanation(raw=False)
print(engineered_explanations.get_feature_importance_dict())
```

### <a name="download-the-raw-feature-importances-from-the-best-run"></a>Descargue las importancias de las características sin diseño de la mejor ejecución

Puede usar `ExplanationClient` para descargar las explicaciones de las características sin diseño del almacén de artefactos de `best_run`.

```python
from azureml.interpret import ExplanationClient

client = ExplanationClient.from_run(best_run)
raw_explanations = client.download_model_explanation(raw=True)
print(raw_explanations.get_feature_importance_dict())
```

## <a name="interpretability-during-training-for-any-model"></a>Interpretación durante el entrenamiento de cualquier modelo 

Al calcular las explicaciones del modelo y visualizarlas, no está limitado a una explicación de modelo existente para un modelo de AutoML. También puede obtener una explicación del modelo con datos de prueba diferentes. Los pasos de esta sección muestran cómo calcular y visualizar la importancia de las características diseñadas según los datos de prueba.

### <a name="retrieve-any-other-automl-model-from-training"></a>Recuperar cualquier otro modelo de aprendizaje automático automatizado del entrenamiento

```python
automl_run, fitted_model = local_run.get_output(metric='accuracy')
```

### <a name="set-up-the-model-explanations"></a>Configuración de las explicaciones del modelo

Utilice `automl_setup_model_explanations` para obtener las explicaciones diseñadas y sin diseño. `fitted_model` puede generar los siguientes elementos:

- Datos destacados de ejemplos entrenados o de prueba
- Listas de nombres de características diseñadas
- Clases de búsqueda en la columna etiquetada de los escenarios de clasificación

`automl_explainer_setup_obj` contiene todas las estructuras de la lista anterior.

```python
from azureml.train.automl.runtime.automl_explain_utilities import automl_setup_model_explanations

automl_explainer_setup_obj = automl_setup_model_explanations(fitted_model, X=X_train, 
                                                             X_test=X_test, y=y_train, 
                                                             task='classification')
```

### <a name="initialize-the-mimic-explainer-for-feature-importance"></a>Inicializar el explicador Mimic para la importancia de la característica

Para generar una explicación de los modelos de aprendizaje automático automatizado, utilice la clase `MimicWrapper`. MimicWrapper se puede inicializar con estos parámetros:

- El objeto de configuración del explicador
- El área de trabajo
- Un modelo suplente para explicar el modelo de ML automatizado `fitted_model`

MimicWrapper también toma el objeto `automl_run` donde se cargarán las explicaciones diseñadas.

```python
from azureml.interpret import MimicWrapper

# Initialize the Mimic Explainer
explainer = MimicWrapper(ws, automl_explainer_setup_obj.automl_estimator,
                         explainable_model=automl_explainer_setup_obj.surrogate_model, 
                         init_dataset=automl_explainer_setup_obj.X_transform, run=automl_run,
                         features=automl_explainer_setup_obj.engineered_feature_names, 
                         feature_maps=[automl_explainer_setup_obj.feature_map],
                         classes=automl_explainer_setup_obj.classes,
                         explainer_kwargs=automl_explainer_setup_obj.surrogate_model_params)
```

### <a name="use-mimic-explainer-for-computing-and-visualizing-engineered-feature-importance"></a>Use Mimic Explainer para calcular y visualizar la importancia de las características diseñadas

Puede llamar al método `explain()` de MimicWrapper con los ejemplos de prueba transformados para obtener la importancia de las características diseñadas que se generaron. También puede iniciar sesión en [Azure Machine Learning Studio](https://ml.azure.com/) para obtener las visualizaciones del panel de explicaciones de los valores de importancia de las características diseñadas generadas por los caracterizadores de aprendizaje automático automatizado.

```python
engineered_explanations = explainer.explain(['local', 'global'], eval_dataset=automl_explainer_setup_obj.X_test_transform)
print(engineered_explanations.get_feature_importance_dict())
```
En el caso de los modelos entrenados con el aprendizaje automático automatizado, puede obtener el mejor modelo mediante el método `get_output()` y las explicaciones de proceso de forma local.  Puede visualizar los resultados de la explicación con `ExplanationDashboard` desde el paquete `interpret-community`.

```python
best_run, fitted_model = remote_run.get_output()

from azureml.train.automl.runtime.automl_explain_utilities import AutoMLExplainerSetupClass, automl_setup_model_explanations
automl_explainer_setup_obj = automl_setup_model_explanations(fitted_model, X=X_train,
                                                             X_test=X_test, y=y_train,
                                                             task='regression')

from interpret.ext.glassbox import LGBMExplainableModel
from azureml.interpret.mimic_wrapper import MimicWrapper

explainer = MimicWrapper(ws, automl_explainer_setup_obj.automl_estimator, LGBMExplainableModel,
                         init_dataset=automl_explainer_setup_obj.X_transform, run=best_run,
                         features=automl_explainer_setup_obj.engineered_feature_names,
                         feature_maps=[automl_explainer_setup_obj.feature_map],
                         classes=automl_explainer_setup_obj.classes)
                         
pip install interpret-community[visualization]

engineered_explanations = explainer.explain(['local', 'global'], eval_dataset=automl_explainer_setup_obj.X_test_transform)
print(engineered_explanations.get_feature_importance_dict()),
from interpret_community.widget import ExplanationDashboard
ExplanationDashboard(engineered_explanations, automl_explainer_setup_obj.automl_estimator, datasetX=automl_explainer_setup_obj.X_test_transform)

 

raw_explanations = explainer.explain(['local', 'global'], get_raw=True,
                                     raw_feature_names=automl_explainer_setup_obj.raw_feature_names,
                                     eval_dataset=automl_explainer_setup_obj.X_test_transform)
print(raw_explanations.get_feature_importance_dict()),
from interpret_community.widget import ExplanationDashboard
ExplanationDashboard(raw_explanations, automl_explainer_setup_obj.automl_pipeline, datasetX=automl_explainer_setup_obj.X_test_raw)
```

### <a name="use-mimic-explainer-for-computing-and-visualizing-raw-feature-importance"></a>Use MimicExplainer para calcular y visualizar la importancia de las características sin procesar

Puede llamar al método `explain()` de MimicWrapper con los ejemplos de prueba transformados para obtener la importancia de las características sin procesar. En [Machine Learning Studio](https://ml.azure.com/), puede obtener la visualización del panel de valores de importancia de las características sin procesar.

```python
raw_explanations = explainer.explain(['local', 'global'], get_raw=True,
                                     raw_feature_names=automl_explainer_setup_obj.raw_feature_names,
                                     eval_dataset=automl_explainer_setup_obj.X_test_transform,
                                     raw_eval_dataset=automl_explainer_setup_obj.X_test_raw)
print(raw_explanations.get_feature_importance_dict())
```

## <a name="interpretability-during-inference"></a>Interpretación durante la inferencia

En esta sección, obtendrá información sobre cómo operar un modelo de aprendizaje automático automatizado con el explicador, que se utilizó para calcular las explicaciones de la sección anterior.

### <a name="register-the-model-and-the-scoring-explainer"></a>Registre el modelo y la explicación de la puntuación

Use `TreeScoringExplainer` para crear el explicador de puntuación, que calculará los valores de importancia de las características diseñadas a la hora de la inferencia. El explicador de puntuación se inicializa con el `feature_map` que se calculó previamente. 

Guarde el explicador de puntuación y, a continuación, registre el modelo y el explicador de puntuación con el servicio de Administración de modelos. Ejecute el código siguiente:

```python
from azureml.interpret.scoring.scoring_explainer import TreeScoringExplainer, save

# Initialize the ScoringExplainer
scoring_explainer = TreeScoringExplainer(explainer.explainer, feature_maps=[automl_explainer_setup_obj.feature_map])

# Pickle scoring explainer locally
save(scoring_explainer, exist_ok=True)

# Register trained automl model present in the 'outputs' folder in the artifacts
original_model = automl_run.register_model(model_name='automl_model', 
                                           model_path='outputs/model.pkl')

# Register scoring explainer
automl_run.upload_file('scoring_explainer.pkl', 'scoring_explainer.pkl')
scoring_explainer_model = automl_run.register_model(model_name='scoring_explainer', model_path='scoring_explainer.pkl')
```

### <a name="create-the-conda-dependencies-for-setting-up-the-service"></a>Crear las dependencias de conda para configurar el servicio

Después, cree las dependencias de entorno necesarias en el contenedor para el modelo implementado. Tenga en cuenta que azureml-defaults con versión > = 1.0.45 debe indicarse como una dependencia de PIP, ya que contiene la funcionalidad necesaria para hospedar el modelo como un servicio web.

```python
from azureml.core.conda_dependencies import CondaDependencies

azureml_pip_packages = [
    'azureml-interpret', 'azureml-train-automl', 'azureml-defaults'
]

myenv = CondaDependencies.create(conda_packages=['scikit-learn', 'pandas', 'numpy', 'py-xgboost<=0.80'],
                                 pip_packages=azureml_pip_packages,
                                 pin_sdk_version=True)

with open("myenv.yml","w") as f:
    f.write(myenv.serialize_to_string())

with open("myenv.yml","r") as f:
    print(f.read())

```

### <a name="create-the-scoring-script"></a>Creación del script de puntuación

Escriba un script que cargue el modelo y genere predicciones y explicaciones basadas en un nuevo lote de datos.

```python
%%writefile score.py
import joblib
import pandas as pd
from azureml.core.model import Model
from azureml.train.automl.runtime.automl_explain_utilities import automl_setup_model_explanations


def init():
    global automl_model
    global scoring_explainer

    # Retrieve the path to the model file using the model name
    # Assume original model is named automl_model
    automl_model_path = Model.get_model_path('automl_model')
    scoring_explainer_path = Model.get_model_path('scoring_explainer')

    automl_model = joblib.load(automl_model_path)
    scoring_explainer = joblib.load(scoring_explainer_path)


def run(raw_data):
    data = pd.read_json(raw_data, orient='records')
    # Make prediction
    predictions = automl_model.predict(data)
    # Setup for inferencing explanations
    automl_explainer_setup_obj = automl_setup_model_explanations(automl_model,
                                                                 X_test=data, task='classification')
    # Retrieve model explanations for engineered explanations
    engineered_local_importance_values = scoring_explainer.explain(automl_explainer_setup_obj.X_test_transform)
    # Retrieve model explanations for raw explanations
    raw_local_importance_values = scoring_explainer.explain(automl_explainer_setup_obj.X_test_transform, get_raw=True)
    # You can return any data type as long as it is JSON-serializable
    return {'predictions': predictions.tolist(),
            'engineered_local_importance_values': engineered_local_importance_values,
            'raw_local_importance_values': raw_local_importance_values}
```

### <a name="deploy-the-service"></a>Implementación del servicio

Implementar el servicio con el archivo conda y el archivo de puntuación de los pasos anteriores.

```python
from azureml.core.webservice import Webservice
from azureml.core.webservice import AciWebservice
from azureml.core.model import Model, InferenceConfig
from azureml.core.environment import Environment

aciconfig = AciWebservice.deploy_configuration(cpu_cores=1,
                                               memory_gb=1,
                                               tags={"data": "Bank Marketing",  
                                                     "method" : "local_explanation"},
                                               description='Get local explanations for Bank marketing test data')
myenv = Environment.from_conda_specification(name="myenv", file_path="myenv.yml")
inference_config = InferenceConfig(entry_script="score_local_explain.py", environment=myenv)

# Use configs and models generated above
service = Model.deploy(ws,
                       'model-scoring',
                       [scoring_explainer_model, original_model],
                       inference_config,
                       aciconfig)
service.wait_for_deployment(show_output=True)
```

### <a name="inference-with-test-data"></a>Inferencia con datos de prueba

La inferencia con algunos datos de prueba para ver el valor de predicción del modelo de AutoML, actualmente solo se admite en Azure Machine Learning SDK. Vea la importancia de las características que contribuyen a un valor predicho. 

```python
if service.state == 'Healthy':
    # Serialize the first row of the test data into json
    X_test_json = X_test[:1].to_json(orient='records')
    print(X_test_json)
    # Call the service to get the predictions and the engineered explanations
    output = service.run(X_test_json)
    # Print the predicted value
    print(output['predictions'])
    # Print the engineered feature importances for the predicted value
    print(output['engineered_local_importance_values'])
    # Print the raw feature importances for the predicted value
    print('raw_local_importance_values:\n{}\n'.format(output['raw_local_importance_values']))
```

## <a name="visualize-to-discover-patterns-in-data-and-explanations-at-training-time"></a>Visualización para detectar patrones en datos y explicaciones durante el entrenamiento

Puede visualizar la tabla de importancia de las características en el área de trabajo de [Azure Machine Learning Studio](https://ml.azure.com). Una vez completada la ejecución de AutoML, seleccione **Ver detalles del modelo** para ver una ejecución específica. Seleccione la pestaña **Explicaciones** para obtener las visualizaciones del panel de explicaciones.

[![Arquitectura de interpretabilidad de Machine Learning](./media/how-to-machine-learning-interpretability-automl/automl-explanation.png)](./media/how-to-machine-learning-interpretability-automl/automl-explanation.png#lightbox)

Para obtener más información sobre las visualizaciones de paneles de explicación y trazados específicos, consulte el [documento de procedimientos sobre la interpretación](how-to-machine-learning-interpretability-aml.md).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo se pueden habilitar las explicaciones del modelo y la importancia de las características en áreas del SDK de Azure Machine Learning distintas al aprendizaje automático automatizado, consulte el [artículo de conceptos sobre la interpretabilidad](how-to-machine-learning-interpretability.md).
