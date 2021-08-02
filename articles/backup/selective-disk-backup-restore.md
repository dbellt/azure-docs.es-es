---
title: Copia de seguridad y restauración selectivas de discos para máquinas virtuales de Azure
description: En este artículo, se describen la copia de seguridad y la restauración selectivas de discos mediante la solución de copia de seguridad de máquinas virtuales de Azure.
ms.topic: conceptual
ms.date: 05/13/2021
ms.custom: references_regions , devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: cee95941aa091f77fe128457434a66398188a0a4
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110678208"
---
# <a name="selective-disk-backup-and-restore-for-azure-virtual-machines"></a>Copia de seguridad y restauración selectivas de discos para máquinas virtuales de Azure

Azure Backup admite la copia de seguridad de todos los discos (sistema operativo y datos) en una máquina virtual junto con la solución de copia de seguridad de máquinas virtuales. Ahora, con la funcionalidad de copia de seguridad y restauración selectivas de discos, puede realizar una copia de seguridad de un subconjunto de los discos de datos de una máquina virtual. Esto proporciona una solución eficaz y rentable para sus necesidades de copia de seguridad y restauración. Cada punto de recuperación contiene solo los discos que se incluyen en la operación de copia de seguridad. Esto le permite además tener un subconjunto de discos restaurados desde el punto de recuperación determinado durante la operación de restauración. Esto se aplica a la restauración desde instantáneas y desde el almacén.

## <a name="scenarios"></a>Escenarios

Esta solución es especialmente útil en los escenarios siguientes:

1. Si tiene datos críticos de los que se va a realizar una copia de seguridad en un solo disco o un subconjunto de discos, y no desea realizar copias de seguridad del resto de los discos conectados a una máquina virtual para minimizar los costos de almacenamiento de copia de seguridad.
2. Si tiene otras soluciones de copia de seguridad para parte de la máquina virtual o los datos. Por ejemplo, si realiza una copia de seguridad de las bases de datos o los datos con una solución de copia de seguridad de carga de trabajo diferente y desea usar la copia de seguridad en el nivel de máquina virtual de Azure para el resto de los discos o datos para crear un sistema eficaz y sólido con las mejores funcionalidades disponibles.

Con PowerShell o la CLI de Azure, puede configurar la copia de seguridad selectiva de discos de la máquina virtual de Azure. Mediante un script, puede incluir o excluir discos de datos con sus números LUN. Actualmente, la capacidad de configurar la copia de seguridad selectiva de discos mediante Azure Portal está limitada a la opción **Backup OS Disk only** (Copia de seguridad solo de disco de SO). Por lo tanto, puede configurar la copia de seguridad de la máquina virtual de Azure con el disco del sistema operativo y excluir todos los discos de datos que tiene conectados.

>[!NOTE]
> El disco del sistema operativo se agrega de forma predeterminada a la copia de seguridad de la máquina virtual y no se puede excluir.

## <a name="using-azure-cli"></a>Uso de la CLI de Azure

Asegúrese de usar la CLI de Azure, versión 2.0.80 o posterior. Puede comprobar la versión de la CLI con el comando siguiente:

```azurecli
az --version
```

Inicie sesión con el identificador de la suscripción en la que se encuentran el almacén de Recovery Services y la máquina virtual:

```azurecli
az account set -s {subscriptionID}
```

>[!NOTE]
>Solo se necesita el nombre **resourcegroup** (no el objeto) correspondiente al almacén en cada uno de los comandos siguientes.

### <a name="configure-backup-with-azure-cli"></a>Configuración de la copia de seguridad con la CLI de Azure

Durante la operación de configuración de la protección, debe especificar la configuración de la lista de discos con un parámetro de **inclusión** / **exclusión**, proporcionando los números LUN de los discos que se van a incluir o excluir en la copia de seguridad.

>[!NOTE]
>La operación de configuración de protección invalida la configuración anterior; no será acumulativa.

```azurecli
az backup protection enable-for-vm --resource-group {resourcegroup} --vault-name {vaultname} --vm {vmname} --policy-name {policyname} --disk-list-setting include --diskslist {LUN number(s) separated by space}
```

```azurecli
az backup protection enable-for-vm --resource-group {resourcegroup} --vault-name {vaultname} --vm {vmname} --policy-name {policyname} --disk-list-setting exclude --diskslist 0 1
```

Si la máquina virtual no está en el mismo grupo de recursos que el almacén, **ResourceGroup** hará referencia al grupo de recursos donde se creó el almacén. En lugar del nombre de la máquina virtual, proporcione el identificador de la máquina virtual como se indica a continuación.

