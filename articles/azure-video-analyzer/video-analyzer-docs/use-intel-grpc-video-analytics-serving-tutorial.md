---
title: Análisis de vídeos en directo mediante la extensión de inteligencia artificial de Edge Intel OpenVINO™ DL Streamer a través de gRPC con Azure Video Analyzer
description: En este tutorial se muestra cómo usar la extensión de inteligencia artificial de Edge Intel OpenVINO™ DL Streamer en Intel para analizar una fuente de vídeo en directo desde una cámara IP (simulada).
ms.topic: tutorial
ms.service: azure-video-analyzer
ms.date: 05/18/2021
ms.openlocfilehash: 449cd027396f92be0443d7cd4fe6dabc3eb449f6
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110384317"
---
# <a name="tutorial-analyze-live-video-with-intel-openvino-dl-streamer--edge-ai-extension"></a>Tutorial: análisis de vídeo en directo con la extensión de inteligencia artificial de Edge Intel OpenVINO™ DL Streamer 

En este tutorial se muestra cómo usar la extensión de inteligencia artificial de Edge OpenVINO™ DL Streamer de Intel para analizar una fuente de vídeo en directo desde una cámara IP (simulada). Verá la forma en que este servidor de inferencia le proporciona acceso a diferentes modelos de detección de objetos (una persona, un vehículo o una bicicleta), clasificación de objetos (atribuciones de los vehículos) y un modelo para hacer un seguimiento de objetos (persona, vehículo y bicicleta). La integración con el módulo gRPC permite enviar fotogramas de vídeo al servidor de inferencia de inteligencia artificial. Luego, los resultados se envían al centro de IoT Edge. Si este servicio de inferencia se ejecuta en el mismo nodo de proceso que Azure Video Analyzer, puede aprovechar el envío de datos de vídeo a través de la memoria compartida. Esto le permite ejecutar la inferencia a la velocidad de fotogramas de la fuente de vídeo en directo (es decir, 30 fotogramas/s.). 

En este tutorial se usa una máquina virtual de Azure como dispositivo IoT Edge simulado y se emplea una secuencia de vídeo en directo simulada. Se basa en el código de ejemplo escrito en C# y se basa en [Inicio rápido: Detección de movimiento y emisión de eventos](detect-motion-emit-events-quickstart.md).

> [!NOTE]
> En este tutorial se requiere el uso de una máquina x86-64 como dispositivo perimetral.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [prerequisites](./includes/common-includes/csharp-prerequisites.md)]

## <a name="set-up-azure-resources"></a>Configuración de los recursos de Azure

