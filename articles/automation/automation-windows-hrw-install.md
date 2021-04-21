---
title: Implementación de una instancia de Hybrid Runbook Worker de Windows en Azure Automation
description: En este artículo se describe cómo implementar una instancia de Hybrid Runbook Worker que puede usar para ejecutar runbooks en máquinas Windows en su centro de datos local o en su entorno de nube.
services: automation
ms.subservice: process-automation
ms.date: 04/02/2021
ms.topic: conceptual
ms.openlocfilehash: 0a28266210fd8b6f0b731b972f00aa3d413c0d0c
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107027744"
---
# <a name="deploy-a-windows-hybrid-runbook-worker"></a>Implementación de Hybrid Runbook Worker en Windows

Puede usar la característica Hybrid Runbook Worker de usuario de Azure Automation para ejecutar runbooks directamente en una máquina de Azure o que no es de Azure, incluidos los servidores registrados con [servidores habilitados para Azure Arc](../azure-arc/servers/overview.md). En la máquina o servidor que hospeda el rol, puede ejecutar runbooks directamente en él y en los recursos del entorno para administrar esos recursos locales.

Azure Automation almacena y administra los runbooks y, después, los entrega a una o más máquinas designadas. En este artículo se describe cómo implementar una instancia de Hybrid Runbook Worker de usuario en una máquina Windows, cómo quitar el trabajo y cómo quitar un grupo de instancias de Hybrid Runbook Worker.

Después de implementar correctamente un trabajo de runbook, revise la [ejecución de runbooks en Hybrid Runbook Worker](automation-hrw-run-runbooks.md) para más información sobre cómo configurar los runbooks para automatizar los procesos del centro de datos local o en otro entorno de nube.

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar, asegúrese de que dispone de lo siguiente.

### <a name="a-log-analytics-workspace"></a>Un área de trabajo de Log Analytics

