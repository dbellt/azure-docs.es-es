---
title: 'Grabación y transmisión de metadatos de inferencia con vídeo: Azure Video Analyzer'
description: En este tutorial, aprenderá a usar Azure Video Analyzer para grabar metadatos de inferencia y vídeo en la nube y reproducir la grabación con los metadatos de inferencia visual.
ms.service: azure-video-analyzer
ms.topic: how-to
ms.date: 05/12/2021
ms.openlocfilehash: 7b1122c098fc30150699f6c878058d37f74a007f
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110465823"
---
# <a name="tutorial-record-and-stream-inference-metadata-with-video"></a>Tutorial: Grabación y transmisión de metadatos de inferencia con vídeo
  
En este tutorial, aprenderá a usar Azure Video Analyzer para grabar metadatos de inferencia y vídeo en directo en la nube y reproducir esa grabación con los metadatos de inferencia visual. En este caso de uso, grabará vídeo continuamente, mientras usa un modelo personalizado para detectar objetos **(yoloV3)** y un procesador de Video Analyzer **(seguimiento de objetos)** para realizar un seguimiento de los objetos. Como el vídeo se graba continuamente, también lo harán los metadatos de inferencia de los objetos que se detectan y de los que se realiza un seguimiento. 

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="suggested-pre-reading"></a>Sugerencias antes de la lectura  

Consulte estos artículos antes de empezar:

* [Información general de Azure Video Analyzer en IoT Edge](overview.md)
* [Terminología de Azure Video Analyzer en IoT Edge](terminology.md)
* [Conceptos sobre la canalización de Video Analyzer](pipeline.md) 
* [Grabación de vídeo continua](continuous-video-recording.md)
* [Inicio rápido: Análisis de vídeo en directo con su propio modelo - HTTP](analyze-live-video-use-your-model-http.md)
* [Inicio rápido: Seguimiento de objetos en un vídeo en directo](track-objects-live-video.md)

## <a name="prerequisites"></a>Requisitos previos

Los requisitos previos de este tutorial son los siguientes: [!INCLUDE [prerequisites](./includes/common-includes/csharp-prerequisites.md)]

## <a name="set-up-azure-resources"></a>Configuración de los recursos de Azure