```azurecli
az backup protection enable-for-vm  --resource-group {ResourceGroup} --vault-name {vaultname} --vm $(az vm show -g VMResourceGroup -n MyVm --query id --output tsv) --policy-name {policyname} --disk-list-setting include --diskslist {LUN number(s) separated by space}
```

### <a name="modify-protection-for-already-backed-up-vms-with-azure-cli"></a>Modificación de la protección de las máquinas virtuales de las que ya ha realizado una copia de seguridad con la CLI de Azure

```azurecli
az backup protection update-for-vm --resource-group {resourcegroup} --vault-name {vaultname} -c {vmname} -i {vmname} --backup-management-type AzureIaasVM --disk-list-setting exclude --diskslist {LUN number(s) separated by space}
```

### <a name="backup-only-os-disk-during-configure-backup-with-azure-cli"></a>Copia de seguridad solo del disco del sistema operativo durante la configuración de la copia de seguridad con la CLI de Azure

```azurecli
az backup protection enable-for-vm --resource-group {resourcegroup} --vault-name {vaultname} --vm {vmname} --policy-name {policyname} --exclude-all-data-disks
```

### <a name="backup-only-os-disk-during-modify-protection-with-azure-cli"></a>Copia de seguridad solo del disco del sistema operativo durante la modificación de la protección con la CLI de Azure

```azurecli
az backup protection update-for-vm --resource-group {resourcegroup} --vault-name {vaultname} -c {vmname} -i {vmname} --backup-management-type AzureIaasVM --exclude-all-data-disks
```

### <a name="restore-disks-with-azure-cli"></a>Restauración de discos con la CLI de Azure

```azurecli
az backup restore restore-disks --resource-group {resourcegroup} --vault-name {vaultname} -c {vmname} -i {vmname} -r {restorepoint} --target-resource-group {targetresourcegroup} --storage-account {storageaccountname} --diskslist {LUN number of the disk(s) to be restored}
```

### <a name="restore-only-os-disk-with-azure-cli"></a>Restauración solo del disco del sistema operativo con la CLI de Azure

```azurecli
az backup restore restore-disks --resource-group {resourcegroup} --vault-name {vaultname} -c {vmname} -i {vmname} -r {restorepoint} } --target-resource-group {targetresourcegroup} --storage-account {storageaccountname} --restore-only-osdisk
```

### <a name="get-protected-item-to-get-disk-exclusion-details-with-azure-cli"></a>Obtención de un elemento protegido para obtener detalles de exclusión de disco con la CLI de Azure

```azurecli
az backup item show -c {vmname} -n {vmname} --vault-name {vaultname} --resource-group {resourcegroup} --backup-management-type AzureIaasVM
```

Hay un parámetro **diskExclusionProperties** adicional que se agrega al elemento protegido, como se muestra a continuación:

```azurecli
"extendedProperties": {
      "diskExclusionProperties": {
        "diskLunList": [
          0,
          1
        ],
        "isInclusionList": true
      }
```

### <a name="get-backup-job-with-azure-cli"></a>Obtención del trabajo de copia de seguridad con la CLI de Azure

```azurecli
az backup job show --vault-name {vaultname} --resource-group {resourcegroup} -n {BackupJobID}
```

Este comando ayuda a obtener los detalles de los discos con copia de seguridad y los discos excluidos, como se muestra a continuación:

```output
   "Backed-up disk(s)": "diskextest_OsDisk_1_170808a95d214428bad92efeecae626b; diskextest_DataDisk_0; diskextest_DataDisk_1",  "Backup Size": "0 MB",
   "Excluded disk(s)": "diskextest_DataDisk_2",
```

_BackupJobID_ es el nombre del trabajo de copia de seguridad. Para capturar el nombre del trabajo, ejecute el siguiente comando:

```azurecli
az backup job list --resource-group {resourcegroup} --vault-name {vaultname}
```
### <a name="list-recovery-points-with-azure-cli"></a>Enumeración de los puntos de recuperación con la CLI de Azure

```azurecli
az backup recoverypoint list --vault-name {vaultname} --resource-group {resourcegroup} -c {vmname} -i {vmname} --backup-management-type AzureIaasVM
```

Esto proporciona la información del número de discos conectados y de los que se ha realizado una copia de seguridad en la máquina virtual.

```azurecli
      "recoveryPointDiskConfiguration": {
        "excludedDiskList": null,
        "includedDiskList": null,
        "numberOfDisksAttachedToVm": 4,
        "numberOfDisksIncludedInBackup": 3
};
```

### <a name="get-recovery-point-with-azure-cli"></a>Obtención del punto de recuperación con la CLI de Azure

