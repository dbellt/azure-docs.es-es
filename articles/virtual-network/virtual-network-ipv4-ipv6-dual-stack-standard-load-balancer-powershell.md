---
title: 'Implementación de una aplicación de pila doble IPv6: Standard Load Balancer: PowerShell'
titlesuffix: Azure Virtual Network
description: En este artículo, se muestra cómo implementar una aplicación de pila doble IPv6 en Azure Virtual Network con Standard Load Balancer.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/01/2020
ms.author: kumud
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: aba9c503efc99c461009dbced90903674ecd50bb
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110677034"
---
# <a name="deploy-an-ipv6-dual-stack-application-in-azure---powershell"></a>Implementación de una aplicación de pila doble IPv6 en Azure - PowerShell

En este artículo se explica cómo se implementa mediante Standard Load Balancer una aplicación de doble pila (IPv4 + IPv6) en Azure que contiene una red virtual de doble pila y una subred, un equilibrador de carga estándar con configuraciones de front-end duales (IPv4 + IPv6), máquinas virtuales con NIC que tienen una configuración de IP dual, un grupo de seguridad de red y direcciones IP públicas.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de forma local, la versión del módulo de Azure PowerShell que necesita este artículo es la 6.9.0 u otra posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para poder generar la red virtual de doble pila, debe crear primero un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). En el ejemplo siguiente, se crea un grupo de recursos llamado *myRGDualStack* en la ubicación *Este de EE. UU.* :

```azurepowershell-interactive
   $rg = New-AzResourceGroup `
  -ResourceGroupName "dsRG1"  `
  -Location "east us"
```

## <a name="create-ipv4-and-ipv6-public-ip-addresses"></a>Creación de direcciones IP públicas IPv4 e IPv6
Para obtener acceso a las máquinas virtuales desde Internet, necesita direcciones IP públicas IPv4 e IPv6 para el equilibrador de carga. Cree las direcciones IP públicas con [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress). En el ejemplo siguiente, se crean unas direcciones IP públicas IPv4 e IPv6 denominadas *dsPublicIP_v4* y *dsPublicIP_v6* en el grupo de recursos *dsRG1*:

```azurepowershell-interactive
$PublicIP_v4 = New-AzPublicIpAddress `
  -Name "dsPublicIP_v4" `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -AllocationMethod Static `
  -IpAddressVersion IPv4 `
  -Sku Standard
  
$PublicIP_v6 = New-AzPublicIpAddress `
  -Name "dsPublicIP_v6" `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -AllocationMethod Static `
  -IpAddressVersion IPv6 `
  -Sku Standard
```
Para obtener acceso a las máquinas virtuales mediante una conexión RDP, cree una dirección IP pública de IPV4 para las máquinas virtuales con [New AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress).

```azurepowershell-interactive
  $RdpPublicIP_1 = New-AzPublicIpAddress `
  -Name "RdpPublicIP_1" `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -AllocationMethod Static `
  -Sku Standard `
  -IpAddressVersion IPv4
  
  $RdpPublicIP_2 = New-AzPublicIpAddress `
   -Name "RdpPublicIP_2" `
   -ResourceGroupName $rg.ResourceGroupName `
   -Location $rg.Location  `
   -AllocationMethod Static `
   -Sku Standard `
   -IpAddressVersion IPv4
```

## <a name="create-standard-load-balancer"></a>Creación de Standard Load Balancer

En esta sección, configurará la dirección IP de front-end doble (IPv4 e IPv6) y el grupo de direcciones de back-end del equilibrador de carga, además de crear una instancia de Standard Load Balancer.

### <a name="create-front-end-ip"></a>Creación de una dirección IP de front-end

Cree una dirección IP de front-end con [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig). En el ejemplo siguiente, se crean unas configuraciones de IP de front-end IPv4 e IPv6 llamadas *dsLbFrontEnd_v4* y *dsLbFrontEnd_v6*:

```azurepowershell-interactive
$frontendIPv4 = New-AzLoadBalancerFrontendIpConfig `
  -Name "dsLbFrontEnd_v4" `
  -PublicIpAddress $PublicIP_v4

$frontendIPv6 = New-AzLoadBalancerFrontendIpConfig `
  -Name "dsLbFrontEnd_v6" `
  -PublicIpAddress $PublicIP_v6

```

### <a name="configure-back-end-address-pool"></a>Configuración del grupo de direcciones de back-end

Cree un grupo de direcciones de back-end con [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig). En el resto de los pasos, las máquinas virtuales se conectan a este grupo back-end. En el ejemplo siguiente, se crean los grupos de direcciones de back-end *dsLbBackEndPool_v4* y *dsLbBackEndPool_v6*, que van a incluir máquinas virtuales con configuraciones de NIC IPV4 e IPv6:

