---
title: Ocultación de una aplicación empresarial de la experiencia del usuario en Azure AD
description: Cómo ocultar una aplicación empresarial de la experiencia del usuario en los paneles de acceso de Azure Active Directory o los iniciadores de Microsoft 365.
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 03/25/2020
ms.author: mtillman
ms.reviewer: kasimpso
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7b1d590c19c158532df8577adc1bddb4fab19fd8
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112078013"
---
# <a name="hide-enterprise-applications-from-end-users-in-azure-active-directory"></a>Ocultación de aplicaciones empresariales de usuarios finales en Azure Active Directory

Instrucciones sobre cómo ocultar las aplicaciones del panel MyApps o del iniciador de Microsoft 365 de los usuarios finales. Aunque una aplicación se oculte, los usuarios seguirán teniendo los permisos para ella. 

## <a name="prerequisites"></a>Requisitos previos

Se requieren privilegios de administrador de aplicaciones para ocultar una aplicación del panel MyApps y del iniciador de Microsoft 365.

Se requieren privilegios de administrador global para ocultar todas las aplicaciones de Microsoft 365.


## <a name="hide-an-application-from-the-end-user"></a>Ocultar una aplicación al usuario final
Siga estos pasos para ocultar una aplicación del panel MyApps y del iniciador de aplicaciones de Microsoft 365.

1.  Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global de su directorio.
2.  Seleccione **Azure Active Directory**.
3.  Seleccione **Aplicaciones empresariales**. Se abrirá la hoja **Aplicaciones empresariales - Todas las aplicaciones**.
4.  En **Tipo de aplicación**, seleccione **Aplicaciones empresariales**, si aún no está seleccionado.
5.  Busque la aplicación que quiere ocultar y haga clic en la aplicación.  Se abrirá la información general de la aplicación.
6.  Haga clic en **Propiedades**. 
7.  Para la pregunta **¿Es visible para los usuarios?** , haga clic en **No**.
8.  Haga clic en **Save**(Guardar).

> [!NOTE]
> Estas instrucciones solo se aplican a las aplicaciones empresariales.

## <a name="use-azure-ad-powershell-to-hide-an-application"></a>Uso de Azure AD PowerShell para ocultar una aplicación

Para ocultar una aplicación del panel Mis aplicaciones, puede agregar manualmente la etiqueta HideApp a la entidad de servicio de la aplicación. Ejecute los siguientes comandos de [Azure AD PowerShell](/powershell/module/azuread/#service_principals) para establecer la propiedad **¿Es visible para los usuarios?** en **No**. 

```PowerShell
Connect-AzureAD

$objectId = "<objectId>"
$servicePrincipal = Get-AzureADServicePrincipal -ObjectId $objectId
$tags = $servicePrincipal.tags
$tags += "HideApp"
Set-AzureADServicePrincipal -ObjectId $objectId -Tags $tags
```

## <a name="hide-microsoft-365-applications-from-the-myapps-panel"></a>Ocultación de las aplicaciones de Microsoft 365 del panel MyApps

Siga estos pasos para ocultar todas las aplicaciones de Microsoft 365 del panel MyApps. Las aplicaciones siguen siendo visibles en el portal de Office 365.

1.  Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global de su directorio.
2.  Seleccione **Azure Active Directory**.
3.  Seleccione **Usuarios**.
4.  Seleccione **Configuración de usuario**.
5.  En **Aplicaciones empresariales**, haga clic en **Administrar cómo los usuarios finales inician y ven sus aplicaciones**.
6.  En **Los usuarios solo pueden ver las aplicaciones de Office 365 en el Portal de Office 365**, haga clic en **Sí**.
7.  Haga clic en **Save**(Guardar).

## <a name="next-steps"></a>Pasos siguientes
* [Ver todos mis grupos](../fundamentals/active-directory-groups-view-azure-portal.md)
* [Asignar un usuario o grupo a una aplicación empresarial](assign-user-or-group-access-portal.md)
* [Quitar una asignación de usuario o grupo de una aplicación empresarial](./assign-user-or-group-access-portal.md)
* [Cambio del nombre o el logotipo de una aplicación empresarial](./add-application-portal-configure.md)
