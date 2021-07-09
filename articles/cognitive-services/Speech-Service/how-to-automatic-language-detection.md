---
title: Uso de la identificación del idioma
titleSuffix: Azure Cognitive Services
description: La identificación del idioma se usa para determinar el idioma que se habla en el audio que se pasa al SDK de Voz mediante la comparación con una lista de idiomas proporcionados.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/21/2021
ms.author: trbye
zone_pivot_groups: programming-languages-speech-services-nomore-variant
ms.openlocfilehash: 2c9adacbb9e333c81c4f90868a69f8a23a3c15a7
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111962925"
---
# <a name="how-to-use-language-identification"></a>Uso de la identificación del idioma

La identificación del idioma se usa para determinar el idioma que se habla en el audio que se pasa al SDK de Voz mediante la comparación con una lista de idiomas proporcionados. El valor devuelto por la identificación del idioma se usa para seleccionar el modelo de idioma para la conversión de voz en texto, lo que proporciona una transcripción más precisa. 

La identificación del idioma también se puede usar con la [traducción de voz](./get-started-speech-translation.md?pivots=programming-language-csharp&tabs=script%2cwindowsinstall#multi-lingual-translation-with-language-identification) o mediante la [identificación independiente](#standalone-language-identification). Para ver los idiomas que están disponibles, consulte la [compatibilidad de idioma](language-support.md).

## <a name="prerequisites"></a>Prerrequisitos

En este artículo se da por supuesto que tiene una suscripción de Azure y un recurso de voz, y también, que conoce los conceptos básicos del reconocimiento de voz. [Complete el inicio rápido](get-started-speech-to-text.md) si aún no lo ha hecho.

## <a name="language-identification-with-speech-to-text"></a>Identificación del idioma con voz a texto

La identificación del idioma tiene actualmente un límite de **cuatro idiomas** para el reconocimiento único y de **10 idiomas** para el reconocimiento continuo. Tenga en cuenta esta limitación al construir el objeto `AutoDetectSourceLanguageConfig`. En los ejemplos siguientes se usa `AutoDetectSourceLanguageConfig` para definir una lista de los posibles idiomas que desea identificar y hacer referencia a ellos al ejecutar el reconocimiento de voz.

> [!IMPORTANT]
> La identificación continua del idioma está actualmente en **versión preliminar** y solo se admite en C# y C++.

::: zone pivot="programming-language-csharp"

En el ejemplo siguiente se ejecuta el reconocimiento único, priorizando `Latency`. Esta propiedad también se puede establecer en `Accuracy` en función de la prioridad del caso de uso. `Latency` es la mejor opción si necesita un resultado de baja latencia (por ejemplo, para escenarios de streaming en vivo), si no conoce el idioma del ejemplo de audio. 

`Accuracy` se debe usar en escenarios en los que la calidad del audio puede ser deficiente y se acepta más latencia. Por ejemplo, un sonido de voz podría tener ruido de fondo o algo de silencio al principio; permitir que el motor tenga más tiempo mejorará los resultados del reconocimiento.

En cualquier caso, el reconocimiento único como se muestra a continuación **no debe usarse** para escenarios en los que el idioma puede cambiar dentro de la misma muestra de audio. Consulte a continuación el reconocimiento continuo para estos tipos de escenarios.

```csharp
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;

var speechConfig = SpeechConfig.FromSubscription("<paste-your-subscription-key>","<paste-your-region>");
// can switch "Latency" to "Accuracy" depending on priority
speechConfig.SetProperty(PropertyId.SpeechServiceConnection_SingleLanguageIdPriority, "Latency");

var autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig.FromLanguages(
        new string[] { "en-US", "de-DE", "ja-JP", "de-DE" });

using var audioConfig = AudioConfig.FromDefaultMicrophoneInput();
using (var recognizer = new SpeechRecognizer(
    speechConfig,
    autoDetectSourceLanguageConfig,
    audioConfig))
{
    var speechRecognitionResult = await recognizer.RecognizeOnceAsync();
    var autoDetectSourceLanguageResult =
        AutoDetectSourceLanguageResult.FromResult(speechRecognitionResult);
    var detectedLanguage = autoDetectSourceLanguageResult.Language;
}
```

En el ejemplo siguiente se muestra la configuración del reconocimiento de voz continuo para un escenario multilingüe. En este ejemplo solo se usan `en-US` y `ja-JP` en la configuración del idioma, pero se pueden usar hasta **diez idiomas** para este patrón de diseño. Cada vez que se detecta voz, se identifica el idioma de origen y el audio también se convierte en salida de texto. En este ejemplo se usa el modo `Latency`, que da prioridad al tiempo de respuesta.

```csharp
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;

var region = "<paste-your-region>";
// currently the v2 endpoint is required for this design pattern
var endpointString = $"wss://{region}.stt.speech.microsoft.com/speech/universal/v2";
var endpointUrl = new Uri(endpointString);

var config = SpeechConfig.FromEndpoint(endpointUrl, "<paste-your-subscription-key>");
// can switch "Latency" to "Accuracy" depending on priority
config.SetProperty(PropertyId.SpeechServiceConnection_ContinuousLanguageIdPriority, "Latency");

var autoDetectSourceLanguageConfig = AutoDetectSourceLanguageConfig.FromLanguages(new string[] { "en-US", "ja-JP" });

var stopRecognition = new TaskCompletionSource<int>();
using (var audioInput = AudioConfig.FromWavFileInput(@"path-to-your-audio-file.wav"))
{
    using (var recognizer = new SpeechRecognizer(config, autoDetectSourceLanguageConfig, audioInput))
    {
        // Subscribes to events.
        recognizer.Recognizing += (s, e) =>
        {
            if (e.Result.Reason == ResultReason.RecognizingSpeech)
            {
                Console.WriteLine($"RECOGNIZING: Text={e.Result.Text}");
                var autoDetectSourceLanguageResult = AutoDetectSourceLanguageResult.FromResult(e.Result);
                Console.WriteLine($"DETECTED: Language={autoDetectSourceLanguageResult.Language}");
            }
        };

        recognizer.Recognized += (s, e) =>
        {
            if (e.Result.Reason == ResultReason.RecognizedSpeech)
            {
                Console.WriteLine($"RECOGNIZED: Text={e.Result.Text}");
                var autoDetectSourceLanguageResult = AutoDetectSourceLanguageResult.FromResult(e.Result);
                Console.WriteLine($"DETECTED: Language={autoDetectSourceLanguageResult.Language}");
            }
            else if (e.Result.Reason == ResultReason.NoMatch)
            {
                Console.WriteLine($"NOMATCH: Speech could not be recognized.");
            }
        };

        recognizer.Canceled += (s, e) =>
        {
            Console.WriteLine($"CANCELED: Reason={e.Reason}");

            if (e.Reason == CancellationReason.Error)
            {
                Console.WriteLine($"CANCELED: ErrorCode={e.ErrorCode}");
                Console.WriteLine($"CANCELED: ErrorDetails={e.ErrorDetails}");
                Console.WriteLine($"CANCELED: Did you update the subscription info?");
            }

            stopRecognition.TrySetResult(0);
        };

        recognizer.SessionStarted += (s, e) =>
        {
            Console.WriteLine("\n    Session started event.");
        };

        recognizer.SessionStopped += (s, e) =>
        {
            Console.WriteLine("\n    Session stopped event.");
            Console.WriteLine("\nStop recognition.");
            stopRecognition.TrySetResult(0);
        };

        // Starts continuous recognition. Uses StopContinuousRecognitionAsync() to stop recognition.
        await recognizer.StartContinuousRecognitionAsync().ConfigureAwait(false);

        // Waits for completion.
        // Use Task.WaitAny to keep the task rooted.
        Task.WaitAny(new[] { stopRecognition.Task });

        // Stops recognition.
        await recognizer.StopContinuousRecognitionAsync().ConfigureAwait(false);
    }
}
```

> [!NOTE]
> Los modos `Latency` y `Accuracy`, y el reconocimiento continuo multilingüe solo se admiten actualmente en C# y C++.
 
::: zone-end

::: zone pivot="programming-language-cpp"

> [!IMPORTANT]
> Esta característica que se muestra a continuación se encuentra actualmente en **versión preliminar**.

En el ejemplo siguiente se ejecuta el reconocimiento único, priorizando `Latency`. Esta propiedad también se puede establecer en `Accuracy` en función de la prioridad del caso de uso. `Latency` es la mejor opción si necesita un resultado de baja latencia (por ejemplo, para un caso de streaming en vivo), si no conoce el idioma del ejemplo de audio. 

`Accuracy` se debe usar en escenarios en los que la calidad del audio puede ser deficiente y se acepta más latencia. Por ejemplo, un sonido de voz podría tener ruido de fondo o algo de silencio al principio; permitir que el motor tenga más tiempo mejorará los resultados del reconocimiento.

En cualquier caso, el reconocimiento único como se muestra a continuación **no debe usarse** para escenarios en los que el idioma puede cambiar dentro de la misma muestra de audio. Consulte a continuación el reconocimiento continuo para estos tipos de escenarios.

```cpp
using namespace std;
using namespace Microsoft::CognitiveServices::Speech;
using namespace Microsoft::CognitiveServices::Speech::Audio;

auto speechConfig = SpeechConfig::FromSubscription("<paste-your-subscription-key>","<paste-your-region>");
speechConfig->SetProperty(PropertyId::SpeechServiceConnection_SingleLanguageIdPriority, "Latency");

auto autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig::FromLanguages({ "en-US", "de-DE", "ja-JP", "de-DE" });

auto recognizer = SpeechRecognizer::FromConfig(
    speechConfig,
    autoDetectSourceLanguageConfig
    );

speechRecognitionResult = recognizer->RecognizeOnceAsync().get();
auto autoDetectSourceLanguageResult =
    AutoDetectSourceLanguageResult::FromResult(speechRecognitionResult);
auto detectedLanguage = autoDetectSourceLanguageResult->Language;
```

En el ejemplo siguiente se muestra la configuración del reconocimiento de voz continuo para un escenario multilingüe. En este ejemplo solo se usan `en-US` y `ja-JP` en la configuración del idioma, pero se pueden usar hasta **diez idiomas** para este patrón de diseño. Cada vez que se detecta voz, se identifica el idioma de origen y el audio también se convierte en salida de texto. En este ejemplo se usa el modo `Latency`, que da prioridad al tiempo de respuesta.

```cpp
using namespace std;
using namespace Microsoft::CognitiveServices::Speech;
using namespace Microsoft::CognitiveServices::Speech::Audio;

auto region = "<paste-your-region>";
// currently the v2 endpoint is required for this design pattern
auto endpointString = std::format("wss://{}.stt.speech.microsoft.com/speech/universal/v2", region);
auto config = SpeechConfig::FromEndpoint(endpointString, "<paste-your-subscription-key>");

config->SetProperty(PropertyId::SpeechServiceConnection_ContinuousLanguageIdPriority, "Latency");
auto autoDetectSourceLanguageConfig = AutoDetectSourceLanguageConfig::FromLanguages({ "en-US", "ja-JP" });

auto audioInput = AudioConfig::FromWavFileInput("path-to-your-audio-file.wav");
auto recognizer = SpeechRecognizer::FromConfig(config, autoDetectSourceLanguageConfig, audioInput);

// promise for synchronization of recognition end.
promise<void> recognitionEnd;

// Subscribes to events.
recognizer->Recognizing.Connect([](const SpeechRecognitionEventArgs& e)
    {
        auto lidResult = AutoDetectSourceLanguageResult::FromResult(e.Result);
        cout << "Recognizing in " << lidResult->Language << ": Text =" << e.Result->Text << std::endl;
    });

recognizer->Recognized.Connect([](const SpeechRecognitionEventArgs& e)
    {
        if (e.Result->Reason == ResultReason::RecognizedSpeech)
        {
            auto lidResult = AutoDetectSourceLanguageResult::FromResult(e.Result);
            cout << "RECOGNIZED in " << lidResult->Language << ": Text=" << e.Result->Text << "\n"
                << "  Offset=" << e.Result->Offset() << "\n"
                << "  Duration=" << e.Result->Duration() << std::endl;
        }
        else if (e.Result->Reason == ResultReason::NoMatch)
        {
            cout << "NOMATCH: Speech could not be recognized." << std::endl;
        }
    });

recognizer->Canceled.Connect([&recognitionEnd](const SpeechRecognitionCanceledEventArgs& e)
    {
        cout << "CANCELED: Reason=" << (int)e.Reason << std::endl;

        if (e.Reason == CancellationReason::Error)
        {
            cout << "CANCELED: ErrorCode=" << (int)e.ErrorCode << "\n"
                << "CANCELED: ErrorDetails=" << e.ErrorDetails << "\n"
                << "CANCELED: Did you update the subscription info?" << std::endl;

            recognitionEnd.set_value(); // Notify to stop recognition.
        }
    });

recognizer->SessionStopped.Connect([&recognitionEnd](const SessionEventArgs& e)
    {
        cout << "Session stopped.";
        recognitionEnd.set_value(); // Notify to stop recognition.
    });

recognizer->StartContinuousRecognitionAsync().get();

// Waits for recognition end.
recognitionEnd.get_future().get();

// Stops recognition.
recognizer->StopContinuousRecognitionAsync().get();
```

> [!NOTE]
> Los modos `Latency` y `Accuracy`, y el reconocimiento continuo multilingüe solo se admiten actualmente en C# y C++.

::: zone-end

::: zone pivot="programming-language-java"

```java
AutoDetectSourceLanguageConfig autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig.fromLanguages(Arrays.asList("en-US", "de-DE"));

SpeechRecognizer recognizer = new SpeechRecognizer(
    speechConfig,
    autoDetectSourceLanguageConfig,
    audioConfig);

Future<SpeechRecognitionResult> future = recognizer.recognizeOnceAsync();
SpeechRecognitionResult result = future.get(30, TimeUnit.SECONDS);
AutoDetectSourceLanguageResult autoDetectSourceLanguageResult =
    AutoDetectSourceLanguageResult.fromResult(result);
String detectedLanguage = autoDetectSourceLanguageResult.getLanguage();

recognizer.close();
speechConfig.close();
autoDetectSourceLanguageConfig.close();
audioConfig.close();
result.close();
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
auto_detect_source_language_config = \
        speechsdk.languageconfig.AutoDetectSourceLanguageConfig(languages=["en-US", "de-DE"])
speech_recognizer = speechsdk.SpeechRecognizer(
        speech_config=speech_config, 
        auto_detect_source_language_config=auto_detect_source_language_config, 
        audio_config=audio_config)
result = speech_recognizer.recognize_once()
auto_detect_source_language_result = speechsdk.AutoDetectSourceLanguageResult(result)
detected_language = auto_detect_source_language_result.language
```

::: zone-end

::: zone pivot="programming-language-objectivec"

```Objective-C
NSArray *languages = @[@"zh-CN", @"de-DE"];
SPXAutoDetectSourceLanguageConfiguration* autoDetectSourceLanguageConfig = \
        [[SPXAutoDetectSourceLanguageConfiguration alloc]init:languages];
SPXSpeechRecognizer* speechRecognizer = \
        [[SPXSpeechRecognizer alloc] initWithSpeechConfiguration:speechConfig
                           autoDetectSourceLanguageConfiguration:autoDetectSourceLanguageConfig
                                              audioConfiguration:audioConfig];
SPXSpeechRecognitionResult *result = [speechRecognizer recognizeOnce];
SPXAutoDetectSourceLanguageResult *languageDetectionResult = [[SPXAutoDetectSourceLanguageResult alloc] init:result];
NSString *detectedLanguage = [languageDetectionResult language];
```

::: zone-end

::: zone pivot="programming-language-javascript"

```Javascript
var autoDetectConfig = SpeechSDK.AutoDetectSourceLanguageConfig.fromLanguages(["en-US", "de-DE"]);
var speechRecognizer = SpeechSDK.SpeechRecognizer.FromConfig(speechConfig, autoDetectConfig, audioConfig);
speechRecognizer.recognizeOnceAsync((result: SpeechSDK.SpeechRecognitionResult) => {
        var languageDetectionResult = SpeechSDK.AutoDetectSourceLanguageResult.fromResult(result);
        var detectedLanguage = languageDetectionResult.language;
},
{});
```

::: zone-end

## <a name="standalone-language-identification"></a>Identificación del idioma independiente

::: zone pivot="programming-language-csharp"

En los casos de uso en los que solo desee detectar el idioma de origen que se habla, puede usar la identificación del idioma independiente como se muestra en el ejemplo de código siguiente. `SourceLanguageRecognizer` también se puede usar en escenarios de reconocimiento continuo.

```csharp
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;

var speechConfig = SpeechConfig.FromSubscription("<paste-your-subscription-key>","<paste-your-region>");
// can switch "Latency" to "Accuracy" depending on priority
speechConfig.SetProperty(PropertyId.SpeechServiceConnection_SingleLanguageIdPriority, "Latency");

var autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig.FromLanguages(
        new string[] { "en-US", "de-DE" });

using (var recognizer = new SourceLanguageRecognizer(speechConfig, autoDetectSourceLanguageConfig))
{
    var result = await recognizer.RecognizeOnceAsync();
    if (result.Reason == ResultReason.RecognizedSpeech)
    {
        var lang = AutoDetectSourceLanguageResult.FromResult(result).Language;
        Console.WriteLine($"DETECTED: Language={lang}");
    }
}
```

Consulte el [ejemplo en GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/csharp/sharedcontent/console/standalone_language_detection_samples.cs) para más ejemplos de identificación del idioma independiente, incluido un ejemplo de identificación continua.

::: zone-end

::: zone pivot="programming-language-cpp"

En los casos de uso en los que solo desee detectar el idioma de origen que se habla, puede usar la identificación del idioma independiente como se muestra en el ejemplo de código siguiente. `SourceLanguageRecognizer` también se puede usar en escenarios de reconocimiento continuo.

```cpp
using namespace std;
using namespace Microsoft::CognitiveServices::Speech;
using namespace Microsoft::CognitiveServices::Speech::Audio;

auto config = SpeechConfig::FromSubscription("<paste-your-subscription-key>","<paste-your-region>");
config->SetProperty(PropertyId::SpeechServiceConnection_SingleLanguageIdPriority, "Latency");

auto autoDetectSourceLanguageConfig = AutoDetectSourceLanguageConfig::FromLanguages({ "en-US", "de-DE" });

auto recognizer = SourceLanguageRecognizer::FromConfig(config, autoDetectSourceLanguageConfig);
cout << "Say something...\n";

auto result = recognizer->RecognizeOnceAsync().get();
if (result->Reason == ResultReason::RecognizedSpeech)
{
    auto lidResult = AutoDetectSourceLanguageResult::FromResult(result);
    cout << "DETECTED: Language="<< lidResult->Language << std::endl;
}
```

Consulte el [ejemplo en GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/cpp/windows/console/samples/standalone_language_detection_samples.cpp) para más ejemplos de identificación del idioma independiente, incluido un ejemplo de identificación continua.

::: zone-end

::: zone pivot="programming-language-java"
> [!IMPORTANT]
> Actualmente esta característica solo se admite en C# y C++.
::: zone-end

::: zone pivot="programming-language-python"
> [!IMPORTANT]
> Actualmente esta característica solo se admite en C# y C++.
::: zone-end

::: zone pivot="programming-language-objectivec"
> [!IMPORTANT]
> Actualmente esta característica solo se admite en C# y C++.
::: zone-end

::: zone pivot="programming-language-javascript"
> [!IMPORTANT]
> Actualmente esta característica solo se admite en C# y C++.
::: zone-end

## <a name="use-a-custom-model-for-language-identification"></a>Uso de un modelo personalizado para la identificación del idioma

Además de la identificación del idioma mediante los modelos base del servicio de Voz, puede especificar un modelo personalizado para mejor reconocimiento. Si no se proporciona un modelo personalizado, el servicio utilizará el modelo de lenguaje predeterminado.

En los fragmentos de código siguientes se muestra cómo especificar un modelo personalizado en la llamada al servicio de Voz. Si el idioma detectado es `en-US`, se usa el modelo predeterminado. Si el idioma detectado es `fr-FR`, se usa el punto de conexión para el modelo personalizado:

::: zone pivot="programming-language-csharp"

```csharp
var sourceLanguageConfigs = new SourceLanguageConfig[]
{
    SourceLanguageConfig.FromLanguage("en-US"),
    SourceLanguageConfig.FromLanguage("fr-FR", "The Endpoint Id for custom model of fr-FR")
};
var autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig.FromSourceLanguageConfigs(
        sourceLanguageConfigs);
```

::: zone-end

::: zone pivot="programming-language-cpp"

```cpp
std::vector<std::shared_ptr<SourceLanguageConfig>> sourceLanguageConfigs;
sourceLanguageConfigs.push_back(
    SourceLanguageConfig::FromLanguage("en-US"));
sourceLanguageConfigs.push_back(
    SourceLanguageConfig::FromLanguage("fr-FR", "The Endpoint Id for custom model of fr-FR"));

auto autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig::FromSourceLanguageConfigs(
        sourceLanguageConfigs);
```

::: zone-end

::: zone pivot="programming-language-java"

```java
List sourceLanguageConfigs = new ArrayList<SourceLanguageConfig>();
sourceLanguageConfigs.add(
    SourceLanguageConfig.fromLanguage("en-US"));
sourceLanguageConfigs.add(
    SourceLanguageConfig.fromLanguage("fr-FR", "The Endpoint Id for custom model of fr-FR"));

AutoDetectSourceLanguageConfig autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig.fromSourceLanguageConfigs(
        sourceLanguageConfigs);
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
 en_language_config = speechsdk.languageconfig.SourceLanguageConfig("en-US")
 fr_language_config = speechsdk.languageconfig.SourceLanguageConfig("fr-FR", "The Endpoint Id for custom model of fr-FR")
 auto_detect_source_language_config = speechsdk.languageconfig.AutoDetectSourceLanguageConfig(
        sourceLanguageConfigs=[en_language_config, fr_language_config])
```

::: zone-end

::: zone pivot="programming-language-objectivec"

```Objective-C
SPXSourceLanguageConfiguration* enLanguageConfig = [[SPXSourceLanguageConfiguration alloc]init:@"en-US"];
SPXSourceLanguageConfiguration* frLanguageConfig = \
        [[SPXSourceLanguageConfiguration alloc]initWithLanguage:@"fr-FR"
                                                     endpointId:@"The Endpoint Id for custom model of fr-FR"];
NSArray *languageConfigs = @[enLanguageConfig, frLanguageConfig];
SPXAutoDetectSourceLanguageConfiguration* autoDetectSourceLanguageConfig = \
        [[SPXAutoDetectSourceLanguageConfiguration alloc]initWithSourceLanguageConfigurations:languageConfigs];
```

::: zone-end

::: zone pivot="programming-language-javascript"

```Javascript
var enLanguageConfig = SpeechSDK.SourceLanguageConfig.fromLanguage("en-US");
var frLanguageConfig = SpeechSDK.SourceLanguageConfig.fromLanguage("fr-FR", "The Endpoint Id for custom model of fr-FR");
var autoDetectConfig = SpeechSDK.AutoDetectSourceLanguageConfig.fromSourceLanguageConfigs([enLanguageConfig, frLanguageConfig]);
```

::: zone-end

## <a name="next-steps"></a>Pasos siguientes

::: zone pivot="programming-language-csharp"
* Consulte el [código de ejemplo](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/csharp/sharedcontent/console/speech_recognition_samples.cs#L741) en GitHub para la identificación del idioma.
::: zone-end

::: zone pivot="programming-language-cpp"
* Consulte el [código de ejemplo](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/cpp/windows/console/samples/speech_recognition_samples.cpp#L507) en GitHub para la identificación del idioma.
::: zone-end

::: zone pivot="programming-language-java"
* Consulte el [código de ejemplo](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/java/jre/console/src/com/microsoft/cognitiveservices/speech/samples/console/SpeechRecognitionSamples.java#L521) en GitHub para la identificación del idioma.
::: zone-end

::: zone pivot="programming-language-python"
* Consulte el [código de ejemplo](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/python/console/speech_sample.py#L458) en GitHub para la identificación del idioma.
::: zone-end

::: zone pivot="programming-language-objectivec"
* Consulte el [código de ejemplo](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/objective-c/ios/speech-samples/speech-samples/ViewController.m#L525) en GitHub para la identificación del idioma.
::: zone-end

* [Documentación de referencia del SDK de voz](speech-sdk.md)