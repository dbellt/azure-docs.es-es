---
title: Inicio rápido para la conexión de un dispositivo STMicroelectronics B-L475E-IOT01A o B-L4S5I-IOT01A a Azure IoT Central
description: Use el software insertado de Azure RTOS para conectar un dispositivo STMicroelectronics B-L475E-IOT01A o B-L4S5I-IOT01A a Azure IoT y enviar datos de telemetría.
author: timlt
ms.author: timlt
ms.service: iot-develop
ms.devlang: c
ms.topic: quickstart
ms.date: 06/02/2021
ms.openlocfilehash: dd8a214e060f9f4ace9f310fec3ebca33d4619c0
ms.sourcegitcommit: 98308c4b775a049a4a035ccf60c8b163f86f04ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113109240"
---
# <a name="quickstart-connect-an-stmicroelectronics-b-l475e-iot01a-or-b-l4s5i-iot01a-discovery-kit-to-iot-central"></a>Inicio rápido: Conexión de un dispositivo STMicroelectronics B-L475E-IOT01A o B-L4S5I-IOT01A a Azure IoT Central

**Se aplica a**: [desarrollo de dispositivos insertados](about-iot-develop.md#embedded-device-development)<br>
**Tiempo total para realizarlo**: 30 minutos

[![Examinar el código](media/common/browse-code.svg)](https://github.com/azure-rtos/getting-started/tree/master/STMicroelectronics/STM32L4_L4+)

En este inicio rápido, va a utilizar Azure RTOS para conectar los kits de detección STMicroelectronics [B-L475E-IOT01A](https://www.st.com/en/evaluation-tools/b-l475e-iot01a.html) o [B-L4S5I-IOT01A](https://www.st.com/en/evaluation-tools/b-l4s5i-iot01a.html) (en adelante, STM DevKit) a Azure IoT.

Deberá completar las siguientes tareas:

* Instalar un conjunto de herramientas de desarrollo insertado para programar el dispositivo STM DevKit en C
* Compilar una imagen y guardarla en la memoria flash en STM DevKit
* Usar Azure IoT Central para crear componentes de nube, ver las propiedades, ver la telemetría de los dispositivos y llamar a comandos directos

## <a name="prerequisites"></a>Requisitos previos

* Un equipo con Microsoft Windows 10
* [Git](https://git-scm.com/downloads) para clonar el repositorio
* Hardware

    * [B-L475E-IOT01A](https://www.st.com/en/evaluation-tools/b-l475e-iot01a.html) o [B-L4S5I-IOT01A](https://www.st.com/en/evaluation-tools/b-l4s5i-iot01a.html) (STM DevKit)
    * Conexión Wi-Fi a 2,4 GHz
    * Conector USB 2.0 A macho a cable macho micro USB

## <a name="prepare-the-development-environment"></a>Preparación del entorno de desarrollo

Para configurar el entorno de desarrollo, primero tiene que clonar un repositorio de GitHub que contenga todos los recursos que necesita para el inicio rápido. Luego, instalará un conjunto de herramientas de programación.

### <a name="clone-the-repo-for-the-quickstart"></a>Clonación del repositorio para el inicio rápido

Clone el siguiente repositorio para descargar todo el código de dispositivo de ejemplo, los scripts de instalación y las versiones sin conexión de la documentación. Si ya ha clonado anteriormente este repositorio en otro inicio rápido, no es necesario que lo haga de nuevo.

Para clonar el repositorio, ejecute el siguiente comando:

```shell
git clone --recursive https://github.com/azure-rtos/getting-started.git
```

### <a name="install-the-tools"></a>Instalar las herramientas

El repositorio clonado contiene un script de instalación que instala y configura las herramientas necesarias. Si instaló estas herramientas en otro inicio rápido de dispositivo insertado, no es necesario volver a hacerlo.

> [!NOTE]
> El script de instalación instala las siguientes herramientas:
> * [CMake](https://cmake.org): compilación
> * [ARM GCC](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm): compilación
> * [Termite](https://www.compuphase.com/software_termite.htm): supervise la salida del puerto serie de los dispositivos conectados.

Para instalar las herramientas:

1. En el Explorador de archivos, vaya a la siguiente ruta de acceso en el repositorio y ejecute el script de instalación denominado *get-toolchain.bat*:

    *getting-started\tools\get-toolchain.bat*

1. Después de la instalación, abra una nueva ventana de consola para reconocer los cambios de configuración realizados con el script de instalación. Use esta consola para realizar el resto de las tareas de programación del inicio rápido. Puede usar la línea de comandos de Windows, PowerShell o Git Bash para Windows.
1. Ejecute el código siguiente para confirmar la instalación de CMake versión 3.14 o posterior.

    ```shell
    cmake --version
    ```
[!INCLUDE [iot-develop-embedded-create-central-app-with-device](../../includes/iot-develop-embedded-create-central-app-with-device.md)]

## <a name="prepare-the-device"></a>Preparar el dispositivo

Para conectar STM DevKit a Azure, modificará un archivo de configuración de las opciones de Wi-Fi y Azure IoT, recompilará la imagen y la guardará en la memoria flash en el dispositivo.

### <a name="add-configuration"></a>Adición de configuración

1. Abra el siguiente archivo en un editor de texto:

    *getting-started\STMicroelectronics\STM32L4_L4+\app\azure_config.h*

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

1. En la consola o en el Explorador de archivos, ejecute el script *rebuild.bat* en la siguiente ruta de acceso para compilar la imagen:

    *getting-started\STMicroelectronics\STM32L4_L4+\tools\rebuild.bat*

2. Una vez completada la compilación, confirme que se crearon dos archivos binarios. Hay una imagen binaria para cada STM Devkit. La compilación guarda las imágenes en las rutas de acceso siguientes:

    *getting-started\STMicroelectronics\STM32L4_L4+\build\app\stm32l475_azure_iot.bin*

    *getting-started\STMicroelectronics\STM32L4_L4+\build\app\stm32l4S5_azure_iot.bin*

### <a name="flash-the-image"></a>Guardar la imagen en la memoria flash

1. En el MCU de STM DevKit, busque el botón **Restablecer**, el puerto Micro USB, que tiene la etiqueta **USB STLink**, y el número de pieza de la placa. En los pasos siguientes se hará referencia a estos elementos. Todos ellos se resaltan en la siguiente imagen:

    :::image type="content" source="media/quickstart-devkit-stm-b-l475e/stm-devkit-board.png" alt-text="Búsqueda de componentes clave en la placa STM DevKit":::

1. Conecte el cable Micro USB al puerto **USB STLINK** del STM DevKit y, a continuación, conéctelo al equipo.

    > [!NOTE]
    > Para obtener información detallada sobre la configuración de STM DevKit, vea las instrucciones sobre el empaquetado o consulte los [recursos de B-L475E-IOT01A](https://www.st.com/en/evaluation-tools/b-l475e-iot01a.html#resource) o [los recursos de B-L4S5I-IOT01A](https://www.st.com/en/evaluation-tools/b-l4s5i-iot01a.html#resource).

1. En el Explorador de archivos, busque los archivos binarios que creó en la sección anterior.

1. Copie el archivo binario cuyo nombre de archivo corresponde al número de pieza del STM Devkit que está usando. Por ejemplo, si el número de pieza de la placa es **B-L475E-IOT01A1**, copie el archivo binario denominado *stm32l475_azure_iot.bin*.

1. En el Explorador de archivos, busque el dispositivo STM Devkit conectado al equipo. El dispositivo aparece como una unidad en el sistema con la etiqueta de unidad **DIS_L4IOT**.

1. Pegue el archivo binario en la carpeta raíz de STM Devkit. El almacenamiento en la memoria flash comienza inmediatamente y termina al cabo de unos segundos.

    > [!NOTE]
    > Durante el proceso de flashing, un LED alterna entre rojo y verde en STM DevKit.

### <a name="confirm-device-connection-details"></a>Confirmación de los detalles de conexión del dispositivo

Puede usar la aplicación **Termite** para supervisar la comunicación y confirmar que el dispositivo está configurado correctamente.

1. Inicie **Termite**.
    > [!TIP]
    > Si no puede conectar Termite al kit de desarrollo, instale el [controlador ST-LINK](https://my.st.com/content/ccc/resource/technical/software/driver/files/stsw-link009.zip) e inténtelo de nuevo. Consulte [Solución de problemas](troubleshoot-embedded-device-quickstarts.md) para ver otros pasos.
1. Haga clic en **Configuración**.
1. En el cuadro de diálogo **Serial port settings** (Configuración de puerto serie), compruebe los siguientes valores y actualícelos si es necesario:
    * **Baud rate** (Velocidad en baudios): 115.200
    * **Puerto:** Puerto al que está conectado el STM DevKit. Si hay varias opciones de puerto en la lista desplegable, busque ahí el puerto correcto que debe usar. Abra el **Administrador de dispositivos** de Windows y, en **Puertos**, identifique qué puerto debe usar.

    :::image type="content" source="media/quickstart-devkit-stm-b-l475e/termite-settings.png" alt-text="Captura de pantalla de la configuración del puerto serie en la aplicación Termite":::

1. Seleccione Aceptar.
1. Presione el botón **Reset** (Restablecer) en el dispositivo. El botón es negro y está etiquetado en el dispositivo.
1. En la aplicación **Termite**, compruebe los siguientes valores de punto de control para confirmar que el dispositivo se ha inicializado y está conectado a Azure IoT.

    ```output
    Starting Azure thread

    Initializing WiFi
        Module: ISM43362-M3G-L44-SPI
        MAC address: C4:7F:51:8F:67:F6
        Firmware revision: C3.5.2.5.STM
        Connecting to SSID 'iot'
    SUCCESS: WiFi connected to iot

    Initializing DHCP
        IP address: 192.168.0.22
        Gateway: 192.168.0.1
    SUCCESS: DHCP initialized

    Initializing DNS client
        DNS address: 75.75.75.75
    SUCCESS: DNS client initialized

    Initializing SNTP client
        SNTP server 0.pool.ntp.org
        SNTP IP address: 108.62.122.57
        SNTP time update: May 21, 2021 22:42:8.394 UTC 
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
    > [!IMPORTANT]
    > Si se produce un error en la inicialización del cliente DNS y le notifica que el firmware de Wi-Fi no está actualizado, deberá actualizar el firmware del módulo Wi-Fi. Descargue e instale la [actualización de firmware del módulo de Wi-Fi Inventek ISM 43362](https://www.st.com/resource/en/utilities/inventek_fw_updater.zip). A continuación, presione el botón **Reset** en el dispositivo para volver a comprobar la conexión y continúe con este inicio rápido.


Mantenga abierto Termite para supervisar la salida del dispositivo en los pasos siguientes.

## <a name="verify-the-device-status"></a>Comprobación del estado del dispositivo

Para ver el estado del dispositivo en el portal de IoT Central:
1. En el panel de la aplicación, seleccione **Dispositivos** en el menú de navegación lateral.
1. Confirme que **Estado del dispositivo** se ha actualizado a **Aprovisionado**.
1. Confirme que **Plantilla de dispositivo** se ha actualizado a **Guía de introducción**.

    :::image type="content" source="media/quickstart-devkit-stm-b-l475e/iot-central-device-view-status.png" alt-text="Captura de pantalla del estado del dispositivo en IoT Central":::

## <a name="view-telemetry"></a>Ver datos de telemetría

Con IoT Central, puede ver el flujo de telemetría desde el dispositivo hasta la nube.

Para ver la telemetría en el portal de IoT Central:

1. En el panel de la aplicación, seleccione **Dispositivos** en el menú de navegación lateral.
1. Seleccione el dispositivo de la lista.
1. En la pestaña **Información general**, puede ver la telemetría a medida que el dispositivo envía mensajes a la nube.

    :::image type="content" source="media/quickstart-devkit-stm-b-l475e/iot-central-device-telemetry.png" alt-text="Captura de pantalla de la telemetría del dispositivo en IoT Central":::

    > [!NOTE]
    > También puede supervisar la telemetría desde el dispositivo mediante la aplicación Termite.

## <a name="call-a-direct-method-on-the-device"></a>Llamar a un método directo en un dispositivo

También puede usar IoT Central para llamar a un método directo que haya implementado en el dispositivo. Los métodos directos tienen un nombre y, opcionalmente, pueden tener una carga de JSON, una conexión configurable y un tiempo de espera del método. En esta sección, se llama a un método que le permite apagar o encender un LED.

Para llamar a un método en el portal de IoT Central:

1. Seleccione la pestaña **Comando** en la página del dispositivo.
1. En la lista desplegable **Estado**, seleccione **Verdadero** y, a continuación, seleccione **Ejecutar**.  La luz LED debe encenderse.

    :::image type="content" source="media/quickstart-devkit-stm-b-l475e/iot-central-invoke-method.png" alt-text="Captura de pantalla de la llamada a un método directo en un dispositivo en IoT Central":::

1. En la lista desplegable **Estado**, seleccione **Falso** y, luego, elija **Ejecutar**. La luz LED debe apagarse.

## <a name="view-device-information"></a>Ver la información del dispositivo

Puede ver la información del dispositivo en IoT Central.

En la página del dispositivo, seleccione la pestaña **Acerca de**.

:::image type="content" source="media/quickstart-devkit-stm-b-l475e/iot-central-device-about.png" alt-text="Captura de pantalla de la información del dispositivo en IoT Central":::

## <a name="troubleshoot-and-debug"></a>Solución de problemas y depuración

Si tiene problemas para compilar el código del dispositivo, guardar la imagen en la memoria flash del dispositivo o conectarse, consulte [Solución de problemas](troubleshoot-embedded-device-quickstarts.md).

Para depurar la aplicación, consulte [Depuración con Visual Studio Code](https://github.com/azure-rtos/getting-started/blob/master/docs/debugging.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no necesita los recursos de Azure que se han creado en este inicio rápido, puede eliminarlos desde el portal de IoT Central.

Para quitar la aplicación de ejemplo entera de Azure IoT Central y todos sus dispositivos y recursos:
1. Seleccione **Administración**  > **Su aplicación**.
1. Seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha compilado una imagen personalizada que contiene código de ejemplo de Azure RTOS y, después, la ha guardado en la memoria flash en el dispositivo STM DevKit. También ha usado el portal de IoT Central para crear recursos de Azure, conectar STM DevKit de forma segura a Azure, ver la telemetría y enviar mensajes.

Como siguiente paso, explore los siguientes artículos para más información sobre el uso de los SDK de dispositivos IoT para conectar dispositivos a Azure IoT. 

> [!div class="nextstepaction"]
> [Conexión de un dispositivo simulado a IoT Central](quickstart-send-telemetry-central.md)
> [!div class="nextstepaction"]
> [Conexión de un dispositivo simulado a IoT Hub](quickstart-send-telemetry-iot-hub.md)


> [!IMPORTANT]
> Azure RTOS proporciona a los OEM componentes para proteger la comunicación y crear código y aislamiento de datos mediante los mecanismos de protección de hardware MCU/MPU subyacentes. Sin embargo, cada OEM es responsable en última instancia de garantizar que el dispositivo cumpla los requisitos de seguridad en constante evolución.



