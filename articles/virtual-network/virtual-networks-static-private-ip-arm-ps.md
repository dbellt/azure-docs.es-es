---
title: 'Creación de una VM con una dirección IP privada estática: Azure PowerShell'
description: Obtenga información sobre la creación de una máquina virtual con una dirección IP privada mediante PowerShell.
services: virtual-network
documentationcenter: na
author: KumudD
editor: ''
tags: azure-resource-manager
ms.assetid: d5f18929-15e3-40a2-9ee3-8188bc248ed8
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/07/2019
ms.author: kumud
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 71d8c9b30923f6b3fd79a42542cac43cbfa09964
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110678636"
---
# <a name="create-a-virtual-machine-with-a-static-private-ip-address-using-powershell"></a>Creación de una máquina virtual con una dirección IP privada estática mediante PowerShell

Puede crear una máquina virtual (VM) con una dirección IP privada estática. Asigne una dirección IP privada estática en lugar de una dirección dinámica si quiere seleccionar la dirección de una subred que se asignará a una VM. Obtenga más información sobre las [direcciones IP privadas estáticas](./public-ip-addresses.md#allocation-method). Para cambiar una dirección IP privada asignada a una VM existente de dinámica a estática, o para trabajar con direcciones IP públicas, consulte [Incorporación, cambio o eliminación de direcciones IP](virtual-network-network-interface-addresses.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

Puede realizar los pasos siguientes desde el equipo local o mediante Azure Cloud Shell. Para usar el equipo local, asegúrese de que tiene [instalado Azure PowerShell](/powershell/azure/install-az-ps?toc=%2fazure%2fvirtual-network%2ftoc.json). Para usar Azure Cloud Shell, seleccione **Probar** en la esquina superior derecha de cualquier cuadro de comando que sigue. Cloud Shell inicia su sesión en Azure.

1. Si usa Cloud Shell, continúe al paso 2. Abra una sesión de comandos e inicie sesión en Azure con `Connect-AzAccount`.
2. Cree un grupo de recursos con el comando [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). En el siguiente ejemplo se crea un grupo de recursos en la región Este de EE. UU. de Azure:

   ```azurepowershell-interactive
   $RgName = "myResourceGroup"
   $Location = "eastus"
   New-AzResourceGroup -Name $RgName -Location $Location
   ```

3. Cree una configuración de subred y una red virtual con los comandos [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) y [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork):

   ```azurepowershell-interactive
   # Create a subnet configuration
   $SubnetConfig = New-AzVirtualNetworkSubnetConfig `
   -Name MySubnet `
   -AddressPrefix 10.0.0.0/24

   # Create a virtual network
   $VNet = New-AzVirtualNetwork `
   -ResourceGroupName $RgName `
   -Location $Location `
   -Name MyVNet `
   -AddressPrefix 10.0.0.0/16 `
   -Subnet $subnetConfig

   # Get the subnet object for use in a later step.
   $Subnet = Get-AzVirtualNetworkSubnetConfig -Name $SubnetConfig.Name -VirtualNetwork $VNet
   ```

4. Cree una interfaz de red en la red virtual y asigne una dirección IP privada de la subred a la interfaz de red con los comandos [New-AzNetworkInterfaceIpConfig](/powershell/module/Az.Network/New-AzNetworkInterfaceIpConfig) y [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface):

   ```azurepowershell-interactive
   $IpConfigName1 = "IPConfig-1"
   $IpConfig1     = New-AzNetworkInterfaceIpConfig `
     -Name $IpConfigName1 `
     -Subnet $Subnet `
     -PrivateIpAddress 10.0.0.4 `
     -Primary

   $NIC = New-AzNetworkInterface `
     -Name MyNIC `
     -ResourceGroupName $RgName `
     -Location $Location `
     -IpConfiguration $IpConfig1
   ```

5. Cree una configuración de VM con [New-AzVMConfig](/powershell/module/Az.Compute/New-AzVMConfig) y, a continuación, cree la VM con [New-AzVM](/powershell/module/az.Compute/New-azVM). Cuando se le solicite, proporcione un nombre de usuario y una contraseña que se usarán como credenciales de inicio de sesión para la VM:

   ```azurepowershell-interactive
   $VirtualMachine = New-AzVMConfig -VMName MyVM -VMSize "Standard_DS3"
   $VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName MyServerVM -ProvisionVMAgent -EnableAutoUpdate
   $VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
   $VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName 'MicrosoftWindowsServer' -Offer 'WindowsServer' -Skus '2012-R2-Datacenter' -Version latest
   New-AzVM -ResourceGroupName $RgName -Location $Location -VM $VirtualMachine -Verbose
   ```

> [!WARNING]
> Aunque puede agregar la configuración de dirección IP privada al sistema operativo, se recomienda no hacerlo hasta después de haber leído el tema sobre la [incorporación de una dirección IP privada a un sistema operativo](virtual-network-network-interface-addresses.md#private).
> 
> 
> <a name = "change-the-allocation-method-for-a-private-ip-address-assigned-to-a-network-interface"></a>
> 
> [!IMPORTANT]
> Para acceder a la VM desde Internet, debe asignar una dirección IP pública a la VM. También puede cambiar una asignación de dirección IP privada dinámica por una asignación estática. Para obtener más información, consulte [Incorporación o eliminación de direcciones IP](virtual-network-network-interface-addresses.md). Además, se recomienda limitar el tráfico de red a la VM mediante la asociación de un grupo de seguridad de red a la interfaz de red, a la subred que creó en la interfaz de red o a ambas. Para más información, consulte el tema sobre la [administración de grupos de seguridad de red](manage-network-security-group.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, puede usar [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos y todos los recursos que contiene:

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Pasos siguientes

- Obtener más información acerca de las [direcciones IP privadas](./private-ip-addresses.md) y la asignación de una [dirección IP privada estática](virtual-network-network-interface-addresses.md#add-ip-addresses) a una máquina virtual de Azure.
- Obtener más información acerca de cómo crear máquinas virtuales [Linux](../virtual-machines/windows/tutorial-manage-vm.md?toc=%2fazure%2fvirtual-network%2ftoc.json) y [Windows](../virtual-machines/windows/tutorial-manage-vm.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
