---
title: 'Tutorial: Grabación continua de vídeo y reproducción: Azure Video Analyzer'
description: En este tutorial aprenderá a usar Azure Video Analyzer para grabar vídeo de forma continua en la nube y reproducir esa grabación.
ms.topic: tutorial
ms.date: 04/01/2021
ms.openlocfilehash: d62b9f413f0613d8152d3b0106fd398f89d3b95c
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110384320"
---
# <a name="tutorial-continuous-video-recording-and-playback"></a>Tutorial: Grabación continua de vídeo y reproducción

En este tutorial aprenderá a usar Azure Video Analyzer para [grabar vídeo de forma continua](continuous-video-recording.md) (CVR) en la nube y reproducir esa grabación. Esta funcionalidad resulta útil en escenarios de seguridad y cumplimiento, en los que es necesario mantener un archivo de las imágenes de una cámara durante días, semanas, meses o incluso años. 

En este tutorial, aprenderá lo siguiente:

> [!div class="checklist"]
> * Configuración de los recursos correspondientes
> * Examen del código que realiza la CVR
> * Ejecución del código de ejemplo
> * Examen de los resultados y visualización del vídeo

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="suggested-pre-reading"></a>Sugerencias antes de la lectura  

Consulte estos artículos antes de empezar:

* [Introducción a Video Analyzer](overview.md)
* [Terminología de Video Analyzer](terminology.md)
* [Conceptos sobre la canalización de Video Analyzer](pipeline.md) 
* [Escenarios de grabación continua de vídeo](continuous-video-recording.md)

## <a name="prerequisites"></a>Requisitos previos

Los requisitos previos de este tutorial son los siguientes: [!INCLUDE [prerequisites](./includes/common-includes/csharp-prerequisites.md)]

## <a name="set-up-azure-resources"></a>Configuración de los recursos de Azure

