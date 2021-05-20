---
title: Copia de seguridad de blobs de Azure en una cuenta de almacenamiento mediante Azure PowerShell
description: Aprenda a hacer una copia de seguridad de todos los blobs de Azure de una cuenta de almacenamiento mediante Azure PowerShell.
ms.topic: conceptual
ms.date: 05/05/2021
ms.openlocfilehash: ed657d40a5967431c9bf2017d47d0d3c13cd9908
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108803141"
---
# <a name="back-up-all-azure-blobs-in-a-storage-account-using-azure-powershell"></a>Copia de seguridad de todos los blobs de Azure de una cuenta de almacenamiento mediante Azure PowerShell

En este artículo se describe cómo hacer una copia de seguridad de todos los [blobs de Azure](/azure/backup/blob-backup-overview) de una cuenta de almacenamiento mediante Azure PowerShell.

En este artículo, aprenderá a:

- Creación de un almacén de Backup

- Crear una directiva de copia de seguridad

- Configurar una copia de seguridad de todos los blobs de Azure en cuentas de almacenamiento

Para obtener información sobre la disponibilidad regional de los blobs de Azure, los escenarios admitidos y las limitaciones, consulte la [matriz de compatibilidad](blob-backup-support-matrix.md).

> [!IMPORTANT]
> La compatibilidad con blobs de Azure está disponible a partir de la versión Az 5.9.0.

## <a name="create-a-backup-vault"></a>Creación de un almacén de Backup

Un almacén de Backup es una entidad de almacenamiento de Azure que contiene los datos de las copias de seguridad de varias cargas de trabajo recientes que admite Azure Backup, como los servidores de Azure Database for PostgreSQL y los blobs de Azure. Los almacenes de Backup facilitan la tarea de organizar los datos de copia de seguridad, al mismo tiempo que reducen al mínimo la sobrecarga administrativa. Los almacenes de copias de seguridad se basan en el modelo de Azure Resource Manager de Azure, que proporciona características para proteger los datos de las copias de seguridad.

