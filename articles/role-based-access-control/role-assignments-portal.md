---
title: 'Asignación de roles de Azure mediante Azure Portal: Control de acceso basado en roles de Azure'
description: Aprenda a conceder acceso a recursos de Azure para usuarios, grupos, entidades de servicio e identidades administradas mediante Azure Portal y el control de acceso basado en roles de Azure (RBAC de Azure).
services: active-directory
author: rolyon
manager: daveba
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 05/07/2021
ms.author: rolyon
ms.custom: contperf-fy21q3-portal,subject-rbac-steps
ms.openlocfilehash: 95ac82bfbdef8843f3eb4fc10cea7a0309d9eb89
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109656871"
---
# <a name="assign-azure-roles-using-the-azure-portal"></a>Asignación de roles de Azure mediante Azure Portal

[!INCLUDE [Azure RBAC definition grant access](../../includes/role-based-access-control/definition-grant.md)] En este artículo se describe cómo asignar roles con Azure Portal.

Si necesita asignar roles de administrador en Azure Active Directory, consulte [Asignación de roles de Azure AD a usuarios](../active-directory/roles/manage-roles-portal.md).

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [Azure role assignment prerequisites](../../includes/role-based-access-control/prerequisites-role-assignments.md)]

## <a name="choose-experience"></a>Elección de la experiencia

El control de acceso basado en roles de Azure tiene una nueva experiencia para asignar roles de Azure en Azure Portal que se encuentra actualmente en versión preliminar pública. Si desea probar esta nueva experiencia, siga los pasos de la pestaña **(Versión preliminar)** .

#### <a name="current"></a>[Current](#tab/current/)

## <a name="step-1-identify-the-needed-scope"></a>Paso 1: identificar el ámbito necesario

[!INCLUDE [Scope for Azure RBAC introduction](../../includes/role-based-access-control/scope-intro.md)] Para más información, consulte [Descripción del ámbito](scope-overview.md).

![Diagrama que muestra los niveles del ámbito para el control de acceso basado en roles de Azure.](../../includes/role-based-access-control/media/scope-levels.png)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En el cuadro de búsqueda de la parte superior, busque el ámbito al que quiere conceder acceso. Por ejemplo, busque **Grupos de administración**, **Suscripciones**, **Grupos de recursos** o un recurso específico.

    ![Captura de pantalla de la búsqueda en Azure Portal de un grupo de recursos.](./media/shared/rg-portal-search.png)

1. Haga clic en el recurso específico de ese ámbito.

    A continuación se muestra un grupo de recursos de ejemplo.

    ![Captura de pantalla de la página de información general del grupo de recursos.](./media/shared/rg-overview.png)

## <a name="step-2-open-the-add-role-assignment-pane"></a>Paso 2: abrir el panel Agregar asignación de roles

**Control de acceso (IAM)** es la página que se usa normalmente para asignar roles y conceder acceso a los recursos de Azure. También se conoce como administración de identidad y acceso (IAM) y aparece en varias ubicaciones de Azure Portal.

1. Haga clic en **Control de acceso (IAM).**

    A continuación se muestra un ejemplo de la página Control de acceso (IAM) de un grupo de recursos.

    ![Captura de pantalla de la página Control de acceso (IAM) para un grupo de recursos.](./media/shared/rg-access-control.png)

1. Haga clic en la pestaña **Asignaciones de roles** para ver todas las asignaciones de roles en este ámbito.

1. Haga clic en **Agregar** > **Agregar asignación de roles**.
   Si no tiene permisos para asignar roles, la opción Agregar asignación de roles se deshabilitará.

   ![Captura de pantalla del menú Agregar > Agregar asignación de roles.](./media/shared/add-role-assignment-menu.png)

    Se abre el panel Agregar asignación de roles.

   ![Captura de pantalla de la página Agregar asignación de roles con las opciones Rol, Asignar acceso a y Seleccionar.](../../includes/role-based-access-control/media/add-role-assignment-page.png)

## <a name="step-3-select-the-appropriate-role"></a>Paso 3: seleccionar el rol apropiado

1. En la lista **Role**, busque o desplácese para buscar el rol que quiere asignar.

    Para ayudarle a determinar el rol adecuado, puede mantener el puntero sobre el icono de información para mostrar una descripción del rol. Para más información, puede ver el artículo [Roles integrados de Azure](built-in-roles.md).

   ![Captura de pantalla de la lista Seleccionar un rol en Agregar asignación de roles.](./media/role-assignments-portal/add-role-assignment-role.png)

1. Haga clic para seleccionar el rol.

## <a name="step-4-select-who-needs-access"></a>Paso 4: elegir quién necesita acceso

