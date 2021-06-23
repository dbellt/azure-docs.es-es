---
title: Identificación y transcripción automáticas del contenido en varios idiomas con Azure Video Analyzer for Media (anteriormente, Video Indexer)
titleSuffix: Azure Media Services
description: En este tema se muestra cómo identificar y transcribir automáticamente el contenido en varios idiomas con Azure Video Analyzer for Media (anteriormente, Video Indexer).
services: media-services
author: Juliako
manager: femila
ms.topic: article
ms.date: 09/01/2019
ms.author: juliako
ms.openlocfilehash: 38e6c809d2201e56ae3cae9a7ca44daca1db2077
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388429"
---
# <a name="automatically-identify-and-transcribe-multi-language-content"></a>Identificación y transcripción automática del contenido de varios idiomas

Azure Video Analyzer for Media (anteriormente, Video Indexer) admite la identificación y la transcripción automática del contenido en varios idiomas. Este proceso implica la identificación automática del idioma que se habla en diferentes segmentos del audio, el envío de cada segmento del archivo multimedia que se va a transcribir y la combinación de la transcripción para tener una transcripción unificada. 

## <a name="choosing-multilingual-identification-on-indexing-with-portal"></a>Elección de la identificación multilingüe en la indexación con el portal

Puede elegir la **detección en varios idiomas** al cargar y indexar el vídeo. También puede elegir la **detección en varios idiomas** al volver a indexar el vídeo. Los pasos siguientes describen cómo volver a indexar:

1. Vaya al sitio web de [Video Analyzer for Media](https://vi.microsoft.com/) e inicie sesión.
1. Vaya a la página de la **biblioteca** y mantenga el puntero sobre el nombre del vídeo que desea volver a indexar. 
1. En la esquina inferior derecha, haga clic en el botón **Volver a indexar el vídeo**. 
1. En el cuadro de diálogo **Volver a indexar el vídeo**, elija la **detección en varios idiomas** en el cuadro desplegable **Idioma de origen del vídeo**.

    * Cuando un vídeo se indexa como multilingüe, la página de información incluirá esa opción, y aparecerá un tipo de información adicional que permite al usuario ver qué segmento se ha transcrito en qué "idioma hablado".
    * La traducción a todos los idiomas está totalmente disponible desde la transcripción de varios idiomas.
    * El resto de información se mostrará en el idioma principal detectado; es decir, en el idioma que aparecía más en el audio.
    * Los subtítulos del reproductor también están disponibles en varios idiomas.

![Experiencia del portal](./media/multi-language-identification-transcription/portal-experience.png)

## <a name="choosing-multilingual-identification-on-indexing-with-api"></a>Elección de la identificación multilingüe en la indexación con la API

Al indexar o [volver a indexar](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Re-Index-Video) un vídeo mediante la API, elija la opción `multi-language detection` en el parámetro `sourceLanguage`.

### <a name="model-output"></a>Salida del modelo

El modelo recuperará todos los idiomas detectados en el vídeo en una lista.

```json
"sourceLanguage": null,
"sourceLanguages": [
    "es-ES",
    "en-US"
],
```

Además, cada instancia de la sección transcripción incluirá el idioma en el que se ha transcrito.

```json
{
  "id": 136,
  "text": "I remember well when my youth Minister took me to hear Doctor King I was a teenager.",
  "confidence": 0.9343,
  "speakerId": 1,
  "language": "en-US",
  "instances": [
    {
       "adjustedStart": "0:21:10.42",
       "adjustedEnd": "0:21:17.48",
       "start": "0:21:10.42",
       "end": "0:21:17.48"
    }
  ]
},
```

## <a name="guidelines-and-limitations"></a>Pautas y limitaciones

* Conjunto de idiomas admitidos: inglés, francés, alemán, español.
* Existe compatibilidad con contenido en varios idiomas, hasta un máximo de tres.
* Si el audio contiene idiomas distintos de los que se incluyen en la lista anterior, el resultado es inesperado.
* Longitud mínima de segmento para detectar para cada idioma: 15 segundos.
* El retardo para la detección de idioma es de 3 segundos de media.
* Se espera que el discurso sea continuo. Las alternancias frecuentes entre los idiomas puede afectar al rendimiento de los modelos.
* El discurso de los hablantes no nativos puede afectar al rendimiento del modelo (por ejemplo, cuando los hablantes usan su lengua nativa y cambian a otro idioma).
* El modelo está diseñado para reconocer una voz conversacional espontánea con una acústica razonable (no comandos de voz, canciones, etc.).
* La creación y edición de proyectos no está disponible actualmente para vídeos en varios idiomas.
* Los modelos de lenguaje personalizados no están disponibles cuando se usa la detección en varios idiomas.
* No se admite la adición de palabras clave.
* Al exportar archivos de subtítulos, la indicación de idioma no aparecerá.
* La API de transcripción de actualización no admite archivos en varios idiomas.

## <a name="next-steps"></a>Pasos siguientes

[Información general de Video Analyzer for Media](video-indexer-overview.md)
