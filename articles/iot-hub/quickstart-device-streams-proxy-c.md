---
title: 'Inicio rápido: Inicio rápido sobre los flujos de dispositivo de Azure IoT Hub en C para SSH y RDP'
description: En este tutorial, ejecutará una aplicación de C de ejemplo que actúa como proxy para habilitar escenarios de SSH y RDP mediante flujos de dispositivo de IoT Hub.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: c
ms.topic: quickstart
ms.custom: references_regions, devx-track-azurecli
ms.date: 03/14/2019
ms.author: robinsh
ms.openlocfilehash: c9c09914e30bdeae809924bc4256c773f42abd52
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107479947"
---
# <a name="quickstart-enable-ssh-and-rdp-over-an-iot-hub-device-stream-by-using-a-c-proxy-application-preview"></a>Inicio rápido: Habilitar SSH y RDP mediante un flujo de dispositivo de IoT Hub con una aplicación proxy en C (versión preliminar)

[!INCLUDE [iot-hub-quickstarts-4-selector](../../includes/iot-hub-quickstarts-4-selector.md)]

Azure IoT Hub admite actualmente flujos de dispositivos como una [característica en vista previa (GB)](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[Los flujos de dispositivo de IoT Hub](./iot-hub-device-streams-overview.md) permiten que las aplicaciones de servicio y de dispositivo se comuniquen de forma segura y apta para el firewall. Consulte la [página Ejemplo Proxy local](./iot-hub-device-streams-overview.md#local-proxy-sample-for-ssh-or-rdp) para obtener información general de la configuración.

En este inicio rápido se describe cómo configurar la tunelización del tráfico de Secure Shell (en el puerto 22) mediante flujos de dispositivo. La configuración del tráfico del Protocolo de escritorio remoto (RDP) es similar y solo requiere un cambio de configuración. Como los flujos de dispositivos son independientes del protocolo y de la aplicación, se puede modificar este inicio rápido para dar cabida a otros tipos de tráfico de la aplicación.

## <a name="prerequisites"></a>Requisitos previos

* La versión preliminar de los flujos de dispositivo solo se admite en este momento en instancias de IoT Hub creadas en las siguientes regiones:

  * Centro de EE. UU.
  * EUAP del centro de EE. UU.
  * Norte de Europa
  * Sudeste de Asia

* Instale [Visual Studio 2019](https://www.visualstudio.com/vs/) con la carga de trabajo [Desarrollo para el escritorio con C++](https://www.visualstudio.com/vs/support/selecting-workloads-visual-studio-2017/) habilitada.
* Instale la última versión de [Git](https://git-scm.com/download/).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

## <a name="how-it-works"></a>Funcionamiento

En la siguiente ilustración se muestra cómo configurar los programas de proxy local de dispositivo y local de servicio para permitir la conectividad de un extremo a otro entre los procesos de cliente y demonio de SSH. Durante la versión preliminar pública, el SDK para C solo admite flujos de dispositivo en el lado del dispositivo. Como resultado, en este inicio rápido solo se incluyen las instrucciones para ejecutar la aplicación de proxy local de servicio. Para compilar y ejecutar la aplicación del servicio complementaria, siga las instrucciones de uno de los siguientes inicios rápidos:

* [SSH/RDP mediante flujos de dispositivo de IoT Hub con aplicaciones proxy de C#](./quickstart-device-streams-proxy-csharp.md)
* [SSH/RDP mediante flujos de dispositivos de IoT Hub con aplicaciones proxy de Node.js](./quickstart-device-streams-proxy-nodejs.md)

![Configuración de proxy local](./media/quickstart-device-streams-proxy-c/device-stream-proxy-diagram.png)

1. El proxy local de servicio se conecta a IoT Hub e inicia un flujo de dispositivo al dispositivo de destino.

2. El proxy local de dispositivo completa el protocolo de enlace de iniciación del flujo y establece un túnel de streaming de un extremo a otro hasta el lado del servicio mediante el punto de conexión de streaming de IoT Hub.

3. El proxy local de dispositivo se conecta al demonio SSH que escucha en el puerto 22 del dispositivo. Esta configuración se puede modificar como se indica en la sección "Ejecución de la aplicación de proxy local de dispositivo".

4. El proxy local de servicio espera nuevas conexiones SSH de un usuario mediante la escucha en un puerto designado que, en este caso, es el puerto 2222. Esta configuración se puede modificar como se indica en la sección "Ejecución de la aplicación de proxy local de dispositivo". Cuando el usuario se conecta a través del cliente de SSH, el túnel permite que el tráfico de la aplicación de SSH se transfiera entre los programas cliente y servidor de SSH.

> [!NOTE]
> El tráfico de SSH que se envía en un flujo de dispositivo se tuneliza mediante el punto de conexión de streaming de IoT Hub en lugar de enviarse directamente entre el servicio y el dispositivo. Para más información, consulte las [ventajas que aporta el uso de los flujos de dispositivo de IoT Hub](iot-hub-device-streams-overview.md#benefits). Además, en la ilustración se muestra que el demonio de SSH se ejecuta en el mismo dispositivo (o máquina) que el proxy local de dispositivo. En este inicio rápido, como se proporciona la dirección IP del demonio de SSH, el proxy local de dispositivo y el demonio se pueden ejecutar también en máquinas diferentes.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prepare-the-development-environment"></a>Preparación del entorno de desarrollo

En este inicio rápido, va a usar el [SDK de dispositivo IoT de Azure para C](iot-hub-device-sdk-c-intro.md). Va a preparar un entorno de desarrollo utilizado para clonar y compilar el [SDK de IoT de Azure para C](https://github.com/Azure/azure-iot-sdk-c) desde GitHub. El SDK de GitHub incluye el código de ejemplo usado en este inicio rápido.

1. Descargue el [sistema de compilación CMake](https://cmake.org/download/).

    Es importante que los requisitos previos de Visual Studio (Visual Studio y la carga de trabajo *Desarrollo para el escritorio con C++*) estén instalados en la máquina, *antes* de comenzar la instalación de CMake. Cuando estén dispuestos los requisitos previos y haya comprobado la descarga, puede instalar el sistema de compilación de CMake.

1. Abra un símbolo del sistema o el shell de Bash de Git. Ejecute los siguientes comandos para clonar el repositorio de GitHub del [SDK de Azure IoT para C](https://github.com/Azure/azure-iot-sdk-c):

    ```cmd/sh
    git clone -b public-preview https://github.com/Azure/azure-iot-sdk-c.git
    cd azure-iot-sdk-c
    git submodule update --init
    ```

    Esta operación puede tardar algunos minutos.

1. Cree el subdirectorio *cmake* en el directorio raíz del repositorio de Git y vaya a esa carpeta. Ejecute los siguientes comandos desde el directorio *azure-iot-sdk-c*:

    ```cmd/sh
    mkdir cmake
    cd cmake
    ```

1. Ejecute el siguiente comando en el directorio *cmake* para compilar una versión del SDK específica para su plataforma de cliente de desarrollo.

   * En Linux:

      ```bash
      cmake ..
      make -j
      ```

   * En Windows, ejecute los siguientes comandos en el símbolo del sistema para desarrolladores correspondiente a Visual Studio 2015 o 2017. Se generará una solución de Visual Studio para el dispositivo simulado en el directorio *cmake*.

      ```cmd
      rem For VS2015
      cmake .. -G "Visual Studio 14 2015"

      rem Or for VS2017
      cmake .. -G "Visual Studio 15 2017"

      rem Or for VS2019
      cmake .. -G "Visual Studio 16 2019"

      rem Then build the project
      cmake --build . -- /m /p:Configuration=Release
      ```

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Registrar un dispositivo

Debe registrar un dispositivo con IoT Hub antes de poder conectarlo. En esta sección, usará Azure Cloud Shell con la [extensión de IoT](/cli/azure/ext/azure-iot/iot) para registrar un dispositivo simulado.

1. Ejecute el siguiente comando en Cloud Shell para crear la identidad del dispositivo:

   > [!NOTE]
   > * Reemplace el marcador de posición *YourIoTHubName* por el nombre que eligió para su centro de IoT.
   > * Para el nombre del dispositivo que va a registrar se recomienda usar *MyDevice*, tal como se muestra. Si elige otro nombre para el dispositivo, úselo en todo el artículo y actualice el nombre del dispositivo en las aplicaciones de ejemplo antes de ejecutarlas.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyDevice
    ```

1. Ejecute los siguientes comandos en Cloud Shell para obtener la *cadena de conexión del dispositivo* que acaba de registrar:

   > [!NOTE]
   > Reemplace el marcador de posición *YourIoTHubName* por el nombre que eligió para su centro de IoT.

    ```azurecli-interactive
    az iot hub device-identity connection-string show --hub-name {YourIoTHubName} --device-id MyDevice --output table
    ```

    Anote la cadena de conexión del dispositivo devuelta para usarla más adelante en este inicio rápido. Debe ser similar al ejemplo siguiente:

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyDevice;SharedAccessKey={YourSharedAccessKey}`

## <a name="ssh-to-a-device-via-device-streams"></a>Conexión mediante SSH a un dispositivo mediante flujos de dispositivo

En esta sección, establece una secuencia de un extremo a otro para tunelizar el tráfico SSH.

### <a name="run-the-device-local-proxy-application"></a>Ejecución de la aplicación de proxy local de dispositivo

1. Edite el archivo de origen **iothub_client_c2d_streaming_proxy_sample.c** en la carpeta `iothub_client/samples/iothub_client_c2d_streaming_proxy_sample`y especifique la cadena de conexión del dispositivo, el nombre de host o la dirección IP del dispositivo de destino, y el puerto 22 de SSH:

   ```C
   /* Paste in your device connection string  */
   static const char* connectionString = "{DeviceConnectionString}";
   static const char* localHost = "{IP/Host of your target machine}"; // Address of the local server to connect to.
   static const size_t localPort = 22; // Port of the local server to connect to.
   ```

1. Compile el ejemplo:

   ```bash
   # In Linux
   # Go to the sample's folder cmake/iothub_client/samples/iothub_client_c2d_streaming_proxy_sample
   make -j
   ```

   ```cmd
   rem In Windows
   rem Go to cmake at root of repository
   cmake --build . -- /m /p:Configuration=Release
   ```

1. Ejecute el programa compilado en el dispositivo:

   ```bash
   # In Linux
   # Go to the sample's folder cmake/iothub_client/samples/iothub_client_c2d_streaming_proxy_sample
   ./iothub_client_c2d_streaming_proxy_sample
   ```

   ```cmd
   rem In Windows
   rem Go to the sample's release folder cmake\iothub_client\samples\iothub_client_c2d_streaming_proxy_sample\Release
   iothub_client_c2d_streaming_proxy_sample.exe
   ```

### <a name="run-the-service-local-proxy-application"></a>Ejecución de la aplicación de proxy local de servicio

Como se explicó en la sección "Cómo funciona", para establecer un flujo de un extremo a otro a fin de tunelizar el tráfico de SSH, se necesita un proxy local en cada extremo (es decir, en el servicio y el dispositivo). Durante la versión preliminar pública, el SDK para C de IoT Hub solo admite flujos de dispositivo en el lado del dispositivo. Para compilar y ejecutar el proxy local de servicio, siga las instrucciones disponibles en uno de los siguientes inicios rápidos:

   * [SSH/RDP mediante flujos de dispositivo de IoT Hub con aplicaciones proxy de C#](./quickstart-device-streams-proxy-csharp.md)
   * [SSH/RDP mediante flujos de dispositivos de IoT Hub con aplicaciones proxy de Node.js](./quickstart-device-streams-proxy-nodejs.md)

### <a name="establish-an-ssh-session"></a>Establecimiento de una sesión de SSH

Con el proxy local de dispositivo y de servicio en ejecución, use su programa cliente de SSH y conéctese al proxy local de servicio en el puerto 2222 (en lugar de al demonio de SSH directamente).

```cmd/sh
ssh {username}@localhost -p 2222
```

En este momento, la ventana de inicio de sesión de SSH le pedirá que escriba sus credenciales.

La siguiente imagen muestra la salida de la consola en el proxy local de dispositivo que conecta con el demonio de SSH en `IP_address:22`:

![Salida del proxy local de dispositivo](./media/quickstart-device-streams-proxy-c/device-console-output.png)

La siguiente imagen muestra la salida de la consola del programa cliente de SSH. El cliente de SSH se comunica con el demonio de SSH mediante la conexión al puerto 22 en el que escucha el proxy local de servicio:

![Resultado del cliente de SSH](./media/quickstart-device-streams-proxy-csharp/ssh-console-output.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources-device-streams.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha configurado un centro de IoT, ha registrado un dispositivo, ha implementado un programa de proxy local de servicio y proxy local de dispositivo para establecer un flujo de dispositivo mediante IoT Hub y ha usado los servidores proxy para tunelizar el tráfico de SSH.

Para más información sobre los flujos de dispositivo, consulte:

> [!div class="nextstepaction"]
> [Introducción a los flujos de dispositivo](./iot-hub-device-streams-overview.md)
