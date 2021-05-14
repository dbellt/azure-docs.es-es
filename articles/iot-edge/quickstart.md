---
title: Inicio rápido para crear un dispositivo de Azure IoT Edge en Windows | Microsoft Docs
description: En esta guía de inicio rápido, obtenga información sobre cómo crear un dispositivo IoT Edge y luego implementar código creado previamente de manera remota desde Azure Portal.
author: rsameser
manager: kgremban
ms.author: riameser
ms.date: 01/20/2021
ms.topic: quickstart
ms.service: iot-edge
services: iot-edge
ms.custom: mvc, devx-track-azurecli
monikerRange: =iotedge-2018-06
ms.openlocfilehash: 9f0562d4471ac1129bf9bc7ecfee058cddac7c61
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107533136"
---
# <a name="quickstart-deploy-your-first-iot-edge-module-to-a-windows-device-preview"></a>Inicio rápido: Implementación del primer módulo de IoT Edge en un dispositivo Windows (versión preliminar)

[!INCLUDE [iot-edge-version-201806](../../includes/iot-edge-version-201806.md)]

Pruebe Azure IoT Edge en este inicio rápido mediante la implementación de código en contenedor en un dispositivo IoT Edge para Linux en Windows. IoT Edge permite administrar de forma remota el código de los dispositivos para que pueda enviar más cargas de trabajo al perímetro. Para este inicio rápido, es aconsejable que use su propio dispositivo para ver lo fácil que es utilizar Azure IoT Edge para Linux en Windows.

En esta guía de inicio rápido, aprenderá a hacer lo siguiente:

* Crear un Centro de IoT.
* Registrar un dispositivo IoT Edge en su instancia de IoT Hub.
* Instalar e iniciar Azure IoT Edge para Linux en el runtime de Windows del dispositivo.
* Implementar un módulo de forma remota en un dispositivo IoT Edge y enviar datos de telemetría.

![Diagrama que muestra la arquitectura de este inicio rápido para el dispositivo y la nube.](./media/quickstart/install-edge-full.png)

En este inicio rápido se explica cómo configurar Azure IoT Edge para Linux en un dispositivo Windows. Después, podrá implementar un módulo de Azure Portal en el dispositivo. El módulo que utilizará es un sensor simulado que genera datos de temperatura, humedad y presión. Otros tutoriales de Azure IoT Edge se basan en el trabajo que se realiza aquí mediante la implementación de módulos que analizan los datos simulados para obtener información empresarial.

Si no tiene una suscripción activa a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de comenzar.

