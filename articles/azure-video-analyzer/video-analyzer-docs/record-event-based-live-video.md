---
title: 'Tutorial de grabación de vídeo basada en eventos en la nube y reproducción desde la nube: Azure'
description: En este tutorial, obtendrá información sobre cómo usar Azure Video Analyzer para realizar una grabación de vídeo basada en eventos en la nube y reproducirla desde la nube.
ms.topic: tutorial
ms.date: 04/13/2021
ms.openlocfilehash: 05c28fbc3b410f792d10adf7e59e43f070d7d57a
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110384269"
---
# <a name="tutorial-event-based-video-recording-and-playback"></a>Tutorial: Grabación de vídeo basada en eventos y reproducción

En este tutorial, aprenderá a usar Azure Video Analyzer para grabar de forma selectiva partes de un origen de vídeo en directo en Video Analyzer en la nube. En este tutorial, este caso de uso se conoce como [grabación de vídeo basada en eventos](event-based-video-recording-concept.md) (EVR). Para grabar partes de un vídeo en directo, se usará un modelo de inteligencia artificial de detección de objetos para buscar objetos en el vídeo y grabar clips de vídeo solo cuando se detecte un tipo determinado de objeto. También aprenderá cómo reproducir los clips de vídeo grabados mediante Video Analyzer. Esta funcionalidad resulta útil en diversos escenarios en los que es necesario mantener un archivo de clips de vídeo de interés. 



