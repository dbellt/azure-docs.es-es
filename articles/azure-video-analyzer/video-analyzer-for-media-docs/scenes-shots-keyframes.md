---
title: Escenas, capturas y fotogramas clave de Azure Video Analyzer for Media (anteriormente, Video Indexer)
titleSuffix: Azure Media Services
description: En este tema encontrará información general sobre las escenas, capturas y fotogramas clave de Azure Video Analyzer for Media (anteriormente, Video Indexer).
services: media-services
author: Juliako
manager: femila
ms.topic: article
ms.date: 07/05/2019
ms.author: juliako
ms.openlocfilehash: a3408ea644e5a2ac8ccc9bde808c232a705fd57e
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386918"
---
# <a name="scenes-shots-and-keyframes"></a>Escenas, capturas y fotogramas clave

Azure Video Analyzer for Media (anteriormente, Video Indexer) admite la segmentación de vídeos en unidades temporales basadas en propiedades estructurales y semánticas. Esta funcionalidad permite a los clientes examinar, administrar y editar fácilmente su contenido de vídeo en función de distintas granularidades. Por ejemplo, según las escenas, los cortes y los fotogramas clave, como se describe en este tema.   

![Escenas, capturas y fotogramas clave](./media/scenes-shots-keyframes/scenes-shots-keyframes.png)
 
## <a name="scene-detection"></a>Detección de escenas  
 
Video Analyzer for Media determina cuándo cambia una escena en el vídeo según indicaciones visuales. Una escena representa un evento único, y se compone de una serie de cortes consecutivos que están semánticamente relacionadas. Una vista en miniatura de la escena es el primer fotograma clave de su captura subyacente. Video Analyzer for Media segmenta un vídeo en escenas según la coherencia de color entre cortes consecutivos, y recupera la hora de comienzo y final de cada escena. La detección de escenas se considera una tarea complicada ya que supone cuantificar los aspectos semánticas de los vídeos.

> [!NOTE]
> Se aplica a los vídeos que contienen al menos tres escenas.

## <a name="shot-detection"></a>Detección de cortes

Video Analyzer for Media determina cuándo cambia un corte en el vídeo según indicaciones visuales, y realiza el seguimiento de las transacciones abruptas y graduales en el esquema de color de los fotogramas adyacentes. Los metadatos del corte incluyen una hora de inicio y de finalización, así como la lista de fotogramas clave incluidos en ese corte. Los cortes son fotogramas consecutivos que se toman con la misma cámara al mismo tiempo.

## <a name="keyframe-detection"></a>Detección de fotogramas clave

Video Analyzer for Media selecciona los fotogramas que mejor representan cada corte. Los fotogramas clave son los fotogramas representativos seleccionados de todo el vídeo según propiedades estéticas (por ejemplo, el contraste y la estabilidad). Video Analyzer for Media recupera una lista de identificadores de fotograma clave como parte de los metadatos del corte, en función de los cuales los clientes pueden extraer el fotograma clave como una imagen en alta resolución.  

### <a name="extracting-keyframes"></a>Extracción de fotogramas clave

Para extraer fotogramas clave de alta resolución para el vídeo, primero debe cargar e indexar el vídeo.

![Fotogramas clave](./media/scenes-shots-keyframes/extracting-keyframes.png)

#### <a name="with-the-video-analyzer-for-media-website"></a>Con el sitio web de Video Analyzer for Media

Para extraer fotogramas clave mediante el sitio web de Video Analyzer for Media, cargue e indexe el vídeo. Una vez completado el trabajo de indexación, haga clic en el botón **Descargar** y seleccione **Artefactos (ZIP)** . Se descargará la carpeta de artefactos en el equipo. 

![Captura de pantalla que muestra el menú desplegable "Descargar" con la opción "Artefactos" seleccionada.](./media/scenes-shots-keyframes/extracting-keyframes2.png)
 
Descomprima y abra la carpeta. Vaya a la carpeta *_KeyframeThumbnail* y encontrará todos los fotogramas clave que se extrajeron del vídeo. 

#### <a name="with-the-video-analyzer-for-media-api"></a>Con la API de Video Analyzer for Media

Para obtener fotogramas clave mediante la API de Video Indexer, cargue e indexe el vídeo con la llamada a [Upload Video](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Upload-Video) (Cargar vídeo). Una vez completado el trabajo de indexación, llame a [Get Video Index](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Get-Video-Index) (Obtener índice del vídeo). Esto le proporcionará toda la información que Video Indexer extrae del contenido en un archivo JSON.  

Obtendrá una lista de identificadores de fotogramas clave como parte de los metadatos de cada captura. 

```json
"shots":[  
    {  
      "id":0,
      "keyFrames":[  
          {  
            "id":0,
            "instances":[  
                {  
                  "thumbnailId":"00000000-0000-0000-0000-000000000000",
                  "start":"0:00:00.209",
                  "end":"0:00:00.251",
                  "duration":"0:00:00.042"
                }
            ]
          },
          {  
            "id":1,
            "instances":[  
                {  
                  "thumbnailId":"00000000-0000-0000-0000-000000000000",
                  "start":"0:00:04.755",
                  "end":"0:00:04.797",
                  "duration":"0:00:00.042"
                }
            ]
          }
      ],
      "instances":[  
          {  
            "start":"0:00:00",
            "end":"0:00:06.34",
            "duration":"0:00:06.34"
          }
      ]
    },

]
```

Ahora tendrá que ejecutar cada uno de estos identificadores de fotogramas clave en la llamada a [Get Thumbnails](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Get-Video-Thumbnail) (Obtener miniaturas). Se descargará cada una de las imágenes de fotogramas clave en el equipo. 

## <a name="editorial-shot-type-detection"></a>Detección del tipo de toma editorial

Los fotogramas clave están asociados con cortes del archivo JSON de salida. 

El tipo de toma asociado a una toma individual en el JSON de las conclusiones representa su tipo editorial. Las características de estos tipos de toma son útiles al editar vídeos en clips, avances o al buscar un estilo concreto de fotograma clave para fines artísticos. Los diferentes tipos se determinan en función del análisis del primer fotograma clave de cada toma. Las capturas se identifican por la escala, el tamaño y la ubicación de las caras que aparecen en su primer fotograma clave. 

El tamaño y la escala de la toma se determinan en función de la distancia entre la cámara y las caras que aparecen en el fotograma. Con estas propiedades, Video Analyzer for Media detecta los siguientes tipos de corte:

* Plano general: muestra todo el cuerpo de una persona.
* Plano medio: muestra la cara y la parte superior del cuerpo de una persona.
* Primer plano: muestra principalmente la cara de una persona.
* Primerísimo primer plano: muestra la cara de una persona que rellena la pantalla. 

Los tipos de capturas también pueden determinarse por la ubicación de los personajes del sujeto con respecto al centro del fotograma. Esta propiedad define los siguientes tipos de corte de Video Analyzer for Media:

* Cara en la izquierda: una persona aparece en el lado izquierdo del fotograma.
* Cara en el centro: una persona aparece en el área central del fotograma.
* Cara en la derecha: una persona aparece en el lado derecho del fotograma.
* Exterior: una persona aparece en un entorno exterior.
* Interior: una persona aparece en un entorno interior.

Características adicionales:

* Dos tomas: muestra las caras de dos personas en tamaño medio.
* Varias caras: más de dos personas.


## <a name="next-steps"></a>Pasos siguientes

[Examen de la salida de Video Analyzer for Media producida por la API](video-indexer-output-json-v2.md#scenes)
