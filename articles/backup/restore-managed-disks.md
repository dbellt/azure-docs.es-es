---
title: Restauración de Azure Managed Disks
description: Aprenda a restaurar Azure Managed Disks desde Azure Portal.
ms.topic: conceptual
ms.date: 01/07/2021
ms.openlocfilehash: c57d60047a5bcef58c721ee25bd8a0b3ed523aa4
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107517205"
---
# <a name="restore-azure-managed-disks"></a>Restauración de Azure Managed Disks

En este artículo se explica cómo restaurar [Azure Managed Disks](../virtual-machines/managed-disks-overview.md) desde un punto de restauración que haya creado Azure Backup.

Actualmente, no se admite la opción de Recuperación de la ubicación original (OLR) de la restauración mediante la sustitución del disco de origen desde el que se realizaron las copias de seguridad. Puede realizar la restauración desde un punto de recuperación para crear un nuevo disco en el mismo grupo de recursos que el disco de origen desde el que se realizaron las copias de seguridad o en cualquier otro grupo de recursos. Esto se conoce como Recuperación de la ubicación alternativa (ALR), y le será de ayuda para mantener el disco de origen y el disco restaurado (nuevo).

En este artículo, aprenderá a:

- Realizar una restauración para crear un disco nuevo.

- Realizar el seguimiento del estado de la operación de restauración.

## <a name="restore-to-create-a-new-disk"></a>Realizar una restauración para crear un disco nuevo.

El almacén de Backup usa la identidad administrada para obtener acceso a otros recursos de Azure. Para restaurar contenido a partir de una copia de seguridad, la identidad administrada del almacén de Backup debe tener un conjunto de permisos en el grupo de recursos en el que se vaya a restaurar el disco.

El almacén de Backup usa una identidad administrada asignada por el sistema; cada recurso solo puede tener una identidad, y cada una de ellas está asociada al ciclo de vida del recurso. Puede conceder permisos a la identidad administrada mediante el control de acceso basado en roles de Azure (Azure RBAC). Tenga en cuenta que una identidad administrada es una entidad de servicio de un tipo especial que solo se puede usar con recursos de Azure. Obtenga más información sobre las [identidades administradas](../active-directory/managed-identities-azure-resources/overview.md).

Los siguientes requisitos previos son necesarios para realizar una operación de restauración:

1. Asigne el rol de **Operador de restauración de discos** a la identidad administrada del almacén de Backup que se encuentra en el grupo de recursos en el que el servicio Azure Backup restaurará el disco.

    >[!NOTE]
    > Puede elegir el mismo grupo de recursos que el disco de origen desde el que se realizan las copias de seguridad o de cualquier otro grupo de recursos que se encuentre dentro de la misma suscripción o que esté en otra diferente.

    1. Vaya al grupo de recursos en el que se va a restaurar el disco. Por ejemplo, el grupo de recursos es *TargetRG*.

    1. Vaya al **Control de acceso (IAM)** y seleccione **Add role assignments** (Agregar asignaciones de roles).

    1. En el panel de contexto derecho, seleccione **Disk Restore Operator** (Operador de restauración de discos) en la lista desplegable **Rol**. Seleccione la identidad administrada del almacén de Backup y haga clic en **Guardar**.

        >[!TIP]
        >Escriba el nombre del almacén de Backup para seleccionar la identidad administrada del almacén.

        ![Selección del rol de operador de restauración de discos](./media/restore-managed-disks/disk-restore-operator-role.png)