[![Implementación en Azure](https://aka.ms/deploytoazurebutton)](https://aka.ms/ava-click-to-deploy)

[!INCLUDE [resources](./includes/common-includes/azure-resources.md)]

## <a name="review-the-sample-video"></a>Revisión del vídeo de ejemplo

Cuando se configuran los recursos de Azure, se copia un vídeo corto de un aparcamiento en la máquina virtual Linux en Azure que se va a usar como dispositivo IoT Edge. En este tutorial se usa un archivo de vídeo para simular una secuencia en directo.

Abra una aplicación como [VLC Media Player](https://www.videolan.org/vlc/). Seleccione Ctrl + N y, después, pegue un vínculo al [vídeo](https://lvamedia.blob.core.windows.net/public/lots_015.mkv) para iniciar la reproducción. Se verá el metraje de los vehículos de un aparcamiento, la mayoría de ellos estacionados, y uno en movimiento.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4LUbN]

En este tutorial, usará Video Analyzer junto con la extensión de inteligencia artificial de Edge Intel OpenVINO™ DL Streamer para detectar o clasificar objetos como vehículos o para hacer un seguimiento de vehículos, personas o bicicletas. Los eventos de inferencia resultantes se publicarán en el centro de IoT Edge.

## <a name="overview"></a>Información general

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/use-intel-openvino-tutorial/grpc-vas-extension-with-vino.png" alt-text="Información general de la canalización de Azure Video Analyzer con el módulo de inteligencia artificial de Edge Intel DL Streamer":::

En este diagrama se muestra el flujo de las señales en este tutorial. Un [módulo perimetral](https://github.com/Azure/video-analyzer/tree/main/edge-modules/sources/rtspsim-live555) simula una cámara IP que hospeda un servidor de protocolo RTSP. Un nodo de [origen RTSP](pipeline.md#rtsp-source) extrae la fuente de vídeo de este servidor y envía fotogramas de vídeo al nodo del [procesador de extensiones gRPC](pipeline.md#grpc-extension-processor). 

Este nodo toma fotogramas de vídeo descodificados como entrada y los retransmite a un punto de conexión [gRPC](terminology.md#grpc) expuesto por un servidor de gRPC. El nodo admite la transferencia de datos mediante [memoria compartida](https://en.wikipedia.org/wiki/Shared_memory) o la incrustación directa del contenido en el cuerpo de los mensajes gRPC. Además, el nodo tiene un formateador de imagen integrado para escalar y codificar los fotogramas de vídeo antes de retransmitirlos al punto de conexión gRPC. El escalador tiene opciones para conservar, rellenar o ajustar la relación de aspecto de la imagen. El codificador de imágenes admite los formatos jpeg, png o bmp. Obtenga más información sobre el procesador [aquí](pipeline.md#grpc-extension-processor).

## <a name="about-intel-openvino-dl-streamer--edge-ai-extension-module"></a>Acerca del módulo Extensión de inteligencia artificial de Edge Intel OpenVINO™ DL Streamer


El módulo Extensión de inteligencia artificial perimetral Intel OpenVINO™ DL Streamer es un microservicio basado en el servicio de análisis de vídeo de Intel (servicio de VA) que sirve de canalizaciones de análisis de vídeo creadas con OpenVINO™ DL Streamer. Los desarrolladores pueden enviar fotogramas de vídeo descodificados al módulo de extensión de inteligencia artificial que realiza la detección, la clasificación o el seguimiento, y devuelve los resultados. El módulo de extensión de inteligencia artificial expone las API de gRPC que son compatibles con plataformas de análisis de vídeo como Azure Video Analyzer de Microsoft.

Con el fin de crear soluciones de análisis de vídeo complejas y de alto rendimiento, el módulo Azure Video Analyzer se debe emparejar con un potente motor de inferencia que pueda aprovechar la escala en el perímetro. En este tutorial, se envían solicitudes de inferencia a la extensión de inteligencia artificial de Edge Intel OpenVINO™ DL Streamer, un módulo de Edge que se ha diseñado para funcionar con Azure Video Analyzer.

En la versión inicial de este servidor de inferencia, se tiene acceso a los siguientes [modelos](https://aka.ms/intel-dlstreamer-docs):

- object_detection for person_vehicle_bike_detection ![detección de objetos para vehículo](./media/use-intel-openvino-tutorial/object-detection.png)

- object_classification for vehicle_attributes_recognition ![clasificación de objetos para vehículo](./media/use-intel-openvino-tutorial/object-classification.png)

- object_tracking for person_vehicle_bike_tracking ![seguimiento de objetos para vehículo y persona](./media/use-intel-openvino-tutorial/object-tracking.png)

Usa la detección de objetos cargados previamente, la clasificación de objetos y las canalizaciones de seguimiento de objetos para empezar a trabajar rápidamente. Además, incluye los modelos [person-vehicle-bike-detection-crossroad-0078](https://github.com/openvinotoolkit/open_model_zoo/blob/master/models/intel/person-vehicle-bike-detection-crossroad-0078/description/person-vehicle-bike-detection-crossroad-0078.md) y [vehicle-attributes-recognition-barrier-0039](https://github.com/openvinotoolkit/open_model_zoo/blob/master/models/intel/vehicle-attributes-recognition-barrier-0039/description/vehicle-attributes-recognition-barrier-0039.md) precargados.

> [!NOTE]
> Al descargar y usar el módulo de Edge: Extensión OpenVINO™ DL Streamer – AI de Intel, y el software incluido, acepte los términos y condiciones del [contrato de licencia](https://www.intel.com/content/www/us/en/legal/terms-of-use.html).
> Intel se compromete a respetar los derechos humanos y a evitar ser cómplice de la vulneración de tales derechos. Consulte los [principios de los derechos humanos globales de Intel](https://www.intel.com/content/www/us/en/policy/policy-human-rights.html). Los productos y el software de Intel están concebidos únicamente para usarse en aplicaciones que no infrinjan los derechos humanos reconocidos internacionalmente, ni contribuyan a tal infracción.

En su caso concreto, puede usar la flexibilidad de las distintas canalizaciones. Para ello solo tiene que cambiar las variables de entorno de la canalización en la plantilla de implementación. Esto le permite cambiar rápidamente el modelo de canalización y, cuando se combina con Azure Video Analyzer, en segundos se cambia la canalización de medios y el modelo de inferencia.

En este tutorial, aprenderá lo siguiente:

1. Configure un entorno de desarrollo.
1. Implementar los módulos perimetrales necesarios.
1. Crear e implementar la canalización en directo.
1. Interpretará los resultados.
1. Limpieza de recursos.

## <a name="set-up-your-development-environment"></a>Configurado su entorno de desarrollo
[!INCLUDE [setup development environment](./includes/set-up-dev-environment/csharp/csharp-set-up-dev-env.md)]

## <a name="deploy-the-required-modules"></a>Implementación de los módulos necesarios

1. En Visual Studio Code, haga clic con el botón derecho en el archivo *src/edge/deployment.openvino.grpc.template.json* y seleccione **Generate IoT Edge Deployment Manifest** (Generar manifiesto de implementación de IoT Edge).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/analyze-live-video-use-your-model-http/generate-deployment-manifest.png" alt-text="Generar un manifiesto de implementación de IoT Edge":::
1. El archivo de manifiesto *deployment.openvino.grpc.amd64.json* se crea en la carpeta *src/edge/config*.
1. Haga clic con el botón derecho en *src/edge/config/deployment.openvino.grpc.amd64.json* y seleccione **Crear una implementación para un dispositivo individual**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/analyze-live-video-use-your-model-http/deployment-single-device.png" alt-text= "Create Deployment for a Single Device":::
1. Cuando se le pida que seleccione un dispositivo IoT Hub, elija **avasample-iot-edge-device**.
1. Tras aproximadamente 30 segundos, en la esquina inferior izquierda de la ventana, actualice Azure IoT Hub. El dispositivo perimetral ahora muestra los siguientes módulos implementados:

    * El módulo perimetral de Video Analyzer, llamado **avaedge**.
    * El módulo **rtspsim**, que simula un servidor RTSP y actúa como el origen de una fuente de vídeo en directo. 
    * El módulo **avaextension**, que es el módulo Intel OpenVINO DL Streamer con el modelo de detección, clasificación y seguimiento de objetos que aplica Computer Vision a las imágenes y devuelve varias clases de tipos de objeto.

        > [!div class="mx-imgBorder"]
        > :::image type="content" source="./media/vscode-common-screenshots/avaextension.png" alt-text= "OpenVINO object detection model":::

> [!NOTE]
> También se ha incluido una plantilla *deployment.openvino.grpc.xpu.template.json* que habilita el soporte de CPU, VPU y GPU, si está disponible en el dispositivo, para el módulo de extensión de inteligencia artificial de Edge Intel OpenVINO DL Streamer. Estas plantillas apuntan a la imagen de Docker Hub de Intel.

### <a name="prepare-to-monitor-events"></a>Preparación para supervisar eventos

Haga clic con el botón derecho en el dispositivo de Azure Video Analyzer y seleccione **Start Monitoring Built-in Event Endpoint** (Iniciar supervisión del punto de conexión de eventos integrado). Este paso es necesario para supervisar los eventos de IoT Hub en la ventana **SALIDA** de Visual Studio Code.

![Iniciar supervisión](./media/quickstarts/start-monitoring-iot-hub-events.png) 

### <a name="run-the-sample-program-to-detect-vehicles-persons-or-bike"></a>Ejecución del programa de ejemplo para detectar personas, vehículos o bicicletas
Si abre la [topología de canalización](https://raw.githubusercontent.com/Azure/azure-video-analyzer/main/pipelines/live/topologies/grpcExtensionOpenVINO/topology.json) para este tutorial en un explorador, verá que el valor de `grpcExtensionAddress` se ha establecido en `tcp://avaExtension:5001`, en comparación con el tutorial *httpExtensionOpenVINO*, no necesita cambiar la dirección URL al servidor de gRPC. En su lugar, indique al módulo que ejecute una canalización específica según `extensionConfiguration` en el archivo de operaciones. Cuando no se proporciona, de manera predeterminada, su valor es "object_detection" para "person_vehicle_bike_detection". Puede experimentar con otras canalizaciones admitidas.

1. Edite el archivo *operations.json*:
    * Cambie el vínculo a la topología de la canalización en directo:

        `"topologyUrl" : "https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/grpcExtensionOpenVINO/topology.json"`

    * En `pipelineTopologySet`, edite el nombre de la topología de la canalización en directo para que coincida con el valor del vínculo anterior:

      `"topologyName" : "InferencingWithOpenVINOgRPC"`

    * En `pipelineTopologyDelete`, edite el nombre:

      `"name": "InferencingWithOpenVINOgRPC"`
    
    * En `rtspUrl`, edite la dirección URL para: `"value": "rtsp://rtspsim:554/media/lots_015.mkv"`

Agregue el elemento extensionConfiguration siguiente después de los parámetros rtsp:

```
{
    "name": "extensionConfiguration",
    "value": "{\"pipeline\":{\"name\":\"object_detection\",\"version\":\"person_vehicle_bike_detection\"}}"
}
```


`operations.json` ahora debería presentar un aspecto similar a este:
```
{
  "apiVersion": "1.0",
  "operations": [
      {
          "opName": "pipelineTopologyList",
          "opParams": {}
      },
      {
          "opName": "WaitForInput",
          "opParams": {
              "message": "Press Enter to continue."
          }
      },
      {
          "opName": "livePipelineList",
          "opParams": {}
      },
      {
          "opName": "WaitForInput",
          "opParams": {
              "message": "Press Enter to continue. This is start the activating steps."
          }
      },
      {
          "opName": "pipelineTopologySet",
          "opParams": {
              "pipelineTopologyUrl": "https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/grpcExtensionOpenVINO/topology.json"
          }
      },
      {
          "opName": "livePipelineSet",
          "opParams": {
              "name": "Sample-Pipeline-1",
              "properties": {
                  "topologyName": "InferencingWithOpenVINOgRPC",
                  "description": "Sample pipeline description",
                  "parameters": [
                      {
                          "name": "rtspUrl",
                          "value": "rtsp://rtspsim:554/media/lots_015.mkv"
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
                        "name": "extensionConfiguration",
                        "value": "{\"pipeline\":{\"name\":\"object_detection\",\"version\":\"person_vehicle_bike_detection\"}}"
                      }
                  ]
              }
          }
      },
      {
          "opName": "livePipelineActivate",
          "opParams": {
              "name": "Sample-Pipeline-1"
          }
      },
      {
          "opName": "WaitForInput",
          "opParams": {
              "message": "Press Enter to continue. This is start the deactivating steps."
          }
      },
      {
          "opName": "livePipelineDeactivate",
          "opParams": {
              "name": "Sample-Pipeline-1"
          }
      },
      {
          "opName": "livePipelineDelete",
          "opParams": {
              "name": "Sample-Pipeline-1"
          }
      },
      {
          "opName": "livePipelineList",
          "opParams": {}
      },
      {
          "opName": "pipelineTopologyDelete",
          "opParams": {
              "name": "InferencingWithOpenVINOgRPC"
          }
      },
      {
          "opName": "pipelineTopologyList",
          "opParams": {}
      }
  ]
}
```

El uso de `extensionConfiguration` le permite pasar una cadena de datos al módulo de extensión de inteligencia artificial de Edge Intel DL Streamer. Con los datos correctos, puede indicar al módulo de inteligencia artificial de Edge que use un modelo específico para cada canalización. Por ejemplo, puede reemplazar el modelo "object_detection" usado anteriormente por otros modelos admitidos, como los de clasificación o seguimiento. Para ello, use los ejemplos siguientes:

Clasificación:
```
{
    "name": "extensionConfiguration",
    "value": "{\"pipeline\":{\"name\":\"object_classification\",\"version\":\"vehicle_attributes_recognition\"}}"
}
```

Seguimiento:
```
{
    "name": "extensionConfiguration",
    "value": "{\"pipeline\":{\"name\":\"object_tracking\",\"version\":\"person_vehicle_bike_tracking\"}}"
}
```

1. En Visual Studio Code, abra la pestaña **Extensiones** (o presione Ctrl + Mayús + X) y busque Azure IoT Hub.
1. Haga clic con el botón derecho y seleccione la **Configuración de la extensión**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/extensions-tab.png" alt-text="Configuración de la extensión":::
1. Busque y habilite "Show Verbose Message" (Mostrar mensaje detallado).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/show-verbose-message.png" alt-text="Show Verbose Message"::: (Mostrar mensaje detallado)
1. Para iniciar una sesión de depuración, seleccione la tecla F5. Verá mensajes impresos en la ventana **TERMINAL**.
1. El código de *operations.json* comienza con llamadas a los métodos directos `pipelineTopologyList` y `livePipelineList`. Si ha limpiado los recursos tras haber completado los tutoriales e inicios rápidos anteriores, este proceso devolverá listas vacías y, después, se pausará. Para continuar, seleccione la tecla Entrar.

    La ventana **TERMINAL** muestra el siguiente conjunto de llamadas al método directo:

     * Una llamada a `pipelineTopologySet` que utiliza la instancia anterior de `topologyUrl`.
     * Una llamada a `livePipelineSet` que usa el cuerpo siguiente:

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
                 "value": "rtsp://rtspsim:554/media/lots_015.mkv"
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
                 "value": "tcp://avaextension:5001"
               },
               {
                 "name": "extensionConfiguration",
                 "value": "{\"pipeline\":{\"name\":\"object_detection\",\"version\":\"person_vehicle_bike_detection\"}}"
               }
             ]
           }
         }
         ```

     * Una llamada a `livePipelineActivate` que activa la canalización y el flujo de vídeo.
1. La salida de la ventana **TERMINAL** se pone en pausa tras el mensaje `Press Enter to continue`. No seleccione Entrar todavía. Desplácese hacia arriba para ver las cargas de la respuesta JSON para los métodos directos que ha invocado.
1. Cambie a la ventana **SALIDA** de Visual Studio Code. Verá los mensajes que el módulo Video Analyzer está enviando al centro de IoT. En la siguiente sección de este tutorial se analizan estos mensajes.
1. La canalización en directo continúa ejecutándose y se imprimen los resultados. El simulador RTSP sigue recorriendo el vídeo de origen. Para detener la canalización en directo, vuelva a la ventana **TERMINAL** y seleccione Entrar. 

    La siguiente serie de llamadas limpia los recursos:
      * Una llamada a `livePipelineDeactivate` desactiva la canalización.
      * Una llamada a `livePipelineDelete` elimina la canalización.
      * Una llamada a `pipelineTopologyDelete` elimina la topología.
      * Una llamada final a `pipelineTopologyList` muestra que la lista está vacía.

## <a name="interpret-results"></a>Interpretación de los resultados

Cuando se ejecuta la canalización en directo, los resultados del nodo procesador de extensiones gRPC recorren el nodo receptor de mensajes de IoT Hub hasta llegar al centro de IoT. Los mensajes que aparecen en la ventana **SALIDA** contienen una sección `body`. Para más información, consulte [Creación y lectura de mensajes de IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).

En los mensajes siguientes, el módulo Video Analyzer define el contenido del cuerpo. 

### <a name="mediasessionestablished-event"></a>Evento MediaSessionEstablished

Cuando se activa una canalización en directo, el nodo de origen RTSP intenta conectarse al servidor RTSP que se ejecuta en el contenedor rtspsim-live555. Si la conexión se realiza correctamente, se imprime el evento siguiente.

```
[IoTHubMonitor] [10:51:34 AM] Message received from [avasample-iot-edge-device/avaedge]:
{
  "sdp&quot;: &quot;SDP:\nv=0\r\no=- 1620204694595500 1 IN IP4 xxx.xxx.xxx.xxx\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/camera-300s.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.08.19\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/camera-300s.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets=Z00AKeKQCgC3YC3AQEBpB4kRUA==,aO48gA==\r\na=control:track1\r\n"
}
```

En la salida anterior: 

* El mensaje es un evento de diagnóstico, `MediaSessionEstablished`. Indica que el nodo de origen RTSP (el sujeto) ha conectado con el simulador RTSP y ha comenzado a recibir una fuente en directo (simulada).
* La sección `sdp` contiene datos sobre el evento de diagnóstico. En este caso, los datos incluyen detalles sobre el [protocolo de descripción de sesiones (SDP)](https://en.wikipedia.org/wiki/Session_Description_Protocol).

### <a name="inference-event"></a>Evento de inferencia

El nodo del procesador de la extensión gRPC recibe los resultados de la inferencia de Extensión de inteligencia artificial de Edge Intel OpenVINO™ DL Streamer. Luego, emite los resultados a través del nodo receptor de mensajes de IoT Hub como eventos de inferencia. 

En estos eventos, el tipo se establece en `entity` para indicar que es una entidad, como un coche o un camión. 

En el ejemplo siguiente, se detectó un vehículo, con un valor de confianza superior a 0,9.

```
[IoTHubMonitor] [3:10:23 PM] Message received from [avasample-iot-edge-device/avaedge]:
{
  "timestamp": 145819820073974,
  "inferences": [
    {
      "type": "entity",
      "subtype": "vehicleDetection",
      "entity": {
        "tag": {
          "value": "vehicle",
          "confidence": 0.9147264
        },
        "box": {
          "l": 0.6853116,
          "t": 0.5035262,
          "w": 0.04322505,
          "h": 0.03426218
        }
      }
    }
```

En los mensajes, tenga en cuenta los siguientes detalles:

* La sección `body` contiene datos sobre el evento de análisis. En este caso, el evento es un evento de inferencia, por lo que el cuerpo contiene los datos `inferences`.
* La sección `inferences` indica que el valor de `type` es `entity`. En esta sección se incluyen datos adicionales sobre la entidad.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si su intención es probar otros inicios rápidos o tutoriales, conserve los recursos creados. En caso contrario, vaya a Azure Portal, luego a los grupos de recursos, seleccione el grupo de recursos donde ejecutó este tutorial y elimine todos los recursos.

## <a name="next-steps"></a>Pasos siguientes

Revise los desafíos adicionales para los usuarios avanzados:

* Use una [cámara IP](https://en.wikipedia.org/wiki/IP_camera) que sea compatible con RTSP, en lugar de utilizar el simulador RTSP. Puede buscar cámaras IP compatibles con RTSP en la página de [productos compatibles con ONVIF](https://www.onvif.org/conformant-products/). Busque dispositivos que se ajusten a los perfiles G, S o T.
* Use un dispositivo Intel x64 con Linux, en lugar de una máquina virtual Linux de Azure. El dispositivo debe estar en la misma red que la cámara IP. Puede seguir las instrucciones que aparecen en [Instalación del entorno de ejecución de Azure IoT Edge en Linux](../../iot-edge/how-to-install-iot-edge.md). Luego, registre el dispositivo en Azure IoT Hub, para lo que debe seguir las instrucciones que se encuentran en [Implementación del primer módulo IoT Edge en un dispositivo virtual Linux](../../iot-edge/quickstart-linux.md).
