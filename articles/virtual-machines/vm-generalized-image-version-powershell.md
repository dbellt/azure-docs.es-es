---
title: Creación de una imagen a partir de una máquina virtual generalizada
description: Cree una VM con una imagen generalizada en Shared Image Gallery.
author: cynthn
ms.service: virtual-machines
ms.subservice: shared-image-gallery
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: c01fa1aed8265f759e520d47c0d0cfe6ad182861
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112032634"
---
# <a name="create-a-vm-using-a-generalized-image"></a>Creación de una VM con una imagen generalizada 

Cree una VM con una imagen generalizada almacenada en Shared Image Gallery. Si quiere crear una VM mediante una imagen especializada, consulte [Creación de una VM a partir de una imagen especializada](vm-specialized-image-version-powershell.md).

Cuando tenga una versión de una imagen generalizada, puede crear una o varias VM. Mediante el uso del cmdlet [New-AzVM](/powershell/module/az.compute/new-azvm). 

En este ejemplo, se usa el identificador de definición de imagen para asegurarse de que la nueva VM usará la versión más reciente de una imagen. También puede usar una versión específica con el identificador de la versión de imagen de `Set-AzVMSourceImage -Id`. Por ejemplo, para usar la versión de imagen *1.0.0*, escriba: `Set-AzVMSourceImage -Id "/subscriptions/<subscription ID where the gallery is located>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition/versions/1.0.0"`. 

Tenga en cuenta que el uso de una versión de imagen específica significa que la automatización podría producir un error si dicha versión específica no está disponible porque se eliminó o se quitó de la región. Se recomienda usar el id. de definición de la imagen para crear la nueva VM, a menos que se requiera una versión de imagen específica.

Reemplace los nombres de recursos según sea necesario en estos ejemplos. 

## <a name="simplified-parameter-set"></a>Conjunto de parámetros simplificado

Puede usar el conjunto de parámetros simplificado para crear rápidamente una VM a partir de una imagen. El conjunto de parámetros simplificado usa el nombre de la VM para crear automáticamente algunos de los recursos necesarios, como la red virtual y la dirección IP pública. 

```azurepowershell-interactive
# Create some variables for the new VM 
$resourceGroup = "myResourceGroup"
$location = "South Central US"
$vmName = "myVMfromImage"

# Get the image. Replace the name of your resource group, gallery, and image definition. This will create the VM from the latest image version available.

$imageDefinition = Get-AzGalleryImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myResourceGroup `
   -Name myImageDefinition

# Create user object
$cred = Get-Credential `
   -Message "Enter a username and password for the virtual machine."

# Create a resource group
New-AzResourceGroup `
   -Name $resourceGroup `
   -Location $location

New-AzVM `
   -ResourceGroupName $resourceGroup `
   -Location $location `
   -Name $vmName `
   -Image $imageDefinition.Id
   -Credential $cred
```



## <a name="full-parameter-set"></a>Conjunto de parámetros completo

Puede crear una VM mediante recursos específicos con el conjunto de parámetros completo.

```azurepowershell-interactive
# Create some variables for the new VM 
$resourceGroup = "myResourceGroup"
$location = "South Central US"
$vmName = "myVMfromImage"

# Get the image. Replace the name of your resource group, gallery, and image definition. This will create the VM from the latest image version available.

$imageDefinition = Get-AzGalleryImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myResourceGroup `
   -Name myImageDefinition

# Create user object
$cred = Get-Credential `
   -Message "Enter a username and password for the virtual machine."

# Create a resource group
New-AzResourceGroup `
   -Name $resourceGroup `
   -Location $location

# Network pieces
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
   -DestinationPortRange 3389 `
   -Access Allow
$nsg = New-AzNetworkSecurityGroup `
   -ResourceGroupName $resourceGroup `
   -Location $location `
  -Name myNetworkSecurityGroup `
  -SecurityRules $nsgRuleRDP
$nic = New-AzNetworkInterface `
   -Name myNic `
   -ResourceGroupName $resourceGroup `
   -Location $location `
  -SubnetId $vnet.Subnets[0].Id `
  -PublicIpAddressId $pip.Id `
  -NetworkSecurityGroupId $nsg.Id

# Create a virtual machine configuration using $imageDefinition.Id to use the latest image version.
$vmConfig = New-AzVMConfig `
   -VMName $vmName `
   -VMSize Standard_D1_v2 | `
   Set-AzVMOperatingSystem -Windows -ComputerName $vmName -Credential $cred | `
   Set-AzVMSourceImage -Id $imageDefinition.Id | `
   Add-AzVMNetworkInterface -Id $nic.Id

# Create a virtual machine
New-AzVM `
   -ResourceGroupName $resourceGroup `
   -Location $location `
   -VM $vmConfig
```


## <a name="next-steps"></a>Pasos siguientes
[Azure Image Builder (versión preliminar)](./image-builder-overview.md) puede ayudar a automatizar la creación de versiones de la imagen, incluso se puede usar para actualizar y [crear una nueva versión de la imagen a partir de una versión de imagen existente](./windows/image-builder-gallery-update-image-version.md). 

Puede crear también recursos de galería de imágenes compartidas con plantillas. Hay varias plantillas de Inicio rápido de Azure disponibles: 

- [Creación de una galería de imágenes compartidas](https://azure.microsoft.com/resources/templates/sig-create/)
- [Creación de una definición de imagen en una galería de imágenes compartidas](https://azure.microsoft.com/resources/templates/sig-image-definition-create/)
- [Creación de una versión de imagen en una galería de imágenes compartidas](https://azure.microsoft.com/resources/templates/sig-image-version-create/)
- [Creación de una máquina virtual a partir de la versión de la imagen](https://azure.microsoft.com/resources/templates/vm-from-sig/)

Para más información sobre las galerías de imágenes compartidas, consulte la [Introducción](./shared-image-galleries.md). Si encuentra problemas, consulte [Solución de problemas de galerías de imágenes compartidas](troubleshooting-shared-images.md).
