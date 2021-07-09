---
title: Reducción de la latencia de la síntesis de voz mediante el SDK de Voz
titleSuffix: Azure Cognitive Services
description: Reducción de la latencia de la síntesis de voz mediante el SDK de Voz, lo que incluye el streaming, la conexión previa, etc.
services: cognitive-services
author: yulin-li
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/29/2021
ms.author: yulili
ms.custom: references_regions
zone_pivot_groups: programming-languages-set-nineteen
ms.openlocfilehash: 8aaa087d1de85b77c9407f8d39e029e3bf2696f2
ms.sourcegitcommit: 6323442dbe8effb3cbfc76ffdd6db417eab0cef7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110614853"
---
# <a name="lower-speech-synthesis-latency-using-speech-sdk"></a>Menor latencia de síntesis de voz mediante el SDK de Voz

> [!NOTE]
> Este artículo requiere la versión 1.17.0 del SDK de Voz, o cualquier versión posterior.

La latencia de síntesis es crítica para las aplicaciones.
En este artículo, se presentan varios procedimientos recomendados no solo para reducir la latencia, sino también para que los usuarios finales tengan el mejor rendimiento posible.

Habitualmente, la latencia se mide por `first byte latency` y `finish latency`, como se muestra a continuación:

::: zone pivot="programming-language-csharp"

| Latencia | Descripción | Clave de la propiedad [SpeechSynthesisResult](/dotnet/api/microsoft.cognitiveservices.speech.speechsynthesisresult) |
|-----------|-------------|------------|
| latencia del primer byte | Indica el retraso entre el inicio de la tarea de síntesis y la recepción del primer fragmento de datos de audio. | SpeechServiceResponse_SynthesisFirstByteLatencyMs |
| latencia de finalización | Indica el retraso entre el inicio de la tarea de síntesis y la recepción de todos los datos de audio sintetizados. | SpeechServiceResponse_SynthesisFinishLatencyMs |

El SDK de Voz coloca las duraciones de la latencia en la colección Properties de [`SpeechSynthesisResult`](/dotnet/api/microsoft.cognitiveservices.speech.speechsynthesisresult). En el siguiente código de ejemplo se muestran estos valores.

```csharp
var result = await synthesizer.SpeakTextAsync(text);
Console.WriteLine($"first byte latency: \t{result.Properties.GetProperty(PropertyId.SpeechServiceResponse_SynthesisFirstByteLatencyMs)} ms");
Console.WriteLine($"finish latency: \t{result.Properties.GetProperty(PropertyId.SpeechServiceResponse_SynthesisFinishLatencyMs)} ms");
// you can also get the result id, and send to us when you need help for diagnosis
var resultId = result.ResultId;
```

::: zone-end

::: zone pivot="programming-language-cpp"

| Latencia | Descripción | Clave de la propiedad [SpeechSynthesisResult](/cpp/cognitive-services/speech/speechsynthesisresult) |
|-----------|-------------|------------|
| `first byte latency` | Indica el retraso entre que se inicia la síntesis y que se recibe el primer fragmento de audio. | `SpeechServiceResponse_SynthesisFirstByteLatencyMs` |
| `finish latency` | Indica el retraso entre que se inicia la síntesis y que se recibe el todo el audio sintetizado. | `SpeechServiceResponse_SynthesisFinishLatencyMs` |

El SDK de Voz ha medido las latencias y las coloca en el bolsa de propiedades de [`SpeechSynthesisResult`](/cpp/cognitive-services/speech/speechsynthesisresult). Consulte los códigos siguientes para obtenerlas.

```cpp
auto result = synthesizer->SpeakTextAsync(text).get();
auto firstByteLatency = std::stoi(result->Properties.GetProperty(PropertyId::SpeechServiceResponse_SynthesisFirstByteLatencyMs));
auto finishedLatency = std::stoi(result->Properties.GetProperty(PropertyId::SpeechServiceResponse_SynthesisFinishLatencyMs));
// you can also get the result id, and send to us when you need help for diagnosis
auto resultId = result->ResultId;
```

::: zone-end

::: zone pivot="programming-language-java"