El rol Hybrid Runbook Worker depende de un área de trabajo de Log Analytics de Azure Monitor para instalar y configurar el rol. Puede usar [Azure Resource Manager](../azure-monitor/logs/resource-manager-workspace.md#create-a-log-analytics-workspace), mediante [PowerShell](../azure-monitor/logs/powershell-sample-create-workspace.md?toc=/powershell/module/toc.json) o [Azure Portal](../azure-monitor/logs/quick-create-workspace.md), para crearla.

Si no tiene ningún área de trabajo de Log Analytics de Azure Monitor, revise la [guía de diseño de registros de Azure Monitor](../azure-monitor/logs/design-logs-deployment.md) antes de crear el área de trabajo.

### <a name="log-analytics-agent"></a>Agente de Log Analytics

El rol de Hybrid Runbook Worker requiere el [agente de Log Analytics](../azure-monitor/agents/log-analytics-agent.md) para el sistema operativo Windows compatible. En el caso de servidores o máquinas hospedados fuera de Azure, puede instalar el agente de Log Analytics con [servidores habilitados para Azure Arc](../azure-arc/servers/overview.md).

### <a name="supported-windows-operating-system"></a>Sistemas operativos Windows compatibles

La característica Hybrid Runbook Worker admite los siguientes sistemas operativos:

* Windows Server 2019 (incluido Server Core)
* Windows Server 2016, versiones 1709 y 1803 (no se incluye Server Core)
* Windows Server 2012, 2012 R2
* Windows Server 2008 SP2 (x64), 2008 R2
* Windows 10 Enterprise (incluida la sesión múltiple) y Pro
* Windows 8 Enterprise y Pro
* Windows 7 SP1

### <a name="minimum-requirements"></a>Requisitos mínimos

Los requisitos mínimos de un sistema Windows y una instancia de usuario de Hybrid Runbook Worker son los siguientes:

* Windows PowerShell 5.1 ([descargue WMF 5.1](https://www.microsoft.com/download/details.aspx?id=54616)). No se admite PowerShell Core.
* .NET Framework 4.6.2, o posterior
* Dos núcleos
* 4 GB de RAM
* Puerto 443 (saliente)

### <a name="network-configuration"></a>Network configuration (Configuración de red)

Para conocer los requisitos de red de Hybrid Runbook Worker, consulte [Configuración de la red](automation-hybrid-runbook-worker.md#network-planning).

### <a name="adding-a-machine-to-a-hybrid-runbook-worker-group"></a>Adición de una máquina a un grupo de Hybrid Runbook Worker.

Puede agregar la máquina de trabajo a un grupo de Hybrid Runbook Worker de una de sus cuentas de Automation. Las máquinas que hospedan la instancia de Hybrid Runbook Worker del sistema administrada por Update Management se pueden agregar a un grupo de Hybrid Runbook Worker. Sin embargo, debe usar la misma cuenta de Automation para Update Management y para la pertenencia al grupo de Hybrid Runbook Worker.

>[!NOTE]
>[Update Management](./update-management/overview.md) de Azure Automation instala automáticamente la instancia de Hybrid Runbook Worker del sistema en una máquina de Azure o que no es de Azure habilitada para Update Management. Sin embargo, este trabajo no se registra con ninguno de los grupos de Hybrid Runbook Worker de la cuenta de Automation. Para ejecutar los runbooks en esas máquinas, debe agregarlas a un grupo de Hybrid Runbook Worker. Siga el paso 6 de la sección [Implementación manual](#manual-deployment) para agregarla a un grupo.

## <a name="enable-for-management-with-azure-automation-state-configuration"></a>Habilitación de la administración con State Configuration de Azure Automation

Para obtener información sobre cómo habilitar máquinas para su administración con Azure Automation State Configuration, consulte [Habilitar máquinas para administrarlas con Azure Automation State Configuration](automation-dsc-onboarding.md).

> [!NOTE]
> Para administrar la configuración de las máquinas que admiten el rol de Hybrid Runbook Worker con Desired State Configuration (DSC), debe agregar dichas máquinas como nodos de DSC.

## <a name="installation-options"></a>Opción de instalación

Para instalar y configurar una instancia de Hybrid Runbook Worker de usuario en Windows, puede usar uno de los métodos siguientes.

* Use un script de PowerShell proporcionado para [automatizar](#automated-deployment) por completo el proceso de configuración de una o más máquinas Windows. Este es el método recomendado para las máquinas de su centro de recursos u otro entorno en la nube.
* Importe manualmente la solución de Automation, instale el agente de Log Analytics para Windows y configure el rol de trabajo en la máquina.

## <a name="automated-deployment"></a>Implementación automatizada

Hay dos métodos para implementar automáticamente Hybrid Runbook Worker. Puede importar un runbook desde la galería de runbooks en Azure Portal y ejecutarlo, o puede descargar manualmente un script de la Galería de PowerShell.

### <a name="importing-a-runbook-from-the-runbook-gallery"></a>Importación de un runbook desde la galería de runbooks

El procedimiento de importación se describe en detalle en [Importación de runbooks desde GitHub con Azure Portal](automation-runbook-gallery.md#import-runbooks-from-github-with-the-azure-portal). El nombre del runbook que se va a importar es **Create Automation Windows HybridWorker**.

El runbook usa los parámetros siguientes.

| Parámetro | Estado | Descripción |
| ------- | ----- | ----------- |
| `Location` | Mandatory | La ubicación del área de trabajo de Log Analytics. |
| `ResourceGroupName` | Mandatory | El grupo de recursos de la cuenta de Automation. |
| `AccountName` | Mandatory | El nombre de la cuenta de Automation en la que se registrará Hybrid Run Worker. |
| `CreateLA` | Mandatory | Si es true, usa el valor de `WorkspaceName` para crear un área de trabajo de Log Analytics. Si es false, el valor de `WorkspaceName` debe hacer referencia a un área de trabajo existente. |
| `LAlocation` | Opcional | Ubicación en la que se creará el área de trabajo de Log Analytics o donde ya existe. |
| `WorkspaceName` | Opcional | Nombre del área de trabajo de Log Analytics que se va a usar. |
| `CreateVM` | Mandatory | Si es true, use el valor de `VMName` como nombre de una nueva VM. Si es false, use `VMName` para buscar y registrar la VM existente. |
| `VMName` | Opcional | Nombre de la máquina virtual que se crea o se registra, dependiendo del valor de `CreateVM`. |
| `VMImage` | Opcional | Nombre de la imagen de VM que se va a crear. |
| `VMlocation` | Opcional | Ubicación de la VM que se ha creado o registrado. Si no se especifica esta ubicación, se usa el valor de `LAlocation`. |
| `RegisterHW` | Mandatory | Si es true, registre la VM como Hybrid Worker. |
| `WorkerGroupName` | Mandatory | Nombre del Grupo de Hybrid Worker. |

### <a name="download-a-script-from-the-powershell-gallery"></a>Descarga de un script  de la Galería de PowerShell

El método de implementación automatizada usa el script **New-OnPremiseHybridWorker.ps1** de PowerShell para automatizar y configurar el rol de Hybrid Runbook Worker de Windows. Se realizan los pasos siguientes:

* Instala los módulos necesarios.
* Inicia sesión con su cuenta de Azure.
* Comprueba la existencia de la cuenta de Automation y el grupo de recursos especificados.
* Crea referencias a los atributos de la cuenta de Automation.
* Crea un área de trabajo de Log Analytics en Azure Monitor si no se especifica.
* Habilitar la solución Azure Automation en el área de trabajo
* Descarga e instalación del agente de Log Analytics para Windows
* Registro de las máquinas como Hybrid Runbook Worker

Realice los pasos siguientes para instalar el rol en la máquina Windows mediante el script.

1. Descargue el script **New-OnPremiseHybridWorker.ps1** de la [Galería de PowerShell](https://www.powershellgallery.com/packages/New-OnPremiseHybridWorker). Una vez descargado el script, cópielo o ejecútelo en la máquina de destino. El script usa los parámetros siguientes.

    | Parámetro | Estado | Descripción |
    | --------- | ------ | ----------- |
    | `AAResourceGroupName` | Mandatory | el nombre del grupo de recursos que está asociado a su cuenta de Automation. |
    | `AutomationAccountName` | Mandatory | el nombre de la cuenta de Automation.
    | `Credential` | Opcional | credenciales que se usarán al iniciar sesión en el entorno de Azure. |
    | `HybridGroupName` | Mandatory | el nombre de un grupo de Hybrid Runbook Worker que se especifica como destino para los runbooks que admiten este escenario. |
    | `OMSResourceGroupName` | Opcional | el nombre del grupo de recursos para el área de trabajo de Log Analytics. Si no se especifica este grupo de recursos, se usa el valor de `AAResourceGroupName`. |
    | `SubscriptionID` | Mandatory | identificador de la suscripción a Azure asociada a su cuenta de Automation. |
    | `TenantID` | Opcional | identificador de la organización del inquilino asociado a su cuenta de Automation. |
    | `WorkspaceName` | Opcional | Nombre del área de trabajo de Log Analytics. Si no tiene un área de trabajo de Log Analytics, el script creará y configurará una. |

1. Abra un símbolo del sistema de PowerShell de 64 bits con privilegios elevados.

1. En el símbolo del sistema de PowerShell, vaya a la carpeta que contiene el script que ha descargado. Cambie los valores de los parámetros `AutomationAccountName`, `AAResourceGroupName`, `OMSResourceGroupName`, `HybridGroupName`, `SubscriptionID` y `WorkspaceName`. Después, ejecute el script.

    Se le solicitará que se autentique en Azure después de ejecutar el script. Debe iniciar sesión con una cuenta que sea miembro del rol **Administradores de suscripciones** y coadministrador de la suscripción.

    ```powershell-interactive
    $NewOnPremiseHybridWorkerParameters = @{
      AutomationAccountName = <nameOfAutomationAccount>
      AAResourceGroupName   = <nameOfResourceGroup>
      OMSResourceGroupName  = <nameOfResourceGroup>
      HybridGroupName       = <nameOfHRWGroup>
      SubscriptionID        = <subscriptionId>
      WorkspaceName         = <nameOfLogAnalyticsWorkspace>
    }
    .\New-OnPremiseHybridWorker.ps1 @NewOnPremiseHybridWorkerParameters
    ```

1. Se le pedirá que acepte la instalación de NuGet y que se autentique con sus credenciales de Azure. Si no tiene la versión más reciente de NuGet, puede descargarla en [Versiones de distribución de NuGet disponibles](https://www.nuget.org/downloads).

1. Después de que finalice el script, compruebe la implementación. En la página **Hybrid Runbook Worker Groups** (Grupos de Hybrid Runbook Worker) de la cuenta de Automation, en la pestaña **User hybrid runbook workers group** (Grupo de instancias de Hybrid Runbook Worker de usuario), se muestra el nuevo grupo y el número de miembros. Si se trata de un grupo existente, se incrementa el número de miembros. Seleccione el grupo en la lista de la página y, en el menú de la izquierda, elija **Hybrid Workers**. En la página **Hybrid Workers**, puede ver cada uno de los miembros del grupo mostrado.

## <a name="manual-deployment"></a>Implementación manual

Para instalar y configurar una instancia de Hybrid Runbook Worker para Windows, siga los pasos que se indican a continuación.

1. Habilite la solución Azure Automation en el área de trabajo de Log Analytics ejecutando el siguiente comando en un símbolo del sistema de PowerShell con privilegios elevados o en Cloud Shell en [Azure Portal](https://portal.azure.com).

    ```powershell
    Set-AzOperationalInsightsIntelligencePack -ResourceGroupName <resourceGroupName> -WorkspaceName <workspaceName> -IntelligencePackName "AzureAutomation" -Enabled $true
    ```

1. Implemente el agente de Log Analytics en la máquina de destino.

    * En el caso de las máquinas virtuales de Azure, instale el agente de Log Analytics para Windows mediante la [extensión de máquina virtual para Windows](../virtual-machines/extensions/oms-windows.md). La extensión instala el agente de Log Analytics en Azure Virtual Machines e inscribe las máquinas virtuales en un área de trabajo de Log Analytics. Puede usar una plantilla de Azure Resource Manager, PowerShell o Azure Policy para asignar la directiva integrada [Implementar el agente de Log Analytics para las VM *Linux* o *Windows*](../governance/policy/samples/built-in-policies.md#monitoring). Una vez instalado el agente, la máquina se puede agregar a un grupo de Hybrid Runbook Worker de su cuenta de Automation.
    
    * En el caso de máquinas que no son de Azure, puede instalar el agente de Log Analytics mediante [servidores habilitados para Azure Arc](../azure-arc/servers/overview.md). Los servidores habilitados para Arc admiten la implementación del agente de Log Analytics mediante los métodos siguientes:
    
        - Uso del marco de extensiones de VM.
        
            Esta característica de los servidores habilitados para Azure Arc le permite implementar la extensión de VM del agente de Log Analytics en un servidor de Windows o Linux que no sea de Azure. Las extensiones de VM se pueden administrar con los siguientes métodos en las máquinas híbridas o servidores administrados por servidores habilitados para Arc:
        
            - [Azure Portal](../azure-arc/servers/manage-vm-extensions-portal.md)
            - La [CLI de Azure](../azure-arc/servers/manage-vm-extensions-cli.md)
            - [Azure PowerShell](../azure-arc/servers/manage-vm-extensions-powershell.md)
            - [Plantillas de Azure Resource Manager](../azure-arc/servers/manage-vm-extensions-template.md)
        
        - Uso de Azure Policy.
        
            Con este enfoque, puede usar la directiva integrada [Implementación del agente de Log Analytics en máquinas de Azure Arc de Linux o de Windows](../governance/policy/samples/built-in-policies.md#monitoring) de Azure Policy para auditar si el servidor habilitado para Arc tiene instalado el agente de Log Analytics. Si el agente no está instalado, lo implementa automáticamente mediante una tarea de corrección. Como alternativa, si planea supervisar las máquinas con Azure Monitor para VM, puede usar en su lugar la iniciativa [Habilitar Azure Monitor para VM](../governance/policy/samples/built-in-initiatives.md#monitoring) para instalar y configurar el agente de Log Analytics.

    Se recomienda instalar el agente de Log Analytics para Windows o Linux con Azure Policy.

1. Compruebe que el agente informa al área de trabajo.

    El agente de Log Analytics para Windows conecta las máquinas a un área de trabajo de Log Analytics de Azure Monitor. Cuando instala el agente en la máquina y lo conecta al área de trabajo, se descargan automáticamente los componentes necesarios para la instancia de Hybrid Runbook Worker.

    Una vez que el agente se ha conectado correctamente al área de trabajo de Log Analytics, transcurridos unos minutos, puede ejecutar la consulta siguiente para comprobar que envía datos de latido al área de trabajo.

    ```kusto
    Heartbeat 
    | where Category == "Direct Agent"
    | where TimeGenerated > ago(30m)
    ```

    En los resultados de la búsqueda, verá los registros de latido de la máquina que indican que está conectada y que está creando informes para el servicio. De forma predeterminada, cada agente reenvía un registro de latido a su área de trabajo asignada. Realice los siguientes pasos para completar la instalación y configuración del agente.

1. Confirme la versión de Hybrid Runbook Worker en la máquina que hospeda el agente de Log Analytics, vaya a `C:\Program Files\Microsoft Monitoring Agent\Agent\AzureAutomation\` y fíjese en la subcarpeta **version**. Esta carpeta aparecerá en la máquina varios minutos después de habilitar la solución en el área de trabajo.

1. Instale el entorno del runbook y conéctese a Azure Automation. Cuando se configura un agente para que informe a un área de trabajo de Log Analytics y se importa la solución **Automation**, esta inserta el módulo `HybridRegistration` de PowerShell. Este módulo contiene el cmdlet `Add-HybridRunbookWorker`. Este cmdlet se usa para instalar el entorno de runbook en la máquina y registrarlo en Azure Automation.

    Abra una sesión de PowerShell en modo de Administrador y ejecute el comando siguientes para importar el módulo.

    ```powershell-interactive
    cd "C:\Program Files\Microsoft Monitoring Agent\Agent\AzureAutomation\<version>\HybridRegistration"
    Import-Module .\HybridRegistration.psd1
    ```

1. Ejecute el cmdlet `Add-HybridRunbookWorker` y especifique los valores de los parámetros `Url`, `Key` y `GroupName`.

    ```powershell-interactive
    Add-HybridRunbookWorker –GroupName <String> -Url <Url> -Key <String>
    ```

    Puede obtener la información necesaria para los parámetros `Url` y `Key` de la página **Claves** de la cuenta de Automation. Seleccione **Claves** en la sección **Configuración de la cuenta** en el lado izquierdo de la página.

    ![Página Administrar claves](media/automation-hybrid-runbook-worker/elements-panel-keys.png)

    * Para el parámetro `Url`, copie el valor para **URL**.

    * Para el parámetro `Key`, copie el valor para **CLAVE DE ACCESO PRIMARIA**.

    * En el parámetro `GroupName`, use el nombre del grupo de Hybrid Runbook Worker. Si este grupo ya existe en la cuenta de Automation, se le agregará la máquina actual. Si este grupo no existe, se agrega.

    * Si es necesario, establezca el parámetro `Verbose` para recibir detalles sobre la instalación.

1. Después de que finalice el comando, compruebe la implementación. En la página **Hybrid Runbook Worker Groups** (Grupos de Hybrid Runbook Worker) de la cuenta de Automation, en la pestaña **User hybrid runbook workers group** (Grupo de instancias de Hybrid Runbook Worker de usuario), se muestra el grupo nuevo o existente y el número de miembros. Si se trata de un grupo existente, se incrementa el número de miembros. Puede seleccionar el grupo en la lista de la página y, en el menú de la izquierda, elegir **Hybrid Workers**. En la página **Hybrid Workers**, puede ver cada uno de los miembros del grupo mostrado.

## <a name="install-powershell-modules"></a>Instalación de módulos PowerShell

Los runbooks pueden usar cualquiera de las actividades y los cmdlets definidos en los módulos instalados en el entorno de Azure Automation. Dado que estos módulos no se implementan automáticamente en las máquinas locales, debe instalarlos de forma manual. La excepción es el módulo de Azure. Este módulo se instala de manera predeterminada y ofrece acceso a los cmdlets para todos los servicios y actividades de Azure para Azure Automation.

Como el propósito principal de la instancia de Hybrid Runbook Worker es administrar recursos locales, es muy probable que deba instalar los módulos que admiten estos recursos, específicamente el módulo `PowerShellGet`. Para más información sobre cómo instalar módulos de Windows PowerShell, consulte [Windows PowerShell](/powershell/scripting/developer/windows-powershell).

Los módulos instalados deben estar en una ubicación a la que haga referencia la variable de entorno `PSModulePath` para que Hybrid Worker los importe automáticamente. Para más información, consulte [Instalación de módulos en PSModulePath](/powershell/scripting/developer/module/installing-a-powershell-module).

## <a name="remove-the-hybrid-runbook-worker"></a><a name="remove-windows-hybrid-runbook-worker"></a>Eliminación de la instancia de Hybrid Runbook Worker

1. En Azure Portal, abra su cuenta de Automation.

1. En **Configuración de la cuenta**, seleccione **Claves** y anote los valores de **URL** y **Clave de acceso primaria**.

1. Abra una sesión de PowerShell en modo de administrador y ejecute el siguiente comando con los valores de la dirección URL y la clave de acceso principal. Utilice el parámetro `Verbose` para ver un registro detallado del proceso de eliminación. Para eliminar máquinas obsoletas del grupo Hybrid Worker, use el parámetro `machineName` opcional.

```powershell-interactive
Remove-HybridRunbookWorker -Url <URL> -Key <primaryAccessKey> -MachineName <computerName>
```

## <a name="remove-a-hybrid-worker-group"></a>Eliminación de un grupo de Hybrid Worker

Para quitar un grupo de Hybrid Runbook Worker, primero debe quitar la instancia de Hybrid Runbook Worker de todas las máquinas que sean miembros del grupo. Después, siga estos pasos para quitar el grupo:

1. Abra la cuenta de Automation en Azure Portal.

1. Seleccione **Grupos de Hybrid Worker** en **Automatización de procesos**. Seleccione el grupo que quiere eliminar. Aparece la página de propiedades de ese grupo.

   ![Página de propiedades](media/automation-hybrid-runbook-worker/automation-hybrid-runbook-worker-group-properties.png)

1. En la página de propiedades del grupo seleccionado, seleccione **Eliminar**. Un mensaje le solicita que confirme esta acción. Seleccione **Sí** si está seguro de que quiere continuar.

   ![Mensaje de confirmación](media/automation-hybrid-runbook-worker/automation-hybrid-runbook-worker-confirm-delete.png)

   Este proceso puede tardar varios segundos en finalizar. Puede realizar un seguimiento de su progreso en **Notificaciones** en el menú.

## <a name="next-steps"></a>Pasos siguientes

* Consulte [Ejecución de runbooks en Hybrid Runbook Worker](automation-hrw-run-runbooks.md) para más información sobre cómo configurar los runbooks para automatizar los procesos del centro de datos local o de otros entornos de nube.

* Para aprender a solucionar problemas con las instancias de Hybrid Runbook Worker, consulte [Solución de incidencias de Hybrid Runbook Worker](troubleshoot/hybrid-runbook-worker.md#general).
