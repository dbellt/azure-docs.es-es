---
title: Restauración de Azure Managed Disks a través de Azure PowerShell
description: Aprenda a restaurar Azure Managed Disks con Azure PowerShell.
ms.topic: conceptual
ms.date: 03/26/2021
ms.openlocfilehash: c6625b43c313d45d4b295dd406e29a2b1d85b387
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107520044"
---
# <a name="restore-azure-managed-disks-using-azure-powershell"></a>Restauración de Azure Managed Disks a través de Azure PowerShell

En este artículo se explica cómo restaurar [Azure Managed Disks](../virtual-machines/managed-disks-overview.md) desde un punto de restauración que haya creado Azure Backup.

Actualmente, no se admite la opción de Recuperación de la ubicación original (OLR) de la restauración mediante la sustitución del disco de origen desde el que se realizaron las copias de seguridad. Puede realizar la restauración desde un punto de recuperación para crear un nuevo disco en el mismo grupo de recursos que el disco de origen desde el que se realizaron las copias de seguridad o en cualquier otro grupo de recursos. Esto se conoce como Recuperación de la ubicación alternativa (ALR), y le será de ayuda para mantener el disco de origen y el disco restaurado (nuevo).

En este artículo, aprenderá a:

- Realizar una restauración para crear un disco nuevo.

- Realizar el seguimiento del estado de la operación de restauración.

Haremos referencia a un almacén de copia de seguridad existente "TestBkpVault" en el grupo de recursos "testBkpVaultRG" en los ejemplos.

```azurepowershell-interactive
$TestBkpVault = Get-AzDataProtectionBackupVault -VaultName TestBkpVault -ResourceGroupName "testBkpVaultRG"
```

## <a name="restore-to-create-a-new-disk"></a>Realizar una restauración para crear un disco nuevo.

### <a name="setting-up-permissions"></a>Configuración de permisos

El almacén de Backup usa la identidad administrada para obtener acceso a otros recursos de Azure. Para restaurar contenido a partir de una copia de seguridad, la identidad administrada del almacén de Backup debe tener un conjunto de permisos en el grupo de recursos en el que se vaya a restaurar el disco.

El almacén de Backup usa una identidad administrada asignada por el sistema; cada recurso solo puede tener una identidad, y cada una de ellas está asociada al ciclo de vida del recurso. Puede conceder permisos a la identidad administrada mediante el control de acceso basado en roles de Azure (Azure RBAC). Tenga en cuenta que una identidad administrada es una entidad de servicio de un tipo especial que solo se puede usar con recursos de Azure. Obtenga más información sobre las [identidades administradas](../active-directory/managed-identities-azure-resources/overview.md).

Asigne los permisos pertinentes para la identidad administrada asignada por el sistema del almacén en el grupo de recursos de destino en el que se restaurarán o crearán los discos tal y como se mencionó [aquí](restore-managed-disks.md#restore-to-create-a-new-disk).

### <a name="fetching-the-relevant-recovery-point"></a>Captura del punto de recuperación pertinente

Recupere todas las instancias mediante el comando [Get-AzDataProtectionBackupInstance](/powershell/module/az.dataprotection/get-azdataprotectionbackupinstance?view=azps-5.7.0&preserve-view=true) e identifique la instancia pertinente.

```azurepowershell-interactive
$AllInstances = Get-AzDataProtectionBackupInstance -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name
```

También puede usar **Az.Resourcegraph** y el comando [Search-AzDataProtectionBackupInstanceInAzGraph](/powershell/module/az.dataprotection/search-azdataprotectionbackupinstanceinazgraph?view=azps-5.7.0&preserve-view=true) para buscar en instancias en muchos almacenes y suscripciones.

```azurepowershell-interactive
$AllInstances = Search-AzDataProtectionBackupInstanceInAzGraph -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name -DatasourceType AzureDisk -ProtectionStatus ProtectionConfigured
```

Una vez identificada la instancia, capture el punto de recuperación pertinente.

```azurepowershell-interactive
$rp = Get-AzDataProtectionRecoveryPoint -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name -BackupInstanceName $AllInstances[2].BackupInstanceName
```

### <a name="preparing-the-restore-request"></a>Preparación de la solicitud de restauración

Construya el id. de ARM del nuevo disco que se va a crear con el grupo de recursos de destino, al que se asignaron los permisos tal y como se ha indicado [anteriormente](#setting-up-permissions), y el nombre de disco necesario. Por ejemplo, un disco puede denominarse **PSTestDisk2** en un grupo de recursos **targetrg** con una suscripción diferente.

```azurepowershell-interactive
$targetDiskId = /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx/resourceGroups/targetrg/providers/Microsoft.Compute/disks/PSTestDisk2
```

Use el comando [Initialize-AzDataProtectionRestoreRequest](/powershell/module/az.dataprotection/initialize-azdataprotectionrestorerequest?view=azps-5.7.0&preserve-view=true) para preparar la solicitud de restauración con todos los detalles pertinentes.

```azurepowershell-interactive
$restorerequest = Initialize-AzDataProtectionRestoreRequest -DatasourceType AzureDisk -SourceDataStore OperationalStore -RestoreLocation $TestBkpVault.Location  -RestoreType AlternateLocation -TargetResourceId $targetDiskId -RecoveryPoint $rp[0].Name
```

### <a name="trigger-the-restore"></a>Desencadenamiento de la restauración

Use el comando [Start-AzDataProtectionBackupInstanceRestore](/powershell/module/az.dataprotection/start-azdataprotectionbackupinstancerestore?view=azps-5.7.0&preserve-view=true) para desencadenar la restauración con la solicitud preparada anteriormente.

```azurepowershell-interactive
Start-AzDataProtectionBackupInstanceRestore -BackupInstanceName $AllInstances[2].BackupInstanceName -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name -Parameter $restorerequest
```

## <a name="tracking-job"></a>Seguimiento de trabajos

Realice el seguimiento de todos los trabajos mediante el comando [Get-AzDataProtectionJob](/powershell/module/az.dataprotection/get-azdataprotectionjob?view=azps-5.7.0&preserve-view=true). Puede enumerar todos los trabajos y capturar un detalle de trabajo determinado.

También puede usar **Az.ResourceGraph** para realizar el seguimiento de todos los trabajos en todos los almacenes de copia de seguridad. Use el comando [Search-AzDataProtectionJobInAzGraph](/powershell/module/az.dataprotection/search-azdataprotectionjobinazgraph?view=azps-5.7.0&preserve-view=true) para obtener el trabajo pertinente, que puede encontrarse en cualquier almacén de copia de seguridad.

```azurepowershell-interactive
$job = Search-AzDataProtectionJobInAzGraph -Subscription $sub -ResourceGroupName "testBkpVaultRG" -Vault $TestBkpVault.Name -DatasourceType AzureDisk -Operation OnDemandBackup
```

## <a name="next-steps"></a>Pasos siguientes

- [Preguntas más frecuentes sobre Azure Disk Backup](disk-backup-faq.yml)
