---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/06/2020
ms.author: trbye
ms.openlocfilehash: 4848f698490f6a79f9f1664b0011e5aec465ed81
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110165485"
---
Una de las características principales del servicio de voz es la capacidad para reconocer y transcribir la voz humana (que a menudo se denomina "conversión de voz en texto"). En este inicio rápido, aprenderá a usar el SDK de voz en sus aplicaciones y productos para realizar una conversión de voz en texto de alta calidad.

## <a name="skip-to-samples-on-github"></a>Pasar a los ejemplos en GitHub

Si quiere pasar directamente al código de ejemplo, consulte los [ejemplos del inicio rápido de C++](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/cpp/windows) en GitHub.

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por sentado que tiene una cuenta de Azure y una suscripción al servicio de voz. Si no dispone de una cuenta y una suscripción, [pruebe el servicio de voz de forma gratuita](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-the-speech-sdk"></a>Instalación de Speech SDK

En primer lugar, deberá instalar Speech SDK. Utilice las siguientes instrucciones en función de la plataforma:

* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-cpp&tabs=linux" target="_blank">Linux </a>
* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-cpp&tabs=macos" target="_blank">macOS </a>
* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-cpp&tabs=windows" target="_blank">Windows </a>

## <a name="create-a-speech-configuration"></a>Creación de una configuración de voz

Para llamar al servicio de voz con Speech SDK, debe crear un elemento [`SpeechConfig`](/cpp/cognitive-services/speech/speechconfig). Esta clase incluye información sobre la suscripción, como la clave, la región asociada, el punto de conexión, el host o el token de autorización. Cree una clase [`SpeechConfig`](/cpp/cognitive-services/speech/speechconfig) mediante su clave y región. Consulte la página [Búsqueda de las claves y la región](../../../overview.md#find-keys-and-region) para encontrar el par clave-región.

```cpp
using namespace std;
using namespace Microsoft::CognitiveServices::Speech;

auto config = SpeechConfig::FromSubscription("<paste-your-subscription-key>", "<paste-your-region>");
```

Existen otras maneras de inicializar una clase [`SpeechConfig`](/cpp/cognitive-services/speech/speechconfig):

* Con un punto de conexión: pase un punto de conexión del servicio de voz. La clave y el token de autorización son opcionales.
* Con un host: pase una dirección de host. La clave y el token de autorización son opcionales.
* Con un token de autorización: pase el token de autorización y la región asociada.

> [!NOTE]
> Debe crear siempre una configuración, independientemente de si va a realizar reconocimiento de voz, síntesis de voz, traducción o reconocimiento de intenciones.

## <a name="recognize-from-microphone"></a>Reconocimiento desde un micrófono

Para reconocer la voz desde el micrófono del dispositivo, cree una clase `AudioConfig` mediante `FromDefaultMicrophoneInput()`. A continuación, inicialice una clase [`SpeechRecognizer`](/cpp/cognitive-services/speech/speechrecognizer), y pase `audioConfig` y `config`.

```cpp
using namespace Microsoft::CognitiveServices::Speech::Audio;

auto audioConfig = AudioConfig::FromDefaultMicrophoneInput();
auto recognizer = SpeechRecognizer::FromConfig(config, audioConfig);

cout << "Speak into your microphone." << std::endl;
auto result = recognizer->RecognizeOnceAsync().get();
cout << "RECOGNIZED: Text=" << result->Text << std::endl;
```

Si desea usar un dispositivo de entrada de audio *específico*, debe especificar el identificador de dispositivo en la clase `AudioConfig`. Obtenga información sobre [cómo obtener el identificador del dispositivo](../../../how-to-select-audio-input-devices.md) de entrada de audio.

## <a name="recognize-from-file"></a>Reconocimiento desde un archivo

Si desea reconocer la voz de un archivo de audio, en lugar de usar un micrófono, tiene que crear un elemento `AudioConfig`. Sin embargo, al crear [`AudioConfig`](/cpp/cognitive-services/speech/audio-audioconfig), en lugar de llamar a `FromDefaultMicrophoneInput()`, llame a `FromWavFileInput()` y pase la ruta de acceso del archivo.

```cpp
using namespace Microsoft::CognitiveServices::Speech::Audio;

auto audioInput = AudioConfig::FromWavFileInput("YourAudioFile.wav");
auto recognizer = SpeechRecognizer::FromConfig(config, audioInput);

auto result = recognizer->RecognizeOnceAsync().get();
cout << "RECOGNIZED: Text=" << result->Text << std::endl;
```

## <a name="recognize-speech"></a>Reconocer la voz

La [clase Recognizer](/cpp/cognitive-services/speech/speechrecognizer) de Speech SDK para C++ expone algunos métodos que puede usar para el reconocimiento de voz.

### <a name="single-shot-recognition"></a>Reconocimiento de una sola captura

El reconocimiento de una sola captura reconoce de forma asincrónica una única expresión. El final de una expresión única se determina mediante la escucha de un silencio al final o hasta que se procesa un máximo de 15 segundos de audio. Este es un ejemplo de reconocimiento asincrónico de una sola captura mediante [`RecognizeOnceAsync`](/cpp/cognitive-services/speech/speechrecognizer#recognizeonceasync):

```cpp
auto result = recognizer->RecognizeOnceAsync().get();
```

Deberá escribir código para controlar el resultado. Este ejemplo evalúa el elemento [`result->Reason`](/cpp/cognitive-services/speech/recognitionresult#reason):

* Imprime el resultado del reconocimiento: `ResultReason::RecognizedSpeech`.
* Si no hay ninguna coincidencia de reconocimiento, se informa al usuario: `ResultReason::NoMatch`.
* Si se produce un error, se imprime el mensaje de error: `ResultReason::Canceled`.

```cpp
switch (result->Reason)
{
    case ResultReason::RecognizedSpeech:
        cout << "We recognized: " << result->Text << std::endl;
        break;
    case ResultReason::NoMatch:
        cout << "NOMATCH: Speech could not be recognized." << std::endl;
        break;
    case ResultReason::Canceled:
        {
            auto cancellation = CancellationDetails::FromResult(result);
            cout << "CANCELED: Reason=" << (int)cancellation->Reason << std::endl;
    
            if (cancellation->Reason == CancellationReason::Error) {
                cout << "CANCELED: ErrorCode= " << (int)cancellation->ErrorCode << std::endl;
                cout << "CANCELED: ErrorDetails=" << cancellation->ErrorDetails << std::endl;
                cout << "CANCELED: Did you update the subscription info?" << std::endl;
            }
        }
        break;
    default:
        break;
}
```

### <a name="continuous-recognition"></a>Reconocimiento continuo

El reconocimiento continuo es un poco más complicado que el reconocimiento de una sola captura. Requiere que se suscriba a los eventos `Recognizing`, `Recognized` y `Canceled` para obtener los resultados del reconocimiento. Para detener el reconocimiento, debe llamar a [StopContinuousRecognitionAsync](/cpp/cognitive-services/speech/speechrecognizer#stopcontinuousrecognitionasync). Este es un ejemplo de cómo se realiza el reconocimiento continuo en un archivo de entrada de audio.

Comencemos por definir la entrada e inicializar un elemento [`SpeechRecognizer`](/cpp/cognitive-services/speech/speechrecognizer):

```cpp
auto audioInput = AudioConfig::FromWavFileInput("YourAudioFile.wav");
auto recognizer = SpeechRecognizer::FromConfig(config, audioInput);
```

A continuación, vamos a crear una variable para administrar el estado del reconocimiento de voz. Para empezar, vamos a declarar un elemento `promise<void>`, ya que al inicio del reconocimiento podemos suponer que no ha terminado.

```cpp
promise<void> recognitionEnd;
```

Nos suscribiremos a los eventos enviados desde el elemento [`SpeechRecognizer`](/cpp/cognitive-services/speech/speechrecognizer).

* [`Recognizing`](/cpp/cognitive-services/speech/asyncrecognizer#recognizing): se señalizan los eventos que contienen los resultados intermedios del reconocimiento.
* [`Recognized`](/cpp/cognitive-services/speech/asyncrecognizer#recognized): se señalizan los eventos que contienen los resultados finales del reconocimiento (lo que indica un intento de reconocimiento correcto).
* [`SessionStopped`](/cpp/cognitive-services/speech/asyncrecognizer#sessionstopped): se señalizan los eventos que indican el final de una sesión de reconocimiento (operación).
* [`Canceled`](/cpp/cognitive-services/speech/asyncrecognizer#canceled): se señalizan los eventos que contienen los resultados de reconocimiento cancelados (que indican un intento de reconocimiento que se canceló como resultado de una solicitud de cancelación directa o, de forma alternativa, por un error de protocolo o transporte).

```cpp
recognizer->Recognizing.Connect([](const SpeechRecognitionEventArgs& e)
    {
        cout << "Recognizing:" << e.Result->Text << std::endl;
    });

recognizer->Recognized.Connect([](const SpeechRecognitionEventArgs& e)
    {
        if (e.Result->Reason == ResultReason::RecognizedSpeech)
        {
            cout << "RECOGNIZED: Text=" << e.Result->Text 
                 << " (text could not be translated)" << std::endl;
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
```

Con todo configurado, podemos llamar a [`StopContinuousRecognitionAsync`](/cpp/cognitive-services/speech/speechrecognizer#startcontinuousrecognitionasync).

```cpp
// Starts continuous recognition. Uses StopContinuousRecognitionAsync() to stop recognition.
recognizer->StartContinuousRecognitionAsync().get();

// Waits for recognition end.
recognitionEnd.get_future().get();

// Stops recognition.
recognizer->StopContinuousRecognitionAsync().get();
```

### <a name="dictation-mode"></a>Modo de dictado

Al usar el reconocimiento continuo, puede habilitar el procesamiento de dictado mediante la función "habilitar dictado" correspondiente. Este modo hará que la instancia de configuración de voz interprete las descripciones de palabras de estructuras de oraciones como puntuación. Por ejemplo, la expresión "Interrogación de apertura vive en la ciudad interrogación de cierre" se interpretaría como el texto "¿Vive en la ciudad?".

Para habilitar el modo de dictado, use el método [`EnableDictation`](/cpp/cognitive-services/speech/speechconfig#enabledictation) del elemento [`SpeechConfig`](/cpp/cognitive-services/speech/speechconfig).

```cpp
config->EnableDictation();
```

## <a name="change-source-language"></a>Cambio del idioma de origen

Una tarea común en el reconocimiento de voz es especificar el idioma de entrada (u origen). Echemos un vistazo a cómo se cambiaría el idioma de entrada a alemán. En el código, busque [`SpeechConfig`](/cpp/cognitive-services/speech/speechconfig) y, a continuación, agregue esta línea directamente debajo.

```cpp
config->SetSpeechRecognitionLanguage("de-DE");
```

[`SetSpeechRecognitionLanguage`](/cpp/cognitive-services/speech/speechconfig#setspeechrecognitionlanguage) es un parámetro que toma una cadena como argumento. Puede proporcionar cualquier valor de la lista de [idiomas o configuraciones regionales](../../../language-support.md) compatibles.

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

Para usar una lista de frases, primero debe crear un objeto [`PhraseListGrammar`](/cpp/cognitive-services/speech/phraselistgrammar) y, a continuación, agregar palabras y frases específicas con [`AddPhrase`](/cpp/cognitive-services/speech/phraselistgrammar#addphrase).

Los cambios realizados en el objeto [`PhraseListGrammar`](/cpp/cognitive-services/speech/phraselistgrammar) se aplican en el siguiente reconocimiento o después de una reconexión al servicio de voz.

```cpp
auto phraseListGrammar = PhraseListGrammar::FromRecognizer(recognizer);
phraseListGrammar->AddPhrase("Supercalifragilisticexpialidocious");
```

Si necesita borrar la lista de frases: 

```cpp
phraseListGrammar->Clear();
```

### <a name="other-options-to-improve-recognition-accuracy"></a>Otras opciones para mejorar la precisión del reconocimiento

Las listas de frases son solo una opción para mejorar la precisión del reconocimiento. También puede: 

* [Mejora de la precisión con Habla personalizada](../../../custom-speech-overview.md)
* [Mejora de la precisión con modelos de inquilino](../../../tutorial-tenant-model.md)