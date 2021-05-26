---
title: Detectar si los objetos cruzan una línea virtual en un vídeo en directo con Azure Video Analyzer
description: En este inicio rápido se muestra cómo usar Azure Video Analyzer para detectar si los objetos cruzan una línea a partir de una fuente de vídeo en directo desde una cámara IP (simulada).
ms.topic: tutorial
ms.date: 05/18/2021
ms.openlocfilehash: 8cca0aca44f2cb2ebdbee7869d189b0cd2b2451f
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110465667"
---
# <a name="tutorial-detect-when-objects-cross-a-virtual-line-in-a-live-video"></a>Tutorial: Detectar si los objetos cruzan una línea virtual en un vídeo en directo

En este tutorial se muestra cómo usar Azure Video Analyzer para detectar si los objetos cruzan una línea virtual a partir de una fuente de vídeo en directo desde una cámara IP (simulada). Verá cómo aplicar un modelo de Computer Vision para detectar objetos en un subconjunto de los fotogramas de la fuente de vídeo en directo. A continuación, puede usar un nodo de seguimiento de objetos para realizar un seguimiento de esos objetos en los otros fotogramas y enviar los resultados a un nodo de detección de cruce de líneas.

El nodo de detección de cruce de líneas permite detectar si los objetos cruzan la línea virtual. Los eventos contienen la dirección (en el sentido de las agujas del reloj o en sentido contrario a las agujas del reloj) y un contador total por dirección.  

En este tutorial se usa una máquina virtual de Azure como dispositivo IoT Edge y se emplea una secuencia de vídeo en directo simulada.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [prerequisites](./includes/common-includes/csharp-prerequisites.md)]

## <a name="set-up-azure-resources"></a>Configuración de los recursos de Azure

