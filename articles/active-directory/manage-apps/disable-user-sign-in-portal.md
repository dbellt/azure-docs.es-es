---
title: Deshabilitación de los inicios de sesión de los usuarios de una aplicación empresarial en Azure AD
description: Cómo deshabilitar una aplicación empresarial para que ningún usuario pueda iniciar sesión en ella en Azure Active Directory
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 04/12/2019
ms.author: mtillman
ms.reviewer: asteen
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: ac4970a3dc62331e4a9f56c069f32def7351a8bb
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112081432"
---
# <a name="disable-user-sign-ins-for-an-enterprise-app-in-azure-active-directory"></a>Deshabilitación de los inicios de sesión de usuario de una aplicación empresarial en Azure Active Directory

Deshabilitar una aplicación empresarial para que ningún usuario pueda iniciar sesión en ella en Azure Active Directory (Azure AD) es fácil. Debe tener los permisos adecuados para administrar la aplicación empresarial. Y tiene que ser el administrador global del directorio.

## <a name="how-do-i-disable-user-sign-ins"></a>¿Cómo puedo deshabilitar los inicios de sesión de usuario?

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta que tenga el rol de administrador global en el directorio.
1. Seleccione **Todos los servicios**, escriba **Azure Active Directory** en el cuadro de texto y, después, seleccione **Entrar**.
1. En el panel **Azure Active Directory** -  **_nombreDelDirectorio_ *_ (es decir, el panel de Azure AD del directorio que está administrando), seleccione _* Aplicaciones empresariales**.
1. En el panel **Aplicaciones empresariales - Todas las aplicaciones**, verá una lista de las aplicaciones que puede administrar. Seleccione una aplicación.
1. En el panel **appname** _pane (es decir, el panel con el nombre de la aplicación seleccionada en el título), seleccione _*Propiedades**.
1. En el panel ***appname** _ - _ *Propiedades*, en **¿Habilitado para que los usuarios inicien sesión?** , seleccione **No**.
1. Haga clic en el comando **Guardar** .

## <a name="use-azure-ad-powershell-to-disable-an-unlisted-app"></a>Uso de Azure AD PowerShell para deshabilitar una aplicación que no está en la lista

Si conoce el AppId de una aplicación que no aparece en la lista de aplicaciones empresariales (por ejemplo, porque eliminó la aplicación o porque la entidad de servicio todavía no se ha creado debido a que la aplicación está autorizada previamente por Microsoft), puede crear manualmente la entidad de servicio para la aplicación y, a continuación, deshabilitarla mediante el [cmdlet de AzureAD PowerShell](/powershell/module/azuread/New-AzureADServicePrincipal).

```PowerShell
# The AppId of the app to be disabled
$appId = "{AppId}"

# Check if a service principal already exists for the app
$servicePrincipal = Get-AzureADServicePrincipal -Filter "appId eq '$appId'"
if ($servicePrincipal) {
    # Service principal exists already, disable it
    Set-AzureADServicePrincipal -ObjectId $servicePrincipal.ObjectId -AccountEnabled $false
} else {
    # Service principal does not yet exist, create it and disable it at the same time
    $servicePrincipal = New-AzureADServicePrincipal -AppId $appId -AccountEnabled $false
}
```

## <a name="next-steps"></a>Pasos siguientes

* [Ver todos mis grupos](../fundamentals/active-directory-groups-view-azure-portal.md)
* [Asignar un usuario o grupo a una aplicación empresarial](assign-user-or-group-access-portal.md)
* [Quitar una asignación de usuario o grupo de una aplicación empresarial](./assign-user-or-group-access-portal.md)
* [Cambio del nombre o el logotipo de una aplicación empresarial](./add-application-portal-configure.md)
