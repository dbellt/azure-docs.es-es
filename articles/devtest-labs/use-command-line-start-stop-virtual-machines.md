---
title: Uso de las herramientas de la línea de comandos para iniciar y detener las máquinas virtuales en Azure DevTest Labs
description: Aprenda a usar las herramientas de la línea de comandos para iniciar y detener las máquinas virtuales en Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 62d4a33df13b1eb5cc7a0360a90ab92c2914b1f0
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110692455"
---
# <a name="use-command-line-tools-to-start-and-stop-azure-devtest-labs-virtual-machines"></a>Uso de las herramientas de la línea de comandos para iniciar y detener las máquinas virtuales en Azure DevTest Labs
En este artículo se muestra cómo usar Azure PowerShell o la CLI de Azure para iniciar o detener las máquinas virtuales en un laboratorio de Azure DevTest Labs. Puede crear scripts de PowerShell o CLI para automatizar estas operaciones. 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="overview"></a>Información general
Azure DevTest Labs es una forma sencilla de crear entornos de desarrollo y pruebas rápidos, sencillos y ligeros. Permite administrar los costos, aprovisionar máquinas virtuales rápidamente y minimizar el desperdicio.  En Azure Portal hay características integradas que le permiten configurar las máquinas virtuales en un laboratorio para que se inicien y detengan automáticamente en momentos específicos. 

Sin embargo, en algunos escenarios, puede que desee automatizar el inicio y la detención de las máquinas virtuales desde los scripts de PowerShell o CLI. Ofrece cierta flexibilidad con el inicio y la detención de las máquinas individuales en cualquier momento en lugar de en momentos específicos. Estas son algunas de las situaciones en las que ejecutar estas tareas mediante el uso de scripts sería útil.

- Cuando se usa una aplicación de 3 niveles como parte de un entorno de prueba, los niveles deben iniciarse en secuencia. 
- Desactivar una máquina virtual cuando se cumple un criterio personalizado para ahorrar dinero. 
- Usarlo como una tarea dentro de un flujo de trabajo de CI/CD para comenzar al principio del flujo, usar las máquinas virtuales como equipos de compilación, equipos de prueba o infraestructura y luego detener dichas máquinas virtuales cuando el proceso se completa. Un ejemplo de esto sería la fábrica de imágenes personalizada con Azure DevTest Labs.  

## <a name="azure-powershell"></a>Azure PowerShell

> [!NOTE]
> El script siguiente usa el módulo Az de Azure PowerShell. 

El siguiente script de PowerShell inicia una máquina virtual de un laboratorio. [Invoke-AzResourceAction](/powershell/module/az.resources/invoke-azresourceaction) es el enfoque principal para este script. El parámetro **ResourceId** es el identificador de recurso completo para la máquina virtual del laboratorio. El parámetro **Action** es donde las opciones **Start** o **Stop** se establecen según lo que se necesite.

```powershell
# The id of the subscription
$subscriptionId = "111111-11111-11111-1111111"

# The name of the lab
$devTestLabName = "yourlabname"

# The name of the virtual machine to be started
$vMToStart = "vmname"

# The action on the virtual machine (Start or Stop)
$vmAction = "Start"

# Select the Azure subscription
Select-AzSubscription -SubscriptionId $subscriptionId

# Get the lab information
$devTestLab = Get-AzResource -ResourceType 'Microsoft.DevTestLab/labs' -ResourceName $devTestLabName

# Start the VM and return a succeeded or failed status
$returnStatus = Invoke-AzResourceAction `
                    -ResourceId "$($devTestLab.ResourceId)/virtualmachines/$vMToStart" `
                    -Action $vmAction `
                    -Force

if ($returnStatus.Status -eq 'Succeeded') {
    Write-Output "##[section] Successfully updated DTL machine: $vMToStart, Action: $vmAction"
}
else {
    Write-Error "##[error]Failed to update DTL machine: $vMToStart, Action: $vmAction"
}
```


## <a name="azure-cli"></a>Azure CLI
La [CLI de Azure](/cli/azure/get-started-with-azure-cli) es otra manera de automatizar el inicio y la detención de máquinas virtuales de DevTest Labs. La CLI de Azure se puede [instalar](/cli/azure/install-azure-cli) en diferentes sistemas operativos. El siguiente script proporciona comandos para iniciar y detener una máquina virtual de un laboratorio. 

```azurecli
# Sign in to Azure
az login 

## Get the name of the resource group that contains the lab
az resource list --resource-type "Microsoft.DevTestLab/labs" --name "yourlabname" --query "[0].resourceGroup" 

## Start the VM
az lab vm start --lab-name yourlabname --name vmname --resource-group labResourceGroupName

## Stop the VM
az lab vm stop --lab-name yourlabname --name vmname --resource-group labResourceGroupName
```


## <a name="next-steps"></a>Pasos siguientes
Consulte el artículo siguiente para usar Azure Portal para realizar estas operaciones: [Reinicio de una máquina virtual](devtest-lab-restart-vm.md).