```azurepowershell-interactive
$backendPoolv4 = New-AzLoadBalancerBackendAddressPoolConfig `
-Name "dsLbBackEndPool_v4"

$backendPoolv6 = New-AzLoadBalancerBackendAddressPoolConfig `
-Name "dsLbBackEndPool_v6"
```
### <a name="create-a-health-probe"></a>Creación de un sondeo de estado
Utilice [Add-AzLoadBalancerProbeConfig](/powershell/module/az.network/add-azloadbalancerprobeconfig) para crear un sondeo de estado para supervisar el estado de las VM.
```azurepowershell
$probe = New-AzLoadBalancerProbeConfig -Name MyProbe -Protocol tcp -Port 3389 -IntervalInSeconds 15 -ProbeCount 2
```
### <a name="create-a-load-balancer-rule"></a>Creación de una regla de equilibrador de carga

Las reglas de equilibrador de carga se utilizan para definir cómo se distribuye el tráfico a las máquinas virtuales. Defina la configuración de la IP de front-end para el tráfico entrante y el grupo de IP de back-end para el tráfico entrante, junto con los puertos de origen y destino requeridos. Para tener la seguridad de que solo reciben tráfico las máquinas virtuales correctas, también puede definir un sondeo de estado. Los equilibradores de carga básicos usan un sondeo de IPv4 para evaluar el estado de los puntos de conexión IPv4 e IPv6 de las máquinas virtuales. Los equilibradores de carga estándar permiten realizar explícitamente sondeos de estado de IPv6.

Cree una regla del equilibrador de carga con [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/add-azloadbalancerruleconfig). En el ejemplo siguiente, se crean reglas del equilibrador de carga llamadas *dsLBrule_v4* y *dsLBrule_v6*, y se equilibra el tráfico del puerto *TCP* *80* dirigido a las configuraciones de IP de front-end IPv4 e IPv6:

```azurepowershell-interactive
$lbrule_v4 = New-AzLoadBalancerRuleConfig `
  -Name "dsLBrule_v4" `
  -FrontendIpConfiguration $frontendIPv4 `
  -BackendAddressPool $backendPoolv4 `
  -Protocol Tcp `
  -FrontendPort 80 `
  -BackendPort 80 `
   -probe $probe

$lbrule_v6 = New-AzLoadBalancerRuleConfig `
  -Name "dsLBrule_v6" `
  -FrontendIpConfiguration $frontendIPv6 `
  -BackendAddressPool $backendPoolv6 `
  -Protocol Tcp `
  -FrontendPort 80 `
  -BackendPort 80 `
   -probe $probe
```

### <a name="create-load-balancer"></a>Creación de un equilibrador de carga

Cree una instancia de Standard Load Balancer con [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer). En el ejemplo siguiente, se crea una instancia pública de Standard Load Balancer llamada *myLoadBalancer* mediante las configuraciones de direcciones IP de front-end IPv4 e IPv6, grupos de servidores back-end y las reglas de equilibrio de carga que creó en los pasos anteriores:

```azurepowershell-interactive
$lb = New-AzLoadBalancer `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location  `
-Name "MyLoadBalancer" `
-Sku "Standard" `
-FrontendIpConfiguration $frontendIPv4,$frontendIPv6 `
-BackendAddressPool $backendPoolv4,$backendPoolv6 `
-LoadBalancingRule $lbrule_v4,$lbrule_v6 `
-Probe $probe
```

## <a name="create-network-resources"></a>Crear recursos de red
Para poder implementar algunas máquinas virtuales y probar el equilibrador, debe crear unos recursos de red que lo permitan: un conjunto de disponibilidad, un grupo de seguridad de red, una red virtual y varias NIC virtuales. 
### <a name="create-an-availability-set"></a>Crear un conjunto de disponibilidad
Para mejorar la alta disponibilidad de la aplicación, coloque las máquinas virtuales en un conjunto de disponibilidad.

Cree un conjunto de disponibilidad con [New-AzAvailabilitySet](/powershell/module/az.compute/new-azavailabilityset). En el ejemplo siguiente se crea un conjunto de disponibilidad denominado *myAvailabilitySet*:

```azurepowershell-interactive
$avset = New-AzAvailabilitySet `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -Name "dsAVset" `
  -PlatformFaultDomainCount 2 `
  -PlatformUpdateDomainCount 2 `
  -Sku aligned
