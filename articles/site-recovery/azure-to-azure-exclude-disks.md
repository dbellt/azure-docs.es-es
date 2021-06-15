---
title: Exclusión de los discos de máquina virtual de Azure de la replicación con Azure Site Recovery y Azure PowerShell
description: Aprenda excluir discos de máquinas virtuales de Azure con Azure PowerShell durante Azure Site Recovery.
author: sideeksh
manager: rochakm
ms.topic: how-to
ms.date: 02/18/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 2b9ccb90c13bf4d2e764af10f52718db928fb8e5
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110698297"
---
# <a name="exclude-disks-from-powershell-replication-of-azure-vms"></a>Exclusión de discos de la replicación de máquinas virtuales de Azure en PowerShell

En este artículo se describe cómo excluir discos al replicar máquinas virtuales de Azure. Puede excluir discos para optimizar el ancho de banda consumido con la replicación o los recursos de destino que usan esos discos. Actualmente, esta funcionalidad solo está disponible mediante Azure PowerShell.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar:

- Asegúrese de entender la [arquitectura y los componentes de la recuperación ante desastres](azure-to-azure-architecture.md).
- Revise los [requisitos de compatibilidad](azure-to-azure-support-matrix.md) de todos los componentes.
- Asegúrese de que dispone del módulo AzureRm PowerShell "Az". Para instalar o actualizar PowerShell, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps).
- Asegúrese de que ha creado un almacén de Recovery Services y máquinas virtuales protegidas al menos una vez. Si aún no lo ha hecho, siga el proceso en [Configuración de la recuperación ante desastres en máquinas virtuales de Azure mediante Azure PowerShell](azure-to-azure-powershell.md).
- Si está buscando información sobre cómo agregar discos a una máquina virtual de Azure habilitada para la replicación, [consulte este artículo](azure-to-azure-enable-replication-added-disk.md).

## <a name="why-exclude-disks-from-replication"></a>¿Por qué excluir discos de la replicación?
Es posible que deba excluir discos de la replicación por los siguientes motivos:

- La máquina virtual ha alcanzado los [límites de Azure Site Recovery para replicar frecuencias de cambio de datos](./azure-to-azure-support-matrix.md).

- Los datos fragmentados en el disco excluido no son importantes o no es necesario replicarlos.

- No quiere replicar los datos para ahorrar recursos de almacenamiento y red.

## <a name="how-to-exclude-disks-from-replication"></a>¿Cómo se excluyen los discos de la replicación?

En nuestro ejemplo, se replica una máquina virtual que tiene un sistema operativo y tres discos de datos que están en la región Este de EE. UU. en la región Oeste de EE. UU. 2. El nombre de la máquina virtual es *AzureDemoVM*. Se excluye el disco 1 y se mantienen los discos 2 y 3.

## <a name="get-details-of-the-virtual-machines-to-replicate"></a>Obtención de detalles de las máquinas virtuales que se van a replicar

```azurepowershell
# Get details of the virtual machine
$VM = Get-AzVM -ResourceGroupName "A2AdemoRG" -Name "AzureDemoVM"

Write-Output $VM     
```

```
ResourceGroupName  : A2AdemoRG
Id                 : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/A2AdemoRG/providers/Microsoft.Compute/virtualMachines/AzureDemoVM
VmId               : 1b864902-c7ea-499a-ad0f-65da2930b81b
Name               : AzureDemoVM
Type               : Microsoft.Compute/virtualMachines
Location           : eastus
Tags               : {}
DiagnosticsProfile : {BootDiagnostics}
HardwareProfile    : {VmSize}
NetworkProfile     : {NetworkInterfaces}
OSProfile          : {ComputerName, AdminUsername, WindowsConfiguration, Secrets}
ProvisioningState  : Succeeded
StorageProfile     : {ImageReference, OsDisk, DataDisks}
```

Obtenga detalles sobre los discos de la máquina virtual. Esta información se usará más adelante al iniciar la replicación de la máquina virtual.

