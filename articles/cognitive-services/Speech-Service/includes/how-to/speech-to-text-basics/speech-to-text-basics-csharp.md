---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/11/2020
ms.author: trbye
ms.custom: devx-track-csharp
ms.openlocfilehash: 9fecf4e32da13964f8c27dc559c78921b22bcb4c
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110166294"
---
Una de las características principales del servicio de voz es la capacidad para reconocer y transcribir la voz humana (que a menudo se denomina "conversión de voz en texto"). En este inicio rápido, aprenderá a usar el SDK de voz en sus aplicaciones y productos para realizar una conversión de voz en texto de alta calidad.

## <a name="skip-to-samples-on-github"></a>Pasar a los ejemplos en GitHub

Si quiere pasar directamente al código de ejemplo, consulte los [ejemplos del inicio rápido de C#](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/csharp/dotnet) en GitHub.

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por sentado que tiene una cuenta de Azure y una suscripción al servicio de voz. Si no dispone de una cuenta y una suscripción, [pruebe el servicio de voz de forma gratuita](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-the-speech-sdk"></a>Instalación de Speech SDK

Si solo desea el nombre del paquete para empezar, ejecute `Install-Package Microsoft.CognitiveServices.Speech` en la consola de NuGet.

Para obtener instrucciones de instalación específicas de la plataforma, consulte los vínculos siguientes:

* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-csharp&tabs=dotnet" target="_blank">.NET Framework </a>
* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-csharp&tabs=dotnetcore" target="_blank">.NET Core </a>
* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-csharp&tabs=unity" target="_blank">Unity </a>
* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-csharp&tabs=uwps" target="_blank">UWP </a>
* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-csharp&tabs=xaml" target="_blank">Xamarin </a>

## <a name="create-a-speech-configuration"></a>Creación de una configuración de voz

Para llamar al servicio de voz con Speech SDK, debe crear un elemento [`SpeechConfig`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig). Esta clase incluye información sobre la suscripción, como la clave, la región asociada, el punto de conexión, el host o el token de autorización. Cree una clase [`SpeechConfig`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig) mediante su clave y región. Consulte la página [Búsqueda de las claves y la región](../../../overview.md#find-keys-and-region) para encontrar el par clave-región.

```csharp
using System;
using System.IO;
using System.Threading.Tasks;
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;

class Program 
{
    async static Task Main(string[] args)
    {
        var speechConfig = SpeechConfig.FromSubscription("<paste-your-subscription-key>", "<paste-your-region>");
    }
}
```

Existen otras maneras de inicializar una clase [`SpeechConfig`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig):

* Con un punto de conexión: pase un punto de conexión del servicio de voz. La clave y el token de autorización son opcionales.
* Con un host: pase una dirección de host. La clave y el token de autorización son opcionales.
* Con un token de autorización: pase el token de autorización y la región asociada.

> [!NOTE]
> Debe crear siempre una configuración, independientemente de si va a realizar reconocimiento de voz, síntesis de voz, traducción o reconocimiento de intenciones.

## <a name="recognize-from-microphone"></a>Reconocimiento desde un micrófono

Para reconocer la voz desde el micrófono del dispositivo, cree una clase `AudioConfig` mediante `FromDefaultMicrophoneInput()`. A continuación, inicialice una clase [`SpeechRecognizer`](/dotnet/api/microsoft.cognitiveservices.speech.speechrecognizer), y pase `audioConfig` y `speechConfig`.

```csharp
using System;
using System.IO;
using System.Threading.Tasks;
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;

class Program 
{
    async static Task FromMic(SpeechConfig speechConfig)
    {
        using var audioConfig = AudioConfig.FromDefaultMicrophoneInput();
        using var recognizer = new SpeechRecognizer(speechConfig, audioConfig);

        Console.WriteLine("Speak into your microphone.");
        var result = await recognizer.RecognizeOnceAsync();
        Console.WriteLine($"RECOGNIZED: Text={result.Text}");
    }

    async static Task Main(string[] args)
    {
        var speechConfig = SpeechConfig.FromSubscription("<paste-your-subscription-key>", "<paste-your-region>");
        await FromMic(speechConfig);
    }
}
```

Si desea usar un dispositivo de entrada de audio *específico*, debe especificar el identificador de dispositivo en la clase `AudioConfig`. Obtenga información sobre [cómo obtener el identificador del dispositivo](../../../how-to-select-audio-input-devices.md) de entrada de audio.

## <a name="recognize-from-file"></a>Reconocimiento desde un archivo

Si desea reconocer la voz de un archivo de audio, en lugar de un micrófono, tiene que crear un elemento `AudioConfig`. Sin embargo, al crear [`AudioConfig`](/dotnet/api/microsoft.cognitiveservices.speech.audio.audioconfig), en lugar de llamar a `FromDefaultMicrophoneInput()`, llame a `FromWavFileInput()` y pase la ruta de acceso del archivo.

```csharp
using System;
using System.IO;
using System.Threading.Tasks;
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;

class Program 
{
    async static Task FromFile(SpeechConfig speechConfig)
    {
        using var audioConfig = AudioConfig.FromWavFileInput("PathToFile.wav");
        using var recognizer = new SpeechRecognizer(speechConfig, audioConfig);

        var result = await recognizer.RecognizeOnceAsync();
        Console.WriteLine($"RECOGNIZED: Text={result.Text}");
    }

    async static Task Main(string[] args)
    {
        var speechConfig = SpeechConfig.FromSubscription("<paste-your-subscription-key>", "<paste-your-region>");
        await FromFile(speechConfig);
    }
}
```

## <a name="recognize-from-in-memory-stream"></a>Reconocimiento desde la secuencia en memoria

En muchos casos de uso, es probable que los datos de audio procedan de Blob Storage o que ya estén en memoria como `byte[]` o una estructura similar de datos sin procesar. En el ejemplo siguiente se usa una clase [`PushAudioInputStream`](/dotnet/api/microsoft.cognitiveservices.speech.audio.pushaudioinputstream) para reconocer la voz, que es básicamente una secuencia de memoria abstracta. En el código de ejemplo se realizan las tareas siguientes:

* Escribe los datos de audio sin formato (PCM) en `PushAudioInputStream` mediante la función `Write()`, que acepta `byte[]`.
* Lee un archivo `.wav` mediante `FileReader` con fines de demostración, pero si ya tiene datos de audio en `byte[]`, puede pasar directamente a escribir el contenido en el flujo de entrada.
* El formato predeterminado es PCM mono de 16 bits y 16 kHz. Para personalizar el formato, puede pasar un objeto [`AudioStreamFormat`](/dotnet/api/microsoft.cognitiveservices.speech.audio.audiostreamformat) a `CreatePushStream()` mediante la función estática `AudioStreamFormat.GetWaveFormatPCM(sampleRate, (byte)bitRate, (byte)channels)`.

```csharp
using System;
using System.IO;
using System.Threading.Tasks;
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;

class Program 
{
    async static Task FromStream(SpeechConfig speechConfig)
    {
        var reader = new BinaryReader(File.OpenRead("PathToFile.wav"));
        using var audioInputStream = AudioInputStream.CreatePushStream();
        using var audioConfig = AudioConfig.FromStreamInput(audioInputStream);
        using var recognizer = new SpeechRecognizer(speechConfig, audioConfig);

        byte[] readBytes;
        do
        {
            readBytes = reader.ReadBytes(1024);
            audioInputStream.Write(readBytes, readBytes.Length);
        } while (readBytes.Length > 0);

        var result = await recognizer.RecognizeOnceAsync();
        Console.WriteLine($"RECOGNIZED: Text={result.Text}");
    }

    async static Task Main(string[] args)
    {
        var speechConfig = SpeechConfig.FromSubscription("<paste-your-subscription-key>", "<paste-your-region>");
        await FromStream(speechConfig);
    }
}
```

El uso de una secuencia de extracción como entrada interpreta que los datos de audio son un PCM sin formato; es decir, se omiten los encabezados.
La API seguirá funcionando en determinados casos si no se ha omitido el encabezado, pero para obtener mejores resultados considere la posibilidad de aplicar la lógica para leer los encabezados, de manera que `byte[]` se inicie al *principio de los datos de audio*.

## <a name="error-handling"></a>Control de errores

Los ejemplos anteriores sencillamente obtienen el texto reconocido de `result.text`, pero para gestionar los errores y otras respuestas, deberá escribir código para determinar el resultado. En el código siguiente se evalúa la propiedad [`result.Reason`](/dotnet/api/microsoft.cognitiveservices.speech.recognitionresult.reason) y:

* Imprime el resultado del reconocimiento: `ResultReason.RecognizedSpeech`.
* Si no hay ninguna coincidencia de reconocimiento, se informa al usuario: `ResultReason.NoMatch`.
* Si se produce un error, se imprime el mensaje de error: `ResultReason.Canceled`.

```csharp
switch (result.Reason)
{
    case ResultReason.RecognizedSpeech:
        Console.WriteLine($"RECOGNIZED: Text={result.Text}");
        break;
    case ResultReason.NoMatch:
        Console.WriteLine($"NOMATCH: Speech could not be recognized.");
        break;
    case ResultReason.Canceled:
        var cancellation = CancellationDetails.FromResult(result);
        Console.WriteLine($"CANCELED: Reason={cancellation.Reason}");

        if (cancellation.Reason == CancellationReason.Error)
        {
            Console.WriteLine($"CANCELED: ErrorCode={cancellation.ErrorCode}");
            Console.WriteLine($"CANCELED: ErrorDetails={cancellation.ErrorDetails}");
            Console.WriteLine($"CANCELED: Did you update the subscription info?");
        }
        break;
}
```

## <a name="continuous-recognition"></a>Reconocimiento continuo

En los ejemplos anteriores se usa el reconocimiento de una sola captura, que reconoce una única expresión. El final de una expresión única se determina mediante la escucha de un silencio al final o hasta que se procesa un máximo de 15 segundos de audio.

En cambio, el reconocimiento continuo se usa para **establecer** cuándo cesar el reconocimiento. Requiere que se suscriba a los eventos `Recognizing`, `Recognized` y `Canceled` para obtener los resultados del reconocimiento. Para detener el reconocimiento, debe llamar a [`StopContinuousRecognitionAsync`](/dotnet/api/microsoft.cognitiveservices.speech.speechrecognizer.stopcontinuousrecognitionasync). Este es un ejemplo de cómo se realiza el reconocimiento continuo en un archivo de entrada de audio.

Comience por definir la entrada e inicializar una clase [`SpeechRecognizer`](/dotnet/api/microsoft.cognitiveservices.speech.speechrecognizer):

```csharp
using var audioConfig = AudioConfig.FromWavFileInput("YourAudioFile.wav");
using var recognizer = new SpeechRecognizer(speechConfig, audioConfig);
```

A continuación, cree `TaskCompletionSource<int>` para administrar el estado del reconocimiento de voz.

```csharp
var stopRecognition = new TaskCompletionSource<int>();
```

A continuación, suscríbase a los eventos enviados desde [`SpeechRecognizer`](/dotnet/api/microsoft.cognitiveservices.speech.speechrecognizer).

* [`Recognizing`](/dotnet/api/microsoft.cognitiveservices.speech.speechrecognizer.recognizing): se señalizan los eventos que contienen los resultados intermedios del reconocimiento.
* [`Recognized`](/dotnet/api/microsoft.cognitiveservices.speech.speechrecognizer.recognized): se señalizan los eventos que contienen los resultados finales del reconocimiento (lo que indica un intento de reconocimiento correcto).
* [`SessionStopped`](/dotnet/api/microsoft.cognitiveservices.speech.recognizer.sessionstopped): se señalizan los eventos que indican el final de una sesión de reconocimiento (operación).
* [`Canceled`](/dotnet/api/microsoft.cognitiveservices.speech.speechrecognizer.canceled): se señalizan los eventos que contienen los resultados de reconocimiento cancelados (que indican un intento de reconocimiento que se canceló como resultado de una solicitud de cancelación directa o, de forma alternativa, por un error de protocolo o transporte).

```csharp
recognizer.Recognizing += (s, e) =>
{
    Console.WriteLine($"RECOGNIZING: Text={e.Result.Text}");
};

recognizer.Recognized += (s, e) =>
{
    if (e.Result.Reason == ResultReason.RecognizedSpeech)
    {
        Console.WriteLine($"RECOGNIZED: Text={e.Result.Text}");
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

recognizer.SessionStopped += (s, e) =>
{
    Console.WriteLine("\n    Session stopped event.");
    stopRecognition.TrySetResult(0);
};
```

Cuando todo esté configurado, llame a `StartContinuousRecognitionAsync` para iniciar el reconocimiento.

```csharp
await recognizer.StartContinuousRecognitionAsync();

// Waits for completion. Use Task.WaitAny to keep the task rooted.
Task.WaitAny(new[] { stopRecognition.Task });

// make the following call at some point to stop recognition.
// await recognizer.StopContinuousRecognitionAsync();
```

### <a name="dictation-mode"></a>Modo de dictado

Al usar el reconocimiento continuo, puede habilitar el procesamiento de dictado mediante la función "habilitar dictado" correspondiente. Este modo hará que la instancia de configuración de voz interprete las descripciones de palabras de estructuras de oraciones como puntuación. Por ejemplo, la expresión "Interrogación de apertura vive en la ciudad interrogación de cierre" se interpretaría como el texto "¿Vive en la ciudad?".

Para habilitar el modo de dictado, use el método [`EnableDictation`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig.enabledictation) del elemento [`SpeechConfig`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig).

```csharp
speechConfig.EnableDictation();
```

## <a name="change-source-language"></a>Cambio del idioma de origen

Una tarea común en el reconocimiento de voz es especificar el idioma de entrada (u origen). Echemos un vistazo a cómo se cambiaría el idioma de entrada a italiano. En el código, busque [`SpeechConfig`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig) y, a continuación, agregue esta línea directamente debajo.

```csharp
speechConfig.SpeechRecognitionLanguage = "it-IT";
```

La propiedad [`SpeechRecognitionLanguage`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig.speechrecognitionlanguage) espera una cadena con formato de configuración regional de idioma. En la columna **Locale**, puede proporcionar cualquier valor de la lista de idiomas o configuraciones regionales [compatibles](../../../language-support.md).

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

Para usar una lista de frases, primero debe crear un objeto [`PhraseListGrammar`](/dotnet/api/microsoft.cognitiveservices.speech.phraselistgrammar) y, a continuación, agregar palabras y frases específicas con [`AddPhrase`](/dotnet/api/microsoft.cognitiveservices.speech.phraselistgrammar.addphrase).

Los cambios realizados en el objeto [`PhraseListGrammar`](/dotnet/api/microsoft.cognitiveservices.speech.phraselistgrammar) se aplican en el siguiente reconocimiento o después de una reconexión al servicio de voz.

```csharp
var phraseList = PhraseListGrammar.FromRecognizer(recognizer);
phraseList.AddPhrase("Supercalifragilisticexpialidocious");
```

Si necesita borrar la lista de frases: 

```csharp
phraseList.Clear();
```

### <a name="other-options-to-improve-recognition-accuracy"></a>Otras opciones para mejorar la precisión del reconocimiento

Las listas de frases son solo una opción para mejorar la precisión del reconocimiento. También puede: 

* [Mejora de la precisión con Habla personalizada](../../../custom-speech-overview.md)
* [Mejora de la precisión con modelos de inquilino](../../../tutorial-tenant-model.md)