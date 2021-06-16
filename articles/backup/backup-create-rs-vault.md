---
title: Creación y configuración de almacenes de Recovery Services
description: En este artículo, aprenderá a crear y configurar almacenes de Recovery Services que almacenan las copias de seguridad y los puntos de recuperación. Aprenda a usar Restauración entre regiones para restaurar en una región secundaria.
ms.topic: conceptual
ms.date: 06/01/2021
ms.custom: references_regions
ms.openlocfilehash: b301ce0ab2f4b57a03e3ba24d672bb4102a1a997
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110794228"
---
# <a name="create-and-configure-a-recovery-services-vault"></a>Creación y configuración de un almacén de Recovery Services

[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

## <a name="set-storage-redundancy"></a>Establecimiento de la redundancia de almacenamiento

Azure Backup administra automáticamente el almacenamiento para el almacén. Debe especificar cómo se replica ese almacenamiento.

> [!NOTE]
> El cambio del **tipo de replicación de almacenamiento** (con redundancia local o con redundancia geográfica) para un almacén de Recovery Services se debe realizar antes de configurar las copias de seguridad en el almacén. Una vez que configure la copia de seguridad, se deshabilita la opción para modificar.
>
>- Si todavía no ha configurado la copia de seguridad, [siga estos pasos](#set-storage-redundancy) para revisar y modificar la configuración.
>- Si ya ha configurado la copia de seguridad y debe pasar de GRS a LRS, [revise estas soluciones alternativas](#how-to-change-from-grs-to-lrs-after-configuring-backup).

1. En el panel **Almacenes de Recovery Services**, seleccione un almacén nuevo. En la sección **Configuración**, seleccione **Propiedades**.
1. En **Propiedades**, en **Configuración de copia de seguridad**, seleccione **Actualizar**.

1. Seleccione el tipo de replicación almacenamiento y seleccione **Guardar**.

     ![Establecimiento de la configuración de almacenamiento del nuevo almacén](./media/backup-create-rs-vault/recovery-services-vault-backup-configuration.png)

   - Se recomienda que, si usa Azure como punto de conexión del almacenamiento de copia de seguridad principal, siga utilizando la configuración **con redundancia geográfica** predeterminada.
   - Si no utiliza Azure como punto de conexión de almacenamiento de copia de seguridad principal, elija **Redundancia local** para reducir los costes de almacenamiento de Azure.
   - Obtenga más información sobre la redundancia [geográfica](../storage/common/storage-redundancy.md#geo-redundant-storage) y [local](../storage/common/storage-redundancy.md#locally-redundant-storage).
   - Si necesita disponibilidad de datos sin tiempo de inactividad en una región, garantizando la residencia de los datos, elija [almacenamiento con redundancia de zona](../storage/common/storage-redundancy.md#zone-redundant-storage).

>[!NOTE]
>Los valores de Replicación de almacenamiento para el almacén no son pertinentes para la copia de seguridad de recursos compartidos de archivos de Azure, ya que la solución actual se basa en instantáneas y no se transfieren datos al almacén. Las instantáneas se almacenan en la misma cuenta de almacenamiento que el recurso compartido de archivos del que se ha realizado la copia de seguridad.

## <a name="set-cross-region-restore"></a>Establecimiento de la restauración entre regiones

La opción **Restauración entre regiones (CRR)** permite restaurar datos en una [región de Azure emparejada](../best-practices-availability-paired-regions.md) secundaria.

Admite los orígenes de datos siguientes:

- Máquinas virtuales de Azure (disponibilidad general)
- Bases de datos SQL hospedadas en máquinas virtuales de Azure (versión preliminar)
- Bases de datos de SAP HANA hospedadas en máquinas virtuales de Azure (versión preliminar)

Usar Restauración entre regiones le permite hacer lo siguiente:

- Realizar simulacros cuando existen requisitos de cumplimiento o auditoría.
- Restaurar los datos si se produce un desastre en la región primaria.

Al restaurar una máquina virtual, puede restaurar la máquina virtual o su disco. Si va a restaurar a partir de bases de datos SQL o de SAP HANA hospedadas en máquinas virtuales de Azure, puede restaurar las bases de datos o sus archivos.

Para elegir esta característica, seleccione la opción **Habilitar la restauración entre regiones** en el panel **Configuración de copia de seguridad**.

Como este proceso se encuentra en el nivel de almacenamiento, hay [implicaciones de precios](https://azure.microsoft.com/pricing/details/backup/).

>[!NOTE]
>Antes de empezar:
>
>- Revise la [matriz de compatibilidad](backup-support-matrix.md#cross-region-restore) para obtener una lista de regiones y tipos administrados compatibles.
>- La característica de restauración entre regiones (CRR) para máquinas virtuales de Azure ahora está disponible con carácter general en todas las regiones públicas de Azure.
>- La restauración entre regiones de bases de datos de SQL y SAP HANA está en versión preliminar en todas las regiones públicas de Azure.
>- CRR es una característica opcional de nivel de almacén para cualquier almacén GRS (está desactivada de forma predeterminada).
>- Tras la incorporación, los elementos de copia de seguridad pueden tardar hasta 48 horas en estar disponibles en las regiones secundarias.
>- Actualmente, se admite CRR para las máquinas virtuales de Azure, las máquinas virtuales de Azure Resource Manager y las máquinas virtuales de Azure cifradas. No se admitirán las máquinas virtuales de Azure clásicas. Si otros tipos de administración admiten CRR, se inscribirán **automáticamente**.
>- Una vez que la protección se inicia por primera vez, **actualmente no se puede revertir** la restauración entre regiones a GRS o LRS.
>- Actualmente, la región secundaria [RPO](azure-backup-glossary.md#rpo-recovery-point-objective) está activa hasta 12 horas a partir de la región primaria, aunque la replicación del [almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) ](../storage/common/storage-redundancy.md#redundancy-in-a-secondary-region) es de 15 minutos.

### <a name="configure-cross-region-restore"></a>Configuración de la restauración entre regiones

Un almacén creado con redundancia GRS incluye la opción para configurar la característica Restauración entre regiones. Cada almacén GRS tendrá un banner, que se vinculará a la documentación. Para configurar CRR para el almacén, vaya al panel Configuración de copia de seguridad, que contiene la opción para habilitar esta característica.

 ![Banner Configuración de copia de seguridad](./media/backup-azure-arm-restore-vms/banner.png)

1. Desde el portal, diríjase a Almacén de Recovery Services > **Propiedades** (en **Configuración**).
1. En **Configuración de copia de seguridad**, seleccione **Actualizar**.
1. Seleccione la opción **Habilitar la restauración entre regiones en este almacén** para habilitar la funcionalidad.

   ![Habilitación de la restauración entre regiones](./media/backup-azure-arm-restore-vms/backup-configuration.png)

Consulte estos artículos para más información sobre la copia de seguridad y restauración con CRR:

- [Restauración entre regiones para máquinas virtuales de Azure](backup-azure-arm-restore-vms.md#cross-region-restore)
- [Restauración entre regiones para bases de datos SQL](restore-sql-database-azure-vm.md#cross-region-restore)
- [Restauración entre regiones para bases de datos de SAP HANA](sap-hana-db-restore.md#cross-region-restore)

## <a name="set-encryption-settings"></a>Definición de la configuración de cifrado

De forma predeterminada, los datos del almacén de Recovery Services se cifran mediante claves administradas por la plataforma. No es necesario que realice ninguna acción explícita por su parte para habilitar este cifrado y se aplica a todas las cargas de trabajo de las que se realiza una copia de seguridad en el almacén de Recovery Services.  Puede optar por traer su propia clave para cifrar los datos de copia de seguridad en este almacén. Esto se conoce como claves administradas por el cliente. Si desea cifrar los datos de copia de seguridad mediante su propia clave, la clave de cifrado debe especificarse antes de que se proteja cualquier elemento en este almacén. Una vez que habilite el cifrado con su clave, no se puede invertir.

### <a name="configuring-a-vault-to-encrypt-using-customer-managed-keys"></a>Configuración de un almacén para cifrar mediante claves administradas por el cliente

Para configurar el almacén con el fin de realizar el cifrado con claves administradas por el cliente, estos pasos deben seguirse en este orden:

1. Habilitación de la identidad administrada para el almacén de Recovery Services

1. Asignación de permisos al almacén para tener acceso a la clave de cifrado en Azure Key Vault

1. Habilitación de la eliminación temporal y la protección de purga para Azure Key Vault

1. Asignación de la clave de cifrado al almacén de Recovery Services

Encontrará instrucciones de cada uno de estos pasos [en este artículo](encryption-at-rest-with-cmk.md#configuring-a-vault-to-encrypt-using-customer-managed-keys).

## <a name="modifying-default-settings"></a>Modificación de la configuración predeterminada

Le recomendamos encarecidamente que revise la configuración predeterminada para el **tipo de replicación de almacenamiento** y la **configuración de seguridad** antes de configurar copias de seguridad en el almacén.

- El **tipo de replicación de almacenamiento** se establece de forma predeterminada en **Almacenamiento con redundancia geográfica** (GRS). Una vez que configure la copia de seguridad, se deshabilitará la opción para modificar.
  - Si todavía no ha configurado la copia de seguridad, [siga estos pasos](#set-storage-redundancy) para revisar y modificar la configuración.
  - Si ya ha configurado la copia de seguridad y debe pasar de GRS a LRS, [revise estas soluciones alternativas](#how-to-change-from-grs-to-lrs-after-configuring-backup).

- La **eliminación temporal** está **habilitada** de forma predeterminada en los almacenes recién creados para proteger los datos de copia de seguridad de eliminaciones accidentales o malintencionadas. [Siga estos pasos](./backup-azure-security-feature-cloud.md#enabling-and-disabling-soft-delete) para revisar y modificar la configuración.

### <a name="how-to-change-from-grs-to-lrs-after-configuring-backup"></a>Procedimiento para cambiar de GRS a LRS después de configurar la copia de seguridad

Antes de decidir pasar de GRS a almacenamiento con redundancia local (LRS), revise las ventajas y desventajas de los costos más bajos y la durabilidad más alta de los datos que se adapten al escenario. Si debe cambiar de GRS a LRS, tiene dos opciones. Dependen de los requisitos de la empresa para conservar los datos de copia de seguridad:

- [No es necesario conservar los datos de copia de seguridad anteriores](#dont-need-to-preserve-previous-backed-up-data)
- [Se deben conservar los datos de copia de seguridad anteriores](#must-preserve-previous-backed-up-data)

#### <a name="dont-need-to-preserve-previous-backed-up-data"></a>No es necesario conservar los datos de copia de seguridad anteriores

Para proteger las cargas de trabajo en un nuevo almacén LRS, la protección actual y los datos se tendrán que eliminar en el almacén GRS y las copias de seguridad se deben volver a configurar.

>[!WARNING]
>La operación siguiente es destructiva y no se puede deshacer. Todos los datos de copia de seguridad y los elementos de copia de seguridad asociados con el servidor protegido se eliminarán de forma permanente. Proceda con precaución.

Detenga y elimine la protección actual en el almacén GRS:

1. Deshabilite la eliminación temporal en las propiedades del almacén GRS. Siga [estos pasos](backup-azure-security-feature-cloud.md#disabling-soft-delete-using-azure-portal) para deshabilitar la eliminación temporal.

1. Detenga la protección y elimine las copias de seguridad del almacén GRS existente. En el menú del panel del almacén, seleccione **Elementos de copia de seguridad**. Los elementos que aparecen aquí y que deben moverse al almacén LRS se deben quitar junto con sus datos de copia de seguridad. Vea cómo [eliminar elementos protegidos en la nube](backup-azure-delete-vault.md#delete-protected-items-in-the-cloud) y [eliminar elementos protegidos en el entorno local](backup-azure-delete-vault.md#delete-protected-items-on-premises).

1. Si tiene previsto mover AFS (recursos compartidos de archivos de Azure), servidores SQL o servidores SAP HANA, también tendrá que anular su registro. En el menú del panel del almacén, seleccione **Infraestructura de Backup**. Vea cómo [anular el registro de SQL Server](manage-monitor-sql-database-backup.md#unregister-a-sql-server-instance), [anular el registro de una cuenta de almacenamiento asociada a recursos compartidos de archivos de Azure](manage-afs-backup.md#unregister-a-storage-account) y [anular el registro de una instancia de SAP HANA](sap-hana-db-manage.md#unregister-an-sap-hana-instance).

1. Una vez que se hayan quitado del almacén GRS, continúe con la configuración de las copias de seguridad de la carga de trabajo en el nuevo almacén LRS.

#### <a name="must-preserve-previous-backed-up-data"></a>Se deben conservar los datos de copia de seguridad anteriores

Si tiene que conservar los datos protegidos actuales en el almacén GRS y continuar la protección en un nuevo almacén LRS, hay opciones limitadas para algunas de las cargas de trabajo:

- Para MARS, puede [detener la protección con conservación de datos](backup-azure-manage-mars.md#stop-protecting-files-and-folder-backup) y registrar el agente en el nuevo almacén LRS.

  - El servicio Azure Backup seguirá conservando todos los puntos de recuperación existentes en el almacén GRS.
  - Tendrá que pagar para conservar los puntos de recuperación en el almacén GRS.
  - Solo podrá restaurar los datos de copia de seguridad de los puntos de recuperación que no hayan expirado en el almacén GRS.
  - Tendrá que crear una réplica inicial de los datos en el almacén LRS.

- En el caso de una máquina virtual de Azure, puede [detener la protección con conservación de datos](backup-azure-manage-vms.md#stop-protecting-a-vm) para la máquina virtual en el almacén GRS, moverla a otro grupo de recursos y, después, protegerla en el almacén LRS. Vea [Instrucciones y limitaciones](../azure-resource-manager/management/move-limitations/virtual-machines-move-limitations.md) para mover una máquina virtual a otro grupo de recursos.

  Una máquina virtual solo se puede proteger en un almacén a la vez. Pero la máquina virtual en el nuevo grupo de recursos se puede proteger en el almacén LRS, ya que se considera una máquina virtual distinta.

  - El servicio Azure Backup conservará los puntos de recuperación de los que se ha realizado una copia de seguridad en el almacén GRS.
  - Deberá pagar para mantener los puntos de recuperación en el almacén GRS (vea [Precios de Azure Backup](azure-backup-pricing.md) para obtener detalles).
  - Podrá restaurar la máquina virtual si es necesario, desde el almacén GRS.
  - La primera copia de seguridad en el almacén LRS de la máquina virtual en el nuevo recurso será una réplica inicial.

## <a name="next-steps"></a>Pasos siguientes

[Más información](backup-azure-recovery-services-vault-overview.md) sobre los almacenes de Recovery Services.
[Más información](backup-azure-delete-vault.md) sobre la eliminación de los almacenes de Recovery Services.