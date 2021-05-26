---
title: Análisis de vídeo en directo mediante la extensión OpenVINO™ Model Server – AI de Intel
description: En este tutorial, usará un servidor de modelos de IA con modelos preentrenados proporcionado por Intel para analizar la fuente de vídeo en directo desde una cámara IP (simulada).
ms.service: azure-video-analyzer
ms.topic: tutorial
ms.date: 05/18/2021
titleSuffix: Azure
ms.openlocfilehash: d9c7beeb75d8a8785f58b1d0b9b3a93dfc6e53f1
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110384293"
---
# <a name="tutorial-analyze-live-video-using-openvino-model-server--ai-extension-from-intel"></a>Tutorial: Análisis de vídeo en directo mediante la extensión OpenVINO™ Model Server – AI de Intel 

En este tutorial se muestra cómo usar la [extensión OpenVINO™ Model Server – AI de Intel](https://aka.ms/ava-intel-ovms) para analizar una fuente de vídeo en directo desde una cámara IP (simulada). Verá cómo este servidor de inferencia le proporciona acceso a modelos para detectar objetos (una persona, un vehículo o una bicicleta), y a un modelo para clasificar vehículos. Un subconjunto de los fotogramas de la fuente de vídeo en directo se envía a este servidor de inferencia y los resultados se envían al centro de IoT Edge.

En este tutorial se usa una máquina virtual de Azure como dispositivo IoT Edge y se emplea una secuencia de vídeo en directo simulada. Se basa en un ejemplo de código escrito en C#.

> [!NOTE]
> En este tutorial se requiere el uso de una máquina x86-64 como dispositivo perimetral.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [prerequisites](./includes/common-includes/csharp-prerequisites.md)]

## <a name="set-up-azure-resources"></a>Configuración de los recursos de Azure

