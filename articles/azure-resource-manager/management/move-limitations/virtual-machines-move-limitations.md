---
title: Traslado de máquinas virtuales de Azure a una nueva suscripción o grupo de recursos
description: Use Azure Resource Manager para trasladar máquinas virtuales a un nuevo grupo de recursos o a una nueva suscripción.
ms.topic: conceptual
ms.date: 04/23/2021
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: f57b09900a955a3988c27f8f6990ff18db41c97c
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109751750"
---
# <a name="move-guidance-for-virtual-machines"></a>Guía del traslado de máquinas virtuales

En este artículo se describen los escenarios que actualmente no se admiten y los pasos para trasladar máquinas virtuales con copia de seguridad.

## <a name="scenarios-not-supported"></a>Escenarios no admitidos

Todavía no se admiten los siguientes escenarios:

* No es posible trasladar Virtual Machine Scale Sets con equilibrador de carga o IP pública de SKU estándar.
* Las máquinas virtuales de una red virtual existente no se pueden mover a una suscripción nueva si no se van a mover todos los recursos de la red virtual.
* Las máquinas virtuales creadas a partir de recursos de Marketplace con planes asociados no se pueden mover entre suscripciones. Para una posible solución alternativa, consulte [Máquinas virtuales con planes de marketplace](#virtual-machines-with-marketplace-plans).
* Las máquinas virtuales de prioridad baja y los conjuntos de escalado de máquinas virtuales de prioridad baja no pueden moverse entre grupos de recursos o suscripciones.
* Las máquinas virtuales de un conjunto de disponibilidad no se pueden mover individualmente.

## <a name="azure-disk-encryption"></a>Azure Disk Encryption

No puede mover una VM que esté integrada con un almacén de claves para implementar [Azure Disk Encryption para máquinas virtuales Linux](../../../virtual-machines/linux/disk-encryption-overview.md) o [Azure Disk Encryption para máquinas virtuales Windows](../../../virtual-machines/windows/disk-encryption-overview.md). Para mover la VM, debe deshabilitar el cifrado.

```azurecli-interactive
az vm encryption disable --resource-group demoRG --name myVm1
```

```azurepowershell-interactive
Disable-AzVMDiskEncryption -ResourceGroupName demoRG -VMName myVm1
```

## <a name="virtual-machines-with-marketplace-plans"></a>Virtual Machines con planes de marketplace

Las máquinas virtuales creadas a partir de recursos de Marketplace con planes asociados no se pueden mover entre suscripciones. Para solucionar esta limitación, puede desaprovisionar la máquina virtual en la suscripción actual y volver a implementarla en la nueva suscripción. Los siguientes pasos le ayudarán a volver a crear la máquina virtual en la nueva suscripción. Sin embargo, es posible que no funcionen en todos los escenarios. Si el plan ya no está disponible en el marketplace, estos pasos no funcionarán.

1. Copie la información sobre el plan.

1. Clone el disco del sistema operativo en la suscripción de destino, o bien mueva el disco original después de eliminar la máquina virtual de la suscripción de origen.

1. En la suscripción de destino, acepte los términos del marketplace para el plan. Para aceptar los términos, puede ejecutar el siguiente comando de PowerShell:

   ```azurepowershell
   Get-AzMarketplaceTerms -Publisher {publisher} -Product {product/offer} -Name {name/SKU} | Set-AzMarketplaceTerms -Accept
   ```

   O bien, puede crear una nueva instancia de una máquina virtual con el plan mediante el portal. Puede eliminar la máquina virtual después de aceptar los términos de la nueva suscripción.

1. En la suscripción de destino, vuelva a crear la máquina virtual del disco del sistema operativo clonado mediante PowerShell, la CLI o una plantilla de Azure Resource Manager. Incluya el plan de marketplace que está conectado al disco. La información sobre el plan debe coincidir con el plan que compró en la nueva suscripción.

## <a name="virtual-machines-with-azure-backup"></a>Máquinas virtuales con Azure Backup

Para trasladar las máquinas virtuales configuradas con Azure Backup, debe eliminar las colecciones de puntos de restauración (instantáneas) del almacén. Los puntos de restauración ya copiados en el almacén se pueden conservar y mover.

Si la [eliminación temporal](../../../backup/soft-delete-virtual-machines.md) está habilitada para la máquina virtual, no podrá moverla mientras se conserven esos puntos de restauración. [Deshabilite la eliminación temporal](../../../backup/backup-azure-security-feature-cloud.md#enabling-and-disabling-soft-delete) o espere 14 días después de eliminar los puntos de restauración.

### <a name="portal"></a>Portal

1. Detenga temporalmente la copia de seguridad y conserve los datos de esta.
2. Para mover máquinas virtuales configuradas con Azure Backup, siga los pasos a continuación:

   1. Busque la ubicación de la máquina virtual.
   2. Busque un grupo de recursos con el siguiente patrón de nomenclatura: `AzureBackupRG_<VM location>_1`. Por ejemplo, el nombre tiene el formato *AzureBackupRG_westus2_1*.
   3. En Azure Portal, active **Mostrar tipos ocultos**.
   4. Busque el recurso con el tipo **Microsoft.Compute/restorePointCollections** que tenga el patrón de nomenclatura `AzureBackup_<VM name>_###########`.
   5. Elimine este recurso. Esta operación elimina solo los puntos de recuperación instantáneos, no los datos de copia de seguridad que se encuentran en el almacén.
   6. Una vez completada la operación de eliminación, puede mover la máquina virtual.

3. Traslade la máquina virtual al grupo de recursos de destino.
4. Vuelva a configurar la copia de seguridad.

### <a name="powershell"></a>PowerShell

1. Busque la ubicación de la máquina virtual.

1. Busque un grupo de recursos con el patrón de nomenclatura `AzureBackupRG_<VM location>_1`. Por ejemplo, el nombre podría ser `AzureBackupRG_westus2_1`.

1. Si solo va a mover una máquina virtual, obtenga la colección de puntos de restauración de esa máquina virtual.

   ```azurepowershell-interactive
   $restorePointCollection = Get-AzResource -ResourceGroupName AzureBackupRG_<VM location>_1 -name AzureBackup_<VM name>* -ResourceType Microsoft.Compute/restorePointCollections
   ```

   Elimine este recurso. Esta operación elimina solo los puntos de recuperación instantáneos, no los datos de copia de seguridad que se encuentran en el almacén.

   ```azurepowershell-interactive
   Remove-AzResource -ResourceId $restorePointCollection.ResourceId -Force
   ```

1. Si va a mover todas las máquinas virtuales con copias de seguridad de esta ubicación, obtenga las colecciones de puntos de restauración de esas máquinas virtuales.

   ```azurepowershell-interactive
   $restorePointCollection = Get-AzResource -ResourceGroupName AzureBackupRG_<VM location>_1 -ResourceType Microsoft.Compute/restorePointCollections
   ```

   Elimine cada recurso. Esta operación elimina solo los puntos de recuperación instantáneos, no los datos de copia de seguridad que se encuentran en el almacén.

   ```azurepowershell-interactive
   foreach ($restorePoint in $restorePointCollection)
   {
     Remove-AzResource -ResourceId $restorePoint.ResourceId -Force
   }
   ```

### <a name="azure-cli"></a>Azure CLI

1. Busque la ubicación de la máquina virtual.

1. Busque un grupo de recursos con el patrón de nomenclatura `AzureBackupRG_<VM location>_1`. Por ejemplo, el nombre podría ser `AzureBackupRG_westus2_1`.

1. Si solo va a mover una máquina virtual, obtenga la colección de puntos de restauración de esa máquina virtual.

   ```azurecli-interactive
   RESTOREPOINTCOL=$(az resource list -g AzureBackupRG_<VM location>_1 --resource-type Microsoft.Compute/restorePointCollections --query "[?starts_with(name, 'AzureBackup_<VM name>')].id" --output tsv)
   ```

   Elimine este recurso. Esta operación elimina solo los puntos de recuperación instantáneos, no los datos de copia de seguridad que se encuentran en el almacén.

   ```azurecli-interactive
   az resource delete --ids $RESTOREPOINTCOL
   ```

1. Si va a mover todas las máquinas virtuales con copias de seguridad de esta ubicación, obtenga las colecciones de puntos de restauración de esas máquinas virtuales.

   ```azurecli-interactive
   RESTOREPOINTCOL=$(az resource list -g AzureBackupRG_<VM location>_1 --resource-type Microsoft.Compute/restorePointCollections)
   ```

   Elimine cada recurso. Esta operación elimina solo los puntos de recuperación instantáneos, no los datos de copia de seguridad que se encuentran en el almacén.

   ```azurecli-interactive
   az resource delete --ids $RESTOREPOINTCOL
   ```

## <a name="next-steps"></a>Pasos siguientes

* Para ver los comandos para trasladar recursos, vea [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../move-resource-group-and-subscription.md).

* Para más información sobre cómo mover los almacenes de Recovery Services para realizar copias de seguridad, consulte [Limitaciones de Recovery Services](../../../backup/backup-azure-move-recovery-services-vault.md?toc=/azure/azure-resource-manager/toc.json).
