---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 02/10/2021
ms.author: trbye
ms.custom: devx-track-js
ms.openlocfilehash: 4d228388d951314b03ecd950052f76a20b6e4166
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107108860"
---
En este inicio rápido aprenderá patrones de diseño comunes para realizar la síntesis de texto a voz mediante el SDK de voz. Para empezar, puede realizar una configuración y síntesis básicas y, después, pasar a ejemplos más avanzados para el desarrollo de aplicaciones personalizadas, entre las que se incluyen:

* Obtención de respuestas como flujos de datos en memoria
* Personalización de la frecuencia de muestreo y la velocidad de bits de salida
* Envío de solicitudes de síntesis mediante SSML (lenguaje de marcado de síntesis de voz)
* Uso de voces neuronales

## <a name="skip-to-samples-on-github"></a>Pasar a los ejemplos en GitHub

Si quiere pasar directamente al código de ejemplo, consulte los [ejemplos del inicio rápido de JavaScript](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript/node/text-to-speech) en GitHub.

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por sentado que tiene una cuenta de Azure y una suscripción al servicio Voz. Si no dispone de una cuenta y un recurso, [pruebe el servicio Voz de forma gratuita](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-the-speech-sdk"></a>Instalación de Speech SDK

En primer lugar, deberá instalar <a href="https://www.npmjs.com/package/microsoft-cognitiveservices-speech-sdk" target="_blank">Speech SDK para JavaScript</a>. Utilice las siguientes instrucciones en función de la plataforma:
- <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk?tabs=nodejs#get-the-speech-sdk" target="_blank">Node.js <span
class="docon docon-navigate-external x-hidden-focus"></span></a>
- <a href="/azure/cognitive-services/speech-service/speech-sdk?tabs=browser#get-the-speech-sdk" target="_blank">Explorador web </a>

Además, en función del entorno de destino, use una de las siguientes:

# <a name="script"></a>[script](#tab/script)

Descargue y extraiga el archivo *microsoft.cognitiveservices.speech.sdk.bundle.js* de <a href="https://aka.ms/csspeech/jsbrowserpackage" target="_blank">Speech SDK para JavaScript</a> y colóquelo en una carpeta accesible para el archivo HTML.

```html
<script src="microsoft.cognitiveservices.speech.sdk.bundle.js"></script>;
```

> [!TIP]
> Si el destino es un explorador web y usa la etiqueta `<script>`, el prefijo `sdk` no es necesario. El prefijo `sdk` es un alias que se usa para asignar un nombre al módulo `require`.

# <a name="import"></a>[import](#tab/import)

```javascript
import * as sdk from "microsoft-cognitiveservices-speech-sdk";
```

Para más información sobre `import`, consulte <a href="https://javascript.info/import-export" target="_blank">export and import </a> (export e import).

# <a name="require"></a>[require](#tab/require)

```javascript
const sdk = require("microsoft-cognitiveservices-speech-sdk");
```

Para más información sobre `require`, consulte <a href="https://nodejs.org/en/knowledge/getting-started/what-is-require/" target="_blank">what is require? </a> (¿Qué es require?).

---


## <a name="create-a-speech-configuration"></a>Creación de una configuración de voz

Para llamar al servicio de voz con Speech SDK, debe crear un elemento [`SpeechConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig). Esta clase incluye información sobre el recurso, como la clave, la región asociada, el punto de conexión, el host o el token de autorización.

> [!NOTE]
> Debe crear siempre una configuración, independientemente de si va a realizar reconocimiento de voz, síntesis de voz, traducción o reconocimiento de intenciones.

Existen diversas maneras para inicializar un elemento [`SpeechConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig):

* Con un recurso: pase una clave y la región asociada.
* Con un punto de conexión: pase un punto de conexión del servicio de voz. La clave y el token de autorización son opcionales.
* Con un host: pase una dirección de host. La clave y el token de autorización son opcionales.
* Con un token de autorización: pase el token de autorización y la región asociada.

En este ejemplo, se crea un elemento [`SpeechConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig) mediante una clave de recurso y una región. Para obtener estas credenciales, siga los pasos descritos en [Prueba gratuita del servicio Voz](../../../overview.md#try-the-speech-service-for-free). Cree también código reutilizable básico para usarlo en el resto del artículo y que modificará cuando realice distintas personalizaciones.

```javascript
function synthesizeSpeech() {
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
}

synthesizeSpeech();
```

## <a name="synthesize-speech-to-a-file"></a>Síntesis de voz en un archivo

A continuación, creará un objeto [`SpeechSynthesizer`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesizer), que ejecuta conversiones de texto a voz y envía la salida a altavoces, archivos u otros flujos de salida. [`SpeechSynthesizer`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesizer) acepta como parámetros tanto el objeto [`SpeechConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig) que se creó creado en el paso anterior y un objeto [`AudioConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/audioconfig) que especifica cómo se deben controlar los resultados de la salida.

Para empezar, cree un objeto `AudioConfig` para escribir automáticamente la salida en un archivo `.wav` mediante la función estática `fromAudioFileOutput()`.

```javascript
function synthesizeSpeech() {
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
    const audioConfig = AudioConfig.fromAudioFileOutput("path/to/file.wav");
}
```

Luego, cree una instancia de `SpeechSynthesizer` y use los objetos `speechConfig` y `audioConfig` como parámetros. Después, para ejecutar la síntesis de voz y realizar la escritura en un archivo solo hay que ejecutar `speakTextAsync()` con una cadena de texto. La devolución de llamada de resultados es una excelente ubicación para llamar a `synthesizer.close()`; de hecho, esta llamada es necesaria para que la síntesis funcione correctamente.

```javascript
function synthesizeSpeech() {
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
    const audioConfig = AudioConfig.fromAudioFileOutput("path-to-file.wav");

    const synthesizer = new SpeechSynthesizer(speechConfig, audioConfig);
    synthesizer.speakTextAsync(
        "A simple test to write to a file.",
        result => {
            synthesizer.close();
            if (result) {
                // return result as stream
                return fs.createReadStream("path-to-file.wav");
            }
        },
        error => {
            console.log(error);
            synthesizer.close();
        });
}
```

Ejecute el programa y se escribirá un archivo `.wav` sintetizado en la ubicación que haya especificado. Este es un buen ejemplo del uso más básico, pero a continuación puede examinar cómo personalizar la salida y controlar la respuesta de la salida como una secuencia en memoria para trabajar con escenarios personalizados.

## <a name="synthesize-to-speaker-output"></a>Síntesis a la salida de altavoz

En algunos casos, puede que desee enviar la voz sintetizada directamente a un altavoz. Para ello, cree una instancia de `AudioConfig` mediante la función estática `fromDefaultSpeakerOutput()`. De esta forma, la salida s realiza a través del dispositivo de salida activo actual.

```javascript
function synthesizeSpeech() {
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
    const audioConfig = AudioConfig.fromDefaultSpeakerOutput();

    const synthesizer = new SpeechSynthesizer(speechConfig, audioConfig);
    synthesizer.speakTextAsync(
        "Synthesizing directly to speaker output.",
        result => {
            if (result) {
                synthesizer.close();
                return result.audioData;
            }
        },
        error => {
            console.log(error);
            synthesizer.close();
        });
}
```

## <a name="get-result-as-an-in-memory-stream"></a>Obtención del resultado en forma de secuencia en memoria

Para muchos de los escenarios del desarrollo de aplicaciones de voz, es probable que necesite los datos de audio resultantes en forma de secuencia en memoria, en lugar de que se escriban directamente en un archivo. Esto le permitirá crear un comportamiento personalizado, lo que incluye:

* Abstraer la matriz de bytes resultante como una secuencia que permita la búsqueda para los servicios de bajada personalizados.
* Integrar el resultado con otros servicios o API.
* Modificar los datos de audio, escribir encabezados `.wav` personalizados, etc.

Es sencillo hacer este cambio en el ejemplo anterior. En primer lugar, quite el bloqueo de `AudioConfig`, porque a partir de este momento administrará el comportamiento de la salida de forma manual, ya que así obtiene mayor control. Después, use `undefined` en `AudioConfig` en el constructor `SpeechSynthesizer`.

> [!NOTE]
> Al usar `undefined` en `AudioConfig`, en lugar de omitirlo como en el ejemplo de salida del altavoz anterior, el audio no se reproducirá de forma predeterminada en el dispositivo de salida activo actual.

Esta vez se guarda el resultado en una variable [`SpeechSynthesisResult`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesisresult). La propiedad `SpeechSynthesisResult.audioData` devuelve un `ArrayBuffer` de los datos de salida, el tipo de flujo de datos del explorador predeterminado. En el caso del código de servidor, convierta arrayBuffer en un flujo del búfer.

El código siguiente funciona para el código del lado cliente.

```javascript
function synthesizeSpeech() {
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
    const synthesizer = new sdk.SpeechSynthesizer(speechConfig);

    synthesizer.speakTextAsync(
        "Getting the response as an in-memory stream.",
        result => {
            synthesizer.close();
            return result.audioData;
        },
        error => {
            console.log(error);
            synthesizer.close();
        });
}
```

A partir de aquí se puede implementar cualquier comportamiento personalizado mediante el objeto `ArrayBuffer` resultante. ArrayBuffer es un tipo común que se recibe en un explorador y se reproduce desde este formato.

En el caso de cualquier código basado en servidor, si necesita trabajar con los datos como un flujo, en lugar de un ArrayBuffer, debe convertir el objeto en un flujo.

```javascript
function synthesizeSpeech() {
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
    const synthesizer = new sdk.SpeechSynthesizer(speechConfig);

    synthesizer.speakTextAsync(
        "Getting the response as an in-memory stream.",
        result => {
            const { audioData } = result;

            synthesizer.close();

            // convert arrayBuffer to stream
            // return stream
            const bufferStream = new PassThrough();
            bufferStream.end(Buffer.from(audioData));
            return bufferStream;
        },
        error => {
            console.log(error);
            synthesizer.close();
        });
}
```

## <a name="customize-audio-format"></a>Personalización del formato de audio

En la siguiente sección se muestra cómo personalizar los atributos de la salida de audio, entre los que se incluyen:

* Tipo de archivo de audio
* Frecuencia de muestreo
* Profundidad de bits

Para cambiar el formato de audio, se usa la propiedad `speechSynthesisOutputFormat` en el objeto `SpeechConfig`. Esta propiedad espera un elemento `enum` del tipo [`SpeechSynthesisOutputFormat`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesisoutputformat), que se usa para seleccionar el formato de salida. En los documentos de referencia encontrará una [lista de los formatos de audio](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesisoutputformat) disponibles.

Hay varias opciones para los distintos tipos de archivo, por lo que puede elegir la que necesite. Tenga en cuenta que, por definición, los formatos sin procesar, como `Raw24Khz16BitMonoPcm`, no incluyen encabezados de audio. Los formatos sin procesar solo se deben usar cuando se sepa que la implementación de bajada puede descodificar una secuencia de bits sin procesar, o bien si planea crear manualmente encabezados basados en la profundidad de bits, la frecuencia de muestreo, el número de canales, etc.

En este ejemplo, se especifica un formato RIFF de alta fidelidad `Riff24Khz16BitMonoPcm`, para lo que se establece `speechSynthesisOutputFormat` en el objeto `SpeechConfig`. De forma similar al ejemplo de la sección anterior, obtenga los datos de `ArrayBuffer` de audio e interactúe con ellos.

```javascript
function synthesizeSpeech() {
    const speechConfig = SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");

    // Set the output format
    speechConfig.speechSynthesisOutputFormat = SpeechSynthesisOutputFormat.Riff24Khz16BitMonoPcm;

    const synthesizer = new sdk.SpeechSynthesizer(speechConfig, undefined);
    synthesizer.speakTextAsync(
        "Customizing audio output format.",
        result => {
            // Interact with the audio ArrayBuffer data
            const audioData = result.audioData;
            console.log(`Audio data byte size: ${audioData.byteLength}.`)

            synthesizer.close();
        },
        error => {
            console.log(error);
            synthesizer.close();
        });
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

A continuación, debe cambiar la solicitud de síntesis de voz para que haga referencia al archivo XML. La solicitud es básicamente la misma, pero en lugar de usar la función `speakTextAsync()`, se usa `speakSsmlAsync()`. Esta función espera una cadena XML, por lo que primero debe crear una función para cargar un archivo XML y devolverlo como una cadena.

```javascript
function xmlToString(filePath) {
    const xml = readFileSync(filePath, "utf8");
    return xml;
}
```

Para más información sobre `readFileSync`, consulte <a href="https://nodejs.org/api/fs.html#fs_fs_readlinksync_path_options" target="_blank">Node.js file system</a> (Sistema de archivos de Node.js). Desde aquí, el objeto del resultado es exactamente el mismo que el de los ejemplos anteriores.

```javascript
function synthesizeSpeech() {
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
    const synthesizer = new sdk.SpeechSynthesizer(speechConfig, undefined);

    const ssml = xmlToString("ssml.xml");
    synthesizer.speakSsmlAsync(
        ssml,
        result => {
            if (result.errorDetails) {
                console.error(result.errorDetails);
            } else {
                console.log(JSON.stringify(result));
            }

            synthesizer.close();
        },
        error => {
            console.log(error);
            synthesizer.close();
        });
}
```

> [!NOTE]
> Para cambiar la voz sin usar SSML, puede establecer la propiedad en `SpeechConfig` mediante `SpeechConfig.speechSynthesisVoiceName = "en-US-AriaNeural";`

## <a name="get-facial-pose-events"></a>Obtención de eventos de postura facial

La voz puede ser una buena forma de dotar de animación a las expresiones faciales.
Para representar las posiciones principales en el habla observada, como la colocación de los labios, la mandíbula y la lengua al producir un fonema determinado, con frecuencia se usan [visemas](../../../how-to-speech-synthesis-viseme.md).
Puede suscribirse al evento viseme en Speech SDK.
Después, puede aplicar eventos de visema para animar la cara de un personaje mientras se reproduce el audio de voz.
Aprenda a [obtener eventos de visema](../../../how-to-speech-synthesis-viseme.md#get-viseme-events-with-the-speech-sdk).
