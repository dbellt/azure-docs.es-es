---
title: Introducción a la grabación de llamadas con Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Proporciona una introducción a la característica y las API de grabación de llamadas.
author: joseys
manager: anvalent
services: azure-communication-services
ms.author: joseys
ms.date: 04/13/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: df9638588b4d677a7ceb80bafbe7157bb5c2df42
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107730382"
---
# <a name="calling-recording-overview"></a>Introducción a la grabación de llamadas

[!INCLUDE [Private Preview Notice](../../includes/private-preview-include.md)]

> [!NOTE]
> Muchos países y estados tienen leyes y regulaciones que se aplican a la grabación de llamadas RTC, de voz y de vídeo, que a menudo requieren que los usuarios den su consentimiento para la grabación de sus comunicaciones. Es su responsabilidad usar las funcionalidades de grabación de llamadas de acuerdo con la ley. Debe obtener el consentimiento de las partes que integran la comunicación grabada de forma que cada participante cumpla las leyes aplicables.

> [!NOTE]
> La normativa sobre el mantenimiento de datos personales requiere la capacidad de exportar datos de usuario. Para cumplir con estos requisitos, la grabación de archivos de metadatos incluye el identificador participantId de cada participante de la llamada en la matriz `participants`. Puede hacer referencia cruzada a los MRI de la matriz `participants` con las identidades de usuario internas para identificar a los participantes de una llamada. A continuación se proporciona un ejemplo de un archivo de metadatos de grabación como referencia.

Grabación de llamadas proporciona un conjunto de API para iniciar, detener, pausar y reanudar la grabación. Se puede acceder a estas API desde la lógica de negocios del lado servidor o a través de eventos desencadenados por acciones del usuario. La salida de los elementos multimedia grabados está en formato `MP4 Audio+Video`, que es el mismo formato que usa Teams para grabar. Las notificaciones relacionadas con los elementos multimedia y los metadatos se emiten a través de Event Grid. Las grabaciones se almacenan durante 48 horas en el almacenamiento temporal integrado para su recuperación y traslado a una solución de almacenamiento a largo plazo de su elección. 

## <a name="run-time-control-apis"></a>API de control de un entorno de ejecución
Las API de control de un entorno de ejecución se pueden usar para administrar la grabación mediante desencadenadores internos de lógica de negocios, como una aplicación que crea una llamada grupal y graba la conversación, o a partir de una acción desencadenada por el usuario que indica a la aplicación de servidor que inicie la grabación. En ambos escenarios, se necesita un `<conversation-id>` para grabar una reunión o llamada específica. 

#### <a name="getting-conversation-id-from-a-server-initiated-call"></a>Obtención del identificador de conversación de una llamada iniciada por el servidor

Se devuelve `ConversationId` mediante el evento `Microsoft.Communication.CallLegStateChanged`. Esta notificación de eventos se emite una vez establecida una llamada. Se puede encontrar en el campo `data.ConversationId`. Este valor se puede usar directamente como parámetro `{conversationId}` en las API de control de un entorno de ejecución:
```
      {
        "id": null,
        "topic": null,
        "subject": "callLeg/<callLegId>/callState",
        "data": {
---->       "ConversationId": "<conversation-id>",    <----
            "CallLegId": "<callLegId>",
            "CallState": "Established"
        },
        "eventType": "Microsoft.Communication.CallLegStateChanged",
        "eventTime": "2021-04-14T16:32:34.1115003Z",
        "metadataVersion": null,
        "dataVersion": null
    }
```
                                                            
#### <a name="getting-the-conversation-id-from-a-user-triggered-event-on-the-client"></a>Obtención del identificador de conversación a partir de un evento desencadenado por el usuario en el cliente

En la biblioteca `@azure/communication-calling` de JavaScript, después de establecer una invocación de llamada `let result = call.info.getConversationUrl()` para obtener `conversationUrl`, **Base64Url codifica `conversationUrl` para obtener `{conversationId}` para su uso en las API de control de un entorno de ejecución**. La codificación se puede realizar en el cliente antes de enviar el evento al servidor o en el lado del servidor.

