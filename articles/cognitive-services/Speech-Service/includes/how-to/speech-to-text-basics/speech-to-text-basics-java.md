---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/11/2020
ms.custom: devx-track-java
ms.author: trbye
ms.openlocfilehash: d9ea89d63582a460fcbde2f5568801d41729f5bf
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110163112"
---
Una de las características principales del servicio de voz es la capacidad para reconocer y transcribir la voz humana (que a menudo se denomina "conversión de voz en texto"). En este inicio rápido, aprenderá a usar el SDK de voz en sus aplicaciones y productos para realizar una conversión de voz en texto de alta calidad.

## <a name="skip-to-samples-on-github"></a>Pasar a los ejemplos en GitHub

Si quiere pasar directamente al código de ejemplo, consulte los [ejemplos del inicio rápido de Java](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/java/jre) en GitHub.

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por sentado que tiene una cuenta de Azure y una suscripción al servicio de voz. Si no dispone de una cuenta y una suscripción, [pruebe el servicio de voz de forma gratuita](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-the-speech-sdk"></a>Instalación de Speech SDK

En primer lugar, deberá instalar Speech SDK. Utilice las siguientes instrucciones en función de la plataforma:

* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-java&tabs=jre" target="_blank">Java Runtime </a>
* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-java&tabs=android" target="_blank">Android </a>

## <a name="create-a-speech-configuration"></a>Creación de una configuración de voz

Para llamar al servicio de voz con Speech SDK, debe crear un elemento [`SpeechConfig`](/java/api/com.microsoft.cognitiveservices.speech.speechconfig). Esta clase incluye información sobre la suscripción, como la clave, la región asociada, el punto de conexión, el host o el token de autorización. Cree una clase [`SpeechConfig`](/java/api/com.microsoft.cognitiveservices.speech.speechconfig) mediante su clave y región. Consulte la página [Búsqueda de las claves y la región](../../../overview.md#find-keys-and-region) para encontrar el par clave-región.

```java
import com.microsoft.cognitiveservices.speech.*;
import com.microsoft.cognitiveservices.speech.audio.AudioConfig;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;

public class Program {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        SpeechConfig speechConfig = SpeechConfig.fromSubscription("<paste-your-subscription-key>", "<paste-your-region>");
    }
}
```

Existen otras maneras de inicializar una clase [`SpeechConfig`](/java/api/com.microsoft.cognitiveservices.speech.speechconfig):

* Con un punto de conexión: pase un punto de conexión del servicio de voz. La clave y el token de autorización son opcionales.
* Con un host: pase una dirección de host. La clave y el token de autorización son opcionales.
* Con un token de autorización: pase el token de autorización y la región asociada.

> [!NOTE]
> Debe crear siempre una configuración, independientemente de si va a realizar reconocimiento de voz, síntesis de voz, traducción o reconocimiento de intenciones.

## <a name="recognize-from-microphone"></a>Reconocimiento desde un micrófono

Para reconocer la voz desde el micrófono del dispositivo, cree una clase `AudioConfig` mediante `fromDefaultMicrophoneInput()`. A continuación, inicialice una clase [`SpeechRecognizer`](/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer), y pase `audioConfig` y `config`.

```java
import com.microsoft.cognitiveservices.speech.*;
import com.microsoft.cognitiveservices.speech.audio.AudioConfig;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;

public class Program {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        SpeechConfig speechConfig = SpeechConfig.fromSubscription("<paste-your-subscription-key>", "<paste-your-region>");
        fromMic(speechConfig);
    }

    public static void fromMic(SpeechConfig speechConfig) throws InterruptedException, ExecutionException {
        AudioConfig audioConfig = AudioConfig.fromDefaultMicrophoneInput();
        SpeechRecognizer recognizer = new SpeechRecognizer(speechConfig, audioConfig);

        System.out.println("Speak into your microphone.");
        Future<SpeechRecognitionResult> task = recognizer.recognizeOnceAsync();
        SpeechRecognitionResult result = task.get();
        System.out.println("RECOGNIZED: Text=" + result.getText());
    }
}
```

Si desea usar un dispositivo de entrada de audio *específico*, debe especificar el identificador de dispositivo en la clase `AudioConfig`. Obtenga información sobre [cómo obtener el identificador del dispositivo](../../../how-to-select-audio-input-devices.md) de entrada de audio.

## <a name="recognize-from-file"></a>Reconocimiento desde un archivo

Si desea reconocer la voz de un archivo de audio, en lugar de usar un micrófono, tiene que crear un elemento `AudioConfig`. Sin embargo, al crear [`AudioConfig`](/java/api/com.microsoft.cognitiveservices.speech.audio.audioconfig), en lugar de llamar a `fromDefaultMicrophoneInput()`, llame a `fromWavFileInput()` y pase la ruta de acceso del archivo.

```java
import com.microsoft.cognitiveservices.speech.*;
import com.microsoft.cognitiveservices.speech.audio.AudioConfig;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;

public class Program {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        SpeechConfig speechConfig = SpeechConfig.fromSubscription("<paste-your-subscription-key>", "<paste-your-region>");
        fromFile(speechConfig);
    }

    public static void fromFile(SpeechConfig speechConfig) throws InterruptedException, ExecutionException {
        AudioConfig audioConfig = AudioConfig.fromWavFileInput("YourAudioFile.wav");
        SpeechRecognizer recognizer = new SpeechRecognizer(speechConfig, audioConfig);
        
        Future<SpeechRecognitionResult> task = recognizer.recognizeOnceAsync();
        SpeechRecognitionResult result = task.get();
        System.out.println("RECOGNIZED: Text=" + result.getText());
    }
}
```

## <a name="error-handling"></a>Control de errores

En los ejemplos anteriores se obtiene sencillamente el texto reconocido mediante `result.getText()`; sin embargo, para controlar los errores y otras respuestas, deberá escribir código para determinar el resultado. En el ejemplo siguiente se evalúa [`result.getReason()`](/java/api/com.microsoft.cognitiveservices.speech.recognitionresult.getreason) y se:

* Imprime el resultado del reconocimiento: `ResultReason.RecognizedSpeech`.
* Si no hay ninguna coincidencia de reconocimiento, se informa al usuario: `ResultReason.NoMatch`.
* Si se produce un error, se imprime el mensaje de error: `ResultReason.Canceled`.

```java
switch (result.getReason()) {
    case ResultReason.RecognizedSpeech:
        System.out.println("We recognized: " + result.getText());
        exitCode = 0;
        break;
    case ResultReason.NoMatch:
        System.out.println("NOMATCH: Speech could not be recognized.");
        break;
    case ResultReason.Canceled: {
            CancellationDetails cancellation = CancellationDetails.fromResult(result);
            System.out.println("CANCELED: Reason=" + cancellation.getReason());

            if (cancellation.getReason() == CancellationReason.Error) {
                System.out.println("CANCELED: ErrorCode=" + cancellation.getErrorCode());
                System.out.println("CANCELED: ErrorDetails=" + cancellation.getErrorDetails());
                System.out.println("CANCELED: Did you update the subscription info?");
            }
        }
        break;
}
```

## <a name="continuous-recognition"></a>Reconocimiento continuo

En los ejemplos anteriores se usa el reconocimiento de una sola captura, que reconoce una única expresión. El final de una expresión única se determina mediante la escucha de un silencio al final o hasta que se procesa un máximo de 15 segundos de audio.

En cambio, el reconocimiento continuo se usa para **establecer** cuándo cesar el reconocimiento. Requiere que se suscriba a los eventos `recognizing`, `recognized` y `canceled` para obtener los resultados del reconocimiento. Para detener el reconocimiento, debe llamar a [`stopContinuousRecognitionAsync`](/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer.stopcontinuousrecognitionasync). Este es un ejemplo de cómo se realiza el reconocimiento continuo en un archivo de entrada de audio.

Comencemos por definir la entrada e inicializar un elemento [`SpeechRecognizer`](/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer):

```java
AudioConfig audioConfig = AudioConfig.fromWavFileInput("YourAudioFile.wav");
SpeechRecognizer recognizer = new SpeechRecognizer(config, audioConfig);
```

A continuación, vamos a crear una variable para administrar el estado del reconocimiento de voz. Para empezar, declararemos un elemento `Semaphore` en el ámbito de clase.

```java
private static Semaphore stopTranslationWithFileSemaphore;
```

Nos suscribiremos a los eventos enviados desde el elemento [`SpeechRecognizer`](/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer).

* [`recognizing`](/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer.recognizing): se señalizan los eventos que contienen los resultados intermedios del reconocimiento.
* [`recognized`](/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer.recognized): se señalizan los eventos que contienen los resultados finales del reconocimiento (lo que indica un intento de reconocimiento correcto).
* [`sessionStopped`](/java/api/com.microsoft.cognitiveservices.speech.recognizer.sessionstopped): se señalizan los eventos que indican el final de una sesión de reconocimiento (operación).
* [`canceled`](/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer.canceled): se señalizan los eventos que contienen los resultados de reconocimiento cancelados (que indican un intento de reconocimiento que se canceló como resultado de una solicitud de cancelación directa o, de forma alternativa, por un error de protocolo o transporte).

```java
// First initialize the semaphore.
stopTranslationWithFileSemaphore = new Semaphore(0);

recognizer.recognizing.addEventListener((s, e) -> {
    System.out.println("RECOGNIZING: Text=" + e.getResult().getText());
});

recognizer.recognized.addEventListener((s, e) -> {
    if (e.getResult().getReason() == ResultReason.RecognizedSpeech) {
        System.out.println("RECOGNIZED: Text=" + e.getResult().getText());
    }
    else if (e.getResult().getReason() == ResultReason.NoMatch) {
        System.out.println("NOMATCH: Speech could not be recognized.");
    }
});

recognizer.canceled.addEventListener((s, e) -> {
    System.out.println("CANCELED: Reason=" + e.getReason());

    if (e.getReason() == CancellationReason.Error) {
        System.out.println("CANCELED: ErrorCode=" + e.getErrorCode());
        System.out.println("CANCELED: ErrorDetails=" + e.getErrorDetails());
        System.out.println("CANCELED: Did you update the subscription info?");
    }

    stopTranslationWithFileSemaphore.release();
});

recognizer.sessionStopped.addEventListener((s, e) -> {
    System.out.println("\n    Session stopped event.");
    stopTranslationWithFileSemaphore.release();
});
```

Con todo configurado, podemos llamar a [`startContinuousRecognitionAsync`](/java/api/com.microsoft.cognitiveservices.speech.speechrecognizer.startcontinuousrecognitionasync).

```java
// Starts continuous recognition. Uses StopContinuousRecognitionAsync() to stop recognition.
recognizer.startContinuousRecognitionAsync().get();

// Waits for completion.
stopTranslationWithFileSemaphore.acquire();

// Stops recognition.
recognizer.stopContinuousRecognitionAsync().get();
```

### <a name="dictation-mode"></a>Modo de dictado

Al usar el reconocimiento continuo, puede habilitar el procesamiento de dictado mediante la función "habilitar dictado" correspondiente. Este modo hará que la instancia de configuración de voz interprete las descripciones de palabras de estructuras de oraciones como puntuación. Por ejemplo, la expresión "Interrogación de apertura vive en la ciudad interrogación de cierre" se interpretaría como el texto "¿Vive en la ciudad?".

Para habilitar el modo de dictado, use el método [`enableDictation`](/java/api/com.microsoft.cognitiveservices.speech.speechconfig.enabledictation) del elemento [`SpeechConfig`](/java/api/com.microsoft.cognitiveservices.speech.speechconfig).

```java
config.enableDictation();
```

## <a name="change-source-language"></a>Cambio del idioma de origen

Una tarea común en el reconocimiento de voz es especificar el idioma de entrada (u origen). Echemos un vistazo a cómo se cambiaría el idioma de entrada a francés. En el código, busque [`SpeechConfig`](/java/api/com.microsoft.cognitiveservices.speech.speechconfig) y, a continuación, agregue esta línea directamente debajo.

```java
config.setSpeechRecognitionLanguage("fr-FR");
```

[`setSpeechRecognitionLanguage`](/java/api/com.microsoft.cognitiveservices.speech.speechconfig.setspeechrecognitionlanguage) es un parámetro que toma una cadena como argumento. Puede proporcionar cualquier valor de la lista de [idiomas o configuraciones regionales](../../../language-support.md) compatibles.

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

Para usar una lista de frases, primero debe crear un objeto [`PhraseListGrammar`](/java/api/com.microsoft.cognitiveservices.speech.phraselistgrammar) y, a continuación, agregar palabras y frases específicas con [`AddPhrase`](/java/api/com.microsoft.cognitiveservices.speech.phraselistgrammar.addphrase#com_microsoft_cognitiveservices_speech_PhraseListGrammar_addPhrase_String_).

Los cambios realizados en el objeto [`PhraseListGrammar`](/java/api/com.microsoft.cognitiveservices.speech.phraselistgrammar) se aplican en el siguiente reconocimiento o después de una reconexión al servicio de voz.

```java
PhraseListGrammar phraseList = PhraseListGrammar.fromRecognizer(recognizer);
phraseList.addPhrase("Supercalifragilisticexpialidocious");
```

Si necesita borrar la lista de frases: 

```java
phraseList.clear();
```

### <a name="other-options-to-improve-recognition-accuracy"></a>Otras opciones para mejorar la precisión del reconocimiento

Las listas de frases son solo una opción para mejorar la precisión del reconocimiento. También puede: 

* [Mejora de la precisión con Habla personalizada](../../../custom-speech-overview.md)
* [Mejora de la precisión con modelos de inquilino](../../../tutorial-tenant-model.md)