1. Compruebe que la identidad administrada del almacén de Backup tiene el conjunto adecuado de asignaciones de roles en el grupo de recursos donde se restaurará el disco.

    1. Vaya a **Almacén de Backup-> Identidad** y seleccione **Asignaciones de roles de Azure**.

        ![Selección de la opción Asignaciones de roles de Azure](./media/restore-managed-disks/azure-role-assignments.png)

    1. Compruebe que el rol, el nombre del recurso y el tipo de recurso aparecen correctamente.

        ![Comprobación del rol, el nombre del recurso y el tipo de recurso](./media/restore-managed-disks/verify-role.png)

    >[!NOTE]
    >Aunque las asignaciones de roles se reflejan correctamente en el portal, el permiso puede tardar aproximadamente 15 minutos en aplicarse en la identidad administrada del almacén de Backup.
    >
    >Durante las copias de seguridad programadas o una operación de copia de seguridad a petición, Azure Backup almacena las instantáneas incrementales del disco en el grupo de recursos de instantáneas que se proporciona durante la configuración de la copia de seguridad del disco. Azure Backup usa estas instantáneas incrementales durante la operación de restauración. Si las instantáneas se eliminan o se mueven desde el grupo de recursos de instantáneas o si se revocan las asignaciones de roles del almacén de Backup en el grupo de recursos de instantáneas, se producirá un error en la operación de restauración.

1. Si el disco que se va a restaurar está cifrado con [claves administradas por el cliente (CMK)](../virtual-machines/disks-enable-customer-managed-keys-portal.md) o con el [cifrado doble que usa claves administradas por la plataforma y claves administradas por el cliente](../virtual-machines/disks-enable-double-encryption-at-rest-portal.md), asigne el permiso del rol **Lector** a la identidad administrada del almacén de Backup del recurso **Conjunto de cifrado de disco**.

Una vez cumplidos los requisitos previos, siga estos pasos para realizar la operación de restauración.

