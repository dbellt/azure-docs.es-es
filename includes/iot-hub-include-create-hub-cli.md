---
title: archivo de inclusión
description: archivo de inclusión
author: timlt
ms.author: timlt
ms.service: iot-develop
ms.topic: include
ms.date: 01/14/2021
ms.openlocfilehash: be51d4db6c98b63fcb1c9c60df045b494536d78f
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107766533"
---
En las secciones siguientes, configurará un terminal y usará la CLI de Azure para crear un centro de IoT. Para configurar un terminal que ejecute comandos de la CLI de Azure, puede usar una sesión de Azure Cloud Shell basada en el explorador, o bien un terminal local.
* Para usar Cloud Shell, vaya a la sección siguiente: [Inicio de Cloud Shell](#launch-the-cloud-shell). 
* Para usar un terminal local, omita la sección siguiente y vaya a [Apertura de un terminal local](#open-a-local-terminal).

## <a name="launch-the-cloud-shell"></a>Inicie el Cloud Shell
En esta sección, va a crear una sesión de Cloud Shell y a configurar el entorno de terminal.

Inicie sesión en Azure Portal en https://portal.azure.com.  

Para iniciar el Cloud Shell:

1. Seleccione el botón **Cloud Shell** en la barra de menú de la parte superior derecha del Azure Portal. 

    ![Azure Portal botón Cloud Shell](media/iot-hub-include-create-hub-cli/cloud-shell-button.png)

    > [!NOTE]
    > Si es la primera vez que usa el Cloud Shell, le pedirá que cree el almacenamiento, que es necesario para usar el Cloud Shell.  Seleccione una suscripción para crear una cuenta de almacenamiento y un recurso compartido de Microsoft Azure Files. 

2. Seleccione el entorno de CLI preferido en la lista desplegable **Seleccionar entorno**. Esta guía de inicio rápido usa el entorno de **Bash**. Los siguientes comandos de la CLI funcionan también en el entorno de PowerShell. 

    ![Seleccionar el entorno de CLI](media/iot-hub-include-create-hub-cli/cloud-shell-environment.png)

3. Omita la siguiente sección y vaya a [Instalación de la extensión de Azure IOT](#install-the-azure-iot-extension). 

## <a name="open-a-local-terminal"></a>Apertura de un terminal local
Si decide usar un terminal local en lugar de Cloud Shell, complete esta sección.  

1. Abra un terminal local.
1. Ejecute el comando [az login](/cli/azure/reference-index#az_login):

   ```azurecli
   az login
   ```

    Si la CLI puede abrir el explorador predeterminado, lo hará y cargará una página de inicio de sesión de Azure.

    En caso contrario, abra una página del explorador en https://aka.ms/devicelogin y escriba el código de autorización que se muestra en el terminal.

    Si no hay ningún explorador web disponible o no se puede abrir el explorador web, use el flujo de código del dispositivo con `az login --use-device-code`.

1. Inicie sesión con las credenciales de su cuenta en el explorador.

    Para más información acerca de los diferentes métodos de autenticación, consulte [Inicio de sesión con la CLI de Azure]( /cli/azure/authenticate-azure-cli ).

1. Vaya a la siguiente sección: [Instalación de la extensión de Azure IoT](#install-the-azure-iot-extension). 

## <a name="install-the-azure-iot-extension"></a>Instalación de la extensión de la Azure IoT
En esta sección, va a instalar la extensión de Microsoft Azure IoT para la CLI de Azure en el shell de la CLI. La extensión IOT agrega comandos específicos de IoT Hub, IoT Edge e IoT Device Provisioning Service (DPS) a la CLI de Azure.

> [!IMPORTANT]
> Los comandos de terminal del resto de este inicio rápido funcionan igual en Cloud Shell o en un terminal local. Para ejecutar un comando, seleccione **Copiar** para copiar un bloque de código en este inicio rápido. Péguelo en el shell de la CLI y ejecútelo.

Ejecute el comando [az extension add](/cli/azure/extension#az_extension_add). 

   ```azurecli
   az extension add --name azure-iot
   ```
[!INCLUDE [iot-hub-cli-version-info](iot-hub-cli-version-info.md)]

## <a name="create-an-iot-hub"></a>Crear un centro de IoT
En esta sección, usará la CLI de Azure para crear un centro de IoT y un grupo de recursos.  Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Un centro de IoT funciona como centro de mensajes unificado para la comunicación bidireccional entre la aplicación de IoT y los dispositivos. 

Para crear un centro de IoT y un grupo de recursos:

1. Ejecute el comando [az group create](/cli/azure/group#az_group_create) para crear un grupo de recursos. El siguiente comando crea un grupo de recursos denominado *MyResourceGroup* en la ubicación *eastus*. 
    >[!NOTE]
    > Como alternativa, puede establecer otra ubicación. Para ver las ubicaciones disponibles, ejecute `az account list-locations`. En este tutorial se usa *eastus* tal y como se muestra en el comando de ejemplo. 

    ```azurecli
    az group create --name MyResourceGroup --location eastus
    ```

1. Ejecute el comando [az iot hub create](/cli/azure/iot/hub#az_iot_hub_create) para crear una instancia de IoT Hub. La creación de una instancia de IoT Hub puede tardar unos minutos. 

    *YourIotHubName*. Reemplace este marcador de posición y las llaves circundantes en el siguiente comando, con el nombre que eligió para el centro de IoT. Un nombre de centro de IoT debe ser único globalmente en Azure. Use el nombre del centro de IoT a lo largo de este inicio rápido cuando vea el marcador de posición.

    ```azurecli
    az iot hub create --resource-group MyResourceGroup --name {YourIoTHubName}
    ```

## <a name="create-a-simulated-device"></a>Cree un dispositivo simulado:
En esta sección, creará un dispositivo IoT simulado que está conectado a su centro de IoT. 

Para crear un dispositivo simulado:
1. Ejecute el comando [az iot hub device-identity create](/cli/azure/ext/azure-iot/iot/hub/device-identity#ext-azure-iot-az-iot-hub-device-identity-create) en el shell de la CLI. Esto crea la identidad del dispositivo simulado. 

    *YourIotHubName*. reemplace este marcador de posición por el nombre elegido para el centro de IoT. 

    *myDevice*. Puede usar este nombre directamente para el identificador de dispositivo simulado a lo largo de este artículo. Si lo desea, use otro nombre. 

    ```azurecli
    az iot hub device-identity create --device-id myDevice --hub-name {YourIoTHubName} 
    ```

1.  Ejecute el comando [az iot hub device-identity connection-string show](/cli/azure/ext/azure-iot/iot/hub/device-identity/connection-string#ext_azure_iot_az_iot_hub_device_identity_connection_string_show). 

    ```azurecli
    az iot hub device-identity connection-string show --device-id myDevice --hub-name {YourIoTHubName}
    ```

    La cadena de conexión tiene el formato siguiente:

    ```Output
    HostName=<your IoT Hub name>.azure-devices.net;DeviceId=<your device id>;SharedAccessKey=<some value>
    ```

1. Guarde la cadena de conexión en una ubicación segura. 

> [!NOTE]
> Mantenga abierto el shell de la CLI. Lo usará en pasos posteriores.