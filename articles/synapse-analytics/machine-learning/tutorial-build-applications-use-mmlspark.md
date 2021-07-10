---
title: 'Tutorial: Creación de aplicaciones de aprendizaje automático con Microsoft Machine Learning para Apache Spark (versión preliminar)'
description: Aprenda a usar Microsoft Machine Learning para Apache Spark para crear aplicaciones de aprendizaje automático en Azure Synapse Analytics.
services: synapse-analytics
ms.service: synapse-analytics
ms.subservice: machine-learning
ms.topic: tutorial
ms.reviewer: ''
ms.date: 03/08/2021
author: ruixinxu
ms.author: ruxu
ms.openlocfilehash: 8a4c5892e5b7b542177376fcd0adae76527957a8
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110578097"
---
# <a name="tutorial-build-machine-learning-applications-using-microsoft-machine-learning-for-apache-spark-preview"></a>Tutorial: Creación de aplicaciones de aprendizaje automático con Microsoft Machine Learning para Apache Spark (versión preliminar)

En este artículo aprenderá a usar Microsoft Machine Learning para Apache Spark ([MMLSpark](https://github.com/Azure/mmlspark)) para crear aplicaciones de aprendizaje automático. MMLSpark amplía la solución de aprendizaje automático distribuida de Apache Spark al agregar un gran número de herramientas de ciencia de datos y aprendizaje profundo, como [Azure Cognitive Services](../../cognitive-services/big-data/cognitive-services-for-big-data.md), [OpenCV](https://opencv.org/), [LightGBM](https://github.com/Microsoft/LightGBM), etc.  MMLSpark permite crear modelos analíticos y predictivos eficaces y altamente escalables a partir de diversos orígenes de datos de Spark.
Synapse Spark proporciona bibliotecas de MMLSpark integradas, entre las que se incluyen:

- [Vowpal Wabbit](https://github.com/Azure/mmlspark/blob/master/docs/vw.md): servicios de biblioteca para habilitar el análisis de texto, como el análisis de sentimiento en los tweets, en el aprendizaje automático.
- [Cognitive Services en Spark](https://github.com/Azure/mmlspark/blob/master/docs/cogsvc.md): para combinar la característica de Azure Cognitive Services con las canalizaciones de SparkML para ofrecer diseño de soluciones para los servicios de modelado de datos cognitivos, como la detección de anomalías.
- [LightGBM](https://github.com/Azure/mmlspark/blob/master/docs/lightgbm.md): LightGBM es un marco de potenciación del gradiente que usa algoritmos de aprendizaje basados en árboles. Está diseñado para distribuirse y ofrecer una mayor eficacia.
- KKN condicional: modelos de KNN escalables con consultas condicionales.
- [HTTP en Spark](https://github.com/Azure/mmlspark/blob/master/docs/http.md): habilita la orquestación de microservicios distribuida al integrar la accesibilidad basada en el protocolo HTTP y en Spark.

En este tutorial se describen ejemplos de uso de Azure Cognitive Services en MMLSpark para: 

- Text Analytics: para obtener la opinión (o el estado de ánimo) de un conjunto de frases.
- Computer Vision: para obtener las etiquetas (descripciones de una sola palabra) asociadas a un conjunto de imágenes.
- Bing Image Search: para buscar en la Web las imágenes relacionadas con una consulta en lenguaje natural.
- Anomaly Detector: para detectar anomalías en los datos de una serie temporal.

Si no tiene una suscripción a Azure, [cree una cuenta gratuita antes de empezar](https://azure.microsoft.com/free/).

## <a name="prerequisites"></a>Requisitos previos 

- Necesitará un [área de trabajo de Azure Synapse Analytics](../get-started-create-workspace.md) con una cuenta de almacenamiento de Azure Data Lake Storage Gen2 que esté configurada como almacenamiento predeterminado. Asegúrese de que es el *colaborador de datos de Storage Blob* en el sistema de archivos de Data Lake Storage Gen2 con el que trabaja.
- Grupo de Spark en el área de trabajo de Azure Synapse Analytics. Para más información, consulte el artículo sobre [creación de un grupo de Spark en Azure Synapse](../get-started-analyze-spark.md).
- Haber completado los pasos de configuración previos que se detallan en este tutorial sobre la [configuración de Cognitive Services en Azure Synapse](./tutorial-configure-cognitive-services-synapse.md).


## <a name="get-started"></a>Introducción
Para empezar, importe mmlspark y configure las claves del servicio. 

```python
import mmlspark
mmlspark.__spark_package_version__ # current version: 1.0.0-rc3-6-a862d6b1-SNAPSHOT

from mmlspark.cognitive import *
from notebookutils import mssparkutils

# A general Cognitive Services key for Text Analytics and Computer Vision (or use separate keys that belong to each service)
cognitive_service_key = mssparkutils.credentials.getSecret("ADD_YOUR_KEY_VAULT_NAME", "ADD_YOUR_SERVICE_KEY","ADD_YOUR_KEY_VAULT_LINKED_SERVICE_NAME") 
# A Bing Search v7 subscription key
bingsearch_service_key = mssparkutils.credentials.getSecret("ADD_YOUR_KEY_VAULT_NAME", "ADD_YOUR_BING_SEARCH_KEY","ADD_YOUR_KEY_VAULT_LINKED_SERVICE_NAME")
# An Anomaly Dectector subscription key
anomalydetector_key = mssparkutils.credentials.getSecret("ADD_YOUR_KEY_VAULT_NAME", "ADD_YOUR_ANOMALY_KEY","ADD_YOUR_KEY_VAULT_LINKED_SERVICE_NAME")


```


## <a name="text-analytics-sample"></a>Ejemplo de Text Analytics

El servicio [Text Analytics](../../cognitive-services/text-analytics/index.yml) proporciona varios algoritmos para extraer conclusiones inteligentes de un texto. Por ejemplo, podemos encontrar la opinión de un texto de entrada determinado. El servicio devolverá una puntuación entre 0,0 y 1,0, donde las puntuaciones bajas indican una opinión negativa y una puntuación alta indica una opinión positiva. Este ejemplo utiliza tres frases simples y devuelve la opinión de cada una.

```python
from pyspark.sql.functions import col

# Create a dataframe that's tied to it's column names
df_sentences = spark.createDataFrame([
  ("I am so happy today, its sunny!", "en-US"), 
  ("this is a dog", "en-US"), 
  ("I am frustrated by this rush hour traffic!", "en-US") 
], ["text", "language"])

# Run the Text Analytics service with options
sentiment = (TextSentiment()
    .setTextCol("text")
    .setLocation("eastasia") # Set the location of your cognitive service
    .setSubscriptionKey(cognitive_service_key)
    .setOutputCol("sentiment")
    .setErrorCol("error")
    .setLanguageCol("language"))

# Show the results of your text query in a table format

display(sentiment.transform(df_sentences).select("text", col("sentiment")[0].getItem("sentiment").alias("sentiment")))
```

### <a name="expected-results"></a>Resultados esperados

|text | opinión|
|--|--|
| I am frustrated by this rush hour traffic! | negativa |
| this is a dog | neutral |
| I am so happy today, its sunny! (Hoy estoy muy feliz, el día está soleado) | positiva |

## <a name="computer-vision-sample"></a>Ejemplo de Computer Vision
[Computer Vision](../../cognitive-services/computer-vision/index.yml) analiza las imágenes para identificar estructuras como caras, objetos y descripciones en lenguaje natural. En este ejemplo se etiqueta la imagen siguiente. Las etiquetas son descripciones de una sola palabra de las cosas que hay en la imagen, como objetos reconocibles, personas, escenarios y acciones.


![imagen](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/objects.jpg)

```python
# Create a dataframe with the image URL
df_images = spark.createDataFrame([
        ("https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/objects.jpg", )
    ], ["image", ])

# Run the Computer Vision service. Analyze Image extracts information from/about the images.
analysis = (AnalyzeImage()
    .setLocation("eastasia") # Set the location of your cognitive service
    .setSubscriptionKey(cognitive_service_key)
    .setVisualFeatures(["Categories","Color","Description","Faces","Objects","Tags"])
    .setOutputCol("analysis_results")
    .setImageUrlCol("image")
    .setErrorCol("error"))

# Show the results of what you wanted to pull out of the images.
display(analysis.transform(df_images).select("image", "analysis_results.description.tags"))
```
### <a name="expected-results"></a>Resultados esperados

|imagen | etiquetas|
|--|--|
| `https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/objects.jpg` | [skating, person, man, outdoor, riding, sport, skateboard, young, board, shirt, air, park, boy, side, jumping, ramp, trick, doing, flying] |

## <a name="bing-image-search-sample"></a>Ejemplo de Bing Image Search
[Bing Image Search](../../cognitive-services/bing-image-search/overview.md) busca en la Web para recuperar las imágenes relacionadas con una consulta en lenguaje natural del usuario. En este ejemplo, usamos una consulta de texto que busca imágenes con citas de un personaje célebre. Devuelve una lista de direcciones URL de las imágenes que contienen fotos relacionadas con la consulta.


```python
from pyspark.ml import PipelineModel

# Number of images Bing will return per query
imgsPerBatch = 2
# A list of offsets, used to page into the search results
offsets = [(i*imgsPerBatch,) for i in range(10)]
# Since web content is our data, we create a dataframe with options on that data: offsets
bingParameters = spark.createDataFrame(offsets, ["offset"])

# Run the Bing Image Search service with our text query
bingSearch = (BingImageSearch()
    .setSubscriptionKey(bingsearch_service_key)
    .setOffsetCol("offset")
    .setQuery("Martin Luther King Jr. quotes")
    .setCount(imgsPerBatch)
    .setOutputCol("images"))

# Transformer that extracts and flattens the richly structured output of Bing Image Search into a simple URL column
getUrls = BingImageSearch.getUrlTransformer("images", "url")
pipeline_bingsearch = PipelineModel(stages=[bingSearch, getUrls])

# Show the results of your search: image URLs
res_bingsearch = pipeline_bingsearch.transform(bingParameters)
display(res_bingsearch.dropDuplicates())
```

### <a name="expected-results"></a>Resultados esperados

|imagen | 
|--|
|`http://everydaypowerblog.com/wp-content/uploads/2014/01/Martin-Luther-King-Jr.-Quotes-16.jpg` |
|`http://www.scrolldroll.com/wp-content/uploads/2017/06/6-25.png` |
| `http://abettertodaymedia.com/wp-content/uploads/2017/01/86783bd7a92960aedd058c91a1d10253.jpg`|
| `https://weneedfun.com/wp-content/uploads/2016/05/martin-luther-king-jr-quotes-11.jpg` |
| `http://www.sofreshandsogreen.com/wp-content/uploads/2012/01/martin-luther-king-jr-quote-sofreshandsogreendotcom.jpg` |
| `https://cdn.quotesgram.com/img/72/57/1104209728-martin_luther_king_jr_quotes_16.jpg` |
| `http://comicbookandbeyond.com/wp-content/uploads/2019/05/Martin-Luther-King-Jr.-Quotes.jpg` |
| `https://exposingthepain.files.wordpress.com/2015/01/martin-luther-king-jr-quotes-08.png` |
| `https://topmemes.me/wp-content/uploads/2020/01/Top-10-Martin-Luther-King-jr.-Quotes2-1024x538.jpg` |
| `http://img.picturequotes.com/2/581/580286/dr-martin-luther-king-jr-quote-1-picture-quote-1.jpg` |
| `http://parryz.com/wp-content/uploads/2017/06/Amazing-Martin-Luther-King-Jr-Quotes.jpg` |
| `http://everydaypowerblog.com/wp-content/uploads/2014/01/Martin-Luther-King-Jr.-Quotes1.jpg` |
| `https://lessonslearnedinlife.net/wp-content/uploads/2020/05/Martin-Luther-King-Jr.-Quotes-2020.jpg` |
| `https://quotesblog.net/wp-content/uploads/2015/10/Martin-Luther-King-Jr-Quotes-Wallpaper.jpg` |

## <a name="anomaly-detector-sample"></a>Ejemplo de Anomaly Detector

[Anomaly Detector](../../cognitive-services/anomaly-detector/index.yml) es ideal para detectar irregularidades en los datos de series temporales. En este ejemplo, usamos el servicio para buscar anomalías en toda la serie temporal.

```python
from pyspark.sql.functions import lit

# Create a dataframe with the point data that Anomaly Detector requires
df_timeseriesdata = spark.createDataFrame([
    ("1972-01-01T00:00:00Z", 826.0),
    ("1972-02-01T00:00:00Z", 799.0),
    ("1972-03-01T00:00:00Z", 890.0),
    ("1972-04-01T00:00:00Z", 900.0),
    ("1972-05-01T00:00:00Z", 766.0),
    ("1972-06-01T00:00:00Z", 805.0),
    ("1972-07-01T00:00:00Z", 821.0),
    ("1972-08-01T00:00:00Z", 20000.0), # anomaly
    ("1972-09-01T00:00:00Z", 883.0),
    ("1972-10-01T00:00:00Z", 898.0),
    ("1972-11-01T00:00:00Z", 957.0),
    ("1972-12-01T00:00:00Z", 924.0),
    ("1973-01-01T00:00:00Z", 881.0),
    ("1973-02-01T00:00:00Z", 837.0),
    ("1973-03-01T00:00:00Z", 9000.0) # anomaly
], ["timestamp", "value"]).withColumn("group", lit("series1"))

# Run the Anomaly Detector service to look for irregular data
anamoly_detector = (SimpleDetectAnomalies()
  .setSubscriptionKey(anomalydetector_key)
  .setLocation("eastasia")
  .setTimestampCol("timestamp")
  .setValueCol("value")
  .setOutputCol("anomalies")
  .setGroupbyCol("group")
  .setGranularity("monthly"))

# Show the full results of the analysis with the anomalies marked as "True"
display(anamoly_detector.transform(df_timeseriesdata).select("timestamp", "value", "anomalies.isAnomaly"))

```

### <a name="expected-results"></a>Resultados esperados

|timestamp | value | isAnomaly |
|--|--|--|
| 1972-01-01T00:00:00Z|826.0|false|
|1972-02-01T00:00:00Z|799.0|false|
|1972-03-01T00:00:00Z|890.0|false|
|1972-04-01T00:00:00Z|900.0|false|
|1972-05-01T00:00:00Z|766.0|false|
|1972-06-01T00:00:00Z|805.0|false|
|1972-07-01T00:00:00Z|821.0|false|
|1972-08-01T00:00:00Z|20000.0|true|
|1972-09-01T00:00:00Z|883.0|false|
|1972-10-01T00:00:00Z|898.0|false|
|1972-11-01T00:00:00Z|957.0|false|
|1972-12-01T00:00:00Z|924.0|false|
|1973-01-01T00:00:00Z|881.0|false|
|1973-02-01T00:00:00Z|837.0|false|
|1973-03-01T00:00:00Z|9000.0|true|


## <a name="speech-to-text-sample"></a>Ejemplo de conversión de voz en texto

El servicio [Speech to Text](../../cognitive-services/speech-service/index-text-to-speech.yml) convierte secuencias o archivos de audio hablado en texto. En este ejemplo, se transcribe un archivo de audio a texto. 

```python
# Create a dataframe with our audio URLs, tied to the column called "url"
df = spark.createDataFrame([("https://mmlspark.blob.core.windows.net/datasets/Speech/audio2.wav",)
                           ], ["url"])

# Run the Speech-to-text service to translate the audio into text
speech_to_text = (SpeechToTextSDK()
    .setSubscriptionKey(service_key)
    .setLocation("northeurope") # Set the location of your cognitive service
    .setOutputCol("text")
    .setAudioDataCol("url")
    .setLanguage("en-US")
    .setProfanity("Masked"))

# Show the results of the translation
display(speech_to_text.transform(df).select("url", "text.DisplayText"))
```
### <a name="expected-results"></a>Resultados esperados

|url | Texto para mostrar |
|--|--|
| `https://mmlspark.blob.core.windows.net/datasets/Speech/audio2.wav` | Custom Speech proporciona herramientas que permiten inspeccionar visualmente la calidad del reconocimiento de un modelo mediante la comparación de los datos de audio con el resultado de reconocimiento correspondiente del portal de Custom Speech. You can playback uploaded audio and determine if the provided recognition result is correct. This tool allows you to quickly inspect quality of Microsoft's baseline speech to text model or a trained custom model without having to transcribe any audio data.|

## <a name="clean-up-resources"></a>Limpieza de recursos
Para asegurarse de que se cierra la instancia de Spark, finalice todas las sesiones (cuadernos) conectadas. El grupo se cierra cuando se alcanza el **tiempo de inactividad** especificado en el grupo de Apache Spark. También puede seleccionar **finalizar la sesión** en la barra de estado en la parte superior derecha del cuaderno.

![screenshot-showing-stop-session](./media/tutorial-build-applications-use-mmlspark/stop-session.png)

## <a name="next-steps"></a>Pasos siguientes

* [Consulte los cuadernos de ejemplo de Synapse](https://github.com/Azure-Samples/Synapse/tree/main/MachineLearning) 
* [Repositorio de GitHub para MMLSpark](https://github.com/Azure/mmlspark)
