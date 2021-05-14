---
author: v-jaswel
ms.service: cognitive-services
ms.topic: include
ms.date: 10/09/2020
ms.author: v-jawe
ms.openlocfilehash: 6df6fc7e4c2cc7a1271d3ca01e34fe6802e7abde
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109718946"
---
En este inicio rápido, aprenderá a convertir texto a voz mediante el servicio de voz y cURL.

Para obtener una visión general de los conceptos de la conversión de texto a voz, consulte el artículo en el que se proporciona [información general](../../../text-to-speech.md).

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por sentado que tiene una cuenta de Azure y una suscripción al servicio de voz. Si no dispone de una cuenta y una suscripción, [pruebe el servicio de voz de forma gratuita](../../../overview.md#try-the-speech-service-for-free).

## <a name="convert-text-to-speech"></a>Conversión de texto en voz

En el símbolo del sistema, ejecute el siguiente comando. Tendrá que insertar los siguientes valores en el comando.
- Clave de suscripción del servicio de voz.
- Región del servicio de voz.

Es posible que también quiera cambiar los siguientes valores.
- El valor del encabezado `X-Microsoft-OutputFormat`, que controla el formato de la salida de audio. Puede encontrar una lista de formatos de salida de audio compatibles en la [referencia de la API REST de texto a voz](../../../rest-text-to-speech.md#audio-outputs).
- Voz de salida. Para obtener una lista de las voces disponibles para el punto de conexión de Voz, consulte la sección siguiente.
- Archivo de salida. En este ejemplo, la respuesta del servidor se dirige a un archivo denominado `output.wav`.

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/text-to-speech.sh":::

## <a name="list-available-voices-for-your-speech-endpoint"></a>Enumeración de las voces disponibles para el punto de conexión de Voz

Para enumerar las voces disponibles para el punto de conexión de Voz, ejecute el siguiente comando.

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/get-voices.sh" id="request":::

Debería recibir una respuesta similar a la siguiente.

```http
[
    {
        "Name": "Microsoft Server Speech Text to Speech Voice (ar-EG, Hoda)",
        "DisplayName": "Hoda",
        "LocalName": "هدى",
        "ShortName": "ar-EG-Hoda",
        "Gender": "Female",
        "Locale": "ar-EG",
        "SampleRateHertz": "16000",
        "VoiceType": "Standard"
    },
    {
        "Name": "Microsoft Server Speech Text to Speech Voice (ar-SA, Naayf)",
        "DisplayName": "Naayf",
        "LocalName": "نايف",
        "ShortName": "ar-SA-Naayf",
        "Gender": "Male",
        "Locale": "ar-SA",
        "SampleRateHertz": "16000",
        "VoiceType": "Standard"
    },
    {
        "Name": "Microsoft Server Speech Text to Speech Voice (bg-BG, Ivan)",
        "DisplayName": "Ivan",
        "LocalName": "Иван",
        "ShortName": "bg-BG-Ivan",
        "Gender": "Male",
        "Locale": "bg-BG",
        "SampleRateHertz": "16000",
        "VoiceType": "Standard"
    },
    {
        // This response is truncated. The response will include 
        // a complete list of supported languages and specific 
        // details like short name, gender, etc. 
    }
]
```