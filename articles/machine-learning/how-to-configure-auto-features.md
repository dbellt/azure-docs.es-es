---
title: Caracterización con aprendizaje automático automatizado
titleSuffix: Azure Machine Learning
description: Obtenga información sobre la configuración de la caracterización de datos en Azure Machine Learning y cómo personalizar esas características para los experimentos de aprendizaje automático automatizado.
author: nibaccam
ms.author: nibaccam
ms.reviewer: nibaccam
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.custom: automl,contperf-fy21q2
ms.date: 12/18/2020
ms.openlocfilehash: 563f4e84bf17f749eb4a02bd9a470984e20aa289
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108131447"
---
# <a name="data-featurization-in-automated-machine-learning"></a>Caracterización de datos en aprendizaje automático automatizado

Obtenga información sobre la configuración de la caracterización de datos en Azure Machine Learning, y cómo personalizar esas características para [experimentos de aprendizaje automático automatizado](concept-automated-ml.md).

## <a name="feature-engineering-and-featurization"></a>Ingeniería de características y caracterización

Los datos de entrenamiento constan de filas y columnas. Cada fila es una observación o registro, y las columnas de cada fila son las características que describen cada registro. Normalmente, las características que mejor caracterizan los patrones de los datos se seleccionan para crear modelos predictivos.

Aunque muchos de los campos de datos sin procesar se pueden usar directamente para entrenar un modelo, a menudo es necesario crear características adicionales (de ingeniería) que ofrezcan información que mejor distinga los patrones en los datos. Este proceso se denomina **ingeniería de características**, donde se aprovecha el uso del conocimiento de dominio de los datos para crear características que, a su vez, ayuden a que los algoritmos de aprendizaje automático aprendan mejor. 

En Azure Machine Learning, se aplican técnicas de escalado de datos y normalización para facilitar la ingeniería de características. El conjunto de estas técnicas y la ingeniería de características se conocen como **caracterización** en los experimentos de aprendizaje automático automatizado.

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por supuesto que ya sabe cómo configurar un experimento de ML automatizado. Para obtener información sobre la configuración, vea los artículos siguientes:

- Para obtener una experiencia de codificar por primera vez: [Configuración de experimentos de ML automatizado con el SDK de Azure Machine Learning para Python](how-to-configure-auto-train.md).
- Para obtener una experiencia sin código o con poco código: [Creación, revisión e implementación de modelos de aprendizaje automático automatizado con Azure Machine Learning Studio](how-to-use-automated-ml-for-ml-models.md).

## <a name="configure-featurization"></a>Configuración de la caracterización

