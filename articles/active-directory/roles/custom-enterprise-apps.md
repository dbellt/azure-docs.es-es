---
title: Creación de roles personalizados para administrar aplicaciones empresariales en Azure Active Directory
description: Creación y asignación de roles de Azure AD personalizados para el acceso a aplicaciones empresariales en Azure Active Directory
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
ms.openlocfilehash: db22b44e032261d138d74e34340dca6fcaf75779
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110092879"
---
# <a name="create-custom-roles-to-manage-enterprise-apps-in-azure-active-directory"></a>Creación de roles personalizados para administrar aplicaciones empresariales en Azure Active Directory

En este artículo se explica cómo crear un rol personalizado con permisos para administrar asignaciones de aplicaciones empresariales para usuarios y grupos en Azure Active Directory (Azure AD). Para ver los elementos de las asignaciones de roles y el significado de términos tales como subtipo, permiso y conjunto de propiedades, consulte la [información general sobre roles personalizados](custom-overview.md).

## <a name="prerequisites"></a>Requisitos previos

- Una licencia de Azure AD Premium P1 o P2
- Administrador global o administrador de roles con privilegios
- Módulo de AzureADPreview al usar PowerShell
- Consentimiento del administrador al usar Probador de Graph para Microsoft Graph API

Para obtener más información, consulte [Requisitos previos para usar PowerShell o Probador de Graph](prerequisites.md).

## <a name="enterprise-app-role-permissions"></a>Permisos de rol de aplicación empresarial

En este artículo se describen dos permisos de aplicación empresarial. En todos los ejemplos se usa el permiso de actualización.

* Para leer las asignaciones de usuarios y grupos en el ámbito, conceda el permiso `microsoft.directory/servicePrincipals/appRoleAssignedTo/read`
* Para administrar las asignaciones de usuarios y grupos en el ámbito, conceda el permiso `microsoft.directory/servicePrincipals/appRoleAssignedTo/update`

Al conceder el permiso de actualización, el usuario asignado puede administrar asignaciones de usuarios y grupos en aplicaciones empresariales. El ámbito de las asignaciones de usuarios y grupos se puede conceder para una sola aplicación o para todas las aplicaciones. Si se concede en el nivel de toda la organización, el usuario asignado puede administrar asignaciones en todas las aplicaciones. Si se realiza en un nivel de aplicación, el usuario asignado solo puede administrar asignaciones en la aplicación especificada.

La concesión del permiso de actualización se realiza en dos pasos:

1. Creación de un rol personalizado con permiso `microsoft.directory/servicePrincipals/appRoleAssignedTo/update`
1. Conceda permisos a usuarios o grupos para administrar asignaciones de usuarios y grupos en las aplicaciones empresariales. En ese caso, se puede establecer el ámbito en el nivel de toda la organización o en una sola aplicación.

## <a name="azure-portal"></a>Portal de Azure

### <a name="create-a-new-custom-role"></a>Creación de un rol personalizado

>[!NOTE]
> Los roles personalizados se crean y administran en el nivel de toda la organización y solo están disponibles en la página de información general de la organización.

