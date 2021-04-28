---
title: Consulta del punto de conexión del contenedor Conversión de texto a voz
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 08/31/2020
ms.author: aahi
ms.openlocfilehash: 4571acfc46d9b336be23eaa3287dce5696c707b2
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107930240"
---
El contenedor proporciona [API de punto de conexión basadas en REST](../rest-text-to-speech.md). Hay muchos [proyectos de código fuente de ejemplo](https://github.com/Azure-Samples/Cognitive-Speech-TTS) disponibles para las variaciones de lenguaje, marco y plataforma.

Con los contenedores de texto a voz estándar y texto a voz neuronal, debe utilizar como base la configuración regional y la voz de la etiqueta de imagen que descargó. Por ejemplo, si descargó la etiqueta `latest`, la configuración regional predeterminada es `en-US` y la voz `AriaNeural`. El argumento `{VOICE_NAME}` sería [`en-US-AriaNeural`](../language-support.md#neural-voices). Vea el SSML de ejemplo siguiente:

```xml
<speak version="1.0" xml:lang="en-US">
    <voice name="en-US-AriaNeural">
        This text will get converted into synthesized speech.
    </voice>
</speak>
```