| Latencia | Descripción | Clave de la propiedad [SpeechSynthesisResult](/java/api/com.microsoft.cognitiveservices.speech.speechsynthesisresult) |
|-----------|-------------|------------|
| `first byte latency` | Indica el retraso entre que se inicia la síntesis y que se recibe el primer fragmento de audio. | `SpeechServiceResponse_SynthesisFirstByteLatencyMs` |
| `finish latency` | Indica el retraso entre que se inicia la síntesis y que se recibe el todo el audio sintetizado. | `SpeechServiceResponse_SynthesisFinishLatencyMs` |

El SDK de Voz ha medido las latencias y las coloca en el bolsa de propiedades de [`SpeechSynthesisResult`](/java/api/com.microsoft.cognitiveservices.speech.speechsynthesisresult). Consulte los códigos siguientes para obtenerlas.

```java
SpeechSynthesisResult result = synthesizer.SpeakTextAsync(text).get();
System.out.println("first byte latency: \t" + result.getProperties().getProperty(PropertyId.SpeechServiceResponse_SynthesisFirstByteLatencyMs) + " ms.");
System.out.println("finish latency: \t" + result.getProperties().getProperty(PropertyId.SpeechServiceResponse_SynthesisFinishLatencyMs) + " ms.");
// you can also get the result id, and send to us when you need help for diagnosis
String resultId = result.getResultId();
```

::: zone-end


::: zone pivot="programming-language-python"

| Latencia | Descripción | Clave de la propiedad [SpeechSynthesisResult](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechsynthesisresult) |
|-----------|-------------|------------|
| `first byte latency` | Indica el retraso entre que se inicia la síntesis y que se recibe el primer fragmento de audio. | `SpeechServiceResponse_SynthesisFirstByteLatencyMs` |
| `finish latency` | Indica el retraso entre que se inicia la síntesis y que se recibe el todo el audio sintetizado. | `SpeechServiceResponse_SynthesisFinishLatencyMs` |

El SDK de Voz ha medido las latencias y las coloca en el bolsa de propiedades de [`SpeechSynthesisResult`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechsynthesisresult). Consulte los códigos siguientes para obtenerlas.

```python
result = synthesizer.speak_text_async(text).get()
first_byte_latency = int(result.properties.get_property(speechsdk.PropertyId.SpeechServiceResponse_SynthesisFirstByteLatencyMs))
finished_latency = int(result.properties.get_property(speechsdk.PropertyId.SpeechServiceResponse_SynthesisFinishLatencyMs))
# you can also get the result id, and send to us when you need help for diagnosis
result_id = result.result_id
```

::: zone-end

::: zone pivot="programming-language-objectivec"

| Latencia | Descripción | Clave de la propiedad [SPXSpeechSynthesisResult](/objectivec/cognitive-services/speech/spxspeechsynthesisresult) |
|-----------|-------------|------------|
| `first byte latency` | Indica el retraso entre que se inicia la síntesis y que se recibe el primer fragmento de audio. | `SPXSpeechServiceResponseSynthesisFirstByteLatencyMs` |
| `finish latency` | Indica el retraso entre que se inicia la síntesis y que se recibe el todo el audio sintetizado. | `SPXSpeechServiceResponseSynthesisFinishLatencyMs` |

El SDK de Voz ha medido las latencias y las coloca en el bolsa de propiedades de [`SPXSpeechSynthesisResult`](/objectivec/cognitive-services/speech/spxspeechsynthesisresult). Consulte los códigos siguientes para obtenerlas.

```Objective-C
SPXSpeechSynthesisResult *speechResult = [speechSynthesizer speakText:text];
int firstByteLatency = [intString [speechResult.properties getPropertyById:SPXSpeechServiceResponseSynthesisFirstByteLatencyMs]];
int finishedLatency = [intString [speechResult.properties getPropertyById:SPXSpeechServiceResponseSynthesisFinishLatencyMs]];
// you can also get the result id, and send to us when you need help for diagnosis
NSString *resultId = result.resultId;
```

::: zone-end

La latencia del primer byte es mucho menor que la latencia de finalización en la mayoría de los casos.
La latencia del primer byte es independiente de la longitud del texto, mientras que la latencia de finalización aumenta con la longitud del texto.