1. Inicie sesión en el [Centro de administración de Azure AD](https://aad.portal.azure.com).
1. Seleccione **Azure Active Directory**, elija **Roles y administradores** y, luego, seleccione **Nuevo rol personalizado**.

    ![Agregue un nuevo rol personalizado de la lista roles en Azure AD](./media/custom-enterprise-apps/new-custom-role.png)

1. En la pestaña **Aspectos básicos**, escriba "Administrar asignaciones de usuario y de grupo" como nombre del rol y "Conceder permisos para administrar asignaciones de usuarios y grupos" como descripción del rol y, luego, seleccione **Siguiente**.

    ![Suministro de un nombre y una descripción del rol personalizado](./media/custom-enterprise-apps/role-name-and-description.png)

1. En la pestaña **Permisos**, escriba "microsoft.directory/servicePrincipals/appRoleAssignedTo/update" en el cuadro de búsqueda, active las casillas situadas junto a los permisos deseados y, por último, seleccione **Siguiente**.

    ![Adición de permisos al rol personalizado](./media/custom-enterprise-apps/role-custom-permissions.png)

1. En la pestaña **Revisar y crear**, revise los detalles y seleccione **Crear**.

    ![Ya puede crear el rol personalizado.](./media/custom-enterprise-apps/role-custom-create.png)

### <a name="assign-the-role-to-a-user-using-the-azure-portal"></a>Asignación del rol a un usuario mediante Azure Portal

1. Inicie sesión en el [Centro de administración de Azure AD](https://aad.portal.azure.com).
1. Seleccione **Azure Active Directory** y elija **Roles y administradores**.
1. Seleccione el rol **Grant permissions to manage user and group assignments** (Conceder permisos para administrar asignaciones de usuarios y grupos).

    ![Abra "Roles y administradores" y busque el rol personalizado.](./media/custom-enterprise-apps/select-custom-role.png)

1. Seleccione **Agregar asignación**, seleccione el usuario que quiera y haga clic en **Seleccionar** para agregar la asignación de rol al usuario.

    ![Adición de una asignación para el rol personalizado al usuario](./media/custom-enterprise-apps/assign-user-to-role.png)

#### <a name="assignment-tips"></a>Sugerencias de asignación

* Para conceder permisos a los usuarios asignados para administrar el acceso de usuarios y grupos en todas las aplicaciones empresariales de toda la organización, empiece por la lista **Roles y administradores** de toda la organización en la página **Información general** de Azure AD de su organización.
* Para conceder permisos a los usuarios asignados para administrar el acceso de usuarios y grupos en una aplicación empresarial específica, vaya a esa aplicación en Azure AD y abra la lista **Roles y administradores** de esa aplicación. Seleccione el nuevo rol personalizado y complete la asignación de usuario o grupo. Los usuarios asignados solo pueden administrar el acceso de usuarios y grupos en la aplicación específica.
* Para probar la asignación de roles personalizados, inicie sesión como usuario asignado y abra la página **Usuarios y grupos** de la aplicación para comprobar que la opción **Agregar usuario** está habilitada.

    ![Comprobación de los permisos de usuario](./media/custom-enterprise-apps/verify-user-permissions.png)

## <a name="powershell"></a>PowerShell

Para más información, consulte [Creación y asignación de un rol personalizado](custom-create.md) y [Asignación de roles personalizados con ámbito de recurso mediante PowerShell](custom-assign-powershell.md).

### <a name="create-a-custom-role"></a>Crear un rol personalizado

Cree un nuevo rol mediante el siguiente script de PowerShell:

```PowerShell
# Basic role information
$description = "Manage user and group assignments"
$displayName = "Can manage user and group assignments for Applications"
$templateId = (New-Guid).Guid

# Set of permissions to grant
$allowedResourceAction = @("microsoft.directory/servicePrincipals/appRoleAssignedTo/update")
$resourceActions = @{'allowedResourceActions'= $allowedResourceAction}
$rolePermission = @{'resourceActions' = $resourceActions}
$rolePermissions = $rolePermission

# Create new custom admin role
$customRole = New-AzureADMSRoleDefinition -RolePermissions $rolePermissions -DisplayName $displayName -Description $description -TemplateId $templateId -IsEnabled $true
```

### <a name="assign-the-custom-role"></a>Asignación del rol personalizado

Asigne el rol mediante el siguiente script de PowerShell.

```powershell
# Get the user and role definition you want to link
$user = Get-AzureADUser -Filter "userPrincipalName eq 'chandra@example.com'"
$roleDefinition = Get-AzureADMSRoleDefinition -Filter "displayName eq 'Manage user and group assignments'"

# Get app registration and construct resource scope for assignment.
$appRegistration = Get-AzureADApplication -Filter "displayName eq 'My Filter Photos'"
$resourceScope = '/' + $appRegistration.objectId

# Create a scoped role assignment
$roleAssignment = New-AzureADMSRoleAssignment -ResourceScope $resourceScope -RoleDefinitionId $roleDefinition.Id -PrincipalId $user.objectId
```

## <a name="microsoft-graph-api"></a>Microsoft Graph API

Cree un rol personalizado mediante el ejemplo proporcionado en Microsoft Graph API. Para más información, consulte [Creación y asignación de un rol personalizado](custom-create.md) y [Asignación de roles de administrador personalizados mediante Microsoft Graph API](custom-assign-graph.md).

La solicitud HTTP para crear el rol personalizado.

```HTTP
POST
https://graph.microsoft.com/beta/roleManagement/directory/roleDefinitionsIsEnabled $true
{
    "description":"Can manage user and group assignments for Applications.",
    "displayName":" Manage user and group assignments",
    "isEnabled":true,
    "rolePermissions":
    [
        {
            "resourceActions":
            {
                "allowedResourceActions":
                [
                    "microsoft.directory/servicePrincipals/appRoleAssignedTo/update"
                ]
            },
            "condition":null
        }
    ],
    "templateId":"<PROVIDE NEW GUID HERE>",
    "version":"1"
}
```

### <a name="assign-the-custom-role-using-the-microsoft-graph-api"></a>Asignación del rol personalizado mediante Microsoft Graph API

La asignación de roles combina un identificador de entidad de seguridad (que puede ser un usuario o una entidad de servicio), un identificador de definición de rol y un ámbito de recurso de Azure AD. Para más información sobre los elementos de una asignación de roles, consulte la [información general sobre roles personalizados](custom-overview.md).

Solicitud HTTP para asignar un rol personalizado.

```HTTP
POST https://graph.microsoft.com/beta/roleManagement/directory/roleAssignments

{
    "principalId":"<PROVIDE OBJECTID OF USER TO ASSIGN HERE>",
    "roleDefinitionId":"<PROVIDE OBJECTID OF ROLE DEFINITION HERE>",
    "resourceScopes":["/"]
}
```

## <a name="next-steps"></a>Pasos siguientes

* [Exploración de los permisos de rol personalizado disponibles para aplicaciones empresariales](custom-enterprise-app-permissions.md)
