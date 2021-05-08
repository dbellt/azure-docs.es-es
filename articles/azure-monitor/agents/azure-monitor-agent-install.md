---
title: Instalación del agente de Azure Monitor
description: Opciones para instalar el agente de Azure Monitor (AMA) en máquinas virtuales de Azure y servidores habilitados para Azure Arc.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 11/17/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8498c3a867a17685432fae7f7613c1e9c0f2cdd1
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108321220"
---
# <a name="install-the-azure-monitor-agent-preview"></a>Instalación del agente de Azure Monitor (versión preliminar)
En este artículo se proporcionan las distintas opciones disponibles actualmente para instalar el [agente de Azure Monitor](azure-monitor-agent-overview.md) en máquinas virtuales de Azure y en los servidores habilitados para Azure Arc, así como las opciones para crear [asociaciones con reglas de recopilación de datos](data-collection-rule-azure-monitor-agent.md) que definen qué datos debe recopilar el agente.

## <a name="prerequisites"></a>Prerrequisitos
Antes de instalar el agente de Azure Monitor, debe cumplir con los siguientes requisitos previos.

- La [identidad administrada asignada por el sistema](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md) debe estar habilitada en las máquinas virtuales de Azure. Esto no es necesario en el caso de los servidores habilitados para Azure Arc. La identidad del sistema se habilitará automáticamente si el agente se instala como parte del proceso de [creación y asignación de una regla de recopilación de datos mediante Azure Portal](#install-with-azure-portal).
- La [etiqueta de servicio AzureResourceManager](../../virtual-network/service-tags-overview.md) debe estar habilitada en la red virtual de la máquina virtual.

> [!IMPORTANT]
> El agente de Azure Monitor no admite actualmente servidores proxy de red.

## <a name="virtual-machine-extension-details"></a>Detalles de la extensión de máquina virtual
El agente de Azure Monitor se implementa como [extensión de VM de Azure](../../virtual-machines/extensions/overview.md) con los detalles de la tabla siguiente. Se puede instalar con cualquiera de los métodos para instalar las extensiones de máquina virtual, incluidos los descritos en este artículo.

| Propiedad | Windows | Linux |
|:---|:---|:---|
| Publicador | Microsoft.Azure.Monitor  | Microsoft.Azure.Monitor |
| Tipo      | AzureMonitorWindowsAgent | AzureMonitorLinuxAgent  |
| TypeHandlerVersion  | 1.0 | 1.5 |


## <a name="install-with-azure-portal"></a>Instalación con Azure Portal
Para instalar el agente de Azure Monitor mediante Azure Portal, siga el proceso para [crear una regla de recopilación de datos](data-collection-rule-azure-monitor-agent.md#create-rule-and-association-in-azure-portal) en Azure Portal. Esto le permite asociar la regla de recopilación de datos con una o varias máquinas virtuales de Azure o servidores habilitados para Azure Arc. El agente se instalará en cualquiera de estas máquinas virtuales que no lo tengan aún.


## <a name="install-with-resource-manager-template"></a>Instalación con una plantilla de Resource Manager
Puede usar las plantillas de Resource Manager para instalar el agente de Azure Monitor en máquinas virtuales de Azure y en servidores habilitados para Azure Arc, así como para crear una asociación con las reglas de recopilación de datos. Debe crear todas las reglas de recopilación de datos antes de crear la asociación.

Obtenga plantillas de ejemplo para instalar el agente y crear la asociación de los siguientes vínculos: 

- [Plantilla para instalar el agente de Azure Monitor (Azure y Azure Arc)](../agents/resource-manager-agent.md#azure-monitor-agent-preview) 
- [Plantilla para crear una asociación con una regla de recopilación de datos](./resource-manager-data-collection-rules.md)

Instale las plantillas con [cualquier método de implementación para las plantillas de Resource Manager](../../azure-resource-manager/templates/deploy-powershell.md), como los siguientes comandos.

# <a name="powershell"></a>[PowerShell](#tab/ARMAgentPowerShell)
```powershell
New-AzResourceGroupDeployment -ResourceGroupName "<resource-group-name>" -TemplateFile "<template-filename.json>" -TemplateParameterFile "<parameter-filename.json>"
```
# <a name="cli"></a>[CLI](#tab/ARMAgentCLI)
```powershell
New-AzResourceGroupDeployment -ResourceGroupName "<resource-group-name>" -TemplateFile "<template-filename.json>" -TemplateParameterFile "<parameter-filename.json>"
```
---

## <a name="install-with-powershell"></a>Instalación con PowerShell
Puede instalar el agente de Azure Monitor en máquinas virtuales de Azure y en servidores habilitados para Azure Arc mediante el comando de PowerShell para agregar una extensión de máquina virtual. 

### <a name="azure-virtual-machines"></a>Azure Virtual Machines
Use los siguientes comandos de PowerShell para instalar el agente de Azure Monitor en máquinas virtuales de Azure.
# <a name="windows"></a>[Windows](#tab/PowerShellWindows)
```powershell
Set-AzVMExtension -Name AMAWindows -ExtensionType AzureMonitorWindowsAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName <resource-group-name> -VMName <virtual-machine-name> -Location <location> -TypeHandlerVersion 1.0
```
# <a name="linux"></a>[Linux](#tab/PowerShellLinux)
```powershell
Set-AzVMExtension -Name AMALinux -ExtensionType AzureMonitorLinuxAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName <resource-group-name> -VMName <virtual-machine-name> -Location <location> -TypeHandlerVersion 1.0
```
---

### <a name="azure-arc-enabled-servers"></a>Servidores habilitados para Azure Arc
Use los siguientes comandos de PowerShell para instalar el agente de Azure Monitor en los servidores habilitados para Azure Arc.
# <a name="windows"></a>[Windows](#tab/PowerShellWindowsArc)
```powershell
New-AzConnectedMachineExtension -Name AMAWindows -ExtensionType AzureMonitorWindowsAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName <resource-group-name> -MachineName <virtual-machine-name> -Location <location>
```
# <a name="linux"></a>[Linux](#tab/PowerShellLinuxArc)
```powershell
New-AzConnectedMachineExtension -Name AMALinux -ExtensionType AzureMonitorLinuxAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName <resource-group-name> -MachineName <virtual-machine-name> -Location <location>
```
---
## <a name="azure-cli"></a>Azure CLI
Puede instalar el agente de Azure Monitor en máquinas virtuales de Azure y en servidores habilitados para Azure Arc mediante el comando de la CLI de Azure para agregar una extensión de máquina virtual. 

### <a name="azure-virtual-machines"></a>Azure Virtual Machines
Use los siguientes comandos de la CLI para instalar el agente de Azure Monitor en máquinas virtuales de Azure.
# <a name="windows"></a>[Windows](#tab/CLIWindows)
```azurecli
az vm extension set --name AzureMonitorWindowsAgent --publisher Microsoft.Azure.Monitor --ids <vm-resource-id>
```
# <a name="linux"></a>[Linux](#tab/CLILinux)
```azurecli
az vm extension set --name AzureMonitorLinuxAgent --publisher Microsoft.Azure.Monitor --ids <vm-resource-id>
```
---
### <a name="azure-arc-enabled-servers"></a>Servidores habilitados para Azure Arc
Use los siguientes comandos de la CLI para instalar el agente de Azure Monitor en los servidores habilitados para Azure Arc.

# <a name="windows"></a>[Windows](#tab/CLIWindowsArc)
```azurecli
az connectedmachine machine-extension create --name AzureMonitorWindowsAgent --publisher Microsoft.Azure.Monitor --ids <vm-resource-id>
```
# <a name="linux"></a>[Linux](#tab/CLILinuxArc)
```azurecli
az connectedmachine machine-extension create --name AzureMonitorLinuxAgent --publisher Microsoft.Azure.Monitor --ids <vm-resource-id>
```
---


## <a name="next-steps"></a>Pasos siguientes

- [Cree una regla de recopilación de datos](data-collection-rule-azure-monitor-agent.md) para recopilar datos del agente y enviarlos a Azure Monitor.
