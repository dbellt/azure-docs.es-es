---
title: Creación y configuración de un plan de Azure DDoS Protection mediante Azure PowerShell
description: Aprenda a crear un plan de DDoS Protection mediante Azure PowerShell
services: ddos-protection
documentationcenter: na
author: aletheatoh
ms.service: ddos-protection
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/28/2020
ms.author: yitoh
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: f48848edc6e42ccf39f608b103deb4311810d177
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110702650"
---
# <a name="quickstart-create-and-configure-azure-ddos-protection-standard-using-azure-powershell"></a>Inicio rápido: Creación y configuración de Azure DDoS Protection Estándar mediante Azure PowerShell

Introducción a Azure DDoS Protection Estándar mediante Azure PowerShell. 

Un plan de protección contra DDoS define un conjunto de redes virtuales que tiene habilitada la protección contra DDoS estándar en distintas suscripciones. Puede configurar un plan de protección contra DDoS para la organización y vincular redes virtuales de distintas suscripciones al mismo plan. 

En este inicio rápido, creará un plan de protección contra DDoS y lo vinculará a una red virtual. 

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Azure PowerShell instalado localmente o Azure Cloud Shell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-a-ddos-protection-plan"></a>Creación de un plan de DDoS Protection

En Azure, puede asignar recursos relacionados a un grupo de recursos. Puede usar un grupo de recursos existente o crear uno nuevo.

Para crear un grupo de recursos, use [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). En este ejemplo, se asignará el nombre _MyResourceGroup_ al grupo de recursos y se usará la ubicación _Este de EE. UU._ :

```azurepowershell-interactive
New-AzResourceGroup -Name MyResourceGroup -Location "East US"
```

Ahora, cree un plan de protección contra DDoS denominado _MyDdosProtectionPlan_:

```azurepowershell-interactive
New-AzDdosProtectionPlan -ResourceGroupName MyResourceGroup -Name MyDdosProtectionPlan -Location "East US"
```

## <a name="enable-ddos-for-a-virtual-network"></a>Habilitación de DDoS para una red virtual

### <a name="enable-ddos-for-a-new-virtual-network"></a>Habilitación de DDoS para una red virtual nueva

Puede habilitar la protección contra DDoS al crear una red virtual. En este ejemplo, se asignará el nombre _MyVnet_ a la red virtual: 

```azurepowershell-interactive
New-AzVirtualNetwork -Name MyVnet -ResourceGroupName MyResourceGroup -Location "East US" -AddressPrefix 10.0.0.0/16
```

### <a name="enable-ddos-for-an-existing-virtual-network"></a>Habilitación de DDoS para una red virtual existente

Puede asociar una red virtual existente al crear un plan de protección contra DDoS:

```azurepowershell-interactive
# Creates the DDoS protection plan
$ddosProtectionPlan = New-AzDdosProtectionPlan -ResourceGroupName MyResourceGroup -Name MyDdosProtectionPlan -Location "East US"

# Gets the most updated version of the virtual network
$vnet = Get-AzVirtualNetwork -Name MyVnet -ResourceGroupName MyResourceGroup
$vnet.DdosProtectionPlan = New-Object Microsoft.Azure.Commands.Network.Models.PSResourceId

# Update the properties and enable DDoS protection
$vnet.DdosProtectionPlan.Id = $ddosProtectionPlan.Id
$vnet.EnableDdosProtection = $true
$vnet | Set-AzVirtualNetwork
``` 

## <a name="validate-and-test"></a>Validación y pruebas

En primer lugar, compruebe los detalles del plan de protección contra DDoS:

```azurepowershell-interactive
Get-AzDdosProtectionPlan -ResourceGroupName MyResourceGroup -Name MyDdosProtectionPlan
```

Compruebe que el comando devuelve los detalles correctos del plan de protección contra DDoS.

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede mantener los recursos para el tutorial siguiente. Si ya no lo necesita, elimine el grupo de recursos _MyResourceGroup_. Al eliminar el grupo de recursos, también elimina el plan de protección contra DDoS y todos sus recursos relacionados. 

```azurepowershell-interactive
Remove-AzResourceGroup -Name MyResourceGroup
```

Para deshabilitar la protección contra DDoS en una red virtual: 

```azurepowershell-interactive
# Gets the most updated version of the virtual network
$vnet = Get-AzVirtualNetwork -Name MyVnet -ResourceGroupName MyResourceGroup
$vnet.DdosProtectionPlan = $null
$vnet.EnableDdosProtection = $false
$vnet | Set-AzVirtualNetwork
```

Si quiere eliminar un plan de protección contra DDoS, primero debe desasociar todas las redes virtuales que tenga.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo ver y configurar la telemetría del plan de protección contra DDoS, continúe con los tutoriales.

> [!div class="nextstepaction"]
> [Visualización y configuración de la telemetría de protección contra DDoS](telemetry.md)
