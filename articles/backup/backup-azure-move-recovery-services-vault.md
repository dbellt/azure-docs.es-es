---
title: Cómo mover los almacenes de Recovery Services de Azure Backup
description: Instrucciones sobre cómo mover el almacén de Recovery Services entre suscripciones y grupos de recursos de Azure.
ms.topic: conceptual
ms.date: 04/08/2019
ms.custom: references_regions
ms.openlocfilehash: 239294f1744da9f040ac62435e6930b60f6bc357
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111954367"
---
# <a name="move-a-recovery-services-vault-across-azure-subscriptions-and-resource-groups"></a>Traslado del almacén de Recovery Services entre suscripciones y grupos de recursos de Azure

En este artículo se explica cómo mover un almacén de Recovery Services configurado para Azure Backup entre suscripciones de Azure, o a otro grupo de recursos en la misma suscripción. Puede usar Azure Portal o PowerShell para mover un almacén de Recovery Services.

## <a name="supported-regions"></a>Regiones admitidas

Se admiten todas las regiones públicas y soberanas, excepto el Centro de Francia, Sur de Francia, Nordeste de Alemania, Centro de Alemania, Norte de China, Norte de China 2, Este de China y Este de China 2.

## <a name="prerequisites-for-moving-recovery-services-vault"></a>Requisitos previos para el traslado al almacén de Recovery Services

