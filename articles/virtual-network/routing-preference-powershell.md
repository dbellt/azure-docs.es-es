---
title: Configuración de la preferencia de enrutamiento para una dirección IP pública - Azure PowerShell
titlesuffix: Azure Virtual Network
description: Obtenga información para configurar la preferencia de enrutamiento para una dirección IP pública mediante Azure PowerShell.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2021
ms.author: mnayak
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 2f8db32c8e4619b61439cb13b4aca1ad6ed25862
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110672542"
---
# <a name="configure-routing-preference-for-a-public-ip-address-using-azure-powershell"></a>Configuración de la preferencia de enrutamiento para una dirección IP pública mediante Azure PowerShell

En este artículo se muestra cómo configurar las preferencias de enrutamiento a través de la red de ISP (opción de **Internet**) para una dirección IP pública mediante Azure PowerShell. Después de crear la dirección IP pública, puede asociarla a los siguientes recursos de Azure para el tráfico entrante y saliente de Internet:

* Máquina virtual
* Conjunto de escalado de máquina virtual
* Azure Kubernetes Service (AKS)
* Equilibrador de carga accesible desde Internet
* Application Gateway
* Azure Firewall

De forma predeterminada, la preferencia de enrutamiento de la dirección IP pública se establece en la red global de Microsoft para todos los servicios de Azure y puede asociarse a cualquier servicio de Azure.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) ahora.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]
Si decide instalar y usar PowerShell de forma local, la versión del módulo de Azure PowerShell que necesita este artículo es la 6.9.0 u otra posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.Resources/New-azResourceGroup). En este ejemplo, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurepowershell
$rg = New-AzResourceGroup -Name myResourceGroup -Location EastUS
```

## <a name="create-a-public-ip-with-internet-routing-preference"></a>Creación de una dirección IP pública con preferencia de enrutamiento de Internet

El siguiente comando crea una nueva dirección IP pública con preferencia de enrutamiento de tipo *Internet* en la región de Azure del *Este de EE. UU.* :

```azurepowershell
$iptagtype="RoutingPreference"
$tagName = "Internet"
$ipTag = New-AzPublicIpTag -IpTagType $iptagtype -Tag $tagName 
# attach the tag
$publicIp = New-AzPublicIpAddress  `
-Name "MyPublicIP" `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location `
-IpTag $ipTag `
-AllocationMethod Static `
-Sku Standard `
-IpAddressVersion IPv4
```

Puede asociar la dirección IP pública creada anteriormente con una máquina virtual de [Windows](../virtual-machines/windows/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o [Linux](../virtual-machines/linux/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Use la sección CLI en la siguiente página del tutorial: [asociación de una dirección IP pública a una máquina virtual](associate-public-ip-address-vm.md#azure-cli) para asociar la dirección IP pública a la VM. Asimismo, puede asociar una dirección IP pública que haya creado con anterioridad con una instancia de [Azure Load Balancer](../load-balancer/load-balancer-overview.md) asignándola a la configuración de **front-end** del equilibrador de carga. La dirección IP pública actúa como dirección IP virtual (VIP) de carga equilibrada.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos, la VM y todos los recursos relacionados.

```azurepowershell
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre la [preferencia de enrutamiento en direcciones IP públicas](routing-preference-overview.md).
- [Configuración de la preferencia de enrutamiento de una VM mediante Azure PowerShell](configure-routing-preference-virtual-machine-powershell.md).
