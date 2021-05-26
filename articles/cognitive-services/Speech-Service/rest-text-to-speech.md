---
title: 'Referencia de la API de conversión de texto a voz (REST): servicio de voz'
titleSuffix: Azure Cognitive Services
description: Aprenda a usar la API de REST Text-to-Speech. En este artículo, obtendrá más información sobre las opciones de autorización y de consulta y sobre cómo estructurar una solicitud y recibir una respuesta.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 01/08/2021
ms.author: trbye
ms.custom: references_regions
ms.openlocfilehash: 7a15f51b76cc6109537a8c2e0cbbe27bfb9ffb0f
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110368774"
---
# <a name="text-to-speech-rest-api"></a>Text-to-speech REST API

El servicio de voz le permite [convertir texto en voz sintetizada](#convert-text-to-speech) y [obtener una lista de voces admitidas](#get-a-list-of-voices) para una región con un conjunto de API REST. Cada punto de conexión disponible se asocia con una región. Se requiere una clave de suscripción para el punto de conexión o región que se va a usar.

Text to Speech REST API admite voces neuronales y de texto a voz estándar, y cada una de ellas admite un idioma y un dialecto específicos, que se identifican mediante la configuración regional.

* Para ver una lista completa de voces, consulte [compatibilidad con idiomas](language-support.md#text-to-speech).
* Para obtener información acerca de la disponibilidad regional, consulte las [regiones](regions.md#text-to-speech).

> [!IMPORTANT]
> Los costes varían para voces estándar, personalizadas y neuronales. Para obtener más información, consulte el apartado [Precios](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).

Antes de utilizar esta API, comprenda:

* La API REST de texto a voz requiere un encabezado de autorización. Esto significa que tiene que completar un intercambio de tokens para acceder al servicio. Para más información, consulte [Autenticación](#authentication).

> [!TIP]
> Consulte [este artículo](sovereign-clouds.md) para puntos de conexión de Azure Government y Azure China.

[!INCLUDE [](../../../includes/cognitive-services-speech-service-rest-auth.md)]

## <a name="get-a-list-of-voices"></a>Obtener una lista de voces

El punto de conexión `voices/list` le permite obtener una lista completa de las voces de una región o punto de conexión en concreto.

### <a name="regions-and-endpoints"></a>Regiones y puntos de conexión

| Region | Punto de conexión |
|--------|----------|
| Este de Australia | `https://australiaeast.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Sur de Brasil | `https://brazilsouth.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Centro de Canadá | `https://canadacentral.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Centro de EE. UU. | `https://centralus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Este de Asia | `https://eastasia.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Este de EE. UU. | `https://eastus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Este de EE. UU. 2 | `https://eastus2.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Centro de Francia | `https://francecentral.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| India central | `https://centralindia.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Japón Oriental | `https://japaneast.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Centro de Corea del Sur | `https://koreacentral.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Centro-Norte de EE. UU | `https://northcentralus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Norte de Europa | `https://northeurope.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Norte de Sudáfrica | `https://southafricanorth.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Centro-sur de EE. UU. | `https://southcentralus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Sudeste de Asia | `https://southeastasia.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Sur de Reino Unido | `https://uksouth.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Centro-Oeste de EE. UU. | `https://westcentralus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Oeste de Europa | `https://westeurope.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Oeste de EE. UU. | `https://westus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Oeste de EE. UU. 2 | `https://westus2.tts.speech.microsoft.com/cognitiveservices/voices/list` |

> [!TIP]
> Las [voces en versión preliminar](language-support.md#neural-voices-in-preview) solo están disponibles en estas tres regiones: Este de EE. UU., Oeste de Europa y Sudeste de Asia.

### <a name="request-headers"></a>Encabezados de solicitud

En esta tabla se enumeran los encabezados obligatorios y opcionales para las solicitudes de texto a voz.

| Encabezado | Descripción | Obligatorio u opcional |
|--------|-------------|---------------------|
| `Ocp-Apim-Subscription-Key` | Clave de suscripción del servicio de voz. | Se necesita este encabezado, o bien `Authorization`. |
| `Authorization` | Un token de autorización precedido por la palabra `Bearer`. Para más información, consulte [Autenticación](#authentication). | Se necesita este encabezado, o bien `Ocp-Apim-Subscription-Key`. |



### <a name="request-body"></a>Cuerpo de la solicitud

No es necesario un cuerpo para las solicitudes `GET` a este punto de conexión.

### <a name="sample-request"></a>Solicitud de ejemplo

Esta solicitud solo requiere un encabezado de autorización.

```http
GET /cognitiveservices/voices/list HTTP/1.1

Host: westus.tts.speech.microsoft.com
Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY
```

### <a name="sample-response"></a>Respuesta de muestra

Esta respuesta se ha truncado para ilustrar la estructura de una respuesta.

> [!NOTE]
> La disponibilidad de voces varía según la región o el punto de conexión.

```json
[

    {
    "Name": "Microsoft Server Speech Text to Speech Voice (en-US, AriaNeural)",
    "DisplayName": "Aria",
    "LocalName": "Aria",
    "ShortName": "en-US-AriaNeural",
    "Gender": "Female",
    "Locale": "en-US",
    "StyleList": [
      "chat",
      "customerservice",
      "newscast-casual",
      "newscast-formal",
      "cheerful",
      "empathetic"
    ],
    "SampleRateHertz": "24000",
    "VoiceType": "Neural",
    "Status": "GA"
  },

    ...

    {
    "Name": "Microsoft Server Speech Text to Speech Voice (en-US, JennyMultilingualNeural)",
    "ShortName": "en-US-JennyMultilingualNeural",
    "DisplayName": "JennyMultilingualNeural",
    "LocalName": "JennyMultilingualNeural",
    "Gender": "Female",
    "Locale": "en-US",
    "SampleRateHertz": "24000",
    "VoiceType": "Neural",
    "SecondaryLocaleList": [
        "de-DE",
        "en-AU",
        "en-CA",
        "en-GB",
        "es-ES",
        "es-MX",
        "fr-CA",
        "fr-FR",
        "it-IT",
        "ja-JP",
        "ko-KR",
        "pt-BR",
        "zh-CN"
      ],
    "Status": "Preview"
    },
    
  ...
    
    {
    "Name": "Microsoft Server Speech Text to Speech Voice (ga-IE, OrlaNeural)",
    "DisplayName": "Orla",
    "LocalName": "Orla",
    "ShortName": "ga-IE-OrlaNeural",
    "Gender": "Female",
    "Locale": "ga-IE",
    "SampleRateHertz": "24000",
    "VoiceType": "Neural",
    "Status": "Preview"
  },

  ...

   {
    "Name": "Microsoft Server Speech Text to Speech Voice (zh-CN, YunxiNeural)",
    "DisplayName": "Yunxi",
    "LocalName": "云希",
    "ShortName": "zh-CN-YunxiNeural",
    "Gender": "Male",
    "Locale": "zh-CN",
    "StyleList": [
      "Calm",
      "Fearful",
      "Cheerful",
      "Disgruntled",
      "Serious",
      "Angry",
      "Sad",
      "Depressed",
      "Embarrassed"
    ],
    "SampleRateHertz": "24000",
    "VoiceType": "Neural",
    "Status": "Preview"
  },

    ...

   {
    "Name": "Microsoft Server Speech Text to Speech Voice (ar-EG, Hoda)",
    "DisplayName": "Hoda",
    "LocalName": "هدى",
    "ShortName": "ar-EG-Hoda",
    "Gender": "Female",
    "Locale": "ar-EG",
    "SampleRateHertz": "16000",
    "VoiceType": "Standard",
    "Status": "GA"
  },

...

]
```

### <a name="http-status-codes"></a>Códigos de estado HTTP

El estado HTTP de cada respuesta indica estados de corrección o error comunes.

| Código de estado HTTP | Descripción | Posible motivo |
|------------------|-------------|-----------------|
| 200 | Aceptar | La solicitud fue correcta. |
| 400 | Bad Request | Falta un parámetro requerido, está vacío o es nulo. O bien, el valor pasado a un parámetro obligatorio u opcional no es válido. Un problema común es que el encabezado sea demasiado largo. |
| 401 | No autorizado | La solicitud no está autenticada. Asegúrese de que la clave de suscripción o el token sean válidos y de la región correcta. |
| 429 | Demasiadas solicitudes | Ha superado la cuota o la tasa de solicitudes permitidas para su suscripción. |
| 502 | Puerta de enlace incorrecta    | Problema de red o de servidor. Podría indicar también encabezados no válidos. |


## <a name="convert-text-to-speech"></a>Conversión de texto a voz

El punto de conexión `v1` le permite convertir texto a voz usando [lenguaje de marcado de síntesis de voz (SSML)](speech-synthesis-markup.md).

### <a name="regions-and-endpoints"></a>Regiones y puntos de conexión

Estas regiones son compatibles con la conversión de texto a voz mediante la API REST. Asegúrese de que selecciona el punto de conexión que coincida con la región de su suscripción.

[!INCLUDE [](../../../includes/cognitive-services-speech-service-endpoints-text-to-speech.md)]

### <a name="request-headers"></a>Encabezados de solicitud

En esta tabla se enumeran los encabezados obligatorios y opcionales para las solicitudes de texto a voz.

| Encabezado | Descripción | Obligatorio u opcional |
|--------|-------------|---------------------|
| `Authorization` | Un token de autorización precedido por la palabra `Bearer`. Para más información, consulte [Autenticación](#authentication). | Obligatorio |
| `Content-Type` | Especifica el tipo de contenido para el texto proporcionado. Valor aceptable: `application/ssml+xml`. | Obligatorio |
| `X-Microsoft-OutputFormat` | Especifica el formato de salida del audio. Para obtener una lista completa de los valores aceptados, consulte [salidas de audio](#audio-outputs). | Obligatorio |
| `User-Agent` | Nombre de la aplicación. El valor proporcionado debe tener menos de 255 caracteres. | Obligatorio |

### <a name="audio-outputs"></a>Salidas de audio

Esta es una lista de formatos de audio admitidos que se envían en cada solicitud como encabezado `X-Microsoft-OutputFormat`. Cada uno de ellos incorpora una velocidad de bits y el tipo de codificación. El servicio de voz admite salidas de audio de 24 kHz, 16 kHz y 8 kHz.

```output
raw-16khz-16bit-mono-pcm            riff-16khz-16bit-mono-pcm
raw-24khz-16bit-mono-pcm            riff-24khz-16bit-mono-pcm
raw-48khz-16bit-mono-pcm            riff-48khz-16bit-mono-pcm
raw-8khz-8bit-mono-mulaw            riff-8khz-8bit-mono-mulaw
raw-8khz-8bit-mono-alaw             riff-8khz-8bit-mono-alaw
audio-16khz-32kbitrate-mono-mp3     audio-16khz-64kbitrate-mono-mp3
audio-16khz-128kbitrate-mono-mp3    audio-24khz-48kbitrate-mono-mp3
audio-24khz-96kbitrate-mono-mp3     audio-24khz-160kbitrate-mono-mp3
audio-48khz-96kbitrate-mono-mp3     audio-48khz-192kbitrate-mono-mp3
raw-16khz-16bit-mono-truesilk       raw-24khz-16bit-mono-truesilk
webm-16khz-16bit-mono-opus          webm-24khz-16bit-mono-opus
ogg-16khz-16bit-mono-opus           ogg-24khz-16bit-mono-opus
ogg-48khz-16bit-mono-opus
```

> [!NOTE]
> Si la voz y el formato de salida seleccionados tienen velocidades de bits diferentes, se vuelve a muestrear el audio según sea necesario.
> ogg-24khz-16bit-mono-opus se puede descodificar con el [códec opus](https://opus-codec.org/downloads/)

### <a name="request-body"></a>Cuerpo de la solicitud

El cuerpo de cada solicitud `POST` se envía como [lenguaje de marcado de síntesis de voz (SSML)](speech-synthesis-markup.md). SSML le permite elegir la voz y el idioma de la voz sintetizada que devuelve el servicio de texto a voz. Para ver una lista completa de voces compatibles, consulte [compatibilidad con idiomas](language-support.md#text-to-speech).

> [!NOTE]
> Si usa una voz personalizada, el cuerpo de una solicitud puede enviarse como texto sin formato (ASCII o UTF-8).

### <a name="sample-request"></a>Solicitud de ejemplo

Esta solicitud HTTP utiliza SSML para especificar el idioma y la voz. Si la longitud del cuerpo es larga y el audio resultante supera los 10 minutos, se trunca a 10 minutos. En otras palabras, la longitud del audio no puede superar los 10 minutos.

```http
POST /cognitiveservices/v1 HTTP/1.1

X-Microsoft-OutputFormat: raw-24khz-16bit-mono-pcm
Content-Type: application/ssml+xml
Host: westus.tts.speech.microsoft.com
Content-Length: 225
Authorization: Bearer [Base64 access_token]

<speak version='1.0' xml:lang='en-US'><voice xml:lang='en-US' xml:gender='Female'
    name='en-US-AriaNeural'>
        Microsoft Speech Service Text-to-Speech API
</voice></speak>
```

### <a name="http-status-codes"></a>Códigos de estado HTTP

El estado HTTP de cada respuesta indica estados de corrección o error comunes.

| Código de estado HTTP | Descripción | Posible motivo |
|------------------|-------------|-----------------|
| 200 | Aceptar | La solicitud es correcta; el cuerpo de la respuesta es un archivo de audio. |
| 400 | Bad Request | Falta un parámetro requerido, está vacío o es nulo. O bien, el valor pasado a un parámetro obligatorio u opcional no es válido. Un problema común es que el encabezado sea demasiado largo. |
| 401 | No autorizado | La solicitud no está autenticada. Asegúrese de que la clave de suscripción o el token sean válidos y de la región correcta. |
| 415 | Tipo de medio no compatible | Es posible que se haya proporcionado el `Content-Type` incorrecto. `Content-Type` se debe establecer en `application/ssml+xml`. |
| 429 | Demasiadas solicitudes | Ha superado la cuota o la tasa de solicitudes permitidas para su suscripción. |
| 502 | Puerta de enlace incorrecta    | Problema de red o de servidor. Podría indicar también encabezados no válidos. |

Si el estado HTTP es `200 OK`, el cuerpo de la respuesta contiene un archivo de audio en el formato solicitado. Este archivo se puede reproducir mientras se transfiere o guardarse en un búfer o en un archivo.

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una cuenta de Azure gratuita](https://azure.microsoft.com/free/cognitive-services/)
- [Síntesis asincrónica para audio de formato largo](./long-audio-api.md)
- [Introducción a Voz personalizada](how-to-custom-voice.md)
