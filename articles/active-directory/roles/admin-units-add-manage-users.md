---
title: 'Adición, eliminación y enumeración de usuarios en una unidad administrativa: Azure Active Directory | Microsoft Docs'
description: Administración de usuarios y sus permisos de rol en una unidad administrativa en Azure Active Directory
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
ms.openlocfilehash: 101f1452f547f195288e2d22bc516880100c7735
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107496833"
---
# <a name="add-and-manage-users-in-an-administrative-unit-in-azure-active-directory"></a>Adición y administración de usuarios en las unidades administrativas en Azure Active Directory

En Azure Active Directory (Azure AD), puede agregar usuarios a una unidad administrativa para tener un ámbito de control administrativo más pormenorizado.

Para prepararse para el uso de PowerShell y Microsoft Graph para la administración de unidades administrativas, consulte [Introducción](admin-units-manage.md#get-started).

## <a name="add-users-to-an-administrative-unit"></a>Adición de usuarios a una unidad administrativa

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

Puede asignar usuarios a unidades administrativas de forma individual o en masa.

- Asignar usuarios individuales a partir de un perfil de usuario:

   1. Inicie sesión en el [Centro de administración de Azure AD](https://portal.azure.com) con permisos de Administrador de roles con privilegios.

   1. Seleccione **Usuarios** y, luego, elija el usuario que se va a asignar a una unidad administrativa para abrir el perfil de usuario.
   
   1. Seleccione **Unidades administrativas**. 
   
   1. Para asignar el usuario a una o más unidades administrativas, seleccione **Asignar a la unidad administrativa** y, luego, en el panel derecho, elija las unidades administrativas a las que quiere asignar el usuario.

       ![Captura de pantalla del panel "Unidades administrativas" para asignar un usuario a una unidad administrativa.](./media/admin-units-add-manage-users/assign-users-individually.png)

- Asignar usuarios individuales a partir de una unidad administrativa:

   1. Inicie sesión en el [Centro de administración de Azure AD](https://portal.azure.com) con permisos de Administrador de roles con privilegios.
   1. Seleccione **Unidades administrativas** y, luego, elija la unidad administrativa a la que se va a asignar el usuario.
   1. Seleccione **Todos los usuarios**, elija **Agregar miembro** y, luego, en el panel **Agregar miembro**, seleccione uno o más usuarios que quiera asignar a la unidad administrativa.

        ![Captura de pantalla del panel "Usuarios" de la unidad administrativa para asignar un usuario a una unidad administrativa.](./media/admin-units-add-manage-users/assign-to-admin-unit.png)

- Asignar usuarios en una operación en masa:

   1. Inicie sesión en el [Centro de administración de Azure AD](https://portal.azure.com) con permisos de Administrador de roles con privilegios.

   1. Seleccione **Unidades administrativas**.

   1. Seleccione la unidad administrativa a la que quiere agregar usuarios.

   1. Seleccione **Usuarios** > **Bulk activities** > **Bulk add members** (Actividades en masa > Agregar miembros en masa). Después, puede descargar la plantilla de valores separados por coma (CSV) y editar el archivo. El formato es sencillo y requiere que se agregue un nombre principal de usuario único en cada línea. Una vez que el archivo esté listo, guárdelo en una ubicación adecuada y, luego, cárguelo como parte de este paso.

      ![Captura de pantalla del panel "Usuarios" para asignar usuarios a una unidad administrativa en una operación en masa.](./media/admin-units-add-manage-users/bulk-assign-to-admin-unit.png)

### <a name="use-powershell"></a>Uso de PowerShell

En PowerShell, use el cmdlet `Add-AzureADAdministrativeUnitMember` en el ejemplo siguiente para agregar el usuario a la unidad administrativa. Se toman como argumentos el identificador de objeto de la unidad administrativa a la que quiere agregar el usuario y el identificador de objeto del usuario que quiere agregar. Cambie la sección resaltada según sea necesario para su entorno específico.

```powershell
$adminUnitObj = Get-AzureADMSAdministrativeUnit -Filter "displayname eq 'Test administrative unit 2'"
$userObj = Get-AzureADUser -Filter "UserPrincipalName eq 'bill@example.onmicrosoft.com'"
Add-AzureADMSAdministrativeUnitMember -Id $adminUnitObj.Id -RefObjectId $userObj.ObjectId
```


### <a name="use-microsoft-graph"></a>Uso de Microsoft Graph

Reemplace el marcador de posición por la información de prueba y ejecute el siguiente comando:

Request

```http
POST /administrativeUnits/{admin-unit-id}/members/$ref
```

Cuerpo

```http
{
  "@odata.id":"https://graph.microsoft.com/v1.0/users/{user-id}"
}
```

Ejemplo

```http
{
  "@odata.id":"https://graph.microsoft.com/v1.0/users/john@example.com"
}
```

## <a name="view-a-list-of-administrative-units-for-a-user"></a>Visualización de una lista de unidades administrativas de un usuario

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

En Azure Portal, puede abrir el perfil de un usuario mediante los pasos siguientes:

1. Vaya a **Azure AD** y seleccione **Usuarios**.

1. Seleccione el usuario cuyo perfil quiere ver.

1. Seleccione **Unidades administrativas** para mostrar la lista de unidades administrativas a las que se ha asignado el usuario.

   ![Captura de pantalla de las unidades administrativas a las que se ha asignado un usuario.](./media/admin-units-add-manage-users/list-user-admin-units.png)

### <a name="use-powershell"></a>Uso de PowerShell

Ejecute el siguiente comando:

```powershell
Get-AzureADMSAdministrativeUnit | where { Get-AzureADMSAdministrativeUnitMember -Id $_.ObjectId | where {$_.RefObjectId -eq $userObjId} }
```

> [!NOTE]
> De forma predeterminada, `Get-AzureADAdministrativeUnitMember` devuelve solo 100 miembros de una unidad administrativa. Para recuperar más miembros, puede agregar `"-All $true"`.

### <a name="use-microsoft-graph"></a>Uso de Microsoft Graph

Reemplace el marcador de posición por la información de prueba y ejecute el siguiente comando:

```http
https://graph.microsoft.com/v1.0/users/{user-id}/memberOf/$/Microsoft.Graph.AdministrativeUnit
```

## <a name="remove-a-single-user-from-an-administrative-unit"></a>Eliminación de un solo usuario de una unidad administrativa

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

Hay dos maneras de quitar un usuario de una unidad administrativa: 

* En Azure Portal, vaya a **Azure AD** y seleccione **Usuarios**. 
  1. Seleccione el usuario para abrir su perfil. 
  1. Seleccione la unidad administrativa de la que quiere quitar el usuario y, luego, elija **Quitar de la unidad administrativa**.

     ![Captura de pantalla que muestra cómo quitar un usuario de una unidad administrativa en el panel del perfil del usuario.](./media/admin-units-add-manage-users/user-remove-admin-units.png)

* En Azure Portal, vaya a **Azure AD** y, luego, seleccione **Unidades administrativas**.
  1. Seleccione la unidad administrativa de la que quiere quitar el usuario. 
  1. Seleccione el usuario y, luego, elija **Quitar miembro**.
  
     ![Captura de pantalla que muestra cómo quitar un usuario en el nivel de unidad administrativa.](./media/admin-units-add-manage-users/admin-units-remove-user.png)

### <a name="use-powershell"></a>Uso de PowerShell

Ejecute el siguiente comando:

```powershell
Remove-AzureADMSAdministrativeUnitMember -Id $adminUnitId -MemberId $memberUserObjId
```

### <a name="use-microsoft-graph"></a>Uso de Microsoft Graph

Reemplace los marcadores de posición por la información de prueba y ejecute el siguiente comando:

```http
https://graph.microsoft.com/v1.0/directory/administrativeUnits/{admin-unit-id}/members/{user-id}/$ref
```

## <a name="remove-multiple-users-as-a-bulk-operation"></a>Eliminación de varios usuarios en una operación en masa

Para quitar varios usuarios de una unidad administrativa, haga lo siguiente:

1. En Azure Portal, vaya a **Azure AD**.

1. Seleccione **Unidades administrativas** y, luego, elija aquella de la que quiere quitar usuarios. 

1. Seleccione **Bulk remove members** (Quitar miembros en masa) y, luego, descargue la plantilla de CSV que usará para mostrar los usuarios que quiere quitar.

   ![Captura de pantalla que muestra el vínculo "Bulk remove members" (Quitar miembros en masa) del panel "Usuarios".](./media/admin-units-add-manage-users/bulk-user-remove.png)

1. Edite la plantilla CSV descargada con las entradas de usuario correspondientes. No quite las dos primeras filas de la plantilla. Agregue un nombre principal de usuario (UPN) en cada fila.

   ![Captura de pantalla de un archivo CSV editado para quitar usuarios de una unidad administrativa en masa.](./media/admin-units-add-manage-users/bulk-user-entries.png)

1. Guarde los cambios, cargue el archivo y, luego, seleccione **Enviar**.

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de un rol a una unidad administrativa](admin-units-assign-roles.md)
- [Adición de grupos a una unidad administrativa](admin-units-add-manage-groups.md)
