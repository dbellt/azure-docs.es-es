---
title: Especificación de la información de un plan de compra de Marketplace mediante Azure PowerShell
description: Aprenda a especificar los detalles del plan de compra de Azure Marketplace al crear imágenes en una instancia de Shared Image Gallery.
author: cynthn
ms.service: virtual-machines
ms.subservice: shared-image-gallery
ms.topic: how-to
ms.workload: infrastructure
ms.date: 07/07/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 1ba23474034623228cc7c645ffb3d0ec5ae65474
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110679053"
---
# <a name="supply-azure-marketplace-purchase-plan-information-when-creating-images"></a>Indicación de la información del plan de compra de Azure Marketplace al crear imágenes

Si va a crear una imagen en una galería compartida y va a usar para ello un origen que se creó a partir de una imagen de Azure Marketplace, es posible que necesite realizar un seguimiento de la información del plan de compra. En este artículo se muestra cómo buscar información del plan de compra para una máquina virtual y, después, cómo usar esa información al crear una definición de imagen. También se tratará el uso de la información de la definición de la imagen para simplificar el suministro de información del plan de compra al crear una máquina virtual para una imagen.

Para más información sobre cómo buscar y usar imágenes de Marketplace, consulte [Búsqueda y uso de imágenes de Azure Marketplace](./windows/cli-ps-findimage.md).


## <a name="get-the-source-vm-information"></a>Obtención de información acerca de la máquina virtual de origen
Si aún tiene la máquina virtual original, puede obtener su información de nombre de plan, editor y producto mediante Get-AzVM. En este ejemplo se obtiene una máquina virtual llamada *myVM* en el grupo de recursos *myResourceGroup* y, después, se muestra la información del plan de compra de dicha máquina.

```azurepowershell-interactive
$vm = Get-azvm `
   -ResourceGroupName myResourceGroup `
   -Name myVM
$vm.Plan
```

## <a name="create-the-image-definition"></a>Creación de una definición de imagen

Obtenga la galería de imágenes que desee usar para almacenar la imagen. Si lo desea, primero puede enumerar todas las galerías.

```azurepowershell-interactive
Get-AzResource -ResourceType Microsoft.Compute/galleries | Format-Table
```

Después, cree las variables de la galería que desee usar. En este ejemplo, se va a crear una variable llamada `$gallery` para *myGallery* en el grupo de recursos *myGalleryRG*.

```azurepowershell-interactive
$gallery = Get-AzGallery `
   -Name myGallery `
   -ResourceGroupName myGalleryRG
```

Cree la definición de la imagen con los parámetros `-PurchasePlanPublisher`, `-PurchasePlanProduct` y `-PurchasePlanName`.

```azurepowershell-interactive
 $imageDefinition = New-AzGalleryImageDefinition `
   -GalleryName $gallery.Name `
   -ResourceGroupName $gallery.ResourceGroupName `
   -Location $gallery.Location `
   -Name 'myImageDefinition' `
   -OsState specialized `
   -OsType Linux `
   -Publisher 'myPublisher' `
   -Offer 'myOffer' `
   -Sku 'mySKU' `
   -PurchasePlanPublisher $vm.Plan.Publisher `
   -PurchasePlanProduct $vm.Plan.Product `
   -PurchasePlanName  $vm.Plan.Name
```

Luego, cree una versión de la imagen mediante [New-AzGalleryImageVersion](/powershell/module/az.compute/new-azgalleryimageversion). Se puede crear una versión de imagen desde una [máquina virtual](image-version-vm-powershell.md#create-an-image-version), una [imagen administrada](image-version-managed-image-powershell.md#create-an-image-version), un [disco duro virtual o instantánea](image-version-snapshot-powershell.md#create-an-image-version) u [otra versión de la imagen](image-version-another-gallery-powershell.md#create-the-image-version). 


## <a name="create-the-vm"></a>Creación de la máquina virtual

Cuando vaya a crear una máquina virtual a partir de la imagen, puede usar la información de la definición de la imagen para pasar la información del editor mediante [Set-AzVMPlan](/powershell/module/az.compute/set-azvmplan).


```azurepowershell-interactive
# Create some variables for the new VM.
$resourceGroup = "mySIGPubVM"
$location = "West Central US"
$vmName = "mySIGPubVM"

# Create a resource group
New-AzResourceGroup -Name $resourceGroup -Location $location

# Create the network resources.
$subnetConfig = New-AzVirtualNetworkSubnetConfig `
   -Name mySubnet `
   -AddressPrefix 192.168.1.0/24
$vnet = New-AzVirtualNetwork `
   -ResourceGroupName $resourceGroup `
   -Location $location `
   -Name MYvNET `
   -AddressPrefix 192.168.0.0/16 `
   -Subnet $subnetConfig
$pip = New-AzPublicIpAddress `
   -ResourceGroupName $resourceGroup `
   -Location $location `
  -Name "mypublicdns$(Get-Random)" `
  -AllocationMethod Static `
  -IdleTimeoutInMinutes 4
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig `
   -Name myNetworkSecurityGroupRuleRDP  `
   -Protocol Tcp `
   -Direction Inbound `
   -Priority 1000 `
   -SourceAddressPrefix * `
   -SourcePortRange * `
   -DestinationAddressPrefix * `
   -DestinationPortRange 3389 -Access Allow
$nsg = New-AzNetworkSecurityGroup `
   -ResourceGroupName $resourceGroup `
   -Location $location `
   -Name myNetworkSecurityGroup `
   -SecurityRules $nsgRuleRDP
$nic = New-AzNetworkInterface `
   -Name $vmName `
   -ResourceGroupName $resourceGroup `
   -Location $location `
  -SubnetId $vnet.Subnets[0].Id `
  -PublicIpAddressId $pip.Id `
  -NetworkSecurityGroupId $nsg.Id

# Create a virtual machine configuration using Set-AzVMSourceImage -Id $imageDefinition.Id to use the latest available image version. Set-AZVMPlan is used to pass the plan information in for the VM.

$vmConfig = New-AzVMConfig `
   -VMName $vmName `
   -VMSize Standard_D1_v2   | `
   Set-AzVMSourceImage -Id $imageDefinition.Id | `
   Set-AzVMPlan `
     -Publisher $imageDefinition.PurchasePlan.Publisher `
     -Product $imageDefinition.PurchasePlan.Product `
     -Name $imageDefinition.PurchasePlan.Name | `
   Add-AzVMNetworkInterface -Id $nic.Id

# Create the virtual machine
New-AzVM `
   -ResourceGroupName $resourceGroup `
   -Location $location `
   -VM $vmConfig
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo buscar y usar imágenes de Marketplace, consulte [Búsqueda y uso de imágenes de Azure Marketplace](./windows/cli-ps-findimage.md).
