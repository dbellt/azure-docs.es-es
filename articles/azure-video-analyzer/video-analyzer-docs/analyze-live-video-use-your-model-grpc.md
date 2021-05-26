---
title: Análisis de vídeo en directo con su propio modelo gRPC
description: En este inicio rápido se describe cómo analizar vídeo en directo con su propio modelo gRPC con Video Analyzer.
ms.service: azure-video-analyzer
ms.topic: quickstart
ms.date: 04/21/2021
zone_pivot_groups: video-analyzer-programming-languages
ms.openlocfilehash: a9b086f4b2d856e3e534ac2603f61dfbfef31db0
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110371880"
---
# <a name="quickstart-analyze-live-video-with-your-own-model---grpc"></a>Inicio rápido: Análisis de vídeo en directo con su propio modelo gRPC

En este inicio rápido se muestra cómo usar Azure Live Video Analyzer para analizar una fuente de vídeo en directo desde una cámara IP (simulada). Verá cómo aplicar un modelo de Computer Vision para detectar objetos. Un subconjunto de los fotogramas de la fuente de vídeo en directo se envía a un servicio de inferencia. Los resultados se envían al centro de IoT Edge.

Este inicio rápido usa una máquina virtual de Azure como dispositivo IoT Edge y emplea una secuencia de vídeo en directo simulada. Se basa en el código de ejemplo escrito en C# y en el inicio rápido [Detección de movimiento y emisión de eventos](detect-motion-emit-events-quickstart.md).

## <a name="prerequisites"></a>Requisitos previos

