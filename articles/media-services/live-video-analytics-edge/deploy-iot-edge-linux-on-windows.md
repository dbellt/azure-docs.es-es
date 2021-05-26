---
title: Implementación en un dispositivo de IoT Edge para Linux en Windows con Live Video Analytics- Azure
description: En este artículo se proporcionan instrucciones sobre cómo realizar la implementación en un dispositivo IoT Edge para Linux en Windows con Live Video Analytics.
ms.topic: how-to
ms.date: 02/18/2021
ms.openlocfilehash: c504ecf2bb7405ee49d6e9ee94f069336924f5c0
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110378101"
---
# <a name="deploy-to-an-iot-edge-for-linux-on-windows-eflow-device-with-live-video-analytics"></a>Implementación en un dispositivo IoT Edge para Linux en Windows (EFLOW) con Live Video Analytics

[!INCLUDE [redirect to Azure Video Analyzer](./includes/redirect-video-analyzer.md)]

En este artículo, aprenderá a implementar Live Video Analytics en un dispositivo perimetral que tiene [IoT Edge para Linux en Windows (EFLOW)](../../iot-edge/iot-edge-for-linux-on-windows.md). Una vez que haya terminado de realizar los pasos que se indican en este documento, podrá ejecutar un [grafo de elementos multimedia](media-graph-concept.md) que detecte el movimiento en un vídeo y emita esos eventos a IoT Hub en la nube. Después, puede desactivar el grafo de elementos multimedia para escenarios avanzados y aportar la potencia de Live Video Analytics a su dispositivo de IoT Edge basado en Windows.

## <a name="prerequisites"></a>Requisitos previos 

* Una cuenta de Azure que tenga una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), en caso de que aún no lo haya hecho.

    > [!NOTE]
    > Necesitará una suscripción de Azure con permisos para crear entidades de servicio (el **rol de propietario** permite esto). Si no tiene los permisos adecuados, póngase en contacto con el administrador de la cuenta para que se los conceda.
