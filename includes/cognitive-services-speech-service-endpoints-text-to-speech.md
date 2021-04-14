---
author: wolfma61
ms.service: cognitive-services
ms.topic: include
ms.date: 05/06/2019
ms.author: wolfma
ms.openlocfilehash: 832cc1d4f9ec3cec4ada6e964e3ab2f6f023dd41
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106554322"
---
### <a name="neural-and-standard-voices"></a>Voces neuronales y estándar

Utilice esta tabla para determinar la **disponibilidad de voces neuronales y estándar** según la región o el punto de conexión:

| Region | Punto de conexión |
|--------|----------|
| Este de Australia | `https://australiaeast.tts.speech.microsoft.com/cognitiveservices/v1` |
| Sur de Brasil | `https://brazilsouth.tts.speech.microsoft.com/cognitiveservices/v1` |
| Centro de Canadá | `https://canadacentral.tts.speech.microsoft.com/cognitiveservices/v1` |
| Centro de EE. UU. | `https://centralus.tts.speech.microsoft.com/cognitiveservices/v1` |
| Este de Asia | `https://eastasia.tts.speech.microsoft.com/cognitiveservices/v1` |
| Este de EE. UU. | `https://eastus.tts.speech.microsoft.com/cognitiveservices/v1` |
| Este de EE. UU. 2 | `https://eastus2.tts.speech.microsoft.com/cognitiveservices/v1` |
| Centro de Francia | `https://francecentral.tts.speech.microsoft.com/cognitiveservices/v1` |
| India central | `https://centralindia.tts.speech.microsoft.com/cognitiveservices/v1` |
| Japón Oriental | `https://japaneast.tts.speech.microsoft.com/cognitiveservices/v1` |
| Japón Occidental | `https://japanwest.tts.speech.microsoft.com/cognitiveservices/v1` |
| Centro de Corea del Sur | `https://koreacentral.tts.speech.microsoft.com/cognitiveservices/v1` |
| Centro-Norte de EE. UU | `https://northcentralus.tts.speech.microsoft.com/cognitiveservices/v1` |
| Norte de Europa | `https://northeurope.tts.speech.microsoft.com/cognitiveservices/v1` |
| Centro-sur de EE. UU. | `https://southcentralus.tts.speech.microsoft.com/cognitiveservices/v1` |
| Sudeste de Asia | `https://southeastasia.tts.speech.microsoft.com/cognitiveservices/v1` |
| Sur de Reino Unido | `https://uksouth.tts.speech.microsoft.com/cognitiveservices/v1` |
| Centro-Oeste de EE. UU. | `https://westcentralus.tts.speech.microsoft.com/cognitiveservices/v1` |
| Oeste de Europa | `https://westeurope.tts.speech.microsoft.com/cognitiveservices/v1` |
| Oeste de EE. UU. | `https://westus.tts.speech.microsoft.com/cognitiveservices/v1` |
| Oeste de EE. UU. 2 | `https://westus2.tts.speech.microsoft.com/cognitiveservices/v1` |

> [!TIP]
> Las [voces en versión preliminar](../articles/cognitive-services/Speech-Service/language-support.md#neural-voices-in-preview) solo están disponibles en estas tres regiones: Este de EE. UU., Oeste de Europa y Sudeste de Asia.

### <a name="custom-voices"></a>Voces personalizadas

Si ha creado una fuente de voz personalizada, use el punto de conexión que ha creado. También puede utilizar los puntos de conexión que se indican a continuación; para ello, reemplace `{deploymentId}` por el identificador de implementación para el modelo de voz.

| Region | Punto de conexión |
|--------|----------|
| Este de Australia | `https://australiaeast.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Sur de Brasil | `https://brazilsouth.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Centro de Canadá | `https://canadacentral.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Centro de EE. UU. | `https://centralus.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Este de Asia | `https://eastasia.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Este de EE. UU. | `https://eastus.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Este de EE. UU. 2 | `https://eastus2.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Centro de Francia | `https://francecentral.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| India central | `https://centralindia.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Japón Oriental | `https://japaneast.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Japón Occidental | `https://japanwest.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Centro de Corea del Sur | `https://koreacentral.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Centro-Norte de EE. UU | `https://northcentralus.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Norte de Europa | `https://northeurope.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Centro-sur de EE. UU. | `https://southcentralus.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Sudeste de Asia | `https://southeastasia.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Sur de Reino Unido | `https://uksouth.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Oeste de Europa | `https://westeurope.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Oeste de EE. UU. | `https://westus.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Oeste de EE. UU. 2 | `https://westus2.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |

### <a name="custom-neural-voice"></a>Voz neuronal personalizada

En la tabla siguiente se describe la compatibilidad regional con las características de Voz neuronal personalizada.

| Característica | Regiones admitidas |
|---|---|
| Hospedaje de modelos de voz | Este de EE. UU., Oeste de EE. UU. 2, Centro-sur de EE. UU., Sudeste Asiático, Sur de Reino Unido, Oeste de Europa, Este de Australia |
| Caracteres en tiempo real | Este de EE. UU., Oeste de EE. UU. 2, Centro-sur de EE. UU., Sudeste Asiático, Sur de Reino Unido, Oeste de Europa, Este de Australia |
| Caracteres de audio largos | Este de EE. UU., Oeste de Europa, Sur de Reino Unido, Sudeste Asiático, Centro de la India |
| Entrenamiento neuronal personalizado | Este de EE. UU., Sur de Reino Unido |

### <a name="long-audio-api"></a>Long audio API

Long Audio API está disponible en varias regiones con puntos de conexión únicos.

| Region | Punto de conexión |
|--------|----------|
| Este de EE. UU. | `https://eastus.customvoice.api.speech.microsoft.com` |
| India central | `https://centralindia.customvoice.api.speech.microsoft.com` |
| Sudeste de Asia | `https://southeastasia.customvoice.api.speech.microsoft.com` |
| Sur de Reino Unido | `https://uksouth.customvoice.api.speech.microsoft.com` |
| Oeste de Europa | `https://westeurope.customvoice.api.speech.microsoft.com` |
