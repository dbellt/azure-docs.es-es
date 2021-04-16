---
title: Uso del SDK de voz para evaluar la pronunciación
titleSuffix: Azure Cognitive Services
description: El SDK de voz admite la evaluación de la pronunciación, que evalúa la calidad de la pronunciación de la entrada de voz, con indicadores de precisión, fluidez, integridad, etc.
services: cognitive-services
author: yulin-li
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 01/12/2021
ms.author: yulili
ms.custom: references_regions
zone_pivot_groups: programming-languages-speech-services-nomore-variant
ms.openlocfilehash: 2d1b5e490b7c8212e6103e3d169c1b5491d01dde
ms.sourcegitcommit: d23602c57d797fb89a470288fcf94c63546b1314
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/01/2021
ms.locfileid: "106167437"
---
# <a name="pronunciation-assessment"></a>Evaluación de la pronunciación

La evaluación de la pronunciación evalúa la pronunciación de la voz y ofrece a los oradores información sobre la precisión y la fluidez del audio hablado.
Con la evaluación de la pronunciación, los estudiantes de idiomas pueden practicar, obtener comentarios instantáneos y mejorar su pronunciación para poder hablar y realizar presentaciones con confianza.
Los educadores pueden utilizar la funcionalidad para evaluar la pronunciación de varios hablantes en tiempo real.

En este artículo, aprenderá a configurar `PronunciationAssessmentConfig` y a recuperar `PronunciationAssessmentResult` mediante el SDK de voz.

