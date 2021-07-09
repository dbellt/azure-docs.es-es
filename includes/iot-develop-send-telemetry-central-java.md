---
title: archivo de inclusión
description: archivo de inclusión
author: timlt
ms.service: iot-develop
ms.topic: include
ms.date: 04/28/2021
ms.author: timlt
ms.custom: include file
ms.openlocfilehash: d9f4c0458cb0765706b110996e7c7df391ed2bf2
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110061160"
---
## <a name="prerequisites"></a>Requisitos previos
- Una máquina de desarrollo con Java SE Development Kit 8 o posterior. Puede descargar el JDK de Java 8 (LTS) para varias plataformas desde [Descargar compilaciones de Zulu de OpenJDK](https://www.azul.com/downloads/zulu-community/). En el instalador, seleccione la opción **Agregar a PATH**.
- [Apache Maven 3](https://maven.apache.org/download.cgi). Después de extraer la descarga en una carpeta local, agregue la ruta de acceso completa de la carpeta */bin* de Maven a la variable PATH de Windows.
- Una copia local del repositorio de GitHub [SDK de Microsoft Azure IoT para Java](https://github.com/Azure/azure-iot-sdk-java). Descargue una copia del repositorio y extráigala: [Descargar archivo ZIP](https://github.com/Azure/azure-iot-sdk-java/archive/refs/heads/master.zip).

[!INCLUDE [iot-develop-create-central-app-with-device](iot-develop-create-central-app-with-device.md)]

## <a name="run-a-simulated-device"></a>Ejecución de un dispositivo simulado
En esta sección, configurará el entorno local, instalará el SDK de dispositivo de Azure IoT para Java y ejecutará un ejemplo que crea un controlador de temperatura simulado.

### <a name="configure-your-environment"></a>Configurar su entorno

1. Abra una consola con el símbolo del sistema de Windows, PowerShell o Bash.

1. Establezca las siguientes variables de entorno con los comandos adecuados para la consola. El dispositivo simulado usa estos valores para conectarse a IoT Central. Para `IOTHUB_DEVICE_DPS_ID_SCOPE`, `IOTHUB_DEVICE_DPS_DEVICE_KEY` y `IOTHUB_DEVICE_DPS_DEVICE_ID`, use los valores de conexión del dispositivo que guardó anteriormente.

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

    **Bash**

    ```bash
    export IOTHUB_DEVICE_SECURITY_TYPE='DPS'
    export IOTHUB_DEVICE_DPS_ID_SCOPE='<application ID scope>'
    export IOTHUB_DEVICE_DPS_DEVICE_KEY='<device primary key>'
    export IOTHUB_DEVICE_DPS_DEVICE_ID='<your device ID>'
    export IOTHUB_DEVICE_DPS_ENDPOINT='global.azure-devices-provisioning.net' 
    ```

### <a name="build-and-run-the-code"></a>Compilación y ejecución del código

1. En Windows, vaya a la carpeta raíz del SDK de Azure para Java que descargó y ejecute el siguiente comando para compilar el ejemplo.

    ```console
    cd azure-iot-sdk-java
    mvn install -T 2C -DskipTests
    ```

1. Vaya al directorio de ejemplos.
    ```console
    cd device/iot-device-samples/pnp-device-sample/temperature-controller-device-sample
    ```

1. En la consola, ejecute el siguiente ejemplo de código. El ejemplo crea un controlador de temperatura simulado con sensores termostatos.
    ```console
    mvn exec:java -Dexec.mainClass="samples.com.microsoft.azure.sdk.iot.device.TemperatureController"
    ```

    Una vez que el dispositivo simulado se conecta a la aplicación IoT Central, se conecta a la instancia de dispositivo que creó en la aplicación y comienza a enviar datos de telemetría. Después de algunos detalles de aprovisionamiento iniciales, la consola comienza a generar la telemetría del controlador de temperatura.
    
    ```output
    2021-05-13 15:39:26.411 DEBUG Mqtt:253 - Sending MQTT SUBSCRIBE packet for topic $iothub/twin/res/#
    2021-05-13 15:39:26.428 INFO  IotHubTransport:540 - Message was queued to be sent later ( Message details: Correlation Id [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx] Message Id [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx] Request Id [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx] Device Operation Type [DEVICE_OPERATION_TWIN_UPDATE_REPORTED_PROPERTIES_REQUEST] )
    2021-05-13 15:39:26.432 DEBUG TemperatureController:427 - Property: Update - component = "deviceInformation" is COMPLETED.
    2021-05-13 15:39:26.436 INFO  IotHubTransport:540 - Message was queued to be sent later ( Message details: Correlation Id [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx] Message Id [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx] )
    2021-05-13 15:39:26.438 DEBUG TemperatureController:438 - Telemetry: Sent - {"workingSet": 1024.0KiB }
    2021-05-13 15:39:26.439 INFO  IotHubTransport:540 - Message was queued to be sent later ( Message details: Correlation Id [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx] Message Id [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx] Request Id [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx] Device Operation Type [DEVICE_OPERATION_TWIN_UPDATE_REPORTED_PROPERTIES_REQUEST] )
    2021-05-13 15:39:26.439 DEBUG TemperatureController:446 - Property: Update - {"serialNumber": SR-123456} is COMPLETED
    2021-05-13 15:39:26.447 INFO  IotHubTransport:540 - Message was queued to be sent later ( Message details: Correlation Id [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx] Message Id [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx] )
    2021-05-13 15:39:26.447 DEBUG TemperatureController:465 - Telemetry: Sent - {"temperature": 44.4░C} with message Id xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
    ```