[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="suggested-pre-reading"></a>Sugerencias antes de la lectura  

Consulte estos artículos antes de empezar:

* [Introducción a Azure Video Analyzer](overview.md)
* [Terminología de Azure Video Analyzer](terminology.md)
* [Conceptos sobre la canalización de Video Analyzer](pipeline.md) 
* [Grabación de vídeo basada en eventos](event-based-video-recording-concept.md)
* [Tutorial: Desarrollo de un módulo IoT Edge](../../iot-edge/tutorial-develop-for-linux.md)
* [Cómo editar deployment.*.template.json](https://github.com/microsoft/vscode-azure-iot-edge/wiki/How-to-edit-deployment.*.template.json)
* Sección [Declaración de rutas del manifiesto de implementación de IoT Edge](../../iot-edge/module-composition.md#declare-routes)

## <a name="prerequisites"></a>Requisitos previos

Los requisitos previos de este tutorial son los siguientes:
* Una cuenta de Azure que incluya una suscripción activa. [Cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) gratuita, en caso de que aún no lo haya hecho.

    > [!NOTE]    
    > Necesitará una suscripción a Azure en la que tenga acceso a los roles [Colaborador](../../role-based-access-control/built-in-roles.md#contributor) y [Administrador de acceso de usuario](../../role-based-access-control/built-in-roles.md#user-access-administrator). Si no tiene los permisos adecuados, póngase en contacto con el administrador de la cuenta para que se los conceda.
* [Instalar Docker](https://docs.docker.com/desktop/#download-and-install) en la máquina.
* [Visual Studio Code](https://code.visualstudio.com/) con las siguientes extensiones:
    * [Herramientas de Azure IoT](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
    * [C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* [SDK de .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1).

## <a name="set-up-azure-resources"></a>Configuración de los recursos de Azure

[![Implementación en Azure](https://aka.ms/deploytoazurebutton)](https://aka.ms/ava-click-to-deploy)
[!INCLUDE [resources](./includes/common-includes/azure-resources.md)]

## <a name="overview"></a>Información general

La grabación de vídeo basada en eventos hace referencia al proceso de grabación de vídeo desencadenado por un evento. Ese evento podría generarse a partir de:
- El procesamiento de la propia señal de vídeo; por ejemplo, al detectar un objeto en movimiento en el vídeo.
- Un origen independiente; por ejemplo, la apertura de una puerta. 

Como alternativa, puede desencadenar la grabación solo cuando un servicio de inferencia detecta que se ha producido un evento específico. En este tutorial se usará un vídeo de vehículos en movimiento en una autovía y se grabarán clips de vídeo cada vez que se detecta un camión.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording/overview.svg" alt-text="Canalización":::

El diagrama es una representación gráfica de una [canalización](pipeline.md) y los módulos adicionales que llevan a cabo el escenario deseado. Hay cuatro módulos IoT Edge implicados:

* Video Analyzer en un módulo IoT Edge.
* Un módulo IoT Edge que ejecuta un modelo de inteligencia artificial mediante un punto de conexión HTTP. Este módulo de inteligencia artificial utiliza el modelo [YOLOv3](https://github.com/Azure/live-video-analytics/tree/master/utilities/video-analysis/yolov3-onnx), que es capaz de detectar muchos tipos de objetos.
* Un módulo personalizado para contar y filtrar objetos, al que se hace referencia en el diagrama como un contador de objetos. En este tutorial, creará un contador de objetos y lo implementará.
* Un [módulo simulador RTSP](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555) para simular una cámara RTSP.
    
Como se muestra en el diagrama, se usará un nodo de [origen RTSP](pipeline.md#rtsp-source) en la canalización para capturar el vídeo en directo simulado de tráfico en una autopista y enviarlo a dos rutas de acceso:

* La primera ruta de acceso es a un nodo de extensión HTTP. El nodo muestrea los fotogramas de vídeo en un valor que se ha establecido mediante el campo `samplingOptions` y, después, retransmite los fotogramas, en forma de imágenes, al módulo de inteligencia artificial YOLOv3, que es un detector de objetos. El nodo recibe los resultados, que son los objetos (vehículos en tráfico) detectados por el modelo. A continuación, el nodo de extensión HTTP publica los resultados mediante el nodo receptor de mensajes de IoT Hub en el centro de IoT Edge.

* El módulo objectCounter está configurado para recibir mensajes desde el centro de IoT Edge, que incluye los resultados de la detección de objetos (vehículos en tráfico). El módulo comprueba estos mensajes en busca de objetos de un tipo determinado, que se configuraron mediante un valor. Cuando se encuentra un objeto de este tipo, este módulo envía un mensaje al centro de IoT Edge. A continuación, los mensajes de "objeto encontrado" se enrutan al nodo de origen de IoT Hub de la canalización. Tras recibir este tipo de mensaje, el nodo de origen de IoT Hub de la canalización desencadena el nodo del [procesador de la puerta de señales](pipeline.md#signal-gate-processor). A continuación, el nodo del procesador de la puerta de señales se abre durante un período de tiempo configurado. El vídeo fluye a través de la puerta al nodo receptor del vídeo durante esa duración. Después, esa parte del streaming en vivo se graba mediante el nodo [receptor del vídeo](pipeline.md#video-sink) en un [vídeo](terminology.md#video) de la cuenta de Video Analyzer. En este tutorial se va a usar un [vídeo de ejemplo de un ](https://lvamedia.blob.core.windows.net/public/camera-300s.mkv)de vídeo de ejemplo de un cruce de autopistas.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4LTY4]

En este tutorial, aprenderá lo siguiente:

1. Configure un entorno de desarrollo.
1. Implementar los módulos perimetrales necesarios.
1. Crear e implementar la canalización en directo.
1. Interpretará los resultados.
1. Limpieza de recursos.

## <a name="set-up-your-development-environment"></a>Configurado su entorno de desarrollo
[!INCLUDE [setup development environment](./includes/set-up-dev-environment/csharp/csharp-set-up-dev-env.md)]


## <a name="examine-the-sample-files"></a>Examen de los archivos de ejemplo

En Visual Studio Code, vaya a src/edge. Verá el archivo .env que ha creado y algunos archivos de plantilla de implementación. Esta plantilla define los módulos IoT Edge que se van a implementar en el dispositivo perimetral (la máquina virtual Linux de Azure). El archivo. env contiene valores para las variables que se usan en estas plantillas, como las credenciales de Video Analyzer.

Abra src/edge/deployment.objectCounter.template.json. Hay cuatro entradas en la sección **modules**, correspondientes a los elementos enumerados anteriormente en la sección "Introducción"

* **avaedge**: módulo de Video Analyzer.
* **yolov3**: módulo de inteligencia artificial creado con el modelo YOLO v3.
* **rtspsim**: simulador RTSP.
* **objectCounter**: módulo que busca objetos específicos en los resultados de yolov3.

Para el módulo objectCounter, observe la cadena (${MODULES.objectCounter}) utilizada para el valor "image". Esto se basa en el [tutorial](../../iot-edge/tutorial-develop-for-linux.md) sobre el desarrollo de un módulo IoT Edge. Visual Studio Code reconoce automáticamente que el código del módulo objectCounter está en src/edge/modules/objectCounter. 

Consulte [esta sección](../../iot-edge/module-composition.md#declare-routes) sobre cómo declarar rutas en el manifiesto de implementación de IoT Edge. A continuación, examine las rutas del archivo JSON de la plantilla. Observe que:

> [!NOTE]
> Compruebe las propiedades deseadas para el módulo objectCounter, que están configuradas para buscar objetos etiquetados como "camión", con un nivel de confianza de al menos el 50 %.

A continuación, vaya a la carpeta src/cloud-to-device-console-app. Aquí podrá ver el archivo appsettings.json que creó junto con algunos otros archivos:

* **c2d-console-app.csproj**: archivo de proyecto de Visual Studio Code.
* **operations.json**: en este archivo se enumeran las distintas operaciones que se ejecutarán.
* **Program.cs**: código del programa de ejemplo, el cual:
    * Carga la configuración de la aplicación.

    * Invoca métodos directos expuestos por el módulo de Video Analyzer. Puede usar el módulo para analizar secuencias de vídeo en directo mediante la invocación de sus [métodos directos](direct-methods.md).

    * Se pone en pausa para que pueda examinar la salida del programa en la ventana **TERMINAL** y los eventos generados por el módulo en la ventana **SALIDA**.
    * Invoca los métodos directos para limpiar los recursos.

## <a name="generate-and-deploy-the-iot-edge-deployment-manifest"></a>Generación e implementación del manifiesto de implementación de IoT Edge 

El manifiesto de implementación define los módulos que se implementan en un dispositivo IoT Edge y los valores de configuración de dichos módulos. Siga estos pasos para generar un manifiesto a partir del archivo de plantilla y, a continuación, impleméntelo en el dispositivo IoT Edge.

Mediante Visual Studio Code, siga [estas instrucciones](../../iot-edge/tutorial-develop-for-linux.md#build-and-push-your-solution) para iniciar sesión en Docker. A continuación, seleccione **Build and Push IoT Edge Solution** (Compilar e insertar solución IoT Edge). Utilice src/edge/deployment.objectCounter.template.json para este paso.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording/build-push.png" alt-text="Compilar e insertar solución IoT Edge":::

Esta acción compila el módulo objectCounter para el recuento de objetos e inserta la imagen en la instancia de Azure Container Registry.

* Compruebe que ha definido las variables de entorno CONTAINER_REGISTRY_USERNAME_myacr y CONTAINER_REGISTRY_PASSWORD_myacr en el archivo .env.

Este paso crea el manifiesto de implementación de IoT Edge en src/edge/config/deployment.objectCounter.amd64.json. Haga clic con el botón derecho en ese archivo y seleccione **Create Deployment for Single Device** (Crear una implementación para un dispositivo individual).

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording/create-deployment-single-device.png" alt-text="Crear una implementación para un dispositivo":::

Si este es su primer tutorial con Video Analyzer, Visual Studio Code le pide que especifique la cadena de conexión de IoT Hub. Puede copiarla desde el archivo appsettings.json.

> [!NOTE]
> Es posible que se le pida que proporcione información del punto de conexión integrado del centro de IoT. Para obtener esa información, en Azure Portal, vaya a su centro de IoT y busque la opción **Puntos de conexión integrados** en el panel de navegación izquierdo. Haga clic ahí y busque el **punto de conexión compatible con el centro de eventos** en la sección **Punto de conexión compatible con el centro de eventos**. Copie y use el texto del cuadro. El punto de conexión será similar a este:  
    ```
    Endpoint=sb://iothub-ns-xxx.servicebus.windows.net/;SharedAccessKeyName=iothubowner;SharedAccessKey=XXX;EntityPath=<IoT Hub name>
    ```

A continuación, Visual Studio Code le pide que seleccione un dispositivo IoT Hub. Seleccione su dispositivo IoT Edge, que debería ser avasample-iot-edge-device.

En esta fase, se ha iniciado la implementación de los módulos perimetrales en el dispositivo IoT Edge.
En unos 30 segundos, actualice Azure IoT Hub en la sección inferior izquierda de Visual Studio Code. Debería ver que hay seis módulos implementados, llamados $edgeAgent, $edgeHub avaedge, rtspsim, yolov3 y objectCounter.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording/modules.png" alt-text="Cuatro módulos implementados":::

## <a name="run-the-program"></a>Ejecución del programa

1. En Visual Studio Code, abra la pestaña **Extensiones** (o presione Ctrl + Mayús + X) y busque Azure IoT Hub.
1. Haga clic con el botón derecho y seleccione la **Configuración de la extensión**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/extensions-tab.png" alt-text="Configuración de la extensión":::
1. Busque y habilite "Show Verbose Message" (Mostrar mensaje detallado).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/show-verbose-message.png" alt-text="Show Verbose Message"::: (Mostrar mensaje detallado)
1. Vaya a "src/cloud-to-device-console-app/operations.json".
1. En el nodo **pipelineTopologySet**, edite lo siguiente:


    `"pipelineTopologyUrl" : "https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/evr-hubMessage-video-sink/topology.json" `
    
1. Luego, en los nodos **livePipelineSet** y **pipelineTopologyDelete**, compruebe que el valor de **topologyName** coincide con el valor de la propiedad **name** de la topología de canalización anterior:

    `"pipelineTopologyName" : "EVRtoVideosOnObjDetect"`
1. Abra la [topología de canalización](https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/evr-hubMessage-videos/topology.json) en un explorador y vea que videoName está codificado de forma rígida como `sample-evr-video`. Para los fines de este tutorial, este valor es aceptable. En producción, tendría que asegurarse de que cada cámara RTSP única se graba en un recurso de vídeo con un nombre único.
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
            "topologyName": "EVRtoVideosOnObjDetect",
            "description": "Sample topology description",
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
    
   * Una llamada a livePipelineActivate para iniciar la canalización en directo y para iniciar el flujo de vídeo
   * Una segunda llamada a livePipelineList para mostrar que la canalización en directo está en estado de ejecución 
     
1. La salida de la ventana **TERMINAL** se pondrá en pausa ahora con el mensaje **Press Enter to continue** (Presione Entrar para continuar). No seleccione **Entrar** en este momento. Desplácese hacia arriba para ver las cargas de la respuesta JSON para los métodos directos que ha invocado.
1. Si ahora cambia a la ventana **SALIDA** de Visual Studio Code, verá que los mensajes se envían a IoT Hub mediante el módulo de Video Analyzer.

   Estos mensajes se describen en la sección siguiente.
1. La canalización en directo continúa para ejecutar y grabar el vídeo. El simulador RTSP sigue recorriendo el vídeo de origen. Para detener la grabación, vuelva a la ventana **TERMINAL** y seleccione **Entrar**. Se realiza la siguiente serie de llamadas para la limpieza de recursos:

   * Una llamada a livePipelineDeactivate para desactivar la canalización en directo.
   * Una llamada a livePipelineDelete para eliminar la canalización en directo.
   * Una llamada a pipelineTopologyDelete para eliminar la topología.
   * Una llamada final a pipelineTopologyList para mostrar que ahora la lista está vacía.

## <a name="interpret-the-results"></a>Interpretación de los resultados 

Al ejecutar la canalización en directo, el módulo de Video Analyzer envía determinados eventos de diagnóstico y operativos al centro de IoT Edge. Estos eventos son los mensajes que aparecen en la ventana **SALIDA** de Visual Studio Code. Contienen una sección body y una sección applicationProperties. Para comprender lo que representan estas secciones, consulte [Creación y lectura de mensajes de IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).

En los mensajes siguientes, se definen las propiedades de la aplicación y el contenido del cuerpo mediante el módulo de Video Analyzer.

## <a name="diagnostics-events"></a>Eventos de diagnóstico

### <a name="mediasessionestablished-event"></a>Evento MediaSessionEstablished 

Cuando se activa la canalización en directo, el nodo de origen RTSP intenta conectarse al servidor RTSP que se ejecuta en el contenedor del simulador RTSP. Si se realiza correctamente, imprime este evento:

```
[IoTHubMonitor] [9:42:18 AM] Message received from [avasample-iot-edge-device/avaadge]:
{
  "body": {
    "sdp&quot;: &quot;SDP:\nv=0\r\no=- 1586450538111534 1 IN IP4 XXX.XX.XX.XX\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/camera-300s.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.03.06\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/camera-300s.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets=XXXXXXXXXXXXXXXXXXXXXX\r\na=control:track1\r\n"
  },
  "applicationProperties": {
    "dataVersion": "1.0",
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{resource-group-name}/providers/microsoft.media/videoanalyzers/{ava-account-name}",
    "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/sources/rtspSource",
    "eventType": "Microsoft.VideoAnalyzers.Diagnostics.MediaSessionEstablished",
    "eventTime": "2021-04-09T09:42:18.1280000Z"
  }
}
```


* El mensaje es un evento de diagnóstico (MediaSessionEstablished). Indica que el nodo de origen RTSP (el sujeto) ha establecido una conexión con el simulador RTSP y ha comenzado a recibir una fuente en directo (simulada).
* La sección subject de applicationProperties hace referencia al nodo de la topología de canalización desde la que se generó el mensaje. En este caso, el mensaje se origina en el nodo de origen RTSP.
* La sección eventType de applicationProperties indica que se trata de un evento de diagnóstico.
* La sección eventTime indica la hora a la que se produjo el evento.
* La sección body contiene datos sobre el evento de diagnóstico que, en este caso, es el detalle de [SDP](https://en.wikipedia.org/wiki/Session_Description_Protocol).

## <a name="operational-events"></a>Eventos operativos

Después de que la canalización se ejecute durante un tiempo, podría recibir un evento del módulo objectCounter. 

```
[IoTHubMonitor] [5:53:44 PM] Message received from [avasample-iot-edge-device/objectCounter]:
{
  "body": {
    "count": 2
  },
  "applicationProperties": {
    "eventTime&quot;: &quot;2020-05-17T17:53:44.062Z"
  }
}
```

La sección applicationProperties contiene la hora del evento. Esta es la hora en la que el módulo objectCounter observó que los resultados del módulo yolov3 contenían objetos de interés (camiones).

Es posible que aparezcan más de estos eventos cuando se detecten otros camiones en el vídeo.

### <a name="recordingstarted-event"></a>Evento RecordingStarted

Casi inmediatamente después de que el contador de objetos envíe el evento, verá un evento de tipo **Microsoft.VideoAnalyzers.Pipeline.Operational.RecordingStarted**:

```
[IoTHubMonitor] [5:53:46 PM] Message received from [avasample-iot-edge-device/avaedge]:
{
  "body": {
    "outputType": "videoName",
    "outputLocation&quot;: &quot;sample-evr-video"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{resource-group-name}/providers/microsoft.media/videoanalyzers/{ava-account-name}",
    "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/sinks/videoSink",
    "eventType": "Microsoft.VideoAnalyzers.Pipeline.Operational.RecordingStarted",
    "eventTime": "2021-04-09T09:42:38.1280000Z",
    "dataVersion&quot;: &quot;1.0"
  }
}
```

La sección subject de applicationProperties hace referencia al nodo receptor de vídeo en la canalización en directo, que generó este mensaje. La sección body contiene información sobre la ubicación de salida. En este caso, es el nombre del recurso de vídeo de Video Analyzer en el que se graba el vídeo.

### <a name="recordingavailable-event"></a>Evento RecordingAvailable

Como sugiere su nombre, el evento RecordingStarted se envía cuando se inicia la grabación, pero puede que los datos multimedia aún no se hayan cargado en el recurso de vídeo. Cuando el nodo receptor de vídeo ha cargado medios, emite un evento de tipo **Microsoft.VideoAnalyzers.Pipeline.Operational.RecordingAvailable**:

```
[IoTHubMonitor] [[9:43:38 AM] Message received from [avasample-iot-edge-device/avaedge]:
{
  "body": {
    "outputType": "videoName",
    "outputLocation&quot;: &quot;sample-evr-video"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{resource-group-name}/providers/microsoft.media/videoanalyzers/{ava-account-name}",
    "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/sinks/videoSink",
    "eventType": "Microsoft.VideoAnalyzers.Pipeline.Operational.RecordingAvailable",
    "eventTime": "2021-04-09T09:43:38.1280000Z",
    "dataVersion&quot;: &quot;1.0"
  }
}
```

Este evento indica que se han escrito suficientes datos en el recurso de vídeo para que los reproductores o clientes inicien la reproducción del vídeo. La sección subject de applicationProperties hace referencia al nodo receptor de vídeo en la canalización en directo, que generó este mensaje. La sección body contiene información sobre la ubicación de salida. En este caso, es el nombre del recurso de Video Analyzer en el que se graba el vídeo.

### <a name="recordingstopped-event"></a>Evento RecordingStopped

Al desactivar la canalización en directo, el nodo receptor de vídeo deja de grabar los elementos multimedia. y emite este evento del tipo **Microsoft.Media.Graph.Operational.RecordingStopped**:

```
[IoTHubMonitor] [11:33:31 PM] Message received from [avasample-iot-edge-device/avaedge]:
{
  "body": {
    "outputType": "videoName",
    "outputLocation&quot;: &quot;sample-evr-video"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{resource-group-name}/providers/microsoft.media/videoanalyzers/{ava-account-name}",
    "subject": "/livePipelines/Sample-Pipeline-1/sinks/videoSink",
    "eventType": "Microsoft.VideoAnalyzers.Pipeline.Operational.RecordingStopped",
    "eventTime": "2021-04-10T11:33:31.051Z",
    "dataVersion&quot;: &quot;1.0"
  }
}
```

Este evento indica que se ha detenido la grabación. La sección subject de applicationProperties hace referencia al nodo receptor de vídeo en la canalización en directo, que generó este mensaje. La sección body contiene información sobre la ubicación de salida. En este caso, es el nombre del recurso de Video Analyzer en el que se graba el vídeo.

## <a name="video-analyzer-video-resource"></a>Recurso de vídeo de Video Analyzer

Para examinar el recurso de vídeo de Video Analyzer que se ha creado con la canalización en directo, inicie sesión en Azure Portal y vea el vídeo.
1. Abra el explorador web y vaya a [Azure Portal](https://portal.azure.com/). Introduzca sus credenciales para iniciar sesión en el portal. La vista predeterminada es el panel del servicio.
1. Busque la cuenta de Video Analyzer entre los recursos que tiene en la suscripción y abra el panel de cuentas.
1. Seleccione **Vídeos** en la sección **Video Analyzer**.

    <!--TODO: add image -- ![Video Analyzers videos]() ./media/event-based-video-recording-tutorial/videos.png -->
1. Encontrará un vídeo con el nombre `sample-evr-video`. Este es el nombre elegido en el archivo de topología de la canalización.
1. Seleccione el vídeo.
1. En la página de detalles del vídeo, seleccione la opción de reproducción. <!-- TODO: fix this-->

    <!--TODO: add image -- ![Video playback]() TODO: new screenshot is needed here -->


> [!NOTE]
> Puesto que el origen de vídeo era un contenedor que simula una fuente de cámara, las marcas de tiempo del vídeo son relativas a cuándo activó y desactivó la canalización en directo.
> 
## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a probar los otros tutoriales, debe mantener los recursos creados. En caso contrario, vaya a Azure Portal, vaya a los grupos de recursos, seleccione el grupo de recursos en el que ejecutó este tutorial y elimine el grupo de recursos.

## <a name="next-steps"></a>Pasos siguientes

* Use una [cámara IP](https://en.wikipedia.org/wiki/IP_camera) compatible con RTSP en lugar de utilizar el simulador RTSP. Puede buscar cámaras IP con compatibilidad con RTSP en la página de productos [compatibles con ONVIF](https://www.onvif.org/conformant-products/), buscando dispositivos que cumplan con los perfiles G, S o T.
* Use un dispositivo Linux AMD64 o x64 (en lugar de usar una máquina virtual Linux de Azure). El dispositivo debe estar en la misma red que la cámara IP. Siga las instrucciones de [Instalación del entorno de ejecución de Azure IoT Edge en Linux](../../iot-edge/how-to-install-iot-edge.md). A continuación, siga las instrucciones de la guía de inicio rápido [Implementación del primer módulo IoT Edge en un dispositivo virtual Linux](../../iot-edge/quickstart-linux.md) para registrar el dispositivo en Azure IoT Hub.
