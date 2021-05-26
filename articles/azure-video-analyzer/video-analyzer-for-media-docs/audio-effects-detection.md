---
title: Detección de efectos de audio
titleSuffix: Azure Video Analyzer
description: La detección de efectos de audio es una de las funcionalidades de la inteligencia artificial de Azure Video Analyzer for Media. Puede detectar varios eventos acústicos y clasificarlos en categorías acústicas diferentes (por ejemplo, disparos, gritos y reacciones de la multitud, entre otros).
author: Juliako
manager: femila
ms.service: azure-video-analyzer
ms.topic: article
ms.date: 05/12/2021
ms.author: juliako
ms.openlocfilehash: 79bcde166be7dac53bc7c7d9d8e3ed5e0fc2f91a
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388691"
---
#  <a name="audio-effects-detection-preview"></a>Detección de efectos de audio (versión preliminar)

La **detección de efectos de audio** es una de las funcionalidades de la inteligencia artificial de Azure Video Analyzer for Media. Puede detectar varios eventos acústicos y clasificarlos en categorías acústicas diferentes (por ejemplo, disparos, gritos y reacción de una multitud, entre otros).
 
La detección de eventos de audio se puede usar en muchos dominios. Dos ejemplos son:

* El uso de la detección de efectos de audio en el dominio de **Justicia y seguridad pública**. La detección de efectos de audio puede detectar y clasificar disparos, explosiones y vidrios rotos. Por lo tanto, se puede implementar en un sistema de ciudad inteligente o en otros entornos públicos que incluyan cámaras y micrófonos. Ofrece una detección rápida y precisa de incidentes violentos. 
* En el dominio **Medios y entretenimiento**, las empresas que cuentan con una gran cantidad de archivos de vídeo pueden mejorar fácilmente sus escenarios de accesibilidad al agregar efectos no verbales a sus transcripciones de vídeo a fin de brindar más contexto a las personas con problemas de audición.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/audio-effects-detection/audio-effects.jpg" alt-text="Imagen de efectos de audio":::
<br/>*Ejemplo de la salida de la detección de efectos de audio de Video Analyzer for Media*

## <a name="supported-audio-categories"></a>Categorías de audio admitidas  

La **detección de efectos de audio** puede detectar y clasificar 8 categorías distintas. En la tabla siguiente, encontrará estas categorías divididas en los distintos valores preestablecidos de VI, que se dividen en **Estándar** y **Avanzada**. Para más información, consulte los [precios](https://azure.microsoft.com/pricing/details/media-services/).

|Tipo de indexación |Indexación estándar| Indexación avanzada|
|---|---|---|
|**Nombre del valor preestablecido** |**"Solo audio”** <br/>**"Vídeo + audio"** |**"Audio avanzado"**<br/> **"Vídeo + audio avanzados"**|
|**Aparecer en el panel de información**|| V|
|Reacción de una multitud |V| V|
| Silencio| V| V|
| Disparo ||V |
| Vidrio roto ||V|
| Sonido de alarma|| V |
| Sirena|| V |
| Risa|| V |
| Ladridos|| V|

## <a name="result-formats"></a>Formatos de resultado

Los efectos de audio se recuperan en el archivo JSON de información que incluye el identificador de categoría, el tipo, el nombre y el conjunto de instancias por categoría, junto con su período específico y la puntuación de confianza.

El parámetro `name` se presentará en el idioma en que se indexó el archivo JSON, mientras que el tipo no se modificará nunca.

```json
audioEffects: [{
        id: 0,
        type: "Gunshot",
        name: "Gunshot",
        instances: [{
                confidence: 0.649,
                adjustedStart: "0:00:13.9",
                adjustedEnd: "0:00:14.7",
                start: "0:00:13.9",
                end: "0:00:14.7"
            }, {
                confidence: 0.7706,
                adjustedStart: "0:01:54.3",
                adjustedEnd: "0:01:55",
                start: "0:01:54.3",
                end: "0:01:55"
            }
        ]
    }, {
        id: 1,
        type: "CrowdReactions",
        name: "Crowd Reactions",
        instances: [{
                confidence: 0.6816,
                adjustedStart: "0:00:47.9",
                adjustedEnd: "0:00:52.5",
                start: "0:00:47.9",
                end: "0:00:52.5"
            },
            {
                confidence: 0.7314,
                adjustedStart: "0:04:57.67",
                adjustedEnd: "0:05:01.57",
                start: "0:04:57.67",
                end: "0:05:01.57"
            }
        ]
    }
],
```

## <a name="how-to-index-audio-effects"></a>Indexación de los efectos de audio

A fin de establecer que el proceso de indexación incluya la detección de los efectos de audio, el usuario debe elegir uno de los valores preestablecidos avanzados en el menú "Indexación de audio y vídeo", tal como se muestra a continuación.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/audio-effects-detection/index-audio-effect.png" alt-text="Imagen de indexación de efectos de audio":::

## <a name="closed-caption"></a>Subtítulos

Cuando los efectos de audio se recuperan en los archivos de subtítulos, se recuperarán entre corchetes según la estructura siguiente:

|Tipo| Ejemplo|
|---|---|
|SRT |00:00:00,000  00:00:03,671<br/>[Disparo]|
|VTT |00:00:00.000  00:00:03.671<br/>[Disparo]|
|TTML|Confianza: 0.9047 <br/> <p begin="00:00:00.000" end="00:00:03.671">[Disparo]</p>|
|TXT |[Disparo]|
|CSV |0.9047,00:00:00.000,00:00:03.671, [Disparo]|

Los efectos de audio del archivo de subtítulos se recuperarán con la ayuda de la lógica siguiente:

* El tipo de evento `Silence` no se agregará a los subtítulos
* La duración máxima para mostrar un evento es de 5 segundos
* La duración mínima del temporizador para mostrar un evento es de 700 milisegundos

## <a name="adding-audio-effects-in-closed-caption-files"></a>Incorporación de efectos de audio en archivos de subtítulos

Los efectos de audio se pueden agregar a los archivos de subtítulos compatibles con Azure Video Analyzer mediante la [API Obtener subtítulos](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Get-Video-Captions) al elegir True en el parámetro `includeAudioEffects` o mediante la experiencia del portal video.ai si se selecciona **Download** -> **Closed Captions** -> **Include Audio Effects** (Descargar > Subtítulos > Incluir efectos de audio).

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/audio-effects-detection/close-caption.jpg" alt-text="Efectos de audio en CC":::

> [!NOTE]
> Al usar la [transcripción de actualizaciones](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Update-Video-Transcript) o [actualizar el modelo de lenguaje personalizado](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Update-Language-Model) de los archivos de subtítulos, se omitirán los efectos de audio incluidos en esos archivos.

## <a name="limitations-and-assumptions"></a>Suposiciones y limitaciones

* El modelo solo funciona en segmentos no verbales.
* Actualmente, el modelo funciona para una sola categoría a la vez. Por ejemplo, no admite llantos y conversaciones de fondo, ni disparos con explosiones.
* Actualmente, el modelo no admite casos en los que suena música fuerte de fondo.
* La duración mínima del segmento es de 2 segundos.

## <a name="next-steps"></a>Pasos siguientes

Revise la [información general](video-indexer-overview.md).
