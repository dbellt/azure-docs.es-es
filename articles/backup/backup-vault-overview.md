---
title: Información general de los almacenes de Backup
description: Información general de los almacenes de Backup.
ms.topic: conceptual
ms.date: 04/19/2021
ms.openlocfilehash: e2d720da9474a35870de01559201d22c9e5b567f
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107739087"
---
# <a name="backup-vaults-overview"></a>Información general de los almacenes de Backup

En este artículo se describen las características de un almacén de Backup. Un almacén de Backup es una entidad de almacenamiento de Azure que hospeda los datos de copia de seguridad para ciertas cargas de trabajo más recientes que Azure Backup admite. Puede usar almacenes de Backup para almacenar los datos de copia de seguridad de varios servicios de Azure, como servidores de Azure Database for PostgreSQL, y cargas de trabajo más recientes que Azure Backup admitirá. Los almacenes de Backup facilitan la tarea de organizar los datos de copia de seguridad, al mismo tiempo que reducen al mínimo la sobrecarga administrativa. Los almacenes de Backup se basan en el modelo de Azure Resource Manager de Azure, que proporciona características como las siguientes:

- **Funcionalidades mejoradas para ayudar a proteger datos de copia de seguridad**: con los almacenes de Backup, Azure Backup proporciona funcionalidades de seguridad para proteger las copias de seguridad en la nube. Estas características de seguridad garantizan que puede proteger las copias de seguridad y recuperar datos de forma segura, incluso si los servidores de producción y copia de seguridad están en peligro. [Más información](backup-azure-security-feature.md)

- **Control de acceso basado en roles de Azure (Azure RBAC)** : Azure RBAC permite un control muy detallado de la administración de acceso en Azure. [Azure proporciona diversas funciones integradas](../role-based-access-control/built-in-roles.md) y Azure Backup tiene tres [roles integrados para administrar puntos de recuperación](backup-rbac-rs-vault.md). Los almacenes de Backup son compatibles con Azure RBAC, que restringe el acceso de copia de seguridad y restauración al conjunto definido de roles de usuario. [Más información](backup-rbac-rs-vault.md)

## <a name="storage-settings-in-the-backup-vault"></a>Configuración de almacenamiento en el almacén de Backup

Un almacén de Backup es una entidad que almacena las copias de seguridad y los puntos de recuperación creados a lo largo del tiempo. El almacén de Backup también contiene las directivas de copia de seguridad asociadas a las máquinas virtuales protegidas.

- Azure Backup administra automáticamente el almacenamiento para el almacén. Elija la redundancia de almacenamiento que se ajuste a sus necesidades empresariales al crear el almacén de Backup.

