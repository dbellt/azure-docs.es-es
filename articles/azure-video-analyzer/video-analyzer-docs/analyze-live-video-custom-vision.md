---
title: Introducción a Azure Video Analyzer
description: Este tutorial le guía por los pasos necesarios para analizar vídeo en directo con Azure Video Analyzer en IoT Edge y Azure Custom Vision.
ms.topic: tutorial
ms.date: 04/21/2021
zone_pivot_groups: video-analyzer-programming-languages
ms.openlocfilehash: 3ba8fe19b08a17e2d2e35cfc34cda3a65795dea5
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110384137"
---
# <a name="tutorial-analyze-live-video-with-azure-video-analyzer-on-iot-edge-and-azure-custom-vision"></a>Tutorial: Análisis de vídeo en directo con Azure Video Analyzer en IoT Edge y Azure Custom Vision

En este tutorial, aprenderá a usar Azure [Custom Vision](https://azure.microsoft.com/services/cognitive-services/custom-vision-service/) para crear un modelo en contenedor que pueda detectar un camión de juguete y usar la [funcionalidad de extensibilidad de IA](analyze-live-video-without-recording.md#analyzing-video-using-a-custom-vision-model) de Azure Video Analyzer en Azure IoT Edge para implementar el modelo en el perímetro y detectar camiones de juguete a partir de una secuencia de vídeo en directo.

Mostraremos cómo reunir el potencial de Custom Vision para crear y entrenar un modelo de Computer Vision mediante la carga y el etiquetado de algunas imágenes. No son necesarios conocimientos de ciencia de datos, aprendizaje automático o inteligencia artificial. También aprenderá sobre las funcionalidades de Video Analyzer para implementar fácilmente un modelo personalizado como un contenedor en el perímetro y analizar una fuente de vídeo en directo simulada.

::: zone pivot="programming-language-csharp"
[!INCLUDE [header](includes/analyze-live-video-custom-vision/csharp/header.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [header](includes/analyze-live-video-custom-vision/python/header.md)]
::: zone-end

En este tutorial se muestra cómo realizar las siguientes acciones:

- Configuración de los recursos correspondientes
- Creación de un modelo de Custom Vision en la nube para detectar camiones de juguete y su implementación en el perímetro.
- Creación e implementación de una canalización con una extensión HTTP en un modelo de Custom Vision.
- Ejecución del código de ejemplo
- Examen e interpretación de los resultados

Si no tiene una [suscripción a Azure](../../guides/developer/azure-developer-guide.md#understanding-accounts-subscriptions-and-billing), cree una [cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) antes de empezar.

## <a name="suggested-pre-reading"></a>Sugerencias antes de la lectura

Consulte los artículos siguientes antes de empezar:

- [Introducción a Video Analyzer en IoT Edge](overview.md)
- [Introducción a Azure Custom Vision](../../cognitive-services/custom-vision-service/overview.md)
- [Terminología de Video Analyzer en IoT Edge](terminology.md)
- [Concepto de canalización](pipeline.md)
- [Video Analyzer sin grabación de vídeo](analyze-live-video-without-recording.md)
- [Tutorial: Desarrollo de un módulo IoT Edge](../../iot-edge/tutorial-develop-for-linux.md)
- [Cómo editar deployment.*.template.json](https://github.com/microsoft/vscode-azure-iot-edge/wiki/How-to-edit-deployment.*.template.json)

## <a name="prerequisites"></a>Requisitos previos

* [Instalar Docker](https://docs.docker.com/desktop/#download-and-install) en la máquina.

::: zone pivot="programming-language-csharp"
[!INCLUDE [prerequisites](includes/analyze-live-video-custom-vision/csharp/prerequisites.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [prerequisites](includes/analyze-live-video-custom-vision/python/prerequisites.md)]
::: zone-end


## <a name="review-the-sample-video"></a>Revisión del vídeo de ejemplo

En este tutorial se usa un archivo de [vídeo de inferencia de coches de juguete](https://lvamedia.blob.core.windows.net/public/t2.mkv) para simular streaming en vivo. Puede examinar el vídeo mediante una aplicación como [VLC Media Player](https://www.videolan.org/vlc/). Seleccione **Ctrl+N** y, después, pegue un vínculo al [vídeo de inferencia de coches de juguete](https://lvamedia.blob.core.windows.net/public/t2.mkv) para iniciar la reproducción. Cuando vea el vídeo, observe que, en el marcador de 36 segundos, aparece un camión de juguete. El modelo personalizado se ha entrenado para detectar este camión de juguete concreto.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4LPwK]

En este tutorial, usará Video Analyzer en IoT Edge para detectar esos camiones de juguete y publicar eventos de inferencia asociados en el centro de IoT Edge.

## <a name="overview"></a>Información general

![Diagrama que muestra una información general de Custom Vision.](./media/custom-vision/topology-custom-vision.svg)

En este diagrama se muestra el flujo de las señales en este tutorial. Un [módulo perimetral](https://github.com/Azure/video-analyzer/tree/main/edge-modules/sources/rtspsim-live555) simula una cámara IP que hospeda un servidor de protocolo RTSP. Un nodo de [origen RTSP](pipeline.md#rtsp-source) extrae la fuente de vídeo de este servidor y envía fotogramas de vídeo al nodo del [procesador de extensiones HTTP](pipeline.md#http-extension-processor).

El nodo de extensión HTTP desempeña el rol de un proxy. Se muestrean los fotogramas de vídeo entrantes que configuró mediante el campo `samplingOptions` y también se convierten los fotogramas de vídeo en el tipo de imagen especificado. A continuación, retransmite la imagen al modelo de detector de camiones de juguete creado mediante Custom Vision. El nodo del procesador de extensiones HTTP recopila los resultados de la detección y publica los eventos en el nodo [receptor de mensajes de Azure IoT Hub](pipeline.md#iot-hub-message-sink), el cual envía esos eventos al [centro de IoT Edge](../../iot-fundamentals/iot-glossary.md#iot-edge-hub).

## <a name="build-and-deploy-a-custom-vision-toy-detection-model&quot;></a>Creación e implementación de un modelo de detección de juguetes de Custom Vision

Custom Vision (que en español significa &quot;visión personalizada") permite crear, como su nombre indica, su propio detector o clasificador personalizado de objetos en la nube. Ofrece una interfaz sencilla, fácil de usar e intuitiva para crear modelos de Custom Vision que se pueden implementar en la nube o en el perímetro mediante contenedores.

Para crear un detector de camiones de juguete, siga los pasos descritos en [Inicio rápido: Creación de un detector de objetos con el sitio web de Custom Vision](../../cognitive-services/custom-vision-service/get-started-build-detector.md).

> [!IMPORTANT]
> Este módulo de Custom Vision solo admite arquitecturas **Intel x86 y amd64**. Compruebe la arquitectura del dispositivo perimetral antes de continuar.

Notas adicionales:

- En este tutorial, no use las imágenes de ejemplo que se proporcionan en la [sección de requisitos previos](../../cognitive-services/custom-vision-service/get-started-build-detector.md#prerequisites) del artículo de inicio rápido. En su lugar, hemos usado un conjunto de imágenes determinado para crear un modelo de Custom Vision de detector de juguetes. Use [estas imágenes](https://lvamedia.blob.core.windows.net/public/ToyCarTrainingImages.zip) cuando se le pida que [elija las imágenes de entrenamiento](../../cognitive-services/custom-vision-service/get-started-build-detector.md#choose-training-images) en el [inicio rápido](../../cognitive-services/custom-vision-service/get-started-build-detector.md).
- En la sección de etiquetado de imágenes del inicio rápido, asegúrese de que asigna al camión de juguete que se ve en la imagen la etiqueta "delivery truck" (camión de entrega).
- Asegúrese de seleccionar General (compacto) como opción en Dominios al crear el proyecto de Custom Vision.

Cuando haya terminado, puede exportar el modelo a un contenedor de Docker mediante el botón **Export** (Exportar) de la pestaña **Performance** (Rendimiento). Asegúrese de elegir Linux como tipo de plataforma de contenedor. Esta es la plataforma en la que se ejecutará el contenedor. Puede descargar el contenedor en una máquina Windows o Linux. En las instrucciones siguientes se supone que el archivo contenedor se ha descargado en una máquina Windows.

![Pantalla que muestra el archivo Dockerfile seleccionado.](./media/custom-vision/docker-file.png)

1. Debe tener un archivo zip descargado en la máquina local denominado `<projectname>.DockerFile.Linux.zip`.
2. Compruebe si tiene instalado Docker. En caso contrario, instale [Docker](https://docs.docker.com/get-docker/) para el escritorio de Windows.
3. Descomprima el archivo descargado en la ubicación que elija. Use la línea de comandos para ir al directorio de carpetas descomprimidas. Debería ver los dos archivos siguientes: app\labels.txt y app\model.pb
4. Clone el [repositorio de Video Analyzer](https://github.com/Azure/video-analyzer) y use la línea de comandos para ir a la carpeta edge-modules\extensions\customvision\avaextension.
5. Copie los archivos labels.txt y model.pb del paso 3 en la carpeta edge-modules\extensions\customvision\avaextension. En la misma carpeta:

   Ejecute los comandos siguientes:

   1. `docker build -t cvtruck .`

      Este comando descarga muchos paquetes, crea la imagen de Docker y la etiqueta como `cvtruck:latest`.

      > [!NOTE]
      > Si se realiza correctamente, debería ver los mensajes siguientes: `Successfully built <docker image id>` y `Successfully tagged cvtruck:latest`. Si se produce un error en el comando de creación, inténtelo de nuevo. A veces, los paquetes de dependencias no se descargan la primera vez.
   2. `docker image ls`

      Este comando comprueba si la nueva imagen está en el registro local.
   3. `docker run -p 127.0.0.1:80:80 -d cvtruck`

      Este comando debe publicar el puerto expuesto de Docker (80) en el puerto de la máquina local (80).
   4. `docker container ls`

      Este comando comprueba las asignaciones de puertos y si el contenedor de Docker se ejecuta correctamente en la máquina. La salida debe ser similar a la siguiente:
      
      ```
      CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
      8b7505398367        cvtruck             "/bin/sh -c 'python …"   13 hours ago        Up 25 seconds       127.0.0.1:80->80/tcp   practical_cohen
      ```
   5. `curl -X POST http://127.0.0.1:80/score -F imageData=@<path to any image file that has the toy delivery truck in it>`

      Este comando prueba el contenedor en el equipo local. Si la imagen tiene el mismo camión de entrega que se ha entrenado en el modelo, la salida debe ser similar a la del ejemplo siguiente. Sugiere que se detectó el camión de entrega con una probabilidad del 90,12 %.

      ```
      {"created":"2020-03-20T07:10:47.827673","id":"","iteration":"","predictions":[{"boundingBox":{"height":0.66167289,"left":-0.03923762,"top":0.12781593,"width":0.70003178},"probability":0.90128148,"tagId":0,"tagName":"delivery truck"},{"boundingBox":{"height":0.63733053,"left":0.25220079,"top":0.0876643,"width":0.53331227},"probability":0.59745145,"tagId":0,"tagName":"delivery truck"}],"project":""}
      ```

## <a name="examine-the-sample-files"></a>Examen de los archivos de ejemplo

::: zone pivot="programming-language-csharp"
[!INCLUDE [examine-sample-files](includes/analyze-live-video-custom-vision/csharp/examine-sample-files.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [examine-sample-files](includes/analyze-live-video-custom-vision/python/examine-sample-files.md)]
::: zone-end

## <a name="generate-and-deploy-the-deployment-manifest"></a>Generación e implementación del manifiesto de implementación

1. En Visual Studio Code, vaya a src/cloud-to-device-console-app/operations.json.
2. En `pipelineTopologySet`, asegúrese de que se cumple lo siguiente:<br/>
   `"pipelineTopologyUrl" : "https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/httpExtension/topology.json"`
3. En `livePipelineSet`, asegúrese de lo siguiente:

   1. `"topologyName" : "InferencingWithHttpExtension"`
   2. Agregue lo siguiente en la parte superior de la matriz de parámetros: `{"name": "inferencingUrl","value": "http://cv/score"},`
   3. Cambie el valor del parámetro `rtspUrl` a `"rtsp://rtspsim:554/media/t2.mkv"`.
4. En `livePipelineDelete`, asegúrese de que `"name": "InferencingWithHttpExtension"`.
5. Haga clic con el botón derecho en el archivo src/edge/deployment.customvision.template.json y seleccione **Generate IoT Edge Deployment Manifest** (Generar manifiesto de implementación de IoT Edge).

   ![Captura de pantalla que muestra la opción para generar un manifiesto de implementación de IoT Edge.](./media/custom-vision/deployment-template-json.png)

   Esta acción debe crear un archivo de manifiesto en la carpeta src/edge/config llamado deployment.customvision.amd64.json.
6. Abra el archivo src/edge/deployment.customvision.template.json y busque el bloque JSON `registryCredentials`. En este bloque, encontrará la dirección del registro de contenedor de Azure, junto con su nombre de usuario y contraseña.
7. Inserte el contenedor de Custom Vision local en la instancia de Azure Container Registry mediante estos pasos en la línea de comandos:

   1. Inicie sesión en el registro mediante la ejecución del comando siguiente:

      `docker login <address>`

      Escriba el nombre de usuario y la contraseña cuando se le solicite autenticación.

      > [!NOTE]
      > La contraseña no resulta visible en la línea de comandos.
   2. Etiquete la imagen con este comando:<br/>
      `docker tag cvtruck <address>/cvtruck`.
   3. Inserte la imagen con este comando:<br/>
      `docker push <address>/cvtruck`.

      Si se realiza correctamente, debería ver `Pushed` (Insertado) en la línea de comandos junto con el SHA de la imagen.
   4. También puede confirmarlo mediante la comprobación de la instancia de Azure Container Registry en Azure Portal. Aquí verá el nombre del repositorio junto con la etiqueta.
8. Establezca la cadena de conexión de IoT Hub; para ello, seleccione el icono **More actions** (Más acciones) situado junto al panel **AZURE IOT HUB** en la esquina inferior izquierda. Puede copiarla del archivo appsettings.json. Este es otro enfoque recomendado para asegurarse de que tiene el centro de IoT adecuado configurado en Visual Studio Code mediante el comando [Select IoT Hub](https://github.com/Microsoft/vscode-azure-iot-toolkit/wiki/Select-IoT-Hub) (Seleccionar centro de IoT).

   ![Captura de pantalla que muestra la cadena de conexión de Set IoT Hub.](./media/custom-vision/connection-string.png)
9. A continuación, haga clic con el botón derecho en src/edge/config/deployment.customvision.amd64.json y seleccione **Create Deployment for Single Device** (Crear una implementación para un dispositivo individual).

   ![Captura de pantalla que muestra la creación de una implementación para un dispositivo individual.](./media/custom-vision/deployment-amd64-json.png)
10. A continuación, se le pedirá que seleccione un dispositivo IoT Hub. Seleccione **ava-sample-iot-edge-device** en la lista desplegable.
11. En unos 30 segundos, actualice la instancia de Azure IoT Hub en la sección inferior izquierda. Debería ver que el dispositivo perimetral tiene los siguientes módulos implementados:

    - El módulo de Video Analyzer en IoT Edge llamado `avaedge`.
    - Un módulo llamado `rtspsim`, que simula un servidor RTSP y actúa como el origen de una fuente de vídeo en directo.
    - Un módulo llamado `cv` que, como su nombre sugiere, es el modelo de detección de camiones de juguete de Custom Vision que aplica Custom Vision a las imágenes y devuelve varios tipos de etiqueta. (Nuestro modelo se ha entrenado sobre una sola etiqueta: "camión de entrega").

## <a name="prepare-for-monitoring-events"></a>Preparación de la supervisión de eventos

Haga clic con el botón derecho en ava-sample-device y seleccione **Iniciar la supervisión del punto de conexión de eventos integrado**. Este paso es necesario para supervisar los eventos de IoT Hub en la ventana **SALIDA** de Visual Studio Code.

![Captura de pantalla que muestra la opción para iniciar la supervisión del punto de conexión de eventos integrado.](./media/custom-vision/start-monitoring.png)

## <a name="run-the-sample-program"></a>Ejecución del programa de ejemplo

Si abre la topología de este tutorial en un explorador, verá que el valor de `inferencingUrl` se ha establecido en `http://cv/image`. Esta configuración significa que el servidor de inferencia devolverá resultados después de detectar camiones de juguete, si los hay, en el vídeo en directo.

1. En Visual Studio Code, abra la pestaña **Extensiones** (o seleccione **Ctrl + Mayús + X**) y busque Azure IoT Hub.
2. Haga clic con el botón derecho y seleccione la opción **Configuración de la extensión**.

   ![Captura de pantalla que muestra la configuración de la extensión.](./media/custom-vision/extensions-tab.png)
3. Busque y habilite **Show Verbose Message** (Mostrar mensaje detallado).

   ![Captura de pantalla que muestra la opción para mostrar mensaje detallado.](./media/custom-vision/show-verbose-message.png)
4. Para iniciar una sesión de depuración, seleccione la tecla **F5**. Verá mensajes impresos en la ventana **TERMINAL**.
5. El código de operations.json comienza con llamadas a los métodos directos `livePipelineList` y `livePipelineList`. Si ha limpiado los recursos tras haber completado los inicios rápidos anteriores, este proceso devolverá listas vacías y, después, se pondrá en pausa. Para continuar, seleccione la tecla **Entrar**.

   La ventana **TERMINAL** muestra el siguiente conjunto de llamadas al método directo:

   - Una llamada a `pipelineTopologySet` que utiliza la instancia anterior de `pipelineTopologyUrl`.
   - Una llamada a `livePipelineSet` que usa el cuerpo siguiente:

   ```
        {
          "@apiVersion": "2.0",
          "name": "Sample-Pipeline-1",
          "properties": {
            "topologyName": "CustomVisionWithHttpExtension",
            "description": "Sample pipeline description",
            "parameters": [
              { 
                "name": "inferencingUrl",
                "value": "http://cv/image"
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

   - Una llamada a `livePipelineActivate` que activa la canalización y el flujo de vídeo.
   - Una segunda llamada a `livePipelineList` que muestra la canalización activa.

6. La salida de la ventana **TERMINAL** se pondrá en pausa con el mensaje **Press Enter to continue** (Presione Entrar para continuar). No seleccione **Entrar** todavía. Desplácese hacia arriba para ver las cargas de la respuesta JSON para los métodos directos que ha invocado.
7. Cambie a la ventana **SALIDA** de Visual Studio Code. Verá los mensajes que el módulo Video Analyzer en IoT Edge está enviando al centro de IoT. En la siguiente sección de este tutorial se analizan estos mensajes.
8. La canalización continúa ejecutándose y se imprimen los resultados. El simulador RTSP sigue recorriendo el vídeo de origen. Para detener la canalización, vuelva a la ventana **TERMINAL** y seleccione **Entrar**. La siguiente serie de llamadas limpia los recursos:

   - Una llamada a `livePipelineDeactivate` desactiva la canalización.
   - Una llamada a `livePipelineDelete` elimina la canalización.
   - Una llamada a `pipelineTopologyDelete` elimina la topología.
   - Una llamada final a `pipelineTopologyList` muestra que la lista está vacía.

## <a name="interpret-the-results"></a>Interpretación de los resultados

Cuando se ejecuta la canalización, los resultados del nodo procesador de extensiones HTTP recorren el nodo receptor de mensajes de IoT Hub hasta llegar al centro de IoT. Los mensajes que aparecen en la ventana **SALIDA** contienen una sección body y una sección `applicationProperties`. Para más información, consulte [Creación y lectura de mensajes de IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).

En los mensajes siguientes, el módulo de Video Analyzer define las propiedades de la aplicación y el contenido del cuerpo.

### <a name="mediasessionestablished-event"></a>Evento MediaSessionEstablished

Cuando se crea una instancia de la canalización, el nodo de origen RTSP intenta conectarse al servidor RTSP que se ejecuta en el contenedor rtspsim-live555. Si la conexión se realiza correctamente, se imprime el evento siguiente.


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

En este mensaje, tenga en cuenta estos detalles:

- El mensaje es un evento de diagnóstico. `MediaSessionEstablished` indica que el nodo de origen RTSP (el sujeto) ha conectado con el simulador RTSP y ha comenzado a recibir una fuente en directo simulada.
- En `properties`, `subject` indica que el mensaje se generó desde el nodo de origen RTSP de la canalización.
- En `properties`, el tipo de evento indica que este evento es un evento de diagnóstico.
- La hora del evento indica la hora a la que se produjo el evento.
- El cuerpo contiene datos sobre el evento de diagnóstico. En este caso, los datos incluyen detalles sobre el [protocolo de descripción de sesiones (SDP)](https://en.wikipedia.org/wiki/Session_Description_Protocol).

### <a name="inference-event"></a>Evento de inferencia

El nodo del procesador de extensiones HTTP recibe los resultados de inferencia del contenedor de Custom Vision y los emite a través del nodo receptor de mensajes de IoT Hub como eventos de inferencia.


```
{
  "body": {
    "timestamp": 145892470449324,
    "inferences": [
      {
        "type": "entity",
        "entity": {
          "tag": {
            "value": "delivery truck",
            "confidence": 0.20541823
          },
          "box": {
            "l": 0.6826309,
            "t": -0.01415127,
            "w": 0.3135161,
            "h": 0.94683206
          }
        }
      },
      {
        "type": "entity",
        "entity": {
          "tag": {
            "value": "delivery truck",
            "confidence": 0.14967085
          },
          "box": {
            "l": 0.33310884,
            "t": 0.03174839,
            "w": 0.13532706,
            "h": 0.54967254
          }
        }
      },
      {
        "type": "entity",
        "entity": {
          "tag": {
            "value": "delivery truck",
            "confidence": 0.1352181
          },
          "box": {
            "l": 0.48884687,
            "t": 0.44746214,
            "w": 0.025887,
            "h": 0.05414263
          }
        }
      }
    ]
  },
  "properties": {
    "topic": "/subscriptions/...",
    "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/processors/httpExtension",
    "eventType": "Microsoft.VideoAnalyzer.Analytics.Inference",
    "eventTime": "2021-05-14T21:24:09.436Z",
    "dataVersion": "1.0"
  },
  "systemProperties": {
    "iothub-connection-device-id": "avasample-iot-edge-device",
    "iothub-connection-module-id": "avaedge",
    "iothub-connection-auth-method": "{\"scope\":\"module\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
    "iothub-connection-auth-generation-id": "637563926153483223",
    "iothub-enqueuedtime": 1621027452077,
    "iothub-message-source": "Telemetry",
    "messageId": "96f7f0b5-728d-4e3e-a7bb-4e3198c58726",
    "contentType": "application/json",
    "contentEncoding": "utf-8"
  }
```

Tenga en cuenta la siguiente información en los mensajes anteriores:

- El sujeto de `properties` hace referencia al nodo de la canalización desde la que se generó el mensaje. En este caso, el mensaje se origina en el procesador de extensiones HTTP.
- El tipo de evento de `properties` indica que se trata de un evento de inferencia analítica.
- La hora del evento indica la hora a la que se produjo el evento.
- El cuerpo contiene datos sobre el evento de análisis. En este caso, el evento es un evento de inferencia y, por lo tanto, el cuerpo contiene una matriz de inferencias llamada predictions.
- La sección inferences contiene una lista de las predicciones en las que se encuentra el camión de entrega de juguete (la etiqueta es "camión de entrega") en el fotograma. Como recordatorio, "camión de entrega" es la etiqueta personalizada que proporcionó a su modelo entrenado personalizado para el camión de juguete. El modelo infiere e identifica el camión de juguete en el vídeo de entrada con diferentes puntuaciones de confianza de probabilidad.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a probar los demás tutoriales o inicios rápidos, conserve los recursos creados. En caso contrario, vaya a Azure Portal, vaya a los grupos de recursos, seleccione el grupo de recursos en el que ejecutó este tutorial y elimine todos los recursos.

## <a name="next-steps"></a>Pasos siguientes

Revise los desafíos adicionales para los usuarios avanzados:

- Use una [cámara IP](https://en.wikipedia.org/wiki/IP_camera) que sea compatible con RTSP, en lugar de utilizar el simulador RTSP. Puede buscar cámaras IP compatibles con RTSP en la página de [productos compatibles con ONVIF](https://www.onvif.org/conformant-products/). Busque dispositivos que se ajusten a los perfiles G, S o T.
- Use un dispositivo Linux AMD64 o x64, en lugar de una máquina virtual Linux de Azure. El dispositivo debe estar en la misma red que la cámara IP. Puede seguir las instrucciones que aparecen en [Instalación del entorno de ejecución de Azure IoT Edge en Linux](../../iot-edge/how-to-install-iot-edge.md).

Luego, registre el dispositivo en Azure IoT Hub, para lo que debe seguir las instrucciones que se encuentran en [Implementación del primer módulo IoT Edge en un dispositivo virtual Linux](../../iot-edge/quickstart-linux.md).  