- Durante el traslado de almacén entre grupos de recursos, los grupos de recursos de origen y destino están bloqueados para evitar las operaciones de escritura y eliminación. Para más información, consulte este [artículo](../azure-resource-manager/management/move-resource-group-and-subscription.md).
- Solo la suscripción del administrador tiene los permisos para mover un almacén.
- Para trasladar almacenes entre suscripciones, la suscripción de destino debe existir en el mismo inquilino que la suscripción de origen y su estado debe estar habilitado. Para trasladar un almacén a un directorio de Azure AD diferente, consulte [Transferencia de una suscripción a otro directorio](../role-based-access-control/transfer-subscription.md) y [Preguntas frecuentes del almacén de Recovery Services](/azure/backup/backup-azure-backup-faq.yml#recovery-services-vault).
- Debe tener permiso para realizar operaciones de escritura en el grupo de recursos de destino.
- El traslado del almacén solo cambia el grupo de recursos. El almacén de Recovery Services existirá en la misma ubicación y no se puede cambiar.
- Solo puede trasladar un almacén de Recovery Services por región cada vez.
- Si una máquina virtual no se traslada con el almacén de Recovery Services entre suscripciones, o a un nuevo grupo de recursos, los puntos de recuperación de la máquina virtual actuales permanecerán intactos en el almacén hasta que expiren.
- Independientemente de que la máquina virtual se mueva con el almacén o no, siempre podrá restaurar la máquina virtual desde el historial de copia de seguridad conservado en el almacén.
- Azure Disk Encryption requiere que el almacén de claves y las máquinas virtuales residan en la misma región y suscripción de Azure.
- Para mover una máquina virtual con discos administrados, consulte este [artículo](../azure-resource-manager/management/move-resource-group-and-subscription.md).
- Las opciones para mover recursos implementados mediante el modelo clásico varían en función de si traslada los recursos dentro de una misma suscripción o a una nueva suscripción. Para más información, consulte este [artículo](../azure-resource-manager/management/move-resource-group-and-subscription.md).
- Las directivas de copia de seguridad definidas para el almacén se conservan después de que el almacén se mueva entre suscripciones o a un nuevo grupo de recursos.
- Solo puede trasladar un almacén que contenga cualquiera de los siguientes tipos de elementos de copia de seguridad. Los elementos de copia de seguridad de tipos que no se enumeran a continuación deberán detenerse y los datos deberán eliminarse de forma permanente antes de mover el almacén.
  - Azure Virtual Machines
  - Agente de Microsoft Azure Recovery Services (MARS)
  - Microsoft Azure Backup Server (MABS)
  - Data Protection Manager (DPM)
- Si traslada un almacén con datos de copia de seguridad de VM entre suscripciones, deberá trasladar sus VM a la misma suscripción y usar el mismo nombre de grupo de recursos de VM de destino (tal como estaba en la suscripción anterior) para continuar con las copias de seguridad.

> [!NOTE]
> No se admite la migración de almacenes de Recovery Services para Azure Backup entre regiones de Azure.<br><br>
> Si ha configurado alguna máquina virtual (IaaS de Azure, Hyper-V, VMware) o máquina física para la recuperación ante desastres mediante **Azure Site Recovery**, se bloquea la operación de traslado. Si desea mover almacenes para Azure Site Recovery, consulte [este artículo](../site-recovery/move-vaults-across-regions.md) para más información sobre cómo mover los almacenes manualmente.

## <a name="use-azure-portal-to-move-recovery-services-vault-to-different-resource-group"></a>Uso de Azure Portal para trasladar un almacén de Recovery Services a otro grupo de recursos

Para mover un almacén de Recovery Services y los recursos asociados a un grupo de recursos diferente:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Abra la lista de **almacenes de Recovery Services** y seleccione el nombre del almacén que desea mover. Cuando se abre el panel del almacén, aparece como se muestra en la siguiente imagen.

   ![Abrir el almacén de Recovery Services](./media/backup-azure-move-recovery-services/open-recover-service-vault.png)

   Si no ve la **Información esencial** del almacén, seleccione el icono de la lista desplegable. Ahora debería ver la información esencial del almacén.

   ![Pestaña Información esencial](./media/backup-azure-move-recovery-services/essentials-information-tab.png)

3. En el menú de información general del almacén, seleccione **Cambiar** junto a **Grupo de recursos** para abrir el panel **Mover recursos**.

   ![Cambiar el grupo de recursos](./media/backup-azure-move-recovery-services/change-resource-group.png)

4. En el panel **Mover recursos**, para el almacén seleccionado, se recomienda mover los recursos relacionados opcionales activando la casilla de verificación, tal como se muestra en la siguiente imagen.

   ![Mover suscripción](./media/backup-azure-move-recovery-services/move-resource.png)

5. Para agregar el grupo de recursos de destino, en la lista desplegable **Grupo de recursos**, seleccione un grupo de recursos existente o la opción **Crear un grupo nuevo**.

   ![Crear recurso](./media/backup-azure-move-recovery-services/create-a-new-resource.png)

6. Después de agregar el grupo de recursos, confirme la opción **Comprendo que las herramientas y los scripts asociados con recursos movidos no funcionarán hasta que los actualice para que usen nuevos identificadores de recursos** y, a continuación, seleccione **Aceptar** para completar el traslado del almacén.

   ![Mensaje de confirmación](./media/backup-azure-move-recovery-services/confirmation-message.png)

## <a name="use-azure-portal-to-move-recovery-services-vault-to-a-different-subscription"></a>Uso de Azure Portal para trasladar un almacén de Recovery Services a otra suscripción

Puede mover un almacén de Recovery Services y sus recursos asociados a otra suscripción

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Abra la lista de almacenes de Recovery Services y seleccione el almacén que desea mover. Cuando se abre el panel del almacén, aparece como se muestra en la siguiente imagen.

    ![Abrir el almacén de Recovery Services](./media/backup-azure-move-recovery-services/open-recover-service-vault.png)

    Si no ve la **Información esencial** del almacén, seleccione el icono de la lista desplegable. Ahora debería ver la información esencial del almacén.

    ![Pestaña Información esencial](./media/backup-azure-move-recovery-services/essentials-information-tab.png)

3. En el menú de información general del almacén, seleccione **Cambiar** junto a **Suscripción** para abrir el panel **Mover recursos**.

   ![Cambiar suscripción](./media/backup-azure-move-recovery-services/change-resource-subscription.png)

4. Seleccione los recursos que se van a mover; para ello, se recomienda usar la opción **Seleccionar todo** para seleccionar todos los recursos enumerados opcionales.

   ![Mover recurso](./media/backup-azure-move-recovery-services/move-resource-source-subscription.png)

5. Seleccione la suscripción de destino desde la lista desplegable **Suscripción** a donde desea que se mueva el almacén.
6. Para agregar el grupo de recursos de destino, en la lista desplegable **Grupo de recursos**, seleccione un grupo de recursos existente o la opción **Crear un grupo nuevo**.

   ![Agregar suscripción](./media/backup-azure-move-recovery-services/add-subscription.png)

7. Seleccione la opción **Comprendo que las herramientas y los scripts asociados con recursos movidos no funcionarán hasta que los actualice para que usen nuevos identificadores de recursos** para confirmar y, a continuación, **Aceptar**.

> [!NOTE]
> La copia de seguridad entre suscripciones (el almacén de RS y las máquinas virtuales protegidas están en distintas suscripciones) no es un escenario admitido. Además, durante la operación de movimiento del almacén, no es posible modificar la opción de redundancia de almacenamiento desde el almacenamiento con redundancia local (LRS) al almacenamiento con redundancia global (GRS) ni viceversa
>
>

## <a name="use-powershell-to-move-recovery-services-vault"></a>Uso de PowerShell para trasladar un almacén de Recovery Services

Para mover un almacén de Recovery Services a otro grupo de recursos, use el cmdlet `Move-AzureRMResource`. `Move-AzureRMResource` requiere el nombre del recurso y el tipo de recurso. Puede obtener ambos con el cmdlet `Get-AzureRmRecoveryServicesVault`.

```powershell
$destinationRG = "<destinationResourceGroupName>"
$vault = Get-AzureRmRecoveryServicesVault -Name <vaultname> -ResourceGroupName <vaultRGname>
Move-AzureRmResource -DestinationResourceGroupName $destinationRG -ResourceId $vault.ID
```

Para mover los recursos a otra suscripción, incluya el parámetro `-DestinationSubscriptionId`.

```powershell
Move-AzureRmResource -DestinationSubscriptionId "<destinationSubscriptionID>" -DestinationResourceGroupName $destinationRG -ResourceId $vault.ID
```

Después de ejecutar los cmdlets anteriores, se le pedirá que confirme que desea mover los recursos especificados. Escriba **Y** para continuar. Cuando se haya validado correctamente, el recurso se moverá.

## <a name="use-cli-to-move-recovery-services-vault"></a>Uso de la CLI para trasladar un almacén de Recovery Services

Para mover un almacén de Recovery Services a otro grupo de recursos, use el siguiente cmdlet:

```azurecli
az resource move --destination-group <destinationResourceGroupName> --ids <VaultResourceID>
```

Para mover a una nueva suscripción, proporcione el parámetro `--destination-subscription-id`.

## <a name="post-migration"></a>Después de la migración

1. Establezca/verifique los controles de acceso para los grupos de recursos.  
2. La característica de supervisión y creación de informes de Backup debe volver a configurarse para el almacén una vez finalizada la operación de movimiento. La configuración anterior se perderá durante la esta operación.

## <a name="move-an-azure-virtual-machine-to-a-different-recovery-service-vault"></a>Mueva una máquina virtual de Azure a otro almacén de Recovery Services. 

Si quiere trasladar una máquina virtual de Azure que tiene habilitado Azure Backup, tiene dos opciones. Estas opciones dependen de sus requisitos empresariales:

- [No es necesario conservar los datos de copia de seguridad anteriores](#dont-need-to-preserve-previous-backed-up-data)
- [Se deben conservar los datos de copia de seguridad anteriores](#must-preserve-previous-backed-up-data)

### <a name="dont-need-to-preserve-previous-backed-up-data"></a>No es necesario conservar los datos de copia de seguridad anteriores

Para proteger las cargas de trabajo en un nuevo almacén, será necesario eliminar la protección actual y los datos del almacén antiguo, y volver a configurar la copia de seguridad.

>[!WARNING]
>La operación siguiente es destructiva y no se puede deshacer. Todos los datos de copia de seguridad y los elementos de copia de seguridad asociados con el servidor protegido se eliminarán de forma permanente. Proceda con precaución.

**Detenga y elimine la protección actual en el almacén antiguo:**

1. deshabilite la eliminación temporal en las propiedades del almacén. Siga [estos pasos](backup-azure-security-feature-cloud.md#disabling-soft-delete-using-azure-portal) para deshabilitar la eliminación temporal.

2. Detenga la protección y elimine las copias de seguridad del almacén actual. En el menú del panel del almacén, seleccione **Elementos de copia de seguridad**. Los elementos que aparecen aquí y que deben moverse al nuevo almacén se deben quitar junto con sus datos de copia de seguridad. Vea cómo [eliminar elementos protegidos en la nube](backup-azure-delete-vault.md#delete-protected-items-in-the-cloud) y [eliminar elementos protegidos en el entorno local](backup-azure-delete-vault.md#delete-protected-items-on-premises).

3. Si tiene previsto mover AFS (recursos compartidos de archivos de Azure), servidores SQL o servidores SAP HANA, también tendrá que anular su registro. En el menú del panel del almacén, seleccione **Infraestructura de Backup**. Vea cómo [anular el registro de SQL Server](manage-monitor-sql-database-backup.md#unregister-a-sql-server-instance), [anular el registro de una cuenta de almacenamiento asociada a recursos compartidos de archivos de Azure](manage-afs-backup.md#unregister-a-storage-account) y [anular el registro de una instancia de SAP HANA](sap-hana-db-manage.md#unregister-an-sap-hana-instance).

4. Una vez que se hayan quitado del almacén antiguo, continúe con la configuración de las copias de seguridad de la carga de trabajo en el nuevo almacén.

### <a name="must-preserve-previous-backed-up-data"></a>Se deben conservar los datos de copia de seguridad anteriores

Si tiene que conservar los datos protegidos actuales en el almacén antiguo y mantener la protección en un nuevo almacén, hay opciones limitadas para algunas de las cargas de trabajo:

- Para MARS, puede [detener la protección con conservación de datos](backup-azure-manage-mars.md#stop-protecting-files-and-folder-backup) y registrar el agente en el nuevo almacén.

  - El servicio Azure Backup seguirá conservando todos los puntos de recuperación existentes en el almacén antiguo.
  - Tendrá que pagar para conservar los puntos de recuperación en el almacén antiguo.
  - Solo podrá restaurar los datos de copia de seguridad de los puntos de recuperación que no hayan expirado en el almacén antiguo.
  - Tendrá que crear una réplica inicial de los datos en el nuevo almacén.

- En el caso de una VM de Azure, puede [detener la protección con conservación de datos](backup-azure-manage-vms.md#stop-protecting-a-vm) para la VM en el almacén antiguo, moverla a otro grupo de recursos y, después, protegerla en el nuevo almacén. Vea [Instrucciones y limitaciones](../azure-resource-manager/management/move-limitations/virtual-machines-move-limitations.md) para mover una máquina virtual a otro grupo de recursos.

  Una máquina virtual solo se puede proteger en un almacén a la vez. No obstante, la VM del nuevo grupo de recursos se puede proteger en el nuevo almacén, ya que se considera una VM distinta.

  - El servicio Azure Backup conservará los puntos de recuperación de los que se ha realizado una copia de seguridad en el almacén antiguo.
  - Deberá pagar para mantener los puntos de recuperación en el almacén antiguo (vea [Precios de Azure Backup](azure-backup-pricing.md) para obtener detalles).
  - Podrá restaurar la VM si es necesario, desde el almacén antiguo.
  - La primera copia de seguridad en el nuevo almacén de la VM en el nuevo recurso será una réplica inicial.

## <a name="next-steps"></a>Pasos siguientes

Puede mover muchos tipos diferentes de recursos entre suscripciones y grupos de recursos.

Para obtener más información, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../azure-resource-manager/management/move-resource-group-and-subscription.md).