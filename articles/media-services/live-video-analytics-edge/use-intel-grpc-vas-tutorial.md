---
title: Análisis de vídeo en directo mediante Live Video Analytics con la extensión de inteligencia artificial de Edge con gRPC OpenVINO™ DL Streamer de Intel
description: En este tutorial se muestra cómo usar Live Video Analytics con la extensión de inteligencia artificial de Edge OpenVINO™ DL Streamer de Intel para analizar una fuente de vídeo en directo desde una cámara IP (simulada).
ms.topic: tutorial
ms.date: 02/04/2021
ms.service: media-services
ms.author: faneerde
author: fvneerden
ms.openlocfilehash: d6d842a59a184470720da50917726eae069c6b02
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110368516"
---
# <a name="tutorial-analyze-live-video-by-using-live-video-analytics-with-intel-openvino-dl-streamer--edge-ai-extension"></a>Tutorial: Análisis de vídeo en directo mediante Live Video Analytics con la extensión de inteligencia artificial de Edge OpenVINO™ DL Streamer 

[!INCLUDE [redirect to Azure Video Analyzer](./includes/redirect-video-analyzer.md)]

En este tutorial se muestra cómo usar la extensión de inteligencia artificial de Edge OpenVINO™ DL Streamer de Intel para analizar una fuente de vídeo en directo desde una cámara IP (simulada). Verá la forma en que este servidor de inferencia le proporciona acceso a diferentes modelos de detección de objetos (una persona, un vehículo o una bicicleta), clasificación de objetos (atribuciones de los vehículos) y un modelo para hacer un seguimiento de objetos (persona, vehículo y bicicleta). La integración con el módulo gRPC permite enviar fotogramas de vídeo al servidor de inferencia de inteligencia artificial. Luego, los resultados se envían al centro de IoT Edge. Su este servicio de inferencia se ejecuta en el mismo nodo de proceso que Live Video Analytics, puede aprovechar el envío de datos de vídeo a través de la memoria compartida. Esto le permite ejecutar la inferencia a la velocidad de fotogramas de la fuente de vídeo en directo (por ejemplo, 30 fotogramas por segundo). 

En este tutorial se usa una máquina virtual de Azure como dispositivo IoT Edge y se emplea una secuencia de vídeo en directo simulada. Se basa en el código de ejemplo escrito en C# y se basa en [Inicio rápido: Detección de movimiento y emisión de eventos](detect-motion-emit-events-quickstart.md).

> [!NOTE]
> En este tutorial se requiere el uso de una máquina x86-64 como dispositivo perimetral.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure que incluya una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), en caso de que aún no lo haya hecho.
  > [!NOTE]
  > Necesitará una suscripción de Azure con permisos para crear entidades de servicio (el **rol de propietario** permite esto). Si no tiene los permisos adecuados, póngase en contacto con el administrador de la cuenta para que se los conceda. 
