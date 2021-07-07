---
title: 'Inicio rápido de la conversión de texto a voz: servicio de voz'
titleSuffix: Azure Cognitive Services
description: Aprenda a usar el SDK de voz para convertir texto a voz. En este inicio rápido, obtendrá información sobre la construcción de objetos y los patrones de diseño, los formatos de salida de audio admitidos, la CLI de voz y las opciones de configuración personalizadas de la síntesis de voz.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: quickstart
ms.date: 05/17/2021
ms.author: trbye
ms.custom: devx-track-python, devx-track-js, devx-track-csharp, cog-serv-seo-aug-2020
zone_pivot_groups: programming-languages-set-twenty-four
keywords: texto a voz
ms.openlocfilehash: 33e19816d219e2be3dae367c5b30da349e2eb879
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110085498"
---
# <a name="get-started-with-text-to-speech"></a>Introducción a la conversión de texto a voz

::: zone pivot="programming-language-csharp"
[!INCLUDE [C# Basics include](includes/how-to/text-to-speech-basics/text-to-speech-basics-csharp.md)]
::: zone-end

::: zone pivot="programming-language-cpp"
[!INCLUDE [C++ Basics include](includes/how-to/text-to-speech-basics/text-to-speech-basics-cpp.md)]
::: zone-end

::: zone pivot="programming-language-go"
[!INCLUDE [C++ Basics include](includes/how-to/text-to-speech-basics/text-to-speech-basics-go.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Java Basics include](includes/how-to/text-to-speech-basics/text-to-speech-basics-java.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [JavaScript Basics include](includes/how-to/text-to-speech-basics/text-to-speech-basics-javascript.md)]
::: zone-end

::: zone pivot="programming-languages-objectivec-swift"
[!INCLUDE [Objective-C and Swift Basics include](includes/how-to/text-to-speech-basics/text-to-speech-basics-objectivec-swift.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Python Basics include](includes/how-to/text-to-speech-basics/text-to-speech-basics-python.md)]
::: zone-end

::: zone pivot="programming-language-curl"
[!INCLUDE [REST include](includes/how-to/text-to-speech-basics/text-to-speech-basics-curl.md)]
::: zone-end

::: zone pivot="programmer-tool-spx"
[!INCLUDE [CLI Basics include](includes/how-to/text-to-speech-basics/text-to-speech-basics-cli.md)]
::: zone-end

## <a name="get-position-information"></a>Obtención de información de posición

Es posible que el proyecto necesite saber cuándo se pronuncia una palabra mediante la conversión de texto a voz para que pueda llevar a cabo acciones específicas en función de ese tiempo.
Por ejemplo, si desea resaltar palabras a medida que se pronuncian, debe saber qué resaltar, cuándo hacerlo y durante cuánto tiempo se debe resaltar.

Para ello, puede usar el evento `WordBoundary` disponible en `SpeechSynthesizer`.
Este evento se desencadena al principio de cada nuevo texto oral y proporcionará un desfase de tiempo dentro del flujo hablado, así como un desplazamiento de texto dentro del indicador de entrada.

* `AudioOffset` informa del tiempo transcurrido del audio de salida entre el inicio de la síntesis y el inicio de la siguiente palabra. Se mide en unidades de centenares de nanosegundos (SNP)y 10 000 SNP equivalen a 1 milisegundo.
* `WordOffset` notifica la posición de un carácter en la cadena de entrada (texto original o [SSML](speech-synthesis-markup.md)) inmediatamente antes de la palabra que se va a pronunciar.

> [!NOTE]
> Los eventos `WordBoundary` se generan cuando los datos de audio de salida están disponibles, lo que será más rápido que la reproducción en un dispositivo de salida. El autor de la llamada debe llevar a cabo una sincronización correcta del tiempo de la transmisión con el "tiempo real".

En GitHub, se pueden encontrar ejemplos de uso de `WordBoundary` en los [ejemplos de conversión de texto a voz](https://aka.ms/csspeech/samples).

## <a name="next-steps"></a>Pasos siguientes

* [Introducción a Voz personalizada](how-to-custom-voice.md)
* [Mejora de síntesis con SSML](speech-synthesis-markup.md)
* Aprenda a usar [Long Audio API](long-audio-api.md) para ejemplos con mucho texto, como libros y artículos de prensa
* Consulte los [ejemplos del inicio rápido](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart) en GitHub