Tenga en cuenta que `conversationUrl` *debe* estar codificado en Base64Url, no debe confundirse solo con la codificación Base64 (es decir, btoa).                                                            

### <a name="start-recording"></a>Iniciar la grabación

#### <a name="request"></a>Solicitud

**HTTP**
<!-- {
  "blockType": "request",
  "name": "start-recording"
}-->
```
POST /conversations/{conversationId}/Recordings
Content-Type: application/json

{
  "operationContext": "string", // developer provided string for correlation context on each operation
  "recordingStateCallbackUri": "string"
}
```
**SDK de C#**
<!-- {
  "blockType": "request",
  "name": "start-recording"
}-->
```C#
string connectionString = "YOUR_CONNECTION_STRING";
ConversationClient conversationClient = new ConversationClient(connectionString);

/// start call recording
StartRecordingResponse startRecordingResponse = await conversationClient.StartRecordingAsync(
    conversationId: "<conversation-id>"
    operationContext: "<operation-context>", /// developer provided string for correlation context on each operation
    recordingStateCallbackUri: "<recording-state-callback-uri>").ConfigureAwait(false);

string recordingId = startRecordingResponse.RecordingId;
```

#### <a name="response"></a>Response

**HTTP**
<!-- {
  "blockType": "response",
  "truncated": true,
} -->

```http
HTTP/1.1 200 Success
Content-Type: application/json

{
  "recordingId": "string"
}
```
```
HTTP/1.1 400 Bad request
Content-Type: application/json

{
  "code": "string",
  "message": "string",
  "target": "string",
  "details": [
    null
  ]
}
```
```
HTTP/1.1 404 Not found
Content-Type: application/json

{
  "code": "string",
  "message": "string",
  "target": "string",
  "details": [
    null
  ]
}
```
```
HTTP/1.1 500    Internal server error
Content-Type: application/json

{
  "code": "string",
  "message": "string",
  "target": "string",
  "details": [
    null
  ]
}
```

### <a name="get-call-recording-state"></a>Obtención del estado de grabación de las llamadas

#### <a name="request"></a>Solicitud

**HTTP**
<!-- {
  "blockType": "request",
  "name": "get-recording-state"
}-->
```http
GET /conversations/{conversationId}/recordings/{recordingId}
Content-Type: application/json

{
}
```
**SDK de C#**
<!-- {
  "blockType": "request",
  "name": "start-recording"
}-->
```C#
string connectionString = "YOUR_CONNECTION_STRING";
ConversationClient conversationClient = new ConversationClient(connectionString);

/// get recording state
GetCallRecordingStateResponse recordingState = await conversationClient.GetRecordingStateAsync(
    conversationId: "<conversation-id>",
    recordingId: <recordingId>).ConfigureAwait(false);
```
#### <a name="response"></a>Response

**HTTP**
<!-- {
  "blockType": "response",
  "truncated": true,
} -->

```http
HTTP/1.1 200 Success
Content-Type: application/json

{
  "recordingState": "active"
}
```
```
HTTP/1.1 400 Bad request
Content-Type: application/json

{
  "code": "string",
  "message": "string",
  "target": "string",
  "details": [
    null
  ]
}
```
```
HTTP/1.1 500    Internal server error
Content-Type: application/json

{
  "code": "string",
  "message": "string",
  "target": "string",
  "details": [
    null
  ]
}
```

### <a name="stop-recording"></a>Detener grabación
#### <a name="request"></a>Solicitud
**HTTP**
<!-- {
  "blockType": "request",
  "name": "stop-recording"
}-->
```
DELETE /conversations/{conversationId}/recordings/{recordingId}
Content-Type: application/json

{
  "operationContext": "string" // developer provided string for correlation context on each operation
}
```
**SDK de C#**
<!-- {
  "blockType": "request",
  "name": "start-recording"
}-->
```C#
string connectionString = "YOUR_CONNECTION_STRING";
ConversationClient conversationClient = new ConversationClient(connectionString);

/// stop recording
StopRecordingResponse response = conversationClient.StopRecordingAsync(
    conversationId: "<conversation-id>",
    recordingId: <recordingId>,
    operationContext: "<operation-context>").ConfigureAwait(false);
```
#### <a name="response"></a>Response
**HTTP**
<!-- {
  "blockType": "response",
  "truncated": true,
} -->

