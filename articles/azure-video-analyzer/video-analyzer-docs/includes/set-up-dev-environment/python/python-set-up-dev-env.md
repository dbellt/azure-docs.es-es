---
author: russell-cooks
ms.topic: include
ms.service: azure-video-analyzer
ms.date: 05/03/2021
ms.author: juliako
ms.openlocfilehash: 8e9992c6005c0bd3ba86bae005a1252a753e0c99
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110371923"
---
### <a name="get-the-sample-code"></a>Obtención del código de ejemplo

1. Clone el [repositorio de ejemplos de Python de AVA](https://github.com/Azure-Samples/video-analyzer-iot-edge-python).
1. Inicie Visual Studio Code y abra la carpeta en la que se descargó el repositorio.
1. En Visual Studio Code, vaya a la carpeta src/cloud-to-device-console-app y cree un archivo llamado **appsettings.json**. Este archivo contiene la configuración necesaria para ejecutar el programa.
1. Vaya al recurso compartido de archivos en la cuenta de almacenamiento creada en el paso de instalación anterior y busque el archivo **appsettings.json** en el recurso compartido de archivos "deployment-output". Haga clic en el archivo y, después, presione el botón "Descargar". El contenido debería abrirse en una nueva pestaña del explorador, que debería parecerse a esta:
   ```json
   {
     "IoThubConnectionString": "HostName=xxx.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=XXX",
     "deviceId": "avasample-iot-edge-device",
     "moduleId": "avaedge"
   }
   ```
   La cadena de conexión de IoT Hub permite usar Visual Studio Code para enviar comandos a los módulos IoT Edge mediante Azure IoT Hub. Copie el JSON anterior en el archivo **src/cloud-to-device-console-app/appsettings.json**.
1. A continuación, vaya a la carpeta src/edge y cree un archivo llamado **.env**. Este archivo contiene las propiedades que Visual Studio Code utiliza para implementar módulos en los dispositivos perimetrales.
1. Vaya al recurso compartido de archivos en la cuenta de almacenamiento creada en el paso de instalación anterior y busque el archivo **env.txt** en el recurso compartido de archivos "deployment-output". Haga clic en el archivo y, después, presione el botón "Descargar". El contenido debería abrirse en una nueva pestaña del explorador, que debería parecerse a esta:
   ```
        SUBSCRIPTION_ID="<Subscription ID>"
        RESOURCE_GROUP="<Resource Group>"
        AVA_PROVISIONING_TOKEN="<Provisioning token>"
        VIDEO_INPUT_FOLDER_ON_DEVICE="/home/localedgeuser/samples/input"
        VIDEO_OUTPUT_FOLDER_ON_DEVICE="/var/media"
        APPDATA_FOLDER_ON_DEVICE="/var/lib/videoAnalyzer"
        CONTAINER_REGISTRY_USERNAME_myacr="<your container registry username>"
        CONTAINER_REGISTRY_PASSWORD_myacr="<your container registry password>"
   ```
   Copie el JSON de **env.txt** en el archivo **src/edge/.env**.

### <a name="connect-to-the-iot-hub"></a>Conexión a IoT Hub

1.  En Visual Studio Code, establezca la cadena de conexión de IoT Hub. Para ello, seleccione el icono **Más acciones**, que se encuentra junto al panel **AZURE IOT HUB**, en la esquina inferior izquierda. Copie la cadena del archivo src/cloud-to-device-console-app/appsettings.json.

    <!-- commenting out the image for now ![Set IoT Hub connection string]()./media/quickstarts/set-iotconnection-string.png-->

    > [!NOTE]
    > Es posible que se le pida que proporcione información del punto de conexión integrado del centro de IoT. Para obtener esa información, en Azure Portal, vaya a su centro de IoT y busque la opción **Puntos de conexión integrados** en el panel de navegación izquierdo. Haga clic ahí y busque el **punto de conexión compatible con el centro de eventos** en la sección **Punto de conexión compatible con el centro de eventos**. Copie y use el texto del cuadro. El punto de conexión será similar a este: <br/>

    ```
    Endpoint=sb://iothub-ns-xxx.servicebus.windows.net/;SharedAccessKeyName=iothubowner;SharedAccessKey=XXX;EntityPath=<IoT Hub name>
    ```

1.  En unos 30 segundos, actualice Azure IoT Hub en la sección inferior izquierda. Debería ver el dispositivo perimetral `avasample-iot-edge-device`, que debe tener los siguientes módulos implementados:
    - Centro de Edge (nombre de módulo **edgeHub**)
    - Agente de Edge (nombre de módulo **edgeHub**)
    - Video Analyzer (nombre del módulo **avaedge**)
    - Simulador RTSP (nombre de módulo **rtspsim**)

### <a name="prepare-to-monitor-the-modules"></a>Preparación para supervisar los módulos

Cuando use o ejecute este inicio rápido o tutorial, los eventos se enviarán a IoT Hub. Para ver estos eventos, siga estos pasos:

1.  Abra el panel del explorador en Visual Studio Code y busque **Azure IoT Hub** en la esquina inferior izquierda.
1.  Expanda el nodo **Devices** (Dispositivos).
1.  Haga clic con el botón derecho en `avasample-iot-edge-device` y seleccione **Iniciar la supervisión del punto de conexión de eventos integrado**.

    > [!NOTE]
    > Es posible que se le pida que proporcione información del punto de conexión integrado del centro de IoT. Para obtener esa información, en Azure Portal, vaya a su centro de IoT y busque la opción **Puntos de conexión integrados** en el panel de navegación izquierdo. Haga clic ahí y busque el **punto de conexión compatible con el centro de eventos** en la sección **Punto de conexión compatible con el centro de eventos**. Copie y use el texto del cuadro. El punto de conexión será similar a este:

    ```
    Endpoint=sb://iothub-ns-xxx.servicebus.windows.net/;SharedAccessKeyName=iothubowner;SharedAccessKey=XXX;EntityPath=<IoT Hub name>
    ```
