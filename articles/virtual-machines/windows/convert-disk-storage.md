---
title: Conversión de almacenamiento de discos administrados entre diferentes tipos de discos mediante Azure PowerShell
description: Cómo convertir discos administrados de Azure entre los diferentes tipos de discos mediante Azure PowerShell.
author: roygara
ms.service: virtual-machines
ms.subservice: disks
ms.topic: how-to
ms.date: 02/13/2021
ms.author: albecker
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: d2b0ac2750515c8a61674b695e0faf3fd54ff7c6
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110670260"
---
# <a name="update-the-storage-type-of-a-managed-disk"></a>Actualizar el tipo de almacenamiento de un disco administrado

Hay cuatro tipos de discos de Azure Managed Disks: Azure Ultra Disks, SSD Premium, SSD estándar y HDD estándar. Puede cambiar entre SSD Premium, SSD estándar y HDD estándar según las necesidades de rendimiento. Todavía no puede cambiar de o a un disco Ultra; debe implementar uno nuevo.

Esta funcionalidad no se admite para discos no administrados. Sin embargo, puede [convertir un disco no administrado en un disco administrado](convert-unmanaged-to-managed-disks.md) de manera sencilla para poder cambiar entre los tipos de disco.



## <a name="before-you-begin"></a>Antes de empezar

* Debido a que la conversión requiere reiniciar la máquina virtual, debería programar la migración del almacenamiento de discos durante una ventana de mantenimiento existente previamente.
* Si el disco es no administrado, primero [debe convertirlo en un disco administrado](convert-unmanaged-to-managed-disks.md) para poder cambiar entre las opciones de almacenamiento.

## <a name="switch-all-managed-disks-of-a-vm-between-from-one-account-to-another"></a>Cambio de todos los discos administrados de una máquina virtual entre una cuenta y otra

En este ejemplo se muestra cómo convertir todos los discos de una máquina virtual en Premium Storage. Sin embargo, al cambiar la variable $storageType en este ejemplo, puede convertir el tipo de los discos de la máquina virtual en SSD estándar o HDD estándar. Para usar Managed Disks premium, la máquina virtual debe usar un [Tamaño de máquina virtual](../sizes.md) que admita Premium Storage. En este ejemplo también se pasa a un tamaño que admite Premium Storage:

```azurepowershell-interactive
# Name of the resource group that contains the VM
$rgName = 'yourResourceGroup'

# Name of the your virtual machine
$vmName = 'yourVM'

# Choose between Standard_LRS, StandardSSD_LRS and Premium_LRS based on your scenario
$storageType = 'Premium_LRS'

# Premium capable size
# Required only if converting storage from Standard to Premium
$size = 'Standard_DS2_v2'

# Stop and deallocate the VM before changing the size
Stop-AzVM -ResourceGroupName $rgName -Name $vmName -Force

$vm = Get-AzVM -Name $vmName -resourceGroupName $rgName

# Change the VM size to a size that supports Premium storage
# Skip this step if converting storage from Premium to Standard
$vm.HardwareProfile.VmSize = $size
Update-AzVM -VM $vm -ResourceGroupName $rgName

# Get all disks in the resource group of the VM
$vmDisks = Get-AzDisk -ResourceGroupName $rgName 

# For disks that belong to the selected VM, convert to Premium storage
foreach ($disk in $vmDisks)
{
    if ($disk.ManagedBy -eq $vm.Id)
    {
        $disk.Sku = [Microsoft.Azure.Management.Compute.Models.DiskSku]::new($storageType)
        $disk | Update-AzDisk
    }
}

Start-AzVM -ResourceGroupName $rgName -Name $vmName
```

## <a name="switch-individual-managed-disks-between-standard-and-premium"></a>Cambio de discos administrados individuales entre estándar y Premium

En el caso de la carga de trabajo de desarrollo y pruebas, puede que quiera tener una combinación de discos estándar y Premium para reducir los costos. Puede optar por actualizar solamente los discos que necesitan un rendimiento más eficaz. En este ejemplo se muestra cómo convertir un solo disco de máquina virtual de almacenamiento estándar al prémium. Sin embargo, al cambiar la variable $storageType en este ejemplo, puede convertir el tipo de los discos de la máquina virtual en SSD estándar o HDD estándar. Para usar Managed Disks premium, la máquina virtual debe usar un [Tamaño de máquina virtual](../sizes.md) que admita Premium Storage. En este ejemplo se muestra cómo pasar a un tamaño que admite Premium Storage:

```azurepowershell-interactive

$diskName = 'yourDiskName'
# resource group that contains the managed disk
$rgName = 'yourResourceGroupName'
# Choose between Standard_LRS, StandardSSD_LRS and Premium_LRS based on your scenario
$storageType = 'Premium_LRS'
# Premium capable size 
$size = 'Standard_DS2_v2'

$disk = Get-AzDisk -DiskName $diskName -ResourceGroupName $rgName

# Get parent VM resource
$vmResource = Get-AzResource -ResourceId $disk.ManagedBy

# Stop and deallocate the VM before changing the storage type
Stop-AzVM -ResourceGroupName $vmResource.ResourceGroupName -Name $vmResource.Name -Force

$vm = Get-AzVM -ResourceGroupName $vmResource.ResourceGroupName -Name $vmResource.Name 

# Change the VM size to a size that supports Premium storage
# Skip this step if converting storage from Premium to Standard
$vm.HardwareProfile.VmSize = $size
Update-AzVM -VM $vm -ResourceGroupName $rgName

# Update the storage type
$disk.Sku = [Microsoft.Azure.Management.Compute.Models.DiskSku]::new($storageType)
$disk | Update-AzDisk

Start-AzVM -ResourceGroupName $vm.ResourceGroupName -Name $vm.Name
```

## <a name="switch-managed-disks-from-one-disk-type-to-another"></a>Cambio de los discos administrados de un tipo de disco a otro

Siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione la máquina virtual de la lista de **Virtual Machines**.
3. Si la máquina virtual no se detiene, seleccione la opción **Detener** que se encuentra en la parte superior del panel **Información general** de la máquina virtual y espere a que se detenga.
4. En el panel de la máquina virtual, seleccione **Discos** del menú.
5. Seleccione el disco que quiera convertir.
6. Seleccione **Configuración** del menú.
7. Cambie el **tipo de cuenta** del tipo de disco original al tipo de disco deseado.
8. Seleccione **Guardar** y cierre el panel de discos.

La conversión del tipo de disco es instantánea. Puede iniciar la máquina virtual después de la conversión.

## <a name="next-steps"></a>Pasos siguientes

Realice una copia de solo lectura de una máquina virtual mediante una [instantánea](snapshot-copy-managed-disk.md).
