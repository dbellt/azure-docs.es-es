---
title: Guía de inicio rápido para el envío de datos de telemetría del dispositivo a Azure IoT Hub (Python)
description: En esta guía de inicio rápido, usará el SDK de dispositivo de Azure IoT Hub para Python para enviar datos de telemetría desde un dispositivo a un centro de IoT.
author: timlt
ms.author: timlt
ms.service: iot-develop
ms.devlang: python
ms.topic: quickstart
ms.date: 03/24/2021
ms.openlocfilehash: ea0b161a9038666e1e7ddd5a6c6af2078afff8aa
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107766539"
---
# <a name="quickstart-send-telemetry-from-a-device-to-an-azure-iot-hub-python"></a>Inicio rápido: Envío de datos de telemetría desde un dispositivo a un centro de Azure IoT (Python)

**Se aplica a**: [desarrollo de aplicaciones para dispositivos](about-iot-develop.md#device-application-development)

En esta guía de inicio rápido, aprenderá un flujo de trabajo básico de desarrollo de aplicaciones para dispositivos IoT. Use la CLI de Azure para crear una instancia de Azure IoT Hub y un dispositivo y, después, use el SDK de Azure IoT para Python para crear un dispositivo cliente simulado y enviar datos de telemetría al centro. 

## <a name="prerequisites"></a>Prerrequisitos
- Si no tiene una suscripción de Azure, [cree una gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de comenzar.
- CLI de Azure. Puede ejecutar todos los comandos de esta guía de inicio rápido mediante el Azure Cloud Shell, un shell interactivo de la CLI que se ejecuta en el explorador. Si usa el Cloud Shell, no necesita instalar nada. Si prefiere usar la CLI de forma local, esta guía de inicio rápido requiere CLI de Azure versión 2.0.76 o posterior. Para saber qué versión tiene, ejecute el comando az --version. Para la instalación o la actualización, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).
- [Python 3.7+](https://www.python.org/downloads/). Para obtener información acerca de otras versiones de Python compatibles, consulte el tema acerca de las [características de los dispositivos IoT de Azure](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device#azure-iot-device-features).
    
    Para asegurarse de que la versión de Python esta actualizada, ejecute `python --version`. Si tiene instalado tanto Python 2 como Python 3 y usa un entorno de Python 3, instale todas las bibliotecas con `pip3`. Este comando garantiza que las bibliotecas se instalan en el entorno de ejecución de Python 3.
    > [!IMPORTANT]
    > En el instalador de Python, seleccione la opción para **agregar Python a la ruta de acceso**. Si ya tiene Python 3.7 o una versión posterior instalada, confirme que ha agregado la carpeta de instalación de Python a la variable de entorno `PATH`.

[!INCLUDE [iot-hub-include-create-hub-cli](../../includes/iot-hub-include-create-hub-cli.md)]

## <a name="use-the-python-sdk-to-send-messages"></a>Uso del SDK para Python para enviar mensajes
En esta sección, usará el SDK para Python para enviar mensajes desde el dispositivo simulado al centro de IoT.

1. Abra una nueva ventana de terminal. Usará este terminal para instalar el SDK para Python y trabajar con el código de ejemplo de Python. Ahora debería tener dos terminales abiertos: el que acaba de abrir para trabajar con Python y el shell de la CLI que usó en las secciones anteriores para escribir los comandos de la CLI de Azure.       

1. Copie los [ejemplos para el SDK de dispositivo IoT de Azure para Python](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples) en la máquina local:

    ```console
    git clone https://github.com/Azure/azure-iot-sdk-python
    ```
1. Vaya al directorio *azure-iot-sdk-python/azure-iot-device/samples/pnp*:

    ```console
    cd azure-iot-sdk-python/azure-iot-device/samples/pnp
    ```
1. Instale el SDK de Azure IoT para Python:

    ```console
    pip install azure-iot-device
    ```
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

1. En el shell de la CLI abierto, ejecute el comando [az iot hub monitor-events](/cli/azure/ext/azure-iot/iot/hub#ext-azure-iot-az-iot-hub-monitor-events) para empezar a supervisar eventos en el dispositivo IoT simulado.  Los mensajes de los eventos se imprimen en el terminal cuando llegan.

    ```azurecli
    az iot hub monitor-events --output table --hub-name {YourIoTHubName}
    ```

1. En el terminal de Python, ejecute el código del archivo de ejemplo instalado *simple_thermostat.py*. Este código accede al dispositivo IoT simulado y envía un mensaje al centro de IoT.

    Para ejecutar el ejemplo de Python desde el terminal:
    ```console
    python ./simple_thermostat.py
    ```
    > [!NOTE]
    > En este ejemplo de código se usa Azure IoT Plug and Play, que le permite integrar dispositivos inteligentes en sus soluciones sin ninguna configuración manual.  De forma predeterminada, la mayoría de los ejemplos de esta documentación usan IoT Plug and Play. Para más información sobre las ventajas de IoT Plug and Play y los casos para su uso o no, consulte [¿Qué es IoT Plug and Play?](../iot-pnp/overview-iot-plug-and-play.md)

 Dado que el código de Python envía un mensaje desde el dispositivo al centro de IoT, el mensaje aparece en el shell de la CLI que supervisa los eventos:

```output
Starting event monitor, use ctrl-c to stop...
event:
  component: ''
  interface: dtmi:com:example:Thermostat;1
  module: ''
  origin: <your device name>
  payload:
    temperature: 35
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
En esta guía de inicio rápido, ha aprendido un flujo de trabajo básico de una aplicación de Azure IoT para conectar un dispositivo a la nube de forma segura y enviar datos de telemetría del dispositivo a la nube. Ha utilizado la CLI de Azure para crear un centro de IoT y un dispositivo y, después, ha utilizado el SDK de Azure IoT para Python para crear un dispositivo simulado y enviar datos de telemetría al centro. 

Como paso siguiente, explore el SDK de Azure IoT para Python mediante ejemplos de aplicaciones.

- [Ejemplos asincrónicos](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/async-hub-scenarios): este directorio contiene ejemplos asincrónicos de Python para escenarios de IoT Hub adicionales.
- [Ejemplos sincrónicos](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/sync-samples): este directorio contiene ejemplos de Python para su uso con Python 2.7 o escenarios de compatibilidad sincrónicos para Python 3.6+.
- [Ejemplos de IoT Edge](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/async-edge-scenarios): este directorio contiene ejemplos de Python para trabajar con módulos IoT Edge y dispositivos de bajada.
