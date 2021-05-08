---
title: 'Ejemplo: creación e implementación de una aptitud personalizada con el diseñador de Azure Machine Learning'
titleSuffix: Azure Cognitive Search
description: En este ejemplo se muestra cómo usar el diseñador de Azure Machine Learning para compilar e implementar una aptitud personalizada de AML para la canalización de enriquecimiento con IA de Azure Cognitive Search.
manager: luiscab
author: vkurpad
ms.author: vikurpad
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/16/2021
ms.openlocfilehash: 7d73bfa4b184cb2a3d1cad9a4b6f7fb918c9f953
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107989510"
---
# <a name="example-build-and-deploy-a-custom-skill-with-azure-machine-learning-designer"></a>Ejemplo: creación e implementación de una aptitud personalizada con el diseñador de Azure Machine Learning

[Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/concept-designer) es un lienzo interactivo fácil de usar para crear modelos de aprendizaje automático para tareas como la regresión y la clasificación. Invocar el modelo creado por el diseñador en una canalización de enriquecimiento de Cognitive Search requiere algunos pasos adicionales. En este ejemplo, creará un modelo de regresión simple para predecir el precio de un automóvil e invocará el punto de conexión de inferencia como una aptitud de AML. 

Siga el tutorial [Regresión: predicción de precios de automóviles (avanzado)](https://github.com/Azure/MachineLearningDesigner/blob/master/articles/samples/regression-automobile-price-prediction-compare-algorithms.md) en la página de documentación de [Conjuntos de datos y canalizaciones de ejemplo](https://docs.microsoft.com/azure/machine-learning/samples-designer) para crear un modelo que predice el precio de un automóvil dadas las distintas características.

> [!IMPORTANT] 
> La implementación del modelo después del proceso de inferencia en tiempo real dará como resultado un punto de conexión válido, pero no uno que pueda usar con la aptitud de AML en Cognitive Search. 

## <a name="register-model-and-download-assets"></a>Registro del modelo y descarga de recursos

Una vez que tenga un modelo entrenado,[regístrelo](https://docs.microsoft.com/azure/machine-learning/how-to-deploy-model-designer) y siga los pasos para descargar todos los archivos de la carpeta `trained_model_outputs` o descargar solo los archivos `score.py` y `conda_env.yml` de la página de artefactos de modelos. Editará el script de puntuación antes de implementar el modelo como un punto de conexión de inferencia en tiempo real.


## <a name="edit-the-scoring-script-for-use-with-cognitive-search"></a>Edición del script de puntuación para usarlo con Cognitive Search 

Las canalizaciones de enriquecimiento de Cognitive Search funcionan en un único documento y generan una solicitud que contiene las entradas para una única predicción. El archivo `score.py` descargado acepta una lista de registros y devuelve una lista de predicciones como una cadena JSON serializada. Realizará dos cambios en el archivo `score.py`.

* Edite el script para trabajar con un único registro de entrada, no con una lista.
* Edite el script para devolver un objeto JSON con una sola propiedad, el precio previsto.

Abra el archivo `score.py` descargado y edite la función `run(data)`. La función está actualmente configurada para esperar la siguiente entrada, tal y como se describe en el archivo `_samples.json` del modelo.

```json
[
  {
    "symboling": 2,
    "make": "mitsubishi",
    "fuel-type": "gas",
    "aspiration": "std",
    "num-of-doors": "two",
    "body-style": "hatchback",
    "drive-wheels": "fwd",
    "engine-location": "front",
    "wheel-base": 93.7,
    "length": 157.3,
    "width": 64.4,
    "height": 50.8,
    "curb-weight": 1944,
    "engine-type": "ohc",
    "num-of-cylinders": "four",
    "engine-size": 92,
    "fuel-system": "2bbl",
    "bore": 2.97,
    "stroke": 3.23,
    "compression-ratio": 9.4,
    "horsepower": 68.0,
    "peak-rpm": 5500.0,
    "city-mpg": 31,
    "highway-mpg": 38,
    "price": 6189.0
  },
  {
    "symboling": 0,
    "make": "toyota",
    "fuel-type": "gas",
    "aspiration": "std",
    "num-of-doors": "four",
    "body-style": "wagon",
    "drive-wheels": "fwd",
    "engine-location": "front",
    "wheel-base": 95.7,
    "length": 169.7,
    "width": 63.6,
    "height": 59.1,
    "curb-weight": 2280,
    "engine-type": "ohc",
    "num-of-cylinders": "four",
    "engine-size": 92,
    "fuel-system": "2bbl",
    "bore": 3.05,
    "stroke": 3.03,
    "compression-ratio": 9.0,
    "horsepower": 62.0,
    "peak-rpm": 4800.0,
    "city-mpg": 31,
    "highway-mpg": 37,
    "price": 6918.0
  },
  {
    "symboling": 1,
    "make": "honda",
    "fuel-type": "gas",
    "aspiration": "std",
    "num-of-doors": "two",
    "body-style": "sedan",
    "drive-wheels": "fwd",
    "engine-location": "front",
    "wheel-base": 96.5,
    "length": 169.1,
    "width": 66.0,
    "height": 51.0,
    "curb-weight": 2293,
    "engine-type": "ohc",
    "num-of-cylinders": "four",
    "engine-size": 110,
    "fuel-system": "2bbl",
    "bore": 3.15,
    "stroke": 3.58,
    "compression-ratio": 9.1,
    "horsepower": 100.0,
    "peak-rpm": 5500.0,
    "city-mpg": 25,
    "highway-mpg": 31,
    "price": 10345.0
  }
]
```

Los cambios garantizarán que el modelo pueda aceptar la entrada generada por Cognitive Search durante la indexación, que es un único registro.

```json
{
    "symboling": 2,
    "make": "mitsubishi",
    "fuel-type": "gas",
    "aspiration": "std",
    "num-of-doors": "two",
    "body-style": "hatchback",
    "drive-wheels": "fwd",
    "engine-location": "front",
    "wheel-base": 93.7,
    "length": 157.3,
    "width": 64.4,
    "height": 50.8,
    "curb-weight": 1944,
    "engine-type": "ohc",
    "num-of-cylinders": "four",
    "engine-size": 92,
    "fuel-system": "2bbl",
    "bore": 2.97,
    "stroke": 3.23,
    "compression-ratio": 9.4,
    "horsepower": 68.0,
    "peak-rpm": 5500.0,
    "city-mpg": 31,
    "highway-mpg": 38,
    "price": 6189.0
}
```

Reemplace las líneas 27 a 30 por
```python

    for key, val in data.items():
        input_entry[key].append(decode_nan(val))
```
También tendrá que editar la salida que el script genera de una cadena a un objeto JSON. Edite la instrucción "return" (línea 37) en el archivo original como se indica a continuación:
```python
    output = result.data_frame.values.tolist()
    return {
        "predicted_price": output[0][-1]
    }
```

Esta es la función `run` actualizada con los cambios en el formato de entrada y la salida prevista que aceptará un único registro como entrada y devolverá un objeto JSON con el precio previsto.

```python
def run(data):
    data = json.loads(data)
    input_entry = defaultdict(list)
    # data is now a JSON object not a list of JSON objects
    for key, val in data.items():
        input_entry[key].append(decode_nan(val))

    data_frame_directory = create_dfd_from_dict(input_entry, schema_data)
    score_module = ScoreModelModule()
    result, = score_module.run(
        learner=model,
        test_data=DataTable.from_dfd(data_frame_directory),
        append_or_result_only=True)
    #return json.dumps({"result": result.data_frame.values.tolist()})
    output = result.data_frame.values.tolist()
    # return the last column of the the first row of the dataframe
    return  {
        "predicted_price": output[0][-1]
    }
```
## <a name="register-and-deploy-the-model"></a>Registro e implementación del modelo

Con los cambios guardados, ahora puede registrar el modelo en el portal. Seleccione Registrar modelo y proporcione un nombre válido. Elija `Other` para Marco del modelo, `Custom` para Nombre del marco y `1.0` para Versión del marco. Seleccione la opción `Upload folder` y seleccione la carpeta con los archivos `score.py` y `conda_env.yaml` actualizados.

Seleccione el modelo y seleccione la acción `Deploy`. En el paso de implementación se da por supuesto que tiene un clúster de inferencia de AKS aprovisionado. Las instancias de contenedor no se admiten actualmente en Cognitive Search.
 1. Proporcione un nombre de punto de conexión válido.
2. Seleccione el tipo de proceso de `Azure Kubernetes Service`.
3. Seleccione el nombre de proceso para el clúster de inferencia.
4. Cambie `enable authentication` a Activado.
5. En Tipo, seleccione `Key-based authentication`.
6. Seleccione el archivo `score.py` actualizado para `entry script file`.
7. Seleccione `conda_env.yaml` para `conda dependencies file`.
8. Seleccione el botón Implementar para implementar el nuevo punto de conexión.

## <a name="integrate-with-cognitive-search"></a>Integración con Cognitive Search

Para integrar el punto de conexión recién creado con Cognitive Search:
1. Agregue un archivo JSON que contenga un único registro de automóvil a un contenedor de blobs.
2. Configure una canalización de enriquecimiento con IA mediante el [flujo de trabajo de importación de datos](https://docs.microsoft.com/azure/search/cognitive-search-quickstart-blob). No olvide seleccionar `JSON` como `parsing mode`.
3. En la pestaña `Add Enrichments`, seleccione una sola aptitud `Extract people names` como marcador de posición.
4. Agregue un nuevo campo al índice denominado `predicted_price` de tipo `Edm.Double` y establezca la propiedad Retrievable en true.
5. Complete el proceso de importación de datos.

### <a name="add-the-aml-skill-to-the-skillset"></a>Adición de la aptitud de AML al conjunto de aptitudes

En la lista de conjuntos de aptitudes, seleccione el conjunto de aptitudes que ha creado. Ahora editará el conjunto de aptitudes para reemplazar la aptitud de identificación de personas por la aptitud de AML para predecir los precios. En la pestaña Definición del conjunto de aptitudes (JSON), seleccione `Azure Machine Learning (AML)` en la lista desplegable Aptitudes. Seleccione el área de trabajo. Para que la aptitud de AML detecte el punto de conexión, el área de trabajo y el servicio de búsqueda deben estar en la misma suscripción de Azure.
Seleccione el punto de conexión que creó anteriormente en el tutorial.
Compruebe que la aptitud se rellena con el URI y la información de autenticación tal como se configuró al implementar el punto de conexión. Copie la plantilla de aptitud y reemplace la aptitud en el conjunto de aptitudes.
Edite la aptitud según se indica a continuación:
1. Establezca el nombre en un nombre válido.
2. Agregar una descripción
3. Establezca degreesOfParallelism en 1.
4. Establezca el contexto en `/document`.
5. Establezca las entradas en todas las entradas necesarias y consulte la definición de aptitud de ejemplo que se indica a continuación.
6. Establezca las salidas para capturar el precio previsto devuelto.

```json
{
      "@odata.type": "#Microsoft.Skills.Custom.AmlSkill",
      "name": "AMLdemo",
      "description": "AML Designer demo",
      "context": "/document",
      "uri": "Your AML endpoint",
      "key": "Your AML endpoint key",
      "resourceId": null,
      "region": null,
      "timeout": "PT30S",
      "degreeOfParallelism": 1,
      "inputs": [
        {
          "name": "symboling",
          "source": "/document/symboling"
        },
        {
          "name": "make",
          "source": "/document/make"
        },
        {
          "name": "fuel-type",
          "source": "/document/fuel-type"
        },
        {
          "name": "aspiration",
          "source": "/document/aspiration"
        },
        {
          "name": "num-of-doors",
          "source": "/document/num-of-doors"
        },
        {
          "name": "body-style",
          "source": "/document/body-style"
        },
        {
          "name": "drive-wheels",
          "source": "/document/drive-wheels"
        },
        {
          "name": "engine-location",
          "source": "/document/engine-location"
        },
        {
          "name": "wheel-base",
          "source": "/document/wheel-base"
        },
        {
          "name": "length",
          "source": "/document/length"
        },
        {
          "name": "width",
          "source": "/document/width"
        },
        {
          "name": "height",
          "source": "/document/height"
        },
        {
          "name": "curb-weight",
          "source": "/document/curb-weight"
        },
        {
          "name": "engine-type",
          "source": "/document/engine-type"
        },
        {
          "name": "num-of-cylinders",
          "source": "/document/num-of-cylinders"
        },
        {
          "name": "engine-size",
          "source": "/document/engine-size"
        },
        {
          "name": "fuel-system",
          "source": "/document/fuel-system"
        },
        {
          "name": "bore",
          "source": "/document/bore"
        },
        {
          "name": "stroke",
          "source": "/document/stroke"
        },
        {
          "name": "compression-ratio",
          "source": "/document/compression-ratio"
        },
        {
          "name": "horsepower",
          "source": "/document/horsepower"
        },
        {
          "name": "peak-rpm",
          "source": "/document/peak-rpm"
        },
        {
          "name": "city-mpg",
          "source": "/document/city-mpg"
        },
        {
          "name": "highway-mpg",
          "source": "/document/highway-mpg"
        },
        {
          "name": "price",
          "source": "/document/price"
        }
      ],
      "outputs": [
        {
          "name": "predicted_price",
          "targetName": "predicted_price"
        }
      ]
    }
```
### <a name="update-the-indexer-output-field-mappings"></a>Actualización de las asignaciones de campos de salida del indexador

Las asignaciones de campos de salida del indexador determinan qué enriquecimientos se guardan en el índice. Reemplace la sección de asignaciones de campos de salida del indexador por el fragmento de código siguiente:

```json
"outputFieldMappings": [
    {
      "sourceFieldName": "/document/predicted_price",
      "targetFieldName": "predicted_price"
    }
  ]
```

Ahora puede ejecutar el indexador y validar que la propiedad `predicted_price` se rellena en el índice con el resultado de la salida de la aptitud de AML.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Revisión de la API web de aptitudes personalizadas](./cognitive-search-custom-skill-web-api.md)

> [Más información sobre cómo agregar aptitudes personalizadas a la canalización de enriquecimiento](./cognitive-search-custom-skill-interface.md)

> [Más información sobre la aptitud de AML](./cognitive-search-tutorial-aml-custom-skill.md)
