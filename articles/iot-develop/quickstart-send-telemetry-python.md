---
title: Inicio rápido para el envío de datos de telemetría del dispositivo a Azure IoT Central (Python)
description: En esta guía de inicio rápido, usará el SDK de dispositivo de Azure IoT Hub para Python para enviar datos de telemetría desde un dispositivo a IoT Central.
author: timlt
ms.author: timlt
ms.service: iot-develop
ms.devlang: python
ms.topic: quickstart
ms.date: 04/27/2021
ms.openlocfilehash: 2464d9d4e6a945620bdbf80728aa3e7df0e2776e
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108226756"
---
# <a name="quickstart-send-telemetry-from-a-device-to-azure-iot-central-python"></a>Inicio rápido: Envío de datos de telemetría desde un dispositivo a Azure IoT Central (Python)

**Se aplica a**: [desarrolladores de aplicaciones para dispositivos](about-iot-develop.md#device-application-development)<br>
**Tiempo para realizarlo**: 12 minutos

En este inicio rápido, aprenderá un flujo de trabajo básico de desarrollo de aplicaciones de IoT. En primer lugar creará una aplicación en la nube para administrar dispositivos en Azure IoT Central. Después, usará el SDK de Python para Azure IoT para crear un dispositivo de termostato simulado, conectarlo a IoT Central y enviar datos de telemetría.

## <a name="prerequisites"></a>Requisitos previos
- [Python 3.7](https://www.python.org/downloads/) o versiones posteriores. Para comprobar la versión de Python, ejecute `python --version`. 

[!INCLUDE [iot-develop-create-central-app-with-device](../../includes/iot-develop-create-central-app-with-device.md)]

## <a name="configure-a-simulated-device"></a>Configuración de un dispositivo simulado
En esta sección, usará los ejemplos del SDK de Python para configurar un dispositivo de termostato simulado.

1. Abra un terminal mediante la línea de comandos de Windows, PowerShell o Bash (para Windows o Linux). Usará el terminal para instalar el SDK de Python, actualizar las variables de entorno y ejecutar el ejemplo de código de Python.

1. Copie los [ejemplos para el SDK de dispositivo IoT de Azure para Python](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples) en la máquina local.

    ```console
    git clone https://github.com/Azure/azure-iot-sdk-python
    ```

1. Vaya al directorio de ejemplos.

    ```console
    cd azure-iot-sdk-python/azure-iot-device/samples/pnp
    ```
1. Instale el SDK de Azure IoT para Python.
    ```console
    pip3 install azure-iot-device
    ```

1. Establezca las siguientes variables de entorno para que el dispositivo simulado se pueda conectar a IoT Central. Para `IOTHUB_DEVICE_DPS_ID_SCOPE`, `IOTHUB_DEVICE_DPS_DEVICE_KEY` y `IOTHUB_DEVICE_DPS_DEVICE_ID`, use los valores de conexión del dispositivo que guardó.

    **Línea de comandos de Windows**

    ```console
    set IOTHUB_DEVICE_SECURITY_TYPE=DPS
    set IOTHUB_DEVICE_DPS_ID_SCOPE=<application ID scope>
    set IOTHUB_DEVICE_DPS_DEVICE_KEY=<device primary key>
    set IOTHUB_DEVICE_DPS_DEVICE_ID=<your device ID>
    set IOTHUB_DEVICE_DPS_ENDPOINT=global.azure-devices-provisioning.net
    ```

    > [!NOTE]
    > En el caso del símbolo de comandos de Windows, no hay comillas alrededor de los valores de variable.

    **PowerShell**

    ```azurepowershell
    $env:IOTHUB_DEVICE_SECURITY_TYPE='DPS'
    $env:IOTHUB_DEVICE_DPS_ID_SCOPE='<application ID scope>'
    $env:IOTHUB_DEVICE_DPS_DEVICE_KEY='<device primary key>'
    $env:IOTHUB_DEVICE_DPS_DEVICE_ID='<your device ID>'
    $env:IOTHUB_DEVICE_DPS_ENDPOINT='global.azure-devices-provisioning.net'
    ```

    **Bash (Linux o Windows)**

    ```bash
    export IOTHUB_DEVICE_SECURITY_TYPE='DPS'
    export IOTHUB_DEVICE_DPS_ID_SCOPE='<application ID scope>'
    export IOTHUB_DEVICE_DPS_DEVICE_KEY='<device primary key>'
    export IOTHUB_DEVICE_DPS_DEVICE_ID='<your device ID>'
    export IOTHUB_DEVICE_DPS_ENDPOINT='global.azure-devices-provisioning.net' 
    ```

## <a name="send-telemetry"></a>Envío de datos de telemetría
Después de configurar el sistema, está listo para ejecutar el código. El código crea un termostato simulado, se conecta a la aplicación IoT Central y a la instancia del dispositivo, y envía datos de telemetría.

1. Ejecute el código de ejemplo siguiente en el terminal. Opcionalmente, puede ejecutar el código de ejemplo de Python desde el entorno de desarrollo integrado de Python:
    ```console
    python temp_controller_with_thermostats.py
    ```

    Una vez que el dispositivo simulado se conecta a la aplicación IoT Central, se conecta a la instancia de dispositivo que creó en la aplicación y comienza a enviar datos de telemetría. En la consola se muestran los detalles de conexión y la salida de telemetría: 
    
    ```output
    c:\azure-iot-sdk-python\azure-iot-device\samples\pnp>python temp_controller_with_thermostats.py
    Device was assigned
    iotc-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.azure-devices.net
    my-sdk-device
    Updating pnp properties for root interface
    {'serialNumber': 'alohomora'}
    Updating pnp properties for thermostat1
    {'thermostat1': {'maxTempSinceLastReboot': 98.34, '__t': 'c'}}
    Updating pnp properties for thermostat2
    {'thermostat2': {'maxTempSinceLastReboot': 48.92, '__t': 'c'}}
    Updating pnp properties for deviceInformation
    {'deviceInformation': {'swVersion': '5.5', 'manufacturer': 'Contoso Device Corporation', 'model': 'Contoso 4762B-turbo', 'osName': 'Mac Os', 'processorArchitecture': 'x86-64', 'processorManufacturer': 'Intel', 'totalStorage': 1024, 'totalMemory': 32, '__t': 'c'}}
    Listening for command requests and property updates
    Press Q to quit
    Sending telemetry from various components
    Sent message
    {"temperature": 33}
    ```

1. En IoT Central, seleccione **Dispositivos**, haga clic en el nombre del dispositivo y, a continuación, seleccione la pestaña **Datos sin procesar**. Esta vista muestra los datos de telemetría sin procesar del dispositivo simulado. 

    :::image type="content" source="media/quickstart-send-telemetry-python/iot-central-telemetry-output.png" alt-text="Salida de datos de telemetría sin procesar del dispositivo de IoT Central":::
    
    El dispositivo ya está conectado de forma segura y envía datos de telemetría a Azure IoT.
    
## <a name="clean-up-resources"></a>Limpieza de recursos
Si ya no necesita los recursos de IoT Central creados en este inicio rápido, puede eliminarlos. Opcionalmente, si planea seguir con la documentación de esta guía, puede conservar la aplicación que creó y reutilizarla para otros ejemplos.

Para quitar la aplicación de ejemplo de Azure IoT Central y todos sus dispositivos y recursos:
1. Seleccione **Administración**  > **Su aplicación**.
1. Seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha aprendido un flujo de trabajo básico de una aplicación de Azure IoT para conectar un dispositivo a la nube de forma segura y enviar datos de telemetría del dispositivo a la nube. Ha utilizado la Azure IoT Central para crear una aplicación y un dispositivo y, después, ha utilizado el SDK de Azure IoT para Python para crear un dispositivo simulado y enviar datos de telemetría al centro. También ha usado IoT Central para supervisar la telemetría.

Como paso siguiente, explore IoT Central como solución para hospedar los dispositivos y explore los ejemplos de código de Python para Azure SDK.

> [!div class="nextstepaction"]
> [Creación de una aplicación de IoT Central](../iot-central/core/quick-deploy-iot-central.md)
> [!div class="nextstepaction"]
> [Ejemplos de dispositivos asincrónicos](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/async-hub-scenarios)
> [!div class="nextstepaction"]
> [Ejemplos de dispositivos sincrónicos](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/sync-samples)
