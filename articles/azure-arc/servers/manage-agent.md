---
title: Administración del agente de servidores habilitados para Azure Arc
description: En este artículo se describen las diferentes tareas de administración que normalmente realizará durante el ciclo de vida del agente de Connected Machine de los servidores habilitados para Azure Arc.
ms.date: 04/27/2021
ms.topic: conceptual
ms.openlocfilehash: 25cd997bc1b90da07fd9c463f0097c7bdf53b885
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108076624"
---
# <a name="managing-and-maintaining-the-connected-machine-agent"></a>Administración y mantenimiento del agente de Connected Machine

Después de la implementación inicial del agente de Connected Machine de los servidores habilitados para Azure Arc en Windows o Linux, debe volver a configurar el agente, actualizarlo o quitarlo del equipo. Puede administrar fácilmente estas tareas de mantenimiento rutinarias manualmente o de manera automática, lo que reduce los errores de funcionamiento y los gastos.

## <a name="before-uninstalling-agent"></a>Antes de desinstalar el agente

Antes de quitar el agente de Connected Machine del servidor habilitado para Arc, tenga en cuenta lo siguiente para evitar incidencias inesperadas o costos agregados en la factura de Azure:

* Si ha implementado extensiones de máquina virtual de Azure en un servidor habilitado y quita el agente de Connected Machine o elimina el recurso que representa el servidor habilitado para Arc en el grupo de recursos, esas extensiones continuarán ejecutándose y llevarán a cabo su funcionamiento normal.

* Si elimina el recurso que representa el servidor habilitado para Arc en el grupo de recursos, pero no desinstala las extensiones de máquina virtual, al volver a registrar la máquina, no podrá administrar las extensiones de máquina virtual instaladas.

En el caso de los servidores o máquinas que ya no quiera administrar con servidores habilitados para Azure Arc, es necesario seguir estos pasos para detener la administración correctamente:

1. Quite las extensiones de máquina virtual de la máquina o el servidor. Debajo se proporcionan los pasos.

