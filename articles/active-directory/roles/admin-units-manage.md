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
ms.date: 11/04/2020
ms.author: rolyon
ms.reviewer: anandy
ms.custom: oldportal;it-pro;
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6fad9356d3379e76aa259d67711d18f14a4e266f
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107505282"
---
# <a name="manage-administrative-units-in-azure-active-directory"></a>Administración de unidades administrativas en Azure Active Directory

Para un control administrativo más pormenorizado en Azure Active Directory (Azure AD), puede asignar a los usuarios un rol de Azure AD con un ámbito limitado a una o varias unidades administrativas.

## <a name="get-started"></a>Introducción

1. Para ejecutar consultas a partir de las siguientes instrucciones mediante el [Probador de Graph](https://aka.ms/ge), asegúrese de cumplir con lo siguiente:

    a. En Azure Portal, vaya a Azure AD. 
    
    b. En la lista de aplicaciones, seleccione **Probador de Graph**.
    
    c. En el panel **Permisos**, seleccione **Conceder consentimiento de administrador para Probador de Graph**.

    ![Captura de pantalla en la que se muestra el vínculo "Conceder consentimiento de administrador para Probador de Graph".](./media/admin-units-manage/select-graph-explorer.png)


1. Use [Azure AD PowerShell](https://www.powershellgallery.com/packages/AzureAD/).

## <a name="add-an-administrative-unit"></a>Incorporación de una unidad administrativa

Puede agregar una unidad administrativa mediante Azure Portal o PowerShell.

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

1. En Azure Portal, vaya a Azure AD. A continuación, en el panel de la izquierda, seleccione **Unidades administrativas**.

    ![Captura de pantalla en la que se muestra el vínculo "Unidades administrativas" en Azure AD.](./media/admin-units-manage/nav-to-admin-units.png)

1. Seleccione el botón **Agregar** en la parte superior del panel y, a continuación, en el cuadro **Nombre**, escriba el nombre de la unidad administrativa. Si quiere, agregue una descripción de la unidad administrativa.

    ![Captura de pantalla en la que se muestra el botón Agregar y el cuadro Nombre para escribir el nombre de la unidad administrativa.](./media/admin-units-manage/add-new-admin-unit.png)

1. Seleccione el botón azul **Agregar** para finalizar la unidad administrativa.

### <a name="use-powershell"></a>Uso de PowerShell

Instale [Azure AD PowerShell](https://www.powershellgallery.com/packages/AzureAD/) antes de intentar ejecutar los siguientes comandos:

```powershell
Connect-AzureAD
New-AzureADMSAdministrativeUnit -Description "West Coast region" -DisplayName "West Coast"
```

Puede modificar los valores entre comillas, según sea necesario.

### <a name="use-microsoft-graph"></a>Uso de Microsoft Graph

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

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

1. En Azure Portal, vaya a **Azure AD** y, luego, seleccione **Unidades administrativas**. 
1. Seleccione la unidad administrativa que quiera eliminar y, a continuación, seleccione **Eliminar**. 
1. Para confirmar que quiere eliminar la unidad administrativa, seleccione **Sí**. Se elimina la unidad administrativa.

![Captura de pantalla del botón Eliminar de la unidad administrativa y de la ventana de confirmación](./media/admin-units-manage/select-admin-unit-to-delete.png)

### <a name="use-powershell"></a>Uso de PowerShell

```powershell
$adminUnitObj = Get-AzureADMSAdministrativeUnit -Filter "displayname eq 'DeleteMe Admin Unit'"
Remove-AzureADMSAdministrativeUnit -Id $adminUnitObj.Id
```

Puede modificar los valores entre comillas, según sea necesario para el entorno específico.

### <a name="use-the-graph-api"></a>Uso de Graph API

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