* [Visual Studio Code](https://code.visualstudio.com/) en la máquina de desarrollo. Asegúrese de tener la [extensión Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* Lea [¿Qué es EFLOW?](../../iot-edge/iot-edge-for-linux-on-windows.md)

## <a name="deployment-steps"></a>Pasos de implementación

A continuación se describe el flujo general del documento y en cinco pasos sencillos estará listo para ejecutar Live Video Analytics en un dispositivo Windows con EFLOW:

:::image type="content" source="./media/deploy-iot-edge-linux-on-windows/eflow.png" alt-text="Diagrama de IoT Edge para Linux en Windows (EFLOW)":::

1. [Instale EFLOW](../../iot-edge/how-to-install-iot-edge-on-windows.md) en el dispositivo Windows. 

    1. Si usa su PC Windows, en la página de inicio de [Windows Admin Center](/windows-server/manage/windows-admin-center/overview), en la lista de conexiones, verá una conexión de host local que representa el equipo en el que se ejecuta Windows Admin Center. 
    1. Los servidores, equipos o clústeres adicionales que administre también aparecerán aquí.
    1. Puede usar Windows Admin Center para instalar y administrar Azure EFLOW tanto en el dispositivo local como en los dispositivos administrados remotos. En esta guía, la conexión de host local actúa como dispositivo de destino para la implementación de Azure IoT Edge para Linux en Windows. Por lo tanto, el host local también aparece como un dispositivo de IoT Edge.

    :::image type="content" source="./media/deploy-iot-edge-linux-on-windows/windows-admin-center.png" alt-text="Pasos de implementaciones: Windows Admin Center":::
1. Haga clic en el dispositivo de IoT Edge para conectarse y aparecerá una pestaña de información general y el shell de comandos. En la pestaña del shell de comandos puede emitir comandos para el dispositivo perimetral.
 
    :::image type="content" source="./media/deploy-iot-edge-linux-on-windows/azure-iot-edge-manager.png" alt-text="Pasos de implementaciones: Administrador de Azure IoT Edge":::
1. Vaya al shell de comandos y escriba el siguiente comando:
    
    ```bash
    bash -c "$(curl -sL https://aka.ms/lva-edge/prep_device)"
    ```

    El módulo de Live Video Analytics se ejecuta en el dispositivo perimetral con [cuentas de usuario locales](deploy-iot-edge-device.md#create-and-use-local-user-account-for-deployment) sin privilegios. Además, [necesita ciertas carpetas locales](deploy-iot-edge-device.md#granting-permissions-to-device-storage) para almacenar los datos de configuración de la aplicación. Por último, para esta guía paso a paso, se emplea un [simulador RTSP](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555) que retransmite una fuente de vídeo en tiempo real a un módulo de Live Video Analytics para su análisis. Este simulador toma como entrada archivos de vídeo grabados previamente desde un directorio de entrada. 
    
    El script prep-device usado anteriormente automatiza estas tareas, por lo que puede ejecutar un comando y hacer que todas las carpetas de entrada y configuración pertinentes, archivos de entrada de vídeo y cuentas de usuario con privilegios se creen sin problemas. Una vez que el comando finalice correctamente, debería ver las siguientes carpetas creadas en el dispositivo perimetral. 
    
    * `/home/lvaedgeuser/samples`
    * `/home/lvaedgeuser/samples/input`
    * `/var/lib/azuremediaservices`
    * `/var/media`
    
    Observe los archivos de vídeo (*.mkv) de la carpeta /home/lvaedgeuser/samples/input, ya que le servirán como los archivos de entrada que se van a analizar. 
    
    :::image type="content" source="./media/deploy-iot-edge-linux-on-windows/azure-iot-edge-manager-analysis.png" alt-text="Análisis":::
1. Ahora que tiene el dispositivo perimetral configurado, registrado en el centro de conectividad y ejecutándose correctamente con las estructuras de carpetas correctas creadas, el paso siguiente consiste en configurar los siguientes recursos adicionales de Azure e implementar el módulo de Live Video Analytics. 

    * Cuenta de almacenamiento
    * Cuenta de Azure Media Services

    En este caso, se recomienda usar el [script de configuración de recursos de Live Video Analytics](https://github.com/Azure/live-video-analytics/tree/master/edge/setup) para implementar los recursos necesarios en su suscripción de Azure. Para hacerlo, siga estos pasos:

    1. Abra [Azure Cloud Shell](https://ms.portal.azure.com/#cloudshell/).

        :::image type="content" source="./media/deploy-iot-edge-linux-on-windows/azure-resources-cloud-shell.png" alt-text="Cloud Shell":::
    1. Si es la primera vez que usa Cloud Shell, se le pedirá que seleccione una suscripción para crear una cuenta de almacenamiento y un recurso compartido de Microsoft Azure Files. Seleccione **Create storage** (Crear almacenamiento) para crear una cuenta de almacenamiento para la información de la sesión de Cloud Shell. Esta cuenta de almacenamiento es independiente de la que creará el script para usarla con su cuenta de Azure Media Services.
    1. En el menú desplegable del lado izquierdo de la ventana de Cloud Shell, seleccione el entorno Bash.

        :::image type="content" source="./media/deploy-iot-edge-linux-on-windows/bash.png" alt-text="Entorno de Bash":::
    1. Ejecute el comando siguiente.

        ```bash
        bash -c "$(curl -sL https://aka.ms/lva-edge/setup-resources-for-samples)"
        ```
        
        Asegúrese de que elige **Y** (Sí) cuando se le pida que elija su propio dispositivo perimetral como dispositivo de IoT Edge, ya que creó anteriormente el dispositivo y la instancia de IoT Hub. También se le pedirá el nombre de la instancia de IoT Hub y el identificador del dispositivo de IoT Edge. Para obtenerlos, inicie sesión en Azure Portal y haga clic en la instancia de IoT Hub y, después, vaya a la opción IoT Edge en la hoja del portal de IoT Hub.

        :::image type="content" source="./media/deploy-iot-edge-linux-on-windows/iot-edge-devices.png" alt-text="Consultar dispositivos de IoT Edge":::

    Una vez finalizado, puede volver a iniciar sesión en el shell de comandos del dispositivo de IoT Edge y ejecutar el siguiente comando.
    
    `sudo iotedge list`
    
    Aparecerán los cuatro módulos siguientes implementados y en ejecución en el dispositivo perimetral. Tenga en cuenta que el script de creación de recursos implementa el módulo de Live Video Analytics junto con los módulos de IoT Edge (edgeAgent y edgeHub) y un módulo de simulador RTSP para proporcionar la fuente de vídeo RTSP simulada.
    
    :::image type="content" source="./media/deploy-iot-edge-linux-on-windows/running.png" alt-text="Consultar estado":::
1. Con los módulos implementados y configurados, está listo para ejecutar el primer grafo de elementos multimedia de Live Video Analytics en EFLOW. Ejecute los siguientes pasos para ejecutar un grafo de detección de movimiento simple como se muestra a continuación y visualizar los resultados:

    :::image type="content" source="./media/analyze-live-video/motion-detection.svg" alt-text="Grafo de detección de movimiento":::

    1. [Configure](get-started-detect-motion-emit-events-quickstart.md#configure-the-azure-iot-tools-extension) la extensión Azure IoT Tools.
    
        > [!Note]
        > Use la siguiente ruta de acceso: `~/clouddrive/lva_byod/appsettings.json. - instead of ~/clouddrive/lva-sample/appsettings.json`.
    1. Establezca la topología, cree instancias de un grafo y actívelas mediante estas [llamadas de método directo](get-started-detect-motion-emit-events-quickstart.md#use-direct-method-calls).
    1. [Observe los resultados](get-started-detect-motion-emit-events-quickstart.md#observe-results) en el centro de conectividad.
    1. Invoque [métodos de limpieza](get-started-detect-motion-emit-events-quickstart.md#invoke-graphinstancedeactivate).
    1. Elimine los recursos si ya no los necesita.

        > [!IMPORTANT]
        > Los recursos sin eliminar pueden seguir activos e incurrir en costos de Azure.
    
## <a name="next-steps"></a>Pasos siguientes

* Pruebe la detección de movimiento junto con la grabación de vídeos importantes en la nube. Siga los pasos que se indican en el inicio rápido [Detección de movimiento y grabación de clips de vídeo en Media Services](detect-motion-record-video-clips-media-services-quickstart.md#review-the-sample-video).
* Ejecute [IA en Live Video](use-your-model-quickstart.md#overview) (puede omitir la configuración de requisitos previos, ya que esta se ha hecho anteriormente)
* Use la [extensión VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.live-video-analytics-edge) para ver grafos multimedia adicionales.
* Use una [cámara IP](https://en.wikipedia.org/wiki/IP_camera) compatible con RTSP en lugar de utilizar el simulador RTSP. Puede encontrar cámaras IP compatibles con RTSP en la página de [productos compatibles con ONVIF](https://www.onvif.org/conformant-products/). Busque dispositivos que cumplan con los perfiles G, S o T.