[![Implementación en Azure](https://aka.ms/deploytoazurebutton)](https://aka.ms/ava-click-to-deploy)
[!INCLUDE [resources](./includes/common-includes/azure-resources.md)]

## <a name="overview"></a>Información general
> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/record-stream-inference-data-with-video/overview.svg" alt-text="Canalización":::

El diagrama es una representación gráfica de una [canalización](pipeline.md) y los módulos adicionales que llevan a cabo el escenario deseado. Hay tres módulos IoT Edge implicados:
* Módulo Video Analyzer.
* Un módulo IoT Edge que ejecuta un modelo de inteligencia artificial mediante un punto de conexión HTTP. Este módulo de inteligencia artificial utiliza el modelo [YOLOv3](https://github.com/Azure/video-analyzer/tree/main/edge-modules/extensions/yolo/yolov3), que es capaz de detectar muchos tipos de objetos.
* Un [módulo simulador RTSP](https://github.com/Azure/video-analyzer/tree/main/edge-modules/sources/rtspsim-live555) para simular una cámara RTSP.

Como se muestra en el diagrama, se usará un nodo de [origen RTSP](pipeline.md#rtsp-source) en la canalización para capturar el vídeo en directo simulado de tráfico en una autopista y enviarlo a dos rutas de acceso:

* La primera ruta de acceso es a un nodo de extensión HTTP. El nodo de extensión HTTP desempeña el rol de un proxy. Convierte cada 10 fotogramas de vídeo en el tipo de imagen especificado. Luego, retransmite la imagen mediante HTTP a otro módulo perimetral que ejecuta un modelo de IA detrás de un punto de conexión HTTP. En este ejemplo, el módulo perimetral se crea mediante el modelo YOLOv3, que puede detectar muchos tipos de objetos. El nodo procesador de extensiones HTTP recopila los resultados de la detección y envía estos resultados y todos los fotogramas del vídeo (no solo cada 10 fotogramas) al nodo de seguimiento de objetos. El nodo de seguimiento de objetos usa técnicas de flujo óptico para realizar un seguimiento del objeto en los 9 fotogramas a los que no se les aplicó el modelo de IA. El nodo de seguimiento publica sus resultados en el nodo receptor de vídeo y el nodo receptor de IoT Hub. El nodo [receptor de vídeo](pipeline.md#video-sink) usará los metadatos de inferencia del nodo de seguimiento de objetos que se reproducirán con el vídeo grabado. El nodo [receptor de mensajes de IoT Hub](pipeline.md#iot-hub-message-sink) envía esos eventos al [centro de IoT Edge](../../iot-fundamentals/iot-glossary.md#iot-edge-hub).

* La segunda ruta de acceso abarca directamente desde el origen RTSP al nodo receptor de vídeo para realizar una grabación de vídeo continua. En este tutorial se va a usar un [vídeo de ejemplo de un ](https://lvamedia.blob.core.windows.net/public/camera-300s.mkv)de vídeo de ejemplo de un cruce de autopistas.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4LTY4]

En este tutorial, aprenderá lo siguiente:

1. Configure un entorno de desarrollo.
1. Implementará los módulos perimetrales necesarios.
1. Creará e implementará la canalización en directo.
1. Interpretará los resultados.
1. Limpieza de recursos.

## <a name="set-up-your-development-environment"></a>Configurado su entorno de desarrollo
[!INCLUDE [setup development environment](./includes/set-up-dev-environment/csharp/csharp-set-up-dev-env.md)]

## <a name="examine-the-sample-files"></a>Examen de los archivos de ejemplo

En Visual Studio Code, vaya a src/edge. Verá el archivo .env que ha creado y algunos archivos de plantilla de implementación. Esta plantilla define los módulos IoT Edge que se van a implementar en el dispositivo perimetral (la máquina virtual Linux de Azure). El archivo .env contiene valores para las variables que se usan en estas plantillas, como las credenciales de Video Analyzer.

Abra src/edge/deployment.yolov3.template.json. Hay tres entradas en la sección **modules**, correspondientes a los elementos enumerados anteriormente en la sección "Introducción"

* **avaedge**: se trata del módulo Video Analyzer en IoT Edge.
* **yolov3**: módulo de inteligencia artificial creado con el modelo YOLO v3.
* **rtspsim**: simulador RTSP.


A continuación, vaya a la carpeta src/cloud-to-device-console-app. Aquí podrá ver el archivo appsettings.json que creó junto con algunos otros archivos:

* **c2d-console-app.csproj**: archivo de proyecto de Visual Studio Code.
* **operations.json**: en este archivo se enumeran las distintas operaciones que se ejecutarán.
* **Program.cs**: código del programa de ejemplo, el cual:
    * Carga la configuración de la aplicación.

    * Invoca los métodos directos que expone el módulo Video Analyzer en IoT Edge. Puede usar el módulo para analizar secuencias de vídeo en directo mediante la invocación de sus [métodos directos](direct-methods.md).

    * Se pone en pausa para que pueda examinar la salida del programa en la ventana **TERMINAL** y los eventos generados por el módulo en la ventana **SALIDA**.
    * Invoca los métodos directos para limpiar los recursos.

## <a name="generate-and-deploy-the-iot-edge-deployment-manifest"></a>Generación e implementación del manifiesto de implementación de IoT Edge 

1. Haga clic con el botón derecho en el archivo _src/edge/deployment.yolov3.template.json_ y seleccione **Generate IoT Edge Deployment Manifest** (Generar manifiesto de implementación de IoT Edge).

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/analyze-live-video-use-your-model-http/generate-deployment-manifest.png" alt-text="Captura de pantalla de la opción para generar un manifiesto de implementación de IoT Edge":::

    * El archivo de manifiesto _deployment.yolov3.amd64.json_ se crea en la carpeta _src/edge/config_.
1. Haga clic con el botón derecho en _src/edge/config/deployment.yolov3.amd64.json_ y seleccione **Create Deployment for Single Device** (Crear una implementación para un dispositivo individual).

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/analyze-live-video-use-your-model-http/deployment-single-device.png" alt-text= "Screenshot of Create Deployment for Single Device":::

1. Cuando se le pida que seleccione un dispositivo IoT Hub, elija **ava-sample-iot-edge-device**.
1. Tras aproximadamente 30 segundos, en la esquina inferior izquierda de la ventana, actualice Azure IoT Hub. El dispositivo perimetral ahora muestra los siguientes módulos implementados:

   - El módulo **avaedge**, que es el módulo Video Analyzer.
   - El módulo **rtspsim**, que simula un servidor RTSP y actúa como el origen de una fuente de vídeo en directo.
   - El módulo **yolov3**, que es el modelo de detección de objetos YoloV3 que aplica Computer Vision a las imágenes y devuelve varias clases de tipos de objeto

     > [!div class="mx-imgBorder"]
     > :::image type="content" source="./media/vscode-common-screenshots/avaextension.png" alt-text= "Screenshot of YoloV3 object detection model":::
     
## <a name="create-and-deploy-the-live-pipeline"></a>Creación e implementación de la canalización en directo

### <a name="edit-the-sample-files"></a>Edición de los archivos de ejemplo

1. En Visual Studio Code, abra la pestaña **Extensiones** (o presione Ctrl + Mayús + X) y busque Azure IoT Hub.
1. Haga clic con el botón derecho y seleccione la **Configuración de la extensión**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/extensions-tab.png" alt-text="Configuración de la extensión":::
1. Busque y habilite "Show Verbose Message" (Mostrar mensaje detallado).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/show-verbose-message.png" alt-text="Show Verbose Message"::: (Mostrar mensaje detallado)
1. Vaya a "src/cloud-to-device-console-app/operations.json".
1. En el nodo **pipelineTopologySet**, edite lo siguiente
 
    `"pipelineTopologyUrl" : "https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/cvr-with-httpExtension-and-objectTracking/topology.json" `
    
1. Luego, en los nodos **livePipelineSet** y **pipelineTopologyDelete**, compruebe que el valor de **topologyName** coincide con el valor de la propiedad **name** de la topología de canalización anterior:

    `"pipelineTopologyName" : "CVRHttpExtensionObjectTracking"`
1. Abra la [topología de la canalización](https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/cvr-with-httpExtension-objTracking/topology.json) en un explorador y vea que videoName está codificado de forma rígida como `sample-cvr-with-inference-metadata`. Para los fines de este tutorial, este valor es aceptable. En producción, tendría que asegurarse de que cada cámara RTSP única se graba en un recurso de vídeo con un nombre único.  

1. Examine la configuración del nodo de extensión HTTP.

  ```=
     "samplingOptions":{
         "skipSamplesWithoutAnnotation":"false",
         "maximumSamplesPerSecond":"2"
    }
  ```

En este caso, `skipSamplesWithoutAnnotation` se establece en `false` porque el nodo de extensión debe pasar por todos los fotogramas, tanto si tienen resultados de inferencia como si no, al nodo de seguimiento de objetos del flujo descendente. El seguimiento de objetos tiene capacidad para realizar un seguimiento de objetos de más de 15 fotogramas, aproximadamente. Si el vídeo en directo se ejecuta a una velocidad de 30 fotogramas por segundo, significa que se deben enviar al menos dos fotogramas por segundo al servidor HTTP para la inferencia; por lo tanto, `maximumSamplesPerSecond` se establece en 2.

## <a name="run-the-sample-program"></a>Ejecución del programa de ejemplo

1. Para iniciar una sesión de depuración, seleccione F5. Verá algunos mensajes impresos en la ventana **TERMINAL**.
1. El archivo operations.json comienza con llamadas a pipelineTopologyList y livePipelineList. Si ha limpiado los recursos después de los inicios rápidos o los tutoriales anteriores, esta acción devolverá listas vacías y entrará en pausa para que seleccione **Entrar**, como se muestra a continuación:
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
1. Después de seleccionar **Entrar** en la ventana **TERMINAL**, se realiza el siguiente conjunto de llamadas de método directo:
   * Una llamada a pipelineTopologySet mediante el uso de pipelinetopologyUrl anterior
   * Una llamada a livePipelineSet mediante el uso del siguiente cuerpo
     
        ```
        {
          "@apiVersion": "1.0",
          "name": "Sample-Pipeline-1",
          "properties": {
            "topologyName": "CVRHttpExtensionObjectTracking",
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
   * Una llamada a livePipelineActivate que inicia la canalización en directo y el flujo de vídeo.
   * Una segunda llamada a livePipelineList que muestra que la canalización en directo está en estado de ejecución.
1. La salida de la ventana TERMINAL se pondrá en pausa con el mensaje Press Enter to continue (Presione Entrar para continuar). No seleccione Entrar todavía. Desplácese hacia arriba para ver las cargas de la respuesta JSON para los métodos directos que ha invocado.
1. Cambie a la ventana SALIDA de Visual Studio Code. Verá los mensajes que el módulo Video Analyzer en IoT Edge está enviando al centro de IoT.
1. La canalización continúa ejecutándose y se imprimen los resultados. El simulador RTSP sigue recorriendo el vídeo de origen. Para detener la canalización en directo, vuelva a la ventana **TERMINAL** y seleccione Entrar.
1. La siguiente serie de llamadas limpia los recursos:

    * Una llamada a livePipelineDeactivate desactiva la canalización en directo.
    * Una llamada a livePipelineDelete elimina la canalización en directo.
    * Una llamada a pipelineTopologyDelete elimina la topología de canalización.
    * Una llamada final a pipelineTopologyList muestra que la lista está vacía.       

## <a name="interpret-results"></a>Interpretación de los resultados

Cuando se ejecuta la canalización en directo, los resultados del nodo procesador de extensiones HTTP recorren el nodo de seguimiento de objetos hasta llegar al nodo receptor de IoT Hub y al centro de IoT. Los mensajes que aparecen en la ventana **SALIDA** contienen una sección "body" y una sección "applicationProperties". Para más información, consulte [Creación y lectura de mensajes de IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).

En los mensajes siguientes, el módulo Video Analyzer define las propiedades de la aplicación y el contenido del cuerpo.

## <a name="diagnostics-events"></a>Eventos de diagnóstico
### <a name="mediasessionestablished-event"></a>Evento MediaSessionEstablished

Cuando se activa una canalización en directo, el nodo de origen de RTSP intenta conectarse al servidor RTSP que se ejecuta en el contenedor rtspsim-live555. Si la conexión se realiza correctamente, se imprime el evento siguiente. El tipo de evento es Microsoft.VideoAnalyzer.Diagnostics.MediaSessionEstablished.

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

* El mensaje es un evento de diagnóstico. MediaSessionEstablished indica que el nodo de origen RTSP (valor de "subject") ha conectado con el simulador RTSP y ha comenzado a recibir una fuente en vivo (simulada).
* En applicationProperties, el sujeto indica que el mensaje se generó desde el nodo de origen RTSP de la canalización en directo.
* En applicationProperties, el valor de "eventType" indica que se trata de un evento de diagnóstico.
* El valor de "eventTime" indica la hora a la que se produjo el evento.
* El cuerpo contiene datos sobre el evento de diagnóstico. En este caso, los datos incluyen detalles sobre el [protocolo de descripción de sesiones (SDP)](https://en.wikipedia.org/wiki/Session_Description_Protocol).

## <a name="operational-events"></a>Eventos operativos

### <a name="object-tracking-events"></a>Eventos de seguimiento de objetos

El nodo procesador de extensiones HTTP envía los fotogramas 0, 15, 30, ... etc. al módulo yolov3 y recibe los resultados de la inferencia. Luego, envía estos resultados y todos los fotogramas de vídeo al nodo de seguimiento de objetos. Supongamos que se detectó un objeto en el fotograma 0; el seguimiento de objetos asignará entonces un valor sequenceId único a ese objeto. A continuación, en los fotogramas 1, 2,...,14, si se puede realizar el seguimiento de ese objeto, se mostrará un resultado con el mismo valor sequenceId. En los fragmentos de código siguientes de los resultados, observe cómo se repite `sequenceId`, pero la ubicación del rectángulo de selección ha cambiado, a medida que el objeto se mueve.

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

### <a name="recordingstarted-event"></a>Evento RecordingStarted
Cuando el nodo receptor de vídeo comienza a grabar elementos multimedia, emite este evento de tipo **Microsoft.VideoAnalyzers.Pipeline.Operational.RecordingStarted**:

```
[IoTHubMonitor] [9:42:38 AM] Message received from [avasample-iot-edge-device/avaedge]:
{
  "body": {
    "outputType": "videoName",
    "outputLocation&quot;: &quot;sample-cvr-with-inference-metadata"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{resource-group-name}/providers/microsoft.media/videoAnalyzers/{ava-account-name}",
    "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/sinks/videoSink",
    "eventType": "Microsoft.VideoAnalyzers.Pipeline.Operational.RecordingStarted",
    "eventTime": "2021-04-09T09:42:38.1280000Z",
    "dataVersion&quot;: &quot;1.0"
  }
}
```

La sección subject de applicationProperties hace referencia al nodo receptor de vídeo en la canalización en directo, que generó este mensaje.

La sección body contiene información sobre la ubicación de salida. En este caso, es el nombre del recurso de Video Analyzer en el que se graba el vídeo.

### <a name="recordingavailable-event"></a>Evento RecordingAvailable

Como sugiere su nombre, el evento RecordingStarted se envía cuando se inicia la grabación, pero puede que los datos multimedia aún no se hayan cargado en el recurso de vídeo. Al cargar elementos multimedia el nodo receptor de vídeo, emite un evento de tipo **Microsoft.VideoAnalyzers.Pipeline.Operational.RecordingAvailable**:

```
[IoTHubMonitor] [[9:43:38 AM] Message received from [ava-sample-device/avaedge]:
{
  "body": {
    "outputType": "videoName",
    "outputLocation&quot;: &quot;sample-cvr-with-inference-metadata"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{resource-group-name}/providers/microsoft.media/videoAnalyzers/{ava-account-name}",
    "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/sinks/videoSink",
    "eventType": "Microsoft.VideoAnalyzers.Pipeline.Operational.RecordingAvailable",
    "eventTime": "2021-04-09T09:43:38.1280000Z",
    "dataVersion&quot;: &quot;1.0"
  }
}
```

Este evento indica que se han escrito suficientes datos en el recurso de vídeo para que los reproductores o clientes inicien la reproducción del vídeo.

La sección subject de applicationProperties hace referencia al nodo receptor de vídeo en la canalización en directo, que generó este mensaje.

La sección body contiene información sobre la ubicación de salida. En este caso, es el nombre del recurso de Video Analyzer en el que se graba el vídeo.

### <a name="recordingstopped-event"></a>Evento RecordingStopped

Al desactivar la canalización en directo, el nodo receptor de vídeo deja de grabar los elementos multimedia. y emite este evento de tipo **Microsoft.Media.Graph.Operational.RecordingStopped**:

```
[IoTHubMonitor] [11:33:31 PM] Message received from [ava-sample-device/avaedge]:
{
  "body": {
    "outputType": "videoName",
    "outputLocation&quot;: &quot;sample-cvr-with-inference-metadata"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{resource-group-name}/providers/microsoft.media/videoAnalyzers/{ava-account-name}",
    "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/sinks/videoSink",
    "eventType": "Microsoft.VideoAnalyzers.Pipeline.Operational.RecordingStopped",
    "eventTime": "2021-04-10T11:33:31.051Z",
    "dataVersion&quot;: &quot;1.0"
  }
}
```

Este evento indica que se ha detenido la grabación.

La sección subject de applicationProperties hace referencia al nodo receptor de vídeo en la canalización en directo, que generó este mensaje.

La sección body contiene información sobre la ubicación de salida, que, en este caso, es el nombre del recurso de Video Analyzer en el que se graba el vídeo.

## <a name="streaming-the-recording-with-visual-inference-metadata"></a>Transmisión de la grabación con metadatos de inferencia visual

Para examinar el recurso de vídeo de Video Analyzer que se ha creado con la canalización en directo, inicie sesión en Azure Portal y vea el vídeo.

1. Abra el explorador web y vaya a [Azure Portal](https://portal.azure.com/). Introduzca sus credenciales para iniciar sesión en el portal. La vista predeterminada es el panel del servicio.
1. Busque la cuenta de Video Analyzer entre los recursos que tiene en la suscripción y abra el panel de cuentas.
1. Seleccione **Vídeos** en la lista **Instancias de Video Analyzer**.
1. Encontrará un vídeo con el nombre `sample-cvr-with-inference-metadata`. Este es el nombre elegido en el archivo de topología de la canalización.
1. Seleccione el vídeo.
1. En la página de detalles del vídeo, haga clic en el icono **Reproducir**

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/record-stream-inference-data-with-video/video-playback.png" alt-text="Captura de pantalla de la reproducción de vídeo":::
   
1. Para ver los metadatos de inferencia como rectángulos de selección en el vídeo, haga clic en el icono del **rectángulo de selección**
   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/record-stream-inference-data-with-video/bounding-box.png" alt-text="Icono del rectángulo de selección":::

> [!NOTE]
> Puesto que el origen del vídeo era un contenedor que simula una fuente de cámara, las marcas de tiempo del vídeo son relativas a cuándo activó y desactivó la canalización en directo.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a probar los otros tutoriales, debe mantener los recursos creados. En caso contrario, vaya a Azure Portal, vaya a los grupos de recursos, seleccione el grupo de recursos en el que ejecutó este tutorial y elimine el grupo de recursos.

## <a name="next-steps"></a>Pasos siguientes

* Use una [cámara IP](https://en.wikipedia.org/wiki/IP_camera) compatible con RTSP en lugar de utilizar el simulador RTSP. Puede buscar cámaras IP con compatibilidad con RTSP en la página de productos [compatibles con ONVIF](https://www.onvif.org/conformant-products/), buscando dispositivos que cumplan con los perfiles G, S o T.
* Use un dispositivo Linux AMD64 o x64 (en lugar de usar una máquina virtual Linux de Azure). El dispositivo debe estar en la misma red que la cámara IP. Siga las instrucciones de [Instalación del entorno de ejecución de Azure IoT Edge en Linux](../../iot-edge/how-to-install-iot-edge.md). A continuación, siga las instrucciones de la guía de inicio rápido [Implementación del primer módulo IoT Edge en un dispositivo virtual Linux](../../iot-edge/quickstart-linux.md) para registrar el dispositivo en Azure IoT Hub.
 