Lo ideal es minimizar la latencia experimentada por el usuario (la latencia antes de que el usuario escucha el sonido) al tiempo de recorrido de una ruta de red, más la latencia del primer fragmento de audio del servicio de síntesis de voz.

## <a name="streaming"></a>Streaming

El streaming es fundamental para reducir la latencia.
El código de cliente puede iniciar la reproducción cuando se recibe el primer fragmento de audio.
En un escenario de servicio, puede reenviar los fragmentos de audio inmediatamente a los clientes, en lugar de esperar todo el audio.

::: zone pivot="programming-language-csharp"

Puede usar [`PullAudioOutputStream`](/dotnet/api/microsoft.cognitiveservices.speech.audio.pullaudiooutputstream), [`PushAudioOutputStream`](/dotnet/api/microsoft.cognitiveservices.speech.audio.pushaudiooutputstream), el evento [`Synthesizing`](/dotnet/api/microsoft.cognitiveservices.speech.speechsynthesizer.synthesizing)y [`AudioDateStream`](/dotnet/api/microsoft.cognitiveservices.speech.audiodatastream) del SDK de Voz para habilitar el streaming.

Tomando `AudioDateStream` como ejemplo:

```csharp
using (var synthesizer = new SpeechSynthesizer(config, null as AudioConfig))
{
    using (var result = await synthesizer.StartSpeakingTextAsync(text))
    {
        using (var audioDataStream = AudioDataStream.FromResult(result))
        {
            byte[] buffer = new byte[16000];
            uint filledSize = 0;
            while ((filledSize = audioDataStream.ReadData(buffer)) > 0)
            {
                Console.WriteLine($"{filledSize} bytes received.");
            }
        }
    }
}
```

::: zone-end

::: zone pivot="programming-language-cpp"

