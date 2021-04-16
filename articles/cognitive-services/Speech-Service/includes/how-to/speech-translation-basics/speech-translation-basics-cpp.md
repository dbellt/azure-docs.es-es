---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 04/13/2020
ms.author: trbye
ms.openlocfilehash: eaf8d8f741c120297e6ae57e8ddb8b62f7c7f534
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105104715"
---
Una de las características principales del servicio de voz es la capacidad para reconocer la voz humana y traducirla a otros idiomas. En este inicio rápido, aprenderá a usar Speech SDK en sus aplicaciones y productos para realizar la traducción de voz de alta calidad. En este inicio rápido se tratan temas que incluyen:

* Traducción de voz a texto
* Traducción de voz a varios idiomas de destino
* Realizar la traducción de voz a voz directa

## <a name="skip-to-samples-on-github"></a>Pasar a los ejemplos en GitHub

Si quiere pasar directamente al código de ejemplo, consulte los [ejemplos del inicio rápido de C++](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/cpp/windows/translate-speech-to-text) en GitHub.

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por sentado que tiene una cuenta de Azure y una suscripción al servicio de voz. Si no dispone de una cuenta y una suscripción, [pruebe el servicio de voz de forma gratuita](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-the-speech-sdk"></a>Instalación de Speech SDK

En primer lugar, deberá instalar Speech SDK. Dependiendo de la plataforma, siga las instrucciones de la sección <a href="/azure/cognitive-services/speech-service/speech-sdk#get-the-speech-sdk" target="_blank">Obtención del SDK de Voz</a> del artículo _Acerca del SDK de Voz_.

## <a name="import-dependencies"></a>Dependencias de importación

Para ejecutar los ejemplos de este artículo, incluya las siguientes instrucciones `#include` y `using` al principio del archivo de código C++.

```cpp
#include <iostream> // cin, cout
#include <fstream>
#include <string>
#include <stdio.h>
#include <stdlib.h>
#include <speechapi_cxx.h>

using namespace std;
using namespace Microsoft::CognitiveServices::Speech;
using namespace Microsoft::CognitiveServices::Speech::Audio;
using namespace Microsoft::CognitiveServices::Speech::Translation;
```

## <a name="sensitive-data-and-environment-variables"></a>Datos confidenciales y variables de entorno

El código fuente de ejemplo de este artículo depende de las variables de entorno para almacenar datos confidenciales, como la región y la clave de suscripción del recurso de voz. El archivo de código C++ contiene dos valores de cadena que se asignan a las variables de entorno de las máquinas host, es decir, `SPEECH__SUBSCRIPTION__KEY` y `SPEECH__SERVICE__REGION`. Ambos campos se encuentran en el ámbito de clase, lo que les permite tener acceso a ellos en los cuerpos de método de la clase. Para más información sobre las variables de entorno, consulte [Configuración de las variables de entorno y de la aplicación](../../../../cognitive-services-security.md#environment-variables-and-application-configuration).

```cpp
auto SPEECH__SUBSCRIPTION__KEY = getenv("SPEECH__SUBSCRIPTION__KEY");
auto SPEECH__SERVICE__REGION = getenv("SPEECH__SERVICE__REGION");
```

## <a name="create-a-speech-translation-configuration"></a>Creación de una configuración de traducción de voz

Para llamar al servicio de voz con Speech SDK, debe crear un elemento [`SpeechTranslationConfig`][config]. Esta clase incluye información sobre la suscripción, como la clave, la región asociada, el punto de conexión, el host o el token de autorización.

> [!TIP]
> Debe crear siempre una configuración, independientemente de si va a realizar reconocimiento de voz, síntesis de voz, traducción o reconocimiento de intenciones.

Existen diversas maneras para inicializar un elemento [`SpeechTranslationConfig`][config]:

* Con una suscripción: pase una clave y la región asociada.
* Con un punto de conexión: pase un punto de conexión del servicio de voz. La clave y el token de autorización son opcionales.
* Con un host: pase una dirección de host. La clave y el token de autorización son opcionales.
* Con un token de autorización: pase el token de autorización y la región asociada.

Vamos a echar un vistazo al procedimiento de creación de un elemento [`SpeechTranslationConfig`][config] con una clave y una región. Para obtener estas credenciales, siga los pasos descritos en [Prueba gratuita del servicio Voz](../../../overview.md#try-the-speech-service-for-free).

```cpp
auto SPEECH__SUBSCRIPTION__KEY = getenv("SPEECH__SUBSCRIPTION__KEY");
auto SPEECH__SERVICE__REGION = getenv("SPEECH__SERVICE__REGION");

void translateSpeech() {
    auto config =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
}

int main(int argc, char** argv) {
    setlocale(LC_ALL, "");
    translateSpeech();
    return 0;
}
```

## <a name="change-source-language"></a>Cambio del idioma de origen

Una tarea común en la traducción de voz es especificar el idioma de entrada (u origen). Echemos un vistazo a cómo se cambiaría el idioma de entrada a italiano. En el código, interactúe con la instancia [`SpeechTranslationConfig`][config], mediante la llamada al método `SetSpeechRecognitionLanguage`.

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    // Source (input) language
    translationConfig->SetSpeechRecognitionLanguage("it-IT");
}
```

La propiedad [`SpeechRecognitionLanguage`][recognitionlang] espera una cadena con formato de configuración regional de idioma. En la columna **Locale**, puede proporcionar cualquier valor de la lista de idiomas o configuraciones regionales [compatibles](../../../language-support.md).

## <a name="add-translation-language"></a>Incorporación del idioma de traducción

Otra tarea común de la traducción de voz es especificar los idiomas de traducción de destino; se requiere al menos uno, pero se admiten varios. En el fragmento de código siguiente, se han especificado el francés y el alemán como destinos del idioma de traducción.

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    translationConfig->SetSpeechRecognitionLanguage("it-IT");

    // Translate to languages. See, https://aka.ms/speech/sttt-languages
    translationConfig->AddTargetLanguage("fr");
    translationConfig->AddTargetLanguage("de");
}
```

Con cada llamada a [`AddTargetLanguage`][addlang], se especifica un nuevo idioma de traducción de destino. En otras palabras, cuando se reconoce la voz del lenguaje de origen, cada traducción de destino está disponible como parte de la operación de traducción resultante.

## <a name="initialize-a-translation-recognizer"></a>Inicialización de un reconocedor de traducción

Una vez creado un elemento [`SpeechTranslationConfig`][config], el paso siguiente consiste en inicializar un elemento [`TranslationRecognizer`][recognizer]. Al inicializar un elemento [`TranslationRecognizer`][recognizer], deberá pasar el elemento `translationConfig`. El objeto de configuración proporciona las credenciales que necesita el servicio de voz para validar la solicitud.

Si va a realizar un reconocimiento de voz con el micrófono predeterminado del dispositivo, el elemento [`TranslationRecognizer`][recognizer] debería ser similar a:

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    auto fromLanguage = "en-US";
    auto toLanguages = { "it", "fr", "de" };
    translationConfig->SetSpeechRecognitionLanguage(fromLanguage);
    for (auto language : toLanguages) {
        translationConfig->AddTargetLanguage(language);
    }

    auto recognizer = TranslationRecognizer::FromConfig(translationConfig);
}
```

Si desea especificar el dispositivo de entrada de audio, deberá crear un elemento [`AudioConfig`][audioconfig] y proporcionar el parámetro `audioConfig` al inicializar el elemento [`TranslationRecognizer`][recognizer].

> [!TIP]
> [Obtenga información sobre cómo obtener el identificador de dispositivo del dispositivo de entrada de audio](../../../how-to-select-audio-input-devices.md).

En primer lugar, haga referencia al objeto `AudioConfig` como se indica a continuación:

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    auto fromLanguage = "en-US";
    auto toLanguages = { "it", "fr", "de" };
    translationConfig->SetSpeechRecognitionLanguage(fromLanguage);
    for (auto language : toLanguages) {
        translationConfig->AddTargetLanguage(language);
    }

    auto audioConfig = AudioConfig::FromDefaultMicrophoneInput();
    auto recognizer = TranslationRecognizer::FromConfig(translationConfig, audioConfig);
}
```

Si desea proporcionar un archivo de audio en lugar de usar un micrófono, deberá proporcionar un elemento `audioConfig`. Sin embargo, cuando se crea un elemento [`AudioConfig`][audioconfig], en lugar de llamar a `FromDefaultMicrophoneInput`, llamará a `FromWavFileInput` y pasará el parámetro `filename`.

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    auto fromLanguage = "en-US";
    auto toLanguages = { "it", "fr", "de" };
    translationConfig->SetSpeechRecognitionLanguage(fromLanguage);
    for (auto language : toLanguages) {
        translationConfig->AddTargetLanguage(language);
    }

    auto audioConfig = AudioConfig::FromWavFileInput("YourAudioFile.wav");
    auto recognizer = TranslationRecognizer::FromConfig(translationConfig, audioConfig);
}
```

## <a name="translate-speech"></a>Traducir voz

Para traducir la voz, el SDK de Voz depende de un micrófono o en una entrada de archivo de audio. El reconocimiento de voz tiene lugar antes de la traducción de voz. Cuando todos los objetos se hayan inicializado, llame a la función recognize-once y obtenga el resultado.

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    string fromLanguage = "en-US";
    string toLanguages[3] = { "it", "fr", "de" };
    translationConfig->SetSpeechRecognitionLanguage(fromLanguage);
    for (auto language : toLanguages) {
        translationConfig->AddTargetLanguage(language);
    }

    auto recognizer = TranslationRecognizer::FromConfig(translationConfig);
    cout << "Say something in '" << fromLanguage << "' and we'll translate...\n";

    auto result = recognizer->RecognizeOnceAsync().get();
    if (result->Reason == ResultReason::TranslatedSpeech)
    {
        cout << "Recognized: \"" << result->Text << "\"" << std::endl;
        for (auto pair : result->Translations)
        {
            auto language = pair.first;
            auto translation = pair.second;
            cout << "Translated into '" << language << "': " << translation << std::endl;
        }
    }
}
```