Antes de crear un almacén de copia de seguridad, elija la redundancia de almacenamiento de los datos dentro del almacén. Después, continúe con la creación del almacén de copia de seguridad con esa redundancia de almacenamiento y la ubicación. En este artículo, se creará el almacén de copia de seguridad _TestBkpVault_ en la región _westus_, dentro del grupo de recursos _testBkpVaultRG_. Use el comando [New-AzDataProtectionBackupVault](/powershell/module/az.dataprotection/new-azdataprotectionbackupvault?view=azps-5.9.0&preserve-view=true) para crear un almacén de copia de seguridad. Más información sobre [cómo crear un almacén de copia de seguridad](./backup-vault-overview.md#create-a-backup-vault).

```azurepowershell-interactive
$storageSetting = New-AzDataProtectionBackupVaultStorageSettingObject -Type LocallyRedundant/GeoRedundant -DataStoreType VaultStore

New-AzDataProtectionBackupVault -ResourceGroupName testBkpVaultRG -VaultName TestBkpVault -Location westus -StorageSetting $storageSetting
$TestBkpVault = Get-AzDataProtectionBackupVault -VaultName TestBkpVault
$TestBKPVault | fl
ETag                :
Id                  : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx/resourceGroups/testBkpVaultRG/providers/Microsoft.DataProtection/backupVaults/TestBkpVault
Identity            : Microsoft.Azure.PowerShell.Cmdlets.DataProtection.Models.Api20210201Preview.DppIdentityDetails
IdentityPrincipalId :
IdentityTenantId    :
IdentityType        :
Location            : westus
Name                : TestBkpVault
ProvisioningState   : Succeeded
StorageSetting      : {Microsoft.Azure.PowerShell.Cmdlets.DataProtection.Models.Api20210201Preview.StorageSetting}
SystemData          : Microsoft.Azure.PowerShell.Cmdlets.DataProtection.Models.Api20210201Preview.SystemData
Tag                 : Microsoft.Azure.PowerShell.Cmdlets.DataProtection.Models.Api20210201Preview.DppTrackedResourceTags
Type                : Microsoft.DataProtection/backupVaults
```

Después de la creación del almacén, vamos a crear una directiva de copia de seguridad para proteger los blobs de Azure.

> [!IMPORTANT]
> Aunque verá la redundancia de almacenamiento de Backup del almacén, esta redundancia no se aplica realmente a la copia de seguridad operativa de blobs, ya que la copia de seguridad es local por naturaleza y no se almacena ningún dato en el almacén de Backup. El almacén de Backup. Aquí, el almacén de Backup es la entidad de administración que le ayuda a administrar la protección de los blobs en bloques en las cuentas de almacenamiento.

## <a name="create-a-backup-policy"></a>Creación de una directiva de copia de seguridad

> [!IMPORTANT]
> Lea [esta sección](blob-backup-configure-manage.md#before-you-start) antes de continuar para crear la directiva y configurar las copias de seguridad de los blobs de Azure.

Para comprender los componentes internos de una directiva de copia de seguridad para los blobs de Azure, recupere la plantilla de directiva con el comando [Get-AzDataProtectionPolicyTemplate](/powershell/module/az.dataprotection/get-azdataprotectionpolicytemplate?view=azps-5.9.0&preserve-view=true). Este comando devuelve una plantilla de directiva predeterminada para un tipo de origen de datos determinado. Use esta plantilla para crear una directiva.

```azurepowershell-interactive
$policyDefn = Get-AzDataProtectionPolicyTemplate -DatasourceType AzureBlob
$policyDefn | fl


DatasourceType : {Microsoft.Storage/storageAccounts/blobServices}
ObjectType     : BackupPolicy
PolicyRule     : {Default}

$policyDefn.PolicyRule | fl

IsDefault  : True
Lifecycle  : {Microsoft.Azure.PowerShell.Cmdlets.DataProtection.Models.Api202101.SourceLifeCycle}
Name       : Default
ObjectType : AzureRetentionRule
```

La plantilla de directiva solo consta de un ciclo de vida (que decide cuándo eliminar, copiar o mover la copia de seguridad). Dado que la copia de seguridad operativa de los blobs es continua por naturaleza, no necesita una programación para realizar copias de seguridad.

```azurepowershell-interactive
$policyDefn.PolicyRule.Lifecycle | fl


DeleteAfterDuration        : P30D
DeleteAfterObjectType      : AbsoluteDeleteOption
SourceDataStoreObjectType  : DataStoreInfoBase
SourceDataStoreType        : OperationalStore
TargetDataStoreCopySetting :
```

> [!NOTE]
> La restauración de duraciones largas puede dar lugar a que las operaciones de restauración tarden más en completarse. Por otra parte, el tiempo que se tarda en restaurar un conjunto de datos se basa en el número de operaciones de escritura y eliminación realizadas durante el período de restauración. Por ejemplo, una cuenta con un millón de objetos con 3000 objetos agregados al día y 1000 objetos eliminados al día requerirá aproximadamente dos horas para restaurar a un período de 30 días determinado del pasado.<br><br>Con esta tasa de cambio, no se recomienda un período de retención ni una restauración que vayan más allá de los 90 días para una cuenta.

Una vez que el objeto de directiva tenga todos los valores deseados, continúe con la creación de una directiva a partir de dicho objeto con el comando [New-AzDataProtectionBackupPolicy](/powershell/module/az.dataprotection/new-azdataprotectionbackuppolicy?view=azps-5.9.0&preserve-view=true).

```azurepowershell-interactive
New-AzDataProtectionBackupPolicy -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name -Name blobBkpPolicy -Policy $policyDefn

Name                   Type
----                   ----
blobBkpPolicy       Microsoft.DataProtection/backupVaults/backupPolicies

$blobBkpPol = Get-AzDataProtectionBackupPolicy -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name -Name "blobBkpPolicy"
```

## <a name="configure-backup"></a>Configuración de la copia de seguridad

Una vez que se han creado el almacén y la directiva, hay dos puntos esenciales que el usuario debe tener en cuenta para proteger todos los blobs de Azure de una cuenta de almacenamiento.

### <a name="key-entities-involved"></a>Entidades clave implicadas

#### <a name="storage-account-which-contains-the-blobs-to-be-protected"></a>Cuenta de almacenamiento que contiene los blobs que se protegerán

Recupere el identificador de Azure Resource Manager de la cuenta de almacenamiento que contiene los blobs que se protegerán. Servirá como identificador de la cuenta de almacenamiento. Usaremos un ejemplo de una cuenta de almacenamiento denominada _PSTestSA_, incluida en el grupo de recursos _blobrg_, en otra suscripción.

```azurepowershell-interactive
$SAId = "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx/resourcegroups/blobrg/providers/Microsoft.Storage/storageAccounts/PSTestSA"
```

#### <a name="backup-vault"></a>Almacén de Backup

El almacén de Backup requiere permisos en la cuenta de almacenamiento para habilitar las copias de seguridad en los blobs presentes en la cuenta de almacenamiento. Para asignar estos permisos, se usa la identidad del almacén administrada asignada por el sistema.

### <a name="assign-permissions"></a>Asignación de permisos

Debe asignar algunos permisos a través de RBAC al almacén (representado por el MSI del almacén) y la cuenta de almacenamiento correspondiente. Para ello se puede usar el portal o PowerShell. Obtenga más información sobre todos los [permisos relacionados](blob-backup-configure-manage.md#grant-permissions-to-the-backup-vault-on-storage-accounts).

### <a name="prepare-the-request"></a>Preparación de la solicitud

Una vez que se han establecido todos los permisos pertinentes, la configuración de la copia de seguridad se realiza en dos pasos. En primer lugar, se prepara la solicitud correspondiente mediante el almacén, la directiva y la cuenta de almacenamiento pertinentes con el comando [Initialize-AzDataProtectionBackupInstance](/powershell/module/az.dataprotection/initialize-azdataprotectionbackupinstance?view=azps-5.9.0&preserve-view=true). A continuación, se envía la solicitud para proteger los blobs con el comando [New-AzDataProtectionBackupInstance](/powershell/module/az.dataprotection/new-azdataprotectionbackupinstance?view=azps-5.9.0&preserve-view=true).

```azurepowershell-interactive
$instance = Initialize-AzDataProtectionBackupInstance -DatasourceType AzureBlob -DatasourceLocation $TestBkpvault.Location -PolicyId $blobBkpPol[0].Id -DatasourceId $SAId 
New-AzDataProtectionBackupInstance -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name -BackupInstance $instance

Name                                                       Type                                                  BackupInstanceName
----                                                       ----                                                  ------------------
blobrg-PSTestSA-3df6ac08-9496-4839-8fb5-8b78e594f166 Microsoft.DataProtection/backupVaults/backupInstances blobrg-PSTestSA-3df6ac08-9496-4839-8fb5-8b78e594f166
```

> [!IMPORTANT]
> Una vez configurada una cuenta de almacenamiento para la copia de seguridad de blobs, algunas funcionalidades se ven afectadas, como la fuente de cambios y el bloqueo de eliminación. [Más información](blob-backup-configure-manage.md#effects-on-backed-up-storage-accounts).

## <a name="next-steps"></a>Pasos siguientes

[Restauración de blobs de Azure mediante Azure PowerShell](restore-blobs-storage-account-ps.md)
