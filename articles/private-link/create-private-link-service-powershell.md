---
title: 'Inicio rápido: Creación de un servicio de vínculo privado de Azure mediante Azure PowerShell'
description: En este inicio rápido aprenderá a crear un servicio Azure Private Link mediante Azure PowerShell
services: private-link
author: asudbring
ms.service: private-link
ms.topic: quickstart
ms.date: 01/24/2021
ms.author: allensu
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: cc921bbbb78e0d6b53165a3d904d90a09144f9d9
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110696752"
---
# <a name="quickstart-create-a-private-link-service-using-azure-powershell"></a>Inicio rápido: Creación de un servicio Private Link mediante Azure PowerShell

Introducción a la creación de un servicio Private Link que hace referencia a su servicio.  Proporcione acceso de Private Link a su servicio o recurso implementado detrás de una instancia de Standard Load Balancer de Azure.  Los usuarios del servicio tienen acceso privado desde su red virtual.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Azure PowerShell instalado localmente o Azure Cloud Shell

Si decide instalar y usar PowerShell de forma local, para realizar los pasos de este artículo necesita la versión 5.4.1 del módulo de Azure PowerShell o cualquier versión posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

Cree un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup):

```azurepowershell-interactive
New-AzResourceGroup -Name 'CreatePrivLinkService-rg' -Location 'eastus2'

```

## <a name="create-an-internal-load-balancer"></a>Creación de un conjunto de equilibrador de carga interno

En esta sección, va a crear una red virtual y una instancia de Azure Load Balancer interna.

### <a name="virtual-network"></a>Virtual network

En esta sección, va a crear una red virtual y una subred para hospedar el equilibrador de carga que accede al servicio Private Link.

* Cree una red virtual con [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork).

```azurepowershell-interactive
## Create backend subnet config ##
$subnet = @{
    Name = 'mySubnet'
    AddressPrefix = '10.1.0.0/24'
    PrivateLinkServiceNetworkPolicies = 'Disabled'
}
$subnetConfig = New-AzVirtualNetworkSubnetConfig @subnet 

## Create the virtual network ##
$net = @{
    Name = 'myVNet'
    ResourceGroupName = 'CreatePrivLinkService-rg'
    Location = 'eastus2'
    AddressPrefix = '10.1.0.0/16'
    Subnet = $subnetConfig
}
$vnet = New-AzVirtualNetwork @net

```

### <a name="create-standard-load-balancer"></a>Creación de un equilibrador de carga estándar

En esta sección se detalla cómo se pueden crear y configurar los componentes siguientes del equilibrador de carga:

* Cree una dirección IP de front-end con [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig) para el grupo de direcciones IP de front-end. Esta dirección IP recibe el tráfico entrante en el equilibrador de carga.

* Cree un grupo de direcciones de back-end con [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) para el tráfico que se envía desde el front-end del equilibrador de carga. Este grupo es donde se implementan las máquinas virtuales de back-end.

* Cree un sondeo de estado con [Add-AzLoadBalancerProbeConfig](/powershell/module/az.network/add-azloadbalancerprobeconfig) que determine el estado de máquinas virtuales de back-end.

* Cree una regla de equilibrador de carga con [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/add-azloadbalancerruleconfig) que defina cómo se distribuye el tráfico a las máquinas virtuales.

* Cree un equilibrador de carga público con [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer).


