---
title: Visualización de los roles asignados a un grupo en Azure Active Directory | Microsoft Docs
description: Obtenga información sobre cómo se pueden ver los roles asignados a un grupo mediante el centro de administración de Azure AD. La visualización de los grupos y los roles asignados son los permisos de usuario predeterminados.
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
ms.openlocfilehash: 5e44dc442180631b0ebe11cae374a59fed37cc13
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110085750"
---
# <a name="view-roles-assigned-to-a-group-in-azure-active-directory"></a>Visualización de los roles asignados a un grupo en Azure Active Directory

En esta sección se describe cómo se pueden ver los roles asignados a un grupo mediante el centro de administración de Azure AD. La visualización de los grupos y los roles asignados son los permisos de usuario predeterminados.

## <a name="prerequisites"></a>Requisitos previos

- Módulo de AzureADPreview al usar PowerShell
- Consentimiento del administrador al usar Probador de Graph para Microsoft Graph API

Para más información, consulte [Requisitos previos para usar PowerShell o Probador de Graph](prerequisites.md).

## <a name="azure-portal"></a>Portal de Azure

1. Inicie sesión en el [Centro de administración de Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview).

1. Seleccione el grupo que le interese.

1. Seleccione **Roles asignados**. Así podrá ver todos los roles de Azure AD asignados a este grupo.

   ![Visualización de todos los roles asignados a un grupo seleccionado](./media/groups-view-assignments/view-assignments.png)

## <a name="powershell"></a>PowerShell

### <a name="get-object-id-of-the-group"></a>Obtención del id. de objeto del grupo

```powershell
Get-AzureADMSGroup -SearchString "Contoso_Helpdesk_Administrators"
```

### <a name="view-role-assignment-to-a-group"></a>Visualización de la asignación de roles a un grupo

```powershell
Get-AzureADMSRoleAssignment -Filter "principalId eq '<object id of group>" 
```

## <a name="microsoft-graph-api"></a>Microsoft Graph API

### <a name="get-object-id-of-the-group"></a>Obtención del id. de objeto del grupo

```http
GET https://graph.microsoft.com/beta/groups?$filter=displayName+eq+'Contoso_Helpdesk_Administrator'
```

### <a name="get-role-assignments-to-a-group"></a>Obtención de la asignación de roles a un grupo

```http
GET https://graph.microsoft.com/beta/roleManagement/directory/roleAssignments?$filter=principalId eq
```

## <a name="next-steps"></a>Pasos siguientes

- [Uso de grupos en la nube para administrar asignaciones de roles](groups-concept.md)
- [Solución de problemas de roles asignados a grupos en la nube](groups-faq-troubleshooting.md)
