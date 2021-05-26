---
title: Seguimiento de objetos en un vídeo en directo con Azure Video Analyzer
description: En este inicio rápido se muestra cómo usar el módulo perimetral de Azure Video Analyzer para realizar un seguimiento de los objetos de una fuente de vídeo en directo desde una cámara IP (simulada). Verá cómo aplicar un modelo de Computer Vision para detectar objetos en un subconjunto de los fotogramas de la fuente de vídeo en directo. Luego, puede usar un nodo de seguimiento de objetos para realizar el seguimiento de esos objetos en los demás fotogramas.
ms.topic: quickstart
ms.date: 05/01/2021
ms.openlocfilehash: 76723d1fb5dfff47e4ad436e1e4bd18fafa7c102
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388627"
---
# <a name="quickstart-track-objects-in-a-live-video"></a>Inicio rápido: Seguimiento de objetos en un vídeo en directo

En este inicio rápido se muestra cómo usar el módulo perimetral de Azure Video Analyzer para realizar un seguimiento de los objetos de una fuente de vídeo en directo desde una cámara IP (simulada). Verá cómo aplicar un modelo de Computer Vision para detectar objetos en un subconjunto de los fotogramas de la fuente de vídeo en directo. Luego, puede usar un nodo de seguimiento de objetos para realizar el seguimiento de esos objetos en los demás fotogramas.

El seguimiento de objetos resulta útil cuando necesita detectar objetos en cada fotograma, pero el dispositivo perimetral no tiene la capacidad de proceso necesaria para poder aplicar el modelo de visión a todos los fotogramas. Si la fuente de vídeo en directo se ejecuta a, pongamos, 30 fotogramas por segundo y solo se puede ejecutar el modelo de Computer Vision cada 15 fotogramas, el seguimiento de objetos toma los resultados de uno de estos fotogramas y, luego, usa técnicas de [flujo óptico](https://en.wikipedia.org/wiki/Optical_flow) para generar resultados para el segundo, tercer y hasta el catorceavo fotograma, hasta que el modelo se aplica de nuevo al siguiente fotograma.

Este inicio rápido usa una máquina virtual de Azure como dispositivo IoT Edge y emplea una secuencia de vídeo en directo simulada.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [prerequisites](./includes/common-includes/csharp-prerequisites.md)]

## <a name="set-up-azure-resources"></a>Configuración de los recursos de Azure

