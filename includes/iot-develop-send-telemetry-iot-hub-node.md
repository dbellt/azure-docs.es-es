---
title: archivo de inclusión
description: archivo de inclusión
author: timlt
ms.service: iot-develop
ms.topic: include
ms.date: 05/05/2021
ms.author: timlt
ms.custom: include file
ms.openlocfilehash: 9544e662fde8f39303ea8bbafd7556d9e3b5af07
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109518320"
---
## <a name="prerequisites"></a>Prerrequisitos
- Si no tiene una suscripción de Azure, [cree una gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de comenzar.
- [Git](https://git-scm.com/downloads).
- [Node.js](https://nodejs.org), versión 10 o posterior. Para comprobar la versión de Node, ejecute `node --version`.
- CLI de Azure. Tiene dos opciones para ejecutar comandos de la CLI de Azure en este inicio rápido:
    - Use Azure Cloud Shell, un shell interactivo que ejecuta comandos de la CLI en el explorador. Esta opción se recomienda porque no es necesario instalar nada. Si usa Cloud Shell por primera vez, inicie sesión en [Azure Portal](https://portal.azure.com). Siga los pasos descritos en el [inicio rápido de Cloud Shell](/azure/cloud-shell/quickstart) para **iniciar Cloud Shell** y **seleccionar el entorno Bash**.
    - Opcionalmente, ejecute la CLI de Azure en su equipo local. El inicio rápido requiere la versión 2.0.76, o posterior, de la CLI de Azure. Ejecute `az --version` para comprobar la versión. Siga los pasos descritos en [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli) para instalar o actualizar la CLI de Azure, ejecutarla e iniciar sesión. Si se le solicita, instale las extensiones de la CLI de Azure la primera vez que la use.

[!INCLUDE [iot-hub-include-create-hub-cli](iot-hub-include-create-hub-cli.md)]

## <a name="run-a-simulated-device"></a>Ejecución de un dispositivo simulado
En esta sección, usará el SDK para Node.js para enviar mensajes desde el dispositivo simulado al centro de IoT. 

1. Abra una nueva ventana de la consola. Esta consola se usará para instalar el SDK para Node.js y trabajar con el código de ejemplo de Node.js. Ahora debería tener dos ventanas de consola abiertas: la que acaba de abrir y la consola Cloud Shell o CLI que usó anteriormente para escribir comandos de la CLI.

1. En la consola de Node, clone los [ejemplos para el SDK de dispositivo de Azure IoT para Node.js](https://github.com/Azure/azure-iot-sdk-node/tree/master/device/samples) en la máquina local:

    ```console
    git clone https://github.com/Azure/azure-iot-sdk-node
    ```

1. Vaya al directorio *azure-iot-sdk-node/device/samples/pnp*:

    ```console
    cd azure-iot-sdk-node/device/samples/pnp
    ```

1. Instale el SDK de Azure IoT para Node.js y las dependencias necesarias:

    ```console
    npm install
    ```

    Este comando instala las dependencias adecuadas, como se especifica en el archivo *package.json*, en el directorio de ejemplos de dispositivos.

1. Establezca las siguientes variables de entorno para que el dispositivo simulado se pueda conectar a Azure IoT.
    * Establezca una variable de entorno llamada `IOTHUB_DEVICE_CONNECTION_STRING`. Como valor de la variable, use la cadena de conexión del dispositivo que ha guardado en la sección anterior.
    * Establezca una variable de entorno llamada `IOTHUB_DEVICE_SECURITY_TYPE`. Para la variable, use el valor de cadena literal `connectionString`.

    **Windows (cmd)**

    ```console
    set IOTHUB_DEVICE_CONNECTION_STRING=<your connection string here>
    set IOTHUB_DEVICE_SECURITY_TYPE=connectionString
    ```

    > [!NOTE]
    > En el caso del símbolo de comandos de Windows, no hay comillas alrededor de los valores de cadena de cada variable.

    **PowerShell**

    ```azurepowershell
    $env:IOTHUB_DEVICE_CONNECTION_STRING='<your connection string here>'
    $env:IOTHUB_DEVICE_SECURITY_TYPE='connectionString'
    ```

    **Bash (Linux o Windows)**

    ```bash
    export IOTHUB_DEVICE_CONNECTION_STRING="<your connection string here>"
    export IOTHUB_DEVICE_SECURITY_TYPE="connectionString"
    ```
1. En la aplicación de la CLI, ejecute el comando [az iot hub monitor-events](/cli/azure/iot/hub#az_iot_hub_monitor_events) para empezar a supervisar eventos en el dispositivo IoT simulado.  Los mensajes de los eventos se imprimirán en el terminal cuando lleguen.

    ```azurecli-interactive
    az iot hub monitor-events --output table --hub-name {YourIoTHubName}
    ```

1. En la consola de Node, ejecute el código para el siguiente archivo de ejemplo. Este código accede al dispositivo IoT simulado y envía un mensaje al centro de IoT.

    Para ejecutar el ejemplo de Node.js desde el terminal:
    ```console
    node pnpTemperatureController.js
    ```
    > [!NOTE]
    > En este ejemplo de código se usa Azure IoT Plug and Play, que le permite integrar dispositivos inteligentes en sus soluciones sin ninguna configuración manual.  De forma predeterminada, la mayoría de los ejemplos de esta documentación usan IoT Plug and Play. Para más información sobre las ventajas de IoT Plug and Play y los casos para su uso o no, consulte [¿Qué es IoT Plug and Play?](../articles/iot-pnp/overview-iot-plug-and-play.md)

Dado que el código de Node.js envía un mensaje de telemetría simulada desde el dispositivo al centro de IoT, el mensaje aparece en la aplicación de la CLI que supervisa los eventos:

```output
Starting event monitor, use ctrl-c to stop...
event:
  component: thermostat1
  interface: dtmi:com:example:TemperatureController;2
  module: ''
  origin: myDevice
  payload:
    temperature: 70.5897683228018

event:
  component: thermostat2
  interface: dtmi:com:example:TemperatureController;2
  module: ''
  origin: myDevice
  payload:
    temperature: 52.87582619316418
```

El dispositivo ya está conectado de forma segura y envía datos de telemetría a Azure IoT Hub.