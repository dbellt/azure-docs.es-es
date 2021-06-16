---
title: 'Tutorial: Creación de una puerta de enlace de NAT: PowerShell'
titlesuffix: Azure Virtual Network NAT
description: Comience a crear una puerta de enlace de NAT mediante Azure PowerShell.
author: asudbring
ms.author: allensu
ms.service: virtual-network
ms.subservice: nat
ms.topic: tutorial
ms.date: 03/09/2021
ms.custom: template-tutorial, devx-track-azurepowershell
ms.openlocfilehash: 8984a6d887afd092c82bc3544a84af32a8245dbc
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110671641"
---
# <a name="tutorial-create-a-nat-gateway-using-azure-powershell"></a>Tutorial: Creación de una puerta de enlace de NAT mediante Azure PowerShell

En este tutorial se muestra cómo usar el servicio Azure Virtual Network NAT. Creará una puerta de enlace de NAT para proporcionar conectividad saliente para una máquina virtual en Azure. 

En este tutorial aprenderá a:

> [!div class="checklist"]
> * Cree una red virtual.
> * Cree una máquina virtual.
> * Cree una puerta de enlace NAT y asóciela a la red virtual.
> * Conéctese a la máquina virtual y compruebe la dirección IP de NAT.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Azure PowerShell instalado localmente o Azure Cloud Shell

Si decide instalar y usar PowerShell de forma local, para realizar los pasos de este artículo necesita la versión 5.4.1 del módulo de Azure PowerShell o cualquier versión posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

En el ejemplo siguiente se crea un grupo de recursos llamado **myResourceGroupNAT** en la ubicación **eastus2**:

```azurepowershell-interactive
$rsg = @{
    Name = 'myResourceGroupNAT'
    Location = 'eastus2'
}
New-AzResourceGroup @rsg
```
## <a name="create-the-nat-gateway"></a>Creación de la puerta de enlace de NAT

En esta sección crearemos la puerta de enlace de NAT y los recursos de soporte.

* Para acceder a Internet, necesita una o varias direcciones IP públicas para la puerta de enlace de NAT. Use [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) para crear el recurso de dirección IP pública llamado **myPublicIP** en **myResourceGroupNAT**. 

* Cree una puerta de enlace de NAT de Azure global con [New-AzNatGateway](/powershell/module/az.network/new-aznatgateway). El resultado de este comando creará un recurso de puerta de enlace llamado **myNATgateway** que usa la dirección IP pública **myPublicIP**. El tiempo de espera de inactividad se establece en 10 minutos.  

* Cree una red virtual denominada **myVnet** con una subred llamada **mySubnet** con [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) en **myResourceGroup** usando [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). El espacio de direcciones IP de la red virtual es **10.1.0.0/16**. La subred de la red virtual es **10.1.0.0/24**.

* Cree un host de Azure Bastion denominado **myBastionHost** para acceder a la máquina virtual. Use [New-AzBastion](/powershell/module/az.network/new-azbastion) para crear un host bastión. Cree una dirección IP pública para el host bastión con [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress).

```azurepowershell-interactive
## Create public IP address for NAT gateway ##
$ip = @{
    Name = 'myPublicIP'
    ResourceGroupName = 'myResourceGroupNAT'
    Location = 'eastus2'
    Sku = 'Standard'
    AllocationMethod = 'Static'
}
$publicIP = New-AzPublicIpAddress @ip

## Create NAT gateway resource ##
$nat = @{
    ResourceGroupName = 'myResourceGroupNAT'
    Name = 'myNATgateway'
    IdleTimeoutInMinutes = '10'
    Sku = 'Standard'
    Location = 'eastus2'
    PublicIpAddress = $publicIP
}
$natGateway = New-AzNatGateway @nat

## Create subnet config and associate NAT gateway to subnet##
$subnet = @{
    Name = 'mySubnet'
    AddressPrefix = '10.1.0.0/24'
    NatGateway = $natGateway
}
$subnetConfig = New-AzVirtualNetworkSubnetConfig @subnet 

## Create Azure Bastion subnet. ##
$bastsubnet = @{
    Name = 'AzureBastionSubnet' 
    AddressPrefix = '10.1.1.0/24'
}
$bastsubnetConfig = New-AzVirtualNetworkSubnetConfig @bastsubnet

## Create the virtual network ##
$net = @{
    Name = 'myVNet'
    ResourceGroupName = 'myResourceGroupNAT'
    Location = 'eastus2'
    AddressPrefix = '10.1.0.0/16'
    Subnet = $subnetConfig,$bastsubnetConfig
}
$vnet = New-AzVirtualNetwork @net

## Create public IP address for bastion host. ##
$ip = @{
    Name = 'myBastionIP'
    ResourceGroupName = 'myResourceGroupNAT'
    Location = 'eastus2'
    Sku = 'Standard'
    AllocationMethod = 'Static'
}
$publicip = New-AzPublicIpAddress @ip

## Create bastion host ##
$bastion = @{
    ResourceGroupName = 'myResourceGroupNAT'
    Name = 'myBastion'
    PublicIpAddress = $publicip
    VirtualNetwork = $vnet
}
New-AzBastion @bastion -AsJob

```

