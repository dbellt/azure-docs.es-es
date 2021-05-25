---
title: Restauración de blobs de Azure mediante Azure PowerShell
description: Aprenda a restaurar blobs de Azure a cualquier momento dado mediante Azure PowerShell.
ms.topic: conceptual
ms.date: 05/05/2021
ms.openlocfilehash: 653c3ce6fbfea4d890c3dc3e52fb1032a72b7eab
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108803069"
---
# <a name="restore-azure-blobs-to-point-in-time-using-azure-powershell"></a>Restauración de blobs de Azure a un momento dado mediante Azure PowerShell

En este artículo se describe cómo restaurar [blobs](blob-backup-overview.md) a cualquier momento dado mediante Azure Backup.

> [!IMPORTANT]
> La compatibilidad con blobs de Azure está disponible a partir de la versión Az 5.9.0.

> [!IMPORTANT]
> Antes de continuar con la restauración de blobs de Azure mediante Azure Backup, consulte los [puntos importantes](blob-restore.md#before-you-start).

En este artículo, aprenderá a:

- Restauración de blobs de Azure a un momento dado

- Realizar el seguimiento del estado de la operación de restauración.

Haremos referencia a un almacén de copia de seguridad existente _TestBkpVault_ en el grupo de recursos _testBkpVaultRG_ en los ejemplos.

```azurepowershell-interactive
$TestBkpVault = Get-AzDataProtectionBackupVault -VaultName TestBkpVault -ResourceGroupName "testBkpVaultRG"
```

## <a name="restoring-azure-blobs-within-a-storage-account"></a>Restauración de blobs de Azure dentro de una cuenta de almacenamiento

### <a name="fetching-the-valid-time-range-for-restore"></a>Captura del intervalo de tiempo válido para la restauración

Como la copia de seguridad operativa de blobs es continua, no hay puntos distintos desde los que restaurar. En su lugar, es necesario capturar el intervalo de tiempo válido en el que los blobs se pueden restaurar a cualquier momento dado. En este ejemplo, vamos a comprobar los intervalos de tiempo válidos para restaurar en los últimos 30 días.

```azurepowershell-interactive
$startDate = (Get-Date).AddDays(-30)
$endDate = Get-Date
```

Recupere primero todas las instancias mediante el comando [Get-AzDataProtectionBackupInstance](/powershell/module/az.dataprotection/get-azdataprotectionbackupinstance?view=azps-5.9.0&preserve-view=true) e identifique la instancia pertinente.

```azurepowershell-interactive
$AllInstances = Get-AzDataProtectionBackupInstance -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name
```

También puede usar Az.Resourcegraph y el comando [Search-AzDataProtectionBackupInstanceInAzGraph](/powershell/module/az.dataprotection/search-azdataprotectionbackupinstanceinazgraph?view=azps-5.9.0&preserve-view=true) para buscar en instancias en muchos almacenes y suscripciones.

```azurepowershell-interactive
$AllInstances = Search-AzDataProtectionBackupInstanceInAzGraph -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name -DatasourceType AzureBlob -ProtectionStatus ProtectionConfigured
```

Una vez identificada la instancia, capture el intervalo de recuperación correspondiente mediante el comando [Find-AzDataProtectionRestorableTimeRange](/powershell/module/az.dataprotection/find-azdataprotectionrestorabletimerange).

```azurepowershell-interactive
Find-AzDataProtectionRestorableTimeRange -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name -BackupInstanceName $AllInstances[2].BackupInstanceName -StartTime $startDate -endTime $endDate

EndTime    : 2021-04-24T08:57:36.4149422Z
ObjectType : RestorableTimeRange
StartTime  : 2021-03-25T14:27:31.0000000Z

$DesiredPIT = (Get-Date -Date "2021-04-23T02:47:02.9500000Z")
```

### <a name="preparing-the-restore-request"></a>Preparación de la solicitud de restauración

Una vez corregido el momento dado para la restauración, hay varias opciones para restaurar. Use el comando [Initialize-AzDataProtectionRestoreRequest](/powershell/module/az.dataprotection/initialize-azdataprotectionrestorerequest?view=azps-5.9.0&preserve-view=true) para preparar la solicitud de restauración con todos los detalles pertinentes.

#### <a name="restoring-all-the-blobs-to-a-point-in-time"></a>Restauración de todos los blobs a un momento dado

Con esta opción, se revierten todos los blobs en bloques de la cuenta de almacenamiento al momento dado seleccionado para restaurarlos. Las cuentas de almacenamiento que contienen grandes cantidades de datos o con una alta tasa de renovación pueden tardar más tiempo en restaurarse.

```azurepowershell-interactive
$restorerequest = Initialize-AzDataProtectionRestoreRequest -DatasourceType AzureBlob -SourceDataStore OperationalStore -RestoreLocation $TestBkpVault.Location  -RestoreType OriginalLocation -PointInTime (Get-Date -Date "2021-04-23T02:47:02.9500000Z") -BackupInstance $AllInstances[2]
```

#### <a name="restoring-selected-containers"></a>Restauración de los contenedores seleccionados

Con esta opción, podrá examinar y seleccionar hasta 10 contenedores para restaurar.

```azurepowershell-interactive
$restorerequest = Initialize-AzDataProtectionRestoreRequest -DatasourceType AzureBlob -SourceDataStore OperationalStore -RestoreLocation $TestBkpVault.Location  -RestoreType OriginalLocation -PointInTime (Get-Date -Date "2021-04-23T02:47:02.9500000Z") -BackupInstance $AllInstances[2] -ItemLevelRecovery -ContainersList "abc","xyz"
```

#### <a name="restoring-containers-using-a-prefix-match"></a>Restauración de contenedores mediante una coincidencia de prefijo

Esta opción permite restaurar un subconjunto de blobs que coincidan con un prefijo. Puede restaurar hasta 10 intervalos lexicográficos de blobs dentro de un solo contenedor o en varios contenedores para devolver esos blobs a su estado anterior en un momento dado. Hay algunos aspectos que se deben tener en cuenta:

- Puede usar una barra diagonal (/) para delimitar el nombre del contenedor del prefijo de blob.
- El inicio del intervalo especificado es inclusivo, pero el intervalo especificado es exclusivo.

[Obtenga más información](blob-restore.md#use-prefix-match-for-restoring-blobs) sobre el uso de prefijos para restaurar intervalos de blobs.

```azurepowershell-interactive
$restorerequest = Initialize-AzDataProtectionRestoreRequest -DatasourceType AzureBlob -SourceDataStore OperationalStore -RestoreLocation $TestBkpVault.Location  -RestoreType OriginalLocation -PointInTime (Get-Date -Date "2021-04-23T02:47:02.9500000Z") -BackupInstance $AllInstances[2] -ItemLevelRecovery -FromPrefixPattern "containerabc/aaa","containerabc/ccc" -ToPrefixPattern "containerabc/bbb","containerabc/ddd"
```

### <a name="trigger-the-restore"></a>Desencadenamiento de la restauración

Use el comando [Start-AzDataProtectionBackupInstanceRestore](/powershell/module/az.dataprotection/start-azdataprotectionbackupinstancerestore?view=azps-5.9.0&preserve-view=true) para desencadenar la restauración con la solicitud preparada anteriormente.

```azurepowershell-interactive
Start-AzDataProtectionBackupInstanceRestore -BackupInstanceName $AllInstances[2].BackupInstanceName -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name -Parameter $restorerequest
```

## <a name="tracking-job"></a>Seguimiento de trabajos

Realice el seguimiento de todos los trabajos mediante el comando [Get-AzDataProtectionJob](/powershell/module/az.dataprotection/get-azdataprotectionjob?view=azps-5.9.0&preserve-view=true). Puede enumerar todos los trabajos y capturar un detalle de trabajo determinado.

También puede usar Az.ResourceGraph para realizar el seguimiento de todos los trabajos en todos los almacenes de copia de seguridad. Use el comando [Search-AzDataProtectionJobInAzGraph](/powershell/module/az.dataprotection/search-azdataprotectionjobinazgraph?view=azps-5.9.0&preserve-view=true) para obtener el trabajo pertinente, que puede encontrarse en cualquier almacén de copia de seguridad.

```azurepowershell-interactive
$job = Search-AzDataProtectionJobInAzGraph -Subscription $sub -ResourceGroupName "testBkpVaultRG" -Vault $TestBkpVault.Name -DatasourceType AzureBlob -Operation Restore
```

## <a name="next-steps"></a>Pasos siguientes

[Introducción a la copia de seguridad de blobs de Azure](blob-backup-overview.md)