1. En la lista **Asignar acceso a** , seleccione el tipo de entidad de seguridad a la que quiere asignar acceso.

    | Tipo | Descripción |
    | --- | --- |
    | **Usuario, grupo o entidad de servicio** | Si quiere asignar el rol a un usuario, grupo o entidad de servicio (aplicación), seleccione este tipo. |
    | **Identidad administrada asignada por el usuario** | Si quiere asignar el rol a una [identidad administrada asignada por el usuario](../active-directory/managed-identities-azure-resources/overview.md), seleccione este tipo. |
    | *Identidad administrada asignada por el sistema* | Si quiere asignar el rol a una [identidad administrada asignada por el sistema](../active-directory/managed-identities-azure-resources/overview.md), seleccione la instancia de servicio de Azure en la que se encuentra la identidad administrada. |

   ![Captura de pantalla de la selección de una entidad de seguridad en Agregar asignación de roles.](./media/role-assignments-portal/add-role-assignment-type.png)

1. Si ha seleccionado una identidad administrada asignada por el usuario o una identidad administrada asignada por el sistema, seleccione la **suscripción** en la que se encuentra la identidad administrada.

1. En la sección **Seleccionar**, escriba una cadena o desplácese por la lista para buscar la entidad de seguridad.

   ![Captura de pantalla de la selección de un usuario en Agregar asignación de roles.](./media/role-assignments-portal/add-role-assignment-user.png)

1. Una vez que haya encontrado la entidad de seguridad, haga clic en esta para seleccionarla.

## <a name="step-5-assign-role"></a>Paso 5: Asignación de un rol

1. Haga clic en **Guardar** para asignar el rol.

   Transcurridos unos instantes, se asigna el rol a la entidad de seguridad en el ámbito seleccionado.

1. En la pestaña **Asignaciones de roles**, compruebe que ve la asignación de roles en la lista.

    ![Captura de pantalla de la lista de asignación de roles después de asignar el rol.](./media/role-assignments-portal/rg-role-assignments.png)

#### <a name="preview"></a>[(Versión preliminar)](#tab/preview/)

## <a name="step-1-identify-the-needed-scope-preview"></a>Paso 1: Identificar el ámbito necesario (versión preliminar)

[!INCLUDE [Scope for Azure RBAC introduction](../../includes/role-based-access-control/scope-intro.md)] Para más información, consulte [Descripción del ámbito](scope-overview.md).

![Diagrama que muestra los niveles del ámbito para el control de acceso basado en roles de Azure para la experiencia con la versión preliminar.](../../includes/role-based-access-control/media/scope-levels.png)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En el cuadro de búsqueda de la parte superior, busque el ámbito al que quiere conceder acceso. Por ejemplo, busque **Grupos de administración**, **Suscripciones**, **Grupos de recursos** o un recurso específico.

    ![Captura de pantalla de la búsqueda en Azure Portal de un grupo de recursos en la experiencia con la versión preliminar.](./media/shared/rg-portal-search.png)

1. Haga clic en el recurso específico de ese ámbito.

    A continuación se muestra un grupo de recursos de ejemplo.

    ![Captura de pantalla de la página de información general del grupo de recursos en la experiencia con la versión preliminar.](./media/shared/rg-overview.png)

## <a name="step-2-open-the-add-role-assignment-page-preview"></a>Paso 2: Abrir la página Agregar asignación de roles (versión preliminar)

**Control de acceso (IAM)** es la página que se usa normalmente para asignar roles y conceder acceso a los recursos de Azure. También se conoce como administración de identidad y acceso (IAM) y aparece en varias ubicaciones de Azure Portal.

1. Haga clic en **Control de acceso (IAM).**

    A continuación se muestra un ejemplo de la página Control de acceso (IAM) de un grupo de recursos.

    ![Captura de pantalla de la página Control de acceso (IAM) de un grupo de recursos en la experiencia con la versión preliminar.](./media/shared/rg-access-control.png)

1. Haga clic en la pestaña **Asignaciones de roles** para ver todas las asignaciones de roles en este ámbito.

1. Haga clic en **Agregar** > **Agregar asignación de roles (versión preliminar)** .

    Si no tiene permisos para asignar roles, la opción Agregar asignación de roles se deshabilitará.

    ![Captura de pantalla del menú Agregar > Agregar asignación de roles en la experiencia con la versión preliminar.](./media/shared/add-role-assignment-menu-preview.png)

    Se abre la página Asignar un rol.

## <a name="step-3-select-the-appropriate-role-preview"></a>Paso 3: Seleccionar el rol apropiado (versión preliminar)

1. En la pestaña **Roles**, seleccione el rol que quiera usar.

    Puede buscar un rol por nombre o por descripción. También puede filtrar los roles por tipo y categoría.

   ![Captura de pantalla de la página Agregar asignación de roles con la pestaña Seleccionar rol en la experiencia con la versión preliminar.](./media/role-assignments-portal/roles.png)

1. En la columna **Detalles** haga clic en **Ver** para obtener más información sobre un rol.

   ![Captura de pantalla del panel Ver detalles del rol con la pestaña Permisos de la experiencia con la versión preliminar.](./media/role-assignments-portal/select-role-permissions.png)

1. Haga clic en **Next**.

