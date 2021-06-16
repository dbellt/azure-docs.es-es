---
title: 'Tutorial: Creación de un equilibrador de carga entre regiones mediante Azure PowerShell'
titleSuffix: Azure Load Balancer
description: Comience a trabajar con este tutorial para la implementación de Azure Load Balancer entre regiones con Azure PowerShell.
author: asudbring
ms.author: allensu
ms.service: load-balancer
ms.topic: tutorial
ms.date: 02/10/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 0f6ee0323768287ee860becc5d02c85efb32e3fe
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110704729"
---
# <a name="tutorial-create-a-cross-region-azure-load-balancer-using-azure-powershell"></a>Tutorial: Creación de una instancia de Azure Load Balancer entre regiones mediante Azure PowerShell

Un equilibrador de carga entre regiones garantiza que un servicio está disponible globalmente en varias regiones de Azure. Si se produce un error en una región, el tráfico se enruta al siguiente equilibrador de carga regional correcto más cercano.  

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un equilibrador de carga entre regiones.
> * Cree una regla de equilibrador de carga.
> * Crear un grupo de back-end que contenga dos equilibradores de carga regionales.
> * Probar el equilibrador de carga.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

- Suscripción a Azure.
- Dos instancias de Azure Load Balancer con SKU **Estándar** con grupos de back-end implementados en dos regiones de Azure diferentes.
    - Para obtener información sobre cómo crear un equilibrador de carga estándar regional y máquinas virtuales para los grupos de back-end, consulte [Inicio rápido: Creación de un equilibrador de carga público para equilibrar la carga de las VM con Azure PowerShell](quickstart-load-balancer-standard-public-powershell.md).
        - Anexe **-R1** y **-R2** al nombre de los equilibradores de carga y las máquinas virtuales de cada región. 
- Azure PowerShell instalado localmente o Azure Cloud Shell.


Si decide instalar y usar PowerShell de forma local, para realizar los pasos de este artículo necesita la versión 5.4.1 del módulo de Azure PowerShell o cualquier versión posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="create-cross-region-load-balancer"></a>Creación de un equilibrador de carga entre regiones


### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

Cree un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).


```azurepowershell-interactive
$rg = @{
    Name = 'MyResourceGroupLB-CR'
    Location = 'westus'
}
New-AzResourceGroup @rg

```

### <a name="create-cross-region-load-balancer-resources"></a>Creación de recursos de equilibrador de carga entre regiones

En esta sección va a crear los recursos necesarios para el equilibrador de carga entre regiones.

Se usa una dirección IP pública de SKU estándar global para el front-end del equilibrador de carga entre regiones.

* Para crear la dirección IP pública, use [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress).

* Cree una configuración IP de front-end con [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig).

* Cree un grupo de direcciones de back-end con [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig).

* Cree una regla del equilibrador de carga con [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/add-azloadbalancerruleconfig).

* Cree un equilibrador de carga entre regiones con [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer).

```azurepowershell-interactive
## Create global IP address for load balancer ##
$ip = @{
    Name = 'myPublicIP-CR'
    ResourceGroupName = 'MyResourceGroupLB-CR'
    Location = 'westus'
    Sku = 'Standard'
    Tier = 'Global'
    AllocationMethod = 'Static'
}
$publicIP = New-AzPublicIpAddress @ip

## Create frontend configuration ##
$fe = @{
    Name = 'myFrontEnd-CR'
    PublicIpAddress = $publicIP
}
$feip = New-AzLoadBalancerFrontendIpConfig @fe

## Create back-end address pool ##
$be = @{
    Name = 'myBackEndPool-CR'
}
$bepool = New-AzLoadBalancerBackendAddressPoolConfig @be

## Create the load balancer rule ##
$rul = @{
    Name = 'myHTTPRule-CR'
    Protocol = 'tcp'
    FrontendPort = '80'
    BackendPort = '80'
    FrontendIpConfiguration = $feip
    BackendAddressPool = $bepool
}
$rule = New-AzLoadBalancerRuleConfig @rul

## Create cross-region load balancer resource ##
$lbp = @{
    ResourceGroupName = 'myResourceGroupLB-CR'
    Name = 'myLoadBalancer-CR'
    Location = 'westus'
    Sku = 'Standard'
    Tier = 'Global'
    FrontendIpConfiguration = $feip
    BackendAddressPool = $bepool
    LoadBalancingRule = $rule
}
$lb = New-AzLoadBalancer @lbp
```

## <a name="configure-backend-pool"></a>Configuración del grupo back-end

En esta sección va a agregar dos equilibradores de carga estándar regionales al grupo de back-end del equilibrador de carga entre regiones.