```

### <a name="create-network-security-group"></a>Creación de un grupo de seguridad de red

Cree un grupo de seguridad de red para las reglas que van a controlar la comunicación entrante y saliente de la red virtual.

#### <a name="create-a-network-security-group-rule-for-port-3389"></a>Creación de una regla de grupo de seguridad de red para el puerto 3389

Cree una regla de grupo de seguridad de red para permitir conexiones RDP en el puerto 3389 con [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig).

```azurepowershell-interactive
$rule1 = New-AzNetworkSecurityRuleConfig `
-Name 'myNetworkSecurityGroupRuleRDP' `
-Description 'Allow RDP' `
-Access Allow `
-Protocol Tcp `
-Direction Inbound `
-Priority 100 `
-SourceAddressPrefix * `
-SourcePortRange * `
-DestinationAddressPrefix * `
-DestinationPortRange 3389
```
#### <a name="create-a-network-security-group-rule-for-port-80"></a>Creación de una regla de grupo de seguridad de red para el puerto 80

Cree una regla de grupo de seguridad de red para permitir conexiones de Internet a través del puerto 80 con [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig).

```azurepowershell-interactive
$rule2 = New-AzNetworkSecurityRuleConfig `
  -Name 'myNetworkSecurityGroupRuleHTTP' `
  -Description 'Allow HTTP' `
  -Access Allow `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 200 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80
```
#### <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

Cree un grupo de seguridad de red con [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup).

```azurepowershell-interactive
$nsg = New-AzNetworkSecurityGroup `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location  `
-Name "dsNSG1"  `
-SecurityRules $rule1,$rule2
```
### <a name="create-a-virtual-network"></a>Creación de una red virtual

Cree una red virtual con [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). En el ejemplo siguiente se crea una red virtual denominada con una subred llamada *dsVnet* con *mySubnet*:

```azurepowershell-interactive
# Create dual stack subnet
$subnet = New-AzVirtualNetworkSubnetConfig `
-Name "dsSubnet" `
-AddressPrefix "10.0.0.0/24","fd00:db8:deca:deed::/64"

# Create the virtual network
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -Name "dsVnet" `
  -AddressPrefix "10.0.0.0/16","fd00:db8:deca::/48"  `
  -Subnet $subnet
```

### <a name="create-nics"></a>Creación de tarjetas NIC

Cree NIC virtuales con [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface). En el ejemplo siguiente, se crean dos NIC virtuales, ambas con las configuraciones IPv4 e IPv6. (Una NIC virtual para cada máquina virtual que cree para la aplicación en los pasos siguientes).

```azurepowershell-interactive
  $Ip4Config=New-AzNetworkInterfaceIpConfig `
    -Name dsIp4Config `
    -Subnet $vnet.subnets[0] `
    -PrivateIpAddressVersion IPv4 `
    -LoadBalancerBackendAddressPool $backendPoolv4 `
    -PublicIpAddress  $RdpPublicIP_1
      
  $Ip6Config=New-AzNetworkInterfaceIpConfig `
    -Name dsIp6Config `
    -Subnet $vnet.subnets[0] `
    -PrivateIpAddressVersion IPv6 `
    -LoadBalancerBackendAddressPool $backendPoolv6
    
  $NIC_1 = New-AzNetworkInterface `
    -Name "dsNIC1" `
    -ResourceGroupName $rg.ResourceGroupName `
    -Location $rg.Location  `
    -NetworkSecurityGroupId $nsg.Id `
    -IpConfiguration $Ip4Config,$Ip6Config 
    
  $Ip4Config=New-AzNetworkInterfaceIpConfig `
    -Name dsIp4Config `
    -Subnet $vnet.subnets[0] `
    -PrivateIpAddressVersion IPv4 `
    -LoadBalancerBackendAddressPool $backendPoolv4 `
    -PublicIpAddress  $RdpPublicIP_2  

  $NIC_2 = New-AzNetworkInterface `
    -Name "dsNIC2" `
    -ResourceGroupName $rg.ResourceGroupName `
    -Location $rg.Location  `
    -NetworkSecurityGroupId $nsg.Id `
    -IpConfiguration $Ip4Config,$Ip6Config 

```

### <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Establezca un nombre de usuario de administrador y una contraseña para las máquinas virtuales con [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential):

```azurepowershell-interactive
$cred = get-credential -Message "DUAL STACK VNET SAMPLE:  Please enter the Administrator credential to log into the VMs."
```

Ahora puede crear las máquinas virtuales con [New-AzVM](/powershell/module/az.compute/new-azvm). En el ejemplo siguiente, se crean dos máquinas virtuales y los componentes de red virtual necesarios, si aún no existen. 

