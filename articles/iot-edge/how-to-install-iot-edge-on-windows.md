---
title: Instalación de Azure IoT Edge para Linux en Windows | Microsoft Docs
description: Instrucciones de instalación de Azure IoT Edge en dispositivos Windows
author: kgremban
manager: philmea
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 01/20/2021
ms.author: v-tcassi
monikerRange: =iotedge-2018-06
ms.openlocfilehash: 98b312066294f2c4ed7f9b089e0f8ea1a68fe4be
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106059707"
---
# <a name="install-and-provision-azure-iot-edge-for-linux-on-a-windows-device-preview"></a>Instalación y aprovisionamiento de Azure IoT Edge para Linux en un dispositivo Windows (versión preliminar)

[!INCLUDE [iot-edge-version-201806](../../includes/iot-edge-version-201806.md)]

El entorno de ejecución de Azure IoT Edge es lo que convierte a un dispositivo en un dispositivo IoT Edge. El entorno de ejecución se puede implementar en dispositivos desde la clase PC hasta servidores industriales. Una vez que un dispositivo está configurado con el entorno de ejecución de IoT Edge, puede empezar a implementar lógica de negocios en él desde la nube. Para obtener más información, consulte [Información del entorno de ejecución de Azure IoT Edge y su arquitectura](iot-edge-runtime.md).

Azure IoT Edge para Linux en Windows le permite usar Azure IoT Edge en dispositivos Windows mediante máquinas virtuales Linux. La versión de Linux de Azure IoT Edge y todos los módulos de Linux que se implementen con ella se ejecutan en la máquina virtual. Desde allí, el código y las aplicaciones Windows, así como el entorno de ejecución de Azure IoT Edge y los módulos pueden libremente entre sí.

En este artículo se enumeran los pasos para configurar una instancia de IoT Edge en un dispositivo Windows. En estos pasos se implementa una máquina virtual Linux que contiene el entorno de ejecución de Azure IoT Edge para que se ejecute en el dispositivo Windows y, a continuación, se aprovisiona el dispositivo con su identidad de dispositivo de IoT Hub.