- Para obtener más información acerca de la redundancia de almacenamiento, consulte estos artículos sobre redundancia [geográfica](../storage/common/storage-redundancy.md#geo-redundant-storage) y [local](../storage/common/storage-redundancy.md#locally-redundant-storage).

## <a name="encryption-settings-in-the-backup-vault"></a>Configuración de cifrado en el almacén de Backup

En esta sección se describen las opciones disponibles para cifrar los datos de copia de seguridad almacenados en el almacén de Backup. El servicio Azure Backup usa la aplicación **Backup Management Service** para acceder a Azure Key Vault, pero no a la identidad administrada del almacén de Backup.


### <a name="encryption-of-backup-data-using-platform-managed-keys"></a>Cifrado de datos de copia de seguridad mediante claves administradas por la plataforma

De forma predeterminada, todos los datos se cifran mediante claves administradas por la plataforma. No es necesario realizar ninguna acción explícita de su parte para habilitar este cifrado. Se aplica a todas las cargas de trabajo de las que se realiza una copia de seguridad en el almacén de Backup.

## <a name="create-a-backup-vault"></a>Creación de un almacén de Backup

Un almacén de Backup es una entidad de administración que almacena los puntos de recuperación creados a lo largo del tiempo y proporciona una interfaz para realizar operaciones relacionadas con la copia de seguridad. Esto incluye realizar copias de seguridad a petición, realizar restauraciones y crear directivas de copia de seguridad.

Para crear un almacén de Backup, siga estos pasos.

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en <https://portal.azure.com>.

### <a name="create-backup-vault"></a>Creación de un almacén de Backup

1. Escriba **almacenes de copia de seguridad** en el cuadro de búsqueda.
1. En **Servicios**, seleccione **Almacenes de Backup**.
1. En la página **Almacenes de Backup**, seleccione **Agregar**.
1. En la pestaña **Aspectos básicos**, en **Detalles del proyecto**, asegúrese de que esté seleccionada la suscripción correcta y, en Grupo de recursos, elija **Crear nuevo**. Escriba *myResourceGroup* para el nombre.

  ![Crear un grupo de recursos](./media/backup-vault-overview/new-resource-group.png)

1. En **Detalles de instancia**, escriba *myVault* como **Nombre del almacén de Backup** y seleccione la región de su elección, en este caso, *Este de EE. UU.* como su **Región**.
1. Ahora elija la **redundancia de almacenamiento**. La redundancia de almacenamiento no se puede cambiar después de proteger los elementos en el almacén.
1. Se recomienda que, si usa Azure como punto de conexión del almacenamiento de copia de seguridad principal, siga utilizando la configuración **con redundancia geográfica** predeterminada.
1. Si no utiliza Azure como punto de conexión de almacenamiento de copia de seguridad principal, elija **Redundancia local** para reducir los costes de almacenamiento de Azure.
1. Obtenga más información sobre la redundancia [geográfica](../storage/common/storage-redundancy.md#geo-redundant-storage) y [local](../storage/common/storage-redundancy.md#locally-redundant-storage).

  ![Elección de la redundancia de almacenamiento](./media/backup-vault-overview/storage-redundancy.png)

1. Seleccione el botón Revisar y crear de la parte inferior de la página.

    ![Selección de Revisar y crear](./media/backup-vault-overview/review-and-create.png)

## <a name="delete-a-backup-vault"></a>Eliminación de un almacén de Backup

En esta sección se describe cómo eliminar un almacén de Backup. Contiene instrucciones para quitar las dependencias y, a continuación, eliminar un almacén.

### <a name="before-you-start"></a>Antes de comenzar

No se puede eliminar un almacén de Backup que tenga alguna de las siguientes dependencias:

- No se puede eliminar un almacén que contenga orígenes de datos protegidos (por ejemplo, servidores de Azure Database for PostgreSQL).
- No puede eliminar un almacén que contiene datos de copia de seguridad.

Si intenta eliminar el almacén sin quitar las dependencias, se producirá uno de los siguientes mensajes de error:

>No se puede eliminar el almacén de Backup porque existen instancias o directivas de copia de seguridad en el almacén. Elimine todas las instancias y directivas de copia de seguridad del almacén y vuelva a intentar eliminarlo.

### <a name="proper-way-to-delete-a-vault"></a>Manera adecuada de eliminar un almacén

>[!WARNING]
La operación siguiente es destructiva y no se puede deshacer. Todos los datos de copia de seguridad y los elementos de copia de seguridad asociados con el servidor protegido se eliminarán de forma permanente. Proceda con precaución.

Para eliminar correctamente un almacén, debe seguir los pasos en este orden:

- Compruebe si hay algún elemento protegido:
  - Vaya a **Instancias de Backup** en la barra de navegación izquierda. Todos los elementos que aparecen aquí deben eliminarse primero.

Después de completar estos pasos, puede continuar y eliminar el almacén.

### <a name="delete-the-backup-vault"></a>Eliminación del almacén de Backup

Cuando no haya ningún otro elemento en el almacén, seleccione **Eliminar** en el panel del almacén. Verá un texto de confirmación que le pregunta si quiere eliminar el almacén.

![Eliminación del almacén](./media/backup-vault-overview/delete-vault.png)

1. Para verificar que desea eliminar el almacén, seleccione **Sí**. El almacén se elimina. El portal vuelve al menú **Nuevo** servicio.

## <a name="monitor-and-manage-the-backup-vault"></a>Supervisión y administración del almacén de Backup

En esta sección se describe cómo usar el panel de **información general** del almacén de Backup para supervisar y administrar los almacenes de Backup. El panel de información general contiene dos iconos: **Trabajos** e **Instancias**.

![Panel de información general](./media/backup-vault-overview/overview-dashboard.png)

### <a name="manage-backup-instances"></a>Administración de instancias de Backup

En el icono **Trabajos**, obtendrá una vista resumida de todos los trabajos relacionados con la copia de seguridad y restauración en el almacén de copia de seguridad. La selección de cualquiera de los números de este icono le permite ver más información sobre los trabajos de un tipo de origen de datos, tipo de operación y estado en concreto.

![Instancias de Backup](./media/backup-vault-overview/backup-instances.png)

### <a name="manage-backup-jobs"></a>Administración de trabajos de copia de seguridad

En el icono **Instancias de Backup**, se obtiene una vista resumida de todas las instancias de copia de seguridad del almacén de Backup. La selección de cualquiera de los números de este icono le permite ver más información sobre las instancias de copia de seguridad para un tipo de origen de datos y estado de protección en concreto.

![Trabajos de copia de seguridad](./media/backup-vault-overview/backup-jobs.png)

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de la copia de seguridad en bases de datos de Azure PostgreSQL](backup-azure-database-postgresql.md#configure-backup-on-azure-postgresql-databases)