```azurepowershell-interactive
## Place virtual network created in previous step into a variable. ##
$vnet = Get-AzVirtualNetwork -Name 'myVNet' -ResourceGroupName 'CreatePrivLinkService-rg'

## Create load balancer frontend configuration and place in variable. ##
$lbip = @{
    Name = 'myFrontEnd'
    PrivateIpAddress = '10.1.0.4'
    SubnetId = $vnet.subnets[0].Id
}
$feip = New-AzLoadBalancerFrontendIpConfig @lbip

## Create backend address pool configuration and place in variable. ##
$bepool = New-AzLoadBalancerBackendAddressPoolConfig -Name 'myBackEndPool'

## Create the health probe and place in variable. ##
$probe = @{
    Name = 'myHealthProbe'
    Protocol = 'http'
    Port = '80'
    IntervalInSeconds = '360'
    ProbeCount = '5'
    RequestPath = '/'
}
$healthprobe = New-AzLoadBalancerProbeConfig @probe

## Create the load balancer rule and place in variable. ##
$lbrule = @{
    Name = 'myHTTPRule'
    Protocol = 'tcp'
    FrontendPort = '80'
    BackendPort = '80'
    IdleTimeoutInMinutes = '15'
    FrontendIpConfiguration = $feip
    BackendAddressPool = $bePool
}
$rule = New-AzLoadBalancerRuleConfig @lbrule -EnableTcpReset

## Create the load balancer resource. ##
$loadbalancer = @{
    ResourceGroupName = 'CreatePrivLinkService-rg'
    Name = 'myLoadBalancer'
    Location = 'eastus2'
    Sku = 'Standard'
    FrontendIpConfiguration = $feip
    BackendAddressPool = $bePool
    LoadBalancingRule = $rule
    Probe = $healthprobe
}
New-AzLoadBalancer @loadbalancer

```

## <a name="create-a-private-link-service"></a>Creación de un servicio Private Link

En esta sección, creará un servicio de vínculo privado que use una instancia de Standard Azure Load Balancer creada en el paso anterior.

* Cree la configuración de IP del servicio de vínculo privado con [New-AzPrivateLinkServiceIpConfig](/powershell/module/az.network/new-azprivatelinkserviceipconfig).

* Cree el servicio de vínculo privado con [New-AzPrivateLinkService](/powershell/module/az.network/new-azprivatelinkservice).

```azurepowershell
## Place the virtual network into a variable. ##
$vnet = Get-AzVirtualNetwork -Name 'myVNet' -ResourceGroupName 'CreatePrivLinkService-rg'

## Create the IP configuration for the private link service. ##
$ipsettings = @{
    Name = 'myIPconfig'
    PrivateIpAddress = '10.1.0.5'
    Subnet = $vnet.subnets[0]
}
$ipconfig = New-AzPrivateLinkServiceIpConfig @ipsettings

## Place the load balancer frontend configuration into a variable. ##
$par = @{
    Name = 'myLoadBalancer'
    ResourceGroupName = 'CreatePrivLinkService-rg'
}
$fe = Get-AzLoadBalancer @par | Get-AzLoadBalancerFrontendIpConfig

## Create the private link service for the load balancer. ##
$privlinksettings = @{
    Name = 'myPrivateLinkService'
    ResourceGroupName = 'CreatePrivLinkService-rg'
    Location = 'eastus2'
    LoadBalancerFrontendIpConfiguration = $fe
    IpConfiguration = $ipconfig
}
New-AzPrivateLinkService @privlinksettings

```

El servicio de vínculo privado se crea y puede recibir tráfico. Si desea ver los flujos de tráfico, configure la aplicación detrás de la instancia local de Standard Load Balancer.

## <a name="create-private-endpoint"></a>Creación de un punto de conexión privado

En esta sección, asignará el servicio de vínculo privado a un punto de conexión privado. Una red virtual contiene el punto de conexión privado para el servicio de vínculo privado. Esta red virtual contiene los recursos que accederán al servicio de vínculo privado.

### <a name="create-private-endpoint-virtual-network"></a>Creación de una red virtual de punto de conexión privado

* Cree una red virtual con [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork).

```azurepowershell-interactive
## Create backend subnet config ##
$subnet = @{
    Name = 'mySubnetPE'
    AddressPrefix = '11.1.0.0/24'
    PrivateEndpointNetworkPolicies = 'Disabled'
}
$subnetConfig = New-AzVirtualNetworkSubnetConfig @subnet 

## Create the virtual network ##
$net = @{
    Name = 'myVNetPE'
    ResourceGroupName = 'CreatePrivLinkService-rg'
    Location = 'eastus2'
    AddressPrefix = '11.1.0.0/16'
    Subnet = $subnetConfig
}
$vnetpe = New-AzVirtualNetwork @net

```