* [Visual Studio Code](https://code.visualstudio.com/) con las siguientes extensiones:
    * [Herramientas de Azure IoT](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
    * [C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* [SDK de .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1).
* Si no realizó el inicio rápido [Detección de movimiento y emisión de eventos](detect-motion-emit-events-quickstart.md), asegúrese de llevar a cabo los pasos para [configurar los recursos de Azure](detect-motion-emit-events-quickstart.md#set-up-azure-resources).

> [!TIP]
> Al instalar Azure IoT Tools, es posible que se le pida que instale Docker. Puede omitir el aviso.

## <a name="review-the-sample-video"></a>Revisión del vídeo de ejemplo

Cuando se configuran los recursos de Azure, se copia un vídeo corto de un aparcamiento en la máquina virtual Linux en Azure que se va a usar como dispositivo IoT Edge. En este inicio rápido se usa el archivo de vídeo para simular una secuencia en directo.

Abra una aplicación como [VLC Media Player](https://www.videolan.org/vlc/). Seleccione Ctrl + N y, después, pegue un vínculo al [vídeo](https://lvamedia.blob.core.windows.net/public/lots_015.mkv) para iniciar la reproducción. Se verá el metraje de los vehículos de un aparcamiento, la mayoría de ellos estacionados, y uno en movimiento.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4LUbN]

En este inicio rápido, usará Live Video Analytics en IoT Edge, junto con la extensión de inteligencia artificial perimetral Intel OpenVINO™ DL Streamer de Intel para detectar o clasificar objetos como vehículos o para hacer un seguimiento de vehículos, personas o bicicletas. Los eventos de inferencia resultantes se publicarán en el centro de IoT Edge.

## <a name="overview"></a>Información general

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/use-intel-openvino-tutorial/grpc-vas-extension-with-vino.svg" alt-text="Introducción a LVA MediaGraph":::

En este diagrama se muestra cómo fluyen las señales en este inicio rápido. Un [módulo de Edge](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555) simula una cámara IP que hospeda un servidor con el Protocolo de streaming en tiempo real (RTSP). Un nodo de [origen RTSP](media-graph-concept.md#rtsp-source) extrae la fuente de vídeo de este servidor y envía fotogramas de vídeo al nodo del [procesador de extensiones gRPC](media-graph-concept.md#grpc-extension-processor). 

Este nodo toma fotogramas de vídeo descodificados como entrada y los retransmite a un punto de conexión [gRPC](terminology.md#grpc) expuesto por un servidor de gRPC. El nodo admite la transferencia de datos mediante [memoria compartida](https://en.wikipedia.org/wiki/Shared_memory) o la incrustación directa del contenido en el cuerpo de los mensajes gRPC. Además, el nodo tiene un formateador de imagen integrado para escalar y codificar los fotogramas de vídeo antes de retransmitirlos al punto de conexión gRPC. El escalador tiene opciones para conservar, rellenar o ajustar la relación de aspecto de la imagen. El codificador de imágenes admite los formatos jpeg, png o bmp. Obtenga más información sobre el procesador [aquí](media-graph-extension-concept.md#grpc-extension-processor).

En este tutorial, aprenderá lo siguiente:

1. Implementar el grafo multimedia.
1. Interpretará los resultados.
1. Limpieza de recursos.

## <a name="about-intel-openvino-dl-streamer--edge-ai-extension-module"></a>Acerca del módulo Extensión de inteligencia artificial de Edge Intel OpenVINO™ DL Streamer


El módulo Extensión de inteligencia artificial perimetral Intel OpenVINO™ DL Streamer es un microservicio basado en el servicio de análisis de vídeo de Intel (servicio de VA) que sirve de canalizaciones de análisis de vídeo creadas con OpenVINO™ DL Streamer. Los desarrolladores pueden enviar fotogramas de vídeo descodificados al módulo de extensión de inteligencia artificial que realiza la detección, la clasificación o el seguimiento, y devuelve los resultados. El módulo de extensión de inteligencia artificial expone las API de gRPC que son compatibles con plataformas de análisis de vídeo como Live Video Analytics en IoT Edge de Microsoft. 

Con el fin de crear soluciones de análisis de vídeo en directo complejas y de alto rendimiento, el módulo Live Video Analytics en IoT Edge se debe emparejar con un potente motor de inferencia que pueda aprovechar la escala en el perímetro. En este tutorial, se envían solicitudes de inferencia a la [Extensión de inteligencia artificial perimetral Intel OpenVINO™ DL Streamer](), un módulo de Edge que se ha diseñado para funcionar con Live Video Analytics en IoT Edge.

En la versión inicial de este servidor de inferencia, se tiene acceso a los siguientes [modelos](https://github.com/intel/video-analytics-serving/tree/master/samples/lva_ai_extension#edge-ai-extension-module-options):

- object_detection for person_vehicle_bike_detection ![detección de objetos para vehículo](./media/use-intel-openvino-tutorial/object-detection.png)

- object_classification for vehicle_attributes_recognition ![clasificación de objetos para vehículo](./media/use-intel-openvino-tutorial/object-classification.png)

- object_tracking for person_vehicle_bike_tracking ![seguimiento de objetos para vehículo y persona](./media/use-intel-openvino-tutorial/object-tracking.png)

Usa la detección de objetos cargados previamente, la clasificación de objetos y las canalizaciones de seguimiento de objetos para empezar a trabajar rápidamente. Además, incluye los modelos [person-vehicle-bike-detection-crossroad-0078](https://github.com/openvinotoolkit/open_model_zoo/blob/master/models/intel/person-vehicle-bike-detection-crossroad-0078/description/person-vehicle-bike-detection-crossroad-0078.md) y [vehicle-attributes-recognition-barrier-0039](https://github.com/openvinotoolkit/open_model_zoo/blob/master/models/intel/vehicle-attributes-recognition-barrier-0039/description/vehicle-attributes-recognition-barrier-0039.md) precargados.

> [!NOTE]
> Al descargar y usar el módulo de Edge: Extensión OpenVINO™ DL Streamer – AI de Intel, y el software incluido, acepte los términos y condiciones del [contrato de licencia](https://www.intel.com/content/www/us/en/legal/terms-of-use.html).
> Intel se compromete a respetar los derechos humanos y a evitar ser cómplice de la vulneración de tales derechos. Consulte los [principios de los derechos humanos globales de Intel](https://www.intel.com/content/www/us/en/policy/policy-human-rights.html). Los productos y el software de Intel están concebidos únicamente para usarse en aplicaciones que no infrinjan los derechos humanos reconocidos internacionalmente, ni contribuyan a tal infracción.

En su caso concreto, puede usar la flexibilidad de las distintas canalizaciones. Para ello solo tiene que cambiar las variables de entorno de la canalización en la plantilla de implementación. Esto le permite cambiar rápidamente el modelo de canalización y, cuando se combina con Live Video Analytics, en segundos se cambia la canalización de medios y el modelo de inferencia.  

## <a name="create-and-deploy-the-media-graph"></a>Creación e implementación del grafo de elementos multimedia

### <a name="examine-and-edit-the-sample-files"></a>Examen y edición de los archivos de ejemplo

Como parte de los requisitos previos, ha descargado el código de ejemplo en una carpeta. Siga estos pasos para examinar y editar los archivos de ejemplo.

1. En Visual Studio Code, vaya a *src/edge*. Puede ver el archivo *.env* y algunos archivos de plantilla de implementación.

    La plantilla de implementación hace referencia al manifiesto de implementación del dispositivo perimetral. Incluye algunos valores de marcador de posición. El archivo *.env* incluye los valores de esas variables.

1. Vaya a la carpeta *src/cloud-to-device-console-app*. Aquí verá tanto su archivo *appsettings.json* como otros archivos:

    * ***c2d-console-app.csproj***: el archivo de proyecto de Visual Studio Code.
    * ***operations.json***: una lista de las operaciones que desea que ejecute el programa.
    * ***Program.cs***: el código del programa de ejemplo. Este código:

        * Carga la configuración de la aplicación.
        * Invoca los métodos directos que expone el módulo Live Video Analytics en IoT Edge. Puede usar el módulo para analizar secuencias de vídeo en directo mediante la invocación de sus [métodos directos](direct-methods.md).
        * Se pone en pausa para que pueda examinar la salida del programa en la ventana **TERMINAL** y los eventos generados por el módulo en la ventana **SALIDA**.
        * Invoca los métodos directos para limpiar los recursos.


1. Edite el archivo *operations.json*:
    * Cambie el vínculo a la topología del grafo:

        `"topologyUrl" : "https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/grpcExtensionOpenVINO/2.0/topology.json"`

    * En `GraphInstanceSet`, edite el nombre de la topología del grafo para que coincida con el valor del vínculo anterior:

      `"topologyName" : "InferencingWithOpenVINOgRPC"`

    * En `GraphTopologyDelete`, edite el nombre:

      `"name": "InferencingWithOpenVINOgRPC"`

### <a name="generate-and-deploy-the-iot-edge-deployment-manifest"></a>Generación e implementación del manifiesto de implementación de IoT Edge

1. Haga clic con el botón derecho en el archivo *src/edge/deployment.openvino.template.json* y seleccione **Generación de un manifiesto de implementación de IoT Edge**.

    ![Generación de un manifiesto de implementación de IoT Edge](./media/use-intel-openvino-tutorial/generate-deployment-manifest.png)  

    El archivo de manifiesto *deployment.openvino.grpc.cpu.amd64.json* se crea en la carpeta *src/edge/config*.

> [!NOTE]
> También se ha incluido una plantilla *deployment.openvino.grpc.gpu.template.jsen* que habilita el soporte de GPU para el módulo Extensión de inteligencia artificial de Edge para el módulo de extensión de la extensión de AI de Intel OpenVINO DL Streamer. Estas plantillas apuntan a la imagen de Docker Hub de Intel.

Las plantillas mencionadas apuntan a la imagen de Docker Hub de Intel. Si prefiere hospedar una copia por su propio Azure Container Registry puede seguir los pasos 1 y 2 siguientes:
1. Incluya SSH en un dispositivo con las herramientas de la CLI de Docker instaladas (es decir, el dispositivo perimetral) y extraiga/etiquete/inserte el contenedor con estos pasos:
    * Extraiga la imagen de Intel de Docker Hub:

        `sudo docker pull intel/video-analytics-serving:0.4.1-dlstreamer-edge-ai-extension`
    
    * Etiquete la imagen de Intel con su propio nombre de Azure Container Registry. Reemplace {SU NOMBRE DE ACR} por el nombre de Azure Container Registry que puede encontrar en el archivo. env:

        `sudo docker image tag intel/video-analytics-serving:0.4.1-dlstreamer-edge-ai-extension {YOUR ACR NAME/video-analytics-serving:0.4.1-dlstreamer-edge-ai-extension}`
    
    * Inserte la imagen etiquetada en Azure Container Registry:

        `sudo docker push {YOUR ACR NAME/video-analytics-serving:0.4.1-dlstreamer-edge-ai-extension}`
    
2. Ahora debe editar las plantillas para hacer referencia a la nueva imagen hospedada en Azure Container Registry.
    * Haga clic con el botón derecho en el archivo *deployment.openvino.grpc.cpu.template.json* y vaya a la parte del módulo *lavExtension* y reemplace:

        `intel/video-analytics-serving:0.4.1-dlstreamer-edge-ai-extension`

        Por:

        `{YOUR ACR NAME/video-analytics-serving:0.4.1-dlstreamer-edge-ai-extension}`
    * Repita el paso 2 en el archivo *deployment.openvino.grpc.gpu.template.json*


3. Si ha completado el inicio rápido [Detección de movimiento y emisión de eventos](detect-motion-emit-events-quickstart.md), omita este paso. 

    Si no lo ha hecho, cerca del panel **AZURE IOT HUB**, en la esquina inferior izquierda, seleccione el icono **Más acciones** y, después, seleccione **Set IoT Hub Connection String** (Establecer cadena de conexión de IoT Hub). La cadena se puede copiar del archivo *appsettings.json*. O bien, para asegurarse de que ha configurado el centro de IoT adecuado en Visual Studio Code, use el [comando de selección de IoT Hub](https://github.com/Microsoft/vscode-azure-iot-toolkit/wiki/Select-IoT-Hub).
    
    ![Establecimiento de la cadena de conexión de IoT Hub](./media/quickstarts/set-iotconnection-string.png)

> [!NOTE]
> Es posible que se le pida que proporcione información del punto de conexión integrado del centro de IoT. Para obtener esa información, en Azure Portal, vaya a su centro de IoT y busque la opción **Puntos de conexión integrados** en el panel de navegación izquierdo. Haga clic ahí y busque el **punto de conexión compatible con el centro de eventos** en la sección **Punto de conexión compatible con el centro de eventos**. Copie y use el texto del cuadro. El punto de conexión será similar a este:  
    ```
    Endpoint=sb://iothub-ns-xxx.servicebus.windows.net/;SharedAccessKeyName=iothubowner;SharedAccessKey=XXX;EntityPath=<IoT Hub name>
    ```

1. Haga clic con el botón derecho en *src/edge/config/deployment.openvino.grpc.cpu.template.json* y seleccione **Crear una implementación para un dispositivo individual**. 

    ![Crear una implementación para un dispositivo individual](./media/use-intel-openvino-tutorial/deploy-manifest.png)

1. Cuando se le pida que seleccione un dispositivo IoT Hub, seleccione **lva-sample-device**.
1. Tras aproximadamente 30 segundos, en la esquina inferior izquierda de la ventana, actualice Azure IoT Hub. El dispositivo perimetral ahora muestra los siguientes módulos implementados:

    * El módulo de Live Video Analytics, denominado **lvaEdge**.
    * El módulo **rtspsim**, que simula un servidor RTSP y actúa como el origen de una fuente de vídeo en directo.
    * El módulo **lvaExtension**, que es la Extensión de inteligencia artificial de Edge Intel OPENVINO™ DL Streamer 

### <a name="prepare-to-monitor-events"></a>Preparación para supervisar eventos

Haga clic con el botón derecho en el dispositivo de Live Video Analytics y seleccione **Start Monitoring Built-in Event Endpoint** (Iniciar supervisión del punto de conexión de eventos integrado). Este paso es necesario para supervisar los eventos de IoT Hub en la ventana **SALIDA** de Visual Studio Code. 

![Iniciar supervisión](./media/quickstarts/start-monitoring-iothub-events.png) 

### <a name="run-the-sample-program-to-detect-vehicles-persons-or-bike"></a>Ejecución del programa de ejemplo para detectar personas, vehículos o bicicletas
Si abre la [topología de grafos](https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/grpcExtensionOpenVINO/2.0/topology.json) para este tutorial en un explorador, verá que el valor de `grpcExtensionAddress` se ha establecido en `tcp://lvaExtension:5001`, en comparación con el ejemplo *httpExtensionOpenVINO*, no necesita cambiar la dirección URL al servidor de gRPC. En su lugar, indique al módulo que ejecute una canalización específica según las variables de entorno, como se ha mencionado. En la plantilla predeterminada, se ha establecido en: "object_detection" para "person_vehicle_bike_detection". Puede experimentar con otras canalizaciones admitidas. 

1. En Visual Studio Code, abra la pestaña **Extensiones** (o presione Ctrl + Mayús + X) y busque Azure IoT Hub.
1. Haga clic con el botón derecho y seleccione la **Configuración de la extensión**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/extensions-tab.png" alt-text="Configuración de la extensión":::
1. Busque y habilite "Show Verbose Message" (Mostrar mensaje detallado).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/show-verbose-message.png" alt-text="Show Verbose Message"::: (Mostrar mensaje detallado)
1. Para iniciar una sesión de depuración, seleccione la tecla F5. Verá mensajes impresos en la ventana **TERMINAL**.
1. El código de *operations.json* comienza con llamadas a los métodos directos `GraphTopologyList` y `GraphInstanceList`. Si ha limpiado los recursos tras haber completado los inicios rápidos anteriores, este proceso devolverá listas vacías y, después, se pausará. Para continuar, seleccione la tecla Entrar.

    La ventana **TERMINAL** muestra el siguiente conjunto de llamadas al método directo:

     * Una llamada a `GraphTopologySet` que utiliza la instancia anterior de `topologyUrl`.
     * Una llamada a `GraphInstanceSet` que usa el cuerpo siguiente:

         ```
         {
           "@apiVersion": "2.0",
           "name": "Sample-Graph-1",
           "properties": {
             "topologyName": "InferencingWithOpenVINOgRPC",
             "description": "Sample graph description",
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
               }
             ]
           }
         }
         ```

     * Una llamada a `GraphInstanceActivate` que inicia la instancia del grafo y el flujo de vídeo.
     * Una segunda llamada a `GraphInstanceList` que muestra que la instancia del grafo está en ejecución.
1. La salida de la ventana **TERMINAL** se pone en pausa tras el mensaje `Press Enter to continue`. No seleccione Entrar todavía. Desplácese hacia arriba para ver las cargas de la respuesta JSON para los métodos directos que ha invocado.
1. Cambie a la ventana **SALIDA** de Visual Studio Code. Verá los mensajes que el módulo Live Video Analytics en IoT Edge envía al centro de IoT. En la siguiente sección de este inicio rápido se analizan estos mensajes.
1. El grafo multimedia continúa ejecutándose e imprimiendo los resultados. El simulador RTSP sigue recorriendo el vídeo de origen. Para detener el grafo multimedia, vuelva a la ventana **TERMINAL** y seleccione Entrar. 

    La siguiente serie de llamadas limpia los recursos:
      * Una llamada a `GraphInstanceDeactivate` desactiva la instancia del grafo.
      * Una llamada a `GraphInstanceDelete` elimina la instancia.
      * Una llamada a `GraphTopologyDelete` elimina la topología.
      * Una llamada final a `GraphTopologyList` muestra que la lista está vacía.

## <a name="interpret-results"></a>Interpretación de los resultados

Cuando se ejecuta el grafo multimedia, los resultados del nodo del procesador de extensión HTTP atraviesan el nodo de receptor de IoT Hub y llegan al centro de IoT. Los mensajes que aparecen en la ventana **SALIDA** contienen una sección `body` y una sección `applicationProperties`. Para más información, consulte [Creación y lectura de mensajes de IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).

En los mensajes siguientes, se definen las propiedades de la aplicación y el contenido del cuerpo mediante el módulo Live Video Analytics. 

### <a name="mediasessionestablished-event"></a>Evento MediaSessionEstablished

Cuando se crean instancias del grafo multimedia, el nodo de origen RTSP intenta conectarse al servidor RTSP que se ejecuta en el contenedor rtspsim-live555. Si la conexión se realiza correctamente, se imprime el evento siguiente. El tipo de evento es **Microsoft.Media.MediaGraph.Diagnostics.MediaSessionEstablished**.

```
[IoTHubMonitor] [9:42:18 AM] Message received from [lvaedgesample/lvaEdge]:
{
  "sdp&quot;: &quot;SDP:\nv=0\r\no=- 1612432131600584 1 IN IP4 172.18.0.6\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/homes_00425.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.08.19\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-214.166\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/homes_00425.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=64001F;sprop-parameter-sets=Z2QAH6zZQFAFuwFsgAAAAwCAAAAeB4wYyw==,aOvhEsiw\r\na=control:track1\r\n"
}
```

En este mensaje, tenga en cuenta estos detalles:

* El mensaje es un evento de diagnóstico. `MediaSessionEstablished` indica que el nodo de origen RTSP (el sujeto) ha conectado con el simulador RTSP y ha comenzado a recibir una fuente en directo (simulada).
* En `applicationProperties`, `subject` indica que el mensaje se generó desde el nodo de origen RTSP del grafo multimedia.
* En `applicationProperties`, `eventType` indica que este evento es de diagnóstico.
* `eventTime` indica la hora a la que se produjo el evento.
* `body` contiene datos sobre el evento de diagnóstico. En este caso, los datos incluyen detalles sobre el [protocolo de descripción de sesiones (SDP)](https://en.wikipedia.org/wiki/Session_Description_Protocol).

### <a name="inference-event"></a>Evento de inferencia

El nodo del procesador de la extensión gRPC recibe los resultados de la inferencia de Extensión de inteligencia artificial de Edge Intel OpenVINO™ DL Streamer. Luego, emite los resultados a través del nodo del receptor de IoT Hub como eventos de inferencia. 

En estos eventos, el tipo se establece en `entity` para indicar que es una entidad, como un coche o un camión. El valor `eventTime` es la hora UTC en que se detectó el objeto. 

En el ejemplo siguiente, verá que se identificó un vehículo, el tipo de vehículo (furgoneta) y el color (blanco), y todo ello con un nivel de confianza por encima de 0,9; también asignó un identificador a la entidad cuando usamos el modelo de seguimiento de objetos.

```
[IoTHubMonitor] [9:43:18 AM] Message received from [lva-sample-device/lvaEdge]:
{
  "timestamp": 145118912223221,
  "inferences": [
    {
      "type": "entity",
      "entity": {
        "tag": {
          "value": "vehicle",
          "confidence": 0.9605301
        },
        "attributes": [
          {
            "name": "color",
            "value": "white",
            "confidence": 0.9605301
          },
          {
            "name": "type",
            "value": "car",
            "confidence": 0.9605301
          }
        ],
        "box": {
          "l": 0.3958135,
          "t": 0.078730375,
          "w": 0.48403296,
          "h": 0.94352424
        },
        "id&quot;: &quot;1"
      }
    }
}
```

En los mensajes, tenga en cuenta los siguientes detalles:

* En `applicationProperties`, `subject` hace referencia al nodo de la topología del grafo multimedia desde el que se generó el mensaje. 
* En `applicationProperties`, `eventType` indica que este es un evento de análisis.
* El valor `eventTime` es la hora en que se produjo el evento.
* La sección `body` contiene datos sobre el evento de análisis. En este caso, el evento es un evento de inferencia, por lo que el cuerpo contiene los datos `inferences`.
* La sección `inferences` indica que el valor de `type` es `entity`. En esta sección se incluyen datos adicionales sobre la entidad.

## <a name="run-the-sample-program-to-detect-persons-or-vehicles-or-bikes"></a>Ejecución del programa de ejemplo para detectar personas, vehículos o bicicletas
Para usar otro modelo, será preciso cambiar la plantilla de implementación. Para alternar entre los modelos admitidos, puede cambiar las variables de entorno que se encuentran en el módulo lvaExtenstion. Consulte este [documento en GitHub](https://github.com/intel/video-analytics-serving/tree/master/samples/lva_ai_extension#edge-ai-extension-module-options) para conocer los valores y las combinaciones admitidos para los modelos.

```
"Env":[
"PIPELINE_NAME=object_classification",
"PIPELINE_VERSION=vehicle_attributes_recognition"
],
```
> [!TIP]
> Copie la plantilla y guárdela con un nombre nuevo para cada canalización posible. De esta manera, puede cambiar de modelo creando una implementación basada en una de estas plantillas.

Una vez que haya cambiado las variables, puede volver a implementar la plantilla en el dispositivo. Ya puede repetir los pasos anteriores para volver a ejecutar el programa de ejemplo con la nueva canalización. Los resultados de la inferencia serán similares (en el esquema), pero muestran más o menos información según el modelo de canalización que se elija.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si su intención es probar otros inicios rápidos o tutoriales, conserve los recursos creados. En caso contrario, vaya a Azure Portal, luego a los grupos de recursos, seleccione el grupo de recursos donde ejecutó este tutorial y elimine todos los recursos.

## <a name="next-steps"></a>Pasos siguientes

Revise los desafíos adicionales para los usuarios avanzados:

* Use una [cámara IP](https://en.wikipedia.org/wiki/IP_camera) que sea compatible con RTSP, en lugar de utilizar el simulador RTSP. Puede buscar cámaras IP compatibles con RTSP en la página de [productos compatibles con ONVIF](https://www.onvif.org/conformant-products/). Busque dispositivos que se ajusten a los perfiles G, S o T.
* Use un dispositivo Intel x64 con Linux, en lugar de una máquina virtual Linux de Azure. El dispositivo debe estar en la misma red que la cámara IP. Puede seguir las instrucciones que aparecen en [Instalación del entorno de ejecución de Azure IoT Edge en Linux](../../iot-edge/how-to-install-iot-edge.md). Luego, registre el dispositivo en Azure IoT Hub, para lo que debe seguir las instrucciones que se encuentran en [Implementación del primer módulo IoT Edge en un dispositivo virtual Linux](../../iot-edge/quickstart-linux.md).
