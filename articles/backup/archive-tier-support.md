---
title: Compatibilidad del nivel de acceso de archivo (versión preliminar)
description: Conozca más sobre la compatibilidad del nivel de acceso de archivo para Azure Backup.
ms.topic: conceptual
ms.date: 02/18/2021
ms.openlocfilehash: 7a42b8702cfdda14a18aa3cdd4e084ed78767b0a
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107012155"
---
# <a name="archive-tier-support-preview"></a>Compatibilidad del nivel de acceso de archivo (versión preliminar)

Los clientes confían en Azure Backup para almacenar los datos de copia de seguridad, incluidos los datos de copia de seguridad de retención a largo plazo (LTR), donde las necesidades de retención están definidas por las reglas de cumplimiento de la organización. En la mayoría de los casos, pocas veces se accede a los datos de copia de seguridad antiguos, que solo se almacenan para satisfacer las necesidades de cumplimiento.

Azure Backup admite la copia de seguridad de puntos de retención a largo plazo en el nivel de archivo, además de instantáneas y el nivel estándar.

## <a name="scope-for-preview"></a>Ámbito de la versión preliminar

Cargas de trabajo compatibles:

- Azure Virtual Machines
  - Solo puntos de recuperación mensuales y anuales. No se admiten los puntos de recuperación diarios y semanales.
  - Antigüedad >= 3 meses en el nivel estándar del almacén
  - Retención restante >= 6 meses
  - Sin dependencias diarias y semanales activas
- SQL Server en máquinas virtuales de Azure
  - Solo puntos de recuperación completos. No se admiten registros ni diferenciales.
  - Antigüedad >= 45 días en el nivel estándar del almacén
  - Retención restante >= 6 meses
  - Ninguna dependencia

Clientes compatibles:

- La funcionalidad se proporciona mediante PowerShell