```azurecli
az backup recoverypoint show --vault-name {vaultname} --resource-group {resourcegroup} -c {vmname} -i {vmname} --backup-management-type AzureIaasVM -n {recoverypointID}
```

Cada punto de recuperación tiene la información de los discos incluidos y excluidos:

```azurecli
  "recoveryPointDiskConfiguration": {
      "excludedDiskList": [
        {
          "lun": 2,
          "name": "diskextest_DataDisk_2"
        }
      ],
      "includedDiskList": [
        {
          "lun": -1,
          "name": "diskextest_OsDisk_1_170808a95d214428bad92efeecae626b"
        },
        {
          "lun": 0,
          "name": "diskextest_DataDisk_0"
        },
        {
          "lun": 1,
          "name": "diskextest_DataDisk_1"
        }
      ],
      "numberOfDisksAttachedToVm": 4,
      "numberOfDisksIncludedInBackup": 3
```

### <a name="remove-disk-exclusion-settings-and-get-protected-item-with-azure-cli"></a>Eliminación de la configuración de exclusión de discos y obtención del elemento protegido con la CLI de Azure

```azurecli
az backup protection update-for-vm --vault-name {vaultname} --resource-group {resourcegroup} -c {vmname} -i {vmname} --backup-management-type AzureIaasVM --disk-list-setting resetexclusionsettings

az backup item show -c {vmname} -n {vmname} --vault-name {vaultname} --resource-group {resourcegroup} --backup-management-type AzureIaasVM
```

Cuando ejecute estos comandos, verá `"diskExclusionProperties": null`.

## <a name="using-powershell"></a>Usar PowerShell

Asegúrese de usar Azure PowerShell versión 3.7.0 o posterior.

Durante la operación de configuración de la protección, debe especificar la configuración de la lista de discos con un parámetro de inclusión/exclusión, proporcionando los números LUN de los discos que se van a incluir o excluir en la copia de seguridad.

>[!NOTE]
>La operación de configuración de protección invalida la configuración anterior; no será acumulativa.

### <a name="enable-backup-with-powershell"></a>Habilitación de la copia de seguridad con PowerShell

Por ejemplo:

```azurepowershell
$disks = ("0","1")
$targetVault = Get-AzRecoveryServicesVault -ResourceGroupName "rg-p-recovery_vaults" -Name "rsv-p-servers"
Get-AzRecoveryServicesBackupProtectionPolicy
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "P-Servers"
```

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"  -InclusionDisksList $disks -VaultId $targetVault.ID
```

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"  -ExclusionDisksList $disks -VaultId $targetVault.ID
```

### <a name="backup-only-os-disk-during-configure-backup-with-powershell"></a>Copia de seguridad solo del disco del sistema operativo durante la configuración de la copia de seguridad con PowerShell

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"  -ExcludeAllDataDisks -VaultId $targetVault.ID
```

### <a name="get-backup-item-object-to-be-passed-in-modify-protection-with-powershell"></a>Obtención del objeto del elemento de copia de seguridad que se va a pasar al modificar la protección con PowerShell

```azurepowershell
$item= Get-AzRecoveryServicesBackupItem -BackupManagementType "AzureVM" -WorkloadType "AzureVM" -VaultId $targetVault.ID -FriendlyName "V2VM"
```

Se debe pasar el objeto obtenido **$item** anterior al parámetro **–Item** en los siguientes cmdlets.

### <a name="modify-protection-for-already-backed-up-vms-with-powershell"></a>Modificación de la protección de las máquinas virtuales de las que ya ha realizado una copia de seguridad con PowerShell

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Item $item -InclusionDisksList[Strings] -VaultId $targetVault.ID
```

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Item $item -ExclusionDisksList[Strings] -VaultId $targetVault.ID
```

### <a name="backup-only-os-disk-during-modify-protection-with-powershell"></a>Copia de seguridad solo del disco del sistema operativo durante la modificación de la protección con PowerShell

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Item $item  -ExcludeAllDataDisks -VaultId $targetVault.ID
```

