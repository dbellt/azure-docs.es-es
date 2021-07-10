---
title: Procedimientos recomendados para usar las API multivariantes de Anomaly Detector
titleSuffix: Azure Cognitive Services
description: Procedimientos recomendados para usar las API multivariantes de Anomaly Detector para aplicar la detección de anomalías a los datos de series temporales.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: conceptual
ms.date: 04/01/2021
ms.author: mbullwin
keywords: anomaly detection, machine learning, algorithms
ms.openlocfilehash: 30778cf48efda57fc0d50964611d5616ce7a84d5
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110062261"
---
# <a name="multivariate-time-series-anomaly-detector-best-practices"></a>Procedimientos recomendados de Anomaly Detector para series temporales multivariantes

En este artículo se proporcionan instrucciones sobre los procedimientos recomendados que deben seguirse al usar las API multivariantes de Anomaly Detector.

## <a name="training-data"></a>Datos de aprendizaje. 

### <a name="data-schema"></a>Esquema de datos
Para usar las API multivariantes de Anomaly Detector, primero debe entrenar sus propios modelos. Los datos de entrenamiento son un conjunto de varias series temporales que cumplen los siguientes requisitos:

Cada serie temporal debe ser un archivo CSV con dos columnas (solo dos) y **"timestamp"** y **"value"** (todo en minúsculas) como fila de encabezado. Los valores de "timestamp" deben cumplir la norma ISO 8601; los de "value" pueden ser números enteros o decimales con cualquier número de posiciones decimales. Por ejemplo:

|timestamp | value|
|-------|-------|
|2019-04-01T00:00:00Z| 5|
|2019-04-01T00:01:00Z| 3.6|
|2019-04-01T00:02:00Z| 4|
|`...`| `...` |

Cada archivo CSV debe tener el nombre de una variable diferente que se usará para el entrenamiento del modelo. Por ejemplo, "temperature.csv" y "humidity.csv". Todos los archivos CSV deben comprimirse en un archivo ZIP sin subcarpetas. El archivo ZIP puede tener el nombre que desee. El archivo ZIP debe cargarse en Azure Blob Storage. Una vez que genere la [dirección URL de SAS de blob (firmas de acceso compartido)](../../../storage/common/storage-sas-overview.md) para el archivo zip, lo puede usar para el entrenamiento. Consulte este documento para obtener información sobre cómo generar direcciones URL de SAS a partir de Azure Blob Storage.

### <a name="data-quality"></a>Calidad de los datos
- A medida que el modelo aprende los patrones normales a partir de los datos históricos, los datos de entrenamiento deben **representar el estado normal general del sistema**. Si hay una gran cantidad de anomalías en los datos de entrenamiento, es difícil que el modelo aprenda estos tipos de patrones. 
-  El modelo tiene millones de parámetros y necesita un número mínimo de puntos de datos para aprender un conjunto óptimo de parámetros. Por regla general, debe proporcionar **al menos 15 000 puntos de datos por variable** para entrenar el modelo correctamente. Cuantos más datos, mejor será el modelo.
- En general, la **proporción de valores que faltan de los datos de entrenamiento debe ser inferior a un 20 %.** El hecho de que falten demasiados datos puede generar valores rellenados de forma automática (normalmente segmentos rectos o valores constantes) que se aprenden como patrones normales. Esto puede dar lugar a que se detecten puntos de datos reales como anomalías. 

    Sin embargo, hay casos en los que una proporción alta es aceptable. Por ejemplo, si tiene dos series temporales en un grupo que utilizan el modo `Outer` para alinear las marcas de tiempo. Una tiene una granularidad de un minuto y la otra, una granularidad por hora. La serie temporal por hora tiene por naturaleza al menos 59 / 60 = 98,33 % de puntos de datos que faltan. En estos casos, es correcto rellenar la serie temporal por hora con el único valor disponible si no suele fluctuar demasiado.

## <a name="parameters"></a>Parámetros

### <a name="sliding-window"></a>Ventana deslizante

La detección de anomalías multivariantes toma un segmento de puntos de datos de longitud `slidingWindow` como entrada y decide si el punto de datos siguiente es una anomalía. Cuanto mayor sea la longitud de la muestra, más datos se tendrán en cuenta para tomar una decisión. Debe considerar dos cosas al elegir un valor adecuado para `slidingWindow`: las propiedades de los datos de entrada y la compensación entre el tiempo de entrenamiento o inferencia y la posible mejora del rendimiento. El valor de `slidingWindow` está compuesto por un entero entre 28 y 2880. Puede decidir cuántos puntos de datos se usan como entradas en función de si los datos son periódicos y de la frecuencia de muestreo de los datos.

Cuando los datos sean periódicos, puede incluir de uno a tres ciclos como entrada y, cuando el muestreo de los datos se realice con una frecuencia alta (granularidad pequeña), por ejemplo, a nivel de minutos o de segundos, puede seleccionar más datos como entrada. Otro problema es que las entradas más largas pueden generar un tiempo de entrenamiento o inferencia superior y no hay ninguna garantía de que un mayor número de puntos de entrada dé lugar a mejoras de rendimiento. Por otra parte, si hay muy pocos puntos de datos, puede dificultar que el modelo converja en una solución óptima. Por ejemplo, es difícil detectar anomalías cuando los datos de entrada solo tienen dos puntos.

### <a name="align-mode"></a>Modo de alineación