### <a name="create-endpoint-and-connection"></a>Creación de un punto de conexión y una conexión

* Use [Get-AzPrivateLinkService](/powershell/module/az.network/get-azprivatelinkservice) para colocar la configuración del servicio de vínculo privado que creó antes en una variable para su uso posterior.

* Use [New-AzPrivateLinkServiceConnection](/powershell/module/az.network/new-azprivatelinkserviceconnection) para crear la configuración de conexión.

* Use [New-AzPrivateEndpoint](/powershell/module/az.network/new-azprivateendpoint) para crear el punto de conexión.



```azurepowershell-interactive
## Place the private link service configuration into variable. ##
$par1 = @{
    Name = 'myPrivateLinkService'
    ResourceGroupName = 'CreatePrivLinkService-rg'
}
$pls = Get-AzPrivateLinkService @par1

## Create the private link configuration and place in variable. ##
$par2 = @{
    Name = 'myPrivateLinkConnection'
    PrivateLinkServiceId = $pls.Id
}
$plsConnection = New-AzPrivateLinkServiceConnection @par2

## Place the virtual network into a variable. ##
$par3 = @{
    Name = 'myVNetPE'
    ResourceGroupName = 'CreatePrivLinkService-rg'
}
$vnetpe = Get-AzVirtualNetwork @par3

## Create private endpoint ##
$par4 = @{
    Name = 'MyPrivateEndpoint'
    ResourceGroupName = 'CreatePrivLinkService-rg'
    Location = 'eastus2'
    Subnet = $vnetpe.subnets[0]
    PrivateLinkServiceConnection = $plsConnection
}
New-AzPrivateEndpoint @par4 -ByManualRequest
```

### <a name="approve-the-private-endpoint-connection"></a>Aprobación de la conexión del punto de conexión privado

En esta sección, aprobará la conexión que creó en los pasos anteriores.

* Use [APPROVE-AzPrivateEndpointConnection](/powershell/module/az.network/approve-azprivateendpointconnection) para aprobar la conexión.

```azurepowershell-interactive
## Place the private link service configuration into variable. ##
$par1 = @{
    Name = 'myPrivateLinkService'
    ResourceGroupName = 'CreatePrivLinkService-rg'
}
$pls = Get-AzPrivateLinkService @par1

$par2 = @{
    Name = $pls.PrivateEndpointConnections[0].Name
    ServiceName = 'myPrivateLinkService'
    ResourceGroupName = 'CreatePrivLinkService-rg'
    Description = 'Approved'
}
Approve-AzPrivateEndpointConnection @par2

```

### <a name="ip-address-of-private-endpoint"></a>Dirección IP del punto de conexión privado

En esta sección, encontrará la dirección IP del punto de conexión privado que se corresponde con el equilibrador de carga y el servicio de vínculo privado.

* Use [Get-AzPrivateEndpoint](/powershell/module/az.network/get-azprivateendpoint) para recuperar la dirección IP.

```azurepowershell-interactive
## Get private endpoint and the IP address and place in a variable for display. ##
$par1 = @{
    Name = 'myPrivateEndpoint'
    ResourceGroupName = 'CreatePrivLinkService-rg'
    ExpandResource = 'networkinterfaces'
}
$pe = Get-AzPrivateEndpoint @par1

## Display the IP address by expanding the variable. ##
$pe.NetworkInterfaces[0].IpConfigurations[0].PrivateIpAddress
```

```bash
❯ $pe.NetworkInterfaces[0].IpConfigurations[0].PrivateIpAddress
11.1.0.4
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos, el equilibrador de carga y el resto de los recursos.

```azurepowershell-interactive
Remove-AzResourceGroup -Name 'CreatePrivLinkService-rg'
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido:

* Ha creado una red virtual y una instancia interna de Azure Load Balancer.
* Ha creado un servicio Private Link.

Para más información sobre el punto de conexión privado de Azure, continúe a
> [!div class="nextstepaction"]
> [Inicio rápido: Creación de un punto de conexión privado mediante Azure PowerShell](create-private-endpoint-powershell.md)

