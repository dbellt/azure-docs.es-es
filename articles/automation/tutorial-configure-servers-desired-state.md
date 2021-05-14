---
title: Configuración de máquinas en un estado deseado de Azure Automation
description: En este artículo se indica cómo configurar las máquinas en un estado deseado con State Configuration de Azure Automation.
services: automation
ms.subservice: dsc
ms.topic: conceptual
ms.date: 04/15/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: d29c8ec4e0b992f38eec9e203ad6ad302f71308b
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108018500"
---
# <a name="configure-machines-to-a-desired-state"></a>Configuración de máquinas en un estado deseado

Azure Automation State Configuration permite especificar configuraciones para los servidores y asegurarse de que dichos servidores se encuentran en el estado especificado a lo largo del tiempo.

> [!div class="checklist"]
> - Incorporación de una máquina virtual para su administración por DSC de Azure Automation
> - Carga de una configuración en Azure Automation
> - Compilación de una configuración en una configuración de nodo
> - Asignación de una configuración de nodo a un nodo administrado
> - Comprobación del estado de cumplimiento de un nodo administrado

Para este tutorial, se usa una [configuración DSC](/powershell/scripting/dsc/configurations/configurations) simple que se asegura de que IIS está instalado en la máquina virtual.

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure Automation Para más información sobre las cuentas de Automation y sus requisitos, consulte [Introducción a la autenticación de cuentas de Automation](./automation-security-overview.md).
- Una máquina virtual de Azure Resource Manager (que no es clásica) ejecuta Windows Server 2008 R2, o cualquier versión posterior. Para obtener instrucciones sobre la creación de una máquina virtual, consulte [Creación de la primera máquina virtual de Windows en Azure Portal](../virtual-machines/windows/quick-create-portal.md).
- Módulo Azure PowerShell, versión 3.6 o posterior. Ejecute `Get-Module -ListAvailable Az` para encontrar la versión. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/azurerm/install-azurerm-ps).
- Familiaridad con Desired State Configuration (DSC). Para obtener información sobre DSC, consulte [Información general sobre la configuración de estado deseado de Windows PowerShell](/powershell/scripting/dsc/overview/overview).

## <a name="support-for-partial-configurations"></a>Compatibilidad con configuraciones parciales

State Configuration de Azure Automation admite el uso de [configuraciones parciales](/powershell/scripting/dsc/pull-server/partialconfigs). En este escenario, DSC está configurado para administrar varias configuraciones de forma independiente, y cada configuración se recupera de Azure Automation. Sin embargo, solo se puede asignar una configuración a un nodo por cuenta de Automation. Esto significa que si usa dos configuraciones para un nodo, necesitará de dos cuentas de Automation.