### <a name="reset-disk-exclusion-setting-with-powershell"></a>Restablecimiento de la configuración de exclusión de discos con PowerShell

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Item $item -ResetExclusionSettings -VaultId $targetVault.ID
```

### <a name="restore-selective-disks-with-powershell"></a>Restauración selectiva de discos con PowerShell

```azurepowershell
$startDate = (Get-Date).AddDays(-7)
$endDate = Get-Date
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -Item $item -StartDate $startdate.ToUniversalTime() -EndDate $enddate.ToUniversalTime() -VaultId $targetVault.ID
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName "DestAccount" -StorageAccountResourceGroupName "DestRG" -TargetResourceGroupName "DestRGforManagedDisks" -VaultId $targetVault.ID -RestoreDiskList [$disks]
```

### <a name="restore-only-os-disk-with-powershell"></a>Restauración solo del disco del sistema operativo con PowerShell

```azurepowershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName "DestAccount" -StorageAccountResourceGroupName "DestRG" -TargetResourceGroupName "DestRGforManagedDisks" -VaultId $targetVault.ID -RestoreOnlyOSDisk
```

## <a name="using-the-azure-portal"></a>Uso de Azure Portal

[!INCLUDE [backup-center.md](../../includes/backup-center.md)]

Con Azure Portal, puede ver los discos incluidos y excluidos en el panel de detalles de copia de seguridad de la máquina virtual y en el panel de detalles del trabajo de copia de seguridad.  Durante la restauración, al seleccionar el punto de recuperación desde el que se realiza la restauración, puede ver los discos de copia de seguridad en ese punto de recuperación.

Aquí puede ver los discos incluidos y excluidos para una máquina virtual en el portal del panel de detalles de copia de seguridad de la máquina virtual:

![Visualización de discos incluidos y excluidos del panel de detalles de copia de seguridad](./media/selective-disk-backup-restore/backup-details.png)

Aquí puede ver los discos incluidos y excluidos en una copia de seguridad desde el panel de detalles del trabajo:

![Visualización de discos incluidos y excluidos desde el panel de detalles del trabajo](./media/selective-disk-backup-restore/job-details.png)

Aquí puede ver los discos con copia de seguridad durante la restauración al seleccionar el punto de recuperación desde el que restaurar:

![Visualización de discos con copia de seguridad durante la restauración](./media/selective-disk-backup-restore/during-restore.png)

La configuración de la copia de seguridad selectiva de discos para una máquina virtual mediante Azure Portal está limitada a la opción **Backup OS Disk only** (Copia de seguridad solo de disco de SO). Para usar la copia de seguridad selectiva de discos en una máquina virtual de la que ya se ha realizado una copia de seguridad o para la inclusión o exclusión avanzadas de discos de datos específicos de una máquina virtual, use PowerShell o la CLI de Azure.

>[!NOTE]
>Si los datos abarcan varios discos, asegúrese de que todos los discos dependientes se incluyan en la copia de seguridad. Si no realiza una copia de seguridad de todos los discos dependientes de un volumen, durante la restauración no se creará el volumen que contiene algunos discos de los que no se ha realizado una copia de seguridad.

### <a name="backup-os-disk-only-in-the-azure-portal"></a>Copia de seguridad solo del disco del sistema operativo en Azure Portal

Al habilitar la copia de seguridad mediante Azure Portal, puede elegir la opción **Backup OS Disk only** (Copia de seguridad solo de disco de SO). Por lo tanto, puede configurar la copia de seguridad de la máquina virtual de Azure con el disco del sistema operativo y excluir todos los discos de datos que tiene conectados.

![Configuración de la copia de seguridad solo para el disco del sistema operativo](./media/selective-disk-backup-restore/configure-backup-operating-system-disk.png)

## <a name="using-azure-rest-api"></a>Uso de la API REST de Azure

Puede configurar el servicio Backup para máquinas virtuales de Azure con unos cuantos discos seleccionados o puede modificar la protección de una máquina virtual existente para incluir o excluir unos pocos discos, como se documenta [aquí](backup-azure-arm-userestapi-backupazurevms.md#excluding-disks-in-azure-vm-backup).

## <a name="selective-disk-restore"></a>Restauración selectiva de discos

La restauración selectiva de discos es una funcionalidad adicional que se obtiene al habilitar la característica de copia de seguridad selectiva de discos. Con esta funcionalidad, puede restaurar discos concretos de entre todos los discos de los que se ha realizado una copia de seguridad en un punto de recuperación. Es más eficaz y ayuda a ahorrar tiempo en escenarios en los que sabe cuáles de los discos deben restaurarse.

- El disco del sistema operativo se incluye de forma predeterminada en la copia de seguridad y restauración de la máquina virtual y no se puede excluir.
- La restauración selectiva de discos solo se admite para los puntos de recuperación creados después de habilitar la funcionalidad de exclusión de discos.
- Las copias de seguridad con la configuración de exclusión de discos **activada** solo admiten la opción **Disk restore** (Restauración de disco). Las opciones de restauración **VM restore** (Restauración de máquina virtual) o **Replace Existing** (Reemplazar existente) no se admiten en este caso.

![La opción para restaurar la máquina virtual y reemplazar la existente no está disponible durante la operación de restauración.](./media/selective-disk-backup-restore/options-not-available.png)

## <a name="limitations"></a>Limitaciones

La funcionalidad de copia de seguridad selectiva de discos no se admite para máquinas virtuales clásicas ni máquinas virtuales cifradas. Por lo tanto, las máquinas virtuales de Azure cifradas con Azure Disk Encryption (ADE) que usan BitLocker para el cifrado de máquinas virtuales Windows y la característica dm-crypt para máquinas virtuales Linux no se admiten.

Las opciones de restauración **Create new VM** (Crear nueva máquina virtual) y **Replace existing** (Reemplazar existente) no se admiten para la máquina virtual para la que está habilitada la funcionalidad de copia de seguridad selectiva de discos.

Actualmente, la copia de seguridad de VM de Azure no admite VM con discos Ultra o discos compartidos conectados. No se puede usar la copia de seguridad de disco selectiva en estos casos, ya que se excluye el disco y la copia de seguridad de la máquina virtual.

Si usa la exclusión de disco o discos selectivos durante la copia de seguridad de máquinas virtuales de Azure, _[detenga la protección y conserve los datos de copia de seguridad](backup-azure-manage-vms.md#stop-protection-and-retain-backup-data)_ . Al reanudar la copia de seguridad de este recurso, debe volver a configurar la configuración de exclusión de disco.

## <a name="billing"></a>Facturación

La copia de seguridad de máquinas virtuales de Azure sigue el modelo de precios existente, que se explica en detalle [aquí](https://azure.microsoft.com/pricing/details/backup/).

El **costo de instancia protegida (PI)** se calcula para el disco del sistema operativo solo si elige realizar una copia de seguridad con la opción **Solo el disco del SO**.  Si configura la copia de seguridad y selecciona al menos un disco de datos, el costo de PI se calculará para todos los discos conectados a la máquina virtual. El **costo del almacenamiento de copia de seguridad** se calcula solo en función de los discos incluidos, lo que supone un ahorro. El **costo de las instantáneas** siempre se calcula para todos los discos de la máquina virtual (tanto los discos incluidos como los excluidos).

Si ha elegido la característica de restauración entre regiones (CRR), los [precios de CRR](https://azure.microsoft.com/pricing/details/backup/) se aplican al costo de almacenamiento de copia de seguridad después de excluir el disco.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="how-is-protected-instance-pi-cost-calculated-for-only-os-disk-backup-in-windows-and-linux"></a>¿Cómo se calcula el costo de instancia protegida (PI) solo para copias de seguridad de disco del sistema operativo en Windows y Linux?

El costo de PI se calcula en función del tamaño real (usado) de la máquina virtual.

- Para Windows: el cálculo del espacio utilizado se basa en la unidad que almacena el sistema operativo (que suele ser C:).
- Para Linux: el cálculo del espacio utilizado se basa en el dispositivo donde se monta el sistema de archivos raíz (/).

### <a name="i-have-configured-only-os-disk-backup-why-is-the-snapshot-happening-for-all-the-disks"></a>He configurado solo la copia de seguridad del disco del sistema operativo, ¿por qué se realiza la instantánea de todos los discos?

Las características de copia de seguridad de discos selectiva permiten ahorrar en el costo de almacenamiento del almacén de copia de seguridad al proteger los discos incluidos que forman parte de la copia de seguridad. Sin embargo, se realiza la instantánea de todos los discos conectados a la máquina virtual. Por tanto, el costo de las instantáneas siempre se calcula para todos los discos de la máquina virtual (tanto los discos incluidos como los excluidos). Para obtener más información, vea [Facturación](#billing).

### <a name="i-cant-configure-backup-for-the-azure-virtual-machine-by-excluding-ultra-disk-or-shared-disks-attached-to-the-vm"></a>No puedo configurar la copia de seguridad de la máquina virtual de Azure excluyendo el disco Ultra o los discos compartidos conectados a la máquina virtual

La característica de copia de seguridad de discos selectiva es una funcionalidad que se proporciona en la solución de copia de seguridad de máquinas virtuales de Azure. Actualmente, la copia de seguridad de VM de Azure no admite VM con discos Ultra o discos compartidos conectados.

## <a name="next-steps"></a>Pasos siguientes

- [Matriz de compatibilidad para copias de seguridad de máquinas virtuales de Azure](backup-support-matrix-iaas.md)
- [Preguntas más frecuentes sobre la copia de seguridad de máquinas virtuales de Azure](backup-azure-vm-backup-faq.yml)