[![Implementación en Azure](https://aka.ms/deploytoazurebutton)](https://aka.ms/ava-click-to-deploy)
[!INCLUDE [resources](./includes/common-includes/azure-resources.md)]

## <a name="concepts"></a>Conceptos

Como se explica en [este](pipeline.md) artículo, una canalización del analizador de vídeo le permite definir lo siguiente:

- Desde dónde se deben capturar los elementos multimedia.
- Cómo se deben procesar.
- Dónde se deben entregar los resultados. 
 
 Para efectuar la grabación de vídeo continua, debe capturar el vídeo con una cámara compatible con RTSP y grabarlo continuamente en un [recurso de vídeo](terminology.md#video). El diagrama muestra una representación gráfica de esa canalización. 

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/continuous-video-recording/continuous-video-recording-overview.svg" alt-text="Canalización de Azure Video Analyzer para CVR":::
<!-- ./media/continuous-video-recording-tutorial/continuous-video-recording-overview.svg -->

En este tutorial, usará un módulo perimetral creado con [Live555 Media Server](https://github.com/Azure/video-analyzer/tree/main/edge-modules/sources/rtspsim-live555) para simular una cámara RTSP. Dentro de la canalización, usará un nodo de [origen RTSP](pipeline.md#rtsp-source) para obtener la fuente en directo y enviar ese vídeo al [nodo receptor del vídeo](pipeline.md#video-sink), que graba el vídeo en la cuenta de Video Analyzer. En este tutorial se va a usar un [vídeo de ejemplo de un ](https://lvamedia.blob.core.windows.net/public/camera-300s.mkv)de vídeo de ejemplo de un cruce de autopistas.

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

En Visual Studio Code, vaya a la carpeta src/cloud-to-device-console-app. Aquí podrá ver el archivo appsettings.json que creó junto con algunos otros archivos:

* **c2d-console-app.csproj**: archivo de proyecto de Visual Studio Code.
* **operations.json**: en este archivo se enumeran las distintas operaciones que se ejecutarán.
* **Program.cs**: código del programa de ejemplo, el cual:
    * Carga la configuración de la aplicación.
    * Invoca métodos directos expuestos por el módulo perimetral de Video Analyzer. Puede usar el módulo para analizar secuencias de vídeo en directo mediante la invocación de sus [métodos directos](direct-methods.md).
    * Se pone en pausa para que pueda examinar la salida del programa en la ventana **TERMINAL** y los eventos generados por el módulo en la ventana **SALIDA**.
    * Invoca los métodos directos para limpiar los recursos.

## <a name="run-the-program"></a>Ejecución del programa 

1. En Visual Studio Code, vaya a src/cloud-to-device-console-app/operations.json.
1. En el nodo **pipelineTopologySet**, modifique lo siguiente:

    `"pipelineTopologyUrl" : "https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/cvr-video-sink/topology.json" `
1. Luego, en los nodos **livePipelineSet** y **pipelineTopologyDelete**, compruebe que el valor de **topologyName** coincide con el valor de la propiedad **name** de la topología de la canalización anterior:

    `"topologyName" : "CVRToVideoSink"`  
1. Abra la [topología de la canalización](https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/cvr-video-sink/topology.json) en un explorador y vea que videoName está codificado de forma rígida como `sample-cvr-video`. Para los fines de este tutorial, este valor es aceptable. En producción, tendría que asegurarse de que cada cámara RTSP única se graba en un recurso de vídeo con un nombre único.
1. Para iniciar una sesión de depuración, seleccione F5. Verá algunos mensajes impresos en la ventana **TERMINAL**.
1. El archivo operations.json se inicia con llamadas a `pipelineTopologyList` y `livePipelineList`. Si ha limpiado los recursos después de los inicios rápidos o los tutoriales anteriores, esta acción devolverá listas vacías y entrará en pausa para que seleccione **Entrar**, como se muestra a continuación:

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
   * Una llamada a `pipelineTopologySet` con el elemento `topologyUrl` anterior.
   * Una llamada a `livePipelineSet` con el siguiente cuerpo:
     
     ```
     {
       "@apiVersion": "1.0",
       "name": "Sample-Pipeline-1",
       "properties": {
         "topologyName": "CVRToVideoSink",
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
   * Una llamada a `livePipelineActivate` para iniciar la canalización en directo y comenzar el flujo de vídeo.
   * Una segunda llamada a `livePipelineList` para mostrar que la canalización en directo está en estado de ejecución. 
1. La salida de la ventana **TERMINAL** se pondrá en pausa ahora con el mensaje **Press Enter to continue** (Presione Entrar para continuar). No seleccione **Entrar** en este momento. Desplácese hacia arriba para ver las cargas de la respuesta JSON para los métodos directos que ha invocado.
1. Si ahora cambia a la ventana **SALIDA** de Visual Studio Code, verá que los mensajes se envían a IoT Hub mediante el módulo perimetral de Video Analyzer.


   Estos mensajes se describen en la sección siguiente.
1. La canalización en directo continúa en ejecución y graba el vídeo. El simulador RTSP sigue recorriendo el vídeo de origen. Para detener la grabación, vuelva a la ventana **TERMINAL** y seleccione **Entrar**. Se realiza la siguiente serie de llamadas para la limpieza de recursos:

   * Una llamada a `livePipelineDeactivate` para desactivar la canalización en directo.
   * Una llamada a `livePipelineDelete` para eliminar la canalización en directo.
   * Una llamada a `pipelineTopologyDelete` para eliminar la topología.
   * Una llamada final a `pipelineTopologyList` para mostrar que la lista está ahora vacía.

## <a name="interpret-the-results"></a>Interpretación de los resultados 

Al ejecutar la canalización en directo, el módulo perimetral de Video Analyzer envía determinados eventos de diagnóstico y operativos al centro de IoT Edge. Estos eventos son los mensajes que aparecen en la ventana **SALIDA** de Visual Studio Code. Estos mensajes contienen dos secciones: `body` y `applicationProperties`. Para comprender lo que representan estas secciones, consulte [Creación y lectura de mensajes de IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).

En los mensajes siguientes, se definen las propiedades de la aplicación y el contenido del cuerpo mediante el módulo perimetral de Video Analyzer.


## <a name="diagnostics-events"></a>Eventos de diagnóstico 

### <a name="mediasession-established-event"></a>Evento MediaSessionEstablished

Cuando se activa la canalización en directo, el nodo de origen RTSP intenta conectarse al servidor RTSP que se ejecuta en el módulo rtspsim. Si se realiza correctamente, imprime este evento:

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

* El mensaje es un evento de diagnóstico (**Microsoft.VideoAnalyzers.Diagnostics.MediaSessionEstablished**). Indica que el nodo de origen RTSP (el sujeto) ha establecido una conexión con el simulador RTSP y ha comenzado a recibir una fuente en directo (simulada).
* La sección subject de applicationProperties hace referencia al nodo de la topología de la canalización desde la que se generó el mensaje. En este caso, el mensaje se origina en el nodo de origen RTSP.
* La sección eventType de applicationProperties indica que se trata de un evento de diagnóstico.
* La sección eventTime indica la hora a la que se produjo el evento.
* La sección body contiene datos sobre el evento de diagnóstico que, en este caso, es el detalle de [SDP](https://en.wikipedia.org/wiki/Session_Description_Protocol).

## <a name="operational-events"></a>Eventos operativos 

### <a name="recordingstarted-event"></a>Evento RecordingStarted

Cuando el nodo receptor de vídeo comienza a grabar elementos multimedia, emite este evento de tipo **Microsoft.VideoAnalyzers.Pipeline.OperationalRecordingStarted**:

```
[IoTHubMonitor] [9:42:38 AM] Message received from [avasample-iot-edge-device/avaedge]:
{
  "body": {
    "outputType": "videoName",
    "outputLocation&quot;: &quot;sample-cvr-video"
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

Como sugiere su nombre, el evento **RecordingStarted** se envía cuando se inicia la grabación, pero puede que los datos multimedia aún no se hayan cargado en el recurso de vídeo. Cuando el nodo receptor de vídeo ha cargado los elementos multimedia, emite un evento de tipo **Microsoft.VideoAnalyzers.Pipeline.Operational.RecordingAvailable**:

```
[IoTHubMonitor] [[9:43:38 AM] Message received from [ava-sample-device/avaedge]:
{
  "body": {
    "outputType": "videoName",
    "outputLocation&quot;: &quot;sample-cvr-video"
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

Al desactivar la canalización en directo, el nodo receptor de vídeo deja de grabar los elementos multimedia y emite este evento de tipo **Microsoft.Media.Graph.Operational.RecordingStopped**:

```
[IoTHubMonitor] [11:33:31 PM] Message received from [avasample-iot-edge-device/avaedge]:
{
  "body": {
    "outputType": "videoName",
    "outputLocation&quot;: &quot;sample-cvr-video"
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

## <a name="playing-back-the-recording"></a>Reproducción de la grabación

Para examinar el recurso de vídeo de Video Analyzer que se ha creado con la canalización en directo, inicie sesión en Azure Portal y vea el vídeo.

1. Abra el explorador web y vaya a [Azure Portal](https://portal.azure.com/). Introduzca sus credenciales para iniciar sesión en el portal. La vista predeterminada es el panel del servicio.
1. Busque la cuenta de Video Analyzer entre los recursos que tiene en la suscripción y abra el panel de cuentas.
1. Seleccione **Vídeos** en la sección **Video Analyzer**.
1. Encontrará un vídeo con el nombre `sample-cvr-video`. Este es el nombre elegido en el archivo de topología de la canalización.
1. Seleccione el vídeo.
1. Se abrirá la página de detalles del vídeo y la reproducción se iniciará automáticamente.

> [!NOTE]
> Puesto que el origen de vídeo era un contenedor que simula una fuente de cámara, las marcas de tiempo del vídeo son relativas a cuándo se activó y desactivó la canalización en directo.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a probar los otros tutoriales, debe mantener los recursos creados. En caso contrario, vaya a Azure Portal, vaya a los grupos de recursos, seleccione el grupo de recursos en el que ejecutó este tutorial y elimine el grupo de recursos.

## <a name="next-steps"></a>Pasos siguientes

* Use una [cámara IP](https://en.wikipedia.org/wiki/IP_camera) compatible con RTSP en lugar de utilizar el simulador RTSP. Puede buscar cámaras IP con compatibilidad con RTSP en la página de productos [compatibles con ONVIF](https://www.onvif.org/conformant-products/), buscando dispositivos que cumplan con los perfiles G, S o T.
* Use un dispositivo Linux AMD64 o x64 (en lugar de usar una máquina virtual Linux de Azure). El dispositivo debe estar en la misma red que la cámara IP. Siga las instrucciones de [Instalación del entorno de ejecución de Azure IoT Edge en Linux](../../iot-edge/how-to-install-iot-edge.md). A continuación, siga las instrucciones de la guía de inicio rápido [Implementación del primer módulo IoT Edge en un dispositivo virtual Linux](../../iot-edge/quickstart-linux.md) para registrar el dispositivo en Azure IoT Hub.
