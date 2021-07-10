---
title: archivo de inclusión
description: archivo de inclusión
author: timlt
ms.service: iot-develop
ms.topic: include
ms.date: 05/05/2021
ms.author: timlt
ms.custom: include file
ms.openlocfilehash: 9baef8226ed44e7b11ce8108bacd2b686bee2d9d
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112041260"
---
## <a name="prerequisites"></a>Prerrequisitos
- Si no tiene una suscripción de Azure, [cree una gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de comenzar.
- [Git](https://git-scm.com/downloads).
- Una máquina de desarrollo con Java SE Development Kit 8 o posterior. Puede descargar el JDK de Java 8 (LTS) para varias plataformas desde [Descargar compilaciones de Zulu de OpenJDK](https://www.azul.com/downloads/zulu-community/). En el instalador, seleccione la opción **Agregar a PATH**.
- [Apache Maven 3](https://maven.apache.org/download.cgi). Después de extraer la descarga en una carpeta local, agregue la ruta de acceso completa de la carpeta */bin* de Maven a la variable PATH de Windows.
- CLI de Azure. Tiene dos opciones para ejecutar comandos de la CLI de Azure en este inicio rápido:
    - Use Azure Cloud Shell, un shell interactivo que ejecuta comandos de la CLI en el explorador. Esta opción se recomienda porque no es necesario instalar nada. Si usa Cloud Shell por primera vez, inicie sesión en [Azure Portal](https://portal.azure.com). Siga los pasos descritos en el [inicio rápido de Cloud Shell](../articles/cloud-shell/quickstart.md) para **iniciar Cloud Shell** y **seleccionar el entorno Bash**.
    - Opcionalmente, ejecute la CLI de Azure en su equipo local. Si la CLI de Azure ya está instalada, ejecute `az upgrade` para actualizar la CLI y las extensiones a la versión actual. Para instalar la CLI de Azure, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

[!INCLUDE [iot-hub-include-create-hub-cli](iot-hub-include-create-hub-cli.md)]

## <a name="run-a-simulated-device"></a>Ejecución de un dispositivo simulado
En esta sección, usará el SDK de Java para enviar mensajes desde el dispositivo simulado hasta el centro de IoT.

### <a name="configure-your-environment"></a>Configurar su entorno
1. Abra una consola para instalar el SDK para dispositivos Java de Azure IoT y compile y ejecute el ejemplo de código.
    > [!NOTE]
    > Ahora debería tener dos ventanas de consola abiertas: la que acaba de abrir y la consola Cloud Shell o CLI que usó anteriormente para escribir comandos de la CLI.
1. Establezca las siguientes variables de entorno para que el dispositivo simulado se pueda conectar a Azure IoT.
    * Establezca una variable de entorno llamada `IOTHUB_DEVICE_CONNECTION_STRING`. Como valor de la variable, use la cadena de conexión del dispositivo que ha guardado en la sección anterior.
    * Establezca una variable de entorno llamada `IOTHUB_DEVICE_SECURITY_TYPE`. Para la variable, use el valor de cadena literal `connectionString`.

    **CMD**

    ```console
    set IOTHUB_DEVICE_CONNECTION_STRING=<your connection string here>
    set IOTHUB_DEVICE_SECURITY_TYPE=connectionString
    ```
    > [!NOTE]
    > En el caso del símbolo de comandos de Windows, no hay comillas alrededor de los valores de cadena de cada variable.

    **Bash**

    ```bash
    export IOTHUB_DEVICE_CONNECTION_STRING="<your connection string here>"
    export IOTHUB_DEVICE_SECURITY_TYPE="connectionString"
    ```

### <a name="build-the-sample"></a>Compilación del ejemplo
1. En la consola, clone el SDK para dispositivos Java de Azure IoT en su máquina local:
    ```console
    git clone https://github.com/Azure/azure-iot-sdk-c.git
    ```
1. Vaya a la carpeta raíz del SDK y ejecute el siguiente comando para compilar el SDK y actualizar los ejemplos.
    ```console
    cd azure-iot-sdk-java
    mvn install -T 2C -DskipTests
    ```
    Esta operación tarda varios minutos.

### <a name="run-the-code"></a>Ejecución del código
1. En la aplicación de la CLI, ejecute el comando [az iot hub monitor-events](/cli/azure/iot/hub#az_iot_hub_monitor_events) para empezar a supervisar eventos en el dispositivo IoT simulado.  Los mensajes de los eventos se imprimen en el terminal cuando llegan.

    ```azurecli-interactive
    az iot hub monitor-events --output table --hub-name {YourIoTHubName}
    ```
1. En la consola de Java, vaya al directorio de ejemplos.
    ```console
    cd device/iot-device-samples/pnp-device-sample/temperature-controller-device-sample
    ```
1. En la consola de Java, ejecute el siguiente ejemplo de código. El ejemplo crea un controlador de temperatura simulado con sensores termostatos.
    ```console
    mvn exec:java -Dexec.mainClass="samples.com.microsoft.azure.sdk.iot.device.TemperatureController"
    ```
    > [!NOTE]
    > En este ejemplo de código se usa Azure IoT Plug and Play, que le permite integrar dispositivos inteligentes en sus soluciones sin ninguna configuración manual.  De forma predeterminada, la mayoría de los ejemplos de esta documentación usan IoT Plug and Play. Para más información sobre las ventajas de IoT Plug and Play y los casos para usarlos o no, consulte [¿Qué es IoT Plug and Play?](../articles/iot-pnp/overview-iot-plug-and-play.md)

    Después de que el dispositivo simulado se conecta al centro de IoT, se conecta a la instancia del dispositivo que creó en la aplicación y comienza a enviar datos de telemetría. Después de algunos detalles de aprovisionamiento iniciales, la consola comienza a generar la telemetría del controlador de temperatura.
    
    ```output
    2021-05-13 22:11:05.639 DEBUG TemperatureController:518 - Telemetry - Response from IoT Hub: message Id=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx, status=OK_EMPTY
    2021-05-13 22:11:10.229 INFO  IotHubTransport:540 - Message was queued to be sent later ( Message details: Correlation Id [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx] Message Id [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx] )
    2021-05-13 22:11:10.229 INFO  IotHubTransport:1473 - Sending message ( Message details: Correlation Id [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx] Message Id [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx] )
    2021-05-13 22:11:10.231 DEBUG TemperatureController:465 - Telemetry: Sent - {"temperature": 22.8░C} with message Id xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
    2021-05-13 22:11:10.234 INFO  IotHubTransport:540 - Message was queued to be sent later ( Message details: Correlation Id [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx] Message Id [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx] )
    2021-05-13 22:11:10.236 DEBUG TemperatureController:465 - Telemetry: Sent - {"temperature": 35.3░C} with message Id xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
    ```