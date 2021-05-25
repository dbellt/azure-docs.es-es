---
title: archivo de inclusión
description: archivo de inclusión
author: timlt
ms.service: iot-develop
ms.topic: include
ms.date: 05/05/2021
ms.author: timlt
ms.custom: include file
ms.openlocfilehash: e096e1259b6f58476d55cd2811ccdbb87d1fd46e
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109518317"
---
## <a name="prerequisites"></a>Prerrequisitos
- Si no tiene una suscripción de Azure, [cree una gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de comenzar.
- [Git](https://git-scm.com/downloads).
- CLI de Azure. Tiene dos opciones para ejecutar comandos de la CLI de Azure en este inicio rápido:
    - Use Azure Cloud Shell, un shell interactivo que ejecuta comandos de la CLI en el explorador. Esta opción se recomienda porque no es necesario instalar nada. Si usa Cloud Shell por primera vez, inicie sesión en [Azure Portal](https://portal.azure.com). Siga los pasos descritos en el [inicio rápido de Cloud Shell](/azure/cloud-shell/quickstart) para **iniciar Cloud Shell** y **seleccionar el entorno Bash**.
    - Opcionalmente, ejecute la CLI de Azure en su equipo local. El inicio rápido requiere la versión 2.0.76 o posterior de la CLI de Azure. Ejecute `az --version` para comprobar la versión. Siga los pasos descritos en [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli) para instalar o actualizar la CLI de Azure, ejecutarla e iniciar sesión. Si se le solicita, instale las extensiones de la CLI de Azure la primera vez que la use.

Instale los requisitos previos restantes para su sistema operativo.

### <a name="linux"></a>Linux
Los pasos de este tutorial se han probado con Ubuntu Linux 18.04.

Para completar este inicio rápido en Linux, es preciso instalar el siguiente software en el entorno Linux local:

Instale **GCC**, **Git**, **cmake** y las dependencias necesarias con el comando `apt-get`:

```sh
sudo apt-get update
sudo apt-get install -y git cmake build-essential curl libcurl4-openssl-dev libssl-dev uuid-dev
```

Compruebe que la versión de `cmake` es posterior a **2.8.12** y que la versión de **GCC** es posterior a **4.4.7**.

```sh
cmake --version
gcc --version
```

### <a name="windows"></a>Windows
Para completar este inicio rápido en Windows, instale Visual Studio 2019 y agregue los componentes necesarios para el desarrollo en C y C++.

1. Para los nuevos usuarios, instale [Visual Studio (Community, Professional o Enterprise) 2019](https://visualstudio.microsoft.com/downloads/). Descargue la edición que desea instalar e inicie el instalador.
    > [!NOTE]
    > Para los usuarios de Visual Studio 2019 existentes, seleccione **Inicio** de Windows, escriba *Instalador de Visual Studio* e inicie el instalador.
1. En la pestaña **Cargas de trabajo** del instalador, seleccione la carga de trabajo **Desarrollo para el escritorio con C++** .
1. Ejecución de la instalación.

[!INCLUDE [iot-hub-include-create-hub-cli](iot-hub-include-create-hub-cli.md)]

## <a name="run-a-simulated-device"></a>Ejecución de un dispositivo simulado
En esta sección, usará el SDK de C para enviar mensajes desde el dispositivo simulado al centro de IoT.

### <a name="build-the-sample"></a>Compilación del ejemplo
1. Abra una consola para instalar el SDK de dispositivo IoT de Azure para C y ejecute el ejemplo de código. En Windows, seleccione **Inicio**, escriba *Símbolo del sistema para desarrolladores para VS 2019* y abra la consola. En el caso de Linux, abra Bash.
    > [!NOTE]
    > Ahora debería tener dos ventanas de consola abiertas: la que acaba de abrir y la consola Cloud Shell o CLI que usó anteriormente para escribir comandos de la CLI.

1. En la consola de C, clone el SDK de dispositivo IoT de Azure para C en su equipo local:
    ```console
    git clone https://github.com/Azure/azure-iot-sdk-c.git
    ```
1. Navegue a la carpeta raíz del SDK y ejecute el siguiente comando para actualizar las dependencias:
    ```console
    cd azure-iot-sdk-c
    git submodule update --init
    ```
    Esta operación tarda unos minutos.
1. Ejecute los siguientes comandos para compilar el SDK y los ejemplos:

    ```console
    cmake -Bcmake -Duse_prov_client=ON -Dhsm_type_symm_key=ON -Drun_e2e_tests=OFF
    cmake --build cmake
    ```
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

### <a name="run-the-code"></a>Ejecución del código
1. En la aplicación de la CLI, ejecute el comando [az iot hub monitor-events](/cli/azure/iot/hub#az_iot_hub_monitor_events) para empezar a supervisar eventos en el dispositivo IoT simulado.  Los mensajes de los eventos se imprimen en el terminal cuando llegan.

    ```azurecli-interactive
    az iot hub monitor-events --output table --hub-name {YourIoTHubName}
    ```
1. Ejecute el código de ejemplo con el comando adecuado para la consola:

    **CMD**
    ```console
    cmake\iothub_client\samples\pnp\pnp_temperature_controller\Debug\pnp_temperature_controller.exe
    ```

    **Bash**
    ```bash
    cmake/iothub_client/samples/pnp/pnp_temperature_controller/Debug/pnp_temperature_controller
    ```
    > [!NOTE]
    > En este ejemplo de código se usa Azure IoT Plug and Play, que le permite integrar dispositivos inteligentes en sus soluciones sin ninguna configuración manual.  De forma predeterminada, la mayoría de los ejemplos de esta documentación usan IoT Plug and Play. Para más información sobre las ventajas de IoT Plug and Play y los casos para usarlos o no, consulte [¿Qué es IoT Plug and Play?](../articles/iot-pnp/overview-iot-plug-and-play.md)

    Una vez que el dispositivo simulado se conecta a la aplicación IoT Central, se conecta a la instancia de dispositivo que creó en la aplicación y comienza a enviar datos de telemetría. En la consola se muestran los detalles de conexión y la salida de telemetría: 
    
    ```output
    Starting event monitor, use ctrl-c to stop...
    event:
      component: ''
      interface: dtmi:com:example:TemperatureController;1
      module: ''
      origin: myDevice
      payload: '{"workingSet":1251}'
    
    event:
      component: thermostat1
      interface: dtmi:com:example:TemperatureController;1
      module: ''
      origin: myDevice
      payload: '{"temperature":22.00}'
    ```