Para obtener más información sobre cómo registrar una configuración parcial de un servicio de extracción, consulte la documentación sobre [configuraciones parciales](/powershell/scripting/dsc/pull-server/partialconfigs#partial-configurations-in-pull-mode).

Para obtener más información acerca de cómo los equipos pueden trabajar juntos para administrar de forma colaborativa los servidores con la configuración como código, consulte [Descripción del rol de DSC en una canalización de CI/CD](/powershell/scripting/dsc/overview/authoringadvanced).

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en la suscripción a Azure con el cmdlet [Connect-AzAccount](/powershell/module/Az.Accounts/Connect-AzAccount) y siga las instrucciones de la pantalla.

```powershell
Connect-AzAccount
```

## <a name="create-and-upload-a-configuration-to-azure-automation"></a>Creación y carga de una configuración en Azure Automation

En un editor de texto, escriba lo siguiente y guárdelo localmente como **TestConfig.ps1**.

```powershell
configuration TestConfig {
   Node WebServer {
      WindowsFeature IIS {
         Ensure               = 'Present'
         Name                 = 'Web-Server'
         IncludeAllSubFeature = $true
      }
   }
}
```

> [!NOTE]
> En escenarios más avanzados en los que se requiere la importación de varios módulos que proporcionen recursos de DSC, asegúrese de que cada módulo tenga una línea única `Import-DscResource` en la configuración.

Llame al cmdlet [Import-AzAutomationDscConfiguration](/powershell/module/Az.Automation/Import-AzAutomationDscConfiguration) para cargar una configuración de nodo en la cuenta de Automation.

```powershell
 Import-AzAutomationDscConfiguration -SourcePath 'C:\DscConfigs\TestConfig.ps1' -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -Published
```

## <a name="compile-a-configuration-into-a-node-configuration"></a>Compilación de una configuración en una configuración de nodo

Una configuración DSC se debe compilar en una configuración de nodo antes de que se pueda asignar a un nodo. Consulte las [configuraciones DSC](/powershell/scripting/dsc/configurations/configurations).

Llame al cmdlet [Start-AzAutomationDscCompilationJob](/powershell/module/Az.Automation/Start-AzAutomationDscCompilationJob) para compilar la configuración `TestConfig` en una configuración de nodo denominada `TestConfig.WebServer` de la cuenta de Automation.

```powershell
Start-AzAutomationDscCompilationJob -ConfigurationName 'TestConfig' -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount'
```

## <a name="register-a-vm-to-be-managed-by-state-configuration"></a>Registro de una máquina virtual para su administración mediante State Configuration

Puede usar Azure Automation State Configuration para administrar máquinas virtuales de Azure (tanto clásicas como de Resource Manager), máquinas virtuales locales, máquinas Linux, máquinas virtuales de AWS y máquinas físicas locales. En este tema, se explicará cómo registrar solo máquinas virtuales de Azure Resource Manager. Para más información sobre el registro de otros tipos de máquinas, consulte [Incorporación de máquinas para administrarlas con Azure Automation State Configuration](automation-dsc-onboarding.md).

Llame al cmdlet [Register-AzAutomationDscNode](/powershell/module/Az.Automation/Register-AzAutomationDscNode) para registrar la máquina virtual con State Configuration de Azure Automation como nodo administrado. 

```powershell
Register-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -AzureVMName 'DscVm'
```

### <a name="specify-configuration-mode-settings"></a>Especificación de opciones del modo de configuración

Use el cmdlet [Register-AzAutomationDscNode](/powershell/module/azurerm.automation/register-azurermautomationdscnode) para registrar una máquina virtual como un nodo administrado y especificar las propiedades de configuración. Por ejemplo, puede especificar que el estado de la máquina se aplique solo una vez; para ello, especifique `ApplyOnly` como valor de la propiedad `ConfigurationMode`. State Configuration no intenta aplicar la configuración después de la comprobación inicial.

```powershell
Register-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -AzureVMName 'DscVm' -ConfigurationMode 'ApplyOnly'
```

También puede especificar la frecuencia con la que DSC comprueba el estado de configuración mediante la propiedad `ConfigurationModeFrequencyMins`. Para más información sobre las opciones de configuración de DSC, consulte [Configuración del administrador de configuración local](/powershell/scripting/dsc/managing-nodes/metaConfig).

```powershell
# Run a DSC check every 60 minutes
Register-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -AzureVMName 'DscVm' -ConfigurationModeFrequencyMins 60
```

## <a name="assign-a-node-configuration-to-a-managed-node"></a>Asignación de una configuración de nodo a un nodo administrado

Ahora se puede asignar la configuración de nodo compilada a la máquina virtual que desea configurar.

```powershell
# Get the ID of the DSC node
$node = Get-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -Name 'DscVm'

# Assign the node configuration to the DSC node
Set-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -NodeConfigurationName 'TestConfig.WebServer' -NodeId $node.Id
```

Esto asigna la configuración de nodo llamada `TestConfig.WebServer` al nodo DSC registrado `DscVm`. De forma predeterminada, se comprueba el cumplimiento de la configuración del nodo DSC cada 30 minutos. Para obtener información sobre cómo cambiar el intervalo de comprobación de cumplimiento, consulte [Configuración del administrador de configuración Local](/powershell/scripting/dsc/managing-nodes/metaConfig).

## <a name="check-the-compliance-status-of-a-managed-node"></a>Comprobación del estado de cumplimiento de un nodo administrado

Puede obtener informes sobre el estado de cumplimiento de un nodo administrado mediante el cmdlet [Get-AzAutomationDscNodeReport](/powershell/module/Az.Automation/Get-AzAutomationDscNodeReport).

```powershell
# Get the ID of the DSC node
$node = Get-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -Name 'DscVm'

# Get an array of status reports for the DSC node
$reports = Get-AzAutomationDscNodeReport -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -NodeId $node.Id

# Display the most recent report
$reports[0]
```

## <a name="next-steps"></a>Pasos siguientes

- Para dar los primeros pasos, consulte [Introducción a Azure Automation State Configuration](automation-dsc-getting-started.md).
- Para aprender a habilitar nodos, consulte el artículo sobre [Habilitar Azure Automation State Configuration](automation-dsc-onboarding.md).
- Para aprender a compilar configuraciones de DSC para poder asignarlas a los nodos de destino, consulte [Compilación de configuraciones de DSC en Azure Automation State Configuration](automation-dsc-compile.md).
- Para ver un ejemplo del uso de State Configuration de Azure Automation en una canalización de implementación continua, consulte [Configuración de la implementación continua con Chocolatey](automation-dsc-cd-chocolatey.md).
- Para obtener información de precios, consulte [Precios de State Configuration de Azure Automation](https://azure.microsoft.com/pricing/details/automation/).
- Para ver una referencia de los cmdlets de PowerShell, consulte [Az.Automation](/powershell/module/az.automation).