Puede usar [`PullAudioOutputStream`](/cpp/cognitive-services/speech/audio-pullaudiooutputstream), [`PushAudioOutputStream`](/cpp/cognitive-services/speech/audio-pushaudiooutputstream), el evento [`Synthesizing`](/cpp/cognitive-services/speech/speechsynthesizer#synthesizing)y [`AudioDateStream`](/cpp/cognitive-services/speech/audiodatastream) del SDK de Voz para habilitar el streaming.

Tomando `AudioDateStream` como ejemplo:

```cpp
auto synthesizer = SpeechSynthesizer::FromConfig(config, nullptr);
auto result = synthesizer->SpeakTextAsync(text).get();
auto audioDataStream = AudioDataStream::FromResult(result);
uint8_t buffer[16000];
uint32_t filledSize = 0;
while ((filledSize = audioDataStream->ReadData(buffer, sizeof(buffer))) > 0)
{
    cout << filledSize << " bytes received." << endl;
}
```

::: zone-end

::: zone pivot="programming-language-java"

Puede usar [`PullAudioOutputStream`](/java/api/com.microsoft.cognitiveservices.speech.audio.pullaudiooutputstream), [`PushAudioOutputStream`](/java/api/com.microsoft.cognitiveservices.speech.audio.pushaudiooutputstream), el evento [`Synthesizing`](/java/api/com.microsoft.cognitiveservices.speech.speechsynthesizer.synthesizing#com_microsoft_cognitiveservices_speech_SpeechSynthesizer_Synthesizing)y [`AudioDateStream`](/java/api/com.microsoft.cognitiveservices.speech.audiodatastream) del SDK de Voz para habilitar el streaming.

Tomando `AudioDateStream` como ejemplo:

```java
SpeechSynthesizer synthesizer = new SpeechSynthesizer(config, null);
SpeechSynthesisResult result = synthesizer.StartSpeakingTextAsync(text).get();
AudioDataStream audioDataStream = AudioDataStream.fromResult(result);
byte[] buffer = new byte[16000];
long filledSize = audioDataStream.readData(buffer);
while (filledSize > 0) {
    System.out.println(filledSize + " bytes received.");
    filledSize = audioDataStream.readData(buffer);
}
```

::: zone-end

::: zone pivot="programming-language-python"

Puede usar [`PullAudioOutputStream`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.audio.pullaudiooutputstream), [`PushAudioOutputStream`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.audio.pushaudiooutputstream), el evento [`Synthesizing`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechsynthesizer#synthesizing)y [`AudioDateStream`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.audiodatastream) del SDK de Voz para habilitar el streaming.

Tomando `AudioDateStream` como ejemplo:

```python
speech_synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config, audio_config=None)
result = speech_synthesizer.start_speaking_text_async(text).get()
audio_data_stream = speechsdk.AudioDataStream(result)
audio_buffer = bytes(16000)
filled_size = audio_data_stream.read_data(audio_buffer)
while filled_size > 0:
    print("{} bytes received.".format(filled_size))
    filled_size = audio_data_stream.read_data(audio_buffer)
```

::: zone-end

::: zone pivot="programming-language-objectivec"

Puede usar [`SPXPullAudioOutputStream`](/objectivec/cognitive-services/speech/spxpullaudiooutputstream), [`SPXPushAudioOutputStream`](/objectivec/cognitive-services/speech/spxpushaudiooutputstream), el evento [`Synthesizing`](/objectivec/cognitive-services/speech/spxspeechsynthesizer#addsynthesizingeventhandler)y [`SPXAudioDataStream`](/objectivec/cognitive-services/speech/spxaudiodatastream) del SDK de Voz para habilitar el streaming.

Tomando `AudioDateStream` como ejemplo:

```Objective-C
SPXSpeechSynthesizer *synthesizer = [[SPXSpeechSynthesizer alloc] initWithSpeechConfiguration:speechConfig audioConfiguration:nil];
SPXSpeechSynthesisResult *speechResult = [synthesizer startSpeakingText:inputText];
SPXAudioDataStream *stream = [[SPXAudioDataStream alloc] initFromSynthesisResult:speechResult];
NSMutableData* data = [[NSMutableData alloc]initWithCapacity:16000];
while ([stream readData:data length:16000] > 0) {
    // Read data here
}
```

::: zone-end

## <a name="pre-connect-and-reuse-speechsynthesizer"></a>Conexión previa y reutilización de SpeechSynthesizer

El SDK de Voz usa un WebSocket para comunicarse con el servicio.
Lo ideal sería que la latencia de red fuera un tiempo de recorrido de ruta (RTT).
Si la conexión se acaba de establecer, la latencia de red incluirá tiempo adicional para establecer la conexión.
El establecimiento de una conexión WebSocket necesita el protocolo de enlace TCP, el protocolo de enlace SSL, la conexión HTTP y la actualización del protocolo, lo que introduce un retraso.
Para evitar la latencia de la conexión, se recomienda realizar una conexión previa y volver a usar `SpeechSynthesizer`.

### <a name="pre-connect"></a>Antes de conectar

Para realizar una conexión previa, establezca una conexión con el servicio de Voz cuando sepa que la conexión se necesitará pronto. Por ejemplo, si va a crear un bot de voz en el cliente, puede conectarse previamente al servicio de síntesis de voz cuando el usuario empiece a hablar y llamar a `SpeakTextAsync` cuando el texto de respuesta del bot esté listo.

::: zone pivot="programming-language-csharp"

```csharp
using (var synthesizer = new SpeechSynthesizer(uspConfig, null as AudioConfig))
{
    using (var connection = Connection.FromSpeechSynthesizer(synthesizer))
    {
        connection.Open(true);
    }
    await synthesizer.SpeakTextAsync(text);
}
```

::: zone-end

::: zone pivot="programming-language-cpp"

```cpp
auto synthesizer = SpeechSynthesizer::FromConfig(config, nullptr);
auto connection = Connection::FromSpeechSynthesizer(synthesizer);
connection->Open(true);
```

::: zone-end

::: zone pivot="programming-language-java"

```java
SpeechSynthesizer synthesizer = new SpeechSynthesizer(speechConfig, (AudioConfig) null);
Connection connection = Connection.fromSpeechSynthesizer(synthesizer);
connection.openConnection(true);
```

::: zone-end

::: zone pivot="programming-language-python"

```python
synthesizer = speechsdk.SpeechSynthesizer(config, None)
connection = speechsdk.Connection.from_speech_synthesizer(synthesizer)
connection.open(True)
```

::: zone-end

::: zone pivot="programming-language-objectivec"

```Objective-C
SPXSpeechSynthesizer* synthesizer = [[SPXSpeechSynthesizer alloc]initWithSpeechConfiguration:self.speechConfig audioConfiguration:nil];
SPXConnection* connection = [[SPXConnection alloc]initFromSpeechSynthesizer:synthesizer];
[connection open:true];
```

::: zone-end

> [!NOTE]
> Si el texto de síntesis está disponible, solo hay que llamar a `SpeakTextAsync` para sintetizar el audio. El SDK controlará la conexión.

### <a name="reuse-speechsynthesizer"></a>Reutilización de SpeechSynthesizer

Otra forma de reducir la latencia de la conexión es volver a usar `SpeechSynthesizer` para que no sea necesario crear `SpeechSynthesizer` para cada síntesis.
Se recomienda usar un grupo de objetos en el escenario de servicio; consulte nuestro código de ejemplo para [C#](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/csharp/sharedcontent/console/speech_synthesis_server_scenario_sample.cs) y [Java](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/java/jre/console/src/com/microsoft/cognitiveservices/speech/samples/console/SpeechSynthesisScenarioSamples.java).


## <a name="transmit-compressed-audio-over-the-network"></a>Transmisión de audio comprimido a través de la red

Si la red no es estable o tiene un límite de ancho de banda, el tamaño de la carga también afectará a la latencia.
Entretanto, un formato de audio comprimido ayuda a ahorrar el ancho de banda de red de los usuarios, algo que resulta especialmente valioso para los usuarios de dispositivos móviles.

Se admiten muchos formatos comprimidos, como `opus`, `webm`, `mp3`, `silk`, etc. La lista completa se puede ver en [SpeechSynthesisOutputFormat](/cpp/cognitive-services/speech/microsoft-cognitiveservices-speech-namespace#speechsynthesisoutputformat).
Por ejemplo, la velocidad de bits del formato `Riff24Khz16BitMonoPcm` es de 384 kbps, mientras que `Audio24Khz48KBitRateMonoMp3` solo cuesta 48 kbps.
Nuestro SDK de Voz usará automáticamente un formato comprimido para la transmisión cuando se establece un formato de salida `pcm` y `GStreamer` se instala correctamente.
Consulte [esta instrucción](how-to-use-codec-compressed-audio-input-streams.md) para instalar y configurar `GStreamer` para el SDK de Voz.

## <a name="others-tips"></a>Otras sugerencias

### <a name="cache-crl-files"></a>Almacenamiento en caché de archivos CRL

El SDK de Voz usa archivos CRL para comprobar la certificación.
El almacenamiento en caché de los archivos CRL hasta que hayan expirado le ayuda a evitar tener que descargar archivos CRL cada vez.
Para más información, consulte [Configuración de OpenSSL para Linux](how-to-configure-openssl-linux.md#certificate-revocation-checks).

### <a name="use-latest-speech-sdk"></a>Uso del SDK de Voz más reciente

El rendimiento del SDK de Voz sigue mejorando, así que intente usar el SDK de Voz más reciente en la aplicación.
Por ejemplo, se ha corregido un `TCP_NODELAY`problema de configuración de la versión [1.16.0](releasenotes.md#speech-sdk-1160-2021-march-release) que reducía el tiempo adicional de un recorrido de ruta.

## <a name="load-test-guideline"></a>Directriz de pruebas de carga

La prueba de carga se puede usar para probar la latencia y capacidad del servicio de síntesis de voz.
Estas son algunas directrices.

 - El servicio de síntesis de voz tiene la capacidad de escalar automáticamente, pero tarda un tiempo en escalar horizontalmente. Si la simultaneidad aumenta en poco tiempo, el cliente puede obtener una latencia larga o un código de error `429` (demasiadas solicitudes). Por consiguiente, se recomienda aumentar la simultaneidad paso a paso en la prueba de carga. Para más información, [consulte este artículo](speech-services-quotas-and-limits.md#general-best-practices-to-mitigate-throttling-during-autoscaling).
 - El servicio tiene una limitación de cuota basada en el tráfico real; por lo tanto, si desea realizar una prueba de carga con la simultaneidad mucho mayor que el tráfico real, conéctese antes de la prueba.

## <a name="next-steps"></a>Pasos siguientes

* [Consulte los ejemplos](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master) en GitHub
