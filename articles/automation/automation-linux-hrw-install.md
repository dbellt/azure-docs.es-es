---
title: Implementación de una instancia de Hybrid Runbook Worker en Azure Automation
description: En este artículo se describe cómo instalar Hybrid Runbook Worker de Azure Automation para ejecutar runbooks en máquinas con Linux en su centro de datos local o en su entorno en la nube.
services: automation
ms.subservice: process-automation
ms.date: 04/06/2021
ms.topic: conceptual
ms.openlocfilehash: 21bb3a353874e401d86741584d102b5c217e69cc
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107030056"
---
# <a name="deploy-a-linux-hybrid-runbook-worker"></a>Implementación de Hybrid Runbook Worker en Linux

Puede usar la característica Hybrid Runbook Worker de usuario de Azure Automation para ejecutar runbooks directamente en la máquina de Azure o que no es de Azure, incluidos los servidores registrados con [servidores habilitados para Azure Arc](../azure-arc/servers/overview.md). En la máquina o el servidor que hospeda el rol, puede ejecutar runbooks directamente y con los recursos del entorno para administrar esos recursos locales.

Hybrid Runbook Worker en Linux ejecuta runbooks como un usuario especial que puede tener permisos elevados para ejecutar comandos que necesitan permisos elevados. Azure Automation almacena y administra los runbooks y, después, los entrega a una o más máquinas designadas. En este artículo se describe cómo instalar la instancia de Hybrid Runbook Worker en una máquina Linux, cómo eliminar el trabajo y cómo eliminar un grupo de Hybrid Runbook Worker.

Después de implementar correctamente un trabajo de runbook, revise la [ejecución de runbooks en Hybrid Runbook Worker](automation-hrw-run-runbooks.md) para más información sobre cómo configurar los runbooks para automatizar los procesos del centro de datos local o en otro entorno de nube.

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar, asegúrese de que dispone de lo siguiente.

### <a name="a-log-analytics-workspace"></a>Un área de trabajo de Log Analytics

