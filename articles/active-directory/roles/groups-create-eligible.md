---
title: Creación de un grupo para asignar roles en Azure Active Directory | Microsoft Docs
description: Aprenda a crear un grupo al que se pueden asignar roles en Azure AD. Administre roles de Azure en Azure Portal, PowerShell o Graph API.
services: active-directory
author: rolyon
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: article
ms.date: 05/14/2021
ms.author: rolyon
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4fb616bce2f169061a6384148e3cbbe463c83be8
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110085913"
---
# <a name="create-a-role-assignable-group-in-azure-active-directory"></a>Creación de un grupo al que se pueden asignar roles en Azure Active Directory

Solo puede asignar un rol a un grupo que se ha creado con la propiedad "isAssignableToRole" establecida en True o que se ha creado en Azure Portal con la opción **Azure AD roles can be assigned to the group** (Los roles de Azure AD se pueden asignar a un grupo) activada. Este atributo de grupo convierte el grupo en uno de los que se puede asignar a un rol en Azure Active Directory (Azure AD). En este artículo se describe cómo crear este tipo especial de grupo. **Nota:** Un grupo con la propiedad isAssignableToRole establecida en true no puede ser de tipo de pertenencia dinámica. Para más información, consulte [Uso de grupos para administrar asignaciones de roles en Azure AD](groups-concept.md).

## <a name="prerequisites"></a>Requisitos previos

- Una licencia de Azure AD Premium P1 o P2
- Administrador global o administrador de roles con privilegios
- Módulo de AzureADPreview al usar PowerShell
- Consentimiento del administrador al usar Probador de Graph para Microsoft Graph API

Para más información, consulte [Requisitos previos para usar PowerShell o Probador de Graph](prerequisites.md).

## <a name="azure-portal"></a>Portal de Azure

1. Inicie sesión en el [Centro de administración de Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview).
1. Seleccione **Grupos** > **Todos los grupos** > **Nuevo grupo**.

    [![Apertura de Azure Active Directory para crear un nuevo grupo.](./media/groups-create-eligible/new-group.png "Apertura de Azure Active Directory para crear un nuevo grupo.")](./media/groups-create-eligible/new-group.png#<lightbox>)

1. En la pestaña **Nuevo grupo**, proporcione el tipo, el nombre y la descripción del grupo.
1. Active **Los roles de Azure AD se pueden asignar a un grupo**. Este modificador solo es visible para los administradores de roles con privilegios y los administradores globales, ya que estos son los dos únicos roles que pueden establecer el modificador.

    [![Hacer que el nuevo grupo sea válido para la asignación de roles](./media/groups-create-eligible/eligible-switch.png "Hacer que el nuevo grupo sea válido para la asignación de roles")](./media/groups-create-eligible/eligible-switch.png#<lightbox>)

1. Seleccione los miembros y propietarios del grupo. También tiene la opción de asignar roles al grupo, pero no es necesario asignar un rol aquí.

    [![Adición de miembros al grupo al que se pueden asignar roles y asignación de roles.](./media/groups-create-eligible/specify-members.png "Adición de miembros al grupo al que se pueden asignar roles y asignación de roles.")](./media/groups-create-eligible/specify-members.png#<lightbox>)

1. Una vez especificados los miembros y los propietarios, seleccione **Crear**.

    [![El botón Crear se encuentra en la parte inferior de la página.](./media/groups-create-eligible/create-button.png "El botón Crear se encuentra en la parte inferior de la página.")](./media/groups-create-eligible/create-button.png#<lightbox>)

El grupo se crea con los roles que le haya asignado.

## <a name="powershell"></a>PowerShell

### <a name="create-a-group-that-can-be-assigned-to-role"></a>Creación de un grupo que se puede asignar a un rol

```powershell
$group = New-AzureADMSGroup -DisplayName "Contoso_Helpdesk_Administrators" -Description "This group is assigned to Helpdesk Administrator built-in role in Azure AD." -MailEnabled $true -SecurityEnabled $true -MailNickName "contosohelpdeskadministrators" -IsAssignableToRole $true
```

Para este tipo de grupo, `isPublic` siempre será False e `isSecurityEnabled` siempre será True.

### <a name="copy-one-groups-users-and-service-principals-into-a-role-assignable-group"></a>Copia de los usuarios y las entidades de servicio de un grupo a un grupo al que se pueden asignar roles

```powershell
#Basic set up
Install-Module -Name AzureADPreview
Import-Module -Name AzureADPreview
Get-Module -Name AzureADPreview

#Connect to Azure AD. Sign in as Privileged Role Administrator or Global Administrator. Only these two roles can create a role-assignable group.
Connect-AzureAD

#Input variabled: Existing group
$idOfExistingGroup = "14044411-d170-4cb0-99db-263ca3740a0c"

#Input variables: New role-assignable group
$groupName = "Contoso_Bellevue_Admins"
$groupDescription = "This group is assigned to Helpdesk Administrator built-in role in Azure AD."
$mailNickname = "contosobellevueadmins"

#Create new security group which is a role assignable group. For creating a Microsoft 365 group, set GroupTypes="Unified" and MailEnabled=$true
$roleAssignablegroup = New-AzureADMSGroup -DisplayName $groupName -Description $groupDescription -MailEnabled $false -MailNickname $mailNickname -SecurityEnabled $true -IsAssignableToRole $true

#Get details of existing group
$existingGroup = Get-AzureADMSGroup -Id $idOfExistingGroup
$membersOfExistingGroup = Get-AzureADGroupMember -ObjectId $existingGroup.Id

#Copy users and service principals from existing group to new group
foreach($member in $membersOfExistingGroup){
if($member.ObjectType -eq 'User' -or $member.ObjectType -eq 'ServicePrincipal'){
Add-AzureADGroupMember -ObjectId $roleAssignablegroup.Id -RefObjectId $member.ObjectId
}
}
```

## <a name="microsoft-graph-api"></a>Microsoft Graph API

### <a name="create-a-role-assignable-group-in-azure-ad"></a>Creación de un grupo al que se pueden asignar roles en Azure AD

```http
POST https://graph.microsoft.com/beta/groups
{
  "description": "This group is assigned to Helpdesk Administrator built-in role of Azure AD.",
  "displayName": "Contoso_Helpdesk_Administrators",
  "groupTypes": [
    "Unified"
  ],
  "isAssignableToRole": true,
  "mailEnabled": true,
  "securityEnabled": true,
  "mailNickname": "contosohelpdeskadministrators",
  "visibility" : "Private"
}
```

Para este tipo de grupo, `isPublic` siempre será False e `isSecurityEnabled` siempre será True.

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de un rol a un grupo en la nube](groups-assign-role.md)
- [Uso de grupos en la nube para administrar asignaciones de roles](groups-concept.md)
- [Solución de problemas de roles asignados a grupos en la nube](groups-faq-troubleshooting.md)