2. Desconecte la máquina de Azure Arc con uno de los métodos siguientes:

    * Ejecutar el comando `azcmagent disconnect` en la máquina o el servidor.

    * Del servidor habilitado para Arc registrado y seleccionado en Azure Portal, al seleccionar **Eliminar** en la barra superior.

    * Mediante la [CLI de Azure](../../azure-resource-manager/management/delete-resource-group.md?tabs=azure-cli#delete-resource) o [Azure PowerShell](../../azure-resource-manager/management/delete-resource-group.md?tabs=azure-powershell#delete-resource). En el caso del parámetro `ResourceType`, use `Microsoft.HybridCompute/machines`.

3. [Desinstale el agente](#remove-the-agent) de la máquina o el servidor siguiendo los pasos a continuación.

## <a name="renaming-a-machine"></a>Cambio de nombre de una máquina

Al cambiar el nombre de la máquina Linux o Windows conectada a servidores habilitados para Azure Arc, el nuevo nombre no se reconoce automáticamente porque el nombre del recurso en Azure es inmutable. Como con otros recursos de Azure, tiene que eliminar el recurso y volver a crearlo para usar el nuevo nombre.

En el caso de los servidores habilitados para Arc, antes de cambiar el nombre de la máquina, es necesario quitar las extensiones de VM antes de continuar.

> [!NOTE]
> Aunque las extensiones instaladas continúan ejecutándose y mantienen su funcionamiento normal una vez completado este procedimiento, no podrá administrarlas. Si intenta volver a implementar las extensiones en la máquina, es posible que experimente un comportamiento imprevisible.

> [!WARNING]
> Se recomienda evitar cambiar el nombre de equipo de la máquina y realizar este procedimiento únicamente si es absolutamente necesario.

1. Audite las extensiones de VM instaladas en la máquina y anote su configuración, con la [CLI de Azure](manage-vm-extensions-cli.md#list-extensions-installed) o con [Azure PowerShell](manage-vm-extensions-powershell.md#list-extensions-installed).

2. Quite las extensiones de VM instaladas desde [Azure Portal](manage-vm-extensions-portal.md#uninstall-extension), con la [CLI de Azure](manage-vm-extensions-cli.md#remove-an-installed-extension) o con [Azure PowerShell](manage-vm-extensions-powershell.md#remove-an-installed-extension).

3. Use la herramienta **azcmagent** con el parámetro [Disconnect](manage-agent.md#disconnect) para desconectar la máquina de Azure Arc y eliminar el recurso de la máquina de Azure. Al desconectar la máquina de los servidores habilitados para Arc, no se quita el agente de Connected Machine, y no es necesario quitar el agente como parte de este proceso. Puede realizar esta operación manualmente con la sesión iniciada de forma interactiva, o bien usar la misma entidad de servicio que utilizó para incorporar varios agentes o un [token de acceso](../../active-directory/develop/access-tokens.md) de la Plataforma de identidad de Microsoft. Si no ha usado una entidad de servicio para registrar la máquina con los servidores habilitados para Azure Arc, vea el siguiente [artículo](onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale) para crear una entidad de servicio.

4. Cambie el nombre de equipo de las máquinas.

5. Vuelva a registrar el agente de Connected Machine con los servidores habilitados para Arc. Ejecute la herramienta `azcmagent` con el parámetro [Connect](manage-agent.md#connect) para completar este paso.

6. Vuelva a implementar las extensiones de VM que se implementaron originalmente en la máquina desde los servidores habilitados para Arc. Si ha implementado el agente de Azure Monitor para VM (conclusiones) o el agente de Log Analytics mediante una directiva de Azure, los agentes se vuelven a implementar después del siguiente [ciclo de evaluación](../../governance/policy/how-to/get-compliance-data.md#evaluation-triggers).

## <a name="upgrading-agent"></a>Actualizar el agente

El agente de Azure Connected Machine se actualiza periódicamente para abordar correcciones de errores, mejoras de estabilidad y funcionalidades nuevas. [Azure Advisor](../../advisor/advisor-overview.md) identifica los recursos que no usan la versión más reciente del agente de la máquina y recomienda actualizar a la versión más reciente. Se le notificará cuando seleccione el servidor habilitado para Arc mediante la presentación de un banner en la página **Información general** o cuando acceder a Advisor a través de Azure Portal.

El agente de Azure Connected Machine para Windows y Linux se puede actualizar a la versión más reciente de forma manual o automática según sus necesidades.

En la tabla siguiente se describen los métodos admitidos para la actualización del agente.

| Sistema operativo | Método de actualización |
|------------------|----------------|
| Windows | Manualmente<br> Windows Update |
| Ubuntu | [Apt](https://help.ubuntu.com/lts/serverguide/apt.html) |
| SUSE Linux Enterprise Server | [zypper](https://en.opensuse.org/SDB:Zypper_usage_11.3) |
| RedHat Enterprise, Amazon, CentOS Linux | [yum](https://wiki.centos.org/PackageManagement/Yum) |

### <a name="windows-agent"></a>Agente de Windows

El paquete de actualización para el agente de Connected Machine para Windows está disponible en:

* Microsoft Update

* [Catálogo de Microsoft Update](https://www.catalog.update.microsoft.com/Home.aspx)

* [Paquete de Windows Installer con el agente](https://aka.ms/AzureConnectedMachineAgent) del centro de descarga de Microsoft.

El agente se puede actualizar a partir de una variedad de métodos para admitir el proceso de administración de actualizaciones de software. Aparte de en Microsoft Update, también se puede ejecutar manualmente desde el símbolo del sistema, un script u otra solución de automatización, o desde el asistente para la interfaz de usuario ejecutando `AzureConnectedMachine.msi`.

> [!NOTE]
> * Para actualizar el agente, debe tener permisos de *administrador*.
> * Para actualizar manualmente, primero debe descargar y copiar el paquete del instalador en una carpeta en el servidor de destino o desde una carpeta de red compartida. 

Si no está familiarizado con las opciones de la línea de comandos para los paquetes de Windows Installer, consulte [Opciones de la línea de comandos estándar de msiexec](/windows/win32/msi/standard-installer-command-line-options) y [Opciones de la línea de comandos de msiexec](/windows/win32/msi/command-line-options).

#### <a name="to-upgrade-using-the-setup-wizard"></a>Para actualizar mediante el Asistente de instalación

1. Inicie sesión en el equipo con una cuenta con derechos administrativos.

2. Ejecute **AzureConnectedMachineAgent.msi** para iniciar el Asistente para la instalación.

El Asistente para la instalación detecta si existe una versión anterior y, a continuación, actualiza el agente automáticamente. Cuando se complete la actualización, el Asistente para la instalación se cerrará automáticamente.

#### <a name="to-upgrade-from-the-command-line"></a>Actualizar desde la línea de comandos

1. Inicie sesión en el equipo con una cuenta con derechos administrativos.

2. Para actualizar el agente de forma silenciosa y crear un archivo de registro de instalación en la carpeta `C:\Support\Logs`, ejecute el comando siguiente.

    ```dos
    msiexec.exe /i AzureConnectedMachineAgent.msi /qn /l*v "C:\Support\Logs\Azcmagentupgradesetup.log"
    ```

### <a name="linux-agent"></a>Agente Linux

Para actualizar el agente en una máquina Linux a la versión más reciente, participan dos comandos. Un comando para actualizar el índice de paquetes locales con la lista de los paquetes disponibles más recientes de los repositorios, y un comando para actualizar el paquete local.

Puede descargar el paquete de agente más reciente del [repositorio de paquetes](https://packages.microsoft.com/) de Microsoft.

> [!NOTE]
> Para actualizar el agente, debe tener permisos de acceso *raíz* o una cuenta que tenga derechos elevados que use Sudo.

#### <a name="upgrade-ubuntu"></a>Actualización de Ubuntu

1. Para actualizar el índice de paquetes locales con los últimos cambios realizados en los repositorios, ejecute el siguiente comando:

    ```bash
    apt update
    ```

2. Para actualizar el sistema, ejecute el siguiente comando:

    ```bash
    apt upgrade
    ```

Las acciones del comando [apt](https://help.ubuntu.com/lts/serverguide/apt.html), como la instalación y la eliminación de paquetes, se registran en el archivo de registro `/var/log/dpkg.log`.

#### <a name="upgrade-red-hatcentosamazon-linux"></a>Actualización de Red Hat/CentOS/Amazon Linux

1. Para actualizar el índice de paquetes locales con los últimos cambios realizados en los repositorios, ejecute el siguiente comando:

    ```bash
    yum check-update
    ```

2. Para actualizar el sistema, ejecute el siguiente comando:

    ```bash
    yum update
    ```

Las acciones del comando [yum](https://access.redhat.com/articles/yum-cheat-sheet), como la instalación y la eliminación de paquetes, se registran en el archivo de registro `/var/log/yum.log`. 

#### <a name="upgrade-suse-linux-enterprise"></a>Actualización de SUSE Linux Enterprise

1. Para actualizar el índice de paquetes locales con los últimos cambios realizados en los repositorios, ejecute el siguiente comando:

    ```bash
    zypper refresh
    ```

2. Para actualizar el sistema, ejecute el siguiente comando:

    ```bash
    zypper update
    ```

Las acciones del comando [zypper](https://en.opensuse.org/Portal:Zypper), como la instalación y la eliminación de paquetes, se registran en el archivo de registro `/var/log/zypper.log`.

## <a name="about-the-azcmagent-tool"></a>Acerca de la herramienta Azcmagent

La herramienta Azcmagent (Azcmagent.exe) se usa para configurar el agente Connected Machine de los servidores habilitados para Azure Arc durante la instalación o bien para modificar la configuración inicial del agente después de la instalación. Azcmagent.exe proporciona parámetros de línea de comandos para personalizar el agente y ver su estado:

* **Connect**: para conectar la máquina a Azure Arc

* **Disconnect**: para desconectar la máquina de Azure Arc

* **Show**: para ver el estado del agente y sus propiedades de configuración (nombre del grupo de recursos, identificador de suscripción, versión, etc.), que puede ayudar a solucionar un problema con el agente. Incluya el parámetro `-j` para generar los resultados en formato JSON.

* **Logs**: crea un archivo .zip en el directorio actual que contiene registros para ayudarle a solucionar problemas.

* **Version**: muestra la versión del Agente de Connected Machine.

* **-useStderr**: dirige la salida de error y detallada a stderr. Incluya el parámetro `-json` para generar los resultados en formato JSON.

* **-h o --help**: muestra los parámetros de línea de comandos disponibles

    Por ejemplo, si desea ver ayuda detallada para el parámetro **Connect**, escriba `azcmagent connect -h`. 

* **-v o --verbose**: habilita el registro detallado

Puede realizar una operación **Connect** y **Disconnect** manualmente mientras inicia sesión de forma interactiva, o bien usar la misma entidad de servicio que ha utilizado para incorporar varios agentes o con un [token de acceso](../../active-directory/develop/access-tokens.md) de la Plataforma de identidad de Microsoft. Si no ha usado una entidad de servicio para registrar la máquina con los servidores habilitados para Azure Arc, vea el siguiente [artículo](onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale) para crear una entidad de servicio.

>[!NOTE]
>Debe tener permisos de acceso *raíz* en máquinas Linux para ejecutar **azcmagent**.

### <a name="connect"></a>Conectar

Este parámetro especifica un recurso en Azure Resource Manager que representa la máquina que se crea en Azure. El recurso está en la suscripción y en el grupo de recursos solicitado, y los datos sobre la máquina se almacenan en la región de Azure especificada por la opción `--location`. El nombre predeterminado del recurso es el nombre de host de la máquina, si no se especifica.

Un certificado correspondiente a la identidad asignada por el sistema de esta máquina se descarga entonces y se almacena localmente. Una vez que este paso se completa, el agente de configuración de invitados y Metadata Service de Azure Connected Machine comienzan a sincronizarse con los servidores habilitados para Azure Arc.

Para conectarse con una entidad de servicio, ejecute el siguiente comando:

`azcmagent connect --service-principal-id <serviceprincipalAppID> --service-principal-secret <serviceprincipalPassword> --tenant-id <tenantID> --subscription-id <subscriptionID> --resource-group <ResourceGroupName> --location <resourceLocation>`

Para conectarse mediante un token de acceso, ejecute el siguiente comando:

`azcmagent connect --access-token <> --subscription-id <subscriptionID> --resource-group <ResourceGroupName> --location <resourceLocation>`

Para conectarse con sus credenciales de sesión iniciada elevadas (interactiva), ejecute el siguiente comando:

`azcmagent connect --tenant-id <TenantID> --subscription-id <subscriptionID> --resource-group <ResourceGroupName> --location <resourceLocation>`

### <a name="disconnect"></a>Desconectar

Este parámetro especifica un recurso en Azure Resource Manager que representa la eliminación de la máquina en Azure. Esta acción no quita el agente de la máquina; desinstale el agente por separado. Una vez que la máquina esté desconectada, si desea volver a registrarla con los servidores habilitados para Azure Arc, use `azcmagent connect` para que se cree un recurso nuevo para ella en Azure.

> [!NOTE]
> Si ha implementado una o varias de las extensiones de máquina virtual de Azure en el servidor habilitado para Arc y elimina su registro en Azure, las extensiones seguirán instaladas. Es importante comprender que, según la extensión instalada, está realizando su función de forma activa. Las máquinas que se vayan a retirar o que ya no administren los servidores habilitados para Arc deben tener las extensiones quitadas antes de eliminar su registro de Azure.

Para desconectarse con una entidad de servicio, ejecute el siguiente comando:

`azcmagent disconnect --service-principal-id <serviceprincipalAppID> --service-principal-secret <serviceprincipalPassword> --tenant-id <tenantID>`

Para desconectarse con un token de acceso, ejecute el siguiente comando:

`azcmagent disconnect --access-token <accessToken>`

Para desconectarse con sus credenciales de sesión iniciada elevadas (interactiva), ejecute el siguiente comando:

`azcmagent disconnect`

## <a name="remove-the-agent"></a>Eliminación del agente

Realice uno de los métodos siguientes para desinstalar el agente de Connected Machine de Windows o Linux desde el equipo. Al quitar el agente, no se anula el registro de la máquina con los servidores habilitados para Arc ni se quitan las extensiones de máquina virtual de Azure instaladas. Anule el registro de la máquina y quite las extensiones de máquina virtual instaladas por separado cuando ya no necesite administrar el equipo en Azure, y estos pasos se deben completar antes de desinstalar el agente.

### <a name="windows-agent"></a>Agente de Windows

Los dos métodos siguientes quitan el agente, pero no quitan la carpeta *C:\Archivos de Files\AzureConnectedMachineAgent* de la máquina.

#### <a name="uninstall-from-control-panel"></a>Desinstalar desde el Panel de control

1. Para desinstalar el agente de Windows de la máquina, haga lo siguiente:

    a. Inicie sesión en el equipo con una cuenta que disponga de permisos de administrador.  
    b. En **Panel de control**, seleccione **Programas y características**.  
    c. En **Programas y características**, seleccione **Agente de Azure Connected Machine**, seleccione **Desinstalar** y, después, seleccione **Sí**.  

    >[!NOTE]
    > También puede ejecutar el asistente para la instalación del agente haciendo doble clic en el paquete del instalador **AzureConnectedMachineAgent.msi**.

#### <a name="uninstall-from-the-command-line"></a>Desinstalar desde la línea de comandos

Para desinstalar el agente manualmente desde el símbolo del sistema o para usar un método automatizado, como un script, puede usar el ejemplo siguiente. En primer lugar, debe recuperar el código del producto, que es un GUID que es el identificador principal del paquete de aplicación, del sistema operativo. La desinstalación se realiza mediante la línea de comandos msiexec.exe: `msiexec /x {Product Code}`.

1. Abra el Editor del Registro.

2. En la clave del Registro `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Uninstall`, busque y copie el GUID del código del producto.

3. Después, puede desinstalar el agente mediante Msiexec con los ejemplos siguientes:

   * Desde la línea de comandos, escriba:

       ```dos
       msiexec.exe /x {product code GUID} /qn
       ```

   * Puede realizar los mismos pasos con PowerShell:

       ```powershell
       Get-ChildItem -Path HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall | `
       Get-ItemProperty | `
       Where-Object {$_.DisplayName -eq "Azure Connected Machine Agent"} | `
       ForEach-Object {MsiExec.exe /x "$($_.PsChildName)" /qn}
       ```

### <a name="linux-agent"></a>Agente Linux

> [!NOTE]
> Para desinstalar el agente, debe tener permisos de acceso *raíz* o una cuenta que tenga derechos elevados que use Sudo.

Para desinstalar el agente de Linux, el comando que se va a usar depende del sistema operativo Linux.

- Para Ubuntu ejecute el siguiente comando:

    ```bash
    sudo apt purge azcmagent
    ```

- Para RHEL, CentOS y Amazon Linux, ejecute el siguiente comando:

    ```bash
    sudo yum remove azcmagent
    ```

- Para SLES, ejecute el siguiente comando:

    ```bash
    sudo zypper remove azcmagent
    ```

## <a name="unregister-machine"></a>Anulación del registro de la máquina

Si tiene previsto dejar de administrar la máquina con los servicios de soporte técnico de Azure, siga estos pasos para anular el registro de la máquina con los servidores habilitados para Arc. Puede realizar estos pasos antes o después de haber quitado el agente de Connected Machine de la máquina.

1. Abra los servidores habilitados para Azure Arc. Para ello, vaya a [Azure Portal](https://aka.ms/hybridmachineportal).

2. Seleccione la máquina en la lista, seleccione los puntos suspensivos ( **...** ) y, a continuación, seleccione **Eliminar**.

## <a name="update-or-remove-proxy-settings"></a>Actualización o eliminación de la configuración de proxy

Si quiere configurar el agente para comunicarse con el servicio a través de un servidor proxy o quitar esta configuración después de la implementación, use uno de los métodos siguientes para completar esta tarea.

> [!NOTE]
> Los servidores con ARC habilitado no admiten el uso de una puerta de enlace de [Log Analytics](../../azure-monitor/agents/gateway.md) como proxy para el agente de máquina conectado.
>

### <a name="windows"></a>Windows

Para establecer la variable de entorno del servidor proxy, ejecute el siguiente comando:

```powershell
# If a proxy server is needed, execute these commands with the proxy URL and port.
[Environment]::SetEnvironmentVariable("https_proxy","http://{proxy-url}:{proxy-port}","Machine")
$env:https_proxy = [System.Environment]::GetEnvironmentVariable("https_proxy","Machine")
# For the changes to take effect, the agent service needs to be restarted after the proxy environment variable is set.
Restart-Service -Name himds
```

Para configurar el agente para que detenga la comunicación a través de un servidor proxy, ejecute el siguiente comando para quitar la variable de entorno del servidor proxy y reiniciar el servicio del agente:

```powershell
[Environment]::SetEnvironmentVariable("https_proxy",$null,"Machine")
$env:https_proxy = [System.Environment]::GetEnvironmentVariable("https_proxy","Machine")
# For the changes to take effect, the agent service needs to be restarted after the proxy environment variable removed.
Restart-Service -Name himds
```

### <a name="linux"></a>Linux

Para establecer el servidor proxy, ejecute el siguiente comando desde el directorio en el que descargó el paquete de instalación del agente:

```bash
# Reconfigure the connected machine agent and set the proxy server.
bash ~/Install_linux_azcmagent.sh --proxy "{proxy-url}:{proxy-port}"
```

Para configurar el agente para que detenga la comunicación a través de un servidor proxy, ejecute el siguiente comando para quitar la configuración de proxy:

```bash
sudo azcmagent_proxy remove
```

## <a name="next-steps"></a>Pasos siguientes

* Puede encontrar información sobre la solución de problemas en la guía [Solución de problemas de conexión del agente de Connected Machine](troubleshoot-agent-onboard.md).

* Obtenga información sobre cómo administrar la máquina con [Azure Policy](../../governance/policy/overview.md) para, por ejemplo, la [configuración de invitado](../../governance/policy/concepts/guest-configuration.md) de VM, la comprobación de que la máquina informa al área de trabajo de Log Analytics esperada, la habilitación de la supervisión con [Azure Monitor con máquinas virtuales](../../azure-monitor/vm/vminsights-enable-policy.md) y mucho más.

* Más información sobre el [agente de Log Analytics](../../azure-monitor/agents/log-analytics-agent.md). El agente de Log Analytics para Windows y Linux es necesario si quiere recopilar datos de supervisión del sistema operativo y de las cargas de trabajo y administrarlos con runbooks de Automation o con características como Update Management, o bien mediante otros servicios de Azure, como [Azure Security Center](../../security-center/security-center-introduction.md).