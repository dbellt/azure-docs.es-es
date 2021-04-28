---
title: Guía de inicio rápido para el envío de datos de telemetría del dispositivo a Azure IoT Hub (Node.js)
description: En esta guía de inicio rápido, usará el SDK de dispositivo de Azure IoT Hub para Node.js para enviar datos de telemetría desde un dispositivo a un centro de IoT.
author: timlt
ms.author: timlt
ms.service: iot-develop
ms.devlang: node
ms.topic: quickstart
ms.date: 03/25/2021
ms.openlocfilehash: 0e1c99124228da9490abaa17ecc41b931631d9fb
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107876983"
---
# <a name="quickstart-send-telemetry-from-a-device-to-an-iot-hub-nodejs"></a>Inicio rápido: Envío de datos de telemetría desde un dispositivo a un centro de IoT (Node.js)

**Se aplica a**: [desarrollo de aplicaciones para dispositivos](about-iot-develop.md#device-application-development)

En esta guía de inicio rápido, aprenderá un flujo de trabajo básico de desarrollo de aplicaciones para dispositivos IoT. Use la CLI de Azure para crear una instancia de Azure IoT Hub y un dispositivo simulado y, después, use el SDK de Azure IoT para Node.js para acceder al dispositivo y enviar datos de telemetría al centro.

## <a name="prerequisites"></a>Prerrequisitos
- Si no tiene una suscripción de Azure, [cree una gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de comenzar.
- CLI de Azure. Puede ejecutar todos los comandos de esta guía de inicio rápido mediante el Azure Cloud Shell, un shell interactivo de la CLI que se ejecuta en el explorador. Si usa el Cloud Shell, no necesita instalar nada. Si prefiere usar la CLI de forma local, esta guía de inicio rápido requiere CLI de Azure versión 2.0.76 o posterior. Para saber qué versión tiene, ejecute el comando az --version. Para la instalación o la actualización, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).
- [Node.js 10+](https://nodejs.org). Si usa Azure Cloud Shell, no actualice la versión instalada de Node.js. Azure Cloud Shell ya tiene la versión más reciente de Node.js.

    Compruebe la versión actual de Node.js en la máquina de desarrollo con el comando siguiente:

    ```cmd/sh
        node --version
    ```

[!INCLUDE [iot-hub-include-create-hub-cli](../../includes/iot-hub-include-create-hub-cli.md)]

## <a name="use-the-nodejs-sdk-to-send-messages"></a>Uso del SDK para Node.js para enviar mensajes
En esta sección, usará el SDK para Node.js para enviar mensajes desde el dispositivo simulado al centro de IoT. 

1. Abra una nueva ventana de terminal. Usará este terminal para instalar el SDK para Node.js y trabajar con el código de ejemplo de Node.js. Ahora debería tener dos terminales abiertos: el que acaba de abrir para trabajar con Node.js y el shell de la CLI que usó en las secciones anteriores para escribir los comandos de la CLI de Azure.

1. Copie los [ejemplos para el SDK de dispositivo IoT de Azure para Node.js](https://github.com/Azure/azure-iot-sdk-node/tree/master/device/samples) en la máquina local:

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
    ```
    ```console
    set IOTHUB_DEVICE_SECURITY_TYPE=connectionString
    ```

    > [!NOTE]
    > En el caso del símbolo de comandos de Windows, no hay comillas alrededor de los valores de cadena de cada variable.

    **PowerShell**

    ```azurepowershell
    $env:IOTHUB_DEVICE_CONNECTION_STRING='<your connection string here>'
    ```
    ```azurepowershell
    $env:IOTHUB_DEVICE_SECURITY_TYPE='connectionString'
    ```

    **Bash (Linux o Windows)**

    ```bash
    export IOTHUB_DEVICE_CONNECTION_STRING="<your connection string here>"
    ```
    ```bash
    export IOTHUB_DEVICE_SECURITY_TYPE="connectionString"
    ```
1. En el shell de la CLI abierto, ejecute el comando [az iot hub monitor-events](/cli/azure/iot/hub#az_iot_hub_monitor_events) para empezar a supervisar eventos en el dispositivo IoT simulado.  Los mensajes de los eventos se imprimirán en el terminal cuando lleguen.

    ```azurecli
    az iot hub monitor-events --output table --hub-name {YourIoTHubName}
    ```

1. En el terminal de Node.js, ejecute el código del archivo de ejemplo instalado *simple_thermostat.js*. Este código accede al dispositivo IoT simulado y envía un mensaje al centro de IoT.

    Para ejecutar el ejemplo de Node.js desde el terminal:
    ```console
    node ./simple_thermostat.js
    ```
    > [!NOTE]
    > En este ejemplo de código se usa Azure IoT Plug and Play, que le permite integrar dispositivos inteligentes en sus soluciones sin ninguna configuración manual.  De forma predeterminada, la mayoría de los ejemplos de esta documentación usan IoT Plug and Play. Para más información sobre las ventajas de IoT Plug and Play y los casos para su uso o no, consulte [¿Qué es IoT Plug and Play?](../iot-pnp/overview-iot-plug-and-play.md)

Dado que el código de Node.js envía un mensaje de telemetría simulado desde el dispositivo al centro de IoT, el mensaje aparece en el shell de la CLI que supervisa los eventos:

```output
Starting event monitor, use ctrl-c to stop...
event:
  component: ''
  interface: dtmi:com:example:Thermostat;1
  module: ''
  origin: <your device ID>
  payload:
    temperature: 36.87027777131555
```

El dispositivo ya está conectado de forma segura y envía datos de telemetría a Azure IoT Hub.

## <a name="clean-up-resources"></a>Limpieza de recursos
Si ya no necesita los recursos de Azure creados en esta guía de inicio rápido, puede usar la CLI de Azure para eliminarlos.

> [!IMPORTANT]
> La eliminación de un grupo de recursos es irreversible. El grupo de recursos y todos los recursos contenidos en él se eliminan permanentemente. Asegúrese de no eliminar por accidente el grupo de recursos o los recursos equivocados. 

Para eliminar un grupo de recursos por el nombre:
1. Ejecute el comando [az group delete](/cli/azure/group#az_group_delete). Este comando quita el grupo de recursos, el centro de IoT y el registro del dispositivo que ha creado.

    ```azurecli
    az group delete --name MyResourceGroup
    ```
1. Ejecute el comando [az group list](/cli/azure/group#az_group_list) para confirmar que se ha eliminado el grupo de recursos.  

    ```azurecli
    az group list
    ```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha aprendido un flujo de trabajo básico de una aplicación de Azure IoT para conectar un dispositivo a la nube de forma segura y enviar datos de telemetría del dispositivo a la nube. Ha utilizado la CLI de Azure para crear un centro de IoT y un dispositivo simulado y, después, ha utilizado el SDK de Azure IoT para Node.js para acceder al dispositivo y enviar datos de telemetría al centro. 

Como paso siguiente, explore el SDK de Azure IoT para Node.js mediante ejemplos de aplicaciones.

- [Más ejemplos de Node.js](https://github.com/Azure/azure-iot-sdk-node/tree/master/device/samples): este directorio contiene más ejemplos del repositorio del SDK para Node.js que muestran escenarios de IoT Hub.