[![Implementación en Azure](https://aka.ms/deploytoazurebutton)](https://aka.ms/ava-click-to-deploy)

[!INCLUDE [resources](./includes/common-includes/azure-resources.md)]

## <a name="overview"></a>Información general

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/track-objects-live-video/line-crossing-tracker-topology.png" alt-text="Detectar si los objetos cruzan una línea virtual en el vídeo en directo.":::

En este diagrama se muestra el flujo de las señales en este tutorial. Un [módulo perimetral](https://github.com/Azure/video-analyzer/tree/main/edge-modules/sources/rtspsim-live555) simula una cámara IP que hospeda un servidor de protocolo RTSP. Un nodo de [origen RTSP](pipeline.md#rtsp-source) extrae la fuente de vídeo de este servidor y envía fotogramas de vídeo al nodo del [procesador de extensiones HTTP](pipeline.md#http-extension-processor).

El nodo de extensión HTTP desempeña el rol de un proxy. Convierte cada 10 fotogramas de vídeo en el tipo de imagen especificado. Luego, retransmite la imagen mediante HTTP a otro módulo perimetral que ejecuta un modelo de IA detrás de un punto de conexión HTTP. En este ejemplo, el módulo perimetral se crea mediante el modelo [YOLOv3](https://github.com/Azure/video-analyzer/tree/main/edge-modules/extensions/yolo/yolov3), que puede detectar muchos tipos de objetos. El nodo procesador de extensiones HTTP recopila los resultados de la detección y envía estos resultados y todos los fotogramas del vídeo (no solo cada 10 fotogramas) al nodo de seguimiento de objetos. El nodo de seguimiento de objetos usa técnicas de flujo óptico para realizar un seguimiento del objeto en los 9 fotogramas a los que no se les aplicó el modelo de IA. El nodo de seguimiento publica sus resultados en el nodo receptor de mensajes de IoT Hub. Este nodo [receptor de mensajes de IoT Hub](pipeline.md#iot-hub-message-sink) envía esos eventos al [centro de IoT Edge](../../iot-fundamentals/iot-glossary.md?view=iotedge-2020-11&preserve-view=true#iot-edge-hub).

El nodo de detección de cruce de líneas recibirá los resultados del nodo de seguimiento de objetos del flujo ascendente. La salida del nodo de seguimiento de objetos contiene las coordenadas de los objetos detectados. El nodo de detección de cruce de líneas evalúa estas coordenadas con respecto a las coordenadas de la línea. Cuando los objetos crucen la línea, el nodo de detección de cruce de líneas emitirá un evento. Los eventos se envían al receptor de mensajes del centro de IoT Edge. 

En este tutorial, aprenderá lo siguiente:

1. Configure un entorno de desarrollo.
1. Implementará los módulos perimetrales necesarios.
1. Creará e implementará la canalización en directo.
1. Interpretará los resultados.
1. Comprender cómo calcular las coordenadas.
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
    * El módulo **avaextension**, que es el modelo de detección de objetos YOLOv3 que aplica Computer Vision a las imágenes y devuelve varias clases de tipos de objeto.

        > [!div class="mx-imgBorder"]
        > :::image type="content" source="./media/vscode-common-screenshots/avaextension.png" alt-text= "YoloV3 object detection model":::


## <a name="create-and-deploy-the-live-pipeline"></a>Creación e implementación de la canalización en directo

### <a name="review-sample-video"></a>Revisión del vídeo de ejemplo

Cuando configura los recursos de Azure, se copia un vídeo corto del tráfico en la autopista en la máquina virtual Linux en Azure que usa como dispositivo IoT Edge. En este tutorial se usa un archivo de vídeo para simular una secuencia en directo.

Abra una aplicación como [VLC Media Player](https://www.videolan.org/vlc/). Seleccione Ctrl+N y, a continuación, pegue un vínculo al [vídeo de ejemplo de cruce de autopista](https://lvamedia.blob.core.windows.net/public/camera-300s.mkv) para iniciar la reproducción. Verá las imágenes de muchos vehículos moviéndose en el tráfico de una autopista.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4LTY4]

### <a name="examine-and-edit-the-sample-files"></a>Examen y edición de los archivos de ejemplo

En Visual Studio Code, vaya a la carpeta src/cloud-to-device-console-app. Aquí podrá ver el archivo appsettings.json que creó junto con algunos otros archivos:

* **c2d-console-app.csproj**: archivo de proyecto de Visual Studio Code.
* **operations.json**: en este archivo se enumeran las distintas operaciones que se ejecutarán.
* **Program.cs**: código del programa de ejemplo, el cual:
    * Carga la configuración de la aplicación.
    * Invoca los métodos directos expuestos por el módulo perimetral de Video Analyzer. Puede usar el módulo para analizar secuencias de vídeo en directo mediante la invocación de sus [métodos directos](direct-methods.md).
    * Se pone en pausa para que pueda examinar la salida del programa en la ventana **TERMINAL** y los eventos generados por el módulo en la ventana **SALIDA**.
    * Invoca los métodos directos para limpiar los recursos.

1. Edite el archivo operations.json:
    
    * Cambie el vínculo a la topología de la canalización:
    * "pipelineTopologyUrl": "https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/line-crossing/topology.json"
    * En livePipelineSet, edite el nombre de la topología para que coincida con el valor del vínculo anterior:
    * "topologyName": "LineCrossingWithHttpExtension"
    * En `pipelineTopologyDelete`, edite el nombre:
    * "name": "LineCrossingWithHttpExtension"
    
Abra la dirección URL de la topología de canalización en un explorador y examine la configuración del nodo de extensión HTTP.

```
   "samplingOptions":{
       "skipSamplesWithoutAnnotation":"false",
       "maximumSamplesPerSecond":"2"
   }
```

En este caso, `skipSamplesWithoutAnnotation` se establece en `false` porque el nodo de extensiones debe pasar por todos los fotogramas, tanto si tienen resultados de inferencia como si no, hasta el nodo de seguimiento de objetos de nivel inferior. El seguimiento de objetos tiene capacidad para realizar un seguimiento de objetos de más de 15 fotogramas, aproximadamente. Si el vídeo en directo se ejecuta a una velocidad de 30 fotogramas por segundo, significa que se deben enviar al menos dos fotogramas por segundo al servidor HTTP para la inferencia; por lo tanto, `maximumSamplesPerSecond` se establece en 2. De hecho, serán 15 fotogramas por segundo.

Observe también los marcadores de posición de los parámetros `linecrossingName` y `lineCoordinates` del nodo de detección de cruce de líneas. Hemos proporcionado valores predeterminados para estos parámetros, pero se pueden sobrescribir mediante el uso del archivo operations.json. Observe cómo se pasan otros parámetros del archivo operations.json a una topología (es decir, dirección URL de RTSP).  

 
## <a name="run-the-sample-program"></a>Ejecución del programa de ejemplo

1. Para iniciar una sesión de depuración, seleccione la tecla F5. Verá mensajes impresos en la ventana **TERMINAL**.
1. El código de operations.json comienza con llamadas a los métodos directos `pipelineTopologyList` y `livePipelineList`. Si ha limpiado los recursos tras haber completado los tutoriales e inicios rápidos anteriores, este proceso devolverá listas vacías y, después, se pausará. Para continuar, seleccione la tecla Entrar.
    
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
      "topologyName": "LineCrossingWithHttpExtension",
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
1. La salida de la ventana TERMINAL se pondrá en pausa con el mensaje Press Enter to continue (Presione Entrar para continuar). No seleccione Entrar todavía. Desplácese hacia arriba para ver las cargas de la respuesta JSON para los métodos directos que ha invocado.
1. Cambie a la ventana SALIDA de Visual Studio Code. Verá los mensajes que el módulo de Video Analyzer está enviando al centro de IoT. En la siguiente sección de este tutorial se analizan estos mensajes.
1. La canalización en directo continúa ejecutándose y se imprimen los resultados. El simulador RTSP sigue recorriendo el vídeo de origen. Para detener la canalización en directo, vuelva a la ventana **TERMINAL** y seleccione Entrar.
1. La siguiente serie de llamadas limpia los recursos:

    * Una llamada a `livePipelineDeactivate` desactiva la canalización en directo.
    * Una llamada a `livePipelineDelete` elimina la canalización en directo.
    * Una llamada a `pipelineTopologyDelete` elimina la topología de canalización.
    * Una llamada final a `pipelineTopologyList` muestra que la lista está vacía.
    
## <a name="interpret-results"></a>Interpretación de los resultados

Cuando se ejecuta la canalización en directo, los resultados del nodo procesador de extensiones HTTP pasan por el nodo receptor de mensajes de IoT Hub hasta llegar al centro de IoT. Los mensajes que aparecen en la ventana **SALIDA** contienen una sección `body` y una sección `applicationProperties`. Para más información, consulte [Creación y lectura de mensajes de IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).

En los mensajes siguientes, el módulo de Video Analyzer define las propiedades de la aplicación y el contenido del cuerpo.

### <a name="mediasessionestablished-event"></a>Evento MediaSessionEstablished

Cuando se activa una canalización en directo, el nodo de origen de RTSP intenta conectarse al servidor RTSP que se ejecuta en el contenedor rtspsim-live555. Si la conexión se realiza correctamente, se imprime el evento siguiente. El tipo de evento es **MediaSessionEstablished**.

```
[IoTHubMonitor] [9:42:18 AM] Message received from [avasample-iot-edge-device/avaedge]:
{  "body": {
    "sdp&quot;: &quot;SDP:\nv=0\r\no=- 1586450538111534 1 IN IP4 nnn.nn.0.6\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/cafetaria-01.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.03.06\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/camera-300s.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets=Z00AKeKQCgC3YC3AQEBpB4kRUA==,aO48gA==\r\na=control:track1\r\n"
  },
  "applicationProperties": {
    "dataVersion": "1.0",
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/videoAnalyzers/{ava-account-name}",
    "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/sources/rtspSource",
    "eventType": "Microsoft.VideoAnalyzer.Diagnostics.MediaSessionEstablished",
    "eventTime": "2021-05-06T17:33:09.554Z",
    "dataVersion": "1.0"
  },
  }
}
```

En este mensaje, tenga en cuenta estos detalles:

* El mensaje es un evento de diagnóstico. **MediaSessionEstablished** indica que el nodo de origen RTSP (valor de "subject") ha conectado con el simulador RTSP y ha comenzado a recibir una fuente en vivo (simulada).
* En `applicationProperties`, el elemento "subject" indica que el mensaje se generó desde el nodo de origen RTSP de la canalización en directo.
* En `applicationProperties`, el elemento "eventType" indica que este evento es un evento de diagnóstico.
* El valor de "eventTime" indica la hora a la que se produjo el evento.
* El cuerpo contiene datos sobre el evento de diagnóstico. En este caso, los datos incluyen detalles sobre el [protocolo de descripción de sesiones (SDP)](https://en.wikipedia.org/wiki/Session_Description_Protocol).

## <a name="line-crossing-events"></a>Eventos de cruce de líneas

El nodo procesador de extensiones HTTP envía los fotogramas 0, 15, 30, etc. al módulo avaextension y recibe los resultados de la inferencia. Luego, envía estos resultados y todos los fotogramas de vídeo al nodo de seguimiento de objetos. El seguimiento de objetos genera resultados de inferencia para todos los fotogramas (0, 1, 2,...), que el nodo de detección de cruce de líneas examina a continuación con las coordenadas de la línea especificadas en la topología. Cuando los objetos cruzan estas coordenadas, se desencadena un evento. El evento tiene este aspecto:
```
{
  "body": {
    "timestamp": 145865319410261,
    "inferences": [
      {
        "type": "event",
        "subtype": "lineCrossing",
        "inferenceId": "8f4f7b25d6654536908bcfe34374a15e",
        "relatedInferences": [
          "c9ea5decdd6a487089ded249c748cf5b"
        ],
        "event": {
          "name": "LineCrossing1",
          "properties": {
            "counterclockwiseTotal": "35",
            "direction": "counterclockwise",
            "total": "38",
            "clockwiseTotal": "3"
          }
        }
      }
    ]
  },
```
En este mensaje, tenga en cuenta estos detalles:
* El tipo es `event`, con el subtipo `lineCrossing`.
* El evento contiene el valor de `name`, como se especifica en la topología de la línea que se ha cruzado.
* Número `total` de cruces de línea en cualquier dirección.
* Número de cruces `clockwiseTotal`.
* Número de cruces `counterclockwiseTotal`.
* El elemento `direction` contiene la dirección de este evento.

## <a name="customize-for-your-own-environment"></a>Personalización para su propio entorno

Este tutorial funcionará con el vídeo de ejemplo proporcionado, para el que hemos calculado las coordenadas de línea correctas de la línea. Al examinar el archivo de topología, verá que el parámetro `lineCoordinates` contiene el siguiente valor: `[[0.5,0.1], [0.5,0.9]]`

¿Qué significa este valor? Cuando se quiere dibujar una línea en una imagen 2D, se necesitan dos puntos, A y B, y entre esos puntos tendrá una línea imaginaria. Cada punto tendrá sus propias coordenadas X e Y para determinar dónde se encuentra con respecto a la resolución de imagen completa. En este caso, el punto A es `[0.5,0.1]` y el punto B es `[0.5,0.9]`. Una representación visual de esa línea tiene el siguiente aspecto:
> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/track-objects-live-video/line-crossing-visual-example.png" alt-text="Ejemplo visual que muestra un cruce de línea en una imagen.":::

En esta imagen verá la línea entre el punto A y el punto B. Cualquier objeto que se mueva a través de la línea creará un evento con sus propiedades, como la dirección, como se ha mencionado anteriormente en este tutorial. Observe también los ejes X e Y en la esquina inferior izquierda. Esto es solo para ilustrar cómo normalizamos las coordenadas con los valores esperados para el nodo de detección de cruce de líneas. 

Este es un cálculo de ejemplo: supongamos que la resolución del vídeo es 1920 x 1080. 1920 y 1080 son el número de píxeles a lo largo del eje X e Y, respectivamente.
Cree una imagen a partir de un fotograma del vídeo que planea usar. Ahora, abra esa imagen en un programa de edición de imágenes (por ejemplo, MSPaint). Mueva el cursor a la ubicación donde desea especificar el punto A. En la esquina inferior izquierda verá las coordenadas X e Y de esa posición del cursor.
> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/track-objects-live-video/line-crossing-mspaint-coordinates.png" alt-text="Ejemplo visual para la detección de cruce de líneas mediante MSPaint.":::

Anote estos valores, repita lo mismo para el punto B y anote los mismos valores. Ahora debería tener los valores X e Y para el punto A y los valores X e Y para el punto B. Por ejemplo: punto A (X=1024, Y=96) y punto B (X=1024, Y=960). Estos valores no se parecen a los valores que irán al nodo de detección de cruce de líneas, ya que necesitamos números entre 0 y 1. Para calcularlo, aplique la fórmula siguiente:

`x coordinate / image resolution along x axis` (coordenada X / resolución de la imagen en el eje X), que en nuestro ejemplo es `1024/1920 = 0.5`. Ahora haga lo mismo para Y: `96/1080=0.1`. Estas son las coordenadas normalizadas para el punto A. Repita esta operación para el punto B. Con ello, tendrá la matriz `[[0.5,0.1], [0.5,0.9]]`, con valores entre 0 y 1, tal como se mostró anteriormente en este tutorial.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [prerequisites](./includes/common-includes/clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

* Pruebe a ejecutar otros vídeos mediante la canalización en directo.