> [!IMPORTANT]
> Para completar estos pasos, asegúrese de que se han implementado en la suscripción dos equilibradores de carga regionales con grupos de back-end.  Para más información, consulte **[Inicio rápido: Creación de un equilibrador de carga público para equilibrar la carga de las VM con Azure PowerShell](quickstart-load-balancer-standard-public-powershell.md)** .

* Use [Get-AzLoadBalancer](/powershell/module/az.network/get-azloadbalancer) y [Get-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/get-azloadbalancerfrontendipconfig) para almacenar la información del equilibrador de carga regional en variables.

* Use [New-AzLoadBalancerBackendAddressConfig](/powershell/module/az.network/new-azloadbalancerbackendaddressconfig) para crear la configuración de grupo de direcciones de back-end para el equilibrador de carga.

* Use [Set-AzLoadBalancerBackendAddressPool](/powershell/module/az.network/new-azloadbalancerbackendaddresspool) para agregar el front-end del equilibrador de carga regional al grupo de back-end entre regiones.

```azurepowershell-interactive
## Place the region one load balancer configuration in a variable ##
$region1 = @{
    Name = 'myLoadBalancer-R1'
    ResourceGroupName = 'CreatePubLBQS-rg-r1'
}
$R1 = Get-AzLoadBalancer @region1

## Place the region two load balancer configuration in a variable ##
$region2 = @{
    Name = 'myLoadBalancer-R2'
    ResourceGroupName = 'CreatePubLBQS-rg-r2'
}
$R2 = Get-AzLoadBalancer @region2

## Place the region one load balancer front-end configuration in a variable ##
$region1fe = @{
    Name = 'MyFrontEnd-R1'
    LoadBalancer = $R1
}
$R1FE = Get-AzLoadBalancerFrontendIpConfig @region1fe

## Place the region two load balancer front-end configuration in a variable ##
$region2fe = @{
    Name = 'MyFrontEnd-R2'
    LoadBalancer = $R2
}
$R2FE = Get-AzLoadBalancerFrontendIpConfig @region2fe

## Create the cross-region backend address pool configuration for region 1 ##
$region1ap = @{
    Name = 'MyBackendPoolConfig-R1'
    LoadBalancerFrontendIPConfigurationId = $R1FE.Id
}
$beaddressconfigR1 = New-AzLoadBalancerBackendAddressConfig @region1ap

## Create the cross-region backend address pool configuration for region 2 ##
$region2ap = @{
    Name = 'MyBackendPoolConfig-R2'
    LoadBalancerFrontendIPConfigurationId = $R2FE.Id
}
$beaddressconfigR2 = New-AzLoadBalancerBackendAddressConfig @region2ap

## Apply the backend address pool configuration for the cross-region load balancer ##
$bepoolcr = @{
    ResourceGroupName = 'myResourceGroupLB-CR'
    LoadBalancerName = 'myLoadBalancer-CR'
    Name = 'myBackEndPool-CR'
    LoadBalancerBackendAddress = $beaddressconfigR1,$beaddressconfigR2
}
Set-AzLoadBalancerBackendAddressPool @bepoolcr

```

## <a name="test-the-load-balancer"></a>Prueba del equilibrador de carga

En esta sección va a probar el equilibrador de carga entre regiones. Se conectará a la dirección IP pública en un explorador web.  Detendrá las máquinas virtuales de uno de los grupos de back-end del equilibrador de carga regional y observará la conmutación por error.

1. Use [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) para obtener la dirección IP pública del equilibrador de carga:

```azurepowershell-interactive
$ip = @{
    Name = 'myPublicIP-CR'
    ResourceGroupName = 'myResourceGroupLB-CR'
}  
Get-AzPublicIPAddress @ip | select IpAddress

```
2. Copie la dirección IP pública y péguela en la barra de direcciones del explorador. La página predeterminada del servidor web de IIS se muestra en el explorador.

3. Detenga las máquinas virtuales del grupo de back-end de uno de los equilibradores de carga regionales.

4. Actualice el explorador web y observe la conmutación por error de la conexión al otro equilibrador de carga regional.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos, el equilibrador de carga y el resto de los recursos.

```azurepowershell-interactive
Remove-AzResourceGroup -Name 'myResourceGroupLB-CR'
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, hizo lo siguiente:

* Ha creado una dirección IP global.
* Creado un equilibrador de carga entre regiones.
* Creado una regla de equilibrio de carga.
* Agregado equilibradores de carga regionales al grupo de back-end del equilibrador de carga entre regiones.
* Probó el equilibrador de carga.


Avance al siguiente artículo para obtener información sobre...
> [!div class="nextstepaction"]
> [Equilibrio de carga de máquinas virtuales en distintas zonas de disponibilidad con Load Balancer Estándar mediante Azure Portal](tutorial-load-balancer-standard-public-zone-redundant-portal.md)