En todos los experimentos de aprendizaje automático automatizado se aplican [técnicas de escalado automático y normalización](#featurization) a los datos de forma predeterminada. Estas técnicas son tipos de caracterización que ayudan a *ciertos* algoritmos que son sensibles a las características a diferentes escalas. Puede habilitar una caracterización adicional, como la *atribución de valores que faltan*, la *codificación* y las *transformaciones*.

> [!NOTE]
> Los pasos de la caracterización del aprendizaje automático automatizado (tales como la normalización de características, el control de los datos que faltan o la conversión de valores de texto a numéricos) se convierten en parte del modelo subyacente. Cuando se usa el modelo para realizar predicciones, se aplican automáticamente a los datos de entrada los mismos pasos de caracterización que se aplican durante el entrenamiento.

En el caso de los experimentos configurados con el SDK de Python, se puede habilitar o deshabilitar el valor de caracterización y especificar con más detalle los pasos de caracterización que se van a usar para el experimento. Si usa Azure Machine Learning Studio, vea los [pasos para habilitar la caracterización](how-to-use-automated-ml-for-ml-models.md#customize-featurization).

En la tabla siguiente se muestran los valores aceptados para `featurization` en la [clase AutoMLConfig](/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig):

|Configuración de la caracterización | Descripción|
------------- | ------------- |
|`"featurization": 'auto'`| Especifica que, como parte del preprocesamiento, los [límites de protección de datos](#data-guardrails) y los [pasos de caracterización](#featurization) se van a realizar automáticamente. Esta es la configuración predeterminada.|
|`"featurization": 'off'`| Especifica que los pasos de caracterización no se van a realizar automáticamente.|
|`"featurization":`&nbsp;`'FeaturizationConfig'`| Especifica que se van a usar los pasos de caracterización personalizados. [Aprenda a personalizar la caracterización](#customize-featurization).|

<a name="featurization"></a>

## <a name="automatic-featurization"></a>Características automáticas

En la tabla siguiente se resumen las técnicas que se aplican automáticamente a los datos. Estas técnicas se aplican para los experimentos que se configuran mediante el SDK o Studio. Para deshabilitar este comportamiento, establezca `"featurization": 'off'` en el objeto `AutoMLConfig`.

> [!NOTE]
> Si tiene previsto exportar los modelos creados mediante AutoML a un [modelo de ONNX](concept-onnx.md), solo se admiten las opciones de caracterización indicadas con un asterisco ("*") en el formato ONNX. Más información sobre la [conversión de modelos a ONNX](how-to-use-automl-onnx-model-dotnet.md).

|Pasos de caracterización&nbsp;| Descripción |
| ------------- | ------------- |
|**Eliminación de las características de cardinalidad alta o sin variación*** |Permite eliminar estas características de los conjuntos de entrenamiento y validación. Se aplican a características en las que faltan todos los valores, que tienen el mismo valor en todas las filas o que tienen una cardinalidad alta (por ejemplo, hashes, id. o GUID).|
|**Atribución de los valores que faltan*** |Para las características numéricas, se atribuyen con el promedio de los valores de la columna.<br/><br/>Para las características de categorías, se atribuyen con el valor más frecuente.|
|**Generación de más características** _ |Para las características de fecha y hora: año, mes, día, día de la semana, día del año, trimestre, semana del año, hora, minuto, segundo.<br><br> _En el caso de las tareas de previsión*, se crean estas características adicionales de fecha y hora: año ISO, semestre, mes natural como cadena, semana, día de la semana como cadena, día del trimestre, día del año, AM/PM (0 si la hora es antes de mediodía (12 p. m.); 1, en caso contrario), AM/PM como cadena, hora del día (formato de 12 h)<br/><br/>Para las características de texto: Frecuencia de término basada en unigramas, bigramas y trigramas. Más información sobre [cómo se hace esto con BERT.](#bert-integration)|
|**Transformación y codificación***|Permite transformar las características numéricas con pocos valores únicos en características de categorías.<br/><br/>La codificación "one-hot" se utiliza para las características de categoría de cardinalidad baja. La codificación "one-hot-hash" se utiliza para las características de categorías de cardinalidad alta.|
|**Inserciones de palabras**|Caracterizador de texto que convierte los vectores de tokens de texto en vectores de oración mediante un modelo previamente entrenado. El vector de inserción de cada palabra en un documento se agrega con el resto para producir un vector de característica de documento.|
|**Distancia del clúster**|Entrena un modelo de agrupación en clústeres k-means en todas las columnas numéricas. Genera *k* nuevas características (una característica numérica nueva por grupo), que contienen la distancia de cada muestra hasta el centroide de cada clúster.|

## <a name="data-guardrails"></a>Límites de protección de datos

Los *límites de protección de datos* permiten identificar posibles incidencias con los datos (por ejemplo, valores que faltan o [desequilibrio de clases](concept-manage-ml-pitfalls.md#identify-models-with-imbalanced-data)). También ayudan a tomar acciones correctivas para mejorar los resultados.

Se aplican límites de protección de datos:

- **En el caso de los experimentos de SDK**: cuando se especifican los parámetros `"featurization": 'auto'` o `validation=auto` en el objeto `AutoMLConfig`.
- **En el caso de experimentos de Studio**: cuando se habilita la caracterización automática.

Puede revisar los límites de protección de datos del experimento:

- Mediante el establecimiento de `show_output=True` cuando se envía un experimento con el SDK.

- En Studio, en la pestaña **Límites de protección de datos** de la ejecución del ML automatizado.

### <a name="data-guardrail-states"></a>Estados de límites de protección de datos

Los límites de protección de datos muestran uno de los tres estados siguientes:

|State| Descripción |
|----|---- |
|**Superado**| No se ha detectado ningún problema con los datos y no se requiere ninguna acción por su parte. |
|**Listo**| Los cambios se han aplicado a los datos. Se recomienda revisar las acciones correctivas que ha realizado ML automatizado para asegurarse de que los cambios se alineen con los resultados esperados. |
|**Con alertas**| Se ha detectado una incidencia con los datos, pero no se ha podido resolver. Se recomienda revisar y solucionar la incidencia.|

### <a name="supported-data-guardrails"></a>Límites de protección de datos admitidos

En la tabla siguiente se describen los límites de protección de datos admitidos actualmente, así como los estados asociados que podría ver al enviar el experimento:

Límite de protección|Status|Condición&nbsp;para&nbsp;el desencadenador
---|---|---
**Atribución de los valores de características que faltan** |Superado <br><br><br> ¡Listo!| No se ha detectado que falten valores de característica en los datos de entrenamiento. Obtenga más información sobre la [imputación de valores que faltan](./how-to-use-automated-ml-for-ml-models.md#customize-featurization). <br><br> Se han detectado valores de característica que faltan en los datos de entrenamiento y se han imputado.
**Control de características de cardinalidad alta** |Superado <br><br><br> ¡Listo!| Se han analizado las entradas y no se han detectado características de cardinalidad alta. <br><br> Se han detectado características de cardinalidad alta en las entradas y se han controlado.
**Control de división de validación** |¡Listo!| La configuración de validación se ha establecido en `'auto'` y los datos de entrenamiento contenían *menos de 20 000 filas*. <br> Todas las iteraciones del modelo entrenado se han validado mediante validación cruzada. Obtenga más información sobre la [validación de datos](./how-to-configure-auto-train.md#training-validation-and-test-data). <br><br> La configuración de validación se ha establecido en `'auto'` y los datos de entrenamiento contenían *más de 20 000 filas*. <br> Los datos de entrada se han dividido en un conjunto de datos de entrenamiento y un conjunto de datos de validación para comprobar el modelo.
**Detección de equilibrio de clases** |Superado <br><br><br><br>Con alertas <br><br><br>¡Listo! | Se analizaron las entradas y todas las clases están equilibradas en los datos de entrenamiento. Se considera que un conjunto de datos está equilibrado si todas las clases tienen una representación adecuada en el conjunto de datos según el número y proporción de las muestras. <br><br> Se han detectado clases desequilibradas en las entradas. Para corregir el sesgo del modelo, corrija el problema de equilibrio. Obtenga más información sobre [datos desequilibrados](./concept-manage-ml-pitfalls.md#identify-models-with-imbalanced-data).<br><br> Se detectaron clases desequilibradas en las entradas, y la lógica de barrido ha determinado aplicar el equilibrio.
**Detección de problemas de memoria** |Superado <br><br><br><br> ¡Listo! |<br> Se han analizado los valores seleccionados (horizonte, retardo y ventana con desplazamiento) sin que se hayan detectado incidencias potenciales de memoria insuficiente. Obtenga más información sobre las [configuraciones de previsión](./how-to-auto-train-forecast.md#configuration-settings) de series temporales. <br><br><br>Se han analizado los valores seleccionados (horizonte, retardo y ventana con desplazamiento) y pueden provocar que el experimento se quede sin memoria. Se han desactivado las configuraciones de ventana con desplazamiento o retardo.
**Detección de frecuencias** |Superado <br><br><br><br> ¡Listo! |<br> Se ha analizado la serie temporal y todos los puntos de datos están alineados con la frecuencia detectada. <br> <br> Se ha analizado la serie temporal y se han detectado puntos de datos que no están alineados con la frecuencia detectada. Estos puntos de datos se quitaron del conjunto de datos. Obtenga más información sobre la [preparación de datos para las previsiones de serie temporal](./how-to-auto-train-forecast.md#preparing-data).

## <a name="customize-featurization"></a>Personalización de la caracterización

Se pueden personalizar los valores de la caracterización para asegurarse de que los datos y las características que se usan a fin de entrenar el modelo de Machine Learning generen predicciones pertinentes.

Para personalizar las caracterizaciones, especifique`"featurization": FeaturizationConfig` en el objeto `AutoMLConfig`. Si usa Azure Machine Learning Studio para el experimento, vea el [artículo de procedimiento](how-to-use-automated-ml-for-ml-models.md#customize-featurization). Para personalizar la caracterización de los tipos de tarea de previsión, consulte el [procedimiento de previsión](how-to-auto-train-forecast.md#customize-featurization).

Las personalizaciones compatibles incluyen:

|Personalización|Definición|
|--|--|
|**Actualización del propósito de la columna**|Invalida el tipo de característica detectado automáticamente para la columna especificada.|
|**Actualización de parámetros del transformador** |Actualizar los parámetros para el transformador especificado. Actualmente admite *Imputer* (media, más frecuente y mediana) y *HashOneHotEncoder*.|
|**Quitar columnas** |Especifica las columnas que se van a eliminar de la caracterización.|
|**Transformadores de bloque**| Especifica los transformadores de bloque que se van a usar en el proceso de características.|

>[!NOTE]
> La funcionalidad **Quitar columnas** está en desuso a partir de la versión 1.19 del SDK. Quite columnas del conjunto de datos como parte de la limpieza de datos antes de consumirlos en el experimento de aprendizaje automático automatizado. 

Cree el objeto `FeaturizationConfig` mediante llamadas API:

```python
featurization_config = FeaturizationConfig()
featurization_config.blocked_transformers = ['LabelEncoder']
featurization_config.drop_columns = ['aspiration', 'stroke']
featurization_config.add_column_purpose('engine-size', 'Numeric')
featurization_config.add_column_purpose('body-style', 'CategoricalHash')
#default strategy mean, add transformer param for for 3 columns
featurization_config.add_transformer_params('Imputer', ['engine-size'], {"strategy": "median"})
featurization_config.add_transformer_params('Imputer', ['city-mpg'], {"strategy": "median"})
featurization_config.add_transformer_params('Imputer', ['bore'], {"strategy": "most_frequent"})
featurization_config.add_transformer_params('HashOneHotEncoder', [], {"number_of_bits": 3})
```

## <a name="featurization-transparency"></a>Transparencia de caracterización

Cada modelo AutoML tiene una caracterización aplicada automáticamente.  La caracterización incluye ingeniería de características automatizada (cuando `"featurization": 'auto'`) y escalado y normalización que, a su vez, afecta al algoritmo seleccionado y a sus valores de hiperparámetro. AutoML admite distintos métodos para asegurarse de que tiene visibilidad sobre lo que se aplicó al modelo.

Considere este ejemplo de previsión:

+ Hay cuatro características de entrada: A (numérico), B (numérico), C (numérico), D (fecha y hora).
+ La característica numérica C se puede quitar porque es una columna de identificador con todos los valores únicos.
+ A las características numéricas A y B les faltan valores y, por tanto, los atribuye la media.
+ La característica de fecha y hora D se divide en 11 diferentes características de diseño.

Para obtener esta información, use la salida `fitted_model` de la ejecución del experimento de aprendizaje automático automatizado.

```python
automl_config = AutoMLConfig(…)
automl_run = experiment.submit(automl_config …)
best_run, fitted_model = automl_run.get_output()
```
### <a name="automated-feature-engineering"></a>Diseño de características automatizadas 
`get_engineered_feature_names()` devuelve una lista de nombres de características diseñadas.

  >[!Note]
  >Utilice 'timeseriestransformer' para la tarea = 'forecasting'; en caso contrario, utilice 'datatransformer' para la tarea 'regression' o 'classification'.

  ```python
  fitted_model.named_steps['timeseriestransformer']. get_engineered_feature_names ()
  ```

En esta lista se incluyen todos los nombres de las características de diseño. 

  ```
  ['A', 'B', 'A_WASNULL', 'B_WASNULL', 'year', 'half', 'quarter', 'month', 'day', 'hour', 'am_pm', 'hour12', 'wday', 'qday', 'week']
  ```

`get_featurization_summary()` obtiene un resumen de características de todas las características de entrada.

  ```python
  fitted_model.named_steps['timeseriestransformer'].get_featurization_summary()
  ```

Resultados

  ```
  [{'RawFeatureName': 'A',
    'TypeDetected': 'Numeric',
    'Dropped': 'No',
    'EngineeredFeatureCount': 2,
    'Tranformations': ['MeanImputer', 'ImputationMarker']},
   {'RawFeatureName': 'B',
    'TypeDetected': 'Numeric',
    'Dropped': 'No',
    'EngineeredFeatureCount': 2,
    'Tranformations': ['MeanImputer', 'ImputationMarker']},
   {'RawFeatureName': 'C',
    'TypeDetected': 'Numeric',
    'Dropped': 'Yes',
    'EngineeredFeatureCount': 0,
    'Tranformations': []},
   {'RawFeatureName': 'D',
    'TypeDetected': 'DateTime',
    'Dropped': 'No',
    'EngineeredFeatureCount': 11,
    'Tranformations': ['DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime']}]
  ```

   |Output|Definición|
   |----|--------|
   |RawFeatureName|Nombre de columna o característica de entrada desde el conjunto de datos proporcionado.|
   |TypeDetected|Tipo de datos detectado de la característica de entrada.|
   |Dropped|Indica si la característica de entrada se ha quitado o se ha utilizado.|
   |EngineeringFeatureCount|Número de características que se generan por las transformaciones de diseño de características automatizadas.|
   |Transformaciones|Lista de transformaciones aplicadas a las características de entrada para generar características de diseño.|

### <a name="scaling-and-normalization"></a>Escalado y normalización

Para entender el escalado o normalización y el algoritmo seleccionado con sus valores de hiperparámetro, use `fitted_model.steps`. 

La siguiente salida de ejemplo procede de la ejecución de `fitted_model.steps` para una ejecución elegida:

```
[('RobustScaler', 
  RobustScaler(copy=True, 
  quantile_range=[10, 90], 
  with_centering=True, 
  with_scaling=True)), 

  ('LogisticRegression', 
  LogisticRegression(C=0.18420699693267145, class_weight='balanced', 
  dual=False, 
  fit_intercept=True, 
  intercept_scaling=1, 
  max_iter=100, 
  multi_class='multinomial', 
  n_jobs=1, penalty='l2', 
  random_state=None, 
  solver='newton-cg', 
  tol=0.0001, 
  verbose=0, 
  warm_start=False))
```

Para más información, use esta función auxiliar: 

```python
from pprint import pprint

def print_model(model, prefix=""):
    for step in model.steps:
        print(prefix + step[0])
        if hasattr(step[1], 'estimators') and hasattr(step[1], 'weights'):
            pprint({'estimators': list(
                e[0] for e in step[1].estimators), 'weights': step[1].weights})
            print()
            for estimator in step[1].estimators:
                print_model(estimator[1], estimator[0] + ' - ')
        else:
            pprint(step[1].get_params())
            print()

print_model(model)
```

Esta función auxiliar devuelve el siguiente resultado para una ejecución determinada usando `LogisticRegression with RobustScalar` como algoritmo específico.

```
RobustScaler
{'copy': True,
'quantile_range': [10, 90],
'with_centering': True,
'with_scaling': True}

LogisticRegression
{'C': 0.18420699693267145,
'class_weight': 'balanced',
'dual': False,
'fit_intercept': True,
'intercept_scaling': 1,
'max_iter': 100,
'multi_class': 'multinomial',
'n_jobs': 1,
'penalty': 'l2',
'random_state': None,
'solver': 'newton-cg',
'tol': 0.0001,
'verbose': 0,
'warm_start': False}
```

### <a name="predict-class-probability"></a>Predicción de la probabilidad de las clases

Los modelos generados mediante el aprendizaje automático automatizado tienen objetos contenedores que reflejan la funcionalidad de su clase de origen de código abierto. La mayoría de los objetos contenedores de modelos de clasificación devueltos por el aprendizaje automático automatizado implementan la función `predict_proba()`, que acepta una muestra de datos similares a una matriz o de matriz dispersa de las características (valores X), y devuelve una matriz de n dimensiones de cada muestra y la probabilidad de la clase respectiva.

Suponiendo que ha recuperado el modelo mejor ejecutado y ajustado con las mismas llamadas anteriores, puede llamar a `predict_proba()` directamente desde el modelo ajustado, proporcionando un ejemplo de `X_test` en el formato adecuado según el tipo de modelo.

```python
best_run, fitted_model = automl_run.get_output()
class_prob = fitted_model.predict_proba(X_test)
```

Si el modelo subyacente no admite la función `predict_proba()` o el formato es incorrecto, se producirá una excepción específica de la clase del modelo. Vea los documentos de referencia [RandomForestClassifier](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html#sklearn.ensemble.RandomForestClassifier.predict_proba) y [XGBoost](https://xgboost.readthedocs.io/en/latest/python/python_api.html) para obtener ejemplos de cómo se implementa esta función para los distintos tipos de modelos.

<a name="bert-integration"></a>

## <a name="bert-integration-in-automated-ml"></a>Integración de BERT en el aprendizaje automático automatizado

[BERT](https://techcommunity.microsoft.com/t5/azure-ai/how-bert-is-integrated-into-azure-automated-machine-learning/ba-p/1194657) se usa en la capa de caracterización del aprendizaje automático automatizado. En esta capa, si una columna contiene texto libre u otros tipos de datos, como marcas de tiempo o números simples, la caracterización se aplica en consecuencia.

En el caso de BERT, el modelo se ajusta y se entrena con las etiquetas proporcionadas por el usuario. Desde aquí, las inserciones de documentos se muestran como características junto con otras, como las basadas en marcas de tiempo, día de la semana. 


### <a name="steps-to-invoke-bert"></a>Pasos para invocar BERT

Para invocar BERT, establezca `enable_dnn: True` en automl_settings y use un proceso de GPU (`vm_size = "STANDARD_NC6"` o una GPU superior). Si en lugar de BERT se usa un proceso de CPU, AutoML habilita el caracterizador BiLSTM DNN.

AutoML realiza los siguientes pasos para BERT. 

1. **El preprocesamiento y la tokenización de todas las columnas de texto**. Por ejemplo, el transformador "StringCast" se puede encontrar en el resumen de la caracterización del modelo final. Puede encontrar un ejemplo de cómo generar el resumen de caracterización del modelo en [este cuaderno](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/classification-text-dnn/auto-ml-classification-text-dnn.ipynb).

2. **Concatene todas las columnas de texto en una sola columna de texto**, por lo tanto, `StringConcatTransformer` en el modelo final. 

    Nuestra implementación de BERT limita la longitud total del texto de un ejemplo de entrenamiento a 128 tokens. Esto significa que todas las columnas de texto, cuando se concatenan, deben tener una longitud máxima de 128 tokens. Si existen varias columnas, cada columna debe eliminarse de forma que se satisfaga esta condición. En caso contrario, en las columnas concatenadas de más de 128 tokens, la capa del tokenizador de BERT trunca esta entrada a 128 tokens.

3. **Como parte del barrido de características, AutoML compara BERT con la línea de base (características de contenedor de palabras) en una muestra de los datos.** Esta comparación determina si BERT proporcionaría mejoras en la precisión. Si BERT funciona mejor que la línea de base, AutoML usa BERT para la caracterización de texto para todos los datos. En ese caso, verá `PretrainedTextDNNTransformer` en el modelo final.

Normalmente, BERT se ejecuta más tiempo que otros administradores de características. Para mejorar el rendimiento, se recomienda usar "STANDARD_NC24r" o "STANDARD_NC24rs_V3" para sus funcionalidades de RDMA. 

AutoML distribuirá el entrenamiento de BERT en varios nodos, si están disponibles (hasta un máximo de ocho nodos). Esto se puede hacer en el objeto `AutoMLConfig` estableciendo el parámetro `max_concurrent_iterations` en un valor mayor que 1. 

## <a name="supported-languages-for-bert-in-automl"></a>Idiomas compatibles para BERT en el aprendizaje automático automatizado 

El aprendizaje automático automatizado admite actualmente unos 100 idiomas y, en función del idioma del conjunto de datos, elige el modelo BERT adecuado. Para los datos en alemán, se usa el modelo BERT en alemán. Para inglés, se usa el modelo BERT en inglés. Para todos los demás idiomas, se usa el modelo multilingüe de BERT.

En el código siguiente, se desencadena el modelo BERT en alemán, ya que el idioma del conjunto de datos se especifica como `deu`, el código de idioma de 3 letras para el alemán según la [clasificación ISO](https://iso639-3.sil.org/code/deu):

```python
from azureml.automl.core.featurization import FeaturizationConfig

featurization_config = FeaturizationConfig(dataset_language='deu')

automl_settings = {
    "experiment_timeout_minutes": 120,
    "primary_metric": 'accuracy', 
# All other settings you want to use 
    "featurization": featurization_config,
    
  "enable_dnn": True, # This enables BERT DNN featurizer
    "enable_voting_ensemble": False,
    "enable_stack_ensemble": False
}
```

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo configurar los experimentos de ML automatizado:

    * Para obtener una experiencia de codificar por primera vez: [Configuración de experimentos de ML automatizado con el SDK de Azure Machine Learning](how-to-configure-auto-train.md).
    * Para obtener una experiencia sin código o con poco código: [Creación de experimentos de aprendizaje automático automatizado en Azure Machine Learning Studio](how-to-use-automated-ml-for-ml-models.md).

* Obtenga más información sobre [cómo y dónde implementar un modelo](how-to-deploy-and-where.md).

* Obtenga más información sobre los [procedimientos para entrenar un modelo de regresión con aprendizaje automático automatizado](tutorial-auto-train-models.md) o los [procedimientos para entrenar con aprendizaje automático automatizado en un recurso remoto](concept-automated-ml.md#local-remote).