[![Implementación en Azure](https://aka.ms/deploytoazurebutton)](https://aka.ms/ava-click-to-deploy)

[!INCLUDE [resources](./includes/common-includes/azure-resources.md)]

## <a name="overview"></a>Información general

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/track-objects-live-video/object-tracker-topology.svg" alt-text="Seguimiento de objetos en vídeo en directo":::

En este diagrama se muestra cómo fluyen las señales en este inicio rápido. Un [módulo perimetral](https://github.com/Azure/video-analyzer/tree/main/edge-modules/sources/rtspsim-live555) simula una cámara IP que hospeda un servidor de protocolo RTSP. Un nodo de [origen RTSP](pipeline.md#rtsp-source) extrae la fuente de vídeo de este servidor y envía fotogramas de vídeo al nodo del [procesador de extensiones HTTP](pipeline.md#http-extension-processor).

El nodo de extensión HTTP desempeña el rol de un proxy. Convierte cada 15 fotogramas de vídeo en el tipo de imagen especificado. Luego, retransmite la imagen a través de HTTP a otro módulo perimetral que ejecuta un modelo de IA detrás de un punto de conexión HTTP. En este ejemplo, el módulo perimetral usa [YOLOv3](https://github.com/Azure/video-analyzer/tree/main/edge-modules/extensions/yolo/yolov3), que puede detectar muchos tipos de objetos. El nodo procesador de extensiones HTTP recibe los resultados de la detección y envía estos resultados y todos los fotogramas de vídeo (no solo el fotograma 15) al nodo de [seguimiento de objetos](pipeline.md#object-tracker-processor). El nodo de seguimiento de objetos usa técnicas de flujo óptico para realizar un seguimiento del objeto en los 14 fotogramas a los que no se les aplicó el modelo de IA. El nodo de seguimiento publica sus resultados en el nodo receptor de mensajes de IoT Hub. Este [receptor de mensajes de IoT Hub](pipeline.md#iot-hub-message-sink) envía esos eventos al [centro de IoT Edge](../../iot-fundamentals/iot-glossary.md?view=iotedge-2020-11&preserve-view=true#iot-edge-hub).

> [!NOTE]
> Revise la explicación sobre el equilibrio entre la precisión y la eficacia de procesamiento con el nodo de [seguimiento de objetos](pipeline.md#object-tracker-processor).

En este inicio rápido realizará lo siguiente:

1. Configure un entorno de desarrollo.
1. Implementará los módulos perimetrales necesarios.
1. Creará e implementará la canalización en directo.
1. Interpretará los resultados.
1. Limpieza de recursos.

## <a name="set-up-your-development-environment"></a>Configurado su entorno de desarrollo
[!INCLUDE [setup development environment](./includes/set-up-dev-environment/csharp/csharp-set-up-dev-env.md)]

## <a name="deploy-the-required-modules"></a>Implementación de los módulos necesarios

1. En Visual Studio Code, haga clic con el botón derecho en el archivo *src/edge/deployment.yolov3.template.json* y seleccione **Generate IoT Edge Deployment Manifest** (Generar manifiesto de implementación de IoT Edge).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/analyze-live-video-use-your-model-http/generate-deployment-manifest.png" alt-text="Generar un manifiesto de implementación de IoT Edge":::
1. El archivo de manifiesto *deployment.yolov3.amd64.json* se crea en la carpeta *src/edge/config*.
1. Haga clic con el botón derecho en *src/edge/config/deployment.yolov3.amd64.json* y seleccione **Create Deployment for Single Device** (Crear una implementación para un dispositivo individual).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/analyze-live-video-use-your-model-http/deployment-single-device.png" alt-text= "Create Deployment for Single Device":::
1. Cuando se le pida que seleccione un dispositivo IoT Hub, elija **avasample-iot-edge-device**.
1. Tras aproximadamente 30 segundos, en la esquina inferior izquierda de la ventana, actualice Azure IoT Hub. El dispositivo perimetral ahora muestra los siguientes módulos implementados:

    * El módulo perimetral de Video Analyzer, llamado **avaedge**.
    * El módulo **rtspsim**, que simula un servidor RTSP y actúa como el origen de una fuente de vídeo en directo. 
    * El módulo **yolov3**, que usa el modelo YOLOV3 para detectar diversos objetos.

        > [!div class="mx-imgBorder"]
        > :::image type="content" source="./media/track-objects-live-video/object-tracker-modules.png" alt-text= "List of deployed IoT Edge modules":::


## <a name="create-and-deploy-the-live-pipeline"></a>Creación e implementación de la canalización en directo

### <a name="examine-and-edit-the-sample-files"></a>Examen y edición de los archivos de ejemplo

En Visual Studio Code, vaya a la carpeta src/cloud-to-device-console-app. Aquí podrá ver el archivo appsettings.json que creó junto con algunos otros archivos:

* **c2d-console-app.csproj**: archivo de proyecto de Visual Studio Code.
* **operations.json**: en este archivo se enumeran las distintas operaciones que se ejecutarán.
* **Program.cs**: código del programa de ejemplo, el cual:
    * Carga la configuración de la aplicación.
    * Invoca métodos directos expuestos por el módulo de Video Analyzer. Puede usar el módulo para analizar secuencias de vídeo en directo mediante la invocación de sus [métodos directos](direct-methods.md).
    * Se pone en pausa para que pueda examinar la salida del programa en la ventana **TERMINAL** y los eventos generados por el módulo en la ventana **SALIDA**.
    * Invoca los métodos directos para limpiar los recursos.

1. Edite el archivo operations.json:
    
    * Cambie el vínculo a la topología de la canalización:
    * "pipelineTopologyUrl" : "https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/object-tracking/topology.json"
    * En livePipelineSet, modifique el nombre de la topología de la canalización para que coincida con el valor del vínculo anterior:
    * "topologyName" : "ObjectTrackingWithHttpExtension"
    * En pipelineTopologyDelete, modifique el nombre:
    * "name" : "ObjectTrackingWithHttpExtension"
    
Abra la dirección URL de la topología de la canalización en un explorador y examine la configuración del nodo de extensiones HTTP.

```
   "samplingOptions":{
       "skipSamplesWithoutAnnotation":"false",
       "maximumSamplesPerSecond":"2"
   }
```

En este caso, `skipSamplesWithoutAnnotation` se establece en `false` porque el nodo de extensiones debe pasar por todos los fotogramas, tanto si tienen resultados de inferencia como si no, hasta el nodo de seguimiento de objetos de nivel inferior. El seguimiento de objetos tiene capacidad para realizar un seguimiento de objetos de más de 15 fotogramas, aproximadamente. Si el vídeo en directo se ejecuta a una velocidad de 30 fotogramas por segundo, significa que se deben enviar al menos dos fotogramas por segundo al servidor HTTP para la inferencia; por lo tanto, `maximumSamplesPerSecond` se establece en 2.
    
## <a name="run-the-sample-program"></a>Ejecución del programa de ejemplo

1. Para iniciar una sesión de depuración, seleccione la tecla F5. Verá mensajes impresos en la ventana **TERMINAL**.
1. El código de operations.json comienza con llamadas a los métodos directos `pipelineTopologyList` y `livePipelineList`. Si ha limpiado los recursos tras haber completado los inicios rápidos anteriores, este proceso devolverá listas vacías y, después, se pausará. Para continuar, seleccione la tecla Entrar.
    
    ```
    -------------------------------Executing operation pipelineTopologyList-----------------------  
    Request: pipelineTopologyList  --------------------------------------------------
    {
    "@apiVersion": "1.0"
    }
    ---------------  
    Response: pipelineTopologyList - Status: 200  ---------------
    {
    "value": []
    }
    --------------------------------------------------------------------------
    Executing operation WaitForInput
    
    Press Enter to continue
    ```
    
    La ventana **TERMINAL** muestra el siguiente conjunto de llamadas al método directo:
    
    * Una llamada a `pipelineTopologySet` que usa el contenido de `pipelineTopologyUrl`.
    * Una llamada a `livePipelineSet` que usa el cuerpo siguiente:
        
    ```json
    {
      "@apiVersion": "1.0",
      "name": "Sample-Pipeline-1",
      "properties": {
        "topologyName": "ObjectTrackingWithHttpExtension",
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
          }
        ]
      }
    }
    ```
    * Una llamada a `livePipelineActivate` que activa la canalización en directo y el flujo de vídeo.
    * Una segunda llamada a `livePipelineList` que muestra que la canalización en directo está en estado de ejecución.
1. La salida de la ventana TERMINAL se pondrá en pausa con el mensaje Press Enter to continue (Presione Entrar para continuar). No presione Entrar todavía. Desplácese hacia arriba para ver las cargas de la respuesta JSON para los métodos directos que ha invocado.
1. Cambie a la ventana SALIDA de Visual Studio Code. Verá los mensajes que el módulo perimetral de Video Analyzer está enviando al centro de IoT. En la siguiente sección de este inicio rápido se analizan estos mensajes.
1. La canalización en directo continúa ejecutándose y se imprimen los resultados. El simulador RTSP sigue recorriendo el vídeo de origen. Para detener la canalización en directo, vuelva a la ventana **TERMINAL** y seleccione Entrar.
1. La siguiente serie de llamadas limpia los recursos:

    * Una llamada a `livePipelineDeactivate` desactiva la canalización en directo.
    * Una llamada a `livePipelineDelete` elimina la canalización en directo.
    * Una llamada a `pipelineTopologyDelete` elimina la topología de la canalización.
    * Una llamada final a `pipelineTopologyList` muestra que la lista está vacía.
    
## <a name="interpret-results"></a>Interpretación de los resultados

Cuando se ejecuta la canalización en directo, los resultados del nodo procesador de extensiones HTTP pasan por el nodo receptor de mensajes de IoT Hub hasta llegar al centro de IoT. Los mensajes que aparecen en la ventana **SALIDA** contienen una sección `body` y una sección `applicationProperties`. Para más información, consulte [Creación y lectura de mensajes de IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).

En los mensajes siguientes, el módulo de Video Analyzer define las propiedades de la aplicación y el contenido del cuerpo.

### <a name="mediasessionestablished-event"></a>Evento MediaSessionEstablished

Cuando se activa una canalización en directo, el nodo de origen RTSP intenta conectarse al servidor RTSP que se ejecuta en el contenedor rtspsim-live555. Si la conexión se realiza correctamente, se imprime el evento siguiente. El tipo de evento es **MediaSessionEstablished**.

```
[IoTHubMonitor] [9:42:18 AM] Message received from [avasample-iot-edge-device/avaedge]:
{  "body": {
    "sdp&quot;: &quot;SDP:\nv=0\r\no=- 1586450538111534 1 IN IP4 nnn.nn.0.6\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/camera-300s.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.03.06\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/camera-300s.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets=Z00AKeKQCgC3YC3AQEBpB4kRUA==,aO48gA==\r\na=control:track1\r\n"
  },
  "applicationProperties": {
    "dataVersion": "1.0",
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/videoAnalyzers/{ava-account-name}",
    "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/sources/rtspSource",
    "eventType": "Microsoft.VideoAnalyzer.Diagnostics.MediaSessionEstablished",
    "eventTime": "2020-04-09T16:42:18.1280000Z"
  }
}
```

En este mensaje, tenga en cuenta estos detalles:

* El mensaje es un evento de diagnóstico. **MediaSessionEstablished** indica que el nodo de origen RTSP (el sujeto) ha conectado con el simulador RTSP y ha comenzado a recibir una fuente en directo (simulada).
* En applicationProperties, el sujeto indica que el mensaje se generó desde el nodo de origen RTSP de la canalización en directo.
* En applicationProperties, el valor de "eventType" indica que se trata de un evento de diagnóstico.
* El valor de "eventTime" indica la hora a la que se produjo el evento.
* El cuerpo contiene datos sobre el evento de diagnóstico. En este caso, los datos incluyen detalles sobre el [protocolo de descripción de sesiones (SDP)](https://en.wikipedia.org/wiki/Session_Description_Protocol).

## <a name="object-tracking-events"></a>Eventos de seguimiento de objetos

El nodo procesador de extensiones HTTP envía los fotogramas 0, 15, 30, ... etc. al módulo yolov3 y recibe los resultados de la inferencia. Luego, envía estos resultados y todos los fotogramas de vídeo al nodo de seguimiento de objetos. Supongamos que se detectó un objeto en el fotograma 0; el seguimiento de objetos asignará entonces un valor de `sequenceId` único a ese objeto. A continuación, en los fotogramas 1, 2,...,14, si se puede realizar el seguimiento de ese objeto, se mostrará un resultado con el mismo valor de `sequenceId`. En los fragmentos de código siguientes de los resultados, observe cómo se repite `sequenceId`, pero la ubicación del rectángulo de selección ha cambiado, a medida que el objeto se mueve.

Desde el fotograma M:

```json
  {
    "type": "entity",
    "subtype": "objectDetection",
    "inferenceId": "4d325fc4dc7a43b2a781bf7d6bdb3ff0",
    "sequenceId": "0999a1dde5b241c3a0b2db025f87ab32",
    "entity": {
      "tag": {
        "value": "car",
        "confidence": 0.95237225
      },
      "box": {
        "l": 0.0025893003,
        "t": 0.550063,
        "w": 0.1086607,
        "h": 0.12116724
      }
    }
  },
```

Desde el fotograma N:

```json
{
  "type": "entity",
  "subtype": "objectDetection",
  "inferenceId": "317aafdab7e940388be1e4c4cc58c366",
  "sequenceId": "0999a1dde5b241c3a0b2db025f87ab32",
  "entity": {
    "tag": {
      "value": "car",
      "confidence": 0.95237225
    },
    "box": {
      "l": 0.0027777778,
      "t": 0.54901963,
      "w": 0.108333334,
      "h": 0.12009804
    }
  }
},
```

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [prerequisites](./includes/common-includes/clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

* Intente [detectar cuándo los objetos cruzan una línea virtual en un vídeo en directo](use-line-crossing.md).