```http
HTTP/1.1 200 Success
Content-Type: application/json

{
}
```
```
HTTP/1.1 400 Bad request
Content-Type: application/json

{
  "code": "string",
  "message": "string",
  "target": "string",
  "details": [
    null
  ]
}
```
```
HTTP/1.1 500    Internal server error
Content-Type: application/json

{
  "code": "string",
  "message": "string",
  "target": "string",
  "details": [
    null
  ]
}
```

### <a name="pause-recording"></a>Pausar grabación
Pausar y reanudar la grabación de llamadas le permite omitir la grabación de una parte de una llamada o reunión, y reanudar la grabación en un solo archivo. 
#### <a name="request"></a>Solicitud
**HTTP**
<!-- {
  "blockType": "request",
  "name": "pause-recording"
}-->
```
POST /conversations/{conversationId}/recordings/{recordingId}/Pause
Content-Type: application/json

{
  "operationContext": "string" // developer provided string for correlation context on each operation
}
```
**SDK de C#**
<!-- {
  "blockType": "request",
  "name": "start-recording"
}-->
```C#
string connectionString = "YOUR_CONNECTION_STRING";
ConversationClient conversationClient = new ConversationClient(connectionString);

/// pause recording
PauseRecordingResponse response = conversationClient.PauseRecordingAsync(
    conversationId: "<conversation-id>",
    recordingId: <recordingId>,
    operationContext: "<operation-context>").ConfigureAwait(false);
```
#### <a name="response"></a>Response
**HTTP**
<!-- {
  "blockType": "response",
  "truncated": true,
} -->

```http
HTTP/1.1 200 Success
Content-Type: application/json

{
}
```
```
HTTP/1.1 400 Bad request
Content-Type: application/json

{
  "code": "string",
  "message": "string",
  "target": "string",
  "details": [
    null
  ]
}
```
```
HTTP/1.1 500    Internal server error
Content-Type: application/json

{
  "code": "string",
  "message": "string",
  "target": "string",
  "details": [
    null
  ]
}
```

### <a name="resume-recording"></a>Reanudación de la grabación
#### <a name="request"></a>Solicitud
**HTTP**
<!-- {
  "blockType": "request",
  "name": "resume-recording"
}-->
```
POST /conversations/{conversationId}/recordings/{recordingId}/Resume
Content-Type: application/json

{
  "operationContext": "string" // developer provided string for correlation context on each operation
}
```
**SDK de C#**
<!-- {
  "blockType": "request",
  "name": "start-recording"
}-->
```C#
string connectionString = "YOUR_CONNECTION_STRING";
ConversationClient conversationClient = new ConversationClient(connectionString);

/// resume recording
ResumeRecordingResponse response = conversationClient.ResumeRecordingAsync(
    conversationId: "<conversation-id>",
    recordingId: <recordingId>,
    operationContext: "<operation-context>").ConfigureAwait(false);
```
#### <a name="response"></a>Response
**HTTP**
<!-- {
  "blockType": "response",
  "truncated": true,
} -->

```http
HTTP/1.1 200 Success
Content-Type: application/json

{
}
```
```
HTTP/1.1 400 Bad request
Content-Type: application/json

{
  "code": "string",
  "message": "string",
  "target": "string",
  "details": [
    null
  ]
}
```
```
HTTP/1.1 500    Internal server error
Content-Type: application/json

{
  "code": "string",
  "message": "string",
  "target": "string",
  "details": [
    null
  ]
}
```

## <a name="media-output-types"></a>Tipos de salida de los elementos multimedia
La grabación de llamadas admite actualmente el formato de salida MP4 de audio y vídeo mixto. El elemento multimedia de salida coincide con las grabaciones de reuniones que produce la característica de grabación de Microsoft Teams.