>[!NOTE]
>La compatibilidad con el nivel de archivo para máquinas virtuales de Azure y SQL Server en máquinas virtuales de Azure se encuentra en versión preliminar pública limitada con suscripciones limitadas. Para suscribirse al soporte de archivo, use este [vínculo](https://aka.ms/ArchivePreviewInterestForm).

## <a name="get-started-with-powershell"></a>Introducción a PowerShell

1. Descargue la versión [más reciente](https://github.com/PowerShell/PowerShell/releases) de PowerShell en GitHub.

1. En PowerShell, ejecute el siguiente comando:
  
    ```azurepowershell
    install-module -name Az.RecoveryServices -Repository PSGallery -RequiredVersion 4.0.0-preview -AllowPrerelease -force
    ```

1. Conéctese a Azure mediante el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).
1. Inicie sesión en su suscripción:

   `Set-AzContext -Subscription "SubscriptionName"`

1. Obtenga el almacén:

    `$vault =  Get-AzRecoveryServicesVault -ResourceGroupName "rgName" -Name "vaultName"`

1. Obtenga la lista de elementos de copia de seguridad:

    - Para las máquinas virtuales de Azure:

        `$BackupItemList = Get-AzRecoveryServicesBackupItem -vaultId $vault.ID -BackupManagementType "AzureVM" -WorkloadType "AzureVM"`

    - Para SQL Server en máquinas virtuales de Azure:

        `$BackupItemList = Get-AzRecoveryServicesBackupItem -vaultId $vault.ID -BackupManagementType "AzureWorkload" -WorkloadType "MSSQL"`

1. Obtenga el elemento de copia de seguridad.

    - Para las máquinas virtuales de Azure:

        `$bckItm = $BackupItemList | Where-Object {$_.Name -match '<vmName>'}`

    - Para SQL Server en máquinas virtuales de Azure:

        `$bckItm = $BackupItemList | Where-Object {$_.Name -match '<dbName>' -and $_.ContainerName -match '<vmName>'}`

## <a name="use-powershell"></a>Uso de PowerShell

### <a name="check-archivable-recovery-points"></a>Comprobación de los puntos de recuperación que se pueden archivar

```azurepowershell
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -VaultId $vault.ID -Item $bckItm  -IsReadyForMove $true -TargetTier VaultArchive
```

Con esto se enumerarán todos los puntos de recuperación asociados a un elemento de copia de seguridad determinado que estén listos para moverse al archivo.

### <a name="check-why-a-recovery-point-cannot-be-moved-to-archive"></a>Comprobación de por qué un punto de recuperación no se puede migrar al archivo

```azurepowershell
$rp[0].RecoveryPointMoveReadinessInfo["ArchivedRP"]
```

Donde `$rp[0]` es el punto de recuperación para el que quiere comprobar por qué no se puede archivar.

Salida del ejemplo:

```output
IsReadyForMove  AdditionalInfo
--------------  --------------
False           Recovery-Point Type is not eligible for archive move as it is already moved to archive tier
```

### <a name="check-recommended-set-of-archivable-points-only-for-azure-vms"></a>Comprobación del conjunto recomendado de puntos que se pueden archivar (solo para VM de Azure)

Los puntos de recuperación asociados con una máquina virtual tienen una naturaleza incremental. Cuando un punto de recuperación determinado se mueve al archivo, se convierte en copia de seguridad completa y, a continuación, se mueve al archivo. Por lo tanto, el ahorro de costos asociado al traslado al archivo depende de la renovación del origen de datos.

Debido a esto, Azure Backup ha ideado un conjunto recomendado de puntos de recuperación que pueden dar lugar a un ahorro de costos si se mueven juntos.

>[!NOTE]
>El ahorro de costos depende de una serie de motivos, y puede que no sean los mismos para dos instancias cualesquiera.

```azurepowershell
$RecommendedRecoveryPointList = Get-AzRecoveryServicesBackupRecommendedArchivableRPGroup -Item $bckItm -VaultId $vault.ID
```

### <a name="move-to-archive"></a>Traslado al nivel de archivo

```azurepowershell
Move-AzRecoveryServicesBackupRecoveryPoint -VaultId $vault.ID -RecoveryPoint $rp[2] -SourceTier VaultStandard -DestinationTier VaultArchive
```

Este comando mueve al archivo un punto de recuperación que se puede archivar. Devuelve un trabajo que se puede usar para hacer el seguimiento de la operación de movimiento desde el portal y con PowerShell.

### <a name="view-archived-recovery-points"></a>Visualización de puntos de recuperación archivados

Este comando devuelve todos los puntos de recuperación archivados.

```azurepowershell
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -VaultId $vault.ID -Item $bckItm -Tier VaultArchive
```

### <a name="restore-with-powershell"></a>Restauración con PowerShell

En el caso de los puntos de recuperación en archivo, Azure Backup proporciona una metodología de restauración integrada.

La restauración integrada es un proceso de dos pasos. El primer paso implica rehidratar los puntos de recuperación almacenados en el archivo y almacenarlos temporalmente en el nivel estándar del almacén durante un período (también conocido como "duración de rehidratación") de 10 a 30 días. El valor predeterminado es de 15 días. Hay dos prioridades diferentes de rehidratación: prioridad estándar y alta. Más información sobre la [prioridad de rehidratación](../storage/blobs/storage-blob-rehydration.md#rehydrate-an-archived-blob-to-an-online-tier).

>[!NOTE]
>
>- La duración de rehidratación no se puede cambiar una vez seleccionada, y los puntos de recuperación rehidratados permanecen en el nivel estándar durante este período de rehidratación.
>- El paso adicional de rehidratación conlleva un costo.

Para obtener más información sobre los distintos métodos de restauración de máquinas virtuales de Azure, consulte [Restauración de una VM de Azure con PowerShell](backup-azure-vms-automation.md#restore-an-azure-vm).

```azurepowershell
Restore-AzRecoveryServicesBackupItem -VaultLocation $vault.Location -RehydratePriority "Standard" -RehydrateDuration 15 -RecoveryPoint $rp -StorageAccountName "SampleSA" -StorageAccountResourceGroupName "SArgName" -TargetResourceGroupName $vault.ResourceGroupName -VaultId $vault.ID
```

Para restaurar SQL Server, siga [estos pasos](backup-azure-sql-automation.md#restore-sql-dbs). Los parámetros adicionales obligatorios son **RehydrationPriority** y **RehydrationDuration**.

### <a name="view-jobs-from-powershell"></a>Visualización de trabajos desde PowerShell

Para ver los trabajos de movimiento y restauración, use el siguiente cmdlet de PowerShell:

```azurepowershell
Get-AzRecoveryServicesBackupJob -VaultId $vault.ID
```

## <a name="use-the-portal"></a>Uso del portal

### <a name="check-archived-recovery-point"></a>Comprobación de puntos de recuperación archivados

Ahora puede ver todos los puntos de recuperación que se han movido al nivel de archivo.

![Todos los puntos de recuperación.](./media/archive-tier-support/restore-points.png)

### <a name="restore-in-the-portal"></a>Restauración en el portal

En el caso de los puntos de recuperación que se han movido al nivel de archivo, la restauración requiere que agregue los parámetros para la duración de rehidratación y la prioridad de rehidratación.

![Restauración en el portal.](./media/archive-tier-support/restore-in-portal.png)

### <a name="view-jobs-in-the-portal"></a>Visualización de trabajos en el portal

![Visualización de trabajos en el portal.](./media/archive-tier-support/view-jobs-portal.png)

### <a name="modify-protection"></a>Modificación de la protección

Hay dos maneras en las que puede modificar la protección de un origen de datos:

- Modificación de una directiva existente
- Protección del origen de datos con una nueva directiva

En ambos casos, la nueva directiva se aplica a todos los puntos de recuperación más antiguos, que están en el nivel estándar, y a aquellos que están en el nivel de archivo. Por tanto, es posible que se eliminen los puntos de recuperación más antiguos si se produce un cambio en la directiva.

Cuando los puntos de recuperación se mueven al nivel de archivo, están sujetos a un período de eliminación temprana de 180 días. Los cargos se prorratean. Si se elimina un punto de recuperación que no se ha mantenido en el archivo durante 180 días, se incurrirá en un costo equivalente a 180 menos el número de días que haya estado en el nivel estándar.

Los puntos de recuperación que no se hayan mantenido en el nivel de archivo durante un período mínimo de seis meses incurrirán en el costo de eliminación temprana en el momento de la eliminación.

## <a name="stop-protection-and-delete-data"></a>Detener la protección y eliminar los datos

La detención de la protección y la eliminación de los datos eliminan todos los puntos de recuperación. En el caso de los puntos de recuperación archivados que no hayan permanecido durante 180 días en el nivel de archivo, la eliminación de los puntos de recuperación dará lugar a un costo de eliminación temprana.

## <a name="error-codes-and-troubleshooting-steps"></a>Códigos de error y pasos de solución de problemas

Hay varios códigos de error que aparecen cuando no se puede mover un punto de recuperación al archivo.

### <a name="recoverypointtypenoteligibleforarchive"></a>RecoveryPointTypeNotEligibleForArchive

**Mensaje de error**: Recovery-Point Type is not eligible for Archive Move (El tipo de punto de recuperación no es válido para mover a archivo).

**Descripción**: este código de error se muestra cuando el tipo de punto de recuperación seleccionado no es válido para moverlo al nivel de archivo.

**Acción recomendada**: compruebe que el punto de recuperación sea apto [aquí](#scope-for-preview).

### <a name="recoverypointhaveactivedependencies"></a>RecoveryPointHaveActiveDependencies

**Mensaje de error**: Recovery-Point having active dependencies for restore is not eligible for Archive Move (El punto de recuperación que tiene dependencias activas para la restauración no es apto para mover a archivo)

**Descripción:** el punto de recuperación seleccionado tiene dependencias activas y, por tanto, no se puede mover al nivel de archivo.

**Acción recomendada**: compruebe que el punto de recuperación sea apto [aquí](#scope-for-preview).

### <a name="minlifespaninstandardrequiredforarchive"></a>MinLifeSpanInStandardRequiredForArchive

**Mensaje de error**: El punto de recuperación no es apto para mover el archivo, ya que la vida útil pasada en el nivel estándar del almacén es inferior al mínimo recomendado.

**Descripción**: el punto de recuperación tiene que permanecer en el nivel Estándar durante un mínimo de tres meses para las máquinas virtuales de Azure, y de 45 días para SQL Server en máquinas virtuales de Azure

**Acción recomendada**: compruebe que el punto de recuperación sea apto [aquí](#scope-for-preview).

### <a name="minremaininglifespaninarchiverequired"></a>MinRemainingLifeSpanInArchiveRequired

**Mensaje de error**: El tiempo de vida restante del punto de recuperación es inferior al mínimo requerido.

**Descripción**: la duración mínima necesaria para que un punto de recuperación sea apto para mover al nivel de archivo es de seis meses.

**Acción recomendada**: compruebe que el punto de recuperación sea apto [aquí](#scope-for-preview).

### <a name="usererrorrecoverypointalreadyinarchivetier"></a>UserErrorRecoveryPointAlreadyInArchiveTier

**Mensaje de error**: El punto de recuperación no es apto para mover el archivo porque ya se movió al nivel de archivo.

**Descripción**: el punto de recuperación seleccionado ya está en archivo. Por lo tanto, no es apto para moverlo al nivel de archivo.

### <a name="usererrordatasourcetypeisnotsupportedforrecommendationapi"></a>UserErrorDatasourceTypeIsNotSupportedForRecommendationApi

**Mensaje de error**: El tipo de origen de datos no es apto para la API de recomendación

**Descripción**: la API de recomendación solo es aplicable a máquinas virtuales de Azure. No es aplicable al tipo de origen de datos seleccionado.

### <a name="usererrorrecoverypointalreadyrehydrated"></a>UserErrorRecoveryPointAlreadyRehydrated

**Mensaje de error**: El punto de recuperación ya está rehidratado. No se permite la rehidratación en este RP.

**Descripción**: el punto de recuperación seleccionado ya está rehidratado.

### <a name="usererrorrecoverypointisnoteligibleforarchivemove"></a>UserErrorRecoveryPointIsNotEligibleForArchiveMove

**Mensaje de error**: El punto de recuperación no es apto para mover el archivo.

**Descripción**: el punto de recuperación seleccionado no es apto para mover al nivel de archivo.

### <a name="usererrorrecoverypointnotrehydrated"></a>UserErrorRecoveryPointNotRehydrated

**Mensaje de** **error**: Archive Recovery Point is not rehydrated. Retry Restore after rehydration completed on Archive RP (El punto de recuperación del archivo no está rehidratado. Vuelva a intentar la restauración una vez completada la rehidratación en el RP del archivo).

**Descripción**: el punto de recuperación no está rehidratado. Pruebe a restaurar después de rehidratar el punto de recuperación.

### <a name="usererrorrecoverypointrehydrationnotallowed"></a>UserErrorRecoveryPointRehydrationNotAllowed

**Mensaje de** **error**: Rehydration is only supported for Archive Recovery Points (La rehidratación solo se admite para puntos de recuperación de archivo).

**Descripción**: no se permite la rehidratación para el punto de recuperación seleccionado.

### <a name="usererrorrecoverypointrehydrationalreadyinprogress"></a>UserErrorRecoveryPointRehydrationAlreadyInProgress

**Mensaje de error**: Rehydration is already In-Progress for Archive Recovery Point (La rehidratación ya está en curso para el punto de recuperación del archivo).

**Descripción**: la rehidratación para el punto de recuperación seleccionado ya está en curso.

### <a name="rpmovenotsupportedduetoinsufficientretention"></a>RPMoveNotSupportedDueToInsufficientRetention

**Mensaje de error**: El punto de recuperación no se puede mover al nivel de archivo debido a que la duración de retención especificada en la directiva es insuficiente.

**Acción recomendada**: actualice la directiva en el elemento protegido con la configuración de retención adecuada y vuelva a intentarlo.

### <a name="rpmovereadinesstobedetermined"></a>RPMoveReadinessToBeDetermined

**Mensaje de error**: We're still determining if this Recovery Point can be moved (Aún se está determinando si se puede mover este punto de recuperación).

**Descripción**: todavía se está determinando la preparación para el movimiento del punto de recuperación.

**Acción recomendada**: vuelva a comprobar después de esperar un tiempo.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="what-will-happen-to-archive-recovery-points-if-i-stop-protection-and-retain-data"></a>¿Qué ocurrirá en los puntos de recuperación de archivo si detengo la protección y conservo los datos?

El punto de recuperación permanecerá en el archivo para siempre. Para obtener más información, consulte [Impacto de detener la protección en los puntos de recuperación](manage-recovery-points.md#impact-of-stop-protection-on-recovery-points).

## <a name="next-steps"></a>Pasos siguientes

- [Precios de Azure Backup](azure-backup-pricing.md)