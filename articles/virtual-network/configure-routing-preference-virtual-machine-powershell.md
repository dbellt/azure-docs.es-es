---
title: Configuración de la preferencia de enrutamiento de una VM - Azure PowerShell
description: Obtenga información sobre cómo crear una VM con una dirección IP pública con preferencia de enrutamiento mediante Azure PowerShell.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/01/2021
ms.author: mnayak
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: ff06237b2baea2fffb0b590540ed4a9349841641
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110675929"
---
# <a name="configure-routing-preference-for-a-vm-using-azure-powershell"></a>Configuración de la preferencia de enrutamiento de una VM mediante Azure PowerShell

En este artículo se muestra cómo configurar las preferencias de enrutamiento para una máquina virtual. El tráfico de Internet enlazado desde la VM se enrutará a través de la red del ISP cuando elija **Internet** como opción de preferencia de enrutamiento. El enrutamiento predeterminado se hará a través de la red global de Microsoft.

En este artículo se muestra cómo crear una máquina virtual con una dirección IP pública que esté configurada para enrutar el tráfico a través de la red ISP mediante Azure PowerShell.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos
1. Si usa Cloud Shell, continúe al paso 2. Abra una sesión de comandos e inicie sesión en Azure con `Connect-AzAccount`.
2. Cree un grupo de recursos con el comando [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). En el siguiente ejemplo se crea un grupo de recursos en la región Este de EE. UU. de Azure:

    ```azurepowershell
    $rg = New-AzResourceGroup -Name MyResourceGroup -Location EastUS
    ```

## <a name="create-a-public-ip-address"></a>Crear una dirección IP pública

Para obtener acceso a las máquinas virtuales desde Internet, necesita tener direcciones IP públicas. Cree las direcciones IP públicas con [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress). En el ejemplo siguiente se crea una dirección IP pública IPv4 denominada *MyPublicIP* con una preferencia de enrutamiento de tipo *Internet* en el grupo de recursos *MyResourceGroup* en la región de *Este de EE. UU.* :

```azurepowershell-interactive
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

## <a name="create-network-resources"></a>Crear recursos de red

Antes de implementar una VM, debe crear recursos de red compatibles, como un grupo de seguridad de red, una red virtual y un NIC virtual.

### <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

Cree un grupo de seguridad de red con [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup). En el ejemplo siguiente se crea un NSG denominado *myNSG*.

```azurepowershell
$nsg = New-AzNetworkSecurityGroup `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location  `
-Name "myNSG"
```

### <a name="create-a-virtual-network"></a>Creación de una red virtual

Cree una red virtual con [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). En el ejemplo siguiente se crea una red virtual denominada *myVnet* con *mySubnet*:

### <a name="create-a-subnet"></a>Creación de una subred

```azurepowershell
$subnet = New-AzVirtualNetworkSubnetConfig `
-Name "mySubnet" `
-AddressPrefix "10.0.0.0/24"
```

```azurepowershell
# Create a virtual network
$vnet = New-AzVirtualNetwork `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location  `
-Name "myVNET" `
-AddressPrefix "10.0.0.0/16" `
-Subnet $subnet
```

### <a name="create-a-nic"></a>Creación de un NIC

Cree NIC virtuales con [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface. En el ejemplo siguiente se crea un NIC virtual.

```azurepowershell
# Create an IP Config
$ipconfig=New-AzNetworkInterfaceIpConfig `
-Name myIpConfig `
-Subnet $vnet.subnets[0] `
-PrivateIpAddressVersion IPv4 `
-PublicIpAddress  $publicIp

# Create a NIC
$nic = New-AzNetworkInterface `
-Name "mynic" `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location  `
-NetworkSecurityGroupId $nsg.Id `
-IpConfiguration $ipconfig 
```

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

Establezca un nombre de usuario de administrador y una contraseña para las máquinas virtuales con [Get-Credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential):

```azurepowershell
 $cred = get-credential -Message "Routing Preference SAMPLE:  Please enter the Administrator credential to log into the VM."
```

Ahora puede crear la máquina virtual con [New-AzVM](/powershell/module/az.compute/new-azvm). En el ejemplo siguiente, se crean dos máquinas virtuales y los componentes de red virtual necesarios, si aún no existen.

```azurepowershell
 $vmsize = "Standard_A2"
 $ImagePublisher = "MicrosoftWindowsServer"
 $imageOffer = "WindowsServer"
 $imageSKU = "2019-Datacenter"

 $vmName= "myVM"
 $vmconfig = New-AzVMConfig -VMName $vmName -VMSize $vmsize | Set-AzVMOperatingSystem -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate | Set-AzVMSourceImage -PublisherName $ImagePublisher -Offer $imageOffer -Skus $imageSKU -Version "latest" | Set-AzVMOSDisk -Name "$vmName.vhd" -CreateOption "FromImage" | Add-AzVMNetworkInterface -Id $nic.Id 
 $VM1 = New-AzVM -ResourceGroupName $rg.ResourceGroupName  -Location $rg.Location  -VM $vmconfig
```

## <a name="allow-network-traffic-to-the-vm"></a>Permitir tráfico de red en la VM

Para poder conectarse a la dirección IP pública desde Internet, asegúrese de que tiene los puertos necesarios abiertos en cualquier grupo de seguridad de red que haya asociado a la interfaz de red o la subred en la que se encuentra la interfaz de red. Puede ver las reglas de seguridad vigentes para una interfaz de red y su subred mediante [Portal](diagnose-network-traffic-filter-problem.md#diagnose-using-azure-portal), la [CLI](diagnose-network-traffic-filter-problem.md#diagnose-using-azure-cli) o [PowerShell](diagnose-network-traffic-filter-problem.md#diagnose-using-powershell).

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados.

 ```azurepowershell
 Remove-AzResourceGroup -Name MyResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre la [preferencia de enrutamiento en direcciones IP públicas](routing-preference-overview.md).
* Obtenga más información acerca de las [direcciones IP públicas](./public-ip-addresses.md#public-ip-addresses) en Azure.
* Obtenga más información sobre la [configuración de las direcciones IP públicas](virtual-network-public-ip-address.md#create-a-public-ip-address).
