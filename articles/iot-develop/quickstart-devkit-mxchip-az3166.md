---
title: 'Inicio rápido: Conexión de un dispositivo MXCHIP AZ3166 a Azure IoT Central'
description: Use software insertado de Azure RTOS para conectar un dispositivo MXCHIP AZ3166 a Azure IoT y enviar telemetría.
author: timlt
ms.author: timlt
ms.service: iot-develop
ms.devlang: c
ms.topic: quickstart
ms.date: 03/17/2021
ms.openlocfilehash: 66ddc9f080383dac7703b00e62878df7714c4201
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110061301"
---
# <a name="quickstart-connect-an-mxchip-az3166-devkit-to-iot-central"></a>Inicio rápido: Conexión de un kit de desarrollo MXCHIP AZ3166 a IoT Central

**Se aplica a**: [desarrollo de dispositivos insertados](about-iot-develop.md#embedded-device-development)<br>
**Tiempo total para realizarlo**: 30 minutos

[![Examinar el código](media/common/browse-code.svg)](https://github.com/azure-rtos/getting-started/tree/master/MXChip/AZ3166)

En este tutorial, usará Azure RTOS para conectar un kit de desarrollo de IoT para MXCHIP AZ3166 (en adelante, MXCHIP DevKit) a Azure IoT. El artículo forma parte de la serie [Introducción al desarrollo de dispositivos insertados de Azure IoT](quickstart-device-development.md). La serie presenta Azure RTOS a los desarrolladores de dispositivos y muestra cómo conectar varios kits de evaluación de dispositivos a Azure IoT.

Deberá completar las siguientes tareas:

* Instalar un conjunto de herramientas de desarrollo insertado para programar un dispositivo MXCHIP DevKit en C
* Compilar una imagen y guardarla en la memoria flash en MXCHIP DevKit
* Usar Azure IoT Central para crear componentes de nube, ver las propiedades, ver la telemetría de los dispositivos y llamar a comandos directos

## <a name="prerequisites"></a>Requisitos previos

* Un equipo con Microsoft Windows 10
* [Git](https://git-scm.com/downloads) para clonar el repositorio
* Hardware

    > * El [kit de desarrollo de IoT para MXCHIP AZ3166](https://aka.ms/iot-devkit) (MXCHIP DevKit)
    > * Conexión Wi-Fi a 2,4 GHz
    > * Conector USB 2.0 A macho a cable macho micro USB

## <a name="prepare-the-development-environment"></a>Preparación del entorno de desarrollo

Para configurar el entorno de desarrollo, primero debe clonar un repositorio de GitHub que contenga todos los recursos que necesita para el tutorial. Luego, instalará un conjunto de herramientas de programación.

### <a name="clone-the-repo-for-the-tutorial"></a>Clonación del repositorio del tutorial

Clone el siguiente repositorio para descargar todo el código de dispositivo de ejemplo, los scripts de instalación y las versiones sin conexión de la documentación. Si ya ha clonado anteriormente este repositorio en otro tutorial, no es necesario que lo haga de nuevo.

Para clonar el repositorio, ejecute el siguiente comando:

```shell
git clone --recursive https://github.com/azure-rtos/getting-started.git
```

### <a name="install-the-tools"></a>Instalar las herramientas

El repositorio clonado contiene un script de instalación que instala y configura las herramientas necesarias. Si instaló estas herramientas en otro tutorial en la guía de introducción, no es necesario volver a hacerlo.

> [!NOTE]
> El script de instalación instala las siguientes herramientas:
> * [CMake](https://cmake.org): compilación
> * [ARM GCC](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm): compilación
> * [Termite](https://www.compuphase.com/software_termite.htm): supervise la salida del puerto serie de los dispositivos conectados.

Para instalar las herramientas:

1. En el Explorador de archivos, vaya a la siguiente ruta de acceso en el repositorio y ejecute el script de instalación denominado *get-toolchain.bat*:

    > *getting-started\tools\get-toolchain.bat*

1. Después de la instalación, abra una nueva ventana de consola para reconocer los cambios de configuración realizados con el script de instalación. Use esta consola para realizar el resto de tareas de programación del tutorial. Puede usar la línea de comandos de Windows, PowerShell o Git Bash para Windows.
1. Ejecute el código siguiente para confirmar la instalación de CMake versión 3.14 o posterior.

    ```shell
    cmake --version
    ```

[!INCLUDE [iot-develop-embedded-create-central-app-with-device](../../includes/iot-develop-embedded-create-central-app-with-device.md)]

## <a name="prepare-the-device"></a>Preparar el dispositivo

Para conectar MXCHIP DevKit a Azure, modificará un archivo de configuración de las opciones de Wi-Fi y Azure IoT, recompilará la imagen y la guardará en la memoria flash en el dispositivo.

### <a name="add-configuration"></a>Adición de configuración

1. Abra el siguiente archivo en un editor de texto:

    > *getting-started\MXChip\AZ3166\app\azure_config.h*

1. Establezca las constantes Wi-Fi en los siguientes valores del entorno local.

    |Nombre invariable|Value|
    |-------------|-----|
    |`WIFI_SSID` |{*el SSID de Wi-Fi*}|
    |`WIFI_PASSWORD` |{*la contraseña de Wi-Fi*}|
    |`WIFI_MODE` |{*uno de los valores de modo Wi-Fi enumerados en el archivo*}|

1. Establezca las constantes de información del dispositivo de Azure IoT en los valores que guardó después de crear los recursos de Azure.

    |Nombre invariable|Value|
    |-------------|-----|
    |`IOT_DPS_ID_SCOPE` |{*el valor de ámbito de id.* }|
    |`IOT_DPS_REGISTRATION_ID` |{*el valor de identificador de dispositivo*}|
    |`IOT_DEVICE_SAS_KEY` |{*el valor de la clave principal*}|

1. Guarde y cierre el archivo.

### <a name="build-the-image"></a>Compilación de la imagen

En la consola o en el Explorador de archivos, ejecute el script *rebuild.bat* en la siguiente ruta de acceso para compilar la imagen:

> *getting-started\MXChip\AZ3166\tools\rebuild.bat*

Una vez finalizada la compilación, confirme que el archivo binario se ha creado en la siguiente ruta de acceso:

> *getting-started\MXChip\AZ3166\build\app\mxchip_azure_iot.bin*

### <a name="flash-the-image"></a>Guardar la imagen en la memoria flash

1. En MXCHIP DevKit, busque el botón **Reset** (Restablecer) y el puerto micro USB. Usará estos componentes en los pasos siguientes. Ambos se resaltan en la siguiente imagen:

    :::image type="content" source="media/quickstart-devkit-mxchip-az3166/mxchip-iot-devkit.png" alt-text="Búsqueda de componentes clave en la placa del kit de desarrollo de MXChip":::

1. Conecte el cable micro USB al puerto micro USB de MXCHIP DevKit y, luego, conéctelo al equipo.
1. En el Explorador de archivos, busque el archivo binario que creó en la sección anterior.
1. Copie el archivo binario *mxchip_azure_iot.bin*.
1. En el Explorador de archivos, busque el dispositivo MXCHIP DevKit conectado al equipo. El dispositivo aparece como una unidad en el sistema con la etiqueta **AZ3166**.
1. Pegue el archivo binario en la carpeta raíz de MXCHIP DevKit. El almacenamiento en la memoria flash comienza inmediatamente y termina al cabo de unos segundos.

    > [!NOTE]
    > Durante este proceso, una luz LED verde se enciende y se apaga en MXCHIP DevKit.

### <a name="confirm-device-connection-details"></a>Confirmación de los detalles de conexión del dispositivo

Puede usar la aplicación **Termite** para supervisar la comunicación y confirmar que el dispositivo está configurado correctamente.

1. Inicie **Termite**.
    > [!TIP]
    > Si no puede conectar Termite al kit de desarrollo, instale el [controlador ST-LINK](https://my.st.com/content/ccc/resource/technical/software/driver/files/stsw-link009.zip) e inténtelo de nuevo. Consulte [Solución de problemas](https://github.com/azure-rtos/getting-started/blob/master/docs/troubleshooting.md) para ver otros pasos.
1. Haga clic en **Configuración**.
1. En el cuadro de diálogo **Serial port settings** (Configuración de puerto serie), compruebe los siguientes valores y actualícelos si es necesario:
    * **Baud rate** (Velocidad en baudios): 115.200
    * **Port** (Puerto): el puerto al que está conectado MXCHIP DevKit. Si hay varias opciones de puerto en la lista desplegable, busque ahí el puerto correcto que debe usar. Abra el **Administrador de dispositivos** de Windows y, en **Puertos**, identifique qué puerto debe usar.

    :::image type="content" source="media/quickstart-devkit-mxchip-az3166/termite-settings.png" alt-text="Confirmación de la configuración en la aplicación Termite":::

1. Seleccione Aceptar.
1. Presione el botón **Reset** (Restablecer) en el dispositivo. El botón está rotulado en el dispositivo y se encuentra cerca del conector micro USB.
1. En la aplicación **Termite**, compruebe los siguientes valores de punto de control para confirmar que el dispositivo se ha inicializado y está conectado a Azure IoT.

    ```output
    Starting Azure thread

    Initializing WiFi
        Connecting to SSID 'iot'
    SUCCESS: WiFi connected to iot

    Initializing DHCP
        IP address: 10.0.0.123
        Mask: 255.255.255.0
        Gateway: 10.0.0.1
    SUCCESS: DHCP initialized

    Initializing DNS client
        DNS address: 10.0.0.1
    SUCCESS: DNS client initialized

    Initializing SNTP client
        SNTP server 0.pool.ntp.org
        SNTP IP address: 185.242.56.3
        SNTP time update: Nov 16, 2020 23:47:35.385 UTC 
    SUCCESS: SNTP initialized

    Initializing Azure IoT DPS client
        DPS endpoint: global.azure-devices-provisioning.net
        DPS ID scope: ***
        Registration ID: ***
    SUCCESS: Azure IoT DPS client initialized

    Initializing Azure IoT Hub client
        Hub hostname: ***
        Device id: ***
        Model id: dtmi:azurertos:devkit:gsgmxchip;1
    Connected to IoTHub
    SUCCESS: Azure IoT Hub client initialized

    Starting Main loop
    ```

Mantenga abierto Termite para supervisar la salida del dispositivo en los pasos siguientes.

## <a name="verify-the-device-status"></a>Comprobación del estado del dispositivo

Para ver el estado del dispositivo en el portal de IoT Central:
1. En el panel de la aplicación, seleccione **Dispositivos** en el menú de navegación lateral.
1. Confirme que **Estado del dispositivo** se ha actualizado a **Aprovisionado**.
1. Confirme que **Plantilla de dispositivo** se ha actualizado a **Guía de introducción**.

    :::image type="content" source="media/quickstart-devkit-mxchip-az3166/iot-central-device-view-status.png" alt-text="Vista del estado del dispositivo en IoT Central":::

## <a name="view-telemetry"></a>Ver datos de telemetría

Con IoT Central, puede ver el flujo de telemetría desde el dispositivo hasta la nube.

Para ver la telemetría en el portal de IoT Central:

1. En el panel de la aplicación, seleccione **Dispositivos** en el menú de navegación lateral.
1. Seleccione el dispositivo de la lista.
1. En la pestaña **Información general**, puede ver la telemetría a medida que el dispositivo envía mensajes a la nube.

    :::image type="content" source="media/quickstart-devkit-mxchip-az3166/iot-central-device-telemetry.png" alt-text="Vista de la telemetría del dispositivo en IoT Central":::

    > [!NOTE]
    > También puede supervisar la telemetría desde el dispositivo mediante la aplicación Termite.

## <a name="call-a-direct-method-on-the-device"></a>Llamar a un método directo en un dispositivo

También puede usar IoT Central para llamar a un método directo que haya implementado en el dispositivo. Los métodos directos tienen un nombre y, opcionalmente, pueden tener una carga de JSON, una conexión configurable y un tiempo de espera del método. En esta sección, se llama a un método que le permite apagar o encender un LED.

Para llamar a un método en el portal de IoT Central:

1. Seleccione la pestaña **Comando** en la página del dispositivo.
1. Seleccione **Estado** y elija **Ejecutar**.  La luz LED debe encenderse.

    :::image type="content" source="media/quickstart-devkit-mxchip-az3166/iot-central-invoke-method.png" alt-text="Llamada a un método directo en un dispositivo":::

1. Deseleccione **Estado** y elija **Ejecutar**. La luz LED debe apagarse.

## <a name="view-device-information"></a>Ver la información del dispositivo

Puede ver la información del dispositivo en IoT Central.

En la página del dispositivo, seleccione la pestaña **Acerca de**.

:::image type="content" source="media/quickstart-devkit-mxchip-az3166/iot-central-device-about.png" alt-text="Vista de información sobre el dispositivo en IoT Central":::

## <a name="debugging"></a>Depuración

Para depurar la aplicación, consulte [Depuración con Visual Studio Code](https://github.com/azure-rtos/getting-started/blob/master/docs/debugging.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no necesita los recursos de Azure que se han creado en este tutorial, puede eliminarlos desde el portal de IoT Central. También, si continúa con otro tutorial de esta guía de introducción, puede mantener los recursos que ya ha creado y volver a usarlos.

Para quitar la aplicación de ejemplo entera de Azure IoT Central y todos sus dispositivos y recursos:
1. Seleccione **Administración**  > **Su aplicación**.
1. Seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha compilado una imagen personalizada que contiene código de ejemplo de Azure RTOS y, después, la ha guardado en la memoria flash en el dispositivo MXCHIP DevKit. También ha usado el portal de IoT Central para crear recursos de Azure, conectar MXCHIP DevKit de forma segura a Azure, ver la telemetría y enviar mensajes.

* Para los desarrolladores de dispositivos, el siguiente paso sugerido es consultar los otros tutoriales de la serie [Introducción al desarrollo de dispositivos insertados de Azure IoT](quickstart-device-development.md).
* Si después de seguir los pasos de esta guía tiene problemas para inicializar el dispositivo o conectarse, consulte [Solución de problemas](https://github.com/azure-rtos/getting-started/blob/master/docs/troubleshooting.md).
* Para más información sobre cómo se usan los componentes de Azure RTOS en el código de ejemplo de este tutorial, consulte [Uso de Azure RTOS en la guía de introducción](https://github.com/azure-rtos/getting-started/blob/master/docs/using-azure-rtos.md).

    > [!IMPORTANT]
    > Azure RTOS proporciona a los OEM componentes para proteger la comunicación y crear código y aislamiento de datos mediante los mecanismos de protección de hardware MCU/MPU subyacentes. Sin embargo, cada OEM es responsable en última instancia de garantizar que el dispositivo cumpla los requisitos de seguridad en constante evolución.

