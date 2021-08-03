---
title: 'Incorporación y eliminación de unidades administrativas: Azure Active Directory | Microsoft Docs'
description: Uso de unidades administrativas para restringir el ámbito de los permisos de rol en Azure Active Directory.
services: active-directory
documentationcenter: ''
author: rolyon
manager: daveba
ms.service: active-directory
ms.topic: how-to
ms.subservice: roles
ms.workload: identity
ms.date: 05/14/2021
ms.author: rolyon
ms.reviewer: anandy
ms.custom: oldportal;it-pro;
ms.collection: M365-identity-device-management
ms.openlocfilehash: 78468c9528c4502ce691dec183c261b9636325f8
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110094337"
---
# <a name="manage-administrative-units-in-azure-active-directory"></a>Administración de unidades administrativas en Azure Active Directory

Para un control administrativo más pormenorizado en Azure Active Directory (Azure AD), puede asignar a los usuarios un rol de Azure AD con un ámbito limitado a una o varias unidades administrativas.


## <a name="prerequisites"></a>Requisitos previos

- Una licencia de Azure AD Premium P1 o P2 para cada administrador de la unidad administrativa
- Licencias de Azure AD Free para los miembros de la unidad administrativa
- Administrador global o administrador de roles con privilegios
- Módulo AzureAD al usar PowerShell
- Consentimiento del administrador al usar Probador de Graph para Microsoft Graph API

Para más información, consulte [Requisitos previos para usar PowerShell o Probador de Graph](prerequisites.md).

## <a name="add-an-administrative-unit"></a>Incorporación de una unidad administrativa

Puede agregar una unidad administrativa mediante Azure Portal o PowerShell.

### <a name="azure-portal"></a>Portal de Azure

1. En Azure Portal, vaya a Azure AD. A continuación, en el panel de la izquierda, seleccione **Unidades administrativas**.

    ![Captura de pantalla en la que se muestra el vínculo "Unidades administrativas" en Azure AD.](./media/admin-units-manage/nav-to-admin-units.png)

1. Seleccione el botón **Agregar** en la parte superior del panel y, a continuación, en el cuadro **Nombre**, escriba el nombre de la unidad administrativa. Si quiere, agregue una descripción de la unidad administrativa.

    ![Captura de pantalla en la que se muestra el botón Agregar y el cuadro Nombre para escribir el nombre de la unidad administrativa.](./media/admin-units-manage/add-new-admin-unit.png)

1. Seleccione el botón azul **Agregar** para finalizar la unidad administrativa.

### <a name="powershell"></a>PowerShell

```powershell
Connect-AzureAD
New-AzureADMSAdministrativeUnit -Description "West Coast region" -DisplayName "West Coast"
```

Puede modificar los valores entre comillas, según sea necesario.

### <a name="microsoft-graph-api"></a>Microsoft Graph API

Request

```http
POST /administrativeUnits
```

Cuerpo

```http
{
  "displayName": "North America Operations",
  "description": "North America Operations administration"
}
```

## <a name="remove-an-administrative-unit"></a>Eliminación de una unidad administrativa

En Azure AD, puede quitar una unidad administrativa que ya no necesite como unidad de ámbito para los roles administrativos.

### <a name="azure-portal"></a>Portal de Azure

1. En Azure Portal, vaya a **Azure AD** y, luego, seleccione **Unidades administrativas**. 
1. Seleccione la unidad administrativa que quiera eliminar y, a continuación, seleccione **Eliminar**. 
1. Para confirmar que quiere eliminar la unidad administrativa, seleccione **Sí**. Se elimina la unidad administrativa.

![Captura de pantalla del botón Eliminar de la unidad administrativa y de la ventana de confirmación](./media/admin-units-manage/select-admin-unit-to-delete.png)

### <a name="powershell"></a>PowerShell

```powershell
$adminUnitObj = Get-AzureADMSAdministrativeUnit -Filter "displayname eq 'DeleteMe Admin Unit'"
Remove-AzureADMSAdministrativeUnit -Id $adminUnitObj.Id
```

Puede modificar los valores entre comillas, según sea necesario para el entorno específico.

### <a name="microsoft-graph-api"></a>Microsoft Graph API

Request

```http
DELETE /administrativeUnits/{admin-unit-id}
```

Body

```http
{}
```

## <a name="next-steps"></a>Pasos siguientes

* [Administración de los usuarios en una unidad administrativa](admin-units-add-manage-users.md)
* [Administración de los grupos en una unidad administrativa](admin-units-add-manage-groups.md)
