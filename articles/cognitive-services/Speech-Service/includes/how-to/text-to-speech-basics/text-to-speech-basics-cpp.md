---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/25/2020
ms.author: trbye
ms.openlocfilehash: 7b1bc00e54abed70d4cbb769e15ff0f304c62b85
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107108935"
---
En este inicio rápido aprenderá patrones de diseño comunes para realizar la síntesis de texto a voz mediante el SDK de voz. Para empezar, puede realizar una configuración y síntesis básicas y, después, pasar a ejemplos más avanzados para el desarrollo de aplicaciones personalizadas, entre las que se incluyen:

* Obtención de respuestas como flujos de datos en memoria
* Personalización de la frecuencia de muestreo y la velocidad de bits de salida
* Envío de solicitudes de síntesis mediante SSML (lenguaje de marcado de síntesis de voz)
* Uso de voces neuronales

## <a name="skip-to-samples-on-github"></a>Pasar a los ejemplos en GitHub

Si quiere pasar directamente al código de ejemplo, consulte los [ejemplos del inicio rápido de C++](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/cpp/windows/text-to-speech) en GitHub.

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por sentado que tiene una cuenta de Azure y una suscripción al servicio de voz. Si no dispone de una cuenta y una suscripción, [pruebe el servicio de voz de forma gratuita](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-the-speech-sdk"></a>Instalación de Speech SDK

En primer lugar, deberá instalar Speech SDK. Utilice las siguientes instrucciones en función de la plataforma:

* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-cpp&tabs=linux" target="_blank">Linux </a>
* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-cpp&tabs=macos" target="_blank">macOS </a>
* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-cpp&tabs=windows" target="_blank">Windows </a>

## <a name="import-dependencies"></a>Dependencias de importación

Para ejecutar los ejemplos de este artículo, incluya las siguientes instrucciones `using` y de importación en el principio del script.

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include <speechapi_cxx.h>

using namespace std;
using namespace Microsoft::CognitiveServices::Speech;
using namespace Microsoft::CognitiveServices::Speech::Audio;
```

## <a name="create-a-speech-configuration"></a>Creación de una configuración de voz

Para llamar al servicio de voz con Speech SDK, debe crear un elemento [`SpeechConfig`](/cpp/cognitive-services/speech/speechconfig). Esta clase incluye información sobre la suscripción, como la clave, la región asociada, el punto de conexión, el host o el token de autorización.

> [!NOTE]
> Debe crear siempre una configuración, independientemente de si va a realizar reconocimiento de voz, síntesis de voz, traducción o reconocimiento de intenciones.

Existen diversas maneras para inicializar un elemento [`SpeechConfig`](/cpp/cognitive-services/speech/speechconfig):

* Con una suscripción: pase una clave y la región asociada.
* Con un punto de conexión: pase un punto de conexión del servicio de voz. La clave y el token de autorización son opcionales.
* Con un host: pase una dirección de host. La clave y el token de autorización son opcionales.
* Con un token de autorización: pase el token de autorización y la región asociada.

En este ejemplo, se crea un elemento [`SpeechConfig`](/cpp/cognitive-services/speech/speechconfig) mediante una clave de suscripción y una región. Para obtener estas credenciales, siga los pasos descritos en [Prueba gratuita del servicio Voz](../../../overview.md#try-the-speech-service-for-free). Cree también código reutilizable básico para usarlo en el resto del artículo y que modificará cuando realice distintas personalizaciones.

```cpp
int wmain()
{
    try
    {
        synthesizeSpeech();
    }
    catch (exception e)
    {
        cout << e.what();
    }
    return 0;
}

void synthesizeSpeech()
{
    auto config = SpeechConfig::FromSubscription("YourSubscriptionKey", "YourServiceRegion");
}
```

## <a name="synthesize-speech-to-a-file"></a>Síntesis de voz en un archivo

A continuación, creará un objeto [`SpeechSynthesizer`](/cpp/cognitive-services/speech/speechsynthesizer), que ejecuta conversiones de texto a voz y envía la salida a altavoces, archivos u otros flujos de salida. [`SpeechSynthesizer`](/cpp/cognitive-services/speech/speechsynthesizer) acepta como parámetros tanto el objeto [`SpeechConfig`](/cpp/cognitive-services/speech/speechconfig) que se creó creado en el paso anterior y un objeto [`AudioConfig`](/cpp/cognitive-services/speech/audio-audioconfig) que especifica cómo se deben controlar los resultados de la salida.

Para empezar, cree un objeto `AudioConfig` para escribir automáticamente la salida en un archivo `.wav` mediante la función `FromWavFileOutput()`.

```cpp
void synthesizeSpeech()
{
    auto config = SpeechConfig::FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    auto audioConfig = AudioConfig::FromWavFileOutput("path/to/write/file.wav");
}
```

Luego, cree una instancia de `SpeechSynthesizer` y use los objetos `config` y `audioConfig` como parámetros. Después, para ejecutar la síntesis de voz y realizar la escritura en un archivo solo hay que ejecutar `SpeakTextAsync()` con una cadena de texto.

```cpp
void synthesizeSpeech()
{
    auto config = SpeechConfig::FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    auto audioConfig = AudioConfig::FromWavFileOutput("path/to/write/file.wav");
    auto synthesizer = SpeechSynthesizer::FromConfig(config, audioConfig);
    auto result = synthesizer->SpeakTextAsync("A simple test to write to a file.").get();
}
```

Ejecute el programa y se escribirá un archivo `.wav` sintetizado en la ubicación que haya especificado. Este es un buen ejemplo del uso más básico, pero a continuación puede examinar cómo personalizar la salida y controlar la respuesta de la salida como una secuencia en memoria para trabajar con escenarios personalizados.

## <a name="synthesize-to-speaker-output"></a>Síntesis a la salida de altavoz

En algunos casos, puede que desee enviar la voz sintetizada directamente a un altavoz. Para ello, omita el parámetro `AudioConfig` al crear `SpeechSynthesizer` en el ejemplo anterior. De esta forma, se sintetiza en el dispositivo de salida activo actual.

```cpp
void synthesizeSpeech()
{
    auto config = SpeechConfig::FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    auto synthesizer = SpeechSynthesizer::FromConfig(config);
    auto result = synthesizer->SpeakTextAsync("Synthesizing directly to speaker output.").get();
}
```

## <a name="get-result-as-an-in-memory-stream"></a>Obtención del resultado en forma de secuencia en memoria

Para muchos de los escenarios del desarrollo de aplicaciones de voz, es probable que necesite los datos de audio resultantes en forma de secuencia en memoria, en lugar de que se escriban directamente en un archivo. Esto le permitirá crear un comportamiento personalizado, lo que incluye:

* Abstraer la matriz de bytes resultante como una secuencia que permita la búsqueda para los servicios de bajada personalizados.
* Integrar el resultado con otros servicios o API.
* Modificar los datos de audio, escribir encabezados `.wav` personalizados, etc.

Es sencillo hacer este cambio en el ejemplo anterior. En primer lugar, quite `AudioConfig`, porque a partir de este momento administrará el comportamiento de la salida de forma manual, ya que así obtiene mayor control. Después, use `NULL` en `AudioConfig` en el constructor `SpeechSynthesizer`.

> [!NOTE]
> Al usar `NULL` en `AudioConfig`, en lugar de omitirlo como en el ejemplo de salida del altavoz anterior, el audio no se reproducirá de forma predeterminada en el dispositivo de salida activo actual.

Esta vez se guarda el resultado en una variable [`SpeechSynthesisResult`](/cpp/cognitive-services/speech/speechsynthesisresult). El captador `GetAudioData` devuelve un `byte []` de los datos de salida. Puede trabajar con `byte []` manualmente, o bien puede usar la clase [`AudioDataStream`](/cpp/cognitive-services/speech/audiodatastream) para administrar la secuencia en memoria. En este ejemplo, se usa la función estática `AudioDataStream.FromResult()` para obtener una secuencia del resultado.

```cpp
void synthesizeSpeech()
{
    auto config = SpeechConfig::FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    auto synthesizer = SpeechSynthesizer::FromConfig(config, NULL);

    auto result = synthesizer->SpeakTextAsync("Getting the response as an in-memory stream.").get();
    auto stream = AudioDataStream::FromResult(result);
}
```

A partir de aquí se puede implementar cualquier comportamiento personalizado mediante el objeto `stream` resultante.

## <a name="customize-audio-format"></a>Personalización del formato de audio

En la siguiente sección se muestra cómo personalizar los atributos de la salida de audio, entre los que se incluyen:

* Tipo de archivo de audio
* Frecuencia de muestreo
* Profundidad de bits

Para cambiar el formato de audio se usa la función `SetSpeechSynthesisOutputFormat()` en el objeto `SpeechConfig`. Esta función espera un elemento `enum` del tipo [`SpeechSynthesisOutputFormat`](/cpp/cognitive-services/speech/microsoft-cognitiveservices-speech-namespace#speechsynthesisoutputformat), que se usa para seleccionar el formato de salida. En los documentos de referencia encontrará una [lista de los formatos de audio](/cpp/cognitive-services/speech/microsoft-cognitiveservices-speech-namespace#speechsynthesisoutputformat) disponibles.

Hay varias opciones para los distintos tipos de archivo, por lo que puede elegir la que necesite. Tenga en cuenta que, por definición, los formatos sin procesar, como `Raw24Khz16BitMonoPcm`, no incluyen encabezados de audio. Los formatos sin procesar solo se deben usar cuando se sepa que la implementación de bajada puede descodificar una secuencia de bits sin procesar, o bien si planea crear manualmente encabezados basados en la profundidad de bits, la frecuencia de muestreo, el número de canales, etc.

En este ejemplo, se especifica un formato RIFF de alta fidelidad `Riff24Khz16BitMonoPcm`, para lo que se establece `SpeechSynthesisOutputFormat` en el objeto `SpeechConfig`. Al igual que en el ejemplo de la sección anterior, se usa [`AudioDataStream`](/cpp/cognitive-services/speech/audiodatastream) para obtener una secuencia en memoria del resultado y, después, escribirla en un archivo.

```cpp
void synthesizeSpeech()
{
    auto config = SpeechConfig::FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    config->SetSpeechSynthesisOutputFormat(SpeechSynthesisOutputFormat::Riff24Khz16BitMonoPcm);

    auto synthesizer = SpeechSynthesizer::FromConfig(config, NULL);
    auto result = synthesizer->SpeakTextAsync("A simple test to write to a file.").get();

    auto stream = AudioDataStream::FromResult(result);
    stream->SaveToWavFileAsync("path/to/write/file.wav").get();
}
```

Si vuelve a ejecutar el programa, se escribirá un archivo `.wav` en la ruta de acceso especificada.

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

A continuación, debe cambiar la solicitud de síntesis de voz para que haga referencia al archivo XML. La solicitud es básicamente la misma, pero en lugar de usar la función `SpeakTextAsync()`, se usa `SpeakSsmlAsync()`. Esta función espera una cadena XML, por lo que lo primero que se debe hacer es cargar la configuración de SSML como una cadena. Desde aquí, el objeto de resultado es exactamente el mismo que el de los ejemplos anteriores.

```cpp
void synthesizeSpeech()
{
    auto config = SpeechConfig::FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    auto synthesizer = SpeechSynthesizer::FromConfig(config, NULL);

    std::ifstream file("./ssml.xml");
    std::string ssml, line;
    while (std::getline(file, line))
    {
        ssml += line;
        ssml.push_back('\n');
    }
    auto result = synthesizer->SpeakSsmlAsync(ssml).get();

    auto stream = AudioDataStream::FromResult(result);
    stream->SaveToWavFileAsync("path/to/write/file.wav").get();
}
```

> [!NOTE]
> Para cambiar la voz sin usar SSML, puede establecer la propiedad en `SpeechConfig` mediante `SpeechConfig.SetSpeechSynthesisVoiceName("en-US-AriaNeural")`

## <a name="get-facial-pose-events"></a>Obtención de eventos de postura facial

La voz puede ser una buena forma de dotar de animación a las expresiones faciales.
Para representar las posiciones principales en el habla observada, como la colocación de los labios, la mandíbula y la lengua al producir un fonema determinado, con frecuencia se usan [visemas](../../../how-to-speech-synthesis-viseme.md).
Puede suscribirse al evento viseme en Speech SDK.
Después, puede aplicar eventos de visema para animar la cara de un personaje mientras se reproduce el audio de voz.
Aprenda a [obtener eventos de visema](../../../how-to-speech-synthesis-viseme.md#get-viseme-events-with-the-speech-sdk).