## <a name="virtual-machine"></a>Máquina virtual

En esta sección, creará una máquina virtual para probar la puerta de enlace NAT y comprobar la dirección IP pública de la conexión saliente.

* Cree una interfaz de red con [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface).

* Establezca un nombre de usuario de administrador y una contraseña para la máquina virtual con [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential).

* Cree la máquina virtual con:
    * [New-AzVM](/powershell/module/az.compute/new-azvm)
    * [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig)
    * [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem)
    * [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage)
    * [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface)
    
```azurepowershell-interactive
# Set the administrator and password for the VMs. ##
$cred = Get-Credential

## Place the virtual network into a variable. ##
$vnet = Get-AzVirtualNetwork -Name 'myVNet' -ResourceGroupName 'myResourceGroupNAT'

## Create network interface for virtual machine. ##
$nic = @{
    Name = "myNicVM"
    ResourceGroupName = 'myResourceGroupNAT'
    Location = 'eastus2'
    Subnet = $vnet.Subnets[0]
}
$nicVM = New-AzNetworkInterface @nic

## Create a virtual machine configuration for VMs ##
$vmsz = @{
    VMName = "myVM"
    VMSize = 'Standard_DS1_v2'  
}
$vmos = @{
    ComputerName = "myVM"
    Credential = $cred
}
$vmimage = @{
    PublisherName = 'MicrosoftWindowsServer'
    Offer = 'WindowsServer'
    Skus = '2019-Datacenter'
    Version = 'latest'    
}
$vmConfig = New-AzVMConfig @vmsz `
    | Set-AzVMOperatingSystem @vmos -Windows `
    | Set-AzVMSourceImage @vmimage `
    | Add-AzVMNetworkInterface -Id $nicVM.Id

## Create the virtual machine for VMs ##
$vm = @{
    ResourceGroupName = 'myResourceGroupNAT'
    Location = 'eastus2'
    VM = $vmConfig
}
New-AzVM @vm

```

Espere a que se complete la creación de la máquina virtual antes de pasar a la siguiente sección.

## <a name="test-nat-gateway"></a>Prueba de la puerta de enlace NAT

En esta sección, probaremos la puerta de enlace NAT. En primer lugar, detectaremos la dirección IP pública de la puerta de enlace NAT. A continuación, nos conectaremos a la máquina virtual de prueba y comprobaremos la conexión saliente mediante la puerta de enlace NAT.
    
1. Inicie sesión en el [Portal de Azure](https://portal.azure.com)

1. Busque la dirección IP pública de la puerta de enlace NAT en la pantalla **Introducción**. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, **myPublicIP**.

2. Anote la dirección IP pública:

    :::image type="content" source="./media/tutorial-create-nat-gateway-portal/find-public-ip.png" alt-text="Detección de la dirección IP pública de la puerta de enlace NAT" border="true":::

3. Seleccione **Todos los servicios** en el menú de la izquierda, elija **Todos los recursos** y, en la lista de recursos, seleccione **myVM**, que se encuentra en el grupo de recursos **myResourceGroupNAT**.

4. En la página **Introducción**, seleccione **Conectar** y después **Instancia de Bastion**.

5. Seleccione el botón **Usar bastión** azul.

6. Escriba el nombre de usuario y la contraseña especificados durante la creación de la máquina virtual.

7. Abra **Internet Explorer** en **myTestVM**.

8. Escriba **https://whatsmyip.com** en la barra de direcciones.

9. Compruebe que la dirección IP mostrada coincida con la dirección de puerta de enlace de NAT anotada en el paso anterior:

    :::image type="content" source="./media/tutorial-create-nat-gateway-portal/my-ip.png" alt-text="Internet Explorer que muestra la dirección IP saliente externa" border="true":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, elimine la red virtual, la máquina virtual y la puerta de enlace NAT mediante los pasos siguientes:

```azurepowershell-interactive
Remove-AzResourceGroup -Name 'myResourceGroupNAT' -Force
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Virtual Network NAT, consulte:
> [!div class="nextstepaction"]
> [Información general de Virtual Network NAT](nat-overview.md)