## <a name="step-4-select-who-needs-access-preview"></a>Paso 4: Elegir quién necesita acceso (versión preliminar)

1. En la pestaña **Miembros**, seleccione **Usuario, grupo o entidad de servicio** para asignar el rol seleccionado a uno o varios usuarios, grupos o entidades de servicio (aplicaciones) de Azure AD.

   ![Captura de pantalla de la página Agregar asignación de roles con la pestaña Agregar miembros de la experiencia con la versión preliminar.](./media/role-assignments-portal/members.png)

1. Haga clic en **Agregar miembros**.

1. Busque y seleccione los usuarios, grupos o entidades de servicio.

    Puede escribir en el cuadro **Seleccionar** para buscar en el directorio por nombres para mostrar, direcciones de correo electrónico o identificadores de objeto.

   ![Captura de pantalla de la opción Agregar miembros mediante el panel Seleccionar miembros de la experiencia con la versión preliminar.](./media/role-assignments-portal/select-principal.png)

1. Haga clic en **Guardar** para agregar los usuarios, grupos o entidades de servicio a la lista Miembros.

1. Para asignar el rol seleccionado a una o varias identidades administradas, seleccione **Identidad administrada**.

1. Haga clic en **Agregar miembros**.

1. En el panel **Selección de identidades administradas**, seleccione si el tipo es una [identidad administrada asignada por el sistema](../active-directory/managed-identities-azure-resources/overview.md) o una [identidad administrada asignada por el usuario](../active-directory/managed-identities-azure-resources/overview.md).

1. Busque y seleccione las identidades administradas.

    Si ha seleccionado una identidad administrada asignada por el sistema, tiene que seleccionar la instancia de servicio de Azure en la que se encuentra la identidad administrada.

   ![Captura de pantalla de la opción para agregar identidades administradas asignadas por el sistema mediante el panel Seleccionar la entidad de seguridad de la experiencia con la versión preliminar.](./media/role-assignments-portal/select-managed-identity-system.png)

   ![Captura de pantalla de la opción para agregar identidades administradas asignadas por el usuario mediante el panel Seleccionar la entidad de seguridad de la experiencia con la versión preliminar.](./media/role-assignments-portal/select-managed-identity-user.png)

1. Haga clic en **Guardar** para agregar las identidades administradas a la lista Miembros.

1. En el cuadro de texto **Descripción**, escriba una descripción opcional para esta asignación de roles.

    Más adelante puede mostrar esta descripción en la lista de asignaciones de roles.

1. Haga clic en **Next**.

## <a name="step-5-optional-add-condition-preview"></a>Paso 5: (Opcional) Agregar condición (versión preliminar)

Si seleccionó un rol que admite condiciones, aparecerá una pestaña **Condición** y tendrá la opción de agregar una condición a la asignación de roles. Una [condición](conditions-overview.md) es una comprobación adicional que puede agregar opcionalmente a la asignación de roles para proporcionar un control de acceso más preciso.

Actualmente, se pueden agregar condiciones a asignaciones de roles integradas o personalizadas que tienen [acciones de datos de Storage Blob](conditions-format.md#actions). Entre estas se incluyen los siguientes roles integrados:


- [Colaborador de datos de blobs de almacenamiento](built-in-roles.md#storage-blob-data-contributor)
- [Propietario de datos de blobs de almacenamiento](built-in-roles.md#storage-blob-data-owner)
- [Lector de datos de blobs de almacenamiento](built-in-roles.md#storage-blob-data-reader)

1. Haga clic en **Agregar condición** si desea refinar aún más las asignaciones de roles en función de los atributos de Blob Storage. Para más información, consulte [Adición o edición de condiciones de asignación de roles de Azure](conditions-role-assignments-portal.md).

   ![Captura de pantalla de la página Agregar asignación de roles con la pestaña Agregar condición de la experiencia con la versión preliminar.](./media/shared/condition.png)

1. Haga clic en **Next**.

## <a name="step-6-assign-role-preview"></a>Paso 6: Asignar rol (versión preliminar)

1. En la pestaña **Revisión y asignación**, revise la configuración de la asignación de roles.

   ![Captura de pantalla de la página Asignar un rol con la pestaña Revisión y asignación en la experiencia con la versión preliminar.](./media/role-assignments-portal/review-assign.png)

1. Haga clic en **Revisión y asignación** para asignar el rol.

   Transcurridos unos instantes, se asigna el rol a la entidad de seguridad en el ámbito seleccionado.

    ![Captura de pantalla de la lista de asignaciones de roles después de asignar el rol en la experiencia con la versión preliminar.](./media/role-assignments-portal/rg-role-assignments.png)

1. Si no ve la descripción de la asignación de roles, haga clic en **Editar columnas** para agregar la columna **Descripción**.

---

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de un usuario como administrador de una suscripción de Azure](role-assignments-portal-subscription-admin.md)
- [Eliminación de asignaciones de roles de Azure](role-assignments-remove.md)
- [Solución de problemas de Azure RBAC](troubleshooting.md)
