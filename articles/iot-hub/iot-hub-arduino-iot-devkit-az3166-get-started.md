---
title: Conexión de IoT DevKit AZ3166 a una instancia de Azure IoT Hub
description: Con este tutorial aprenderá a configurar y conectar IoT DevKit AZ3166 a Azure IoT Hub para que envíe datos a la plataforma en la nube de Azure.
author: wesmc7777
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 04/29/2021
ms.author: wesmc
ms.custom:
- mqtt
- 'Role: Cloud Development'
- devx-track-azurecli
ms.openlocfilehash: 4478d915f854a6272bf4d61b08ce692c310bb2c3
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109655130"
---
# <a name="connect-iot-devkit-az3166-to-azure-iot-hub"></a>Conexión de IoT DevKit AZ3166 a Azure IoT Hub

[!INCLUDE [iot-hub-get-started-device-selector](../../includes/iot-hub-get-started-device-selector.md)]

Se puede usar [MXChip IoT DevKit](https://microsoft.github.io/azure-iot-developer-kit/) para desarrollar y crear prototipos de soluciones de Internet de las cosas (IoT) que aprovechen los servicios de Microsoft Azure. El kit incluye una placa compatible con Arduino con periféricos y sensores avanzados, un paquete de placa de código abierto y una [galería de ejemplos](https://microsoft.github.io/azure-iot-developer-kit/docs/projects/) completa.

## <a name="what-you-learn"></a>Conocimientos que adquirirá

* Cómo crear un centro de IoT y registrar un dispositivo para el MXChip IoT DevKit.
* Procedimiento para conectar IoT DevKit con una red Wi-Fi y configurar la cadena de conexión de IoT Hub.
* Procedimiento para enviar los datos de telemetría del sensor de DevKit a IoT Hub.
* Procedimiento para preparar el entorno de desarrollo y desarrollar aplicaciones para IoT DevKit.

¿Aún no tiene un DevKit? Pruebe el [simulador de DevKit](https://azure-samples.github.io/iot-devkit-web-simulator/) o [compre un DevKit](https://aka.ms/iot-devkit-purchase).

Puede encontrar el código fuente de todos los tutoriales de DevKit en la [galería de códigos de ejemplo](/samples/browse/?term=mxchip).

## <a name="what-you-need"></a>Lo que necesita

- Una placa MXChip IoT DevKit con un cable micro-USB. [Obténgalo ahora](https://aka.ms/iot-devkit-purchase).
- Un equipo con Windows 10, macOS 10.10 o versiones posteriores, o Ubuntu 18.04 o versiones posteriores.
- Una suscripción de Azure activa. [Active una cuenta de Microsoft Azure de prueba de 30 días gratis](https://azureinfo.microsoft.com/us-freetrial.html).
[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]
 
## <a name="prepare-the-development-environment"></a>Preparación del entorno de desarrollo

Siga estos pasos para preparar el entorno de desarrollo para DevKit:

#### <a name="install-visual-studio-code-with-azure-iot-tools-extension-package"></a>Instalación de Visual Studio Code con el paquete de extensión de Azure IoT Tools

1. Instale el [IDE de Arduino](https://www.arduino.cc/en/Main/Software). Este IDE proporciona la cadena de herramientas necesaria para compilar y cargar código de Arduino.
    * **Windows**: use la versión de Windows Installer. No lo instale desde la tienda de aplicaciones.
    * **macOS**: arrastre y coloque el archivo **Arduino.app** extraído en la carpeta `/Applications`.
    * **Ubuntu**: descomprímalo en una carpeta, por ejemplo `$HOME/Downloads/arduino-1.8.8`.

2. Instale [Visual Studio Code](https://code.visualstudio.com/), un editor de código fuente multiplataforma con características completas de IntelliSense, finalización de código y compatibilidad con la depuración, así como extensiones completas que se pueden instalar desde Marketplace.

3. Inicie VS Code, busque **Arduino** en el catálogo de extensiones e instálelo. Esta extensión proporciona experiencias mejoradas para el desarrollo en la plataforma Arduino.

    ![Instalación de Arduino](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/install-arduino.png)

4. Busque [Azure IoT Tools](https://aka.ms/azure-iot-tools) en el catálogo de extensiones e instálelo.

    ![Captura de pantalla que muestra Azure IoT Tools en el catálogo de extensiones.](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/install-azure-iot-tools.png)

    También puede copiar y pegar esta dirección URL en una ventana del explorador: `vscode:extension/vsciot-vscode.azure-iot-tools`

    > [!NOTE]
    > El paquete de extensión de Azure IoT Tools contiene [Azure IoT Device Workbench](https://aka.ms/iot-workbench), que se usa para desarrollar y depurar en varios dispositivos de IoT DevKit. La [extensión de Azure IoT Hub](https://aka.ms/iot-toolkit), que también se incluye con el paquete de extensión de Azure IoT Tools, se usa para administrar centros de Azure IoT Hub e interactuar con ellos.

5. Configure VS Code con la configuración de Arduino.

    En Visual Studio Code, haga clic en **Archivo > Preferencias > Configuración** (en macOS, **Código > Preferencias > Configuración**). Luego, haga clic en el icono **Abrir configuración (JSON)** en la esquina superior derecha de la página *Configuración*.

    ![Instalación de Azure IoT Tools](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/user-settings-arduino.png)

    Debe configurarse la ruta de acceso correcta a la instalación de Arduino en VS Code. Agregue las líneas siguientes para configurar Arduino en función de la plataforma y la ruta de acceso de directorio donde instaló el IDE de Arduino: 

    * **Windows**:

        ```json
        "arduino.path": "C:\\Program Files (x86)\\Arduino",
        "arduino.additionalUrls": "https://raw.githubusercontent.com/VSChina/azureiotdevkit_tools/master/package_azureboard_index.json"
        ```

    * **macOS**:

        ```json
        "arduino.path": "/Applications",
        "arduino.additionalUrls": "https://raw.githubusercontent.com/VSChina/azureiotdevkit_tools/master/package_azureboard_index.json"
        ```

    * **Ubuntu**:

        Reemplace el marcador **{username}** que aparece a continuación por el nombre de usuario.

        ```json
        "arduino.path": "/home/{username}/Downloads/arduino-1.8.13",
        "arduino.additionalUrls": "https://raw.githubusercontent.com/VSChina/azureiotdevkit_tools/master/package_azureboard_index.json"
        ```

6. Haga clic en `F1` para abrir la paleta de comandos y escriba y seleccione **Arduino: Board Manager**. Busque **AZ3166** e instale la última versión.

    ![Instalación del SDK de DevKit](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/install-az3166-sdk.png)

#### <a name="install-st-link-drivers"></a>Instalación de los controladores ST-Link

[ST-Link/V2](https://www.st.com/en/development-tools/st-link-v2.html) es la interfaz USB que usa IoT DevKit para comunicarse con la máquina de desarrollo. Deberá instalarlo en Windows para actualizar el código del dispositivo compilado en el DevKit. Siga los pasos específicos del sistema operativo para permitir que la máquina acceda al dispositivo.

* **Windows**: descargue e instale el controlador USB del [sitio web de STMicroelectronics](https://www.st.com/en/development-tools/stsw-link009.html).
* **macOS**: no se necesita ningún controlador para macOS.
* **Ubuntu**: ejecute los comandos en el terminal, cierre la sesión e iníciela de nuevo para que el cambio de grupo surta efecto:

    ```bash
    # Copy the default rules. This grants permission to the group 'plugdev'
    sudo cp ~/.arduino15/packages/AZ3166/tools/openocd/0.10.0/linux/contrib/60-openocd.rules /etc/udev/rules.d/
    sudo udevadm control --reload-rules

    # Add yourself to the group 'plugdev'
    # Logout and log back in for the group to take effect
    sudo usermod -a -G plugdev $(whoami)
    ```

Ahora ya ha terminado con la preparación y configuración de su entorno de desarrollo.

## <a name="prepare-azure-resources"></a>Preparar recursos de Azure

Para este artículo, deberá tener una instancia de IoT Hub creada y un dispositivo registrado para usar el centro. En las subsecciones siguientes se muestra cómo crear los recursos con la CLI de Azure.  

También puede crear una instancia de IoT Hub y registrar un dispositivo dentro de Visual Studio Code mediante las extensiones de Azure IoT Tools. Para obtener más información sobre cómo crear el centro y el dispositivo en VS Code, consulte [Uso de Azure IoT Tools para VS Code](iot-hub-create-use-iot-toolkit.md).

#### <a name="create-an-iot-hub"></a>Crear un centro de IoT

Si aún no ha creado una instancia de IoT Hub, siga los pasos descritos en [Creación de una instancia de IoT Hub mediante la CLI de Azure](iot-hub-create-using-cli.md).

#### <a name="register-a-device"></a>Registrar un dispositivo

Para poder conectarse, se debe registrar un dispositivo para el IoT DevKit en su instancia de IoT Hub. Siga los pasos que se indican a continuación para usar Azure Cloud Shell para registrar un nuevo dispositivo.

1. Ejecute los siguientes comandos en Azure Cloud Shell para crear la identidad del dispositivo.

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

   **AZ3166Device**: nombre del dispositivo que se va a registrar. Use **AZ3166Device** como se muestra en el ejemplo de este artículo. Si elige otro nombre para el dispositivo, puede que tenga que usarlo en todo el artículo y actualizar el nombre del dispositivo en las aplicaciones de ejemplo antes de ejecutarlas.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name YourIoTHubName --device-id AZ3166Device
    ```

   > [!NOTE]
   > Si recibe un error al ejecutar `device-identity`, instale la [Extensión de Azure IoT para la CLI de Azure](https://github.com/Azure/azure-iot-cli-extension/blob/dev/README.md).
   > Ejecute el siguiente comando para agregar la extensión IoT de Microsoft Azure para la CLI de Azure a la instancia de Cloud Shell. La extensión de IoT agrega comandos específicos de IoT Hub, IoT Edge e IoT Device Provisioning Service (DPS) a la CLI de Azure.
   > 
   > ```azurecli-interactive
   > az extension add --name azure-iot
   >  ```
   >
  
1. Ejecute los siguientes comandos en Azure Cloud Shell para obtener la _cadena de conexión del dispositivo_ que acaba de registrar:

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

    ```azurecli-interactive
    az iot hub device-identity connection-string show --hub-name YourIoTHubName --device-id AZ3166Device --output table
    ```

    Anote la cadena de conexión del dispositivo, que se parecerá a esta:

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=AZ3166Device;SharedAccessKey={YourSharedAccessKey}`

    Necesitará este valor más adelante.


## <a name="prepare-your-hardware"></a>Preparar el hardware

Conecte el siguiente hardware al equipo:

* Placa DevKit
* Cable microUSB

![Requisitos de hardware](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/hardware.jpg)

Para conectar el DevKit al equipo, siga estos pasos:

1. Conecte el extremo USB en el equipo.

2. Conecte el extremo microUSB al DevKit.

3. El LED verde de alimentación confirma la conexión.

   ![Conexiones de hardware](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/connect.jpg)

#### <a name="update-firmware"></a>Actualizar el firmware

El firmware del DevKit de introducción (`devkit-getstarted-*.*.*.bin`) se conecta a un punto de conexión específico del dispositivo en la instancia de IoT Hub y envía datos de telemetría sobre temperatura y humedad. 

1. Descargue la versión más reciente del [firmware de GetStarted (devkit-getstarted- *.* .*.bin)](https://github.com/microsoft/devkit-sdk/releases/) para IoT DevKit. En el momento de esta actualización, el nombre de archivo más reciente es **devkit-getstarted-2.0.0.bin**.

1. Asegúrese de que IoT DevKit se conecta a su equipo mediante un USB. Abra el Explorador de archivos y compruebe que hay un dispositivo de almacenamiento masivo USB denominado **AZ3166**.

    ![Abrir el Explorador de Windows](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/az3166-usb.png)

1. Arrastre y coloque el firmware que acaba de descargar en el dispositivo de almacenamiento masivo y parpadeará automáticamente.

    ![Copiar firmware](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/copy-firmware.png)

1. En el DevKit, mantenga presionado el botón **B** y manténgalo presionado. Presione y suelte el botón de **restablecimiento**. Después, suelte el botón **B**. El DevKit entra en modo AP. Para confirmar la acción, en la pantalla se muestra el identificador de red (SSID) del DevKit y la dirección IP del portal de configuración.

    ![Botón de restablecimiento, botón B y SSID](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/wifi-ap.jpg)

    ![Definición del modo AP](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/set-ap-mode.gif)

1. Use un explorador web en otro dispositivo habilitado para Wi-Fi (equipo o teléfono móvil) para conectarse al SSID de IoT DevKit mostrado en el paso anterior. Si se le pide una contraseña, déjela en blanco.

    ![Conectar SSID](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/connect-ssid.png)

1. Abra **192.168.0.1** en el explorador. Seleccione la red Wi-Fi a la que quiera que se conecte IoT DevKit, escriba la contraseña de la red Wi-Fi y luego pegue la cadena de conexión del dispositivo que ha anotado anteriormente. A continuación, haga clic en **Configurar dispositivo**.

    ![Interfaz de usuario de configuración](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/configuration-ui.png)

    > [!NOTE]
    > IoT DevKit solo es compatible con la red de 2,4 GHz. Consulte la sección [Preguntas más frecuentes](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/#wi-fi-configuration) para obtener más información.

1. La cadena de conexión del dispositivo y la información de la red Wi-Fi se almacenarán en IoT DevKit cuando vea la página de resultados.

    ![Resultado de la configuración](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/configuration-ui-result.png)

    > [!NOTE]
    > Una vez configurada la red Wi-Fi, las credenciales se conservarán en el dispositivo para esa conexión, aunque se desconecte el dispositivo.

1. IoT DevKit se reinicia en unos segundos. En la pantalla de DevKit, verá la dirección IP de DevKit seguida por los datos de telemetría (en los que se incluye el valor de temperatura y humedad) con un recuento de mensajes que se envía a Azure IoT Hub.

    ![IP de Wi-Fi](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/wifi-ip.jpg)

    ![Envío de datos](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/sending-data.jpg)

1. Para comprobar los datos de telemetría que se han enviado a Azure, ejecute el siguiente comando en Azure Cloud Shell:

    ```azurecli
    az iot hub monitor-events --hub-name YourIoTHubName --output table
    ```

    Este comando supervisa los mensajes de dispositivo a nube (D2C) enviados a la instancia de IoT Hub.

## <a name="build-your-first-project"></a>Creación de su primer proyecto

#### <a name="open-sample-code-from-sample-gallery"></a>Apertura del código de ejemplo desde la galería de ejemplos

IoT DevKit contiene una galería de ejemplos completa que puede usar para aprender a conectar el DevKit con diversos servicios de Azure.

1. Asegúrese de que IoT DevKit **no está conectado** al equipo. Primero, inicie VS Code y luego conecte DevKit al equipo.

1. Presione `F1` para abrir la paleta de comandos, escriba y seleccione **Azure IoT Device Workbench: Open Examples...** (abrir ejemplos...). A continuación, seleccione **MXChip IoT DevKit** como placa.

1. En la página de ejemplos de IoT Workbench, busque **Get Started** (Inicio) y haga clic en **Open Sample** (Abrir ejemplo). Después, seleccione la ruta de acceso predeterminada para descargar el ejemplo de código.

    ![Abrir ejemplo](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/open-sample.png)


#### <a name="review-the-code"></a>Revisión del código

`GetStarted.ino` es el archivo principal de boceto de Arduino.

![Mensaje D2C](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/code.png)

Para ver cómo se envían los datos de telemetría del dispositivo a Azure IoT Hub, abra el archivo `utility.cpp` de la misma carpeta. Consulte [API Reference](https://microsoft.github.io/azure-iot-developer-kit/docs/apis/arduino-language-reference/) (Referencia de API) para obtener información sobre cómo usar los sensores y los dispositivos periféricos en IoT DevKit.

El `DevKitMQTTClient` que se usa es un contenedor de **iothub_client** de los [SDK y las bibliotecas de Microsoft Azure IoT para C](https://github.com/Azure/azure-iot-sdk-c/tree/master/iothub_client) para interactuar con Azure IoT Hub.


#### <a name="configure-and-compile-device-code"></a>Configuración y compilación del código del dispositivo

1. En la barra de estado de la parte inferior derecha, compruebe que se muestra **MXCHIP AZ3166** como placa seleccionada y que se usa el puerto serie con **STMicroelectronics**.

    ![Selección de la placa y el puerto COM](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/select-com.png)

1. Presione `F1` para abrir la paleta de comandos y escriba y seleccione **Azure IoT Device Workbench: Configure Device Settings...** (Configurar opciones de dispositivo...) y luego **Config Device Connection String** (configuración de la cadena de conexión de dispositivo). A continuación, pegue la cadena de conexión del dispositivo IoT.  

1. En DevKit, mantenga presionado el **botón A**, presione y suelte el botón de **reinicio** y, a continuación, suelte el **botón A**. DevKit entra en modo de configuración y guarda la cadena de conexión.

    ![Cadena de conexión](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/connection-string.png)

1. Haga clic en `F1` de nuevo, escriba y seleccione **Azure IoT Device Workbench: Upload Device Code** (Carga del código de dispositivo). Se inicia la compilación y la carga del código en DevKit.

    ![Carga de Arduino](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/arduino-upload.png)

El DevKit se reinicia y comienza a ejecutar el código. El DevKit comenzará a enviar telemetría al centro.

> [!NOTE]
> Si hay errores o interrupciones, siempre puede volver a ejecutar el comando.


## <a name="use-the-serial-monitor"></a>Uso del Monitor serie

El Monitor serie de VS Code es útil para ver la información de registro enviada desde el código. Esta información de registro se genera mediante la API de `LogInfo()`.  Esto es útil para fines de depuración. 

1. Haga clic en el icono de interruptor de alimentación de la barra de estado para abrir Serial Monitor:

    ![Monitor de serie](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/serial-monitor.png)

2. La aplicación de ejemplo se ejecuta correctamente si ve los siguientes resultados:

    * El monitor serie muestra el mensaje enviado a IoT Hub.
    * El LED del MXChip IoT DevKit parpadea.
    
    ![Salida del monitor serie](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/result-serial-output.png)
    
    > [!NOTE]
    > Es posible que se produzca un error durante las pruebas en las que el LED no parpadee, Azure Portal no muestre los datos de entrada del dispositivo, pero en la pantalla OLED del dispositivo se muestre **En ejecución...** . Para resolver el problema, en Azure Portal, vaya al dispositivo en el centro de IoT y envíe un mensaje al dispositivo. Si ve la siguiente respuesta en el monitor serie en VS Code, es posible que la comunicación directa desde el dispositivo esté bloqueada en el nivel del enrutador. Compruebe las reglas de firewall y de enrutador configuradas para los dispositivos que se conectan. Además, asegúrese de que el puerto de salida 1833 esté abierto.
    > 
    > ERROR: mqtt_client.c (ln 454): Error: error al abrir la conexión al punto de conexión  
    > INFO: >>>Estado de conexión: desconectado  
    > ERROR: tlsio_mbedtls.c (ln 604): No se pudo abrir la E/S subyacente  
    > ERROR: mqtt_client.c (ln 1042): Error: error de io_open  
    > ERROR: iothubtransport_mqtt_common. c (ln 2283): error al conectar con la dirección atcsliothub.azure-devices.net.  
    > INFO: >>> Vuelva a conectarse.  
    > INFO: Versión de IoThub: 1.3.6  

3. En DevKit, mantenga presionado el botón **A**, presione y suelte el botón **Reinicio** y, a continuación, suelte el botón **A**. El DevKit entrará en modo de configuración. Se muestra una lista completa de los comandos para el modo de configuración en la ventana de salida del Monitor serie de VS Code.

    ```output
    ************************************************
    ** MXChip - Microsoft IoT Developer Kit **
    ************************************************
    Configuration console:
     - help: Help document.
     - version: System version.
     - exit: Exit and reboot.
     - scan: Scan Wi-Fi AP.
     - set_wifissid: Set Wi-Fi SSID.
     - set_wifipwd: Set Wi-Fi password.
     - set_az_iothub: Set IoT Hub device connection string.
     - set_dps_uds: Set DPS Unique Device Secret (UDS) for X.509 certificates..
     - set_az_iotdps: Set DPS Symmetric Key. Format: "DPSEndpoint=global.azure-devices-provisioning.net;IdScope=XXX;DeviceId=XXX;SymmetricKey=XXX".
     - enable_secure: Enable secure channel between AZ3166 and secure chip.
     ```

    Este modo admite comandos como cambiar la cadena de conexión del dispositivo de IoT Hub que quiere usar. Para enviar comandos de texto en el Monitor serie, presione `F1` y elija **Arduino: Send Text to Serial Port** (Arduino: enviar texto al puerto serie).

4. En DevKit, presione y suelte el botón **Reinicio**. Esto reinicia el dispositivo para que comience a enviar telemetría de nuevo.
    
## <a name="use-vs-code-to-view-hub-telemetry"></a>Uso de VS Code para ver la telemetría del centro

Al final de la sección [Preparar el hardware](#prepare-your-hardware), usó la CLI de Azure para supervisar los mensajes del dispositivo a la nube (D2C) en la instancia de IoT Hub. También puede supervisar los mensajes de dispositivo a nube (D2C) en VS Code mediante [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) y busque la instancia de IoT Hub que creó.

    ![Portal de Azure](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/azure-iot-hub-portal.png)

1. En el panel **Directivas de acceso compartido**, haga clic en la directiva **iothubowner** y, luego, anote la cadena de conexión de la instancia de IoT Hub.

    ![Cadena de conexión de Azure IoT Hub](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/azure-portal-conn-string.png)

1. En VS Code, haga clic en `F1`, escriba y seleccione **Azure IoT Hub: Set IoT Hub Connection String** (Establecimiento de la cadena de conexión de IoT Hub). Pegue la cadena de conexión de IoT Hub en el campo.

    ![Establecimiento de la cadena de conexión de Azure IoT Hub](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/set-iothub-connection-string.png)

1. Expanda el panel **AZURE IOT HUB** de la izquierda, haga clic con el botón derecho en el nombre del dispositivo que ha creado y seleccione **Start Monitoring Built-in Event Endpoint** (Iniciar la supervisión del punto de conexión de eventos integrado).

    ![Supervisión de mensajes D2C](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/monitor-d2c.png)

    Si se le pide un punto de conexión integrado, use el **Punto de conexión compatible con el centro de eventos** que se muestra al hacer clic en **Puntos de conexión integrados** para la instancia de IoT Hub en Azure Portal. 

1. En el panel **OUTPUT** (SALIDA), puede ver los mensajes D2C entrantes en IoT Hub. 

    Asegúrese de que la salida se filtre correctamente al seleccionar **Azure IoT Hub**. Este filtrado se puede usar para cambiar entre la supervisión del centro y la salida del Monitor serie.

    ![Captura de pantalla que muestra los mensajes D2C entrantes en IoT Hub.](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/d2c-output.png)


## <a name="problems-and-feedback"></a>Problemas y comentarios

Si tiene problemas, puede buscar una solución en las [preguntas frecuentes de IoT DevKit](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/) o ponerse en contacto con nosotros desde [Gitter](https://gitter.im/Microsoft/azure-iot-developer-kit). Se pueden también enviar comentarios al dejar un comentario en esta página.

## <a name="next-steps"></a>Pasos siguientes

Ha conectado correctamente un MXChip IoT DevKit a IoT Hub y ha enviado los datos del sensor capturados a IoT Hub.

[!INCLUDE [iot-hub-get-started-az3166-next-steps](../../includes/iot-hub-get-started-az3166-next-steps.md)]
