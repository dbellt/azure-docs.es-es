---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/11/2020
ms.author: trbye
ms.openlocfilehash: 25a308f3a162e675b4d0ae3f7a706f6b85a5ce57
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110164514"
---
Una de las características principales del servicio de voz es la capacidad para reconocer y transcribir la voz humana (que a menudo se denomina "conversión de voz en texto"). En este inicio rápido, aprenderá a usar el SDK de voz en sus aplicaciones y productos para realizar una conversión de voz en texto de alta calidad.

## <a name="skip-to-samples-on-github"></a>Pasar a los ejemplos en GitHub

Si quiere pasar directamente al código de ejemplo, consulte los [ejemplos del inicio rápido de Python](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/python) en GitHub.

## <a name="prerequisites"></a>Prerrequisitos

En este artículo se da por hecho que:

* tiene una cuenta de Azure y una suscripción al servicio de voz. Si no dispone de una cuenta y una suscripción, [pruebe el servicio de voz de forma gratuita](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-and-import-the-speech-sdk"></a>Instalación e importación de Speech SDK

En primer lugar, deberá instalar Speech SDK.

```Python
pip install azure-cognitiveservices-speech
```

Si trabaja en macOS y tiene problemas de instalación, puede que tenga que ejecutar este comando primero.

```Python
python3 -m pip install --upgrade pip
```

Una vez instalado el SDK de Voz, impórtelo en el proyecto de Python.

```Python
import azure.cognitiveservices.speech as speechsdk
```

## <a name="create-a-speech-configuration"></a>Creación de una configuración de voz

Para llamar al servicio de voz con Speech SDK, debe crear un elemento [`SpeechConfig`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig). Esta clase incluye información sobre la suscripción, como la clave, la región asociada, el punto de conexión, el host o el token de autorización. Cree una clase [`SpeechConfig`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig) mediante su clave y región. Consulte la página [Búsqueda de las claves y la región](../../../overview.md#find-keys-and-region) para encontrar el par clave-región.

```Python
speech_config = speechsdk.SpeechConfig(subscription="<paste-your-subscription-key>", region="<paste-your-region>")
```

Existen otras maneras de inicializar una clase [`SpeechConfig`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig):

* Con un punto de conexión: pase un punto de conexión del servicio de voz. La clave y el token de autorización son opcionales.
* Con un host: pase una dirección de host. La clave y el token de autorización son opcionales.
* Con un token de autorización: pase el token de autorización y la región asociada.

> [!NOTE]
> Debe crear siempre una configuración, independientemente de si va a realizar reconocimiento de voz, síntesis de voz, traducción o reconocimiento de intenciones.

## <a name="recognize-from-microphone"></a>Reconocimiento desde un micrófono

Para reconocer la voz desde el micrófono del dispositivo, cree una clase `SpeechRecognizer` sin pasar `AudioConfig`, y pase `speech_config`.

```Python
import azure.cognitiveservices.speech as speechsdk

def from_mic():
    speech_config = speechsdk.SpeechConfig(subscription="<paste-your-subscription-key>", region="<paste-your-region>")
    speech_recognizer = speechsdk.SpeechRecognizer(speech_config=speech_config)
    
    print("Speak into your microphone.")
    result = speech_recognizer.recognize_once_async().get()
    print(result.text)

from_mic()
```

Si desea usar un dispositivo de entrada de audio *específico*, debe especificar el identificador de dispositivo en una clase `AudioConfig` y pasarlo al parámetro `audio_config` del constructor `SpeechRecognizer`. Obtenga información sobre [cómo obtener el identificador del dispositivo](../../../how-to-select-audio-input-devices.md) de entrada de audio.

## <a name="recognize-from-file"></a>Reconocimiento desde un archivo

Si desea reconocer la voz de un archivo de audio, en lugar de usar un micrófono, cree un elemento [`AudioConfig`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.audio.audioconfig) y use el parámetro `filename`.

```Python
import azure.cognitiveservices.speech as speechsdk

def from_file():
    speech_config = speechsdk.SpeechConfig(subscription="<paste-your-subscription-key>", region="<paste-your-region>")
    audio_input = speechsdk.AudioConfig(filename="your_file_name.wav")
    speech_recognizer = speechsdk.SpeechRecognizer(speech_config=speech_config, audio_config=audio_input)
    
    result = speech_recognizer.recognize_once_async().get()
    print(result.text)

from_file()
```

## <a name="error-handling"></a>Control de errores

Los ejemplos anteriores sencillamente obtienen el texto reconocido de `result.text`, pero para gestionar los errores y otras respuestas, deberá escribir código para determinar el resultado. En el código siguiente se evalúa la propiedad [`result.reason`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.resultreason) y:

* Imprime el resultado del reconocimiento: `speechsdk.ResultReason.RecognizedSpeech`.
* Si no hay ninguna coincidencia de reconocimiento, se informa al usuario: `speechsdk.ResultReason.NoMatch `.
* Si se produce un error, se imprime el mensaje de error: `speechsdk.ResultReason.Canceled`.

```Python
if result.reason == speechsdk.ResultReason.RecognizedSpeech:
    print("Recognized: {}".format(result.text))
elif result.reason == speechsdk.ResultReason.NoMatch:
    print("No speech could be recognized: {}".format(result.no_match_details))
elif result.reason == speechsdk.ResultReason.Canceled:
    cancellation_details = result.cancellation_details
    print("Speech Recognition canceled: {}".format(cancellation_details.reason))
    if cancellation_details.reason == speechsdk.CancellationReason.Error:
        print("Error details: {}".format(cancellation_details.error_details))
```

## <a name="continuous-recognition"></a>Reconocimiento continuo

En los ejemplos anteriores se usa el reconocimiento de una sola captura, que reconoce una única expresión. El final de una expresión única se determina mediante la escucha de un silencio al final o hasta que se procesa un máximo de 15 segundos de audio.

En cambio, el reconocimiento continuo se usa para **establecer** cuándo cesar el reconocimiento. Requiere que se conecte a `EventSignal` para obtener los resultados del reconocimiento y, para detener el reconocimiento, debe llamar a [stop_continuous_recognition()](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer#stop-continuous-recognition--) o [stop_continuous_recognition()](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer#stop-continuous-recognition-async--). Este es un ejemplo de cómo se realiza el reconocimiento continuo en un archivo de entrada de audio.

Comencemos por definir la entrada e inicializar un elemento [`SpeechRecognizer`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechrecognizer):

```Python
audio_config = speechsdk.audio.AudioConfig(filename=weatherfilename)
speech_recognizer = speechsdk.SpeechRecognizer(speech_config=speech_config, audio_config=audio_config)
```

A continuación, vamos a crear una variable para administrar el estado del reconocimiento de voz. Para empezar, lo estableceremos en `False`, ya que al inicio del reconocimiento podemos suponer que no ha terminado.

```Python
done = False
```

Ahora, vamos a crear una devolución de llamada para detener el reconocimiento continuo cuando se reciba un elemento `evt`. Hay algunos aspectos que se deben tener en cuenta.

* Cuando se recibe un elemento `evt`, se imprime el mensaje del elemento `evt`.
* Una vez recibido un elemento `evt`, se llama a [stop_continuous_recognition()](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer#stop-continuous-recognition--) para detener el reconocimiento.
* El estado del reconocimiento se cambia a `True`.

```Python
def stop_cb(evt):
    print('CLOSING on {}'.format(evt))
    speech_recognizer.stop_continuous_recognition()
    nonlocal done
    done = True
```

En este ejemplo de código se muestra cómo conectar devoluciones de llamada a eventos enviados desde el elemento [`SpeechRecognizer`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer#start-continuous-recognition--).

* [`recognizing`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer#recognizing): se señalizan los eventos que contienen los resultados intermedios del reconocimiento.
* [`recognized`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer#recognized): se señalizan los eventos que contienen los resultados finales del reconocimiento (lo que indica un intento de reconocimiento correcto).
* [`session_started`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer#session-started): se señalizan los eventos que indican el inicio de una sesión de reconocimiento (operación).
* [`session_stopped`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer#session-stopped): se señalizan los eventos que indican el final de una sesión de reconocimiento (operación).
* [`canceled`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer#canceled): se señalizan los eventos que contienen los resultados de reconocimiento cancelados (que indican un intento de reconocimiento que se canceló como resultado de una solicitud de cancelación directa o, de forma alternativa, por un error de protocolo o transporte).

```Python
speech_recognizer.recognizing.connect(lambda evt: print('RECOGNIZING: {}'.format(evt)))
speech_recognizer.recognized.connect(lambda evt: print('RECOGNIZED: {}'.format(evt)))
speech_recognizer.session_started.connect(lambda evt: print('SESSION STARTED: {}'.format(evt)))
speech_recognizer.session_stopped.connect(lambda evt: print('SESSION STOPPED {}'.format(evt)))
speech_recognizer.canceled.connect(lambda evt: print('CANCELED {}'.format(evt)))

speech_recognizer.session_stopped.connect(stop_cb)
speech_recognizer.canceled.connect(stop_cb)
```

Con todo configurado, podemos llamar a [start_continuous_recognition()](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer#session-stopped).

```Python
speech_recognizer.start_continuous_recognition()
while not done:
    time.sleep(.5)
```

### <a name="dictation-mode"></a>Modo de dictado

Al usar el reconocimiento continuo, puede habilitar el procesamiento de dictado mediante la función "habilitar dictado" correspondiente. Este modo hará que la instancia de configuración de voz interprete las descripciones de palabras de estructuras de oraciones como puntuación. Por ejemplo, la expresión "Interrogación de apertura vive en la ciudad interrogación de cierre" se interpretaría como el texto "¿Vive en la ciudad?".

Para habilitar el modo de dictado, use el método [`enable_dictation()`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig#enable-dictation--) del elemento [`SpeechConfig`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig).

```Python 
SpeechConfig.enable_dictation()
```

## <a name="change-source-language"></a>Cambio del idioma de origen

Una tarea común en el reconocimiento de voz es especificar el idioma de entrada (u origen). Echemos un vistazo a cómo se cambiaría el idioma de entrada a alemán. En el código, busque SpeechConfig y, a continuación, agregue esta línea directamente debajo.

```Python
speech_config.speech_recognition_language="de-DE"
```

[`speech_recognition_language`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig#speech-recognition-language) es un parámetro que toma una cadena como argumento. Puede proporcionar cualquier valor de la lista de [idiomas o configuraciones regionales](../../../language-support.md) compatibles.

## <a name="improve-recognition-accuracy"></a>Mejora de la precisión del reconocimiento

Las listas de frases se usan para identificar frases conocidas en datos de audio, como el nombre de una persona o una ubicación específica. Al proporcionar una lista de frases, mejorará la precisión del reconocimiento de voz.

Por ejemplo, si tiene el comando "Mover a" y "Cerca" como posible destino que se puede decir, puede añadir la entrada "Mover a Cerca". Al agregar una frase, aumentará la probabilidad de que, cuando se reconozca el audio, se reconozca "Mover a Cerca" en lugar de "Mover acerca".

A una lista de frases se pueden agregar palabras solas o frases completas. Durante el reconocimiento, se usa una entrada de una lista de frases para mejorar el reconocimiento de las palabras y frases de la lista incluso cuando las entradas aparecen en medio de la expresión. 

> [!IMPORTANT]
> La característica de lista de frases está disponible en los siguientes idiomas: en-US, de-DE, en-AU, en-CA, en-GB, en-IN, es-ES, fr-FR, it-IT, ja-JP, pt-BR, zh-CN
>
> Para otras configuraciones regionales y si tiene una gran cantidad de frases, [entrenar un modelo personalizado](../../../custom-speech-overview.md) probablemente será la mejor opción para mejorar la precisión.
>
> No use la característica Lista de frases con los puntos de conexión personalizados. En su lugar, entrene un modelo personalizado que incluya las frases.

Para usar una lista de frases, primero debe crear un objeto [`PhraseListGrammar`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.phraselistgrammar) y, a continuación, agregar palabras y frases específicas con [`addPhrase`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.phraselistgrammar#addphrase-phrase--str-).

Los cambios realizados en el objeto [`PhraseListGrammar`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.phraselistgrammar) se aplican en el siguiente reconocimiento o después de una reconexión al servicio de voz.

```Python
phrase_list_grammar = speechsdk.PhraseListGrammar.from_recognizer(reco)
phrase_list_grammar.addPhrase("Supercalifragilisticexpialidocious")
```

Si necesita borrar la lista de frases: 

```Python
phrase_list_grammar.clear()
```

### <a name="other-options-to-improve-recognition-accuracy"></a>Otras opciones para mejorar la precisión del reconocimiento

Las listas de frases son solo una opción para mejorar la precisión del reconocimiento. También puede: 

* [Mejora de la precisión con Habla personalizada](../../../custom-speech-overview.md)
* [Mejora de la precisión con modelos de inquilino](../../../tutorial-tenant-model.md)