::: zone pivot="programming-language-csharp"
[!INCLUDE [prerequisites](includes/analyze-live-video-use-your-model-grpc/csharp/prerequisites.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [prerequisites](includes/analyze-live-video-use-your-model-grpc/python/prerequisites.md)]
::: zone-end

## <a name="review-the-sample-video"></a>Revisión del vídeo de ejemplo

Cuando configura los recursos de Azure, se copia un vídeo corto del tráfico en la autopista en la máquina virtual Linux en Azure que usa como dispositivo IoT Edge. En este inicio rápido se usa el archivo de vídeo para simular una secuencia en directo.

Abra una aplicación como [VLC Media Player](https://www.videolan.org/vlc/). Seleccione Ctrl+N y, a continuación, pegue un vínculo al [vídeo de ejemplo de cruce de autopista](https://lvamedia.blob.core.windows.net/public/camera-300s.mkv) para iniciar la reproducción. Verá las imágenes de muchos vehículos moviéndose en el tráfico de una autopista.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4LTY4]

En este inicio rápido, usará Live Video Analyzer para detectar objetos, como vehículos y personas. Publicará eventos de inferencia asociados en IoT Edge Hub.

## <a name="create-and-deploy-the-pipeline"></a>Creará e implementará la canalización.

### <a name="examine-and-edit-the-sample-files"></a>Examen y edición de los archivos de ejemplo

::: zone pivot="programming-language-csharp"
[!INCLUDE [prerequisites](includes/analyze-live-video-use-your-model-grpc/csharp/sample-files.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [prerequisites](includes/analyze-live-video-use-your-model-grpc/python/sample-files.md)]
::: zone-end

## <a name="generate-and-deploy-the-iot-edge-deployment-manifest"></a>Generación e implementación del manifiesto de implementación de IoT Edge

1. Haga clic con el botón derecho en el archivo _src/edge/_ deployment.grpcyolov3icpu.template.json y seleccione **Generate IoT Edge Deployment Manifest** (Generar manifiesto de implementación de IoT Edge).

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/analyze-live-video-use-your-model-grpc/generate-deployment-manifest.png" alt-text="Generar un manifiesto de implementación de IoT Edge":::

1. El archivo de manifiesto _deployment.grpcyolov3icpu.amd64.json_ se crea en la carpeta src/edge/config.

1. Haga clic con el botón derecho en src/edge/config/ **deployment.grpcyolov3icpu.amd64.json** y seleccione **Create Deployment for Single Device** (Crear una implementación para un dispositivo individual).

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/analyze-live-video-use-your-model-grpc/deployment-single-device.png" alt-text= "Create Deployment for Single Device":::

1. Cuando se le pida que seleccione un dispositivo IoT Hub, elija avasample-iot-edge-device.
1. Tras aproximadamente 30 segundos, en la esquina inferior izquierda de la ventana, actualice Azure IoT Hub. El dispositivo perimetral ahora muestra los siguientes módulos implementados:

   - El módulo de Video Analyzer, llamado **avaedge**.
   - El módulo **rtspsim**, que simula un servidor RTSP y actúa como el origen de una fuente de vídeo en directo.
   - El módulo **avaExtension**, que es el modelo de detección de objetos YOLOv3 que usa gRPC como método de comunicación y aplica Computer Vision a las imágenes y devuelve varias clases de tipos de objeto.

     > [!div class="mx-imgBorder"]
     > :::image type="content" source="./media/vscode-common-screenshots/avaextension.png" alt-text= "YoloV3 object detection model":::

## <a name="run-the-sample-program"></a>Ejecución del programa de ejemplo

1. Para iniciar una sesión de depuración, seleccione la tecla F5. Verá mensajes impresos en la ventana TERMINAL.
1. El código **operations.json** comienza con llamadas a los métodos directos pipelineTopologyList y livePipelineList. Si ha limpiado los recursos tras haber completado los inicios rápidos anteriores, este proceso devolverá listas vacías y, después, se pausará. Para continuar, seleccione la tecla Entrar.

   ```
   --------------------------------------------------------------------------
   Executing operation pipelineTopologyList
   -----------------------  Request: pipelineTopologyList  --------------------------------------------------
   {
   "@apiVersion": "1.0"
   }
   ---------------  Response: pipelineTopologyList - Status: 200  ---------------
   {
   "value": []
   }
   --------------------------------------------------------------------------
   Executing operation WaitForInput

   Press Enter to continue
   ```

1. La ventana TERMINAL muestra el siguiente conjunto de llamadas al método directo:

   - Una llamada a pipelineTopologySet que usa el valor de pipelineTopologyUrl anterior.
   - Una llamada a livePipelineSet que usa el siguiente cuerpo:

   ```
   {
     "@apiVersion": "1.0",
     "name": "Sample-Pipeline-1",
     "properties": {
       "topologyName": "InferencingWithGrpcExtension",
       "description": "Sample pipeline description",
       "parameters": [
         {
           "name": "rtspUrl",
           "value": "rtsp://rtspsim:554/media/camera-300s.mkv"
         },
         {
           "name": "rtspUserName",
           "value": "testuser"
         },
         {
           "name": "rtspPassword",
           "value": "testpassword"
         },
         {
           "name": "grpcExtensionAddress",
           "value": "tcp://avaextension:44000"
         }
       ]
     }
   }
   ```

   - Una llamada a livePipelineActivate que inicia la canalización en directo y el flujo de vídeo.
   - Una segunda llamada a livePipelineList que muestra que la canalización en directo está en estado de ejecución.

1. La salida de la ventana TERMINAL se pondrá en pausa con el mensaje Press Enter to continue (Presione Entrar para continuar). No seleccione Entrar todavía. Desplácese hacia arriba para ver las cargas de la respuesta JSON para los métodos directos que ha invocado.
1. Cambie a la ventana SALIDA de Visual Studio Code. Verá los mensajes que el módulo de Video Analyzer está enviando al centro de IoT. En la siguiente sección de este inicio rápido se analizan estos mensajes.
1. La canalización continúa ejecutándose y se imprimen los resultados. El simulador RTSP sigue recorriendo el vídeo de origen. Para detener la canalización, vuelva a la ventana TERMINAL y seleccione Entrar.

   La siguiente serie de llamadas limpia los recursos:

   - Una llamada a `livePipelineDeactivate` desactiva la canalización en directo. 
   - Una llamada a `livePipelineDelete` elimina la canalización en directo.
   - Una llamada a `pipelineTopologyDelete` elimina la topología.
   - Una llamada final a `pipelineTopologyList` muestra que la lista está vacía.

## <a name="interpret-results"></a>Interpretación de los resultados

Cuando se ejecuta la topología de la canalización, los resultados del nodo procesador de extensiones HTTP pasan por el nodo receptor de mensajes de IoT Hub hasta llegar al centro de IoT. Los mensajes que aparecen en la ventana SALIDA contienen una sección "body" y una sección "applicationProperties". Para más información, consulte [Creación y lectura de mensajes de IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).

En los mensajes siguientes, el módulo de Video Analyzer define las propiedades de la aplicación y el contenido del cuerpo.

### <a name="mediasessionestablished-event"></a>Evento MediaSessionEstablished

Cuando se crea una instancia de la canalización, el nodo de origen RTSP intenta conectarse al servidor RTSP que se ejecuta en el contenedor rtspsim-live555. Si la conexión se realiza correctamente, se imprime el evento siguiente. El tipo de evento es Microsoft.VideoAnalyzer..Diagnostics.MediaSessionEstablished.

```
[IoTHubMonitor] [9:42:18 AM] Message received from [avasample-iot-edge-device/avaedge]:
{
  "body": {
    "sdp&quot;: &quot;SDP:\nv=0\r\no=- 1586450538111534 1 IN IP4 XXX.XX.XX.XX\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/camera-300s.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.03.06\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/camera-300s.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets=XXXXXXXXXXXXXXXXXXXXXX\r\na=control:track1\r\n"
  },
  "applicationProperties": {
    "dataVersion": "1.0",
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/videoanalyzers/{ava-account-name}",
    "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/sources/rtspSource",
    "eventType": "Microsoft.VideoAnalyzers.Diagnostics.MediaSessionEstablished",
    "eventTime": "2021-04-09T09:42:18.1280000Z"
  }
}
```

En este mensaje, tenga en cuenta estos detalles:

- El mensaje es un evento de diagnóstico. MediaSessionEstablished indica que el nodo de origen RTSP (valor de "subject") ha conectado con el simulador RTSP y ha comenzado a recibir una fuente en vivo (simulada).
- En applicationProperties, el sujeto indica que el mensaje se generó desde el nodo de origen RTSP de la canalización.
- En applicationProperties, el valor de "eventType" indica que se trata de un evento de diagnóstico.
- El valor de "eventTime" indica la hora a la que se produjo el evento.
- El cuerpo contiene datos sobre el evento de diagnóstico. En este caso, los datos incluyen detalles sobre el protocolo de descripción de sesiones (SDP).

### <a name="inference-event"></a>Evento de inferencia

El nodo procesador de extensiones gRPC recibe los resultados de inferencia del módulo avaextension. Luego, emite los resultados a través del nodo receptor de mensajes de IoT Hub como eventos de inferencia. En estos eventos, el tipo se establece en entity para indicar que es una entidad, como un coche o un camión. El valor eventTime es la hora UTC a la que se detectó el objeto. En el siguiente ejemplo, se han detectado tres automóviles en el mismo fotograma de vídeo, con distintos niveles de confianza.

```json
[IoTHubMonitor] [1:48:04 PM] Message received from [avasample-iot-edge-device/avaedge]:
{
  "timestamp": 145589011404622,
  "inferences": [
    {
      "type": "entity",
      "entity": {
        "tag": {
          "value": "car",
          "confidence": 0.97052866
        },
        "box": {
          "l": 0.40896654,
          "t": 0.60390747,
          "w": 0.045092657,
          "h": 0.029998193
        }
      }
    },
    {
      "type": "entity",
      "entity": {
        "tag": {
          "value": "car",
          "confidence": 0.9547283
        },
        "box": {
          "l": 0.20050547,
          "t": 0.6094412,
          "w": 0.043425046,
          "h": 0.037724357
        }
      }
    },
    {
      "type": "entity",
      "entity": {
        "tag": {
          "value": "car",
          "confidence": 0.94567955
        },
        "box": {
          "l": 0.55363107,
          "t": 0.5320657,
          "w": 0.037418623,
          "h": 0.027014252
        }
      }
    },
    {
      "type": "entity",
      "entity": {
        "tag": {
          "value": "car",
          "confidence": 0.8916893
        },
        "box": {
          "l": 0.6642384,
          "t": 0.581689,
          "w": 0.034349587,
          "h": 0.027812533
        }
      }
    },
    {
      "type": "entity",
      "entity": {
        "tag": {
          "value": "car",
          "confidence": 0.8547814
        },
        "box": {
          "l": 0.584758,
          "t": 0.60079926,
          "w": 0.07082855,
          "h": 0.034121
        }
      }
    }
  ]
}
```

En los mensajes, tenga en cuenta los siguientes detalles:

- La sección body contiene datos sobre el evento de análisis. En este caso, el evento es un evento de inferencia y, por lo tanto, el cuerpo contiene datos de inferencias.
- La sección inferences indica que el tipo es una entidad. En esta sección se incluyen datos adicionales sobre la entidad.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cleanup](./includes/common-includes/clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

- Pruebe a ejecutar diferentes topologías de canalización con el protocolo gRPC.

- Revise los desafíos adicionales para los usuarios avanzados:

  - Use una [cámara IP](https://en.wikipedia.org/wiki/IP_camera) que sea compatible con RTSP, en lugar de utilizar el simulador RTSP. Puede buscar cámaras IP compatibles con RTSP en la página de [productos compatibles con ONVIF](https://www.onvif.org/conformant-products/). Busque dispositivos que se ajusten a los perfiles G, S o T.
  - Use un dispositivo Linux AMD64 o x64, en lugar de una máquina virtual Linux de Azure. El dispositivo debe estar en la misma red que la cámara IP. Puede seguir las instrucciones que aparecen en [Instalación del entorno de ejecución de Azure IoT Edge en Linux](../../iot-edge/how-to-install-iot-edge.md?view=iotedge-2018-06&preserve-view=true). Luego, registre el dispositivo en Azure IoT Hub, para lo que debe seguir las instrucciones que se encuentran en [Implementación del primer módulo IoT Edge en un dispositivo virtual Linux](../../iot-edge/quickstart-linux.md?view=iotedge-2018-06&preserve-view=true).