```azurepowershell-interactive
$vmsize = "Standard_A2"
$ImagePublisher = "MicrosoftWindowsServer"
$imageOffer = "WindowsServer"
$imageSKU = "2019-Datacenter"

$vmName= "dsVM1"
$VMconfig1 = New-AzVMConfig -VMName $vmName -VMSize $vmsize -AvailabilitySetId $avset.Id 3> $null | Set-AzVMOperatingSystem -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent 3> $null | Set-AzVMSourceImage -PublisherName $ImagePublisher -Offer $imageOffer -Skus $imageSKU -Version "latest" 3> $null | Set-AzVMOSDisk -Name "$vmName.vhd" -CreateOption fromImage  3> $null | Add-AzVMNetworkInterface -Id $NIC_1.Id  3> $null 
$VM1 = New-AzVM -ResourceGroupName $rg.ResourceGroupName  -Location $rg.Location  -VM $VMconfig1 

$vmName= "dsVM2"
$VMconfig2 = New-AzVMConfig -VMName $vmName -VMSize $vmsize -AvailabilitySetId $avset.Id 3> $null | Set-AzVMOperatingSystem -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent 3> $null | Set-AzVMSourceImage -PublisherName $ImagePublisher -Offer $imageOffer -Skus $imageSKU -Version "latest" 3> $null | Set-AzVMOSDisk -Name "$vmName.vhd" -CreateOption fromImage  3> $null | Add-AzVMNetworkInterface -Id $NIC_2.Id  3> $null 
$VM2 = New-AzVM -ResourceGroupName $rg.ResourceGroupName  -Location $rg.Location  -VM $VMconfig2
```

## <a name="determine-ip-addresses-of-the-ipv4-and-ipv6-endpoints"></a>Determinación de las direcciones IP de los puntos de conexión IPv4 e IPv6
Obtenga todos los objetos de interfaz de red del grupo de recursos para resumir las direcciones IP que se utilizan en esta implementación con `get-AzNetworkInterface`. Obtenga también las direcciones del front-end del equilibrador de carga de los puntos de conexión IPv4 e IPv6 con `get-AzpublicIpAddress`.

```azurepowershell-interactive
$rgName= "dsRG1"
$NICsInRG= get-AzNetworkInterface -resourceGroupName $rgName 
write-host `nSummary of IPs in this Deployment: 
write-host ******************************************
foreach ($NIC in $NICsInRG) {
 
    $VMid= $NIC.virtualmachine.id 
    $VMnamebits= $VMid.split("/") 
    $VMname= $VMnamebits[($VMnamebits.count-1)] 
    write-host `nPrivate IP addresses for $VMname 
    $IPconfigsInNIC= $NIC.IPconfigurations 
    foreach ($IPconfig in $IPconfigsInNIC) {
 
        $IPaddress= $IPconfig.privateipaddress 
        write-host "    "$IPaddress 
        IF ($IPconfig.PublicIpAddress.ID) {
 
            $IDbits= ($IPconfig.PublicIpAddress.ID).split("/")
            $PipName= $IDbits[($IDbits.count-1)]
            $PipObject= get-azPublicIpAddress -name $PipName -resourceGroup $rgName
            write-host "    "RDP address:  $PipObject.IpAddress
                 }
         }
 }
 
 
 
  write-host `nPublic IP addresses on Load Balancer:
 
  (get-AzpublicIpAddress -resourcegroupname $rgName | where { $_.name -notlike "RdpPublicIP*" }).IpAddress
```
En la ilustración siguiente, se muestra una salida de ejemplo que muestra las direcciones IPv4 e IPv6 privadas de las dos máquinas virtuales y las direcciones IP IPv4 e IPv6 de front-end del equilibrador de carga.

![Resumen IP de la implementación de aplicaciones de doble pila (IPv4/IPv6) en Azure](./media/virtual-network-ipv4-ipv6-dual-stack-powershell/dual-stack-application-summary.png)

## <a name="view-ipv6-dual-stack-virtual-network-in-azure-portal"></a>Visualización de la red virtual de doble pila IPv6 en Azure Portal
Para ver la red virtual de doble pila IPv6 en Azure Portal, siga estos pasos:
1. En la barra de búsqueda del portal, escriba *dsVnet*.
2. Seleccione **dsVnet** cuando aparezca en los resultados de búsqueda. De este modo, se abrirá la **página de información general** de la red virtual de doble pila llamada *dsVnet*. En la red virtual de doble pila, se muestran las dos NIC con las configuraciones IPv4 e IPv6 en una subred de pila doble denominada *dsSubnet*.

  ![Red virtual de doble pila IPv6 de Azure](./media/virtual-network-ipv4-ipv6-dual-stack-powershell/dual-stack-vnet.png)


## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados.

```azurepowershell-interactive
Remove-AzResourceGroup -Name dsRG1
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha creado una instancia de Standard Load Balancer con una configuración IP de front-end doble (IPv4 e IPv6). También ha creado dos máquinas virtuales que contienen NIC con configuraciones de IP duales (IPV4 + IPv6) que se agregaron al grupo de back-end del equilibrador de carga. Para más información sobre la compatibilidad de IPv6 en las redes virtuales de Azure, consulte [¿Qué es IPv6 para Azure Virtual Network?](ipv6-overview.md)
