---
title: Detección de movimiento y grabación de vídeo en dispositivos perimetrales - Azure
description: Use Azure Video Analyzer para analizar la fuente de vídeo en directo desde una cámara IP (simulada). Muestra cómo detectar si existe algún movimiento y, en caso afirmativo, graba un clip de vídeo MP4 en el sistema de archivos local del dispositivo perimetral. El inicio rápido usa una máquina virtual de Azure como dispositivo IoT Edge y emplea también una secuencia de vídeo en directo simulada.
ms.topic: quickstart
ms.date: 04/01/2021
zone_pivot_groups: video-analyzer-programming-languages
ms.openlocfilehash: 38886673f57c4142dc1fd8cd5bd397a375aca4db
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388523"
---
# <a name="quickstart-detect-motion-and-record-video-on-edge-devices"></a>Inicio rápido: Detección de movimiento y grabación de vídeo en dispositivos perimetrales

En este inicio rápido se muestra cómo usar Azure Video Analyzer para analizar la fuente de vídeo en directo desde una cámara IP (simulada). Muestra cómo detectar si existe algún movimiento y, en caso afirmativo, graba un clip de vídeo MP4 en el sistema de archivos local del dispositivo perimetral. El inicio rápido usa una máquina virtual de Azure como dispositivo IoT Edge y emplea también una secuencia de vídeo en directo simulada.

## <a name="prerequisites"></a>Requisitos previos