>[!NOTE]
>IoT Edge para Linux en Windows se encuentra en [versión preliminar pública](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Requisitos previos

Prepare el entorno para la CLI de Azure.

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

Cree un grupo de recursos en la nube para administrar todos los recursos que va a usar en este inicio rápido.

   ```azurecli-interactive
   az group create --name IoTEdgeResources --location westus2
   ```

Asegúrese de que el dispositivo de IoT Edge cumpla con los requisitos siguientes:

* Ediciones
  * Windows 10, versión 1809 o posterior; compilación 17763 o posterior
    * Professional, Enterprise, IoT Enterprise
  * Windows Server 2019, compilación 17763 o posterior

* Requisitos de hardware
  * Memoria libre mínima: 1 GB
  * Espacio libre en disco mínimo: 10 GB

>[!NOTE]
>En este inicio rápido se utiliza Windows Admin Center para crear una implementación de IoT Edge para Linux en Windows. También puede emplear PowerShell. Si desea usar PowerShell para crear la implementación, siga los pasos de la guía paso a paso sobre [instalación y aprovisionamiento de Azure IoT Edge para Linux en un dispositivo Windows](how-to-install-iot-edge-on-windows.md).

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

Empiece por crear un centro de IoT con la CLI de Azure.

![Diagrama que muestra el paso para crear un centro de IoT.](./media/quickstart/create-iot-hub.png)

El nivel gratuito de Azure IoT Hub funciona para este inicio rápido. Si ha usado IoT Hub en el pasado y ya tiene un centro creado, puede usarlo.

El código siguiente crea un centro **F1** gratis en el grupo de recursos `IoTEdgeResources`. Reemplace `{hub_name}` por un nombre único para su centro de IoT. La creación de una instancia de IoT Hub puede tardar unos minutos.

```azurecli-interactive
az iot hub create --resource-group IoTEdgeResources --name {hub_name} --sku F1 --partition-count 2
```

Si se produce un error porque ya hay un centro gratuito en la suscripción, cambie la SKU a `S1`. Si recibe un error que le indica que el nombre del centro de IoT no está disponible, es que alguien más ya tiene un centro con ese nombre. Pruebe con uno nuevo.

## <a name="register-an-iot-edge-device"></a>Registro de un dispositivo de IoT Edge

Registre un dispositivo de IoT Edge con la instancia de IoT Hub recién creada.

![Diagrama que muestra el paso para registrar un dispositivo con una identidad de un centro de IoT.](./media/quickstart/register-device.png)

Cree una identidad para el dispositivo simulado, con el fin de que pueda comunicarse con su centro de IoT. La identidad del dispositivo reside en la nube, y se usa una cadena de conexión de dispositivo única para asociar un dispositivo físico a una identidad de dispositivo.

Los dispositivos de IoT Edge se comportan y se pueden administrar de manera diferente a los dispositivos de IoT típicos. Utilice la marca `--edge-enabled` para declarar que esta identidad es para un dispositivo IoT Edge.

1. En Azure Cloud Shell, escriba el comando siguiente para crear un dispositivo llamado **myEdgeDevice** en el centro.

     ```azurecli-interactive
     az iot hub device-identity create --device-id myEdgeDevice --edge-enabled --hub-name {hub_name}
     ```

     Si recibe un error acerca de las claves de directiva `iothubowner`, asegúrese de que Cloud Shell ejecuta la versión más reciente de la extensión Azure IoT.

1. Vea la cadena de conexión del dispositivo, que vincula el dispositivo físico con su identidad en IoT Hub. Contiene el nombre del centro de IoT, el nombre del dispositivo y una clave compartida que autentica las conexiones entre los dos.

     ```azurecli-interactive
     az iot hub device-identity connection-string show --device-id myEdgeDevice --hub-name {hub_name}
     ```

1. Copie el valor de la clave `connectionString` desde la salida JSON y guárdelo. Este valor es la cadena de conexión del dispositivo. Lo utilizará para configurar el tiempo de ejecución de IoT Edge en la sección siguiente.

     ![Captura de pantalla que muestra la salida connectionString en Cloud Shell.](./media/quickstart/retrieve-connection-string.png)

## <a name="install-and-start-the-iot-edge-runtime"></a>Instale e inicie el runtime de IoT Edge

Instale Azure IoT Edge para Linux en Windows en el dispositivo y configúrelo con la cadena de conexión del dispositivo.

![Diagrama que muestra el paso para iniciar el tiempo de ejecución de IoT Edge.](./media/quickstart/start-runtime.png)

1. [Descargue Windows Admin Center](https://aka.ms/wacdownload).

1. Siga las indicaciones del asistente para la instalación a fin de configurar Windows Admin Center en el dispositivo.

1. Abra Windows Admin Center.

1. En la esquina superior derecha, seleccione el icono de engranaje **Settings** (Configuración) y, a continuación, **Extensions** (Extensiones).

1. En la pestaña **Feeds** (Fuentes), seleccione **Add** (Agregar).

1. Escriba `https://aka.ms/wac-insiders-feed` en el cuadro de texto y, después, seleccione **Add** (Agregar).

1. Una vez agregada la fuente, vaya a la pestaña **Available extensions** (Extensiones disponibles) y espere a que se actualice la lista de extensiones.

1. En la lista **Available extensions** (Extensiones disponibles), seleccione **Azure IoT Edge**.

1. Instale la extensión.

1. Una vez instalada la extensión, seleccione **Windows Admin Center** en la esquina superior izquierda para ir a la página principal del panel.

     La conexión **localhost** representa el equipo en el que se ejecuta Windows Admin Center.

     :::image type="content" source="media/quickstart/windows-admin-center-start-page.png" alt-text="Captura de pantalla de la página de inicio de Windows Admin Center":::

1. Seleccione **Agregar**.

     :::image type="content" source="media/quickstart/windows-admin-center-start-page-add.png" alt-text="Captura de pantalla que muestra la selección del botón Add (Agregar) en Windows Admin Center.":::

1. En el icono de Azure IoT Edge, seleccione **Create new** (Crear nuevo) para iniciar el asistente para la instalación.

     :::image type="content" source="media/quickstart/select-tile-screen.png" alt-text="Captura de pantalla que muestra la creación de una nueva implementación en el icono de Azure IoT Edge.":::

1. Continúe con el asistente para la instalación a fin de aceptar los Términos de licencia del software de Microsoft y, a continuación, seleccione **Next** (Siguiente).

     :::image type="content" source="media/quickstart/wizard-welcome-screen.png" alt-text="Captura de pantalla que muestra la selección de Next (Siguiente) para continuar con el asistente para la instalación.":::

1. Seleccione **Optional diagnostic data** (Datos de diagnóstico opcionales) y, a continuación, **Next: (Siguiente:) Implementación**. Esta selección proporciona datos de diagnóstico extendidos que ayudan a Microsoft a supervisar y mantener la calidad del servicio.

     :::image type="content" source="media/quickstart/diagnostic-data-screen.png" alt-text="Captura de pantalla que muestra las opciones de datos de diagnóstico.":::

1. En la pantalla **Select target device** (Seleccionar dispositivo de destino), seleccione el dispositivo de destino que desee para comprobar que cumple los requisitos mínimos. En este inicio rápido, vamos a instalar IoT Edge en el dispositivo local, por lo que debe elegir la conexión **localhost**. Si el dispositivo de destino cumple los requisitos, seleccione **Next** (Siguiente) para continuar.

     :::image type="content" source="media/quickstart/wizard-select-target-device-screen.png" alt-text="Captura de pantalla que muestra la lista de dispositivos de destino.":::

1. Seleccione **Next** (Siguiente) para aceptar la configuración predeterminada. En la pantalla de implementación se muestra el proceso de descarga del paquete, la instalación del paquete, la configuración del host y la configuración final de la máquina virtual Linux (VM). Una implementación correcta tiene el siguiente aspecto:

     :::image type="content" source="media/quickstart/wizard-deploy-success-screen.png" alt-text="Captura de pantalla de una implementación correcta.":::

1. Seleccione **Siguiente: Connect** (Siguiente: Conectar) para continuar al paso final para aprovisionar un dispositivo Azure IoT Edge con su identificador de dispositivo desde la instancia de IoT Hub.

1. Pegue la cadena de conexión que ha copiado [anteriormente en este inicio rápido](#register-an-iot-edge-device) en el campo **Device connection string** (Cadena de conexión de dispositivo). Luego, seleccione **Provisioning with the selected method** (Aprovisionar con el método seleccionado).

     :::image type="content" source="media/quickstart/wizard-provision.png" alt-text="Captura de pantalla que muestra la cadena de conexión en el campo de cadena de conexión de dispositivo.":::

1. Una vez completado el aprovisionamiento, seleccione **Finish** (Finalizar) para completar y volver a la pantalla de inicio de Windows Admin Center. Debería ver el dispositivo en la lista como un dispositivo IoT Edge.

     :::image type="content" source="media/quickstart/windows-admin-center-device-screen.png" alt-text="Captura de pantalla que muestra todas las conexiones en Windows Admin Center.":::

1. Seleccione el dispositivo Azure IoT Edge para ver su panel. Debería ver que se han implementado las cargas de trabajo del dispositivo gemelo en Azure IoT Hub. En **IoT Edge Module List** (Lista de módulos de IoT Edge), se debe mostrar un módulo en ejecución, **edgeAgent**, e **IoT Edge Status** (Estado de IoT Edge) debe ser **active (running)** [activo (en ejecución)].

El dispositivo de IoT Edge está ya configurado. Está preparado para ejecutar módulos implementados en la nube.

## <a name="deploy-a-module"></a>Implementación de un módulo

Administre el dispositivo Azure IoT Edge desde la nube para implementar un módulo que envía datos de telemetría a IoT Hub.

![Diagrama que muestra el paso para implementar un módulo.](./media/quickstart/deploy-module.png)

<!--
[!INCLUDE [iot-edge-deploy-module](../../includes/iot-edge-deploy-module.md)]

Include content included below to support versioned steps in Linux quickstart. Can update include file once Windows quickstart supports v1.2
-->

Una de las funcionalidades clave de Azure IoT Edge es la implementación de código en dispositivos de IoT Edge desde la nube. Los *módulos de IoT Edge* son paquetes ejecutables que se implementan como contenedores. En esta sección, va a implementar un módulo pregenerado desde la [sección de módulos de IoT Edge de Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/internet-of-things?page=1&subcategories=iot-edge-modules) directamente desde la instancia de Azure IoT Hub.

El módulo que se implementa en esta sección simula un sensor y envía los datos generados. Este módulo es un fragmento de código útil para empezar a trabajar con IoT Edge porque se pueden usar los datos simulados para desarrollo y pruebas. Si desea ver exactamente lo que hace este módulo, puede ver el [código fuente del sensor de temperatura simulado](https://github.com/Azure/iotedge/blob/027a509549a248647ed41ca7fe1dc508771c8123/edge-modules/SimulatedTemperatureSensor/src/Program.cs).

Siga estos pasos para implementar el primer módulo desde Azure Marketplace.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya al centro de IoT.

1. En el menú de la izquierda, en **Administración automática de dispositivos**, seleccione **IoT Edge**.

1. Seleccione el identificador del dispositivo de destino en la lista de dispositivos.

1. En la barra superior, seleccione **Establecer módulos**.

   ![Captura de pantalla que muestra la selección de Establecer módulos.](./media/quickstart/select-set-modules.png)

1. En **Módulos de IoT Edge**, abra el menú desplegable **Agregar** y, a continuación, seleccione **Módulo de Marketplace**.

   ![Captura de pantalla que muestra el menú desplegable Agregar.](./media/quickstart/add-marketplace-module.png)

1. En **Marketplace de módulos IoT Edge**, busque y seleccione el módulo `Simulated Temperature Sensor`.

   El módulo se agrega a la sección Módulos de IoT Edge con el estado **en ejecución** deseado.

1. Seleccione **Siguiente: Rutas** para continuar con el paso siguiente del asistente.

   ![Captura de pantalla que muestra cómo continuar con el paso siguiente después de agregar el módulo.](./media/quickstart/view-temperature-sensor-next-routes.png)

1. En la pestaña **Rutas**, quite la ruta predeterminada (**route**) y, a continuación, seleccione **Siguiente: Revisar y crear** para continuar al próximo paso del asistente.

   >[!Note]
   >Las rutas se construyen mediante pares de nombre-valor. Debería ver dos rutas en esta página. La ruta predeterminada (**route**) envía todos los mensajes a IoT Hub (que se denomina `$upstream`). Una segunda ruta (**SimulatedTemperatureSensorToIoTHub**) se creó automáticamente al agregar el módulo desde Azure Marketplace. Esta ruta envía todos los mensajes desde el módulo de temperatura simulado a IoT Hub. Puede eliminar la ruta predeterminada porque es redundante en este caso.

   ![Captura de pantalla que muestra cómo quitar la ruta predeterminada y pasar al paso siguiente.](./media/quickstart/delete-route-next-review-create.png)

1. Revise el archivo JSON y luego seleccione **Crear**. El archivo JSON define todos los módulos que se implementan en el dispositivo IoT Edge. Verá el módulo **SimulatedTemperatureSensor** y los dos módulos en tiempo de ejecución, **edgeAgent** y **edgeHub**.

   >[!Note]
   >Cuando se envía una implementación nueva a un dispositivo IoT Edge, no se inserta nada en el dispositivo. En lugar de eso, el dispositivo consulta a IoT Hub de manera periódica para comprobar cualquier instrucción nueva. Si el dispositivo encuentra un manifiesto de implementación actualizado, usa la información sobre la nueva implementación para extraer las imágenes del módulo de la nube y, después, comienza a ejecutar localmente los módulos. Este proceso puede tardar unos minutos.

1. Después de crear los detalles de la implementación del módulo, el asistente lo lleva nuevamente a la página de detalles del dispositivo. Vea el estado de implementación en la pestaña **Módulos**.

   Deben aparecer tres módulos: **$edgeAgent**, **$edgeHub** y **SimulatedTemperatureSensor**. Si se muestra **SÍ** para uno o varios de los módulos en **ESPECIFICADO EN LA IMPLEMENTACIÓN** pero no en **NOTIFICADO POR EL DISPOSITIVO**, significa que el dispositivo IoT Edge todavía los está iniciando. Espere unos minutos y actualice la página.

   ![Captura de pantalla que muestra el sensor de temperatura simulado en la lista de módulos implementados.](./media/quickstart/view-deployed-modules.png)

## <a name="view-the-generated-data"></a>Visualización de los datos generados

En esta guía de inicio rápido, ha creado un nuevo dispositivo de IoT Edge y ha instalado el runtime de IoT Edge en él. Luego, ha usado Azure Portal para implementar un módulo de IoT Edge para que se ejecute en el dispositivo sin tener que realizar cambios en el propio dispositivo.

El módulo que ha insertado genera los datos del entorno de ejemplo que puede usar posteriormente para las pruebas. El sensor simulado está supervisando una máquina y el entorno alrededor de esta. Por ejemplo, este sensor podría estar en una sala de servidores, en una fábrica o en un aerogenerador. Los mensajes que envía incluyen la temperatura y la humedad ambientales, la temperatura y la presión de la máquina y una marca de tiempo. Los tutoriales de IoT Edge usan los datos creados por este módulo como datos de prueba con fines de análisis.

En el shell de comandos de Windows Admin Center, confirme que el módulo que ha implementado desde la nube se ejecuta en el dispositivo IoT Edge.

1. Conéctese al dispositivo IoT Edge recién creado.

     :::image type="content" source="media/quickstart/connect-edge-screen.png" alt-text="Captura de pantalla que muestra la selección de Connect (Conectar) en Windows Admin Center.":::

     En la página **Overview** (Información general), verá **IoT Edge Module List** (Lista de módulos de IoT Edge ) e **IoT Edge Status** (Estado de IoT Edge). Puede ver los módulos que se han implementado y el estado del dispositivo.  

1. En **Tools** (Herramientas), seleccione **Command Shell** (Shell de comandos). El shell de comandos es un terminal con PowerShell que usa Secure Shell (SSH) automáticamente para conectarse a la máquina virtual Linux del dispositivo Azure IoT Edge en un PC Windows.

     :::image type="content" source="media/quickstart/command-shell-screen.png" alt-text="Captura de pantalla en la que se muestra cómo abrir el shell de comandos.":::

1. Para comprobar los tres módulos del dispositivo, ejecute el siguiente comando Bash:

     ```bash
     sudo iotedge list
     ```

    :::image type="content" source="media/quickstart/iotedge-list-screen.png" alt-text="Captura de pantalla que muestra la salida de IoT Edge del shell de comandos.":::

1. Vea los mensajes que se envían desde el módulo de sensor de temperatura a la nube.

     ```bash
     iotedge logs SimulatedTemperatureSensor -f
     ```

    >[!Important]
    >Los comandos de IoT Edge distinguen las mayúsculas de las minúsculas en los nombres de los módulos.

    :::image type="content" source="media/quickstart/temperature-sensor-screen.png" alt-text="Captura de pantalla que muestra la lista de mensajes enviados desde el módulo a la nube.":::

También puede utilizar la [extensión de Azure IoT Hub para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit) para ver los mensajes que llegan a su centro de IoT.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si desea continuar con los tutoriales de IoT Edge, omita este paso. Puede utilizar el dispositivo que ha registrado y configurado en este inicio rápido. En caso contrario, puede eliminar los recursos de Azure que creó para evitar gastos.

Si ha creado una máquina virtual y un centro de IoT en un nuevo grupo de recursos, puede eliminar dicho grupo y todos los recursos asociados. Si no desea eliminar todo el grupo, puede eliminar recursos individuales en su lugar.

> [!IMPORTANT]
> Compruebe el contenido del grupo de recursos para asegurarse de que no haya nada que desee conservar. La eliminación de un grupo de recursos es irreversible.

Utilice el comando siguiente para quitar el grupo **IoTEdgeResources**. La eliminación puede tardar unos minutos.

```azurecli-interactive
az group delete --name IoTEdgeResources
```

Puede confirmar que se ha eliminado el grupo de recursos mediante este comando para ver la lista de grupos de recursos.

```azurecli-interactive
az group list
```

### <a name="remove-azure-iot-edge-for-linux-on-windows"></a>Eliminación de Azure IoT Edge para Linux en Windows

Utilice la extensión del panel en Windows Admin Center a fin de desinstalar Azure IoT Edge para Linux en Windows.

1. Conéctese al dispositivo IoT Edge en Windows Admin Center. Se carga la extensión de la herramienta del panel de Azure.

1. Seleccione **Desinstalar**. Después de quitar Azure IoT Edge, Windows Admin Center quita la entrada de conexión de dispositivo de Azure IoT Edge de la página **Inicio**.

>[!Note]
>Otra forma de quitar Azure IoT Edge del sistema Windows es seleccionar **Inicio** > **Configuración** > **Aplicaciones** > **Azure IoT Edge** > **Desinstalar** en el dispositivo IoT Edge. Este método eliminará Azure IoT Edge del dispositivo IoT Edge, pero dejará la conexión atrás en Windows Admin Center. Para completar la eliminación, desinstale también Windows Admin Center en el menú **Settings** (Configuración).

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado un dispositivo IoT Edge y ha usado la interfaz en la nube de Azure IoT Edge para implementar el código en el dispositivo. Ahora tiene un dispositivo de prueba que genera datos sin procesar acerca de su entorno.

A continuación, configure el entorno de desarrollo local para que pueda empezar a crear módulos IoT Edge que ejecutan su lógica de negocios.

> [!div class="nextstepaction"]
> [Empezar a desarrollar módulos de IoT Edge](tutorial-develop-for-linux.md)