```azurepowershell
$OSDiskVhdURI = $VM.StorageProfile.OsDisk.Vhd
$DataDisk1VhdURI = $VM.StorageProfile.DataDisks[0].Vhd
```

## <a name="replicate-an-azure-virtual-machine"></a>Replicación de una máquina virtual de Azure

En el siguiente ejemplo hemos asumido que ya tiene una cuenta de almacenamiento en caché, una directiva de replicación y asignaciones. Si no tiene estas cosas, siga el proceso que se describe en [Configuración de la recuperación ante desastres en máquinas virtuales de Azure mediante Azure PowerShell](azure-to-azure-powershell.md).

Replique la máquina virtual de Azure con *discos administrados*.

```azurepowershell

#Get the resource group that the virtual machine must be created in when failed over.
$RecoveryRG = Get-AzResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"

#Specify replication properties for each disk of the VM that is to be replicated (create disk replication configuration).

#OsDisk
$OSdiskId =  $vm.StorageProfile.OsDisk.ManagedDisk.Id
$RecoveryOSDiskAccountType = $vm.StorageProfile.OsDisk.ManagedDisk.StorageAccountType
$RecoveryReplicaDiskAccountType =  $vm.StorageProfile.OsDisk.ManagedDisk.StorageAccountType

$OSDiskReplicationConfig = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id `
         -DiskId $OSdiskId -RecoveryResourceGroupId  $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType  $RecoveryReplicaDiskAccountType `
         -RecoveryTargetDiskAccountType $RecoveryOSDiskAccountType

# Data Disk 1 i.e StorageProfile.DataDisks[0] is excluded, so we will provide it during the time of replication.

# Data disk 2
$datadiskId2  = $vm.StorageProfile.DataDisks[1].ManagedDisk.id
$RecoveryReplicaDiskAccountType =  $vm.StorageProfile.DataDisks[1]. StorageAccountType
$RecoveryTargetDiskAccountType = $vm.StorageProfile.DataDisks[1]. StorageAccountType

$DataDisk2ReplicationConfig  = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $CacheStorageAccount.Id `
         -DiskId $datadiskId2 -RecoveryResourceGroupId  $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType  $RecoveryReplicaDiskAccountType `
         -RecoveryTargetDiskAccountType $RecoveryTargetDiskAccountType

# Data Disk 3

$datadiskId3  = $vm.StorageProfile.DataDisks[2].ManagedDisk.id
$RecoveryReplicaDiskAccountType =  $vm.StorageProfile.DataDisks[2]. StorageAccountType
$RecoveryTargetDiskAccountType = $vm.StorageProfile.DataDisks[2]. StorageAccountType

$DataDisk3ReplicationConfig  = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $CacheStorageAccount.Id `
         -DiskId $datadiskId3 -RecoveryResourceGroupId  $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType  $RecoveryReplicaDiskAccountType `
         -RecoveryTargetDiskAccountType $RecoveryTargetDiskAccountType

#Create a list of disk replication configuration objects for the disks of the virtual machine that are to be replicated.
$diskconfigs = @()
$diskconfigs += $OSDiskReplicationConfig, $DataDisk2ReplicationConfig, $DataDisk3ReplicationConfig


#Start replication by creating a replication protected item. Using a GUID for the name of the replication protected item to ensure uniqueness of name.
$TempASRJob = New-ASRReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId
```

Cuando la operación de replicación inicial se realiza correctamente, se replican los datos de la máquina virtual en la región de recuperación.

Puede ir a Azure Portal y ver las máquinas virtuales replicadas en "elementos replicados".

El proceso de replicación comienza con la propagación de una copia de los discos de replicación de la máquina virtual en la región de recuperación. Esta fase se conoce como fase de replicación inicial.

Una vez finalizada la replicación inicial, la replicación pasa a la fase de sincronización diferencial. En este momento, la máquina virtual está protegida. Seleccione la máquina virtual protegida para ver si se excluyen todos los discos.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre la [ejecución de una conmutación por error de prueba](site-recovery-test-failover-to-azure.md).
