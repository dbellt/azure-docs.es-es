---
title: Inicio rápido de conexión de un dispositivo Renesas RX65N-2MB a Azure IoT Central
description: Utilice software insertado de Azure RTOS para conectar un dispositivo Renesas RX65N-2MB a Azure IoT y enviar telemetría.
author: timlt
ms.author: timlt
ms.service: iot-develop
ms.devlang: c
ms.topic: quickstart
ms.date: 06/04/2021
ms.openlocfilehash: 9a509d878729648ee2768d536d9cca586604662e
ms.sourcegitcommit: 0af634af87404d6970d82fcf1e75598c8da7a044
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/15/2021
ms.locfileid: "112122016"
---
# <a name="quickstart-connect-a-renesas-starter-kit-for-rx65n-2mb-to-iot-central"></a>Inicio rápido: Conectar un dispositivo Renesas Starter Kit+ para RX65N-2 MB a IoT Central

**Se aplica a**: [desarrollo de dispositivos insertados](about-iot-develop.md#embedded-device-development)<br>
**Tiempo total para realizarlo**: 30 minutos

[![Examinar el código](media/common/browse-code.svg)](https://github.com/azure-rtos/getting-started/tree/master/Renesas/RSK_RX65N_2MB)

En este inicio rápido, usará Azure RTOS para conectar Renesas Starter Kit+ para RX65N-2MB (en adelante, Renesas RX65N) a Azure IoT.

Deberá completar las siguientes tareas:

* Instalación de un conjunto de herramientas de desarrollo insertadas para programar Renesas RX65N en C
* Creación de una imagen y guardado en la memoria flash en Renesas RX65N
* Usar Azure IoT Central para crear componentes de nube, ver las propiedades, ver la telemetría de los dispositivos y llamar a comandos directos

## <a name="prerequisites"></a>Requisitos previos

* Un equipo con Microsoft Windows 10
* [Git](https://git-scm.com/downloads) para clonar el repositorio
* Hardware

    * [Renesas Starter Kit+ para RX65N-2MB](https://www.renesas.com/products/microcontrollers-microprocessors/rx-32-bit-performance-efficiency-mcus/rx65n-2mb-starter-kit-plus-renesas-starter-kit-rx65n-2mb) (Renesas RX65N)
    * [Emulador de Renesas E2 Lite](https://www.renesas.com/software-tool/e2-emulator-lite-rte0t0002lkce00000r)
    * 2 cables de USB 2.0 A macho a mini USB macho
    * Fuente de alimentación de 5 V incluida
    * Cable Ethernet
    * Acceso a Ethernet por cable

## <a name="prepare-the-development-environment"></a>Preparación del entorno de desarrollo

Para configurar el entorno de desarrollo, primero tiene que clonar un repositorio de GitHub que contenga todos los recursos que necesita para el inicio rápido. Luego, instalará un conjunto de herramientas de programación.

### <a name="clone-the-repo-for-the-quickstart"></a>Clonación del repositorio para el inicio rápido

Clone el siguiente repositorio para descargar todo el código de dispositivo de ejemplo, los scripts de instalación y las versiones sin conexión de la documentación. Si ya ha clonado anteriormente este repositorio en otro inicio rápido, no es necesario que lo haga de nuevo.

Para clonar el repositorio, ejecute el siguiente comando:

```shell
git clone --recursive https://github.com/azure-rtos/getting-started.git
```

### <a name="install-the-tools"></a>Instalar las herramientas

El repositorio clonado contiene un script de instalación que instala y configura las herramientas necesarias. Si ha instalado estas herramientas en otro inicio rápido de dispositivo insertado, no es necesario volver a hacerlo.

> [!NOTE]
> El script de instalación instala las siguientes herramientas:
> * [CMake](https://cmake.org): compilación
> * [RX GCC](http://gcc-renesas.com/downloads/get.php?f=rx/8.3.0.202004-gnurx/gcc-8.3.0.202004-GNURX-ELF.exe): compilación
> * [Termite](https://www.compuphase.com/software_termite.htm): supervise la salida del puerto serie de los dispositivos conectados.

Para instalar las herramientas:

1. En el Explorador de archivos, vaya a la siguiente ruta de acceso en el repositorio y ejecute el script de instalación denominado *get-toolchain.bat*:

    *getting-started\tools\get-toolchain.bat*

1. Agregue el compilador RX a la ruta de acceso de Windows:

    *%USERPROFILE%\AppData\Roaming\GCC for Renesas RX 8.3.0.202004-GNURX-ELF\rx-elf\rx-elf\bin*

1. Después de la instalación, abra una nueva ventana de consola para reconocer los cambios de configuración realizados con el script de instalación. Use esta consola para realizar el resto de las tareas de programación del inicio rápido. Puede usar la línea de comandos de Windows, PowerShell o Git Bash para Windows.
1. Ejecute los comandos siguientes para confirmar que la versión 3.14 o posterior de CMake esté instalada y que la ruta de acceso del compilador RX esté correctamente configurada.

    ```shell
    cmake --version
    rx-elf-gcc
    ```
Para instalar las herramientas restantes:

* Instale [Renesas Flash Programmer](https://www.renesas.com/software-tool/renesas-flash-programmer-programming-gui). Renesas Flash Programmer contiene los controladores y las herramientas necesarios para almacenar Renesas RX65N en una memoria flash mediante Renesas E2 Lite.

[!INCLUDE [iot-develop-embedded-create-central-app-with-device](../../includes/iot-develop-embedded-create-central-app-with-device.md)]

## <a name="prepare-the-device"></a>Preparar el dispositivo

Para conectar Renesas RX65N a Azure, modificará un archivo de configuración de las opciones de Wi-Fi y Azure IoT, recompilará la imagen y la guardará en la memoria flash en el dispositivo.

### <a name="add-configuration"></a>Adición de configuración

1. Abra el siguiente archivo en un editor de texto:

    *getting-started\Renesas\RSK_RX65N_2MB\app\azure_config.h*

1. Establezca las constantes de información del dispositivo de Azure IoT en los valores que guardó después de crear los recursos de Azure.

    |Nombre invariable|Value|
    |-------------|-----|
    |`IOT_DPS_ID_SCOPE` |{*el valor de ámbito de id.* }|
    |`IOT_DPS_REGISTRATION_ID` |{*el valor de identificador de dispositivo*}|
    |`IOT_DEVICE_SAS_KEY` |{*el valor de la clave principal*}|

1. Guarde y cierre el archivo.

### <a name="build-the-image"></a>Compilación de la imagen

1. En la consola o en el Explorador de archivos, ejecute el script *rebuild.bat* en la siguiente ruta de acceso para compilar la imagen:

    *getting-started\Renesas\RSK_RX65N_2MB\tools\rebuild.bat*

2. Una vez finalizada la compilación, confirme que el archivo binario se ha creado en la siguiente ruta de acceso:

    *getting-started\Renesas\RSK_RX65N_2MB\build\app\rx65n_azure_iot.hex*

### <a name="connect-the-device"></a>Conexión del dispositivo

> [!NOTE]
> Para más información sobre la configuración y la introducción a Renesas RX65N, consulte [Renesas Starter Kit+ for RX65N-2MB Quick Start](https://www.renesas.com/document/man/e2studio-renesas-starter-kit-rx65n-2mb-quick-start-guide) (Inicio rápido de Renesas Starter Kit+ para RX65N-2 MB).

1. Complete los pasos siguientes con la imagen a continuación como referencia.
    
    :::image type="content" source="media/quickstart-devkit-renesas-rx65n-2mb/renesas-rx65n.jpg" alt-text="Busque reset, power, ethernet, USB y E1/E2Lite en la placa Renesas RX65N.":::

1. Con la fuente de alimentación de 5 V, conecte la **entrada de alimentación** de Renesas RX65N a una toma eléctrica.

1. Con el cable Ethernet, conecte **Ethernet** de Renesas RX65N al enrutador.

1. Con el primer cable mini USB, conecte **USB Serial** de Renesas RX65N al equipo.

1. Con el segundo cable Mini USB, conecte **E2 Lite USB Serial** de Renesas E2 Lite al equipo.

1. Con el cable de cinta proporcionado, conecte **E1/E2Lite** de Renesas RX65N a Renesas E2 Lite.

### <a name="flash-the-image"></a>Guardar la imagen en la memoria flash

1. Inicie la aplicación *Renesas Flash Programmer* desde el menú Inicio.

2. Seleccione *New Project...* (Nuevo proyecto...) en el menú *File* (Archivo) y escriba la siguiente configuración:
    * **Microcontroller** (Microcontrolador): RX65x
    * **Project Name** (Nombre de proyecto): RX65N
    * **Tool** (Herramienta): E2 emulator Lite

    :::image type="content" source="media/quickstart-devkit-renesas-rx65n-2mb/rfp-new.png" alt-text="Captura de pantalla de Renesas Flash Programmer, proyecto nuevo":::

3. Seleccione el botón *Tool Details* (Detalles de la herramienta) y vaya a la pestaña *Reset Settings* (Restablecer configuración).

4. Seleccione *Reset Pin as Hi-Z* (Restablecer pin como Hi-Z) y presione el botón *OK* (Aceptar).

    :::image type="content" source="media/quickstart-devkit-renesas-rx65n-2mb/rfp-reset.png" alt-text="Captura de pantalla de Renesas Flash Programmer, restablecer configuración":::

5. Presione el botón *Connect* (Conectar) y, cuando se le solicite, active la casilla *Auto Authentication* (Autenticación automática); a continuación, presione *OK* (Aceptar).

    :::image type="content" source="media/quickstart-devkit-renesas-rx65n-2mb/rfp-auth.png" alt-text="Captura de pantalla de Renesas Flash Programmer, autenticación":::

6. Seleccione el botón *Browse...* (Examinar...) y busque el archivo *rx65n_azure_iot.hex* creado en la sección anterior.

7. Presione *Start* (Iniciar) para empezar el almacenamiento en la memoria flash. Este proceso tarda aproximadamente 10 segundos.

### <a name="confirm-device-connection-details"></a>Confirmación de los detalles de conexión del dispositivo

Puede usar la aplicación **Termite** para supervisar la comunicación y confirmar que el dispositivo está configurado correctamente.
> [!TIP]
> Si tiene problemas para que el dispositivo se inicialice o se conecte después de guardar la imagen, consulte [Solución de problemas](troubleshoot-embedded-device-quickstarts.md).

1. Inicie **Termite**.
1. Haga clic en **Configuración**.
1. En el cuadro de diálogo **Serial port settings** (Configuración de puerto serie), compruebe los siguientes valores y actualícelos si es necesario:
    * **Baud rate** (Velocidad en baudios): 115.200
    * **Port** (puerto): puerto al que está conectado Renesas RX65N. Si hay varias opciones de puerto en la lista desplegable, busque ahí el puerto correcto que debe usar. Abra el **Administrador de dispositivos** de Windows y, en **Puertos**, identifique qué puerto debe usar.

    :::image type="content" source="media/quickstart-devkit-renesas-rx65n-2mb/termite-settings.png" alt-text="Captura de pantalla de la configuración del puerto en serie en la aplicación Termite":::

1. Seleccione Aceptar.
1. Presione el botón **Reset** (Restablecer) en el dispositivo.
1. En la aplicación **Termite**, compruebe los siguientes valores de punto de control para confirmar que el dispositivo se ha inicializado y está conectado a Azure IoT.

    ```output
    Starting Azure thread

    Initializing DHCP
        IP address: 10.0.0.81
        Mask: 255.255.255.0
        Gateway: 10.0.0.1
    SUCCESS: DHCP initialized

    Initializing DNS client
        DNS address: 10.0.0.1
    SUCCESS: DNS client initialized

    Initializing SNTP client
        SNTP server 0.pool.ntp.org
        SNTP IP address: 104.194.242.237
        SNTP time update: May 28, 2021 22:53:27.54 UTC 
    SUCCESS: SNTP initialized

    Initializing Azure IoT DPS client
        DPS endpoint: global.azure-devices-provisioning.net
        DPS ID scope: ***
        Registration ID: mydevice
    SUCCESS: Azure IoT DPS client initialized

    Initializing Azure IoT Hub client
        Hub hostname: ***.azure-devices.net
        Device id: mydevice
        Model id: dtmi:azurertos:devkit:gsg;1
    Connected to IoT Hub
    SUCCESS: Azure IoT Hub client initialized
    ```

Mantenga abierto Termite para supervisar la salida del dispositivo en los pasos siguientes.

## <a name="verify-the-device-status"></a>Comprobación del estado del dispositivo

Para ver el estado del dispositivo en el portal de IoT Central:
1. En el panel de la aplicación, seleccione **Dispositivos** en el menú de navegación lateral.
1. Confirme que **Estado del dispositivo** se ha actualizado a **Aprovisionado**.
1. Confirme que **Plantilla de dispositivo** se ha actualizado a **Guía de introducción**.

    :::image type="content" source="media/quickstart-devkit-renesas-rx65n-2mb/iot-central-device-view-status.png" alt-text="Captura de pantalla del estado del dispositivo en IoT Central":::

## <a name="view-telemetry"></a>Ver datos de telemetría

Con IoT Central, puede ver el flujo de telemetría desde el dispositivo hasta la nube.

Para ver la telemetría en el portal de IoT Central:

1. En el panel de la aplicación, seleccione **Dispositivos** en el menú de navegación lateral.
1. Seleccione el dispositivo de la lista.
1. En la pestaña **Información general**, puede ver la telemetría a medida que el dispositivo envía mensajes a la nube.

    :::image type="content" source="media/quickstart-devkit-renesas-rx65n-2mb/iot-central-device-telemetry.png" alt-text="Captura de pantalla de la telemetría del dispositivo en IoT Central":::

    > [!NOTE]
    > También puede supervisar la telemetría desde el dispositivo mediante la aplicación Termite.

## <a name="call-a-direct-method-on-the-device"></a>Llamar a un método directo en un dispositivo

También puede usar IoT Central para llamar a un método directo que haya implementado en el dispositivo. Los métodos directos tienen un nombre y, opcionalmente, pueden tener una carga de JSON, una conexión configurable y un tiempo de espera del método. En esta sección, se llama a un método que le permite apagar o encender un LED.

Para llamar a un método en el portal de IoT Central:

1. Seleccione la pestaña **Comando** en la página del dispositivo.
1. En la lista desplegable **Estado**, seleccione **Verdadero** y, a continuación, **Ejecutar**.  La luz LED debe encenderse.

    :::image type="content" source="media/quickstart-devkit-renesas-rx65n-2mb/iot-central-invoke-method.png" alt-text="Captura de pantalla de la llamada a un método directo en un dispositivo en IoT Central":::

1. En la lista desplegable **Estado**, seleccione **Falso** y, luego, elija **Ejecutar**. La luz LED debe apagarse.

## <a name="view-device-information"></a>Ver la información del dispositivo

Puede ver la información del dispositivo en IoT Central.

En la página del dispositivo, seleccione la pestaña **Acerca de**.

:::image type="content" source="media/quickstart-devkit-renesas-rx65n-2mb/iot-central-device-about.png" alt-text="Captura de pantalla de la información del dispositivo en IoT Central":::

## <a name="troubleshoot"></a>Solución de problemas

Si tiene problemas para compilar el código del dispositivo, guardar la imagen en la memoria flash del dispositivo o conectarse, consulte [Solución de problemas](troubleshoot-embedded-device-quickstarts.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no necesita los recursos de Azure que se han creado en este inicio rápido, puede eliminarlos desde el portal de IoT Central.

Para quitar la aplicación de ejemplo entera de Azure IoT Central y todos sus dispositivos y recursos:
1. Seleccione **Administración**  > **Su aplicación**.
1. Seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha compilado una imagen personalizada que contiene código de ejemplo de Azure RTOS y, después, la ha guardado en la memoria flash en el dispositivo Renesas RX65N. También ha usado el portal de IoT Central para crear recursos de Azure, conectar Renesas RX65N de forma segura a Azure, ver la telemetría y enviar mensajes.

Como siguiente paso, explore los siguientes artículos para más información sobre el uso de los SDK de dispositivos IoT para conectar dispositivos a Azure IoT. 

> [!div class="nextstepaction"]
> [Conexión de un dispositivo simulado a IoT Central](quickstart-send-telemetry-central.md)
> [!div class="nextstepaction"]
> [Conexión de un dispositivo simulado a IoT Hub](quickstart-send-telemetry-iot-hub.md)

> [!IMPORTANT]
> Azure RTOS proporciona a los OEM componentes para proteger la comunicación y crear código y aislamiento de datos mediante los mecanismos de protección de hardware MCU/MPU subyacentes. Sin embargo, cada OEM es responsable en última instancia de garantizar que el dispositivo cumpla los requisitos de seguridad en constante evolución.

