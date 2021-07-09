---
title: 'Streaming de audio comprimido con códec mediante el SDK de voz: servicio de voz'
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo hacer secuencias de audio comprimido al servicio de voz con el SDK de voz. Disponible para C++, C#y Java para Linux, Java en Android y Objective-C en iOS.
services: cognitive-services
author: amitkumarshukla
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/30/2020
ms.author: amishu
ms.custom: devx-track-csharp
zone_pivot_groups: programming-languages-set-twenty-two
ms.openlocfilehash: 183679eb9c4eb2480736dfb4cec8a343c67d0ed7
ms.sourcegitcommit: 67cdbe905eb67e969d7d0e211d87bc174b9b8dc0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111854502"
---
# <a name="use-codec-compressed-audio-input"></a>Uso de entrada de audio comprimido con códec

Tanto el SDK de Voz como la CLI de Voz pueden aceptar formatos de audio comprimidos mediante GStreamer. GStreamer descomprime el audio antes de enviarlo mediante la conexión del servicio de Voz como PCM sin procesar.

Plataforma | Idiomas | Versión de GStreamer compatible
| :--- | ---: | :---:
Windows (excepto UWP)  | C++, C#, Java, Python | [1.18.3](https://gstreamer.freedesktop.org/data/pkg/windows/1.18.3/)
Linux  | C++, C#, Java, Python | [Distribuciones y arquitecturas de destino de Linux admitidas](~/articles/cognitive-services/speech-service/speech-sdk.md)
Android  | Java | [1.18.3](https://gstreamer.freedesktop.org/data/pkg/android/1.18.3/)

## <a name="speech-sdk-version-required-for-compressed-audio-input"></a>Versión del SDK de voz necesaria para la entrada de audio comprimido
* Se requiere el SDK de Voz versión 1.10.0 o posterior para RHEL 8 y CentOS 8.
* Se requiere el SDK de Voz versión 1.11.0 o posterior para Windows.
* SDK de Voz versión 1.16.0, o posterior, para la versión más reciente de GStreamer en Windows y Android.

[!INCLUDE [supported-audio-formats](includes/supported-audio-formats.md)]

## <a name="gstreamer-required-to-handle-compressed-audio"></a>GStreamer necesario para controlar el audio comprimido

::: zone pivot="programming-language-csharp"
[!INCLUDE [prerequisites](includes/how-to/compressed-audio-input/csharp/prerequisites.md)]
::: zone-end

::: zone pivot="programming-language-cpp"
[!INCLUDE [prerequisites](includes/how-to/compressed-audio-input/cpp/prerequisites.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [prerequisites](includes/how-to/compressed-audio-input/java/prerequisites.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [prerequisites](includes/how-to/compressed-audio-input/python/prerequisites.md)]
::: zone-end

## <a name="example-code-using-codec-compressed-audio-input"></a>Ejemplo de código que usa una entrada de audio comprimido con códec

::: zone pivot="programming-language-csharp"
[!INCLUDE [prerequisites](includes/how-to/compressed-audio-input/csharp/examples.md)]
::: zone-end

::: zone pivot="programming-language-cpp"
[!INCLUDE [prerequisites](includes/how-to/compressed-audio-input/cpp/examples.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [prerequisites](includes/how-to/compressed-audio-input/java/examples.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [prerequisites](includes/how-to/compressed-audio-input/python/examples.md)]
::: zone-end

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Aprenda a reconocer voz](./get-started-speech-to-text.md)