> [!NOTE]
> Actualmente, la característica de evaluación de pronunciación admite el idioma `en-US`, que está disponible en todas las [regiones de conversión de voz en texto](regions.md#speech-to-text-text-to-speech-and-translation). La compatibilidad con los idiomas `en-GB` y `zh-CN` se encuentra en versión preliminar, que está disponible en las regiones `westus`, `eastasia` y `centralindia`.

## <a name="pronunciation-assessment-with-the-speech-sdk"></a>Evaluación de la pronunciación con el SDK de voz

En los ejemplos siguientes, creará un elemento `PronunciationAssessmentConfig` y le aplicará `SpeechRecognizer`.

En los fragmentos de código siguientes se muestra cómo usar la detección automática de idioma en las aplicaciones:

::: zone pivot="programming-language-csharp"

```csharp
var pronunciationAssessmentConfig = new PronunciationAssessmentConfig(
    "reference text", GradingSystem.HundredMark, Granularity.Phoneme);

using (var recognizer = new SpeechRecognizer(
    speechConfig,
    audioConfig))
{
    // apply the pronunciation assessment configuration to the speech recognizer
    pronunciationAssessmentConfig.ApplyTo(recognizer);
    var speechRecognitionResult = await recognizer.RecognizeOnceAsync();
    var pronunciationAssessmentResult =
        PronunciationAssessmentResult.FromResult(speechRecognitionResult);
    var pronunciationScore = pronunciationAssessmentResult.PronunciationScore;
}
```

::: zone-end

::: zone pivot="programming-language-cpp"

```cpp
auto pronunciationAssessmentConfig =
    PronunciationAssessmentConfig::Create("reference text",
        PronunciationAssessmentGradingSystem::HundredMark,
        PronunciationAssessmentGranularity::Phoneme);

auto recognizer = SpeechRecognizer::FromConfig(
    speechConfig,
    audioConfig);

// apply the pronunciation assessment configuration to the speech recognizer
pronunciationAssessmentConfig->ApplyTo(recognizer);
speechRecognitionResult = recognizer->RecognizeOnceAsync().get();
auto pronunciationAssessmentResult =
    PronunciationAssessmentResult::FromResult(speechRecognitionResult);
auto pronunciationScore = pronunciationAssessmentResult->PronunciationScore;
```

::: zone-end

::: zone pivot="programming-language-java"

```java
PronunciationAssessmentConfig pronunciationAssessmentConfig =
    new PronunciationAssessmentConfig("reference text", 
        PronunciationAssessmentGradingSystem.HundredMark,
        PronunciationAssessmentGranularity.Phoneme);

SpeechRecognizer recognizer = new SpeechRecognizer(
    speechConfig,
    audioConfig);

// apply the pronunciation assessment configuration to the speech recognizer
pronunciationAssessmentConfig.applyTo(recognizer);
Future<SpeechRecognitionResult> future = recognizer.recognizeOnceAsync();
SpeechRecognitionResult result = future.get(30, TimeUnit.SECONDS);
PronunciationAssessmentResult pronunciationAssessmentResult =
    PronunciationAssessmentResult.fromResult(result);
Double pronunciationScore = pronunciationAssessmentResult.getPronunciationScore();

recognizer.close();
speechConfig.close();
audioConfig.close();
pronunciationAssessmentConfig.close();
result.close();
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
pronunciation_assessment_config = \
        speechsdk.PronunciationAssessmentConfig(reference_text='reference text',
                grading_system=msspeech.PronunciationAssessmentGradingSystem.HundredMark,
                granularity=msspeech.PronunciationAssessmentGranularity.Phoneme)
speech_recognizer = speechsdk.SpeechRecognizer(
        speech_config=speech_config, \
        audio_config=audio_config)

# apply the pronunciation assessment configuration to the speech recognizer
pronunciation_assessment_config.apply_to(speech_recognizer)
result = speech_recognizer.recognize_once()
pronunciation_assessment_result = speechsdk.PronunciationAssessmentResult(result)
pronunciation_score = pronunciation_assessment_result.pronunciation_score
```

::: zone-end

::: zone pivot="programming-language-javascript"

```Javascript
var pronunciationAssessmentConfig = new SpeechSDK.PronunciationAssessmentConfig("reference text",
    PronunciationAssessmentGradingSystem.HundredMark,
    PronunciationAssessmentGranularity.Word, true);
var speechRecognizer = SpeechSDK.SpeechRecognizer.FromConfig(speechConfig, audioConfig);
// apply the pronunciation assessment configuration to the speech recognizer
pronunciationAssessmentConfig.applyTo(speechRecognizer);

speechRecognizer.recognizeOnceAsync((result: SpeechSDK.SpeechRecognitionResult) => {
        var pronunciationAssessmentResult = SpeechSDK.PronunciationAssessmentResult.fromResult(result);
        var pronunciationScore = pronunciationAssessmentResult.pronunciationScore;
        var wordLevelResult = pronunciationAssessmentResult.detailResult.Words;
},
{});
```

::: zone-end

::: zone pivot="programming-language-objectivec"

```Objective-C
SPXPronunciationAssessmentConfiguration* pronunciationAssessmentConfig =
    [[SPXPronunciationAssessmentConfiguration alloc]init:@"reference text"
                                           gradingSystem:SPXPronunciationAssessmentGradingSystem_HundredMark
                                             granularity:SPXPronunciationAssessmentGranularity_Phoneme];

SPXSpeechRecognizer* speechRecognizer = \
        [[SPXSpeechRecognizer alloc] initWithSpeechConfiguration:speechConfig
                                              audioConfiguration:audioConfig];

// apply the pronunciation assessment configuration to the speech recognizer
[pronunciationAssessmentConfig applyToRecognizer:speechRecognizer];

SPXSpeechRecognitionResult *result = [speechRecognizer recognizeOnce];
SPXPronunciationAssessmentResult* pronunciationAssessmentResult = [[SPXPronunciationAssessmentResult alloc] init:result];
double pronunciationScore = pronunciationAssessmentResult.pronunciationScore;
```

::: zone-end

### <a name="pronunciation-assessment-configuration-parameters"></a>Parámetros de configuración de evaluación de la pronunciación

En esta tabla se enumeran los parámetros de configuración para la evaluación de la pronunciación.

| Parámetro | Descripción | ¿Necesario? |
|-----------|-------------|---------------------|
| ReferenceText | Texto con el que se va a evaluar la pronunciación. | Obligatorio |
| GradingSystem | Sistema de puntos para la calibración de la puntuación. El sistema `FivePoint` da una puntuación de número de punto flotante entre 0 y 5, mientras que `HundredMark` da una puntuación de número de punto flotante entre 0 y 100. Predeterminado: `FivePoint`. | Opcional |
| Granularidad | Granularidad de la evaluación. Los valores aceptados son `Phoneme`, que muestra la puntuación en el nivel de todo el texto, la palabra y el fonema, `Word`, que muestra la puntuación en el nivel de todo el texto y la palabra, `FullText`, que muestra la puntuación solo en el nivel de todo el texto. Predeterminado: `Phoneme`. | Opcional |
| EnableMiscue | Habilita el cálculo de errores. Con esta opción habilitada, las palabras pronunciadas se comparan con el texto de referencia y se marcan con omisión/inserción en función de la comparación. Los valores aceptados son: `False` y `True`. Predeterminado: `False`. | Opcional |
| ScenarioId | GUID que indica un sistema de puntos personalizado. | Opcional |

### <a name="pronunciation-assessment-result-parameters"></a>Parámetros de resultados de evaluación de la pronunciación

En esta tabla se enumeran los parámetros de resultados de evaluación de la pronunciación.

| Parámetro | Description |
|-----------|-------------|
| `AccuracyScore` | Precisión de pronunciación de la voz. La precisión indica el grado de coincidencia de los fonemas con la pronunciación de un hablante nativo. La puntuación de precisión del nivel de texto completo y de las palabras individuales se agrega a partir de la puntuación de precisión del nivel de fonema. |
| `FluencyScore` | Fluidez del fragmento hablado en cuestión. La fluidez indica el grado de coincidencia de la voz con el uso que hace un hablante nativo de los silencios entre palabras. |
| `CompletenessScore` | Integridad de la voz, se determina mediante el cálculo de la proporción de palabras pronunciadas para hacer referencia a la entrada de texto. |
| `PronunciationScore` | Puntuación global que indica la calidad de la pronunciación del fragmento hablado en cuestión. Se agrega a partir de `AccuracyScore`, `FluencyScore` y `CompletenessScore` con ponderación. |
| `ErrorType` | Este valor indica si se ha omitido, se ha insertado o se ha pronunciado incorrectamente una palabra en comparación con `ReferenceText`. Los valores posibles son `None` (que significa que no hay ningún error en esta palabra), `Omission`, `Insertion` y `Mispronunciation`. |

### <a name="sample-responses"></a>Respuestas de ejemplo

Un resultado de la evaluación de pronunciación típico en JSON:

```json
{
  "RecognitionStatus": "Success",
  "Offset": "400000",
  "Duration": "11000000",
  "NBest": [
      {
        "Confidence" : "0.87",
        "Lexical" : "good morning",
        "ITN" : "good morning",
        "MaskedITN" : "good morning",
        "Display" : "Good morning.",
        "PronunciationAssessment":
        {
            "PronScore" : 84.4,
            "AccuracyScore" : 100.0,
            "FluencyScore" : 74.0,
            "CompletenessScore" : 100.0,
        },
        "Words": [
            {
              "Word" : "Good",
              "Offset" : 500000,
              "Duration" : 2700000,
              "PronunciationAssessment":
              {
                "AccuracyScore" : 100.0,
                "ErrorType" : "None"
              }
            },
            {
              "Word" : "morning",
              "Offset" : 5300000,
              "Duration" : 900000,
              "PronunciationAssessment":
              {
                "AccuracyScore" : 100.0,
                "ErrorType" : "None"
              }
            }
        ]
      }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes

<!-- TODO: update JavaScript sample links after release -->

* Vea la [presentación en vídeo](https://www.youtube.com/watch?v=cBE8CUHOFHQ) y el [vídeo tutorial](https://www.youtube.com/watch?v=zFlwm7N4Awc) de evaluación de pronunciación.

* Pruebe la [demostración de evaluación de la pronunciación](https://github.com/Azure-Samples/Cognitive-Speech-TTS/tree/master/PronunciationAssessment/BrowserJS).

::: zone pivot="programming-language-csharp"
* Consulte el [ejemplo de código](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/csharp/sharedcontent/console/speech_recognition_samples.cs#L949) de GitHub para la evaluación de la pronunciación.
::: zone-end

::: zone pivot="programming-language-cpp"
* Consulte el [ejemplo de código](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/cpp/windows/console/samples/speech_recognition_samples.cpp#L633) de GitHub para la evaluación de la pronunciación.
::: zone-end

::: zone pivot="programming-language-java"
* Consulte el [ejemplo de código](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/java/jre/console/src/com/microsoft/cognitiveservices/speech/samples/console/SpeechRecognitionSamples.java#L697) de GitHub para la evaluación de la pronunciación.
::: zone-end

::: zone pivot="programming-language-python"
* Consulte el [ejemplo de código](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/python/console/speech_sample.py#L576) de GitHub para la evaluación de la pronunciación.
::: zone-end

::: zone pivot="programming-language-objectivec"
* Consulte el [ejemplo de código](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/objective-c/ios/speech-samples/speech-samples/ViewController.m#L642) de GitHub para la evaluación de la pronunciación.
::: zone-end

* [Documentación de referencia del SDK de voz](speech-sdk.md)

* [Creación de una cuenta de Azure gratuita](https://azure.microsoft.com/free/cognitive-services/)
