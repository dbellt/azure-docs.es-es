---
title: Introducción a la grabación de llamadas con Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Proporciona una introducción a la característica y las API de grabación de llamadas.
author: joseys
manager: anvalent
services: azure-communication-services
ms.author: joseys
ms.date: 06/30/2021
ms.topic: overview
ms.custom: references_regions
ms.service: azure-communication-services
ms.openlocfilehash: 292210c12589bed6075378ba4fd38153c4fe10f1
ms.sourcegitcommit: 98308c4b775a049a4a035ccf60c8b163f86f04ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113105352"
---
# <a name="calling-recording-overview"></a>Introducción a la grabación de llamadas

> [!NOTE]
> Muchos países y estados tienen leyes y regulaciones que se aplican a la grabación de llamadas RTC, de voz y de vídeo, que a menudo requieren que los usuarios den su consentimiento para la grabación de sus comunicaciones. Es su responsabilidad usar las funcionalidades de grabación de llamadas de acuerdo con la ley. Debe obtener el consentimiento de las partes que integran la comunicación grabada de forma que cada participante cumpla las leyes aplicables.

> [!NOTE]
> La normativa sobre el mantenimiento de datos personales requiere la capacidad de exportar datos de usuario. Para cumplir con estos requisitos, la grabación de archivos de metadatos incluye el identificador participantId de cada participante de la llamada en la matriz `participants`. Puede hacer referencia cruzada a los MRI de la matriz `participants` con las identidades de usuario internas para identificar a los participantes de una llamada. A continuación se proporciona un ejemplo de un archivo de metadatos de grabación como referencia.

> [!NOTE]
> La grabación de llamadas solo está disponible actualmente para los recursos de Communication Services creados en la región de EE. UU.

Grabación de llamadas proporciona un conjunto de API para iniciar, detener, pausar y reanudar la grabación. Se puede acceder a estas API desde la lógica de negocios del lado servidor o a través de eventos desencadenados por acciones del usuario. La salida de los elementos multimedia grabados está en formato de audio y vídeo MP4, que es el mismo formato que usa Teams para grabar los elementos multimedia. Las notificaciones relacionadas con los elementos multimedia y los metadatos se emiten a través de Event Grid. Las grabaciones se almacenan durante 48 horas en el almacenamiento temporal integrado para su recuperación y traslado a una solución de almacenamiento a largo plazo de su elección. 

![Diagrama de concepto de la grabación de llamadas](../media/call-recording-concept.png)

## <a name="media-output-types"></a>Tipos de salida de los elementos multimedia
La grabación de llamadas admite actualmente el formato de salida MP4 de audio y vídeo mixto. El elemento multimedia de salida coincide con las grabaciones de reuniones que produce la característica de grabación de Microsoft Teams.

| Tipo de canal | Formato de contenido | Vídeo | Audio |
| :----------- | :------------- | :---- | :--------------------------- |
| audioVideo | mp4 | Vídeo con formato 1920x1080 a 8 FPS de todos los participantes con una organización de iconos predeterminada | Audio mixto mp4a de 16 kHz de todos los participantes |


## <a name="run-time-control-apis"></a>API de control de un entorno de ejecución
Las API de control de un entorno de ejecución se pueden usar para administrar la grabación mediante desencadenadores internos de lógica de negocios, como una aplicación que crea una llamada grupal y graba la conversación, o a partir de una acción desencadenada por el usuario que indica a la aplicación de servidor que inicie la grabación. Las API de grabación de llamadas son [API fuera de llamada](./call-automation-apis.md#out-of-call-apis) y usan el valor de `serverCallId` para iniciar la grabación. Al crear una llamada, se devuelve el valor de `serverCallId` mediante el evento `Microsoft.Communication.CallLegStateChanged` una vez establecida una llamada. El valor de `serverCallId` se puede encontrar en el campo `data.serverCallId`. Consulte [Inicio rápido de la API de grabación de llamadas](../../quickstarts/voice-video-calling/call-recording-sample.md) para obtener información sobre cómo recuperar el valor de `serverCallId` desde el SDK de cliente de llamadas. Cuando se inicia la grabación, se devuelve un elemento `recordingOperationId`, que luego se usa para las operaciones de seguimiento, como pausar y reanudar.   

| Operación                            | Opera en            | Comentarios                       |
| :-------------------- | :--------------------- | :----------------------------- |
| Iniciar grabación       | `serverCallId`         | Devuelve `recordingOperationId` | 
| Obtener el estado de la grabación   | `recordingOperationId` | Devuelve `recordingState`       | 
| Pausar la grabación       | `recordingOperationId` | Pausar y reanudar la grabación de llamadas le permite omitir la grabación de una parte de una llamada o reunión, y reanudar la grabación en un solo archivo. | 
| Reanudar la grabación      | `recordingOperationId` | Reanuda una operación de grabación en pausa. El contenido se incluye en el mismo archivo que el contenido anterior a la pausa. | 
| Detener grabación        | `recordingOperationId` | Detiene la grabación e inicia el procesamiento multimedia final para la descarga de archivos. | 


## <a name="event-grid-notifications"></a>Notificaciones de Event Grid

> Azure Communication Services proporciona almacenamiento de elementos multimedia a corto plazo para las grabaciones. **Exporte cualquier contenido grabado que desee conservar en un plazo de 48 horas**. Después de 48 horas, las grabaciones ya no estarán disponibles.

Se publica una notificación `Microsoft.Communication.RecordingFileStatusUpdated` de Event Grid cuando una grabación está lista para su recuperación, normalmente unos minutos después de que se haya completado el proceso de grabación (por ejemplo, finalizó la reunión o se detuvo la grabación). Las notificaciones de eventos de grabación incluyen los elementos `contentLocation` y `metadataLocation`, que se usan para recuperar los elementos multimedia grabados y un archivo de metadatos de la grabación.

### <a name="notification-schema-reference"></a>Referencia del esquema de notificación
```
{
    "id": string, // Unique guid for event
    "topic": string, // Azure Communication Services resource id
    "subject": string, // /recording/call/{call-id}
    "data": {
        "recordingStorageInfo": {
            "recordingChunks": [
                {
                    "documentId": string, // Document id for retrieving from storage
                    "index": int, // Index providing ordering for this chunk in the entire recording
                    "endReason": string, // Reason for chunk ending: "SessionEnded", "ChunkMaximumSizeExceeded”, etc.
                    "metadataLocation": <string>, // url of the metadata for this chunk
                    "contentLocation": <string>   // url of the mp4, mp3, or wav for this chunk
                }
            ]
        },
        "recordingStartTime": string, // ISO 8601 date time for the start of the recording
        "recordingDurationMs": int, // Duration of recording in milliseconds
        "sessionEndReason": string // Reason for call ending: "CallEnded", "InitiatorLeft", etc.
    },
    "eventType": string, // "Microsoft.Communication.RecordingFileStatusUpdated"
    "dataVersion": string, // "1.0"
    "metadataVersion": string, // "1"
    "eventTime": string // ISO 8601 date time for when the event was created
}
```

## <a name="next-steps"></a>Pasos siguientes
Consulte [Inicio rápido de la API de grabación de llamadas](../../quickstarts/voice-video-calling/call-recording-sample.md) para más información.

Más información sobre las [API de automatización de llamadas](./call-automation-apis.md).