::: zone pivot="programming-language-csharp"
[!INCLUDE [prerequisites](includes/detect-motion-record-video-edge-devices/csharp/prerequisites.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [prerequisites](includes/detect-motion-record-video-edge-devices/python/prerequisites.md)]
::: zone-end

## <a name="review-the-sample-video"></a>Revisión del vídeo de ejemplo

A medida que configura los recursos de Azure de este inicio rápido, se copiará un vídeo (corto) de un área de aparcamiento en la máquina virtual Linux en Azure que se usa como dispositivo IoT Edge. Este archivo de vídeo se usará para simular un streaming en vivo para este tutorial.

Abra una aplicación como [VLC Media Player](https://www.videolan.org/vlc/), seleccione Ctrl + N y pegue [este vínculo](https://lvamedia.blob.core.windows.net/public/lots_015.mkv) en el vídeo sobre el área de aparcamiento para iniciar la reproducción. Hacia la marca de 5 segundos, un coche blanco se desplaza por el aparcamiento.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4LUbN]

Realice los pasos que se indican a continuación para usar Video Analyzer para detectar el movimiento del coche y grabar un clip de vídeo que empiece en torno a la marca de los 5 segundos.

## <a name="examine-and-edit-the-sample-files"></a>Examen y edición de los archivos de ejemplo

::: zone pivot="programming-language-csharp"
[!INCLUDE [prerequisites](includes/detect-motion-record-video-edge-devices/csharp/sample-files.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [prerequisites](includes/detect-motion-record-video-edge-devices/python/sample-files.md)]
::: zone-end

## <a name="generate-and-deploy-the-deployment-manifest"></a>Generación e implementación del manifiesto de implementación

::: zone pivot="programming-language-csharp"
[!INCLUDE [prerequisites](includes/detect-motion-record-video-edge-devices/csharp/generate-deploy-deployment-manifest.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [prerequisites](includes/detect-motion-record-video-edge-devices/python/generate-deploy-deployment-manifest.md)]
::: zone-end

## <a name="run-the-sample-program"></a>Ejecución del programa de ejemplo

1. En Visual Studio Code, abra la pestaña Extensiones (o presione Ctrl + Mayús + X) y busque Azure IoT Hub.
1. Haga clic con el botón derecho y seleccione la opción Configuración de la extensión.

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/vscode-common-screenshots/extension-settings.png" alt-text= "Extension settings":::

1. Busque y habilite "Show Verbose Message" (Mostrar mensaje detallado).

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/vscode-common-screenshots/verbose-message.png" alt-text= "Show Verbose Message":::

1. Para iniciar una sesión de depuración, seleccione la tecla F5. La ventana **TERMINAL** muestra algunos mensajes.
1. El código de _operations.json_ llama a los métodos directos `pipelineTopologyList` y `livePipelineList`. Si ha limpiado los recursos después de los inicios rápidos anteriores, este proceso devolverá listas vacías y, a continuación, se pausará. Presione la tecla Entrar.

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

   La ventana TERMINAL muestra el siguiente conjunto de llamadas al método directo:

   - Una llamada a `pipelineTopologySet` que utiliza la instancia de pipelineTopologyUrl.
   - Una llamada a `livePipelineSet` que usa el cuerpo siguiente:

     ```
     {
       "@apiVersion": "1.0",
       "name": "Sample-Pipeline-1",
       "properties": {
         "topologyName": "EVRToFilesOnMotionDetection",
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

   - Una llamada a `livePipelineActivate` que activa la canalización en directo y el flujo de vídeo.
   - Una segunda llamada a `livePipelineList` que muestra que la canalización en directo está en estado de ejecución.

1. La salida de la ventana TERMINAL se pondrá en pausa con el mensaje "Press Enter to continue" (Presione Entrar para continuar). No seleccione Entrar todavía. Desplácese hacia arriba para ver las cargas de la respuesta JSON para los métodos directos que ha invocado.
1. Cambie a la ventana SALIDA de Visual Studio Code. Verá los mensajes que el módulo perimetral de Video Analyzer está enviando al centro de IoT. En la siguiente sección de este inicio rápido se analizan estos mensajes.
1. La topología de canalización continúa ejecutándose y se imprimen los resultados. El simulador RTSP sigue recorriendo el vídeo de origen. Para detener la topología de canalización, vuelva a la ventana TERMINAL y seleccione Entrar.

La siguiente serie de llamadas limpia los recursos:

- Una llamada a `livePipelineDeactivate` desactiva la canalización en directo.
- Una llamada a `livePipelineDelete` elimina la canalización en directo.
- Una llamada a `pipelineTopologyDelete` elimina la topología.
- Una llamada final a `pipelineTopologyList` muestra que la lista está ahora vacía.

## <a name="interpret-results"></a>Interpretación de los resultados

Cuando se ejecuta la topología de canalización, los resultados del nodo del procesador de detección del movimiento pasan a través del nodo de receptor de IoT Hub al centro de IoT. Los mensajes que aparecen en la ventana SALIDA de Visual Studio Code contienen una sección "body" y una sección "applicationProperties". Para más información, consulte [Creación y lectura de mensajes de IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).

En los mensajes siguientes, el módulo perimetral de Video Analyzer define las propiedades de la aplicación y el contenido del cuerpo.

### <a name="mediasessionestablished-event"></a>Evento MediaSessionEstablished

Cuando se crean instancias de la topología de canalización, el nodo de origen RTSP intenta conectarse al servidor RTSP que se ejecuta en el contenedor rtspsim-live555. Si la conexión se realiza correctamente, se imprime el evento siguiente.

```
[IoTHubMonitor] [9:42:18 AM] Message received from [ava-sample-device/avaadge]:
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

En la salida anterior:

- El mensaje es un evento de diagnóstico (MediaSessionEstablished). Indica que el nodo de origen RTSP (el sujeto) ha establecido una conexión con el simulador RTSP y ha comenzado a recibir una fuente en directo (simulada).
- La sección "subject" de applicationProperties hace referencia al nodo de la topología de canalización desde la que se generó el mensaje. En este caso, el mensaje se origina en el nodo de origen RTSP.
- En applicationProperties, el valor de "eventType" indica que se trata de un evento de diagnóstico.
- El valor eventTime es la hora en que se produjo el evento.
- La sección "body" contiene datos sobre el evento de diagnóstico. En este caso, los datos incluyen detalles sobre el protocolo de descripción de sesiones (SDP).

### <a name="recordingstarted-event"></a>Evento RecordingStarted

Cuando se detecta movimiento, se activa el nodo de procesador de la puerta de señales y el nodo de receptor de archivos de la topología de canalización empieza a escribir un archivo MP4. El nodo del receptor de archivos envía un evento operativo. El tipo se establece en "motion" para indicar que se trata de un resultado del procesador de detección del movimiento. El valor eventTime es la hora UTC en la que se produjo el movimiento. Para obtener más información sobre este proceso, consulte la sección [Información general](detect-motion-record-video-edge-devices.md#overview) de este inicio rápido.

A continuación se muestra un ejemplo de este mensaje:

```
[IoTHubMonitor] [05:37:27 AM] Message received from [avasample-iot-edge-device/avaedge]:
{
  "body": {
    "outputType": "filePath",
    "outputLocation&quot;: &quot;/var/media/sampleFilesFromEVR-filesinkOutput-20210511T053726Z.mp4"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{resource-group-name}/providers/microsoft.media/videoAnalyzers/{ava-account-name}",
    "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/sinks/fileSink",
    "eventType": "Microsoft.VideoAnalyzer.Operational.RecordingStarted",
    "eventTime": "2021-05-11T05:37:27.713Z",
    "dataVersion&quot;: &quot;1.0"
  }
}
```

En el mensaje anterior:

- La sección "subject" de applicationProperties hace referencia al nodo de la canalización desde el que se generó el mensaje. En este caso, el mensaje se origina en el nodo del receptor de archivos.
- En applicationProperties, eventType indica que se trata de un evento operativo.
- El valor de eventTime es la hora en que se produjo el evento. Este tiempo es de 5 a 6 segundos después del evento MediaSessionEstablished y después de que empiece el flujo del vídeo. Esta hora se corresponde a la marca de 5 o 6 segundos cuando el [coche empezó a moverse](#review-the-sample-video) por el aparcamiento.
- La sección "body" contiene datos sobre el evento operativo. En este caso, los datos incluyen outputType y outputLocation.
- La variable outputType indica que esta información es sobre la ruta de acceso del archivo.
- El valor outputLocation es la ubicación del archivo MP4 en el módulo perimetral.

### <a name="recordingstopped-and-recordingavailable-events"></a>Eventos RecordingStopped y RecordingAvailable

Si examina las propiedades del nodo de procesador de la puerta de señales en la [topología de canalización](pipeline.md) puede ver que los tiempos de activación se han establecido en 5 segundos. Por tanto, 5 segundos después de que se reciba el evento **RecordingStarted**, recibirá lo siguiente:

- Evento **RecordingStopped**, que indica que la grabación se ha detenido
- Evento **RecordingAvailable**, que indica que ahora se puede usar el archivo MP4 para su visualización

Normalmente, los dos eventos se emiten con unos segundos de diferencia.

## <a name="play-the-mp4-clip"></a>Reproducción del clip de MP4

Los archivos MP4 se escriben en un directorio del dispositivo perimetral que configuró en el archivo .env mediante la clave VIDEO_OUTPUT_FOLDER_ON_DEVICE. Si ha usado el valor predeterminado, los resultados deberían estar en la carpeta /var/media/ .

Para reproducir el clip de MP4:

1. Vaya al grupo de recursos, busque la máquina virtual y conéctese a ella.

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/detect-motion-record-video-edge-devices/sample-iot-edge-device.png" alt-text= "Edge device":::

1. Inicie sesión con las credenciales que se generaron al configurar los recursos de Azure.
1. En el símbolo del sistema, vaya al directorio correspondiente. La ubicación predeterminada es /var/media. Debería ver los archivos MP4 en el directorio.

1. Use [Secure Copy (SCP)](../../virtual-machines/linux/copy-files-to-linux-vm-using-scp.md) para copiar los archivos en la máquina local.
1. Reproduzca los archivos mediante [VLC Media Player](https://www.videolan.org/vlc/) o cualquier otro reproductor MP4.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si su intención es probar los demás inicios rápidos, conserve los recursos creados. En caso contrario, en Azure Portal, vaya a los grupos de recursos, seleccione el grupo de recursos en el que ejecutó este inicio rápido y, a continuación, elimine todos los recursos.

## <a name="next-steps"></a>Pasos siguientes

- Siga las indicaciones del inicio rápido [Ejecución de Azure Video Analyzer con su propio modelo](analyze-live-video-use-your-model-http.md) para aplicar la inteligencia artificial a las fuentes de vídeo en directo.
- Revise los desafíos adicionales para los usuarios avanzados:

  - Use una [cámara IP](https://en.wikipedia.org/wiki/IP_camera) compatible con RTSP en lugar de utilizar el simulador RTSP. Puede encontrar cámaras IP compatibles con RTSP en la página de [productos compatibles con ONVIF](https://www.onvif.org/conformant-products/). Busque dispositivos que cumplan con los perfiles G, S o T.
  - Use un dispositivo Linux AMD64 o x64 en lugar de usar una máquina virtual Linux en Azure. El dispositivo debe estar en la misma red que la cámara IP. Siga las instrucciones de [Instalación del entorno de ejecución de Azure IoT Edge en Linux](../../iot-edge/how-to-install-iot-edge.md?view=iotedge-2020-11&preserve-view=true). A continuación, siga las instrucciones que se encuentran en [Implementación del primer módulo IoT Edge en un dispositivo virtual Linux](../../iot-edge/quickstart-linux.md?view=iotedge-2020-11&preserve-view=true) para registrar el dispositivo con Azure IoT Hub.
