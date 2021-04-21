---
title: 'Adición, eliminación y enumeración de grupos en una unidad administrativa: Azure Active Directory | Microsoft Docs'
description: Administre grupos y sus permisos de rol en una unidad administrativa en Azure Active Directory.
services: active-directory
documentationcenter: ''
author: rolyon
manager: daveba
ms.service: active-directory
ms.topic: how-to
ms.subservice: roles
ms.workload: identity
ms.date: 03/10/2021
ms.author: rolyon
ms.reviewer: anandy
ms.custom: oldportal;it-pro;
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2ad8cce8375ecd670a481541a091e36aacb41240
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107505299"
---
# <a name="add-and-manage-groups-in-an-administrative-unit-in-azure-active-directory"></a>Adición y administración de grupos en una unidad administrativa en Azure Active Directory

En Azure Active Directory (Azure AD), puede agregar grupos a una unidad administrativa para tener un ámbito de control administrativo más pormenorizado.

Para prepararse para el uso de PowerShell y Microsoft Graph para la administración de unidades administrativas, consulte [Introducción](admin-units-manage.md#get-started).

## <a name="add-groups-to-an-administrative-unit"></a>Adición de grupos a una unidad administrativa

Puede agregar grupos a una unidad administrativa mediante Azure Portal, PowerShell o Microsoft Graph.

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

Solo puede asignar grupos individuales a una unidad administrativa. No hay ninguna opción para asignar grupos como una operación masiva. En Azure Portal, puede asignar un grupo a una unidad administrativa de dos formas:

* En el panel **Grupos**:

  1. En Azure Portal, vaya a **Azure AD**.
  1. Seleccione **Grupos** y, a continuación, seleccione el grupo que desea asignar a la unidad administrativa. 
  1. En el panel de la izquierda, seleccione **Unidades administrativas** para ver una lista de las unidades administrativas a las que está asignado el grupo. 

     ![Captura de pantalla del vínculo "Asignar a la unidad administrativa" en el panel "Unidades administrativas".](./media/admin-units-add-manage-groups/assign-to-group-1.png)

  1. Seleccione **Asignar a la unidad administrativa**.
  1. En el panel derecho, seleccione la unidad administrativa.

* En el panel **Unidades administrativas** > **Todos los grupos**:

  1. En Azure Portal, vaya a **Azure AD**.
  
  1. En el panel izquierdo, seleccione **Unidades administrativas** y, después, **Todos los grupos**. 
     Todos los grupos que ya están asignados a la unidad administrativa se muestran en el panel derecho. 

  1. En el panel **Grupos**, seleccione **Agregar**.
    En el panel derecho se enumeran todos los grupos disponibles en la organización de Azure AD. 

     ![Captura de pantalla del botón "Agregar" para agregar un grupo a una unidad administrativa.](./media/admin-units-add-manage-groups/assign-to-admin-unit.png)

  1. Seleccione uno o más grupos para asignarlos a la unidades administrativa y, después, haga clic en el botón **Seleccionar**.

### <a name="use-powershell"></a>Uso de PowerShell

En el ejemplo siguiente, use el cmdlet `Add-AzureADMSAdministrativeUnitMember` para agregar el grupo a la unidad administrativa. El id. de objeto de la unidad administrativa y el id. de objeto del grupo que se va a agregar se usan como argumentos. Cambie la sección resaltada según sea necesario para su entorno específico.


```powershell
$adminUnitObj = Get-AzureADMSAdministrativeUnit -Filter "displayname eq 'Test administrative unit 2'"
$GroupObj = Get-AzureADGroup -Filter "displayname eq 'TestGroup'"
Add-AzureADMSAdministrativeUnitMember -Id $adminUnitObj.Id -RefObjectId $GroupObj.ObjectId
```

### <a name="use-microsoft-graph"></a>Uso de Microsoft Graph

Ejecute los comandos siguientes:

Request

```http
POST /administrativeUnits/{admin-unit-id}/members/$ref
```

Cuerpo

```http
{
"@odata.id":"https://graph.microsoft.com/v1.0/groups/{group-id}"
}
```

Ejemplo

```http
{
"@odata.id":"https://graph.microsoft.com/v1.0/groups/871d21ab-6b4e-4d56-b257-ba27827628f3"
}
```

## <a name="view-a-list-of-groups-in-an-administrative-unit"></a>Visualización de una lista de grupos en una unidad administrativa

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

1. En Azure Portal, vaya a **Azure AD**.

1. En el panel izquierdo, seleccione **Unidades administrativas** y luego la unidad administrativa cuyos grupos desea ver. De forma predeterminada, la opción **Todos los usuarios** está seleccionada en el panel izquierdo. 

1. En el panel izquierdo, seleccione **Grupos**. En el panel de la derecha se muestra la lista de los grupos que son miembros de la unidad administrativa seleccionada.

   ![Captura de pantalla del panel "Grupos" que muestra una lista de grupos en una unidad administrativa.](./media/admin-units-add-manage-groups/list-groups-in-admin-units.png)

### <a name="use-powershell"></a>Uso de PowerShell

Para mostrar una lista de todos los miembros de la unidad administrativa, ejecute el siguiente comando: 

```powershell
$adminUnitObj = Get-AzureADMSAdministrativeUnit -Filter "displayname eq 'Test administrative unit 2'"
Get-AzureADMSAdministrativeUnitMember -Id $adminUnitObj.Id
```

Para mostrar todos los grupos que son miembros de la unidad administrativa, use el siguiente fragmento de código:

```powershell
foreach ($member in (Get-AzureADMSAdministrativeUnitMember -Id $adminUnitObj.Id)) 
{
if($member.ObjectType -eq "Group")
{
Get-AzureADGroup -ObjectId $member.ObjectId
}
}
```

### <a name="use-microsoft-graph"></a>Uso de Microsoft Graph

Ejecute el siguiente comando:

Request

```http
GET /directory/administrativeUnits/{admin-unit-id}/members/$/microsoft.graph.group
```

Cuerpo

```http
{}
```

## <a name="view-a-list-of-administrative-units-for-a-group"></a>Visualización de una lista de unidades administrativas de un grupo

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

1. En Azure Portal, vaya a **Azure AD**.

1. En el panel izquierdo, seleccione **Grupos** para mostrar una lista de grupos.

1. Seleccione un grupo para abrir su perfil. 

1. En el panel izquierdo, seleccione **Unidades administrativas** para ver todas las unidades administrativas en las que el grupo es un miembro.

   ![Captura de pantalla del panel "Unidades administrativas", donde se muestra una lista de las unidades administrativas a la que está asignado un grupo.](./media/admin-units-add-manage-groups/list-group-au.png)

### <a name="use-powershell"></a>Uso de PowerShell

Ejecute el siguiente comando:

```powershell
Get-AzureADMSAdministrativeUnit | where { Get-AzureADMSAdministrativeUnitMember -ObjectId $_.ObjectId | where {$_.ObjectId -eq $groupObjId} }
```

### <a name="use-microsoft-graph"></a>Uso de Microsoft Graph

Ejecute el siguiente comando:

```http
https://graph.microsoft.com/v1.0/groups/{group-id}/memberOf/$/Microsoft.Graph.AdministrativeUnit
```

## <a name="remove-a-group-from-an-administrative-unit"></a>Eliminación de un grupo de una unidad administrativa

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

Puede quitar un grupo de una unidad administrativa en Azure Portal de dos maneras:

- Quitarlo de la información general de un grupo:

  1. En Azure Portal, vaya a **Azure AD**.
  1. En el panel izquierdo, seleccion **Grupos** y, después, abra el perfil para el grupo que quiere quitar de una unidad administrativa.
  1. En el panel izquierdo, seleccione **Unidades administrativas** para ver todas las unidades administrativas a las que está asignado el grupo. 
  1. Seleccione la unidad administrativa de la que quiere quitar el grupo y, a continuación, seleccione **Quitar de la unidad administrativa**.

     ![Captura de pantalla del panel "Unidades administrativas", donde se muestra una lista de los grupos asignados a la unidad administrativa seleccionada.](./media/admin-units-add-manage-groups/group-au-remove.png)

- Quitarlo de una unidad administrativa:

  1. En Azure Portal, vaya a **Azure AD**.
  1. En el panel izquierdo, seleccione **Unidades administrativas** y, después, seleccione la unidad administrativa a la que está asignado el grupo.
  1. En el panel izquierdo, seleccione **Grupos** para enumerar todos los grupos asignados a la unidad administrativa.
  1. Seleccione el grupo que quiera quitar y, a continuación, seleccione **Quitar grupos**.

    ![Captura de pantalla del panel "Grupos" que muestra una lista de grupos en una unidad administrativa.](./media/admin-units-add-manage-groups/list-groups-in-admin-units.png)

### <a name="use-powershell"></a>Uso de PowerShell

Ejecute el siguiente comando:

```powershell
Remove-AzureADMSAdministrativeUnitMember -ObjectId $adminUnitId -MemberId $memberGroupObjId
```

### <a name="use-microsoft-graph"></a>Uso de Microsoft Graph

Ejecute el siguiente comando:

```http
https://graph.microsoft.com/v1.0/directory/administrativeUnits/{admin-unit-id}/members/{group-id}/$ref
```

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de un rol a una unidad administrativa](admin-units-assign-roles.md)
- [Administración de los usuarios en una unidad administrativa](admin-units-add-manage-users.md)