[![Implementación en Azure](https://aka.ms/deploytoazurebutton)](https://aka.ms/ava-click-to-deploy)

[!INCLUDE [resources](./includes/common-includes/azure-resources.md)]

## <a name="about-openvino-model-server--ai-extension-from-intel"></a>Acerca de la extensión OpenVINO™ Model Server – AI de Intel

La distribución del [kit de herramientas de OpenVINO™](https://software.intel.com/content/www/us/en/develop/tools/openvino-toolkit.html) de Intel® (la inferencia visual abierta y la optimización de la red neuronal) es un kit de software gratuito que ayuda a los desarrolladores y a los científicos de datos a acelerar las cargas de trabajo de visión artificial, a simplificar la inferencia y las implementaciones de aprendizaje profundo y a permitir la ejecución sencilla y heterogénea en las plataformas de Intel® del perímetro a la nube. Incluye Deep Learning Deployment Toolkit de Intel® con optimizador de modelos y motor de inferencia, así como el repositorio [Open Model Zoo](https://github.com/openvinotoolkit/open_model_zoo) que incluye más de 40 modelos preentrenados optimizados.

Con el fin de crear soluciones de análisis de vídeo complejas y de alto rendimiento, el módulo Video Analyzer se debe emparejar con un potente motor de inferencia que pueda aprovechar la escala en el perímetro. En este tutorial, se envían solicitudes de inferencia a la [extensión OpenVINO™ Model Server – AI de Intel](https://aka.ms/ava-intel-ovms), un módulo perimetral diseñado para funcionar con Video Analyzer. Este módulo de servidor de inferencia contiene la solución OpenVINO™ Model Server (OVMS), un servidor de inferencia que incluye el kit de herramientas de OpenVINO™; este está perfectamente optimizado para cargas de trabajo de Computer Vision y se ha desarrollado para las arquitecturas de Intel. Se ha agregado una extensión a OVMS para facilitar el intercambio de fotogramas de vídeo y resultados de inferencia entre el servidor de inferencia y el módulo de Video Analyzer, lo que le permite ejecutar cualquier modelo compatible con el kit de herramientas de OpenVINO™ (puede personalizar el módulo de servidor de inferencia al modificar [este código](https://github.com/openvinotoolkit/model_server/tree/master/extras/ams_wrapper)). Además, se puede seleccionar entre una amplia variedad de mecanismos de aceleración proporcionados por hardware de Intel®. Entre ellos figuran las CPU (Atom, Core, Xeon), los FPGA, y las VPU.

En la versión inicial de este servidor de inferencia, se tiene acceso a los siguientes [modelos](https://github.com/openvinotoolkit/model_server/tree/master/extras/ams_models):

- Detección de vehículos (dirección URL de inferencia: http://{module-name}: 4000/vehicleDetection)
- Detección de personas/vehículos/bicicletas (dirección URL de inferencia: http://{module-name}: 4000/personVehicleBikeDetection)
- Clasificación de vehículos (dirección URL de inferencia: http://{module-name}: 4000/vehicleClassification)
- Detección de caras (dirección URL de inferencia: http://{module-name}: 4000/faceDetection)

> [!NOTE]
> Al descargar y usar el módulo de Edge: la extensión OpenVINO™ Model Server – AI de Intel, y el software incluido, acepte los términos y condiciones del [contrato de licencia](https://www.intel.com/content/www/us/en/legal/terms-of-use.html).
> Intel se compromete a respetar los derechos humanos y a evitar ser cómplice de la vulneración de tales derechos. Consulte los [principios de los derechos humanos globales de Intel](https://www.intel.com/content/www/us/en/policy/policy-human-rights.html). Los productos y el software de Intel están concebidos únicamente para usarse en aplicaciones que no infrinjan los derechos humanos reconocidos internacionalmente, ni contribuyan a tal infracción.

## <a name="overview"></a>Información general

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/use-intel-openvino-tutorial/http-extension-with-vino.png" alt-text="Información general":::

En este diagrama se muestra cómo fluyen las señales en este inicio rápido. Un [módulo perimetral](https://github.com/Azure/video-analyzer/tree/main/edge-modules/sources/rtspsim-live555) simula una cámara IP que hospeda un servidor de protocolo RTSP. Un nodo de [origen RTSP](pipeline.md#rtsp-source) extrae la fuente de vídeo de este servidor y envía fotogramas de vídeo al nodo del [procesador de extensiones HTTP](pipeline-extension.md#http-extension-processor). 

El nodo del procesador de extensión HTTP desempeña el rol de un proxy. Selecciona un subconjunto de los fotogramas de vídeo entrantes y los convierte en imágenes. Luego, la imagen se retransmite a través de REST a otro módulo perimetral que ejecuta modelos de IA detrás de un punto de conexión HTTP. En este ejemplo, ese módulo perimetral es la extensión OpenVINO™ Model Server – AI de Intel. El nodo del procesador de extensión HTTP recopila los resultados de la detección y publica los eventos en el nodo del [receptor de mensajes de IoT Hub](pipeline.md#iot-hub-message-sink), quien posteriormente los envía a [IoT Edge Hub](../../iot-fundamentals/iot-glossary.md#iot-edge-hub).

En este tutorial, aprenderá lo siguiente:

1. Configure un entorno de desarrollo.
1. Implementar los módulos perimetrales necesarios.
1. Crear e implementar la canalización en directo.
1. Interpretará los resultados.
1. Limpieza de recursos.

## <a name="set-up-your-development-environment"></a>Configurado su entorno de desarrollo
[!INCLUDE [setup development environment](./includes/set-up-dev-environment/csharp/csharp-set-up-dev-env.md)]

### <a name="review-the-sample-video"></a>Revisión del vídeo de ejemplo

Cuando se configuran los recursos de Azure, se copia un [vídeo corto de un aparcamiento](https://lvamedia.blob.core.windows.net/public/lots_015.mkv) en la máquina virtual Linux en Azure que se va a usar como dispositivo IoT Edge. En este inicio rápido se usa el archivo de vídeo para simular una secuencia en directo.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4LUbN]

## <a name="deploy-the-required-modules"></a>Implementación de los módulos necesarios

Como parte de los requisitos previos, ha descargado el código de ejemplo en una carpeta. Siga estos pasos para implementar los módulos necesarios.

1. En Visual Studio Code, haga clic con el botón derecho en el archivo *src/edge/deployment.openvino.template.json* y seleccione **Generación de un manifiesto de implementación de IoT Edge**.   
1. El archivo de manifiesto *deployment.openvino.amd64.json* se crea en la carpeta *src/edge/config*.
1. Haga clic con el botón derecho en *src/edge/config/deployment.openvino.amd64.json* y seleccione **Crear una implementación para un dispositivo individual**.
1. Cuando se le pida que seleccione un dispositivo IoT Hub, elija **avasample-iot-edge-device**.
1. Tras aproximadamente 30 segundos, en la esquina inferior izquierda de la ventana, actualice Azure IoT Hub. El dispositivo perimetral ahora muestra los siguientes módulos implementados:
    * El módulo perimetral de Video Analyzer, llamado **avaedge**.
    * El módulo **rtspsim**, que simula un servidor RTSP y actúa como el origen de una fuente de vídeo en directo. 
    * El módulo **openvino**, que es el módulo de la extensión OpenVINO™ Model Server – AI de Intel.
 
## <a name="create-and-activate-the-live-pipeline"></a>Creación y activación de la canalización en vivo

### <a name="edit-the-sample-code"></a>Edición del código de ejemplo
1. En Visual Studio, vaya a *src/cloud-to-device-console-app* y edite el archivo *operations.json*:
    * Cambie el vínculo a la topología de la canalización en directo:

        `"pipelineTopologyUrl" : "https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/httpExtensionOpenVINO/topology.json"`

    * En `livePipelineSet`, edite el nombre de la topología de la canalización en directo para que coincida con el valor del vínculo anterior:

      `"topologyName" : "InferencingWithOpenVINO"`

    * En `pipelineTopologyDelete`, edite el nombre:

      `"name": "InferencingWithOpenVINO"`

### <a name="run-the-sample-program-to-detect-vehicles"></a>Ejecución del programa de ejemplo para detectar vehículos
Si abre la topología de la canalización (`pipelineTopologyUrl`) de este tutorial en un explorador, verá que el valor de `inferencingUrl` se ha establecido en `http://openvino:4000/vehicleDetection`, lo que significa que el servidor de inferencia tratará de detectar vehículos en el vídeo en directo.

1. Para iniciar una sesión de depuración, seleccione la tecla F5. Verá mensajes impresos en la ventana **TERMINAL**.
1. El código de *operations.json* comienza con llamadas a los métodos directos `pipelineTopologyList` y `livePipelineList`. Si ha limpiado los recursos tras haber completado los inicios rápidos anteriores, este proceso devolverá listas vacías. La ventana **TERMINAL** muestra el siguiente conjunto de llamadas al método directo:

     * Una llamada a `pipelineTopologySet` que utiliza la instancia anterior de `pipelineTopologyUrl`.
     * Una llamada a `livePipelineSet` que usa el cuerpo siguiente:

         ```json
            {
              "@apiVersion": "1.0",
              "name": "Sample-Pipeline-1",
              "properties": {
                "topologyName": "InferencingWithOpenVINO",
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

     * Una llamada a `livePipelineActivate` que inicia la canalización y el flujo de vídeo.
1. La salida de la ventana **TERMINAL** se pone en pausa tras el mensaje `Press Enter to continue`. No seleccione Entrar todavía. Desplácese hacia arriba para ver las cargas de la respuesta JSON para los métodos directos que ha invocado.
1. Cambie a la ventana **SALIDA** de Visual Studio Code. Verá los mensajes que el módulo de Video Analyzer está enviando al centro de IoT. En la siguiente sección de este inicio rápido se analizan estos mensajes.
1. La canalización en directo continúa ejecutándose y se imprimen los resultados. El simulador RTSP sigue recorriendo el vídeo de origen. Para detener la canalización en directo, vuelva a la ventana **TERMINAL** y seleccione Entrar. 

    La siguiente serie de llamadas limpia los recursos:
      * Una llamada a `livePipelineDeactivate` desactiva la canalización.
      * Una llamada a `livePipelineDelete` elimina la canalización.
      * Una llamada a `pipelineTopologyDelete` elimina la topología.
      * Una llamada final a `pipelineTopologyList` muestra que la lista está vacía.

## <a name="interpret-results"></a>Interpretación de los resultados

Cuando se ejecuta la canalización en directo, los resultados del nodo procesador de extensiones HTTP recorren el nodo receptor de mensajes de IoT Hub hasta llegar al centro de IoT. Los mensajes que aparecen en la ventana **SALIDA** contienen una sección `body`. Para más información, consulte [Creación y lectura de mensajes de IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).

En los mensajes siguientes, el módulo de Video Analyzer define las propiedades de la aplicación y el contenido del cuerpo. 

### <a name="mediasessionestablished-event"></a>Evento MediaSessionEstablished

Cuando se activa una canalización en directo, el nodo de origen RTSP intenta conectarse al servidor RTSP que se ejecuta en el contenedor rtspsim-live555. Si la conexión se realiza correctamente, verá un evento en la ventana **SALIDA** con lo siguiente.

```
[IoTHubMonitor] [9:42:18 AM] Message received from [ava-sample-device/avaadge]:
{
  "body": {
    "sdp&quot;: &quot;SDP:\nv=0\r\no=- 1586450538111534 1 IN IP4 XXX.XX.XX.XX\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/lots_015.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.03.06\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/lots_015.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets=XXXXXXXXXXXXXXXXXXXXXX\r\na=control:track1\r\n"
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

* El mensaje es un evento de diagnóstico, **MediaSessionEstablished**. Indica que el nodo de origen RTSP (el sujeto) ha conectado con el simulador RTSP y ha comenzado a recibir una fuente en directo (simulada).
* La sección `sdp` contiene datos sobre el evento de diagnóstico. En este caso, los datos incluyen detalles sobre el [protocolo de descripción de sesiones (SDP)](https://en.wikipedia.org/wiki/Session_Description_Protocol).

### <a name="inference-event"></a>Evento de inferencia

El nodo procesador de la extensión HTTP recibe resultados de inferencia del módulo de la extensión OpenVINO™ Model Server – AI. Luego, emite los resultados a través del nodo del receptor de mensajes de IoT Hub como eventos de inferencia. 

En estos eventos, el tipo se establece en `entity` para indicar que es una entidad, como un coche o un camión. En el ejemplo siguiente de `body` de este tipo de evento, se detectó un vehículo, con valores de confianza superiores a 0,9.

```json
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

## <a name="run-the-sample-program-to-detect-persons-vehicles-or-bikes"></a>Ejecución del programa de ejemplo para detectar personas, vehículos o bicicletas
Para usar un modelo diferente, tendrá que modificar la topología de la canalización y también el archivo `operations.json`.

Copie la topología de canalización (dirección URL usada en `pipelineTopologyUrl`) en un archivo local, por ejemplo `C:\TEMP\topology.json`. Abra esa copia y cambie el valor de `inferencingUrl` por `http://openvino:4000/personVehicleBikeDetection`.

En Visual Studio Code, vaya a la carpeta *src/cloud-to-device-console-app* y abra el archivo `operations.json`. Cambie la línea con `pipelineTopologyUrl` por:

```
      "pipelineTopologyFile" : "C:\\TEMP\\topology.json" 
```
Ahora puede repetir los pasos anteriores para volver a ejecutar el programa de ejemplo con la nueva topología. Los resultados de la inferencia serán similares (en el esquema) a los del modelo de detección de vehículos, solo que `subtype` está establecido en `personVehicleBikeDetection`.

## <a name="run-the-sample-program-to-classify-vehicles"></a>Ejecución del programa de ejemplo para clasificar vehículos
En Visual Studio Code, abra la copia local de `topology.json` del paso anterior y cambie el valor de `inferencingUrl` por `http://openvino:4000/vehicleClassification`. Si ha ejecutado el ejemplo anterior para detectar personas, vehículos o bicicletas, no es necesario volver a modificar el archivo `operations.json`.

Ahora puede repetir los pasos anteriores para volver a ejecutar el programa de ejemplo con la nueva topología. A continuación, se muestra un resultado de clasificación de ejemplo.

```json
{
  "timestamp": 145819896480527,
  "inferences": [
    {
      "type": "classification",
      "subtype": "color",
      "classification": {
        "tag": {
          "value": "gray",
          "confidence": 0.683415
        }
      }
    },
    {
      "type": "classification",
      "subtype": "type",
      "classification": {
        "tag": {
          "value": "truck",
          "confidence": 0.9978394
        }
      }
    }
  ]
}

```

## <a name="run-the-sample-program-to-detect-faces"></a>Ejecución del programa de ejemplo para detectar caras
En Visual Studio Code, abra la copia local de `topology.json` del paso anterior y cambie el valor de `inferencingUrl` por `http://openvino:4000/faceDetection`. Si ha ejecutado el ejemplo anterior para detectar personas, vehículos o bicicletas, no es necesario volver a modificar el archivo `operations.json`.

Ahora puede repetir los pasos anteriores para volver a ejecutar el programa de ejemplo con la nueva topología. A continuación, se muestra un resultado de detección de ejemplo (Nota: El vídeo del aparcamiento usado anteriormente no contiene ninguna cara detectable, por lo que se debe usar otro vídeo para probar este modelo).

```json
{
  "body": {
    "inferences": [
      {
        "type": "entity",
        "subtype": "faceDetection",
        "entity": {
          "tag": {
            "value": "face",
            "confidence": 0.9997053742408752
          },
          "box": {
            "l": 0.2559490501880646,
            "t": 0.03403960168361664,
            "w": 0.17685115337371826,
            "h": 0.45835764706134796
          }
        }
      }
    ]
  }
```
## <a name="clean-up-resources"></a>Limpieza de recursos

Si su intención es probar otros inicios rápidos o tutoriales, conserve los recursos creados. En caso contrario, vaya a Azure Portal, luego a los grupos de recursos, seleccione el grupo de recursos donde ejecutó este tutorial y elimine todos los recursos.

## <a name="next-steps"></a>Pasos siguientes

Revise los desafíos adicionales para los usuarios avanzados:

* Use una [cámara IP](https://en.wikipedia.org/wiki/IP_camera) que sea compatible con RTSP, en lugar de utilizar el simulador RTSP. Puede buscar cámaras IP compatibles con RTSP en la página de [productos compatibles con ONVIF](https://www.onvif.org/conformant-products/). Busque dispositivos que se ajusten a los perfiles G, S o T.
* Use un dispositivo local x64 con Linux, en lugar de una máquina virtual Linux de Azure. El dispositivo debe estar en la misma red que la cámara IP. Puede seguir las instrucciones que aparecen en [Instalación del entorno de ejecución de Azure IoT Edge en Linux](../../iot-edge/how-to-install-iot-edge.md). Luego, registre el dispositivo en Azure IoT Hub, para lo que debe seguir las instrucciones que se encuentran en [Implementación del primer módulo IoT Edge en un dispositivo virtual Linux](../../iot-edge/quickstart-linux.md).