El rol Hybrid Runbook Worker depende de un área de trabajo de Log Analytics de Azure Monitor para instalar y configurar el rol. Puede usar [Azure Resource Manager](../azure-monitor/logs/resource-manager-workspace.md#create-a-log-analytics-workspace), mediante [PowerShell](../azure-monitor/logs/powershell-sample-create-workspace.md?toc=/powershell/module/toc.json) o [Azure Portal](../azure-monitor/logs/quick-create-workspace.md), para crearla.

Si no tiene ningún área de trabajo de Log Analytics de Azure Monitor, revise la [guía de diseño de registros de Azure Monitor](../azure-monitor/logs/design-logs-deployment.md) antes de crear el área de trabajo.

### <a name="log-analytics-agent"></a>Agente de Log Analytics

El rol de Hybrid Runbook Worker requiere el [agente de Log Analytics](../azure-monitor/agents/log-analytics-agent.md) para el sistema operativo Linux compatible. En el caso de servidores o máquinas hospedados fuera de Azure, puede instalar el agente de Log Analytics con [servidores habilitados para Azure Arc](../azure-arc/servers/overview.md).

>[!NOTE]
>Después de instalar el agente de Log Analytics para Linux, no debe cambiar los permisos de la carpeta `sudoers.d` ni su propiedad. Se requiere el permiso sudo para la cuenta **nxautomation**, que es el contexto de usuario en el que se ejecuta Hybrid Runbook Worker. No se deberían quitar los permisos. Su restricción a determinadas carpetas o comandos puede dar lugar a un cambio importante.
>

### <a name="supported-linux-operating-systems"></a>Sistemas operativos Linux compatibles

La característica Hybrid Runbook Worker admite las siguientes distribuciones. Se supone que todos los sistemas operativos son x64. No se admite x86 para ningún sistema operativo.

* Amazon Linux 2012.09 a 2015.09
* CentOS Linux 5, 6, 7 y 8
* Oracle Linux 5, 6 y 7
* Red Hat Enterprise Linux Server 5, 6, 7 y 8
* Debian GNU/Linux 6, 7 y 8
* Ubuntu 12.04 LTS, 14.04 LTS, 16.04 LTS y 18.04 LTS
* SUSE Linux Enterprise Server 12 y 15 (no se publicaron las versiones 13 y 14 de SUSE)

> [!IMPORTANT]
> Antes de habilitar la característica Update Management, que depende del rol Hybrid Runbook Worker del sistema, confirme las distribuciones que admite [aquí](update-management/overview.md#supported-operating-systems).

### <a name="minimum-requirements"></a>Requisitos mínimos

Los requisitos mínimos de un sistema Linux y una instancia de Hybrid Runbook Worker de usuario son los siguientes:

* Dos núcleos
* 4 GB de RAM
* Puerto 443 (saliente)

| **Paquetes necesarios** | **Descripción** | **Versión mínima**|
|--------------------- | --------------------- | -------------------|
|Glibc |Biblioteca GNU C| 2.5-12 |
|Openssl| Bibliotecas OpenSSL | 1.0 (se admiten TLS 1.1 y TLS 1.2)|
|Curl | Cliente web de cURL | 7.15.5|
|Python ctypes | Se requieren Python 2.x o Python 3.x |
|PAM | Módulos de autenticación conectables|
| **Paquete opcional** | **Descripción** | **Versión mínima**|
| PowerShell Core | Para ejecutar runbooks de PowerShell, es necesario instalar PowerShell Core. Consulte [Instalación de PowerShell Core en Linux](/powershell/scripting/install/installing-powershell-core-on-linux) para obtener información sobre cómo instalarlo. | 6.0.0 |

### <a name="adding-a-machine-to-a-hybrid-runbook-worker-group"></a>Adición de una máquina a un grupo de Hybrid Runbook Worker.

Puede agregar la máquina de trabajo a un grupo de Hybrid Runbook Worker de una de sus cuentas de Automation. Las máquinas que hospedan la instancia de Hybrid Runbook Worker del sistema administrada por Update Management se pueden agregar a un grupo de Hybrid Runbook Worker. Sin embargo, debe usar la misma cuenta de Automation para Update Management y para la pertenencia al grupo de Hybrid Runbook Worker.

>[!NOTE]
>[Update Management](./update-management/overview.md) de Azure Automation instala automáticamente la instancia de Hybrid Runbook Worker del sistema en una máquina de Azure o que no es de Azure habilitada para Update Management. Sin embargo, este trabajo no se registra con ninguno de los grupos de Hybrid Runbook Worker de la cuenta de Automation. Para ejecutar los runbooks en esas máquinas, debe agregarlas a un grupo de Hybrid Runbook Worker. Siga el paso 4 de la sección [Instalación de un Hybrid Runbook Worker de Linux](#install-a-linux-hybrid-runbook-worker) para agregarlo a un grupo.

## <a name="supported-linux-hardening"></a>Protección de Linux admitida

Todavía no se admite lo siguiente:

* CIS

## <a name="supported-runbook-types"></a>Tipos de runbook admitidos

Las instancias de Hybrid Runbook Worker admiten un conjunto limitado de tipos de runbook en Azure Automation y se describen en la tabla siguiente.

|Tipo de runbook | Compatible |
|-------------|-----------|
|Python 3 (versión preliminar)|Sí, necesario solo para estas distribuciones: SUSE LES 15, RHEL 8 y CentOS 8|
|Python 2 |Sí, para cualquier distribución que no requiera Python 3<sup>1</sup> |
|PowerShell |Sí<sup>2</sup> |
|Flujo de trabajo de PowerShell |No |
|Gráfico |No |
|Flujo de trabajo gráfico de PowerShell |No |

<sup>1</sup>Consulte [Sistemas operativos Linux compatibles](#supported-linux-operating-systems).

<sup>2</sup>Los runbooks de PowerShell requieren que PowerShell Core esté instalado en la máquina Linux. Consulte [Instalación de PowerShell Core en Linux](/powershell/scripting/install/installing-powershell-core-on-linux) para obtener información sobre cómo instalarlo.

### <a name="network-configuration"></a>Configuración de red

Para conocer los requisitos de red de Hybrid Runbook Worker, consulte [Configuración de la red](automation-hybrid-runbook-worker.md#network-planning).

## <a name="install-a-linux-hybrid-runbook-worker"></a>Instalación de Hybrid Runbook Worker en Linux

Hay dos métodos para implementar Hybrid Runbook Worker. Puede importar y ejecutar un runbook desde la galería de runbooks de Azure Portal, o puede ejecutar manualmente una serie de comandos de PowerShell para realizar la misma tarea.

### <a name="importing-a-runbook-from-the-runbook-gallery"></a>Importación de un runbook desde la galería de runbooks

El procedimiento de importación se describe en detalle en [Importación de runbooks desde GitHub con Azure Portal](automation-runbook-gallery.md#import-runbooks-from-github-with-the-azure-portal). El nombre del runbook que se va a importar es **Create Automation Linux HybridWorker**.

El runbook usa los parámetros siguientes.

| Parámetro | Estado | Descripción |
| ------- | ----- | ----------- |
| `Location` | Mandatory | La ubicación del área de trabajo de Log Analytics. |
| `ResourceGroupName` | Mandatory | El grupo de recursos de la cuenta de Automation. |
| `AccountName` | Mandatory | El nombre de la cuenta de Automation en la que se registrará Hybrid Run Worker. |
| `CreateLA` | Mandatory | Si es true, usa el valor de `WorkspaceName` para crear un área de trabajo de Log Analytics. Si es false, el valor de `WorkspaceName` debe hacer referencia a un área de trabajo existente. |
| `LAlocation` | Opcional | Ubicación en la que se creará el área de trabajo de Log Analytics o donde ya existe. |
| `WorkspaceName` | Opcional | Nombre del área de trabajo de Log Analytics que se va a crear o a usar. |
| `CreateVM` | Mandatory | Si es true, use el valor de `VMName` como nombre de una nueva VM. Si es false, use `VMName` para buscar y registrar la VM existente. |
| `VMName` | Opcional | Nombre de la máquina virtual que se crea o se registra, dependiendo del valor de `CreateVM`. |
| `VMImage` | Opcional | Nombre de la imagen de VM que se va a crear. |
| `VMlocation` | Opcional | Ubicación de la VM que se ha creado o registrado. Si no se especifica esta ubicación, se usa el valor de `LAlocation`. |
| `RegisterHW` | Mandatory | Si es true, registre la VM como Hybrid Worker. |
| `WorkerGroupName` | Mandatory | Nombre del Grupo de Hybrid Worker. |

### <a name="manually-run-powershell-commands"></a>Ejecución manual de comandos de PowerShell

Para instalar y configurar una instancia de Hybrid Runbook Worker para Linux, siga los pasos que se indican a continuación.

1. Habilite la solución Azure Automation en el área de trabajo de Log Analytics mediante la ejecución del siguiente comando en un símbolo del sistema de PowerShell con privilegios elevados o en Cloud Shell en [Azure Portal](https://portal.azure.com):

    ```powershell
    Set-AzOperationalInsightsIntelligencePack -ResourceGroupName <resourceGroupName> -WorkspaceName <workspaceName> -IntelligencePackName "AzureAutomation" -Enabled $true
    ```

2. Implemente el agente de Log Analytics en la máquina de destino.

    * En el caso de las VM de Azure, instale el agente de Log Analytics para Linux mediante la [extensión de máquina virtual para Linux](../virtual-machines/extensions/oms-linux.md). La extensión instala el agente de Log Analytics en Azure Virtual Machines e inscribe las máquinas virtuales en un área de trabajo de Log Analytics. Puede usar una plantilla de Azure Resource Manager, la CLI de Azure o Azure Policy para asignar la directiva integrada [Implementar el agente de Log Analytics para las VM *Linux* o *Windows*](../governance/policy/samples/built-in-policies.md#monitoring). Una vez instalado el agente, la máquina se puede agregar a un grupo de Hybrid Runbook Worker de su cuenta de Automation.

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

    > [!NOTE]
    > Para administrar la configuración de las máquinas que admiten el rol de Hybrid Runbook Worker con Desired State Configuration (DSC), debe agregar dichas máquinas como nodos de DSC.

    > [!NOTE]
    > La [cuenta nxautomation](automation-runbook-execution.md#log-analytics-agent-for-linux) con los permisos sudo correspondientes debe estar presente durante la instalación de la instancia de Hybrid Worker en Linux. Si intenta instalar el rol de trabajo y la cuenta no está presente o no tiene los permisos adecuados, no se realizará la instalación.

3. Compruebe que el agente informa al área de trabajo.

    El agente de Log Analytics para Linux conecta las máquinas a un área de trabajo de Log Analytics de Azure Monitor. Cuando instala el agente en la máquina y lo conecta al área de trabajo, se descargan automáticamente los componentes necesarios para la instancia de Hybrid Runbook Worker.

    Una vez que el agente se ha conectado correctamente al área de trabajo de Log Analytics, transcurridos unos minutos, puede ejecutar la consulta siguiente para comprobar que envía datos de latido al área de trabajo.

    ```kusto
    Heartbeat
    | where Category == "Direct Agent"
    | where TimeGenerated > ago(30m)
    ```

    En los resultados de la búsqueda, verá los registros de latido de la máquina que indican que está conectada y que está creando informes para el servicio. De forma predeterminada, cada agente reenvía un registro de latido a su área de trabajo asignada.

4. Ejecute el siguiente comando para agregar la máquina a un grupo de Hybrid Runbook Worker y especifique los valores para los parámetros `-w`, `-k`, `-g` y `-e`.

    Puede obtener la información necesaria para los parámetros `-k` y `-e` de la página **Claves** de la cuenta de Automation. Seleccione **Claves** en la sección **Configuración de la cuenta** en el lado izquierdo de la página.

    ![Página Administrar claves](media/automation-hybrid-runbook-worker/elements-panel-keys.png)

    * Para el parámetro `-e`, copie el valor para **URL**.

    * Para el parámetro `-k`, copie el valor para **CLAVE DE ACCESO PRIMARIA**.

    * Para el parámetro `-g`, especifique el nombre del grupo de Hybrid Runbook Worker al que se debe unir la nueva instancia de Hybrid Runbook Worker para Linux. Si este grupo ya existe en la cuenta de Automation, se le agregará la máquina actual. Si no existe, se crea con ese nombre.

    * Para el parámetro `-w`, especifique el identificador del área de trabajo de Log Analytics.

   ```bash
   sudo python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/scripts/onboarding.py --register -w <logAnalyticsworkspaceId> -k <automationSharedKey> -g <hybridGroupName> -e <automationEndpoint>
   ```

5. Después de que se complete el script, compruebe la implementación. En la página **Hybrid Runbook Worker Groups** (Grupos de Hybrid Runbook Worker) de la cuenta de Automation, en la pestaña **User hybrid runbook workers group** (Grupo de instancias de Hybrid Runbook Worker de usuario), se muestra el grupo nuevo o existente y el número de miembros. Si se trata de un grupo existente, se incrementa el número de miembros. Puede seleccionar el grupo en la lista de la página y, en el menú de la izquierda, elegir **Hybrid Workers**. En la página **Hybrid Workers**, se mostrarán todos los miembros del grupo.

    > [!NOTE]
    > Si usa la extensión de máquina virtual de Log Analytics para una VM de Azure para Linux, se recomienda establecer `autoUpgradeMinorVersion` en `false`, ya que las versiones de actualización automática pueden provocar problemas con Hybrid Runbook Worker. Para obtener información sobre cómo actualizar manualmente la extensión, vea [Implementación de la CLI de Azure](../virtual-machines/extensions/oms-linux.md#azure-cli-deployment).

## <a name="turn-off-signature-validation"></a>Desactivación de la validación de firma

De forma predeterminada, las instancias de Hybrid Runbook Worker de Linux requieren la validación de la firma. Si ejecuta un runbook sin firmar en un trabajo, verá un error `Signature validation failed`. Para desactivar la validación de la firma, ejecute el siguiente comando. Reemplace el segundo parámetro por el identificador de área de trabajo de Log Analytics.

 ```bash
 sudo python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/scripts/require_runbook_signature.py --false <logAnalyticsworkspaceId>
 ```

## <a name="remove-the-hybrid-runbook-worker"></a><a name="remove-linux-hybrid-runbook-worker"></a>Eliminación de la instancia de Hybrid Runbook Worker

Puede usar el comando `ls /var/opt/microsoft/omsagent` en Hybrid Runbook Worker para obtener el identificador del área de trabajo. Se creará una carpeta cuyo nombre será el identificador del área de trabajo.

```bash
sudo python onboarding.py --deregister --endpoint="<URL>" --key="<PrimaryAccessKey>" --groupname="Example" --workspaceid="<workspaceId>"
```

> [!NOTE]
> Este script no quita el agente de Log Analytics para Linux de la máquina. Solo quita la funcionalidad y la configuración del rol de Hybrid Runbook Worker.

## <a name="remove-a-hybrid-worker-group"></a>Eliminación de un grupo de Hybrid Worker

Para eliminar un grupo de Hybrid Runbook Worker de máquinas con Linux, use los mismos pasos que para un grupo de Hybrid Worker de Windows. Consulte [Eliminación de un grupo de Hybrid Worker](automation-windows-hrw-install.md#remove-a-hybrid-worker-group).

## <a name="next-steps"></a>Pasos siguientes

* Consulte [Ejecución de runbooks en Hybrid Runbook Worker](automation-hrw-run-runbooks.md) para más información sobre cómo configurar los runbooks para automatizar los procesos del centro de datos local o de otros entornos de nube.

* Para aprender a solucionar problemas de las instancias de Hybrid Runbook Worker, consulte [Solución de problemas de Hybrid Runbook Worker: Linux](troubleshoot/hybrid-runbook-worker.md#linux).