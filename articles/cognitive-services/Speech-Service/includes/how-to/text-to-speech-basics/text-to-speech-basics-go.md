---
author: yulin-li
ms.service: cognitive-services
ms.topic: include
ms.date: 05/17/2021
ms.author: yulili
ms.openlocfilehash: 759fca1d323c72fe5e336a3211a9fb68fc1804b9
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110085503"
---
En este inicio rápido aprenderá patrones de diseño comunes para realizar la síntesis de texto a voz mediante el SDK de voz.

## <a name="skip-to-samples-on-github"></a>Pasar a los ejemplos en GitHub

Si quiere pasar directamente al código de ejemplo, consulte los [ejemplos del inicio rápido de Go](https://github.com/microsoft/cognitive-services-speech-sdk-go/tree/master/samples/synthesizer) en GitHub.

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por sentado que tiene una cuenta de Azure y una suscripción al servicio de voz. Si no dispone de una cuenta y una suscripción, [pruebe el servicio de voz de forma gratuita](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-the-speech-sdk"></a>Instalación de Speech SDK


En primer lugar, deberá instalar el [SDK de Voz para Go](../../../quickstarts/setup-platform.md?pivots=programming-language-go&tabs=dotnet%252cwindows%252cjre%252cbrowser).

## <a name="text-to-speech-to-speaker"></a>Texto a voz para altavoz

Use el siguiente ejemplo de código para ejecutar la síntesis de voz para el dispositivo de salida de audio predeterminado.
Reemplace las variables `subscription` y `region` por sus claves de suscripción y región.
La ejecución del script generará el texto de entrada hablado para el altavoz predeterminado.

```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "strings"
    "time"

    "github.com/Microsoft/cognitive-services-speech-sdk-go/audio"
    "github.com/Microsoft/cognitive-services-speech-sdk-go/common"
    "github.com/Microsoft/cognitive-services-speech-sdk-go/speech"
)

func synthesizeStartedHandler(event speech.SpeechSynthesisEventArgs) {
    defer event.Close()
    fmt.Println("Synthesis started.")
}

func synthesizingHandler(event speech.SpeechSynthesisEventArgs) {
    defer event.Close()
    fmt.Printf("Synthesizing, audio chunk size %d.\n", len(event.Result.AudioData))
}

func synthesizedHandler(event speech.SpeechSynthesisEventArgs) {
    defer event.Close()
    fmt.Printf("Synthesized, audio length %d.\n", len(event.Result.AudioData))
}

func cancelledHandler(event speech.SpeechSynthesisEventArgs) {
    defer event.Close()
    fmt.Println("Received a cancellation.")
}

func main() {
    subscription :=  "YOUR_SUBSCRIPTION_KEY"
    region := "YOUR_SUBSCRIPTIONKEY_REGION"

    audioConfig, err := audio.NewAudioConfigFromDefaultSpeakerOutput()
    if err != nil {
        fmt.Println("Got an error: ", err)
        return
    }
    defer audioConfig.Close()
    config, err := speech.NewSpeechConfigFromSubscription(subscription, region)
    if err != nil {
        fmt.Println("Got an error: ", err)
        return
    }
    defer config.Close()
    speechSynthesizer, err := speech.NewSpeechSynthesizerFromConfig(config, audioConfig)
    if err != nil {
        fmt.Println("Got an error: ", err)
        return
    }
    defer speechSynthesizer.Close()

    speechSynthesizer.SynthesisStarted(synthesizeStartedHandler)
    speechSynthesizer.Synthesizing(synthesizingHandler)
    speechSynthesizer.SynthesisCompleted(synthesizedHandler)
    speechSynthesizer.SynthesisCanceled(cancelledHandler)

    for {
        fmt.Printf("Enter some text that you want to speak, or enter empty text to exit.\n> ")
        text, _ := bufio.NewReader(os.Stdin).ReadString('\n')
        text = strings.TrimSuffix(text, "\n")
        if len(text) == 0 {
            break
        }

        task := speechSynthesizer.SpeakTextAsync(text)
        var outcome speech.SpeechSynthesisOutcome
        select {
        case outcome = <-task:
        case <-time.After(60 * time.Second):
            fmt.Println("Timed out")
            return
        }
        defer outcome.Close()
        if outcome.Error != nil {
            fmt.Println("Got an error: ", outcome.Error)
            return
        }

        if outcome.Result.Reason == common.SynthesizingAudioCompleted {
            fmt.Printf("Speech synthesized to speaker for text [%s].\n", text)
        } else {
            cancellation, _ := speech.NewCancellationDetailsFromSpeechSynthesisResult(outcome.Result)
            fmt.Printf("CANCELED: Reason=%d.\n", cancellation.Reason)

            if cancellation.Reason == common.Error {
                fmt.Printf("CANCELED: ErrorCode=%d\nCANCELED: ErrorDetails=[%s]\nCANCELED: Did you update the subscription info?\n",
                    cancellation.ErrorCode,
                    cancellation.ErrorDetails)
            }
        }
    }
}
```

Ejecute los siguientes comandos para crear un archivo `go.mod` que vincule a los componentes hospedados en GitHub.

```shell
go mod init quickstart
go get github.com/Microsoft/cognitive-services-speech-sdk-go
```

Compile y ejecute ahora el código.

```shell
go build
go run quickstart
```

Vea los documentos de referencia para obtener información detallada sobre las clases [`SpeechConfig`](https://pkg.go.dev/github.com/Microsoft/cognitive-services-speech-sdk-go/speech#SpeechConfig) y [`SpeechSynthesizer`](https://pkg.go.dev/github.com/Microsoft/cognitive-services-speech-sdk-go/speech#SpeechSynthesizer).

## <a name="text-to-speech-to-in-memory-stream"></a>Texto a voz para emisión en memoria

Para muchos de los escenarios del desarrollo de aplicaciones de voz, es probable que necesite los datos de audio resultantes en forma de secuencia en memoria, en lugar de que se escriban directamente en un archivo.
Esto le permitirá crear un comportamiento personalizado, lo que incluye:

* Abstraer la matriz de bytes resultante como una secuencia que permita la búsqueda para los servicios de bajada personalizados.
* Integrar el resultado con otros servicios o API.
* Modificar los datos de audio, escribir encabezados `.wav` personalizados, etc.

Es sencillo hacer este cambio en el ejemplo anterior. En primer lugar, quite `AudioConfig`, porque a partir de este momento administrará el comportamiento de la salida de forma manual, ya que así obtiene mayor control. Después, use `nil` en `AudioConfig` en el constructor `SpeechSynthesizer`.

> [!NOTE]
> Al usar `nil` en `AudioConfig`, en lugar de omitirlo como en el ejemplo de salida del altavoz anterior, el audio no se reproducirá de forma predeterminada en el dispositivo de salida activo actual.

Esta vez se guarda el resultado en una variable [`SpeechSynthesisResult`](https://pkg.go.dev/github.com/Microsoft/cognitive-services-speech-sdk-go/speech#SpeechSynthesisResult).
La propiedad `AudioData` devuelve un `[]byte` de los datos de salida. Puede trabajar con `[]byte` manualmente, o bien puede usar la clase [`AudioDataStream`](https://pkg.go.dev/github.com/Microsoft/cognitive-services-speech-sdk-go/speech#AudioDataStream) para administrar la secuencia en memoria.
En este ejemplo, se usa la función estática `NewAudioDataStreamFromSpeechSynthesisResult()` para obtener una secuencia del resultado.

Reemplace las variables `subscription` y `region` por sus claves de suscripción y región.

```go
package main

import (
    "bufio"
    "fmt"
    "io"
    "os"
    "strings"
    "time"

    "github.com/Microsoft/cognitive-services-speech-sdk-go/speech"
)

func main(subscription string, region string) {
    subscription :=  "YOUR_SUBSCRIPTION_KEY"
    region := "YOUR_SUBSCRIPTIONKEY_REGION"

    config, err := speech.NewSpeechConfigFromSubscription(subscription, region)
    if err != nil {
        fmt.Println("Got an error: ", err)
        return
    }
    defer config.Close()
    speechSynthesizer, err := speech.NewSpeechSynthesizerFromConfig(config, nil)
    if err != nil {
        fmt.Println("Got an error: ", err)
        return
    }
    defer speechSynthesizer.Close()

    speechSynthesizer.SynthesisStarted(synthesizeStartedHandler)
    speechSynthesizer.Synthesizing(synthesizingHandler)
    speechSynthesizer.SynthesisCompleted(synthesizedHandler)
    speechSynthesizer.SynthesisCanceled(cancelledHandler)

    for {
        fmt.Printf("Enter some text that you want to speak, or enter empty text to exit.\n> ")
        text, _ := bufio.NewReader(os.Stdin).ReadString('\n')
        text = strings.TrimSuffix(text, "\n")
        if len(text) == 0 {
            break
        }

        // StartSpeakingTextAsync sends the result to channel when the synthesis starts.
        task := speechSynthesizer.StartSpeakingTextAsync(text)
        var outcome speech.SpeechSynthesisOutcome
        select {
        case outcome = <-task:
        case <-time.After(60 * time.Second):
            fmt.Println("Timed out")
            return
        }
        defer outcome.Close()
        if outcome.Error != nil {
            fmt.Println("Got an error: ", outcome.Error)
            return
        }

        // in most case we want to streaming receive the audio to lower the latency,
        // we can use AudioDataStream to do so.
        stream, err := speech.NewAudioDataStreamFromSpeechSynthesisResult(outcome.Result)
        defer stream.Close()
        if err != nil {
            fmt.Println("Got an error: ", err)
            return
        }

        var all_audio []byte
        audio_chunk := make([]byte, 2048)
        for {
            n, err := stream.Read(audio_chunk)

            if err == io.EOF {
                break
            }

            all_audio = append(all_audio, audio_chunk[:n]...)
        }

        fmt.Printf("Read [%d] bytes from audio data stream.\n", len(all_audio))
    }
}
```

Ejecute los siguientes comandos para crear un archivo `go.mod` que vincule a los componentes hospedados en GitHub.

```shell
go mod init quickstart
go get github.com/Microsoft/cognitive-services-speech-sdk-go
```

Compile y ejecute ahora el código.

```shell
go build
go run quickstart
```

Vea los documentos de referencia para obtener información detallada sobre las clases [`SpeechConfig`](https://pkg.go.dev/github.com/Microsoft/cognitive-services-speech-sdk-go/speech#SpeechConfig) y [`SpeechSynthesizer`](https://pkg.go.dev/github.com/Microsoft/cognitive-services-speech-sdk-go/speech#SpeechSynthesizer).

## <a name="use-ssml-to-customize-speech-characteristics"></a>Uso de SSML para personalizar las características de la voz

El lenguaje de marcado de síntesis de voz (SSML) permite ajustar el tono, la pronunciación, la velocidad del habla, el volumen, etc. de la salida de texto a voz mediante el envío de solicitudes desde un lenguaje de definición de esquema XML. En esta sección se muestra un ejemplo de cambio de voz, pero para obtener una guía más detallada, consulte el [artículo de procedimientos de SSML](../../../speech-synthesis-markup.md).

Para empezar a usar SSML para la personalización, realice un cambio sencillo que cambie la voz.
En primer lugar, cree un archivo XML para la configuración de SSML en el directorio raíz del proyecto, en este ejemplo `ssml.xml`. El elemento raíz es siempre `<speak>` y si se ajusta el texto en un elemento `<voice>`, se puede cambiar la voz mediante el parámetro `name`. Consulte la [lista completa](../../../language-support.md#neural-voices) de voces **neuronales** admitidas.

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
  <voice name="en-US-AriaNeural">
    When you're on the freeway, it's a good idea to use a GPS.
  </voice>
</speak>
```

A continuación, debe cambiar la solicitud de síntesis de voz para que haga referencia al archivo XML.
La solicitud es básicamente la misma, pero en lugar de usar la función `SpeakTextAsync()`, se usa `SpeakSsmlAsync()`. Esta función espera una cadena XML, por lo que lo primero que se debe hacer es cargar la configuración de SSML como una cadena. Desde aquí, el objeto de resultado es exactamente el mismo que el de los ejemplos anteriores.

> [!NOTE]
> Para cambiar la voz sin usar SSML, puede establecer la propiedad en `SpeechConfig` mediante `speechConfig.SetSpeechSynthesisVoiceName("en-US-AriaNeural")`

## <a name="get-facial-pose-events"></a>Obtención de eventos de postura facial

La voz puede ser una buena forma de dotar de animación a las expresiones faciales.
Para representar las posiciones principales en el habla observada, como la colocación de los labios, la mandíbula y la lengua al producir un fonema determinado, con frecuencia se usan [visemas](../../../how-to-speech-synthesis-viseme.md).
Puede suscribirse al evento viseme en Speech SDK.
Después, puede aplicar eventos de visema para animar la cara de un personaje mientras se reproduce el audio de voz.
Aprenda a [obtener eventos de visema](../../../how-to-speech-synthesis-viseme.md#get-viseme-events-with-the-speech-sdk).
