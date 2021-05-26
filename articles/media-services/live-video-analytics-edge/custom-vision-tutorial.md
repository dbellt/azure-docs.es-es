---
title: Análisis de vídeo en directo con Live Video Analytics en IoT Edge y Azure Custom Vision
description: Aprenda a usar Azure Custom Vision para crear un modelo en contenedor que pueda detectar un camión de juguete y usar la funcionalidad de extensibilidad de IA de Live Video Analytics en Azure IoT Edge para implementar el modelo en el perímetro y detectar camiones de juguete a partir de una secuencia de vídeo en directo.
ms.topic: tutorial
ms.date: 09/08/2020
zone_pivot_groups: ams-lva-edge-programming-languages
ms.openlocfilehash: 3d9718a8c7cf6f93af32a3bd33bcf1e8c75c6aa8
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110378273"
---
# <a name="tutorial-analyze-live-video-with-azure-live-video-analytics-on-iot-edge-and-azure-custom-vision"></a>Tutorial: Análisis de vídeo en directo con Live Video Analytics en IoT Edge y Azure Custom Vision

[!INCLUDE [redirect to Azure Video Analyzer](./includes/redirect-video-analyzer.md)]

En este tutorial, aprenderá a usar Azure [Custom Vision](https://azure.microsoft.com/services/cognitive-services/custom-vision-service/) para crear un modelo en contenedor que pueda detectar un camión de juguete y usar la [funcionalidad de extensibilidad de IA](analyze-live-video-concept.md#analyzing-video-using-a-custom-vision-model) de Azure Live Video Analytics en Azure IoT Edge para implementar el modelo en el perímetro y detectar camiones de juguete a partir de una secuencia de vídeo en directo.

Mostraremos cómo reunir el potencial de Custom Vision para crear y entrenar un modelo de Computer Vision mediante la carga y el etiquetado de algunas imágenes. No son necesarios conocimientos de ciencia de datos, aprendizaje automático o inteligencia artificial. También aprenderá sobre las funcionalidades de Live Video Analytics para implementar fácilmente un modelo personalizado como un contenedor en el perímetro y analizar una fuente de vídeo en directo simulada.

::: zone pivot="programming-language-csharp"
[!INCLUDE [header](includes/custom-vision-tutorial/csharp/header.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [header](includes/custom-vision-tutorial/python/header.md)]
::: zone-end

En este tutorial se muestra cómo realizar las siguientes acciones:

> [!div class="checklist"]
> * Configuración de los recursos correspondientes
> * Creación de un modelo de Custom Vision en la nube para detectar camiones de juguete y su implementación en el perímetro.
> * Creación e implementación de un grafo multimedia con una extensión HTTP en un modelo de Custom Vision.
> * Ejecución del código de ejemplo
> * Examen e interpretación de los resultados

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="suggested-pre-reading"></a>Sugerencias antes de la lectura  

Consulte los artículos siguientes antes de empezar:

* [Introducción a Análisis de vídeos en vivo en IoT Edge](overview.md)
* [Introducción a Azure Custom Vision](../../cognitive-services/custom-vision-service/overview.md)
* [Terminología de Live Video Analytics en IoT Edge](terminology.md)
* [Concepto de grafo multimedia](media-graph-concept.md)
* [Análisis de vídeos en vivo sin grabación de vídeo](analyze-live-video-concept.md)
* [Ejecución de Live Video Analytics con su propio modelo](use-your-model-quickstart.md)
* [Tutorial: Desarrollo de un módulo IoT Edge](../../iot-edge/tutorial-develop-for-linux.md)
* [Cómo editar deployment.*.template.json](https://github.com/microsoft/vscode-azure-iot-edge/wiki/How-to-edit-deployment.*.template.json)

## <a name="prerequisites"></a>Requisitos previos

::: zone pivot="programming-language-csharp"
[!INCLUDE [prerequisites](includes/custom-vision-tutorial/csharp/prerequisites.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [prerequisites](includes/custom-vision-tutorial/python/prerequisites.md)]
::: zone-end

> [!IMPORTANT]
> Este módulo de Custom Vision solo admite arquitecturas **Intel x86 y amd64**. Compruebe la arquitectura del dispositivo perimetral antes de continuar.

## <a name="review-the-sample-video"></a>Revisión del vídeo de ejemplo

En este tutorial se usa un archivo de [vídeo de inferencia de coches de juguete](https://lvamedia.blob.core.windows.net/public/t2.mkv) para simular streaming en vivo. Puede examinar el vídeo mediante una aplicación como [VLC Media Player](https://www.videolan.org/vlc/). Seleccione **Ctrl+N** y, después, pegue un vínculo al [vídeo de inferencia de coches de juguete](https://lvamedia.blob.core.windows.net/public/t2.mkv) para iniciar la reproducción. Cuando vea el vídeo, observe que, en el marcador de 36 segundos, aparece un camión de juguete. El modelo personalizado se ha entrenado para detectar este camión de juguete concreto. 

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4LPwK]

En este tutorial, usará Live Video Analytics en IoT Edge para detectar esos camiones de juguete y publicar eventos de inferencia asociados en el centro de IoT Edge.

## <a name="overview"></a>Información general

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/custom-vision-tutorial/topology-custom-vision.svg" alt-text="Diagrama que muestra una información general de Custom Vision.":::

En este diagrama se muestra el flujo de las señales en este tutorial. Un [módulo perimetral](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555) simula una cámara IP que hospeda un servidor de protocolo RTSP. Un nodo de [origen RTSP](media-graph-concept.md#rtsp-source) extrae la fuente de vídeo de este servidor y envía fotogramas de vídeo al nodo del [procesador de extensiones HTTP](media-graph-concept.md#http-extension-processor).

El nodo de extensión HTTP desempeña el rol de un proxy.  Se muestrean los fotogramas de vídeo entrantes que configuró mediante el campo `samplingOptions` y también se convierten los fotogramas de vídeo en el tipo de imagen especificado. Luego, retransmite la imagen a través de REST a otro módulo perimetral que ejecuta un modelo de IA detrás de un punto de conexión HTTP. En este ejemplo, ese módulo perimetral es el modelo de detección de camiones de juguete creado con Custom Vision. El nodo del procesador de extensión HTTP recopila los resultados de la detección y publica los eventos en el nodo del [receptor de Azure IoT Hub](media-graph-concept.md#iot-hub-message-sink). A continuación, el nodo envía esos eventos al [centro de IoT Edge](../../iot-fundamentals/iot-glossary.md#iot-edge-hub).

## <a name="build-and-deploy-a-custom-vision-toy-detection-model"></a>Creación e implementación de un modelo de detección de juguetes de Custom Vision 

Custom Vision (que en español significa "visión personalizada") permite crear, como su nombre indica, su propio detector o clasificador personalizado de objetos en la nube. Ofrece una interfaz sencilla, fácil de usar e intuitiva para crear modelos de Custom Vision que se pueden implementar en la nube o en el perímetro mediante contenedores.

Para crear un detector de camiones de juguete, siga los pasos descritos en [Inicio rápido: Creación de un detector de objetos con el sitio web de Custom Vision](../../cognitive-services/custom-vision-service/get-started-build-detector.md).

> [!IMPORTANT]
> Este módulo de Custom Vision solo admite arquitecturas **Intel x86 y amd64**. Compruebe la arquitectura del dispositivo perimetral antes de continuar.

Notas adicionales:
 
* En este tutorial, no use las imágenes de ejemplo que se proporcionan en la [sección de requisitos previos](../../cognitive-services/custom-vision-service/get-started-build-detector.md#prerequisites) del artículo de inicio rápido. En su lugar, hemos usado un conjunto de imágenes determinado para crear un modelo de Custom Vision de detector de juguetes. Use [estas imágenes](https://lvamedia.blob.core.windows.net/public/ToyCarTrainingImages.zip) cuando se le pida que [elija las imágenes de entrenamiento](../../cognitive-services/custom-vision-service/get-started-build-detector.md#choose-training-images) en la guía de inicio rápido.
* En la sección de etiquetado de imágenes del inicio rápido, asegúrese de que asigna al camión de juguete que se ve en la imagen la etiqueta "delivery truck" (camión de entrega).

Cuando haya terminado, puede exportar el modelo a un contenedor de Docker mediante el botón **Export** (Exportar) de la pestaña **Performance** (Rendimiento). Asegúrese de elegir Linux como tipo de plataforma de contenedor. Esta es la plataforma en la que se ejecutará el contenedor. Puede descargar el contenedor en una máquina Windows o Linux. En las instrucciones siguientes se supone que el archivo contenedor se ha descargado en una máquina Windows.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/custom-vision-tutorial/docker-file.png" alt-text="Pantalla que muestra el archivo Dockerfile seleccionado.":::
 
1. Debe tener un archivo zip descargado en la máquina local denominado `<projectname>.DockerFile.Linux.zip`. 
1. Compruebe si tiene instalado Docker. En caso contrario, instale [Docker](https://docs.docker.com/get-docker/) para el escritorio de Windows.
1. Descomprima el archivo descargado en la ubicación que elija. Use la línea de comandos para ir al directorio de carpetas descomprimidas.
    
    Ejecute los comandos siguientes:
    
    1. `docker build -t cvtruck` 
    
        Este comando descarga muchos paquetes, crea la imagen de Docker y la etiqueta como `cvtruck:latest`.
    
        > [!NOTE]
        > Si se realiza correctamente, debería ver los mensajes siguientes: `Successfully built <docker image id>` y `Successfully tagged cvtruck:latest`. Si se produce un error en el comando de creación, inténtelo de nuevo. A veces, los paquetes de dependencias no se descargan la primera vez.
    1. `docker  image ls`

        Este comando comprueba si la nueva imagen está en el registro local.
    1. `docker run -p 127.0.0.1:80:80 -d cvtruck`
    
        Este comando debe publicar el puerto expuesto de Docker (80) en el puerto de la máquina local (80).
    1. `docker container ls`
    
        Este comando comprueba las asignaciones de puertos y si el contenedor de Docker se ejecuta correctamente en la máquina. La salida debe ser similar a la siguiente:

        ```
        CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
        8b7505398367        cvtruck             "/bin/sh -c 'python …"   13 hours ago        Up 25 seconds       127.0.0.1:80->80/tcp   practical_cohen
        ```
      1. `curl -X POST http://127.0.0.1:80/image -F imageData=@<path to any image file that has the toy delivery truck in it>`
            
            Este comando prueba el contenedor en el equipo local. Si la imagen tiene el mismo camión de entrega que se ha entrenado en el modelo, la salida debe ser similar a la del ejemplo siguiente. Sugiere que se detectó el camión de entrega con una probabilidad del 90,12 %.
    
            ```
            {"created":"2020-03-20T07:10:47.827673","id":"","iteration":"","predictions":[{"boundingBox":{"height":0.66167289,"left":-0.03923762,"top":0.12781593,"width":0.70003178},"probability":0.90128148,"tagId":0,"tagName":"delivery truck"},{"boundingBox":{"height":0.63733053,"left":0.25220079,"top":0.0876643,"width":0.53331227},"probability":0.59745145,"tagId":0,"tagName":"delivery truck"}],"project":""}
            ```

<!--## Create and deploy a media graph with http extension to custom vision model-->

## <a name="examine-the-sample-files"></a>Examen de los archivos de ejemplo

::: zone pivot="programming-language-csharp"
[!INCLUDE [examine-sample-files](includes/custom-vision-tutorial/csharp/examine-sample-files.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [examine-sample-files](includes/custom-vision-tutorial/python/examine-sample-files.md)]
::: zone-end

## <a name="generate-and-deploy-the-deployment-manifest"></a>Generación e implementación del manifiesto de implementación

1. En Visual Studio Code, vaya a src/cloud-to-device-console-app/operations.json.

1. En `GraphTopologySet`, asegúrese de que se cumple lo siguiente:<br/>`"topologyUrl" : "https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/httpExtension/2.0/topology.json"`
1. En `GraphInstanceSet`, asegúrese de lo siguiente:
    1. `"topologyName" : "InferencingWithHttpExtension"`
    1. Agregue lo siguiente en la parte superior de la matriz de parámetros: `{"name": "inferencingUrl","value": "http://cv:80/image"},`
    1. Cambie el valor del parámetro `rtspUrl` a `"rtsp://rtspsim:554/media/t2.mkv"`.
1. En `GraphTopologyDelete`, asegúrese de que `"name": "InferencingWithHttpExtension"`.
1. Haga clic con el botón derecho en el archivo src/edge/deployment.customvision.template.json y seleccione **Generate IoT Edge Deployment Manifest** (Generar manifiesto de implementación de IoT Edge).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/custom-vision-tutorial/deployment-template-json.png" alt-text="Captura de pantalla que muestra la opción para generar un manifiesto de implementación de IoT Edge.":::
  
    Esta acción debe crear un archivo de manifiesto en la carpeta src/edge/config llamado deployment.customvision.amd64.json.
1. Abra el archivo src/edge/deployment.customvision.template.json y busque el bloque JSON `registryCredentials`. En este bloque, encontrará la dirección del registro de contenedor de Azure, junto con su nombre de usuario y contraseña.
1. Inserte el contenedor de Custom Vision local en la instancia de Azure Container Registry mediante estos pasos en la línea de comandos:

    1. Inicie sesión en el registro mediante la ejecución del comando siguiente:
    
        `docker login <address>`
    
        Escriba el nombre de usuario y la contraseña cuando se le solicite autenticación.
        
        > [!NOTE]
        > La contraseña no resulta visible en la línea de comandos.
    1. Etiquete la imagen con este comando: <br/>`docker tag cvtruck   <address>/cvtruck`.
    1. Inserte la imagen con este comando: <br/>`docker push <address>/cvtruck`.

        Si se realiza correctamente, debería ver `Pushed` (Insertado) en la línea de comandos junto con el SHA de la imagen.
    1. También puede confirmarlo mediante la comprobación de la instancia de Azure Container Registry en Azure Portal. Aquí verá el nombre del repositorio junto con la etiqueta.
1. Establezca la cadena de conexión de IoT Hub; para ello, seleccione el icono **More actions** (Más acciones) situado junto al panel **AZURE IOT HUB** en la esquina inferior izquierda. Puede copiarla del archivo appsettings.json. Este es otro enfoque recomendado para asegurarse de que tiene el centro de IoT adecuado configurado en Visual Studio Code mediante el comando [Select IoT Hub](https://github.com/Microsoft/vscode-azure-iot-toolkit/wiki/Select-IoT-Hub) (Seleccionar centro de IoT).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/custom-vision-tutorial/connection-string.png" alt-text="Captura de pantalla que muestra la cadena de conexión de Set IoT Hub.":::
1. A continuación, haga clic con el botón derecho en src/edge/config/deployment.customvision.amd64.json y seleccione **Create Deployment for Single Device** (Crear una implementación para un dispositivo individual).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/custom-vision-tutorial/deployment-amd64-json.png" alt-text="Captura de pantalla que muestra la creación de una implementación para un dispositivo individual.":::
1. A continuación, se le pedirá que seleccione un dispositivo IoT Hub. Seleccione **lva-sample-device** en la lista desplegable.
1. En unos 30 segundos, actualice la instancia de Azure IoT Hub en la sección inferior izquierda. Debería ver que el dispositivo perimetral tiene los siguientes módulos implementados:

    * El módulo	 de Live Video Analytics en IoT Edge llamado `lvaEdge`.
    * Un módulo llamado `rtspsim`, que simula un servidor RTSP y actúa como el origen de una fuente de vídeo en directo.
    * Un módulo llamado `cv` que, como su nombre sugiere, es el modelo de detección de camiones de juguete de Custom Vision que aplica Custom Vision a las imágenes y devuelve varios tipos de etiqueta. (Nuestro modelo se ha entrenado sobre una sola etiqueta: "camión de entrega").

## <a name="prepare-for-monitoring-events"></a>Preparación de la supervisión de eventos

Haga clic con el botón derecho en el dispositivo de Live Video Analytics y seleccione **Start Monitoring Built-in Event Endpoint** (Iniciar supervisión del punto de conexión de eventos integrado). Este paso es necesario para supervisar los eventos de IoT Hub en la ventana **SALIDA** de Visual Studio Code.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/custom-vision-tutorial/start-monitoring.png" alt-text="Captura de pantalla que muestra la opción para iniciar la supervisión del punto de conexión de eventos integrado.":::

## <a name="run-the-sample-program"></a>Ejecución del programa de ejemplo

Si abre la topología de grafos de este tutorial en un explorador, verá que el valor de `inferencingUrl` se ha establecido en `http://cv:80/image`. Esta configuración significa que el servidor de inferencia devolverá resultados después de detectar camiones de juguete, si los hay, en el vídeo en directo.

1. En Visual Studio Code, abra la pestaña **Extensiones** (o seleccione **Ctrl + Mayús + X**) y busque Azure IoT Hub.
1. Haga clic con el botón derecho y seleccione la opción **Configuración de la extensión**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/extensions-tab.png" alt-text="Captura de pantalla que muestra la configuración de la extensión.":::
1. Busque y habilite **Show Verbose Message** (Mostrar mensaje detallado).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/show-verbose-message.png" alt-text="Captura de pantalla que muestra la opción para mostrar mensaje detallado.":::
1. Para iniciar una sesión de depuración, seleccione la tecla **F5**. Verá mensajes impresos en la ventana **TERMINAL**.
1. El código de operations.json comienza con llamadas a los métodos directos `GraphTopologyList` y `GraphInstanceList`. Si ha limpiado los recursos tras haber completado los inicios rápidos anteriores, este proceso devolverá listas vacías y, después, se pondrá en pausa. Para continuar, seleccione la tecla **Entrar**.
    
   La ventana **TERMINAL** muestra el siguiente conjunto de llamadas al método directo:
    
   * Una llamada a `GraphTopologySet` que utiliza la instancia anterior de `topologyUrl`.
   * Una llamada a `GraphInstanceSet` que usa el cuerpo siguiente:
        
   ```
        {
          "@apiVersion": "2.0",
          "name": "Sample-Graph-1",
          "properties": {
            "topologyName": "CustomVisionWithHttpExtension",
            "description": "Sample graph description",
            "parameters": [
              { 
                "name": "inferencingUrl",
                "value": "http://cv:80/image"
              },
              {
                "name": "rtspUrl",
                "value": "rtsp://rtspsim:554/media/t2.mkv"
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
    
1. La salida de la ventana **TERMINAL** se pondrá en pausa con el mensaje **Press Enter to continue** (Presione Entrar para continuar). No seleccione **Entrar** todavía. Desplácese hacia arriba para ver las cargas de la respuesta JSON para los métodos directos que ha invocado.
1. Cambie a la ventana **SALIDA** de Visual Studio Code. Verá los mensajes que el módulo Live Video Analytics en IoT Edge envía al centro de IoT. En la siguiente sección de este tutorial se analizan estos mensajes.
1. El grafo multimedia continúa ejecutándose e imprimiendo los resultados. El simulador RTSP sigue recorriendo el vídeo de origen. Para detener el grafo multimedia, vuelva a la ventana **TERMINAL** y seleccione **Entrar**.
La siguiente serie de llamadas limpia los recursos:
    
   * Una llamada a `GraphInstanceDeactivate` desactiva la instancia del grafo.
   * Una llamada a `GraphInstanceDelete` elimina la instancia.
   * Una llamada a `GraphTopologyDelete` elimina la topología.
   * Una llamada final a `GraphTopologyList` muestra que la lista está vacía.
    
## <a name="interpret-the-results"></a>Interpretación de los resultados

Cuando se ejecuta el grafo multimedia, los resultados del nodo del procesador de extensión HTTP atraviesan el nodo de receptor de IoT Hub y llegan al centro de IoT. Los mensajes que aparecen en la ventana **SALIDA** contienen una sección body y una sección `applicationProperties`. Para más información, consulte [Creación y lectura de mensajes de IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).

En los mensajes siguientes, se definen las propiedades de la aplicación y el contenido del cuerpo mediante el módulo Live Video Analytics.

### <a name="mediasessionestablished-event"></a>Evento MediaSessionEstablished

Cuando se crean instancias del grafo multimedia, el nodo de origen RTSP intenta conectarse al servidor RTSP que se ejecuta en el contenedor rtspsim-live555. Si la conexión se realiza correctamente, se imprime el evento siguiente. El tipo de evento es **Microsoft.Media.MediaGraph.Diagnostics.MediaSessionEstablished**.

```
{
  "body": {
    "sdp": "SDP:\nv=0\r\no=- 1599412849822466 1 IN IP4 172.18.0.3\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/t2.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.04.12\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-78.357\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/t2.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=42C01F;sprop-parameter-sets=Z0LAH9kAUAW6EAAAAwAQAAADAwDxgySA,aMuBcsg=\r\na=control:track1\r\nm=audio 0 RTP/AVP 97\r\nc=IN IP4 0.0.0.0\r\nb=AS:96\r\na=rtpmap:97 MPEG4-GENERIC/44100/2\r\na=fmtp:97 streamtype=5;profile-level-id=1;mode=AAC-hbr;sizelength=13;indexlength=3;indexdeltalength=3;config=121056E500\r\na=control:track2\r\n"
  },
  "applicationProperties": {
    "topic": "/subscriptions/35c2594a-23da-4fce-b59c-f6fb9513eeeb/resourceGroups/lvaavi_0827/providers/microsoft.media/mediaservices/lvasampleulf2rv2x5msy2",
    "subject": "/graphInstances/ GRAPHINSTANCENAMEHERE /sources/rtspSource",
    "eventType": "Microsoft.Media.Graph.Diagnostics.MediaSessionEstablished",
    "eventTime": "2020-09-06T17:20:49.823Z",
    "dataVersion": "1.0"
  }
```

En este mensaje, tenga en cuenta estos detalles:

* El mensaje es un evento de diagnóstico. `MediaSessionEstablished` indica que el nodo de origen RTSP (el sujeto) ha conectado con el simulador RTSP y ha comenzado a recibir una fuente en directo simulada.
* En `applicationProperties`, `subject` indica que el mensaje se generó desde el nodo de origen RTSP del grafo multimedia.
* En `applicationProperties`, el tipo de evento indica que este evento es un evento de diagnóstico.
* La hora del evento indica la hora a la que se produjo el evento.
* El cuerpo contiene datos sobre el evento de diagnóstico. En este caso, los datos incluyen detalles sobre el [protocolo de descripción de sesiones (SDP)](https://en.wikipedia.org/wiki/Session_Description_Protocol).

### <a name="inference-event"></a>Evento de inferencia

El nodo del procesador de extensiones HTTP recibe los resultados de inferencia del contenedor de Custom Vision y los emite a través del nodo receptor de IoT Hub como eventos de inferencia.

```
{
  "body": {
    "created": "2020-05-18T01:08:34.483Z",
    "id": "",
    "iteration": "",
    "predictions": [
      {
        "boundingBox": {
          "height": 0.06219418,
          "left": 0.14977954,
          "top": 0.65847444,
          "width": 0.01879117
        },
        "probability": 0.69806677,
        "tagId": 0,
        "tagName": "delivery truck"
      },
      {
        "boundingBox": {
          "height": 0.1148454,
          "left": 0.77430711,
          "top": 0.54488315,
          "width": 0.11315252
        },
        "probability": 0.29394844,
        "tagId": 0,
        "tagName": "delivery truck"
      },
      {
        "boundingBox": {
          "height": 0.03187029,
          "left": 0.62644471,
          "top": 0.59640052,
          "width": 0.01582896
        },
        "probability": 0.14435098,
        "tagId": 0,
        "tagName": "delivery truck"
      },
      {
        "boundingBox": {
          "height": 0.11421723,
          "left": 0.72737214,
          "top": 0.55907888,
          "width": 0.12627538
        },
        "probability": 0.1141128,
        "tagId": 0,
        "tagName": "delivery truck"
      },
      {
        "boundingBox": {
          "height": 0.04717135,
          "left": 0.66516746,
          "top": 0.34617995,
          "width": 0.03802613
        },
        "probability": 0.10461298,
        "tagId": 0,
        "tagName": "delivery truck"
      },
      {
        "boundingBox": {
          "height": 0.20650843,
          "left": 0.56349564,
          "top": 0.51482571,
          "width": 0.35045707
        },
        "probability": 0.10056595,
        "tagId": 0,
        "tagName": "delivery truck"
      }
    ],
    "project": ""
  },
  "applicationProperties": {
    "topic": "/subscriptions/35c2594a-23da-4fce-b59c-f6fb9513eeeb/resourceGroups/lsravi/providers/microsoft.media/mediaservices/lvasamplerc3he6a7uhire",
    "subject": "/graphInstances/GRAPHINSTANCENAMEHERE/processors/httpExtension",
    "eventType": "Microsoft.Media.Graph.Analytics.Inference",
    "eventTime": "2020-05-18T01:08:34.038Z"
  }
}
```

Tenga en cuenta la siguiente información en los mensajes anteriores:

* El sujeto de `applicationProperties` hace referencia al nodo del grafo multimedia desde el que se generó el mensaje. En este caso, el mensaje se origina en el procesador de extensiones HTTP.
* El tipo de evento de `applicationProperties` indica que se trata de un evento de inferencia analítica.
* La hora del evento indica la hora a la que se produjo el evento.
* El cuerpo contiene datos sobre el evento de análisis. En este caso, el evento es un evento de inferencia y, por lo tanto, el cuerpo contiene una matriz de inferencias llamada predictions.
* La sección predictions contiene una lista de las predicciones en las que se encuentra el camión de entrega de juguete (la etiqueta es "camión de entrega") en el fotograma. Como recordatorio, "camión de entrega" es la etiqueta personalizada que proporcionó a su modelo entrenado personalizado para el camión de juguete. El modelo infiere e identifica el camión de juguete en el vídeo de entrada con diferentes puntuaciones de confianza de probabilidad.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a probar los demás tutoriales o inicios rápidos, conserve los recursos creados. En caso contrario, vaya a Azure Portal, vaya a los grupos de recursos, seleccione el grupo de recursos en el que ejecutó este tutorial y elimine todos los recursos.

## <a name="next-steps"></a>Pasos siguientes

Revise los desafíos adicionales para los usuarios avanzados:

* Use una [cámara IP](https://en.wikipedia.org/wiki/IP_camera) que sea compatible con RTSP, en lugar de utilizar el simulador RTSP. Puede buscar cámaras IP compatibles con RTSP en la página de [productos compatibles con ONVIF](https://www.onvif.org/conformant-products/). Busque dispositivos que se ajusten a los perfiles G, S o T.
* Use un dispositivo Linux AMD64 o x64, en lugar de una máquina virtual Linux de Azure. El dispositivo debe estar en la misma red que la cámara IP. Puede seguir las instrucciones que aparecen en [Instalación del entorno de ejecución de Azure IoT Edge en Linux](../../iot-edge/how-to-install-iot-edge.md).

Luego, registre el dispositivo en Azure IoT Hub, para lo que debe seguir las instrucciones que se encuentran en [Implementación del primer módulo IoT Edge en un dispositivo virtual Linux](../../iot-edge/quickstart-linux.md).
