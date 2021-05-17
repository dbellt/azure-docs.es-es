---
title: Cómo obtener eventos de postura facial para la sincronización de labios
titleSuffix: Azure Cognitive Services
description: El SDK de Voz admite eventos de visema durante la síntesis de voz, que se usa para representar los planteamientos clave en la voz observada, como la posición de los labios, mandíbula y lengua al generar un fonema determinado.
services: cognitive-services
author: yulin-li
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/03/2021
ms.author: yulili
ms.custom: references_regions
zone_pivot_groups: programming-languages-speech-services-nomore-variant
ms.openlocfilehash: 630e05b50f153b816f74a9df1f5783c0e36cb53e
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109734637"
---
# <a name="get-facial-pose-events"></a>Obtención de eventos de postura facial

> [!NOTE]
> Los eventos de visema solo están disponibles para `en-US`voces neuronales[ en ](language-support.md#text-to-speech) inglés (Estados Unidos) por ahora.

Un _visema_ es la descripción visual de un fonema en lenguaje hablado.
Define la posición de la esfera y la boca al hablar de una palabra.
Cada visema describe las principales supuestos faciales para un conjunto específico de fonemas.
No hay una correspondencia uno a uno entre visemas y fonemas.
A menudo, varios fonemas se corresponden con un solo visema, ya que varios fonemas tienen el mismo aspecto en la superficie cuando se producen, como `s` y `z`.
Vea la [tabla de asignación entre visemas y fonemas](#map-phonemes-to-visemes).

Mediante las visemas, puede crear asistentes de difusión de noticias más naturales e inteligentes, juegos interactivos y personajes animados, así como vídeos de aprendizaje de lenguaje más intuitivos. Las personas con deficiencias auditivas también pueden captar los sonidos de forma visual y "leer los labios" del contenido de voz que muestra los visemas en una cara animada.

## <a name="get-viseme-events-with-the-speech-sdk"></a>Obtención de eventos de visema con el SDK de voz

Para realizar eventos de visema, convierta el texto de entrada en un conjunto de secuencias de fonema y sus secuencias visema correspondientes. Estimamos la hora de inicio de cada visema en el audio de voz. Los eventos de visema contienen una secuencia de identificadores de visema, cada uno con un desplazamiento en el audio donde aparece visema. Estos eventos pueden impulsar animaciones de boca que simulan a una persona que habla el texto de entrada.

| Parámetro | Descripción |
|-----------|-------------|
| Id. de visema | Número entero que especifica un visema. En inglés (Estados Unidos), ofrecemos 22 visemas diferentes para describir las formas de la boca para un conjunto específico de fonemas. Consulte la [tabla de asignación entre visemas y fonemas](#map-phonemes-to-visemes).  |
| Desplazamiento de audio | La hora de inicio de cada visema, en pasos (100 nanosegundos). |

Para obtener los eventos visema, suscríbase al evento `VisemeReceived` en SDK de voz.
En los fragmentos de código siguientes se muestra cómo suscribirse al evento de visema.

::: zone pivot="programming-language-csharp"

```csharp
using (var synthesizer = new SpeechSynthesizer(speechConfig, audioConfig))
{
    // Subscribes to viseme received event
    synthesizer.VisemeReceived += (s, e) =>
    {
        Console.WriteLine($"Viseme event received. Audio offset: " +
            $"{e.AudioOffset / 10000}ms, viseme id: {e.VisemeId}.");
    };

    var result = await synthesizer.SpeakSsmlAsync(ssml));
}

```

::: zone-end

::: zone pivot="programming-language-cpp"

```cpp
auto synthesizer = SpeechSynthesizer::FromConfig(speechConfig, audioConfig);

// Subscribes to viseme received event
synthesizer->VisemeReceived += [](const SpeechSynthesisVisemeEventArgs& e)
{
    cout << "viseme event received. "
        // The unit of e.AudioOffset is tick (1 tick = 100 nanoseconds), divide by 10,000 to convert to milliseconds.
        << "Audio offset: " << e.AudioOffset / 10000 << "ms, "
        << "viseme id: " << e.VisemeId << "." << endl;
};

auto result = synthesizer->SpeakSsmlAsync(ssml).get();
```

::: zone-end

::: zone pivot="programming-language-java"

```java
SpeechSynthesizer synthesizer = new SpeechSynthesizer(speechConfig, audioConfig);

// Subscribes to viseme received event
synthesizer.VisemeReceived.addEventListener((o, e) -> {
    // The unit of e.AudioOffset is tick (1 tick = 100 nanoseconds), divide by 10,000 to convert to milliseconds.
    System.out.print("Viseme event received. Audio offset: " + e.getAudioOffset() / 10000 + "ms, ");
    System.out.println("viseme id: " + e.getVisemeId() + ".");
});

SpeechSynthesisResult result = synthesizer.SpeakSsmlAsync(ssml).get();
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
speech_synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config, audio_config=audio_config)

# Subscribes to viseme received event
speech_synthesizer.viseme_received.connect(lambda evt: print(
    "Viseme event received: audio offset: {}ms, viseme id: {}.".format(evt.audio_offset / 10000, evt.viseme_id)))

result = speech_synthesizer.speak_ssml_async(ssml).get()
```

::: zone-end

::: zone pivot="programming-language-javascript"

```Javascript
var synthesizer = new SpeechSDK.SpeechSynthesizer(speechConfig, audioConfig);

// Subscribes to viseme received event
synthesizer.visemeReceived = function (s, e) {
    window.console.log("(Viseme), Audio offset: " + e.audioOffset / 10000 + "ms. Viseme ID: " + e.visemeId);
}

synthesizer.speakSsmlAsync(ssml);
```

::: zone-end

::: zone pivot="programming-language-objectivec"

```Objective-C
SPXSpeechSynthesizer *synthesizer =
    [[SPXSpeechSynthesizer alloc] initWithSpeechConfiguration:speechConfig
                                           audioConfiguration:audioConfig];

// Subscribes to viseme received event
[synthesizer addVisemeReceivedEventHandler: ^ (SPXSpeechSynthesizer *synthesizer, SPXSpeechSynthesisVisemeEventArgs *eventArgs) {
    NSLog(@"Viseme event received. Audio offset: %fms, viseme id: %lu.", eventArgs.audioOffset/10000., eventArgs.visemeId);
}];

[synthesizer speakSsml:ssml];
```

::: zone-end

## <a name="map-phonemes-to-visemes"></a>Asignación de fonemas a visemas

Las visemas varían según el lenguaje. Cada lenguaje tiene un conjunto de visemas que corresponden a sus fonemas específicos. En la tabla siguiente se muestra la correspondencia entre el alfabeto fonético internacional (IPA) fonemas y los identificadores de visema para inglés (Estados Unidos).

| IPA | Ejemplo | Id. de visema |
|-----|---------|-----------|
| i   | **ea** t   | 6 |
| ɪ   | **i** f | 6 |
| eɪ  | **a** te | 4 |
| ɛ | **e** very | 4 |
|æ  |   **a** ctive        |1|
|ɑ  |   **o** bstinate     |2|
|ɔ  |   c **au** se         |3|
|ʊ  |   b **oo** k          |4|
|oʊ |   **o** ld           |8|
|u  |   **U** ber          |7|
|ʌ  |   **u** ncle         |1|
|aɪ |   **i** ce           |11|
|aʊ |   **ou** t           |9|
|ɔɪ |   **oi** l           |10|
|ju |   **Yu** ma          |[6, 7]|
|ə  |   **a** go           |1|
|ɪɹ |   **ear** s          |[6, 13]|
|ɛɹ |   **air** plane      |[4, 13]|
|ʊɹ |   c **ur** e          |[4, 13]|
|aɪ(ə)ɹ |   **Ire** land   |[11, 13]|
|aʊ(ə)ɹ |   **hour** s     |[9, 13]|
|ɔɹ |   **or** ange        |[3, 13]|
|ɑɹ |   **ar** tist        |[2, 13]|
|ɝ  |   **ear** th         |[5, 13]|
|ɚ  |   all **er** gy       |[1, 13]|
|w  |   **w** ith, s **ue** de   |7|
|j  |   **y** ard, f **e** w     |6|
|p  |   **p** ut           |21|
|b  |   **b** ig           |21|
|t  |   **t** alk          |19|
|d  |   **d** ig           |19|
|k  |   **c** ut           |20|
|e  |   **g** o            |20|
|m  |   **m** at, s **m** ash    |21|
|n  |   **n** o, s **n** ow      |19|
|ŋ  |   li **n** k          |20|
|f  |   **f** ork          |18|
|v  |   **v** alue         |18|
|θ  |   **th** in          |17|
|ð  |   **th** en          |17|
|s  |   **s** it           |15|
|z  |   **z** ap           |15|
|ʃ  |   **sh** e           |16|
|ʒ  |   **J** acques       |16|
|h  |   **h** elp          |12|
|tʃ |   **ch** in          |16|
|dʒ |   **j** oy           |16|
|l  |   **l** id, g **l** ad     |14|
|ɹ  |   **r** ed, b **r** ing    |13|


## <a name="next-steps"></a>Pasos siguientes

* [Documentación de referencia del SDK de voz](speech-sdk.md)
