---
title: 'Asignación de roles de Azure AD a usuarios: Azure Active Directory'
description: Obtenga información acerca de cómo conceder acceso a los usuarios en Azure Active Directory mediante la asignación de roles de Azure AD.
services: active-directory
author: rolyon
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: how-to
ms.date: 05/14/2021
ms.author: rolyon
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1d62e723f42838343798603517ca430d727ff517
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110787841"
---
# <a name="assign-azure-ad-roles-to-users"></a>Asignación de roles de Azure AD a usuarios

Ahora puede ver y administrar todos los miembros de los roles de administrador en el centro de administración de Azure AD. Si administra las asignaciones de roles con frecuencia, probablemente preferirá esta experiencia. En este artículo se describe cómo asignar roles de Azure AD mediante el centro de administración de Azure AD.

## <a name="prerequisites"></a>Prerrequisitos

- Administrador global o administrador de roles con privilegios
- Privileged Identity Management requiere una licencia de Azure AD Premium P2

## <a name="assign-a-role"></a>Asignar un rol

1. Inicie sesión en el [Centro de administración de Azure AD](https://aad.portal.azure.com).

1. Seleccione **Azure Active Directory**.

1. Seleccione **Roles y administradores** para ver la lista de todos los roles disponibles.

    ![Captura de pantalla de la página Roles y administradores](./media/manage-roles-portal/roles-and-administrators.png)

1. Seleccione un rol para ver sus asignaciones.

    Para ayudarle a encontrar el rol que necesita, Azure AD puede mostrar los subconjuntos de los roles en función de las categorías de rol. Utilice el filtro **Tipo** para ver solo los roles del tipo seleccionado.

1. Seleccione **Agregar asignaciones** y, a continuación, seleccione los usuarios que desea asignar a este rol.

    Si lo que ve no concuerda con la siguiente imagen, lea la nota de [Privileged Identity Management (PIM)](#privileged-identity-management-pim) para comprobar si está usando PIM.

    ![lista de permisos para un rol de administrador](./media/manage-roles-portal/add-assignments.png)

1. Seleccione **Agregar** para asignar el rol.

## <a name="privileged-identity-management-pim"></a>Privileged Identity Management (PIM)

Puede seleccionar **Administrar en PIM** para otras funcionalidades de administración mediante [Azure AD Privileged Identity Management](../privileged-identity-management/pim-configure.md). Los administradores de roles con privilegios pueden cambiar las asignaciones "Permanentes" (siempre activas en el rol) a "Elegibles" (en el rol solo cuando se elevan). Si no tiene Privileged Identity Management, todavía puede seleccionar **Administrar en PIM** para registrarse en una versión de prueba. Privileged Identity Management requiere una [licencia de Azure AD Premium P2](../privileged-identity-management/subscription-requirements.md).

![Captura de pantalla que muestra la página "Administrador de usuarios - Asignaciones" con la acción "Administrar en PIM" seleccionada.](./media/manage-roles-portal/member-list-pim.png)

Si es un administrador global o un administrador de roles con privilegios, puede agregar o quitar miembros, filtrar la lista o seleccionar un miembro fácilmente para ver sus roles asignados activos.

> [!Note]
> Si tiene una licencia Azure AD Premium P2 y ya usa Privileged Identity Management, todas las tareas de administración de roles se realizan ahí y no en Azure AD.
>
> ![Roles de Azure AD administrados en PIM para los usuarios que ya usan PIM y tienen una licencia Premium P2](./media/manage-roles-portal/pim-manages-roles-for-p2.png)

## <a name="next-steps"></a>Pasos siguientes

* No dude en compartir con nosotros en el [foro de roles administrativos de Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032).
* Para más información acerca de los roles, consulte [Roles integrados en Azure AD](permissions-reference.md).
* Para conocer los permisos predeterminados de usuario, vea una [comparación de los permisos predeterminados de usuario miembro e invitado](../fundamentals/users-default-permissions.md).
