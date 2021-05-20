---
title: archivo de inclusión
description: archivo de inclusión
author: timlt
ms.author: timlt
ms.service: iot-develop
ms.topic: include
ms.date: 01/14/2021
ms.openlocfilehash: a1e2833d49d519d563937783a64db6573414bad2
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109805515"
---
## <a name="create-an-iot-hub"></a>Crear un centro de IoT
En esta sección, usará la CLI de Azure para crear un centro de IoT y un grupo de recursos.  Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Un centro de IoT funciona como centro de mensajes unificado para la comunicación bidireccional entre la aplicación de IoT y los dispositivos.

Para crear un centro de IoT y un grupo de recursos:

1. Inicie la aplicación de la CLI.  Para ejecutar los comandos de la CLI en el resto de este inicio rápido, copie la sintaxis del comando, péguela en la aplicación de la CLI, edite los valores de las variables y presione Entrar.
    - Si prefiere usar Cloud Shell, puede seleccionar el botón **Probar** en los comandos de la CLI para iniciar Cloud Shell en una ventana dividida del explorador. O bien, para abrir Cloud Shell en otra ventana, haga clic con el botón derecho en el vínculo de [Cloud Shell](https://shell.azure.com/bash) y seleccione la opción para abrirlo en una nueva pestaña.
    - Si usa la CLI de Azure localmente, inicie la aplicación de consola de la CLI e inicie sesión en la CLI de Azure.

1. En la aplicación de la CLI, ejecute el comando [az group create](/cli/azure/group#az_group_create) para crear un grupo de recursos. El siguiente comando crea un grupo de recursos denominado *MyResourceGroup* en la ubicación *eastus*. 
    >[!NOTE]
    > Como alternativa, puede establecer otra ubicación. Para ver las ubicaciones disponibles, ejecute `az account list-locations`. En este tutorial se usa *eastus* tal y como se muestra en el comando de ejemplo. 

    ```azurecli-interactive
    az group create --name MyResourceGroup --location eastus
    ```

1. Ejecute el comando [az iot hub create](/cli/azure/iot/hub#az_iot_hub_create) para crear una instancia de IoT Hub. La creación de una instancia de IoT Hub puede tardar unos minutos. 

    *YourIotHubName*. Reemplace este marcador de posición y las llaves circundantes en el siguiente comando, con el nombre que eligió para el centro de IoT. Un nombre de centro de IoT debe ser único globalmente en Azure. Use el nombre del centro de IoT a lo largo de este inicio rápido cuando vea el marcador de posición.

    ```azurecli-interactive
    az iot hub create --resource-group MyResourceGroup --name {YourIoTHubName}
    ```
    > [!TIP]
    > Después de crear un centro de IoT, en el resto de este inicio rápido puede seguir usando comandos de la CLI para interactuar con el centro. También puede usar Azure IoT Explorer en lugar de los comandos de la CLI. IoT Explorer es una aplicación de GUI que le permite conectarse a un dispositivo IoT Hub existente y agregar, administrar y supervisar dispositivos. Para obtener más información, consulte [Instalación y uso del explorador de Azure IoT](../articles/iot-pnp/howto-use-iot-explorer.md).

## <a name="create-a-simulated-device"></a>Cree un dispositivo simulado:
En esta sección, creará un dispositivo IoT simulado que está conectado a su centro de IoT. 

Para crear un dispositivo simulado:
1. Ejecute el comando [az iot hub device-identity create](/cli/azure/iot/hub/device-identity#az_iot_hub_device_identity_create) en el shell de la CLI. Esto crea la identidad del dispositivo simulado. 

    *YourIotHubName*. reemplace este marcador de posición por el nombre elegido para el centro de IoT. 

    *myDevice*. Puede usar este nombre directamente para el identificador de dispositivo simulado a lo largo de este artículo. Si lo desea, use otro nombre. 

    ```azurecli-interactive
    az iot hub device-identity create --device-id myDevice --hub-name {YourIoTHubName} 
    ```

1.  Ejecute el comando [az iot hub device-identity connection-string show](/cli/azure/iot/hub/device-identity/connection-string#az_iot_hub_device_identity_connection_string_show). 

    ```azurecli-interactive
    az iot hub device-identity connection-string show --device-id myDevice --hub-name {YourIoTHubName}
    ```

    La cadena de conexión tiene el formato siguiente:

    ```Output
    HostName=<your IoT Hub name>.azure-devices.net;DeviceId=<your device id>;SharedAccessKey=<some value>
    ```

1. Guarde la cadena de conexión en una ubicación segura. 

> [!NOTE]
> Mantenga abierta la aplicación de la CLI. Lo usará en pasos posteriores.