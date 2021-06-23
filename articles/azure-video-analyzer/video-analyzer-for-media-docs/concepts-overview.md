---
title: 'Conceptos de Azure Video Analyzer for Media (anteriormente, Video Indexer): Azure'
titleSuffix: Azure Video Analyzer for Media (formerly Video Indexer)
description: En este artículo se ofrece una breve introducción a la terminología y los conceptos de Azure Video Analyzer for Media (anteriormente, Video Indexer).
services: media-services
author: Juliako
manager: femila
ms.topic: article
ms.date: 01/19/2021
ms.author: juliako
ms.openlocfilehash: 90a896902f7dcb55f19bb9106750eb9b18a33809
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386698"
---
# <a name="video-analyzer-for-media-concepts"></a>Conceptos de Video Analyzer for Media

En este artículo se ofrece una breve introducción a la terminología y los conceptos de Azure Video Analyzer for Media (anteriormente, Video Indexer).

## <a name="audiovideocombined-insights"></a>Audio/vídeo/información combinada

Cuando se cargan vídeos en Video Analyzer for Media, este analiza los objetos visuales y el audio mediante la ejecución de diferentes modelos de IA. A medida que Video Analyzer for Media analiza el vídeo, los modelos de IA extraen la información. Para más información, consulte [Introducción](video-indexer-overview.md).

## <a name="confidence-scores"></a>Puntuación de confianza 

La puntuación de confianza indica la confianza en una información. Es un número entre 0,0 y 1,0. Cuanto mayor sea la puntuación, mayor será la confianza en la respuesta. Por ejemplo, 

```json
"transcript":[
{
  "id":1,
  "text":"Well, good morning everyone and welcome to",
  "confidence":0.8839,
  "speakerId":1,
  "language":"en-US",
  "instances":[
     {
    "adjustedStart":"0:00:10.21",
    "adjustedEnd":"0:00:12.81",
    "start":"0:00:10.21",
    "end":"0:00:12.81"
     }
  ]
},
```

## <a name="content-moderation"></a>Moderación de contenido

use modelos de moderación de contenido textual y visual para proteger a los usuarios del contenido inadecuado y asegúrese de que el contenido que publica coincide con los valores de la organización. Puede bloquear automáticamente determinados vídeos o avisar a los usuarios sobre el contenido. Para más información, consulte [Información detallada: moderación de contenido visual y textual](video-indexer-output-json-v2.md#visualcontentmoderation). 

## <a name="blocks"></a>Blocks   

Los bloques están pensados para que resulte más fácil el recorrido por los datos. Por ejemplo, el bloque puede dividirse en función de cuándo cambian los oradores o de si hay una pausa larga.  

## <a name="project-and-editor"></a>Proyecto y editor

El sitio web de [Video Analyzer for Media](https://www.videoindexer.ai/) le permite usar la información detallada de los vídeos para buscar el contenido multimedia adecuado, localizar las partes que le interesen y usar los resultados para crear un proyecto totalmente nuevo. Una vez creado, puede representar y descargar el proyecto desde Video Analyzer for Media y usarlo en sus propias aplicaciones de edición o en sus flujos de trabajo descendentes.

Esta característica puede resultarle útil para: 

* Crear destacados de películas en clips finales.
* Usar clips de vídeos antiguos en transmisiones de noticias.
* Crear contenido más breve para las redes sociales.

Para más información, consulte [Uso del editor para crear proyectos](use-editor-create-project.md).

## <a name="keyframes"></a>Fotogramas clave

Video Analyzer for Media selecciona los fotogramas que mejor representan cada corte. Los fotogramas clave son los fotogramas representativos seleccionados de todo el vídeo según propiedades estéticas (por ejemplo, el contraste y la estabilidad). Para más información, consulte [Scenes, shots, and keyframes](scenes-shots-keyframes.md) (Escenas, cortes y fotogramas clave).

## <a name="time-range-vs-adjusted-time-range"></a>intervalo de tiempo frente a intervalo de tiempo ajustado   

TimeRange es el intervalo de tiempo del vídeo original. Intervalo de tiempo relativo a la lista de reproducción actual. Como puede crear una lista de reproducción a partir de diferentes líneas de vídeos distintas, puede grabar un vídeo de una hora y utilizar solo una línea de él, por ejemplo, de 10:00 a 10:15. En ese caso, tendrá una lista de reproducción con 1 línea, donde el intervalo de tiempo es 10:00-10:15, pero adjustedTimeRange es 00:00-00:15. 

## <a name="widgets"></a>Widgets

Video Analyzer for Media admite la inserción de widgets en las aplicaciones. Para más información, consulte [Inserción de widgets de Video Analyzer for Media en las aplicaciones](video-indexer-embed-widgets.md).

## <a name="summarized-insights"></a>Información resumida  

La información resumida contiene una vista agregada de los datos: caras, temas y emociones. Por ejemplo, en lugar de pasar por cada uno de los miles de intervalos de tiempo y comprobar qué caras hay en ellos, la información resumida contiene todas las caras y, para cada una de ellas, los intervalos de tiempo en los que aparece y el porcentaje del tiempo en el que se muestra.  

## <a name="next-steps"></a>Pasos siguientes

- [información general](video-indexer-overview.md)
- [Insights](video-indexer-output-json-v2.md)