1. En [Azure Portal](https://portal.azure.com/), vaya al **Centro de copias de seguridad**. Seleccione **Instancias de copia de seguridad** en la sección **Administrar**. En la lista de instancias de copia de seguridad, seleccione la instancia de copia de seguridad del disco para la que quiera realizar la operación de restauración.

    ![Lista de instancias de copia de seguridad](./media/restore-managed-disks/backup-instances.png)

    Como alternativa, puede realizar esta operación desde el almacén de Backup que usó para configurar la copia de seguridad del disco.

1. En la pantalla **Instancia de copia de seguridad**, seleccione el punto de restauración que quiera usar para realizar la operación de restauración, y haga clic en **Restaurar**.

    ![Seleccionar punto de restauración](./media/restore-managed-disks/select-restore-point.png)

1. En el flujo de trabajo **Restaurar**, revise la información de las pestañas **Conceptos básicos** y **Seleccionar punto de recuperación**; a continuación, haga clic en **Siguiente: restaurar parámetros**.

    ![Revisión de la información de las opciones Conceptos básicos y Seleccionar punto de recuperación](./media/restore-managed-disks/review-information.png)

1. En la pestaña **Restaurar parámetros**, seleccione la **Suscripción de destino** y el **Grupo de recursos de destino** en el que quiera restaurar la copia de seguridad. Proporcione el nombre del disco que se va a restaurar. Seleccione **Siguiente: revisar y restaurar**.

    ![Restauración de parámetros](./media/restore-managed-disks/restore-parameters.png)

    >[!TIP]
    >Los discos de los que Azure Backup realiza una copia de seguridad mediante la solución Disk Backup también se pueden incluir en la solución de copia de seguridad de Azure VM con el almacén de Recovery Services. Si ha configurado la protección de Azure VM a la que está conectado este disco, también puede usar la operación de restauración de Azure VM. Puede restaurar la VM o los discos y archivos o carpetas desde el punto de recuperación de la instancia de copia de seguridad de Azure VM correspondiente. Para obtener más información, consulte [Copia de seguridad de Azure VM](./about-azure-vm-restore.md).

1. Una vez que la validación sea correcta, seleccione **Restaurar** para iniciar la operación de restauración.

    ![Iniciación de la operación de restauración](./media/restore-managed-disks/initiate-restore.png)

    >[!NOTE]
    > La validación puede tardar unos minutos en completarse antes de que pueda desencadenar la operación de restauración. Puede producirse un error de validación si:
    >
    > - ya existe un disco con el mismo nombre en el campo **Nombre de disco restaurado** del **Grupo de recursos de destino**.
    > - la identidad administrada del almacén de Backup no tiene asignaciones de roles válidas en el **Grupo de recursos de destino**.
    > - las asignaciones de roles de la identidad administrada del almacén de Backup se revocan en el **Grupo de recursos de instantáneas** donde se almacenan las instantáneas incrementales.
    > - si se eliminan o se mueven instantáneas incrementales desde el grupo de recursos de instantáneas.

Se creará un nuevo disco a partir del punto de recuperación seleccionado en el grupo de recursos de destino que se proporcionó durante la operación de restauración. Para usar el disco restaurado en una máquina virtual existente, necesitará realizar más pasos:

- Si el disco restaurado es un disco de datos, puede conectar un disco existente a una máquina virtual. Si el disco restaurado es un disco de sistema operativo, puede intercambiar el disco del sistema operativo de una máquina virtual desde el Azure Portal en el panel **Máquina virtual** del menú **Discos** que está en la sección **Configuración**.

    ![Cambio de discos de sistema operativo](./media/restore-managed-disks/swap-os-disks.png)

- En el caso de las máquinas virtuales Windows, si el disco restaurado es un disco de datos, siga las instrucciones para [desasociar el disco de datos original](../virtual-machines/windows/detach-disk.md#detach-a-data-disk-using-the-portal) de la máquina virtual. A continuación, [conecte el disco restaurado](../virtual-machines/windows/attach-managed-disk-portal.md) a la máquina virtual. Siga las instrucciones para [intercambiar el disco del sistema operativo](../virtual-machines/windows/os-disk-swap.md) de la máquina virtual con el disco restaurado.

- En el caso de las máquinas virtuales Linux, si el disco restaurado es un disco de datos, siga las instrucciones para [desasociar el disco de datos original](../virtual-machines/linux/detach-disk.md#detach-a-data-disk-using-the-portal) de la máquina virtual. A continuación, [conecte el disco restaurado](../virtual-machines/linux/attach-disk-portal.md#attach-an-existing-disk) a la máquina virtual. Siga las instrucciones para [intercambiar el disco del sistema operativo](../virtual-machines/linux/os-disk-swap.md) de la máquina virtual con el disco restaurado.

Se recomienda revocar la asignación del rol **Operador de restauración de discos** de la identidad administrada del almacén de Backup que se encuentra en el **Grupo de recursos de destino**, después de que finalice correctamente la operación de restauración.

## <a name="track-a-restore-operation"></a>Seguimiento de una operación de restauración

Una vez que se desencadene la operación de restauración, el servicio de copia de seguridad crea un trabajo para realizar su seguimiento. Azure Backup muestra las notificaciones sobre el trabajo en el portal. Para ver el progreso del trabajo de restauración:

1. Vaya a la pantalla **Instancia de copia de seguridad**. En ella se muestra el panel de trabajos que cuenta con la operación y el estado de los últimos siete días.

    ![Panel de trabajos](./media/restore-managed-disks/jobs-dashboard.png)

1. Para ver el estado de la operación de restauración, seleccione **Ver todos** para mostrar los trabajos en curso y pasados de esta instancia de copia de seguridad.

    ![Seleccionar Ver todo](./media/restore-managed-disks/view-all.png)

1. Revise la lista de trabajos de copia de seguridad y restauración, así como su estado. Seleccione un trabajo de la lista de trabajos para ver los detalles del mismo.

    ![Lista de trabajos](./media/restore-managed-disks/list-of-jobs.png)

## <a name="next-steps"></a>Pasos siguientes

- [Preguntas más frecuentes sobre Azure Disk Backup](disk-backup-faq.yml)