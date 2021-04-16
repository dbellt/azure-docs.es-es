---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 04/13/2020
ms.author: trbye
ms.openlocfilehash: bea61ac9a8c5700cd4f18cfb567b054de6b5613d
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105105287"
---
Una de las características principales del servicio de voz es la capacidad para reconocer la voz humana y traducirla a otros idiomas. En este inicio rápido, aprenderá a usar Speech SDK en sus aplicaciones y productos para realizar la traducción de voz de alta calidad. En este inicio rápido se tratan temas que incluyen:

* Traducción de voz a texto
* Traducción de voz a varios idiomas de destino
* Realizar la traducción de voz a voz directa

## <a name="skip-to-samples-on-github"></a>Pasar a los ejemplos en GitHub

Si quiere pasar directamente al código de ejemplo, consulte los [ejemplos del inicio rápido de Python](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/python) en GitHub.

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por sentado que tiene una cuenta de Azure y una suscripción al servicio de voz. Si no dispone de una cuenta y una suscripción, [pruebe el servicio de voz de forma gratuita](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-the-speech-sdk"></a>Instalación de Speech SDK

En primer lugar, deberá instalar Speech SDK. Dependiendo de la plataforma, siga las instrucciones de la sección <a href="/azure/cognitive-services/speech-service/speech-sdk#get-the-speech-sdk" target="_blank">Obtención del SDK de Voz</a> del artículo _Acerca del SDK de Voz_.

## <a name="import-dependencies"></a>Dependencias de importación

Para ejecutar los ejemplos de este artículo, incluya las siguientes instrucciones `import` al principio del archivo de código Python.

```python
import os
import azure.cognitiveservices.speech as speechsdk
```

## <a name="sensitive-data-and-environment-variables"></a>Datos confidenciales y variables de entorno

El código fuente de ejemplo de este artículo depende de las variables de entorno para almacenar datos confidenciales, como la región y la clave de suscripción del recurso de Voz. El archivo de código Python contiene dos valores que se asignan a las variables de entorno de las máquinas host, es decir, `SPEECH__SUBSCRIPTION__KEY` y `SPEECH__SERVICE__REGION`. Ambas variables se encuentran en el ámbito global, lo que las hace accesibles dentro de la definición de función del archivo de código. Para más información sobre las variables de entorno, consulte [Configuración de las variables de entorno y de la aplicación](../../../../cognitive-services-security.md#environment-variables-and-application-configuration).

```python
speech_key, service_region = os.environ['SPEECH__SUBSCRIPTION__KEY'], os.environ['SPEECH__SERVICE__REGION']
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

```python
from_language, to_language = 'en-US', 'de'

def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)
```

## <a name="change-source-language"></a>Cambio del idioma de origen

Una tarea común en la traducción de voz es especificar el idioma de entrada (u origen). Echemos un vistazo a cómo se cambiaría el idioma de entrada a italiano. En el código, interactúe con la instancia [`SpeechTranslationConfig`][config], asignándola a la propiedad `speech_recognition_language`.

```python
def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    # Source (input) language
    translation_config.speech_recognition_language = from_language
```

La propiedad [`speech_recognition_language`][recognitionlang] espera una cadena con formato de configuración regional de idioma. En la columna **Locale**, puede proporcionar cualquier valor de la lista de idiomas o configuraciones regionales [compatibles](../../../language-support.md).

## <a name="add-translation-language"></a>Incorporación del idioma de traducción

Otra tarea común de la traducción de voz es especificar los idiomas de traducción de destino; se requiere al menos uno, pero se admiten varios. En el fragmento de código siguiente, se han especificado el francés y el alemán como destinos del idioma de traducción.

```python
def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = "it-IT"

    # Translate to languages. See, https://aka.ms/speech/sttt-languages
    translation_config.add_target_language("fr")
    translation_config.add_target_language("de")
```

Con cada llamada a [`add_target_language`][addlang], se especifica un nuevo idioma de traducción de destino. En otras palabras, cuando se reconoce la voz del lenguaje de origen, cada traducción de destino está disponible como parte de la operación de traducción resultante.

## <a name="initialize-a-translation-recognizer"></a>Inicialización de un reconocedor de traducción

Una vez creado un elemento [`SpeechTranslationConfig`][config], el paso siguiente consiste en inicializar un elemento [`TranslationRecognizer`][recognizer]. Al inicializar un elemento [`TranslationRecognizer`][recognizer], deberá pasar el elemento `translation_config`. El objeto de configuración proporciona las credenciales que necesita el servicio de voz para validar la solicitud.

Si va a realizar un reconocimiento de voz con el micrófono predeterminado del dispositivo, el elemento [`TranslationRecognizer`][recognizer] debería ser similar a:

```python
def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = from_language
    translation_config.add_target_language(to_language)

    recognizer = speechsdk.translation.TranslationRecognizer(
            translation_config=translation_config)
```

Si desea especificar el dispositivo de entrada de audio, deberá crear un elemento [`AudioConfig`][audioconfig] y proporcionar el parámetro `audio_config` al inicializar el elemento [`TranslationRecognizer`][recognizer].

> [!TIP]
> [Obtenga información sobre cómo obtener el identificador de dispositivo del dispositivo de entrada de audio](../../../how-to-select-audio-input-devices.md).

En primer lugar, haga referencia al objeto `AudioConfig` como se indica a continuación:

```python
def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = from_language
    for lang in to_languages:
        translation_config.add_target_language(lang)

    audio_config = speechsdk.audio.AudioConfig(use_default_microphone=True)
    recognizer = speechsdk.translation.TranslationRecognizer(
            translation_config=translation_config, audio_config=audio_config)
```

Si desea proporcionar un archivo de audio en lugar de usar un micrófono, deberá proporcionar un elemento `audioConfig`. Sin embargo, cuando crea una [`AudioConfig`][audioconfig], en lugar de hacer una llamada con `use_default_microphone=True`, la hará con `filename="path-to-file.wav"` y proporcionará el parámetro `filename`.

```python
def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = from_language
    for lang in to_languages:
        translation_config.add_target_language(lang)

    audio_config = speechsdk.audio.AudioConfig(filename="path-to-file.wav")
    recognizer = speechsdk.translation.TranslationRecognizer(
            translation_config=translation_config, audio_config=audio_config)
```

## <a name="translate-speech"></a>Traducir voz

Para traducir la voz, el SDK de Voz depende de un micrófono o en una entrada de archivo de audio. El reconocimiento de voz tiene lugar antes de la traducción de voz. Cuando todos los objetos se hayan inicializado, llame a la función recognize-once y obtenga el resultado.

```python
import os
import azure.cognitiveservices.speech as speechsdk

speech_key, service_region = os.environ['SPEECH__SERVICE__KEY'], os.environ['SPEECH__SERVICE__REGION']
from_language, to_languages = 'en-US', 'de'

def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = from_language
    translation_config.add_target_language(to_language)

    recognizer = speechsdk.translation.TranslationRecognizer(
            translation_config=translation_config)
    
    print('Say something...')
    result = recognizer.recognize_once()
    print(get_result_text(reason=result.reason, result=result))

def get_result_text(reason, result):
    reason_format = {
        speechsdk.ResultReason.TranslatedSpeech:
            f'RECOGNIZED "{from_language}": {result.text}\n' +
            f'TRANSLATED into "{to_language}"": {result.translations[to_language]}',
        speechsdk.ResultReason.RecognizedSpeech: f'Recognized: "{result.text}"',
        speechsdk.ResultReason.NoMatch: f'No speech could be recognized: {result.no_match_details}',
        speechsdk.ResultReason.Canceled: f'Speech Recognition canceled: {result.cancellation_details}'
    }
    return reason_format.get(reason, 'Unable to recognize speech')

translate_speech_to_text()
```

Para más información acerca de la conversión de voz a texto, consulte [Aspectos básicos del reconocimiento de voz](../../../get-started-speech-to-text.md).

## <a name="synthesize-translations"></a>Síntesis de traducciones

Después de un reconocimiento de voz y una traducción correctos, el resultado contiene todas las traducciones de un diccionario. La clave del diccionario [`translations`][translations] es el idioma de traducción de destino y el valor es el texto traducido. La voz reconocida se puede traducir y sintetizar en otro idioma (de voz a voz).

### <a name="event-based-synthesis"></a>Síntesis basada en eventos

El objeto `TranslationRecognizer` expone un evento `Synthesizing`. El evento se activa varias veces y proporciona un mecanismo para recuperar el audio sintetizado del resultado del reconocimiento de la traducción. Si va a traducir a varios idiomas, consulte [Síntesis manual](#manual-synthesis). Especifique la voz de síntesis asignando un [`voice_name`][voicename] y proporcione un controlador de eventos para el evento de `Synthesizing` y obtenga el audio. En el ejemplo siguiente se guarda el audio traducido como un archivo *.wav*.

> [!IMPORTANT]
> La síntesis basada en eventos solo funciona con una sola traducción, **no** agrega varios idiomas de traducción de destino. Además, el objeto [`voice_name`][voicename] debe ser el mismo idioma que el idioma de traducción de destino; por ejemplo, `"de"` podría asignarse a `"de-DE-Hedda"`.

```python
import os
import azure.cognitiveservices.speech as speechsdk

speech_key, service_region = os.environ['SPEECH__SERVICE__KEY'], os.environ['SPEECH__SERVICE__REGION']
from_language, to_language = 'en-US', 'de'

def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = from_language
    translation_config.add_target_language(to_language)

    # See: https://aka.ms/speech/sdkregion#standard-and-neural-voices
    translation_config.voice_name = "de-DE-Hedda"

    recognizer = speechsdk.translation.TranslationRecognizer(
            translation_config=translation_config)

    def synthesis_callback(evt):
        size = len(evt.result.audio)
        print(f'Audio synthesized: {size} byte(s) {"(COMPLETED)" if size == 0 else ""}')

        if size > 0:
            file = open('translation.wav', 'wb+')
            file.write(evt.result.audio)
            file.close()

    recognizer.synthesizing.connect(synthesis_callback)

    print(f'Say something in "{from_language}" and we\'ll translate into "{to_language}".')

    result = recognizer.recognize_once()
    print(get_result_text(reason=result.reason, result=result))

def get_result_text(reason, result):
    reason_format = {
        speechsdk.ResultReason.TranslatedSpeech:
            f'Recognized "{from_language}": {result.text}\n' +
            f'Translated into "{to_language}"": {result.translations[to_language]}',
        speechsdk.ResultReason.RecognizedSpeech: f'Recognized: "{result.text}"',
        speechsdk.ResultReason.NoMatch: f'No speech could be recognized: {result.no_match_details}',
        speechsdk.ResultReason.Canceled: f'Speech Recognition canceled: {result.cancellation_details}'
    }
    return reason_format.get(reason, 'Unable to recognize speech')

translate_speech_to_text()
```

### <a name="manual-synthesis"></a>Síntesis manual

El diccionario [`translations`][translations] se puede usar para sintetizar el audio desde el texto de traducción. Recorra en iteración cada traducción y sintetice la traducción. Al crear una instancia de `SpeechSynthesizer`, el objeto `SpeechConfig` debe tener su propiedad [`speech_synthesis_voice_name`][speechsynthesisvoicename] establecida en la voz deseada. El siguiente ejemplo se traduce a cinco idiomas y, a continuación, cada conversión se sintetiza en un archivo de audio en el idioma neuronal correspondiente.

```python
import os
import azure.cognitiveservices.speech as speechsdk

speech_key, service_region = os.environ['SPEECH__SERVICE__KEY'], os.environ['SPEECH__SERVICE__REGION']
from_language, to_languages = 'en-US', [ 'de', 'en', 'it', 'pt', 'zh-Hans' ]

def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = from_language
    for lang in to_languages:
        translation_config.add_target_language(lang)

    recognizer = speechsdk.translation.TranslationRecognizer(
            translation_config=translation_config)
    
    print('Say something...')
    result = recognizer.recognize_once()
    synthesize_translations(result=result)

def synthesize_translations(result):
    language_to_voice_map = {
        "de": "de-DE-KatjaNeural",
        "en": "en-US-AriaNeural",
        "it": "it-IT-ElsaNeural",
        "pt": "pt-BR-FranciscaNeural",
        "zh-Hans": "zh-CN-XiaoxiaoNeural"
    }
    print(f'Recognized: "{result.text}"')

    for language in result.translations:
        translation = result.translations[language]
        print(f'Translated into "{language}": {translation}')

        speech_config = speechsdk.SpeechConfig(subscription=speech_key, region=service_region)
        speech_config.speech_synthesis_voice_name = language_to_voice_map.get(language)
        
        audio_config = speechsdk.audio.AudioOutputConfig(filename=f'{language}-translation.wav')
        speech_synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config, audio_config=audio_config)
        speech_synthesizer.speak_text_async(translation).get()

translate_speech_to_text()
```

Para más información acerca de la síntesis de voz, consulte [Aspectos básicos de la síntesis de voz](../../../get-started-text-to-speech.md).

[config]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.translation.speechtranslationconfig
[audioconfig]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.audio.audioconfig
[recognizer]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.translation.translationrecognizer
[recognitionlang]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig
[addlang]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.translation.speechtranslationconfig#add-target-language-language--str-
[translations]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.translation.translationrecognitionresult#translations
[voicename]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.translation.speechtranslationconfig#voice-name
[speechsynthesisvoicename]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig#speech-synthesis-voice-name