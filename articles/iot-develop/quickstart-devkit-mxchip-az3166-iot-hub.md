---
title: 'Inicio rápido: Conexión de un dispositivo MXCHIP AZ3166 a Azure IoT Hub'
description: Use el software insertado de Azure RTOS para conectar un dispositivo MXCHIP AZ3166 a Azure IoT y enviar telemetría.
author: timlt
ms.author: timlt
ms.service: iot-develop
ms.devlang: c
ms.topic: quickstart
ms.date: 06/09/2021
ms.openlocfilehash: a14f00585eede96814627c941050c3179436ab06
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112059731"
---
# <a name="quickstart-connect-an-mxchip-az3166-devkit-to-iot-hub"></a>Inicio rápido: Conexión de un kit de desarrollo MXCHIP AZ3166 a IoT Hub

**Se aplica a**: [desarrollo de dispositivos insertados](about-iot-develop.md#embedded-device-development)<br>
**Tiempo total para realizarlo**: 30 minutos

[![Examinar el código](media/common/browse-code.svg)](https://github.com/azure-rtos/getting-started/tree/master/MXChip/AZ3166)

En este inicio rápido, usará Azure RTOS para conectar un kit de desarrollo de IoT para MXCHIP AZ3166 (en adelante, MXCHIP DevKit) a Azure IoT. 

También usará IoT Explorer y IoT Plug and Play para administrar el dispositivo MXCHIP DevKit. IoT Plug and Play proporciona un modelo de dispositivo abierto que permite a las aplicaciones consultar mediante programación las funcionalidades de un dispositivo e interactuar con él. Un dispositivo usa este modelo para transmitir sus funcionalidades a una aplicación habilitada para IoT Plug and Play. Con este modelo, puede simplificar y mejorar las tareas de agregar, configurar y administrar dispositivos. Para obtener más información, vea la [documentación de IoT Plug and Play](/azure/iot-pnp).

Deberá completar las siguientes tareas:

* Instalar un conjunto de herramientas de desarrollo insertado para programar el dispositivo MXCHIP DevKit en C
* Compilar una imagen y guardarla en la memoria flash en MXCHIP DevKit
* Usar la CLI de Azure para crear y administrar un centro de Azure IoT al que el dispositivo MXCHIP DevKit se conectará de forma segura
* Usar Azure IoT Explorer para registrar un dispositivo con su centro de IoT, ver las propiedades del dispositivo, ver la telemetría del dispositivo y llamar a comandos directos en el dispositivo

## <a name="prerequisites"></a>Requisitos previos

* Un equipo con Microsoft Windows 10
* Si no tiene una suscripción de Azure, [cree una gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de comenzar.
* [Git](https://git-scm.com/downloads) para clonar el repositorio
* CLI de Azure. Tiene dos opciones para ejecutar comandos de la CLI de Azure en este inicio rápido:
    * Use Azure Cloud Shell, un shell interactivo que ejecuta comandos de la CLI en el explorador. Esta opción se recomienda porque no es necesario instalar nada. Si usa Cloud Shell por primera vez, inicie sesión en [Azure Portal](https://portal.azure.com). Siga los pasos descritos en el [inicio rápido de Cloud Shell](/azure/cloud-shell/quickstart) para **iniciar Cloud Shell** y **seleccionar el entorno Bash**.
    * Opcionalmente, ejecute la CLI de Azure en su equipo local. Si la CLI de Azure ya está instalada, ejecute `az upgrade` para actualizar la CLI y las extensiones a la versión actual. Para instalar la CLI de Azure, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

* [Azure IoT Explorer](https://github.com/Azure/azure-iot-explorer/releases): utilidad multiplataforma para supervisar y administrar Azure IoT 
* Hardware

    * El [kit de desarrollo de IoT para MXCHIP AZ3166](https://aka.ms/iot-devkit) (MXCHIP DevKit)
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
> * [Termite](https://www.compuphase.com/software_termite.htm): supervisión de la salida del puerto serie de los recursos de dispositivos conectados

Para instalar las herramientas:

1. En el Explorador de archivos, vaya a la siguiente ruta de acceso en el repositorio y ejecute el script de instalación denominado *get-toolchain.bat*:

    *getting-started\tools\get-toolchain.bat*

1. Después de la instalación, abra una nueva ventana de consola para reconocer los cambios de configuración realizados con el script de instalación. Use esta consola para realizar el resto de las tareas de programación del inicio rápido. Puede usar la línea de comandos de Windows, PowerShell o Git Bash para Windows.
1. Ejecute el código siguiente para confirmar la instalación de CMake versión 3.14 o posterior.

    ```shell
    cmake --version
    ```

## <a name="create-the-cloud-components"></a>Creación de los componentes de nube

### <a name="create-an-iot-hub"></a>Crear un centro de IoT

Puede usar la CLI de Azure para crear un centro de IoT que controle los eventos y la mensajería del dispositivo.

Para crear un centro de IoT:

1. Inicie la aplicación de la CLI. Para ejecutar los comandos de la CLI en el resto de este inicio rápido, copie la sintaxis del comando, péguela en la aplicación de la CLI, edite los valores de las variables y presione Entrar.
    - Si prefiere usar Cloud Shell, haga clic con el botón derecho en el vínculo de [Cloud Shell](https://shell.azure.com/bash) y seleccione la opción para abrirlo en una nueva pestaña.
    - Si usa la CLI de Azure localmente, inicie la aplicación de consola de la CLI e inicie sesión en la CLI de Azure.

1. Ejecute [az extension add](/cli/azure/extension?view=azure-cli-latest#az_extension_add&preserve-view=true) para instalar o actualizar la extensión *azure-iot* a la versión actual.

    ```azurecli-interactive
    az extension add --upgrade --name azure-iot
    ```

1. Ejecute el comando [az group create](/cli/azure/group#az-group-create) para crear un grupo de recursos. Con el comando siguiente se crea un grupo de recursos denominado *MyResourceGroup* en la ubicación *centralus*.

    > [!NOTE] 
    > También puede establecer un parámetro `location` alternativo. Para ver las ubicaciones disponibles, ejecute [az account list-locations](/cli/azure/account#az-account-list-locations).

    ```azurecli
    az group create --name MyResourceGroup --location centralus
    ```

1. Ejecute el comando [az iot hub create](/cli/azure/iot/hub#az-iot-hub-create) para crear una instancia de IoT Hub. La creación de una instancia de IoT Hub puede tardar unos minutos.

    *YourIotHubName*. reemplace este marcador de posición por el nombre elegido para el centro de IoT. Un nombre de centro de IoT debe ser único globalmente en Azure. Este marcador de posición se usa en el resto de esta guía de inicio rápido para representar el nombre único del centro de IoT.

    El parámetro `--sku F1` crea el centro de IoT en el nivel Gratis. Los centros del nivel Gratis tienen un conjunto de características limitado y se usan para las aplicaciones de prueba de concepto. Para obtener más información sobre los niveles, las características y los precios de IoT Hub, consulte [Precios de Azure IoT Hub](https://azure.microsoft.com/pricing/details/iot-hub). 

    ```azurecli
    az iot hub create --resource-group MyResourceGroup --name {YourIoTHubName} --sku F1 --partition-count 2
    ```

1. Una vez creado el centro de IoT, vea la salida JSON en la consola y copie el valor `hostName` para usarlo en un paso posterior. El valor `hostName` se parece al del ejemplo siguiente:

    `{Your IoT hub name}.azure-devices.net`

### <a name="configure-iot-explorer"></a>Configuración de IoT Explorer

En el resto de este inicio rápido, usará IoT Explorer para registrar un dispositivo en su centro de IoT, ver las propiedades y la telemetría del dispositivo, y enviar comandos al dispositivo. En esta sección, configurará IoT Explorer para conectarse al centro de IoT que acaba de crear y para leer los modelos Plug and Play desde el repositorio de modelos público. 

Para agregar una conexión al centro de IoT:

1. En la aplicación de la CLI, ejecute el comando [az iot hub connection-string show](/cli/azure/iot/hub/connection-string#az_iot_hub_connection_string_show) para obtener la cadena de conexión para el centro de IoT.

    ```azurecli
    az iot hub connection-string  show --hub-name {YourIoTHubName}
    ```

1. Copie la cadena de conexión sin los caracteres de comillas circundantes.
1. En Azure IoT Explorer, seleccione **Centros de IoT** en el menú izquierdo y, a continuación, seleccione **+ Agregar conexión**.
1. Pegue la cadena de conexión en el cuadro **Cadena de conexión**.
1. Seleccione **Guardar**.

    :::image type="content" source="media/quickstart-devkit-mxchip-az3166-iot-hub/iot-explorer-add-connection.png" alt-text="Captura de pantalla de la adición de una conexión en IoT Explorer":::

1. Si la conexión se realiza correctamente, IoT Explorer cambia a la vista **Dispositivos**.

Para agregar el repositorio de modelos público:

1. En IoT Explorer, seleccione **Inicio** para volver a la vista principal.
1. En el menú de la izquierda, seleccione **Configuración de IoT Plug and Play** y, a continuación, seleccione **+Agregar** y elija **Repositorio público** en el menú desplegable.
1. Aparece una entrada para el repositorio de modelos público en `https://devicemodels.azure.com`.

    :::image type="content" source="media/quickstart-devkit-mxchip-az3166-iot-hub/iot-explorer-add-public-repository.png" alt-text="Captura de pantalla de la adición del repositorio de modelos público en IoT Explorer":::

1. Seleccione **Guardar**.

### <a name="register-a-device"></a>Registrar un dispositivo

En esta sección, creará una nueva instancia de dispositivo y la registrará en el centro de IoT que creó. Usará la información de conexión del dispositivo recién registrado para conectar de forma segura el dispositivo físico en una sección posterior.

Para registrar un dispositivo:

1. En la vista principal de IoT Explorer, seleccione **Centros de IoT**.
1. Debería aparecer la conexión que agregó anteriormente. Seleccione **View devices in this hub** (Ver dispositivos en este centro) debajo de las propiedades de conexión.
1. Seleccione **+ Nuevo** y escriba un identificador de dispositivo para el dispositivo. Por ejemplo, *miDispositivo*. Deje todas las demás propiedades tal cual.
1. Seleccione **Crear**.

    :::image type="content" source="media/quickstart-devkit-mxchip-az3166-iot-hub/iot-explorer-device-created.png" alt-text="Captura de pantalla de una identidad de dispositivo de Azure IoT Explorer":::

1. Use los botones de copia para copiar y anotar los campos **Id. de dispositivo** y **Clave principal**.

Antes de continuar con la siguiente sección, confirme que ha copiado los valores siguientes:

* `hostName`
* `deviceId`
* `primaryKey`

## <a name="prepare-the-device"></a>Preparar el dispositivo

Para conectar MXCHIP DevKit a Azure, modificará un archivo de configuración de las opciones de Wi-Fi y Azure IoT, recompilará la imagen y la guardará en la memoria flash en el dispositivo.

### <a name="add-configuration"></a>Adición de configuración

1. Abra el siguiente archivo en un editor de texto:

    *getting-started\MXChip\AZ3166\app\azure_config.h*

1. Comente la siguiente línea cerca de la parte superior del archivo como se muestra:

    ```c
    // #define ENABLE_DPS
    ```

1. Establezca las constantes Wi-Fi en los siguientes valores del entorno local.

    |Nombre invariable|Value|
    |-------------|-----|
    |`WIFI_SSID` |{*el SSID de Wi-Fi*}|
    |`WIFI_PASSWORD` |{*la contraseña de Wi-Fi*}|
    |`WIFI_MODE` |{*uno de los valores de modo Wi-Fi enumerados en el archivo*}|

1. Establezca las constantes de información del dispositivo de Azure IoT en los valores que guardó después de crear los recursos de Azure.

    |Nombre invariable|Value|
    |-------------|-----|
    |`IOT_HUB_HOSTNAME` |{*el valor de hostName del centro de IoT*}|
    |`IOT_DPS_REGISTRATION_ID` |{*el valor de identificador de dispositivo*}|
    |`IOT_DEVICE_SAS_KEY` |{*el valor de la clave principal*}|

1. Guarde y cierre el archivo.

### <a name="build-the-image"></a>Compilación de la imagen

1. En la consola o en el Explorador de archivos, ejecute el script *rebuild.bat* en la siguiente ruta de acceso para compilar la imagen:

    *getting-started\MXChip\AZ3166\tools\rebuild.bat*

2. Una vez finalizada la compilación, confirme que el archivo binario se ha creado en la siguiente ruta de acceso:

    *getting-started\MXChip\AZ3166\build\app\mxchip_azure_iot.bin*

### <a name="flash-the-image"></a>Guardar la imagen en la memoria flash

1. En MXCHIP DevKit, busque el botón **Reset** (Restablecer) y el puerto micro USB. Usará estos componentes en los pasos siguientes. Ambos se resaltan en la siguiente imagen:

    :::image type="content" source="media/quickstart-devkit-mxchip-az3166-iot-hub/mxchip-iot-devkit.png" alt-text="Búsqueda de componentes clave en la placa del kit de desarrollo de MXChip":::

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

    :::image type="content" source="media/quickstart-devkit-mxchip-az3166-iot-hub/termite-settings.png" alt-text="Captura de pantalla de la configuración del puerto serie en la aplicación Termite":::

1. Seleccione Aceptar.
1. Presione el botón **Reset** (Restablecer) en el dispositivo. El botón está rotulado en el dispositivo y se encuentra cerca del conector micro USB.
1. En la aplicación **Termite**, compruebe los siguientes valores de punto de control para confirmar que el dispositivo se ha inicializado y está conectado a Azure IoT.

    ```output
    Starting Azure thread

    Initializing WiFi
        MAC address: C8:93:46:8A:4C:43
        Connecting to SSID 'iot'
    SUCCESS: WiFi connected to iot

    Initializing DHCP
        IP address: 192.168.0.18
        Mask: 255.255.255.0
        Gateway: 192.168.0.1
    SUCCESS: DHCP initialized

    Initializing DNS client
        DNS address: 75.75.75.75
    SUCCESS: DNS client initialized

    Initializing SNTP client
        SNTP server 0.pool.ntp.org
        SNTP IP address: 157.245.166.169
        SNTP time update: Jun 8, 2021 18:16:50.807 UTC
    SUCCESS: SNTP initialized

    Initializing Azure IoT Hub client
        Hub hostname: ***.azure-devices.net
        Device id: mydevice
        Model id: dtmi:azurertos:devkit:gsgmxchip;1
    Connected to IoT Hub
    SUCCESS: Azure IoT Hub client initialized
    ```

Mantenga abierto Termite para supervisar la salida del dispositivo en los pasos siguientes.

## <a name="view-device-properties"></a>Visualización de las propiedades del dispositivo

Puede usar Azure IoT Explorer para ver y administrar las propiedades de los dispositivos. En esta sección y en las secciones siguientes, usará las funcionalidades de Plug and Play expuestas en IoT Explorer para administrar el dispositivo MXCHIP DevKit e interactuar con este. Estas funcionalidades se basan en el modelo de dispositivo publicado para MXCHIP DevKit en el repositorio de modelos público. Configuró IoT Explorer para buscar modelos de dispositivo en este repositorio anteriormente en este inicio rápido. En muchos casos, puede realizar la misma acción sin usar Plug and Play. Para ello, seleccione la misma acción en el menú izquierdo del panel del dispositivo en IoT Explorer. Sin embargo, el uso de Plug and Play suele proporcionar una experiencia mejorada. Esto se debe a que IoT Explorer puede leer el modelo de dispositivo especificado por un dispositivo Plug and Play y presentar información específica de ese dispositivo.  

Para acceder a componentes de IoT Plug and Play del dispositivo en IoT Explorer:

1. En la vista principal de IoT Explorer, seleccione **Centros de IoT** y, a continuación, seleccione **View devices in this hub** (Ver dispositivos en este centro).
1. Seleccione el dispositivo.
1. Seleccione **Componentes de IoT Plug and Play**.
1. Haga clic en **Default component** (Componente predeterminado). IoT Explorer muestra los componentes de IoT Plug and Play implementados en el dispositivo.

    :::image type="content" source="media/quickstart-devkit-mxchip-az3166-iot-hub/iot-explorer-default-component-view.png" alt-text="Captura de pantalla del componente predeterminado MXCHIP DevKit en IoT Explorer":::

1. En la pestaña **Interfaz**, vea el contenido JSON en el campo **Descripción** del modelo de dispositivo. El archivo JSON contiene detalles de configuración de cada uno de los componentes de IoT Plug and Play del modelo de dispositivo.

    Cada pestaña de IoT Explorer corresponde a uno de los componentes de IoT Plug and Play del modelo de dispositivo.

    | Pestaña | Tipo | Nombre | Descripción |
    |---|---|---|---|
    | **Interfaz** | Interfaz | `MXCHIP Getting Started Guide` | Modelo de ejemplo para MXCHIP DevKit |
    | **Propiedades (solo lectura)** | Propiedad | -- | Actualmente, el modelo no tiene ninguna propiedad de solo lectura. |
    | **Propiedades (grabable)** | Propiedad | `telemetryInterval` | Intervalo en que el dispositivo envía telemetría. |
    | **Comandos** | Comando | `setLedState` | Enciende o apaga la luz LED. |
    | **Telemetría** | Telemetría | `temperature` | Temperatura en grados centígrados. |

Para ver las propiedades del dispositivo mediante Azure IoT Explorer:

1. Seleccione la pestaña **Propiedades (solo lectura)** . Actualmente, no hay ninguna propiedad de solo lectura expuesta por el modelo de dispositivo.
1. Seleccione la pestaña **Propiedades (grabable)** . Muestra el intervalo en que se envía la telemetría.
1. Cambie el valor de `telemetryInterval` a *5* y, a continuación, seleccione **Update desired value** (Actualizar valor deseado). El dispositivo ahora usa este intervalo para enviar telemetría.

    :::image type="content" source="media/quickstart-devkit-mxchip-az3166-iot-hub/iot-explorer-set-telemetry-interval.png" alt-text="Captura de pantalla de la configuración del intervalo de telemetría en MXCHIP DevKit en IoT Explorer":::

1. IoT Explorer responde con una notificación. También puede observar la actualización en Termite.
1. Vuelva a establecer el intervalo de telemetría en 10.
 
Para usar la CLI de Azure para ver las propiedades del dispositivo:

1. Ejecute el comando [az iot hub device-identity show](/cli/azure/iot/hub/device-identity#az_iot_hub_device_identity_show).

    ```azurecli
    az iot hub device-identity show --device-id mydevice --hub-name {YourIoTHubName}
    ```

1. Inspeccione las propiedades del dispositivo en la salida de la consola.

## <a name="view-telemetry"></a>Ver datos de telemetría

Con Azure IoT Explorer, puede ver el flujo de telemetría del dispositivo a la nube. Opcionalmente, puede realizar la misma tarea mediante la CLI de Azure.

Para ver la telemetría en Azure IoT Explorer:

1. En el panel **Componentes de IoT Plug and Play** (componente predeterminado) del dispositivo en IoT Explorer, seleccione la pestaña **Telemetría**. Confirme que **Use built-in event hub** (Usar centro de eventos integrado) está establecido en *Sí*.
1. Seleccione **Inicio**.
1. Vea la telemetría a medida que el dispositivo envía mensajes a la nube.

    :::image type="content" source="media/quickstart-devkit-mxchip-az3166-iot-hub/iot-explorer-device-telemetry.png" alt-text="Captura de pantalla de la telemetría del dispositivo en IoT Explorer":::

    > [!NOTE]
    > También puede supervisar la telemetría desde el dispositivo mediante la aplicación Termite.

1. Active la casilla **Mostrar eventos modelados** para ver los eventos en el formato de datos especificado por el modelo de dispositivo.

    :::image type="content" source="media/quickstart-devkit-mxchip-az3166-iot-hub/iot-explorer-show-modeled-events.png" alt-text="Captura de pantalla de eventos de telemetría modelados en IoT Explorer":::

1. Seleccione **Detener** para dejar de recibir eventos.

Para usar la CLI de Azure para ver la telemetría del dispositivo:

1. Ejecute el comando [az iot hub monitor-events](/cli/azure/iot/hub#az_iot_hub_monitor_events). Use los nombres que creó anteriormente en Azure IoT para el dispositivo IoT Hub.

    ```azurecli
    az iot hub monitor-events --device-id mydevice --hub-name {YourIoTHubName}
    ```

1. Vea la salida JSON en la consola.

    ```json
    {
        "event": {
            "origin": "mydevice",
            "module": "",
            "interface": "dtmi:azurertos:devkit:gsgmxchip;1",
            "component": "",
            "payload": "{\"humidity\":41.21,\"temperature\":31.37,\"pressure\":1005.18}"
        }
    }
    ```

1. Seleccione CTRL + C para finalizar la supervisión.


## <a name="call-a-direct-method-on-the-device"></a>Llamar a un método directo en un dispositivo

También puede usar Azure IoT Explorer para llamar a un método directo que haya implementado en el dispositivo. Los métodos directos tienen un nombre y, opcionalmente, pueden tener una carga de JSON, una conexión configurable y un tiempo de espera del método. En esta sección, se llama a un método que le permite apagar o encender una luz LED. Opcionalmente, puede realizar la misma tarea mediante la CLI de Azure.

Para llamar a un método en Azure IoT Explorer:

1. En el panel **Componentes de IoT Plug and Play** (componente predeterminado) del dispositivo en IoT Explorer, seleccione la pestaña **Comandos**.
1. Para el comando **setLedState**, establezca el **estado** en **true**.
1. Seleccione **Enviar comando**. Debería ver una notificación en IoT Explorer y debería activarse la luz LED de usuario amarilla del dispositivo.

    :::image type="content" source="media/quickstart-devkit-mxchip-az3166-iot-hub/iot-explorer-invoke-method.png" alt-text="Captura de pantalla de la llamada al método setLedState en IoT Explorer":::

1. Establezca el **estado** en **false** y, a continuación, seleccione **Enviar comando**. La luz LED de usuario amarilla debe apagarse.
1. Opcionalmente, puede ver la salida en Termite para supervisar el estado de los métodos.

Para usar la CLI de Azure para llamar a un método:

1. Ejecute el comando [az iot hub invoke-device-method](/cli/azure/iot/hub#az_iot_hub_invoke_device_method) y especifique el nombre y la carga del método. Para este método, si se establece `method-payload` en `true`, se enciende la luz LED y, si se establece en `false`, dicha luz se apaga.

    ```azurecli
    az iot hub invoke-device-method --device-id mydevice --method-name setLedState --method-payload true --hub-name {YourIoTHubName}
    ```

    La consola de la CLI muestra el estado de la llamada al método en el dispositivo, donde `204` indica que se ha realizado correctamente.

    ```json
    {
        "payload": {},
        "status": 200
    }    
    ```

1. Compruebe el dispositivo para confirmar el estado de la luz LED.

1. Consulte el terminal de Termite para confirmar los mensajes de salida:

    ```output
    Receive direct method: setLedState
        Payload: true
    LED is turned ON
    Device twin property sent: {"ledState":true}
    ```

## <a name="troubleshoot-and-debug"></a>Solución de problemas y depuración

Si tiene problemas para compilar el código del dispositivo, guardar la imagen en la memoria flash del dispositivo o conectarse, consulte [Solución de problemas](https://github.com/azure-rtos/getting-started/blob/master/docs/troubleshooting.md).

Para depurar la aplicación, consulte [Depuración con Visual Studio Code](https://github.com/azure-rtos/getting-started/blob/master/docs/debugging.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no necesita los recursos de Azure creados en esta guía de inicio rápido, puede usar la CLI de Azure para eliminar el grupo de recursos y todos los recursos que contiene.

> [!IMPORTANT] 
> La eliminación de un grupo de recursos es irreversible. El grupo de recursos y todos los recursos contenidos en él se eliminan permanentemente. Asegúrese de no eliminar por accidente el grupo de recursos o los recursos equivocados.

Para eliminar un grupo de recursos por el nombre:

1. Ejecute el comando [az group delete](/cli/azure/group#az-group-delete). Esto quita el grupo de recursos, el IoT Hub y el registro de dispositivos que ha creado.

    ```azurecli-interactive
    az group delete --name MyResourceGroup
    ```

1. Ejecute el comando [az group list](/cli/azure/group#az-group-list) para confirmar que se ha eliminado el grupo de recursos.  

    ```azurecli-interactive
    az group list
    ```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha compilado una imagen personalizada que contiene código de ejemplo de Azure RTOS y, después, la ha guardado en la memoria flash en el dispositivo MXCHIP DevKit. También ha usado la CLI de Azure o IoT Explorer para crear recursos de Azure, conectar el dispositivo MXCHIP DevKit de forma segura a Azure, ver la telemetría y enviar mensajes.

Como siguiente paso, explore los siguientes artículos para más información sobre el uso de los SDK de dispositivos IoT para conectar dispositivos a Azure IoT. 

> [!div class="nextstepaction"]
> [Conexión de un kit de desarrollo de MXCHIP AZ3166 a IoT Central](quickstart-devkit-mxchip-az3166.md)
> [!div class="nextstepaction"]
> [Conexión de un dispositivo simulado a IoT Central](quickstart-send-telemetry-central.md)
> [!div class="nextstepaction"]
> [Conexión de un dispositivo simulado a IoT Hub](quickstart-send-telemetry-iot-hub.md)

> [!IMPORTANT]
> Azure RTOS proporciona a los OEM componentes para proteger la comunicación y crear código y aislamiento de datos mediante los mecanismos de protección de hardware MCU/MPU subyacentes. Sin embargo, cada OEM es responsable en última instancia de garantizar que el dispositivo cumpla los requisitos de seguridad en constante evolución.