El parámetro `alignMode` se usa para indicar cómo quiere alinear varias series temporales en marcas de tiempo. Esto se debe a que en muchas series temporales faltan valores y es necesario alinearlas en las mismas marcas de tiempo antes de continuar con el procesamiento. Existen dos opciones para este parámetro, `inner join` y `outer join`. La opción `inner join` significa que se notificarán los resultados de la detección en marcas de tiempo en las que **cada serie temporal** tiene un valor, mientras que `outer join` significa que se notificarán los resultados de la detección en marcas de tiempo para **cualquier serie temporal** que tenga un valor.  **El valor `alignMode` también afectará a la secuencia de entrada del modelo**; por tanto, elija un valor `alignMode` adecuado para su escenario, ya que los resultados pueden ser muy diferentes.

Aquí se muestra un ejemplo para explicar los distintos valores de `alignModel`.

#### <a name="series1"></a>Serie1

|timestamp | value|
----------| -----|
|`2020-11-01`| 1  
|`2020-11-02`| 2  
|`2020-11-04`| 4  
|`2020-11-05`| 5

#### <a name="series2"></a>Serie2

timestamp | value  
--------- | -
`2020-11-01`| 1  
`2020-11-02`| 2  
`2020-11-03`| 3  
`2020-11-04`| 4

#### <a name="inner-join-two-series"></a>Combinación interna de dos series
  
timestamp | Serie1 | Serie2
----------| - | -
`2020-11-01`| 1 | 1
`2020-11-02`| 2 | 2
`2020-11-04`| 4 | 4

#### <a name="outer-join-two-series"></a>Combinación externa de dos series

timestamp | serie1 | serie2
--------- | - | -
`2020-11-01`| 1 | 1
`2020-11-02`| 2 | 2
`2020-11-03`| N/D | 3
`2020-11-04`| 4 | 4
`2020-11-05`| 5 | N/D

### <a name="fill-not-available-na"></a>Relleno Not available (NA)

Una vez alineadas las variables en la marca de tiempo por combinación externa, es posible que algunas de las variables tengan el valor `Not Available` (`NA`). Puede especificar el método para rellenar este valor NA. Las opciones para `fillNAMethod` son `Linear`, `Previous`, `Subsequent`, `Zero` y `Fixed`.

| Opción     | Método                                                                                           |
| ---------- | -------------------------------------------------------------------------------------------------|
| Lineal     | Rellenar los valores NA por interpolación lineal                                                           |
| Anterior   | Propagar el último valor válido para rellenar los espacios. Ejemplo: `[1, 2, nan, 3, nan, 4]` -> `[1, 2, 2, 3, 3, 4]` |
| Subsequent | Usar el siguiente valor válido para rellenar los espacios. Ejemplo: `[1, 2, nan, 3, nan, 4]` -> `[1, 2, 3, 3, 4, 4]`       |
| Cero       | Rellenar los valores NA con 0.                                                                           |
| Fijo      | Rellenar los valores NA con un valor válido especificado que debe proporcionarse en `paddingValue`.          |

## <a name="model-analysis"></a>Análisis de modelos

### <a name="training-latency"></a>Latencia de entrenamiento

El entrenamiento de detección de anomalías multivariantes puede ser un proceso lento. Sobre todo, cuando se ha usado una gran cantidad de marcas de tiempo para realizarlo. Por lo tanto, se permite que parte del proceso de entrenamiento se lleve a cabo de forma asincrónica. Normalmente, los usuarios envían la tarea de entrenamiento a través de la API de entrenamiento de modelos. A continuación, se obtiene el estado del modelo a través de `Get Multivariate Model API`. Aquí se muestra cómo extraer el tiempo restante antes de que se complete el entrenamiento. En la respuesta de la API para obtención de modelos multivariantes, hay un elemento que se llama `diagnosticsInfo`. Este, a su vez, tiene un elemento `modelState`. Para calcular el tiempo restante, debemos usar `epochIds` y `latenciesInSeconds`. Una época representa un ciclo completo a través de los datos de entrenamiento. Cada 10 épocas, se generará información de estado. En total, se entrenará durante 100 épocas; la latencia indica cuánto dura una época. Con esta información, sabemos el tiempo que queda para entrenar el modelo.

### <a name="model-performance"></a>Rendimiento del modelo

Detección de anomalías multivariantes, como un modelo no supervisado. La mejor forma de evaluarlo es comprobar los resultados de las anomalías manualmente. En la respuesta de obtención de modelos multivariantes, proporcionamos información básica para analizar el rendimiento del modelo. En el elemento `modelState` que devuelve la API para obtención de modelos multivariantes (Get Multivariate Model API), podemos usar `trainLosses` y `validationLosses` para evaluar si el modelo se ha entrenado según lo previsto. En la mayoría de los casos, ambos elementos de pérdida disminuirán de forma gradual. Otro fragmento de información en el que analizar el rendimiento del modelo es en `variableStates`. La lista de estados de variables se clasifica por `filledNARatio` en orden descendente. Cuanto más grande sea, peor será el rendimiento; normalmente es necesario reducir el valor `NA ratio` tanto como sea posible. La aparición de `NA` puede deberse a valores que faltan o a variables no alineadas desde una perspectiva de marca de tiempo.

## <a name="next-steps"></a>Pasos siguientes

- [Guías de inicio rápido](../quickstarts/client-libraries-multivariate.md)
- [Obtener información sobre los algoritmos subyacentes que se usan en Anomaly Detector para multivariantes](https://arxiv.org/abs/2009.02040)
