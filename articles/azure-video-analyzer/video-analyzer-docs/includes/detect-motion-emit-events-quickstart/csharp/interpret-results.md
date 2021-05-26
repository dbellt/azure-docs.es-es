---
author: fvneerden
ms.service: azure-video-analyzer
ms.topic: include
ms.date: 05/05/2021
ms.author: faneerde
ms.openlocfilehash: e27039f9682a363a101111b84ecc5bd94cf922cb
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388535"
---
Cuando se ejecuta la canalización en directo, los resultados del nodo procesador del detector de movimiento recorren el nodo receptor de mensajes de IoT Hub hasta llegar al centro de IoT. Los mensajes que aparecen en la ventana de **salida** de Visual Studio Code contienen una sección **body** y una sección **applicationProperties**. Para más información, consulte [Creación y lectura de mensajes de IoT Hub](../../../../../iot-hub/iot-hub-devguide-messages-construct.md).

En los mensajes siguientes, el módulo perimetral de Video Analyzer define las propiedades de la aplicación y el contenido del cuerpo.

### <a name="mediasessionestablished-event"></a>Evento MediaSessionEstablished

Cuando se activa una canalización en directo, el nodo de origen de RTSP intenta conectarse al servidor RTSP que se ejecuta en el contenedor rtspsim-live555. Si la conexión se realiza correctamente, se imprime el evento siguiente.

```
[IoTHubMonitor] [10:51:34 AM] Message received from [avasample-iot-edge-device/avaedge]:
{
  "body": {
  {
    "sdp&quot;: &quot;SDP:\nv=0\r\no=- 1620204694595500 1 IN IP4 xxx.xxx.xxx.xxx\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/camera-300s.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.08.19\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/camera-300s.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets=Z00AKeKQCgC3YC3AQEBpB4kRUA==,aO48gA==\r\na=control:track1\r\n"
  },
  "properties": {
    "topic": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/ava-sample-deployment/providers/Microsoft.Media/videoAnalyzers/avasample",
    "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/sources/rtspSource",
    "eventType": "Microsoft.VideoAnalyzer.Diagnostics.MediaSessionEstablished",
    "eventTime": "2021-05-06T00:58:58.602Z",
    "dataVersion": "1.0"
  },
  "systemProperties": {
    "iothub-connection-device-id": "avasample-iot-edge-device",
    "iothub-connection-module-id": "avaedge",
    "iothub-message-source": "Telemetry",
    "messageId": "459c3255-7c86-4ff5-a1e5-7ce3fcb07627",
    "contentType": "application/json",
    "contentEncoding": "utf-8"
  }
}
```

En la salida anterior:

- El mensaje es un evento de diagnóstico, **MediaSessionEstablished**. Indica que el nodo de origen RTSP (el sujeto) ha conectado con el simulador RTSP y ha comenzado a recibir una fuente en directo (simulada).
- La sección sdp contiene datos sobre el evento de diagnóstico. En este caso, los datos incluyen detalles sobre el [protocolo de descripción de sesiones (SDP)](https://en.wikipedia.org/wiki/Session_Description_Protocol).

### <a name="motiondetection-event"></a>Evento MotionDetection

Cuando se detecta movimiento, el módulo de Video Analyzer envía un evento de inferencia. El \* **\*tipo** se establece en **motion** para indicar que se trata de un resultado del procesador de detección del movimiento.

A continuación se muestra un ejemplo de este mensaje:

```json
{
  "body": {
    "timestamp": 145818422564951,
    "inferences": [
      {
        "type": "motion",
        "motion": {
          "box": {
            "l": 0.322176,
            "t": 0.574627,
            "w": 0.525,
            "h": 0.088889
          }
        }
      }
    ]
  },
  "properties": { … },
  "systemProperties": { … }
}
```

En este ejemplo:

- El valor **body** son datos sobre el evento de análisis. En este caso, el evento es un evento de inferencia, por lo tanto, el valor body contiene datos de **timestamp** e **inferences**.
- Los datos de **inferences** indican que el **tipo** es **motion**. Tiene datos adicionales sobre el evento **motion**.
- La sección **box** contiene las coordenadas de un rectángulo delimitador alrededor del objeto en movimiento. Los valores se normalizan según el ancho y el alto del vídeo en píxeles. Por ejemplo, para obtener las coordenadas del píxel original, debe multiplicar las dimensiones horizontales por 1920 y las dimensiones verticales por 1080.

  ```
  l - distance from left of image
  t - distance from top of image
  w - width of bounding box
  h - height of bounding box
  ```
