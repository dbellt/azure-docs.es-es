---
title: Enumeración de asignaciones de roles de Azure AD
description: Ahora puede ver y administrar los miembros de un rol de administrador de Azure Active Directory en el centro de administración de Azure Active Directory.
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
ms.openlocfilehash: f8fe57b5b6df03e135bf77e9064e14f376c57ac9
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110796364"
---
# <a name="list-azure-ad-role-assignments"></a>Enumeración de asignaciones de roles de Azure AD

En este artículo se describe cómo enumerar los roles que ha asignado en Azure Active Directory (Azure AD). En Azure Active Directory (Azure AD), los roles se pueden asignar a un ámbito de toda la organización o con un ámbito de aplicación única.

- Las asignaciones de roles en el ámbito de toda la organización se agregan y pueden verse en la lista de asignaciones de roles de aplicación únicas.
- Las asignaciones de roles en el ámbito de aplicación única no se agregan y no se pueden ver en la lista de asignaciones con un ámbito de toda la organización.

## <a name="prerequisites"></a>Prerrequisitos

- Módulo de AzureADPreview al usar PowerShell
- Consentimiento del administrador al usar Probador de Graph para Microsoft Graph API

Para más información, consulte [Requisitos previos para usar PowerShell o Probador de Graph](prerequisites.md).

## <a name="list-role-assignments-in-the-azure-portal"></a>Enumeración de asignaciones de roles en Azure Portal

En este procedimiento se describe cómo enumerar las asignaciones de roles con ámbito de toda la organización.

1. Inicie sesión en el [Centro de administración de Azure AD](https://aad.portal.azure.com).
1. Seleccione **Azure Active Directory**, elija **Roles y administradores** y, luego, seleccione un rol para abrirlo y ver sus propiedades.
1. Seleccione **Asignaciones** para enumerar las asignaciones de roles.

    ![Enumeración de asignaciones de roles y permisos al abrir un rol de la lista](./media/view-assignments/role-assignments.png)

## <a name="list-my-role-assignments"></a>Enumeraciñon de mis asignaciones de roles

También es fácil enumerar sus propios permisos. Seleccione **su rol** en la página **Roles y administradores** para ver los roles que tiene asignados actualmente.

## <a name="download-role-assignments"></a>Descarga de asignaciones de rol

Para descargar todas las asignaciones de un rol específico, en la página **Roles y administradores**, seleccione un rol y luego **Descargar asignaciones de roles**. Se descargará un archivo CSV en el que se enumeran las asignaciones en todos los ámbitos para ese rol.

![Descarga de todas las asignaciones de un rol](./media/view-assignments/download-role-assignments.png)

## <a name="list-role-assignments-using-powershell"></a>Enumeración de las asignaciones de roles con PowerShell

En esta sección se describe cómo ver las asignaciones de un rol con ámbito de toda la organización. Este artículo se usa el módulo [Azure Active Directory PowerShell, versión 2](/powershell/module/azuread/#directory_roles). Para ver las asignaciones con un ámbito de aplicación única con PowerShell, puede usar los cmdlets de [Asignación de roles personalizados con PowerShell](custom-assign-powershell.md).

Ejemplo de cómo enumerar las asignaciones de roles.

``` PowerShell
# Fetch list of all directory roles with object ID
Get-AzureADDirectoryRole

# Fetch a specific directory role by ID
$role = Get-AzureADDirectoryRole -ObjectId "5b3fe201-fa8b-4144-b6f1-875829ff7543"

# Fetch role membership for a role
Get-AzureADDirectoryRoleMember -ObjectId $role.ObjectId | Get-AzureADUser
```

## <a name="list-role-assignments-using-the-microsoft-graph-api"></a>Enumeración de las asignaciones de roles mediante Microsoft Graph API

En esa sección se describe cómo enumerar las asignaciones de roles con ámbito de toda la organización.  Para enumerar las asignaciones de roles con un ámbito de aplicación única mediante Graph API, puede usar las operaciones de [Asignación de roles personalizados con Graph API](custom-assign-graph.md).

La solicitud HTTP para obtener una asignación de roles para una definición de roles determinada.

GET

``` HTTP
https://graph.microsoft.com/beta/roleManagement/directory/roleAssignments&$filter=roleDefinitionId eq ‘<object-id-or-template-id-of-role-definition>’
```

Response

``` HTTP
HTTP/1.1 200 OK
{
    "id":"CtRxNqwabEKgwaOCHr2CGJIiSDKQoTVJrLE9etXyrY0-1"
    "principalId":"ab2e1023-bddc-4038-9ac1-ad4843e7e539",
    "roleDefinitionId":"3671d40a-1aac-426c-a0c1-a3821ebd8218",
    "resourceScopes":["/"]
}
```

## <a name="list-role-assignments-with-single-application-scope"></a>Enumeración de las asignaciones de roles con ámbito de aplicación única

En esa sección se describe cómo enumerar las asignaciones de roles con ámbito de aplicación única. Esta característica actualmente está en su versión preliminar pública.

1. Inicie sesión en el [Centro de administración de Azure AD](https://aad.portal.azure.com).
1. Seleccione **Registros de aplicaciones** y, luego, seleccione el registro de aplicación para ver sus propiedades. Es posible que tenga que seleccionar **Todas las aplicaciones** para ver la lista completa de los registros de aplicaciones de la organización de Azure AD.

    ![Creación o edición de registros de aplicaciones desde la página Registros de aplicaciones](./media/view-assignments/app-reg-all-apps.png)

1. En el registro de la aplicación, seleccione **Roles y administradores** y después elija un rol para ver sus propiedades.

    ![Enumeración de las asignaciones de roles de registro de aplicaciones desde la página Registros de aplicaciones](./media/view-assignments/app-reg-assignments.png)

1. Seleccione **Asignaciones** para enumerar las asignaciones de roles. Al abrir la página de asignaciones desde el registro de la aplicación, se muestran las asignaciones de roles que se limitan a este recurso de Azure AD.

    ![Enumeración de las asignaciones de roles de registro de aplicaciones desde las propiedades de un registro de aplicaciones](./media/view-assignments/app-reg-assignments-2.png)

## <a name="next-steps"></a>Pasos siguientes

* No dude en compartir con nosotros en el [foro de roles administrativos de Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032).
* Para más información acerca de los permisos de roles, consulte [Roles integrados en Azure AD](permissions-reference.md).
* Para conocer los permisos predeterminados de usuario, vea una [comparación de los permisos predeterminados de usuario miembro e invitado](../fundamentals/users-default-permissions.md).