Para más información acerca de la conversión de voz a texto, consulte [Aspectos básicos del reconocimiento de voz](../../../get-started-speech-to-text.md).

## <a name="synthesize-translations"></a>Síntesis de traducciones

Después de un reconocimiento de voz y una traducción correctos, el resultado contiene todas las traducciones de un diccionario. La clave del diccionario [`Translations`][translations] es el idioma de traducción de destino y el valor es el texto traducido. La voz reconocida se puede traducir y sintetizar en otro idioma (de voz a voz).

### <a name="event-based-synthesis"></a>Síntesis basada en eventos

El objeto `TranslationRecognizer` expone un evento `Synthesizing`. El evento se activa varias veces y proporciona un mecanismo para recuperar el audio sintetizado del resultado del reconocimiento de la traducción. Si va a traducir a varios idiomas, consulte [Síntesis manual](#manual-synthesis). Especifique la voz de síntesis asignando un [`SetVoiceName`][voicename] y proporcione un controlador de eventos para el evento de `Synthesizing` y obtenga el audio. En el ejemplo siguiente se guarda el audio traducido como un archivo *.wav*.

> [!IMPORTANT]
> La síntesis basada en eventos solo funciona con una sola traducción, **no** agrega varios idiomas de traducción de destino. Además, el objeto [`SetVoiceName`][voicename] debe ser el mismo idioma que el idioma de traducción de destino; por ejemplo, `"de"` podría asignarse a `"de-DE-Hedda"`.

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    auto fromLanguage = "en-US";
    auto toLanguage = "de";
    translationConfig->SetSpeechRecognitionLanguage(fromLanguage);
    translationConfig->AddTargetLanguage(toLanguage);

    // See: https://aka.ms/speech/sdkregion#standard-and-neural-voices
    translationConfig->SetVoiceName("de-DE-Hedda");

    auto recognizer = TranslationRecognizer::FromConfig(translationConfig);
    recognizer->Synthesizing.Connect([](const TranslationSynthesisEventArgs& e)
        {
            auto audio = e.Result->Audio;
            auto size = audio.size();
            cout << "Audio synthesized: " << size << " byte(s)" << (size == 0 ? "(COMPLETE)" : "") << std::endl;

            if (size > 0) {
                ofstream file("translation.wav", ios::out | ios::binary);
                auto audioData = audio.data();
                file.write((const char*)audioData, sizeof(audio[0]) * size);
                file.close();
            }
        });

    cout << "Say something in '" << fromLanguage << "' and we'll translate...\n";

    auto result = recognizer->RecognizeOnceAsync().get();
    if (result->Reason == ResultReason::TranslatedSpeech)
    {
        cout << "Recognized: \"" << result->Text << "\"" << std::endl;
        for (auto pair : result->Translations)
        {
            auto language = pair.first;
            auto translation = pair.second;
            cout << "Translated into '" << language << "': " << translation << std::endl;
        }
    }
}
```

### <a name="manual-synthesis"></a>Síntesis manual

El diccionario [`Translations`][translations] se puede usar para sintetizar el audio desde el texto de traducción. Recorra en iteración cada traducción y sintetice la traducción. Al crear una instancia de `SpeechSynthesizer`, el objeto `SpeechConfig` debe tener su propiedad [`SetSpeechSynthesisVoiceName`][speechsynthesisvoicename] establecida en la voz deseada. El siguiente ejemplo se traduce a cinco idiomas y, a continuación, cada conversión se sintetiza en un archivo de audio en el idioma neuronal correspondiente.

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    auto fromLanguage = "en-US";
    auto toLanguages = { "de", "en", "it", "pt", "zh-Hans" };
    translationConfig->SetSpeechRecognitionLanguage(fromLanguage);
    for (auto language : toLanguages) {
        translationConfig->AddTargetLanguage(language);
    }

    auto recognizer = TranslationRecognizer::FromConfig(translationConfig);

    cout << "Say something in '" << fromLanguage << "' and we'll translate...\n";

    auto result = recognizer->RecognizeOnceAsync().get();
    if (result->Reason == ResultReason::TranslatedSpeech)
    {
        // See: https://aka.ms/speech/sdkregion#standard-and-neural-voices
        map<string, string> languageToVoiceMap;
        languageToVoiceMap["de"] = "de-DE-KatjaNeural";
        languageToVoiceMap["en"] = "en-US-AriaNeural";
        languageToVoiceMap["it"] = "it-IT-ElsaNeural";
        languageToVoiceMap["pt"] = "pt-BR-FranciscaNeural";
        languageToVoiceMap["zh-Hans"] = "zh-CN-XiaoxiaoNeural";

        cout << "Recognized: \"" << result->Text << "\"" << std::endl;
        for (auto pair : result->Translations)
        {
            auto language = pair.first;
            auto translation = pair.second;
            cout << "Translated into '" << language << "': " << translation << std::endl;

            auto speech_config =
                SpeechConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
            speech_config->SetSpeechSynthesisVoiceName(languageToVoiceMap[language]);

            auto audio_config = AudioConfig::FromWavFileOutput(language + "-translation.wav");
            auto synthesizer = SpeechSynthesizer::FromConfig(speech_config, audio_config);

            synthesizer->SpeakTextAsync(translation).get();
        }
    }
}
```

Para más información acerca de la síntesis de voz, consulte [Aspectos básicos de la síntesis de voz](../../../get-started-text-to-speech.md).

[config]: /cpp/cognitive-services/speech/translation-speechtranslationconfig
[audioconfig]: /cpp/cognitive-services/speech/audio-audioconfig
[recognizer]: /cpp/cognitive-services/speech/translation-translationrecognizer
[recognitionlang]: /cpp/cognitive-services/speech/speechconfig#setspeechrecognitionlanguage
[addlang]: /cpp/cognitive-services/speech/translation-speechtranslationconfig#addtargetlanguage
[translations]: /cpp/cognitive-services/speech/translation-translationrecognitionresult#translations
[voicename]: /cpp/cognitive-services/speech/translation-speechtranslationconfig#setvoicename
[speechsynthesisvoicename]: /cpp/cognitive-services/speech/speechconfig#setspeechsynthesisvoicename