>[!NOTE]
>IoT Edge para Linux en Windows se encuentra en [versión preliminar pública](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
>
>Aunque IoT Edge para Linux en Windows es la experiencia recomendada para usar Azure IoT Edge en un entorno de Windows, los contenedores de Windows siguen estando disponibles. Si prefiere usar contenedores de Windows, consulte la guía paso a paso sobre la [instalación y administración de Azure IoT Edge para Windows](how-to-install-iot-edge-windows-on-windows.md).

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción válida. Si no tiene una [suscripción a Azure](../guides/developer/azure-developer-guide.md#understanding-accounts-subscriptions-and-billing), cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

* Una instancia de [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) de nivel estándar o gratis en Azure.

* Un dispositivo Windows con los siguientes requisitos mínimos del sistema:

  * Windows 10, versión 1809 o posterior; compilación 17763 o posterior
  * Ediciones Pro, Enterprise o Server
  * Memoria libre mínima: 2 GB
  * Espacio libre en disco mínimo: 10 GB
  * Si va a crear una nueva implementación con Windows 10, asegúrese de que habilita Hyper-V. Para obtener más información, vea cómo [instalar Hyper-V en Windows 10](/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v).
  * Si va a crear una nueva implementación con Windows Server, asegúrese de instalar el rol de Hyper-V. Para obtener más información, vea cómo [instalar el rol de Hyper-V en Windows Server](/windows-server/virtualization/hyper-v/get-started/install-the-hyper-v-role-on-windows-server).
  * Si va a crear una nueva implementación con una máquina virtual, asegúrese de configurar la virtualización anidada correctamente. Para obtener más información, consulte la guía de [virtualización anidada](nested-virtualization.md).

* Acceso a Windows Admin Center con la extensión de Azure IoT Edge para Windows Admin Center instalada:

   1. Descargue el [instalador de Windows Admin Center](https://aka.ms/wacdownload).

   1. Ejecute el instalador descargado y siga las indicaciones del Asistente para la instalación para instalar Windows Admin Center. 

   1. Una vez instalado, use un explorador compatible para abrir Windows Admin Center. Entre los exploradores admitidos se incluyen Microsoft Edge (Windows 10, versión 1709 o posterior), Google Chrome y Microsoft Edge Insider.

   1. Al usar Windows Admin Center por primera vez, se le pedirá que seleccione un certificado para usarlo. Seleccione **Cliente de Windows Admin Center** como certificado.

   1. Llegó el momento de instalar la extensión de Azure IoT Edge. Seleccione el icono de engranaje en la parte superior derecha del panel de Windows Admin Center.

      ![Seleccione el icono de engranaje en la parte superior derecha del panel de Windows Admin Center para acceder a la configuración.](./media/how-to-install-iot-edge-on-windows/select-gear-icon.png)

   1. En el menú **Configuración**, en **Puerta de enlace**, seleccione **Extensiones**.

   1. En la pestaña **Extensiones disponibles**, busque **Azure IoT Edge** en la lista de extensiones. Esta dicha opción y seleccione la indicación **Instalar** sobre la lista de extensiones.

   1. Una vez finalizada la instalación, debería ver Azure IoT Edge en la lista de extensiones instaladas en la pestaña **Extensiones instaladas**.

## <a name="choose-your-provisioning-method"></a>Elección del método de aprovisionamiento

Azure IoT Edge para Linux en Windows admite los siguientes métodos de aprovisionamiento:

* Aprovisionamiento manual mediante la cadena de conexión del dispositivo de IoT Edge. Para usar este método, registre el dispositivo y recupere una cadena de conexión con los pasos de [Registro de un nuevo dispositivo Azure IoT Edge en IoT Hub](how-to-register-device.md).
  * Elija la opción de autenticación de clave simétrica, ya que los certificados autofirmados X.509 no son compatibles actualmente con IoT Edge para Linux en Windows.
* Aprovisionamiento automático mediante Device Provisioning Service (DPS) y claves simétricas. Obtenga más información acerca de la [creación y aprovisionamiento de un dispositivo de IoT Edge con DPS y claves simétricas](how-to-auto-provision-symmetric-keys.md).
* Aprovisionamiento automático con certificados X.509 y DPS. Obtenga más información acerca de la [creación y aprovisionamiento de un dispositivo de IoT Edge con DPS y certificados X.509](how-to-auto-provision-x509-certs.md).

El aprovisionamiento manual es más fácil de empezar con algunos dispositivos. Device Provisioning Service es útil para aprovisionar muchos dispositivos.

Si planea usar uno de los métodos de DPS para aprovisionar sus dispositivos, siga los pasos descritos en el artículo correspondiente vinculado anteriormente para crear una instancia de DPS, vincular la instancia de DPS a su instancia de IoT Hub y crear una inscripción de DPS. Puede crear una *inscripción individual* para un único dispositivo o una *inscripción de grupo* para un conjunto de dispositivos. Para obtener más información sobre los tipos de inscripción, visite los [conceptos de Azure IoT Hub Device Provisioning Service](../iot-dps/concepts-service.md#enrollment).

## <a name="create-a-new-deployment"></a>Crear una nueva implementación

Cree su implementación de Azure IoT Edge para Linux en Windows en el dispositivo de destino.

# <a name="windows-admin-center"></a>[Windows Admin Center](#tab/windowsadmincenter)

En la página de inicio de Windows Admin Center, en la lista de conexiones, verá una conexión de host local que representa el equipo en el que se ejecuta Windows Admin Center. Los servidores, equipos o clústeres adicionales que administre también aparecerán aquí.

Puede usar Windows Admin Center para instalar y administrar Azure IoT Edge para Linux en Windows tanto en el dispositivo local como en los dispositivos administrados remotos. En esta guía, la conexión de host local actuará como el dispositivo de destino para la implementación de Azure IoT Edge para Linux en Windows.

Si quiere realizar la implementación en un dispositivo de destino remoto en lugar de en el dispositivo local y no ve el dispositivo de destino deseado en la lista, siga las [instrucciones para agregar el dispositivo](/windows-server/manage/windows-admin-center/use/get-started#connecting-to-managed-nodes-and-clusters).

   ![Panel inicial de Windows Admin Center con el dispositivo de destino en la lista](./media/how-to-install-iot-edge-on-windows/windows-admin-center-initial-dashboard.png)

1. Seleccione **Agregar**.

1. En el panel **Agregar o crear recursos**, busque el elemento **Azure IoT Edge**. Seleccione **Crear nuevo** para instalar una nueva instancia de Azure IoT Edge para Linux en Windows en un dispositivo.

   Si ya tiene una instancia de IoT Edge para Linux en Windows ejecutándose en un dispositivo, puede seleccionar **Agregar** para conectarse a ese dispositivo de IoT Edge existente y administrarlo con Windows Admin Center.

   ![Seleccione Crear nuevo en el elemento Azure IoT Edge de Windows Admin Center](./media/how-to-install-iot-edge-on-windows/resource-creation-tiles.png)

1. Se abrirá el panel **Crear una implementación de Azure IoT Edge para Linux en Windows**. En la pestaña **1. Introducción**, compruebe que el dispositivo de destino cumple los requisitos mínimos y seleccione **Siguiente**.

1. Revise los términos de licencia, marque la casilla **Acepto** y seleccione **Siguiente**.

1. Puede activar o desactivar la opción **Datos de diagnóstico opcionales** en función de sus preferencias.

1. Seleccione **Siguiente: Implementación**.

   ![Seleccione el botón Siguiente: Implementar después de activar o desactivar la opción Datos de diagnóstico opcionales según sus preferencias.](./media/how-to-install-iot-edge-on-windows/select-next-deploy.png)

1. En la pestaña **2. Implementar**, en **Seleccione un dispositivo de destino**, haga clic en su dispositivo para validar que cumple los requisitos mínimos. Una vez que su estado se confirme como compatible, seleccione **Siguiente**.

   ![Seleccione el dispositivo para comprobar que es compatible](./media/how-to-install-iot-edge-on-windows/evaluate-supported-device.png)

1. En la pestaña **Configuración 2.2**, revise los valores de configuración de la implementación. Cuando esté satisfecho con la configuración, seleccione **Siguiente**.

   ![Revisión de los valores de configuración de la implementación](./media/how-to-install-iot-edge-on-windows/default-deployment-configuration-settings.png)

   >[!NOTE]
   >Si usa una máquina virtual de Windows, se recomienda usar un modificador predeterminado en lugar de uno externo para asegurarse de que la máquina virtual de Linux creada en la implementación pueda obtener una dirección IP.
   >
   >El uso de un modificador predeterminado asigna a la máquina virtual de Linux una dirección IP interna. No se puede obtener acceso a esta dirección IP interna desde fuera de la máquina virtual de Windows, pero se puede conectar localmente mientras se inicia sesión en la máquina virtual de Windows.
   >
   >Si está usando Windows Server, tenga en cuenta que Azure IoT Edge para Linux en Windows no admite automáticamente el modificador predeterminado. En el caso de una máquina virtual de Windows Server local, asegúrese de que la máquina virtual de Linux puede obtener una dirección IP a través del modificador externo. En el caso de una máquina virtual de Windows Server en Azure, configure un modificador interno antes de implementar IoT Edge para Linux en Windows.

1. En la pestaña **2.3 Implementación**, puede ver el progreso de la implementación. El proceso completo incluye la descarga del paquete de Azure IoT Edge para Linux en Windows, la instalación del paquete, la configuración del dispositivo host y la configuración de la máquina virtual Linux. Este proceso puede tardar varios minutos en completarse. A continuación se muestra una implementación correcta.

   ![Una implementación correcta mostrará todos los pasos con una marca de verificación verde y la etiqueta "Completado".](./media/how-to-install-iot-edge-on-windows/successful-deployment.png)

Una vez completada la implementación, está listo para aprovisionar el dispositivo. Seleccione **Siguiente: Conectar** para ir a la pestaña **3. Conectar**, que controla el aprovisionamiento de dispositivos de Azure IoT Edge.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Instale IoT Edge para Linux en Windows en el dispositivo de destino si todavía no lo ha hecho.

> [!NOTE]
> En el siguiente proceso de PowerShell se describe cómo crear una implementación de host local de Azure IoT Edge para Linux en Windows. Para crear una implementación en un dispositivo de destino remoto mediante PowerShell, puede usar [PowerShell remoto](/powershell/module/microsoft.powershell.core/about/about_remote) para establecer una conexión a un dispositivo remoto y ejecutar estos comandos de manera remota en ese dispositivo.

1. En una sesión de PowerShell con privilegios elevados, ejecute cada uno de los siguientes comandos para descargar IoT Edge para Linux en Windows.

   ```azurepowershell-interactive
   $msiPath = $([io.Path]::Combine($env:TEMP, 'AzureIoTEdge.msi'))
   $ProgressPreference = 'SilentlyContinue'
   Invoke-WebRequest "https://aka.ms/AzEflowMSI" -OutFile $msiPath
   ```

1. Instale IoT Edge para Linux en Windows en el dispositivo.

   ```azurepowershell-interactive
   Start-Process -Wait msiexec -ArgumentList "/i","$([io.Path]::Combine($env:TEMP, 'AzureIoTEdge.msi'))","/qn"
   ```

   > [!NOTE]
   > Puede especificar una instalación personalizada de IoT Edge para Linux en Windows y directorios VHDX personalizados si agrega los parámetros INSTALLDIR="<FULLY_QUALIFIED_PATH>" y VHDXDIR="<FULLY_QUALIFIED_PATH>" al comando de instalación anterior.

1. Para que la implementación se ejecute correctamente, debe establecer la directiva de ejecución en el dispositivo de destino en `AllSigned` si aún no está establecido de esta manera. Puede comprobar la directiva de ejecución actual en un símbolo del sistema de PowerShell con privilegios elevados con el siguiente comando:

   ```azurepowershell-interactive
   Get-ExecutionPolicy -List
   ```

   Si la directiva de ejecución de `local machine` no es `AllSigned`, puede establecer la directiva de ejecución con el siguiente comando:

   ```azurepowershell-interactive
   Set-ExecutionPolicy -ExecutionPolicy AllSigned -Force
   ```

1. Creación de la implementación de IoT Edge para Linux en Windows.

   ```azurepowershell-interactive
   Deploy-Eflow
   ```

   > [!NOTE]
   > Puede ejecutar este comando sin parámetros u opcionalmente personalizar la implementación con parámetros. Puede consultar [la referencia de scripts de IoT Edge para Linux en Windows PowerShell](reference-iot-edge-for-linux-on-windows-scripts.md#deploy-eflow) para ver el significado de los parámetros y los valores predeterminados.

1. Escriba "Y" para aceptar los términos de licencia.

1. Escriba "O" o "R" para activar o desactivar la opción **Datos de diagnóstico opcionales** en función de sus preferencias. A continuación se muestra una implementación correcta.

   ![Una implementación correcta indicará "implementación correcta" al final de los mensajes.](./media/how-to-install-iot-edge-on-windows/successful-powershell-deployment.png)

Una vez completada la implementación, está listo para aprovisionar el dispositivo.

---

Para aprovisionar el dispositivo, puede usar los siguientes vínculos para ir a la sección del método de aprovisionamiento seleccionado:

* [Opción 1: Aprovisionamiento manual mediante la cadena de conexión del dispositivo de IoT Edge](#option-1-provisioning-manually-using-the-connection-string)
* [Opción 2: Aprovisionamiento automático mediante Device Provisioning Service (DPS) y claves simétricas](#option-2-provisioning-via-dps-using-symmetric-keys)
* [Opción 3: Aprovisionamiento automático con certificados X.509 y DPS](#option-3-provisioning-via-dps-using-x509-certificates)

## <a name="provision-your-device"></a>Aprovisionamiento del dispositivo

Elija un método para aprovisionar el dispositivo y siga las instrucciones de la sección correspondiente. Puede usar Windows Admin Center o una sesión de PowerShell con privilegios elevados para aprovisionar los dispositivos.

### <a name="option-1-provisioning-manually-using-the-connection-string"></a>Opción 1: Aprovisionamiento manual mediante la cadena de conexión

En esta sección se trata el aprovisionamiento manual del dispositivo con la cadena de conexión del dispositivo de Azure IoT Edge.

# <a name="windows-admin-center"></a>[Windows Admin Center](#tab/windowsadmincenter)

1. En el panel **Azure IoT Edge device provisioning** (Aprovisionamiento de dispositivos de Azure IoT Edge), seleccione **Connection String (Manual)** (cadena de conexión [manual]) en el menú desplegable Método de aprovisionamiento.

1. En [Azure Portal](https://ms.portal.azure.com/), vaya a la pestaña **IoT Edge** de IoT Hub.

1. Haga clic en el id. de dispositivo de su dispositivo. Copie el campo **Cadena de conexión principal**.

1. Pegue el valor en el campo Cadena de conexión de dispositivo en Windows Admin Center. A continuación, elija **Provisioning with the selected method** (aprovisionar con el método seleccionado).

   ![Elija Aprovisionar con el método seleccionado después de pegar la cadena de conexión del dispositivo.](./media/how-to-install-iot-edge-on-windows/provisioning-with-selected-method-connection-string.png)

1. Una vez completado el aprovisionamiento, seleccione **Finalizar**. Volverá al panel principal. Ahora debería mostrarse un nuevo dispositivo, cuyo tipo es `IoT Edge Devices`. Puede seleccionar el dispositivo de IoT Edge para conectarse a él. Una vez en la página **Información general** del dispositivo, puede ver los valores de **IoT Edge Module List** (lista de módulos de IoT Edge) y **IoT Edge Status** (estado de IoT Edge) del dispositivo.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

1. En [Azure Portal](https://ms.portal.azure.com/), vaya a la pestaña **IoT Edge** de IoT Hub.

1. Haga clic en el id. de dispositivo de su dispositivo. Copie el campo **Cadena de conexión principal**.

1. Pegue el texto del marcador de posición en el siguiente comando y ejecútelo en una sesión de PowerShell con privilegios elevados en el dispositivo de destino.

   ```azurepowershell-interactive
   Provision-EflowVm -provisioningType manual -devConnString "<CONNECTION_STRING_HERE>"
   ```

---

### <a name="option-2-provisioning-via-dps-using-symmetric-keys"></a>Opción 2: Aprovisionamiento a través de DPS mediante claves simétricas

En esta sección se trata el aprovisionamiento automático del dispositivo mediante DPS y claves simétricas.

# <a name="windows-admin-center"></a>[Windows Admin Center](#tab/windowsadmincenter)

1. En el panel **Azure IoT Edge device provisioning** (Aprovisionamiento de dispositivos de Azure IoT Edge), seleccione **Symmetric Key (DPS)** (clave simétrica [DPS]) en el menú desplegable Método de aprovisionamiento.

1. En [Azure Portal](https://ms.portal.azure.com/), vaya a su instancia de DPS.

1. En la pestaña **Información general**, copie el valor de **Ámbito de id.** Péguelo en el campo de id. de ámbito de Windows Admin Center.

1. En la pestaña **Administrar inscripciones** de Azure Portal, seleccione la inscripción que creó. Copie el valor de **Clave principal** en los detalles de inscripción. Péguelo en el campo de clave simétrica de Windows Admin Center.

1. Proporcione el id. de registro del dispositivo en el campo Id. de registro de Windows Admin Center.

1. Elija **Provisioning with the selected method** (aprovisionar con el método seleccionado).

   ![Elija Aprovisionar con el método seleccionado después de rellenar los campos obligatorios para el aprovisionamiento de claves simétricas.](./media/how-to-install-iot-edge-on-windows/provisioning-with-selected-method-symmetric-key.png)

1. Una vez completado el aprovisionamiento, seleccione **Finalizar**. Volverá al panel principal. Ahora debería mostrarse un nuevo dispositivo, cuyo tipo es `IoT Edge Devices`. Puede seleccionar el dispositivo de IoT Edge para conectarse a él. Una vez en la página **Información general** del dispositivo, puede ver los valores de **IoT Edge Module List** (lista de módulos de IoT Edge) y **IoT Edge Status** (estado de IoT Edge) del dispositivo.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

1. Copie el comando siguiente en un editor de texto. Sustituya el texto del marcador de posición por sus datos como se indica.

   ```azurepowershell-interactive
   Provision-EflowVm -provisioningType symmetric -scopeId <ID_SCOPE_HERE> -registrationId <REGISTRATION_ID_HERE> -symmKey <PRIMARY_KEY_HERE>
   ```

1. En [Azure Portal](https://ms.portal.azure.com/), vaya a su instancia de DPS.

1. En la pestaña **Información general**, copie el valor de **Ámbito de id.** Péguelo sobre el texto de marcador de posición adecuado en el comando.

1. En la pestaña **Administrar inscripciones** de Azure Portal, seleccione la inscripción que creó. Copie el valor de **Clave principal** en los detalles de inscripción. Péguelo sobre el texto de marcador de posición adecuado en el comando.

1. Proporcione el id. de registro del dispositivo para reemplazar el texto de marcador de posición adecuado en el comando.

1. Ejecute el comando en una sesión de PowerShell con privilegios elevados en el dispositivo de destino.

---

### <a name="option-3-provisioning-via-dps-using-x509-certificates"></a>Opción 3: Aprovisionamiento a través de DPS mediante certificados X.509

En esta sección se trata el aprovisionamiento automático del dispositivo mediante DPS y certificados X.509.

# <a name="windows-admin-center"></a>[Windows Admin Center](#tab/windowsadmincenter)

1. En el panel **Azure IoT Edge device provisioning** (Aprovisionamiento de dispositivos de Azure IoT Edge), seleccione **X.509 Certificate (DPS)** (certificado X.509 [DPS]) en el menú desplegable Método de aprovisionamiento.

1. En [Azure Portal](https://ms.portal.azure.com/), vaya a su instancia de DPS.

1. En la pestaña **Información general**, copie el valor de **Ámbito de id.** Péguelo en el campo de id. de ámbito de Windows Admin Center.

1. Proporcione el id. de registro del dispositivo en el campo Id. de registro de Windows Admin Center.

1. Cargue los archivos de clave privada y certificado.

1. Elija **Provisioning with the selected method** (aprovisionar con el método seleccionado).

   ![Elija Aprovisionar con el método seleccionado después de rellenar los campos obligatorios para el aprovisionamiento de certificados X.509.](./media/how-to-install-iot-edge-on-windows/provisioning-with-selected-method-x509-certs.png)

1. Una vez completado el aprovisionamiento, seleccione **Finalizar**. Volverá al panel principal. Ahora debería mostrarse un nuevo dispositivo, cuyo tipo es `IoT Edge Devices`. Puede seleccionar el dispositivo de IoT Edge para conectarse a él. Una vez en la página **Información general** del dispositivo, puede ver los valores de **IoT Edge Module List** (lista de módulos de IoT Edge) y **IoT Edge Status** (estado de IoT Edge) del dispositivo.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

1. Copie el comando siguiente en un editor de texto. Sustituya el texto del marcador de posición por sus datos como se indica.

   ```azurepowershell-interactive
   Provision-EflowVm -provisioningType x509 -scopeId <ID_SCOPE_HERE> -registrationId <REGISTRATION_ID_HERE> -identityCertLocWin <ABSOLUTE_CERT_SOURCE_PATH_ON_WINDOWS_MACHINE> -identityPkLocWin <ABSOLUTE_PRIVATE_KEY_SOURCE_PATH_ON_WINDOWS_MACHINE> -identityCertLocVm <ABSOLUTE_CERT_DEST_PATH_ON_LINUX_MACHINE -identityPkLocVm <ABSOLUTE_PRIVATE_KEY_DEST_PATH_ON_LINUX_MACHINE>
   ```

1. En [Azure Portal](https://ms.portal.azure.com/), vaya a su instancia de DPS.

1. En la pestaña **Información general**, copie el valor de **Ámbito de id.** Péguelo sobre el texto de marcador de posición adecuado en el comando.

1. Proporcione el id. de registro del dispositivo para reemplazar el texto de marcador de posición adecuado en el comando.

1. Reemplace el texto de marcador de posición correspondiente por la ruta de acceso de origen absoluta al archivo de certificado.

1. Reemplace el texto de marcador de posición correspondiente por la ruta de acceso de origen absoluta al archivo de clave privada.

1. Ejecute el comando en una sesión de PowerShell con privilegios elevados en el dispositivo de destino.

---

## <a name="verify-successful-configuration"></a>Comprobación de configuración correcta

Compruebe que IoT Edge para Linux en Windows se haya instalado y configurado correctamente en el dispositivo de IoT Edge.

# <a name="windows-admin-center"></a>[Windows Admin Center](#tab/windowsadmincenter)

1. Seleccione el dispositivo de IoT Edge en la lista de dispositivos conectados de Windows Admin Center para conectarse a él.

1. En la página Información general del dispositivo se muestran algunos datos sobre el dispositivo:

    1. En la sección **IoT Edge Module List** (lista de módulos de IoT Edge) se muestran los módulos en ejecución en el dispositivo. Cuando el servicio de IoT Edge se inicia por primera vez, solo verá la ejecución del módulo **edgeAgent**. El módulo edgeAgent se ejecuta de forma predeterminada y le ayuda a instalar e iniciar todos los módulos adicionales que implemente en el dispositivo.
    1. En la sección **IoT Edge Status** (estado de IoT Edge) se muestra el estado del servicio y debe aparecer como **activo (en ejecución)** .

1. Si necesita solucionar los problemas del servicio de IoT Edge, use la herramienta **shell de comandos** en la página del dispositivo para ingresar mediante ssh (Secure Shell) en la máquina virtual y ejecute los comandos de Linux.

    1. Si necesita solucionar problemas del servicio, recupere los registros del servicio.

       ```bash
       journalctl -u iotedge
       ```

    2. Use la herramienta `check` para comprobar el estado de configuración y conexión del dispositivo.

       ```bash
       sudo iotedge check
       ```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

1. Inicie sesión en la instancia de IoT Edge para Linux en la máquina virtual Windows con el siguiente comando en la sesión de PowerShell:

   ```azurepowershell-interactive
   Ssh-EflowVm
   ```

   >[!NOTE]
   >La única cuenta que se permite a SSH en la máquina virtual es la del usuario que la creó.

1. Una vez que haya iniciado sesión, puede comprobar la lista de módulos de IoT Edge en ejecución con el comando de Linux siguiente:

   ```bash
   iotedge list
   ```

1. Si tiene que solucionar problemas del servicio IoT Edge, use los siguientes comandos de Linux.

    1. Si necesita solucionar problemas del servicio, recupere los registros del servicio.

       ```bash
       journalctl -u iotedge
       ```

    2. Use la herramienta `check` para comprobar el estado de la configuración y la conexión del dispositivo.

       ```bash
       sudo iotedge check
       ```

---

## <a name="next-steps"></a>Pasos siguientes

* Siga [implementando módulos de IoT Edge](how-to-deploy-modules-portal.md) para información sobre cómo implementar módulos en el dispositivo.
* Obtenga información acerca de cómo [administrar certificados en IoT Edge para Linux en máquinas virtuales de Windows](how-to-manage-device-certificates.md) y transferir archivos desde el sistema operativo host a la máquina virtual de Linux.
* Aprenda a [configurar los dispositivos de IoT Edge para que se comuniquen a través de un servidor proxy](how-to-configure-proxy-support.md).