| Tipo de canal | Formato de contenido | Vídeo | Audio |
| :----------- | :------------- | :---- | :--------------------------- |
| audioVideo | mp4 | Vídeo con formato 1920x1080 a 8 FPS de todos los participantes con una organización de iconos predeterminada | Audio mixto mp4a de 16 kHz de todos los participantes |

## <a name="event-grid-notifications"></a>Notificaciones de Event Grid
Una notificación `Microsoft.Communication.RecordingFileStatusUpdated` de Event Grid se publica cuando una grabación está lista para su recuperación, normalmente de 1 a 2 minutos después de que se haya completado el proceso de grabación (por ejemplo, la reunión finalizó o la grabación se detuvo). Las notificaciones de eventos de grabación incluyen un identificador de documento que se puede usar para recuperar elementos multimedia grabados y un archivo de metadatos de grabación:

- <Azure_Communication_Service_Endpoint>/recording/download/{documentId}
- <Azure_Communication_Service_Endpoint>/recording/download/{documentId}/metadata

Puede encontrar código de ejemplo para controlar las notificaciones de Event Grid y descargar archivos de registro y metadatos [aquí](../../quickstarts/voice-video-calling/download-recording-file-sample.md). 

### <a name="notification-schema"></a>Esquema de la notificación
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
## <a name="file-download"></a>Descarga de archivos

> Azure Communication Services proporciona almacenamiento de elementos multimedia a corto plazo para las grabaciones. **Exporte cualquier contenido grabado que desee conservar en un plazo de 48 horas**. Después de 48 horas, las grabaciones ya no estarán disponibles.

### <a name="download-recording"></a>Descarga de grabación
#### <a name="request"></a>Solicitud
**HTTP**
<!-- {
  "blockType": "request",
  "name": "download-recording"
}-->
```http
GET /recording/download/{documentId}
Content-Type: application/json

{
}
```
#### <a name="response"></a>Response
**HTTP**
<!-- {
  "blockType": "response",
  "truncated": true,
} -->

```
HTTP/1.1 200 Success
Content-Type: video/mp4

{
string // Recording file bytes
}
```
```
HTTP/1.1 400 Bad request
Content-Type: application/json

{
  "code": "string",
  "message": "string",
  "target": "string",
  "details": [
    null
  ]
}
```
```
HTTP/1.1 500    Internal server error
Content-Type: application/json

{
  "code": "string",
  "message": "string",
  "target": "string",
  "details": [
    null
  ]
}
```
### <a name="download-recording-metadata"></a>Descarga de metadatos de grabación
#### <a name="request"></a>Solicitud
**HTTP**
<!-- {
  "blockType": "request",
  "name": "download-recording-metadata"
}-->
```http
GET /recording/download/{documentId}/metadata
Content-Type: application/json

{
}
```
#### <a name="response"></a>Response
**HTTP**
<!-- {
  "blockType": "response",
  "truncated": true,
} -->

```http
HTTP/1.1 200 Success
Content-Type: application/json

{
  "resourceId": "string",
  "callId": "string",
  "chunkDocumentId": "string",
  "chunkIndex": 0,
  "chunkStartTime": "string",
  "chunkDuration": 0,
  "pauseResumeIntervals": [
    {
      "startTime": "string",
      "duration": 0
    }
  ],
  "recordingInfo": {
    "contentType": "string",
    "channelType": "string",
    "format": "string",
    "audioConfiguration": {
      "sampleRate": "string",
      "bitRate": 0,
      "channels": 0
    },
    "videoConfiguration": {
      "longerSideLength": 0,
      "shorterSideLength": 0,
      "framerate": 0,
      "bitRate": 0
    }
  },
  "participants": [
    {
      "participantId": "string"
    }
  ]
}
```
```
HTTP/1.1 400 Bad request
Content-Type: application/json

{
  "code": "string",
  "message": "string",
  "target": "string",
  "details": [
    null
  ]
}
```
```
HTTP/1.1 500    Internal server error
Content-Type: application/json

{
  "code": "string",
  "message": "string",
  "target": "string",
  "details": [
    null
  ]
}
```
