---
title: 'Asignación y enumeración de roles con ámbito de unidad administrativa: Azure Active Directory | Microsoft Docs'
description: Use unidades administrativas para restringir el ámbito de las asignaciones de roles en Azure Active Directory.
services: active-directory
documentationcenter: ''
author: rolyon
manager: daveba
ms.service: active-directory
ms.topic: how-to
ms.subservice: roles
ms.workload: identity
ms.date: 04/14/2021
ms.author: rolyon
ms.reviewer: anandy
ms.custom: oldportal;it-pro;
ms.collection: M365-identity-device-management
ms.openlocfilehash: e24bf5df84015ada6b62c35fdd29571c66e06ebd
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107505265"
---
# <a name="assign-scoped-roles-to-an-administrative-unit"></a>Asignación de roles con ámbito a una unidad administrativa

Para un control administrativo más pormenorizado en Azure Active Directory (Azure AD), puede asignar usuarios a un rol de Azure AD con un ámbito limitado a una o más unidades administrativas.

Vea [Introducción](admin-units-manage.md#get-started) para prepararse para el uso de PowerShell y Microsoft Graph para la administración de unidades administrativas.

## <a name="available-roles"></a>Roles disponibles

Rol  |  Descripción
----- |  -----------
Administrador de autenticación  |  Tiene acceso para ver, configurar y restablecer la información de los métodos de autenticación de cualquier usuario que no sea administrador solo en la unidad administrativa asignada.
Administrador de grupos  |  Puede administrar todos los aspectos de los grupos y la configuración de estos, como las directivas de nomenclatura y expiración, solo en la unidad administrativa asignada.
Administrador del departamento de soporte técnico  |  Puede restablecer contraseñas de usuarios que no son administradores y de administradores del departamento de soporte técnico solo en la unidad administrativa asignada.
Administrador de licencias  |  Puede asignar, quitar y actualizar las asignaciones de licencia solo dentro de la unidad administrativa.
Administrador de contraseñas  |  Puede restablecer contraseñas de usuarios que no son administradores y de administradores de contraseña solo en la unidad administrativa asignada.
Administrador de usuarios  |  Puede administrar todos los aspectos de usuarios y grupos, incluido el restablecimiento de contraseñas para administradores limitados solo en la unidad administrativa asignada.

## <a name="security-principals-that-can-be-assigned-to-a-scoped-role"></a>Entidades de seguridad que se pueden asignar a un rol con ámbito

Las siguientes entidades de seguridad pueden asignarse a un rol con un ámbito de unidad administrativa:

* Usuarios
* Grupos de nube asignables a roles (versión preliminar)
* Nombre de entidad de seguridad de servicio (SPN)

## <a name="assign-a-scoped-role"></a>Asignación de un rol con ámbito

Puede asignar un rol con ámbito mediante Azure Portal, PowerShell o Microsoft Graph.

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

1. En Azure Portal, vaya a **Azure AD**.

1. Seleccione **Unidades administrativas** y luego aquella a la que quiere asignar un ámbito de rol de usuario. 

1. En el panel izquierdo, seleccione **Roles y administradores** para ver todos los roles disponibles.

   ![Captura de pantalla del panel "Roles y administradores" para seleccionar una unidad administrativa cuyo ámbito de rol quiera asignar.](./media/admin-units-assign-roles/select-role-to-scope.png)

1. Seleccione el rol que se va a asignar y luego **Agregar asignaciones**. 

1. En el panel **Agregar asignaciones**, seleccione uno o más usuarios para asignar al rol.

   ![Seleccione el rol al que se agregará el ámbito y, a continuación, seleccione Agregar asignaciones.](./media/admin-units-assign-roles/select-add-assignment.png)

> [!Note]
> Para asignar un rol en una unidad administrativa mediante Azure AD Privileged Identity Management (PIM), vea [Asignación de roles de Azure AD en PIM](../privileged-identity-management/pim-how-to-add-role-to-user.md?tabs=new#assign-a-role-with-restricted-scope).

### <a name="use-powershell"></a>Uso de PowerShell

```powershell
$adminUser = Get-AzureADUser -ObjectId "Use the user's UPN, who would be an admin on this unit"
$role = Get-AzureADDirectoryRole | Where-Object -Property DisplayName -EQ -Value "User Administrator"
$adminUnitObj = Get-AzureADMSAdministrativeUnit -Filter "displayname eq 'The display name of the unit'"
$roleMember = New-Object -TypeName Microsoft.Open.MSGraph.Model.MsRoleMemberInfo
$roleMember.Id = $adminUser.ObjectId
Add-AzureADMSScopedRoleMembership -Id $adminUnitObj.Id -RoleId $role.ObjectId -RoleMemberInfo $roleMember
```

La sección resaltada se puede cambiar según sea necesario para el entorno específico.

### <a name="use-microsoft-graph"></a>Uso de Microsoft Graph

Request

```http
POST /directory/administrativeUnits/{admin-unit-id}/scopedRoleMembers
```
    
Cuerpo

```http
{
  "roleId": "roleId-value",
  "roleMemberInfo": {
    "id": "id-value"
  }
}
```

## <a name="view-a-list-of-the-scoped-admins-in-an-administrative-unit"></a>Visualización de una lista de administradores con ámbito en una unidad administrativa

Puede ver una lista de administradores con ámbito mediante Azure Portal, PowerShell o Microsoft Graph.

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

Todas las asignaciones de roles creadas con un ámbito de unidad administrativa se pueden ver en la sección [Unidades administrativas de Azure AD](https://ms.portal.azure.com/?microsoft_aad_iam_adminunitprivatepreview=true&microsoft_aad_iam_rbacv2=true#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AdminUnit). 

1. En Azure Portal, vaya a **Azure AD**.

1. En el panel izquierdo, seleccione **Unidades administrativas** y luego la unidad administrativa de la lista de asignaciones de roles que quiere ver. 

1. Seleccione **Roles y administradores** y abra un rol para ver las asignaciones de la unidad administrativa.

### <a name="use-powershell"></a>Uso de PowerShell

```powershell
$adminUnitObj = Get-AzureADMSAdministrativeUnit -Filter "displayname eq 'The display name of the unit'"
Get-AzureADMSScopedRoleMembership -Id $adminUnitObj.Id | fl *
```

Puede cambiar la sección resaltada según sea necesario para el entorno específico.

### <a name="use-microsoft-graph"></a>Uso de Microsoft Graph

Request

```http
GET /directory/administrativeUnits/{admin-unit-id}/scopedRoleMembers
```

Body

```http
{}
```

## <a name="next-steps"></a>Pasos siguientes

- [Uso de grupos en la nube para administrar asignaciones de roles](groups-concept.md)
- [Solución de problemas de roles asignados a grupos en la nube](groups-faq-troubleshooting.md)
