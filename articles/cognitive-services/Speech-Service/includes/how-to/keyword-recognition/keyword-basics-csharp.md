---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 01/04/2021
ms.author: trbye
ms.openlocfilehash: 407906727332f3db8d3d0a6840d0c865c6b33ff7
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105609527"
---
En primer lugar, cargue el archivo del modelo de palabra clave mediante la función estática `FromFile()`, que devuelve un `KeywordRecognitionModel`. Use la ruta de acceso al archivo `.table` que descargó de Speech Studio. Además, debe crear una configuración `AudioConfig` con el micrófono predeterminado y, a continuación, cree una instancia nueva de `KeywordRecognizer` mediante la configuración de audio.

```csharp
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;

var keywordModel = KeywordRecognitionModel.FromFile("your/path/to/Activate_device.table");
using var audioConfig = AudioConfig.FromDefaultMicrophoneInput();
using var keywordRecognizer = new KeywordRecognizer(audioConfig);
```

A continuación, se ejecuta el reconocimiento de palabras clave mediante una llamada a `RecognizeOnceAsync()` en la que se pasa el objeto de modelo. De este modo, se inicia una sesión de reconocimiento de palabras clave que continúa hasta que se reconoce la palabra clave. Por lo tanto, normalmente se usa este modelo de diseño en aplicaciones multiproceso o en casos de uso en los que puede esperar una palabra de activación indefinidamente.

```csharp
KeywordRecognitionResult result = await keywordRecognizer.RecognizeOnceAsync(keywordModel);
```

> [!NOTE]
> El ejemplo que se muestra aquí usa el reconocimiento de palabras clave local, ya que no requiere un objeto `SpeechConfig` para el contexto de autenticación y no se comunica con el servidor back-end. Sin embargo, puede ejecutar el reconocimiento y la verificación de palabras clave [mediante una conexión directa con el back-end](../../../tutorial-voice-enable-your-bot-speech-sdk.md#view-the-source-code-that-enables-keyword).

### <a name="continuous-recognition"></a>Reconocimiento continuo

Otras clases del SDK de voz admiten el reconocimiento continuo (tanto de voz como de intenciones) junto con el reconocimiento de palabras clave. Esto le permite usar el mismo código que usaría normalmente para el reconocimiento continuo, con la posibilidad de hacer referencia a un archivo `.table` en el modelo de palabras clave.

Para la conversión de voz a texto, siga el mismo patrón de diseño que se muestra en el [inicio rápido](../../../get-started-speech-to-text.md?pivots=programming-language-csharp&tabs=script%2cbrowser%2cwindowsinstall#continuous-recognition) para configurar el reconocimiento continuo. A continuación, reemplace la llamada a `recognizer.StartContinuousRecognitionAsync()` por `recognizer.StartKeywordRecognitionAsync(KeywordRecognitionModel)` y pase el objeto `KeywordRecognitionModel`. Para detener el reconocimiento continuo con el reconocimiento de palabras clave, use `recognizer.StopKeywordRecognitionAsync()` en lugar de `recognizer.StopContinuousRecognitionAsync()`.

El reconocimiento de intenciones usa un patrón idéntico con las funciones [`StartKeywordRecognitionAsync`](/dotnet/api/microsoft.cognitiveservices.speech.intent.intentrecognizer.startkeywordrecognitionasync#Microsoft_CognitiveServices_Speech_Intent_IntentRecognizer_StartKeywordRecognitionAsync_Microsoft_CognitiveServices_Speech_KeywordRecognitionModel_) y [`StopKeywordRecognitionAsync`](/dotnet/api/microsoft.cognitiveservices.speech.intent.intentrecognizer.stopkeywordrecognitionasync#Microsoft_CognitiveServices_Speech_Intent_IntentRecognizer_StopKeywordRecognitionAsync).
