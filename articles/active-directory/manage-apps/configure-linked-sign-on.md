---
title: Descripción del inicio de sesión vinculado en Azure Active Directory
description: Comprenda el inicio de sesión vinculado en Azure Active Directory.
services: active-directory
author: iantheninja
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: conceptual
ms.workload: identity
ms.date: 07/30/2020
ms.author: iangithinji
ms.reviewer: arvinh,luleon
ms.openlocfilehash: 6ed6f6b69326157573ea043457dbc8d8a3079146
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107374581"
---
# <a name="understand-linked-sign-on"></a>Descripción del inicio de sesión vinculado

En la [serie de guías de inicio rápido](view-applications-portal.md) sobre la administración de aplicaciones, aprendió a usar Azure AD como proveedor de identidades (IdP) para una aplicación. En la guía de inicio rápido, se configura SSO basado en SAML o en OIDC. Otra opción es **Vinculado**. Este artículo explica con más detalle la opción Vinculado.

La opción **Vinculado** permite configurar la ubicación de destino cuando un usuario selecciona la aplicación en el portal de Office 365 o [Aplicaciones](https://myapps.microsoft.com/) de la organización.

Estos son algunos de los escenarios comunes en los que la opción de vínculo resulta de gran utilidad:
- Incorporación de un vínculo a una aplicación web personalizada que actualmente use la federación, como Servicios de federación de Active Directory (AD FS).
- Incorporación de vínculos profundos a páginas específicas de SharePoint o a otras páginas web que desee que aparezcan en los paneles de acceso de su usuario.
- Incorporación de un vínculo a una aplicación que no requiera autenticación. 
 
 La opción **Vinculado** no proporciona la funcionalidad de inicio de sesión a través de las credenciales de Azure AD. Sin embargo, puede seguir usando algunas de las otras características de las **aplicaciones empresariales**. Por ejemplo, puede usar registros de auditoría y agregar un logotipo personalizado y un nombre de la aplicación.

## <a name="before-you-begin"></a>Antes de empezar

Para adquirir conocimientos rápidamente, consulte la [serie de guías de inicio rápido](view-applications-portal.md) sobre la administración de aplicaciones. En el inicio rápido, donde configura el inicio de sesión único, también encontrará la opción **Vinculado**. 

La opción **Vinculado** no proporciona la funcionalidad de inicio de sesión a través de Azure AD. La opción solo establece la ubicación a la que se enviarán los usuarios cuando seleccionen la aplicación en [Aplicaciones](https://myapps.microsoft.com/) o el iniciador de aplicaciones de Microsoft 365.  Como el inicio de sesión no proporciona la funcionalidad de inicio de sesión a través de Azure AD, el acceso condicional no está disponible para las aplicaciones configuradas con el inicio de sesión único vinculado.

> [!IMPORTANT] 
> Hay escenarios en los que la opción **Inicio de sesión único** no estará al desplazarse por una aplicación en las **aplicaciones empresariales**. 
>
> Por ejemplo, si la aplicación se registró mediante **Registros de aplicaciones**, la funcionalidad de inicio de sesión único estará configurada para usar OAuth de OIDC de manera predeterminada. En este caso, la opción de **Inicio de sesión único** no se mostrará en la navegación en **Aplicaciones empresariales**. Cuando use **Registros de aplicaciones** para agregar su aplicación personalizada, configure las opciones en el archivo de manifiesto. Para obtener más información sobre el archivo de manifiesto, consulte [Manifiesto de aplicación de Azure Active Directory](../develop/reference-app-manifest.md). Para obtener más información sobre los estándares de SSO, consulte [Autenticación y autorización mediante la Plataforma de identidad de Microsoft](../develop/authentication-vs-authorization.md#authentication-and-authorization-using-the-microsoft-identity-platform). 
>
> Otros escenarios en los que falta **inicio de sesión único** en la navegación incluyen cuándo se hospeda una aplicación en otro inquilino o si su cuenta no tiene los permisos necesarios (Administrador global, Administrador de aplicaciones en la nube, Administrador de la aplicación o propietario de la entidad de servicio). Los permisos también pueden provocar un escenario en el que puede abrir **Inicio de sesión único** pero no podrá guardar. Para más información acerca de los roles administrativos de Azure AD, consulte (https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles).

### <a name="configure-link"></a>Configuración de un vínculo

A fin de establecer un vínculo para una aplicación, seleccione **Vinculado** en la página **Inicio de sesión único**. A continuación, escriba el vínculo y seleccione **Guardar**. ¿Necesita un recordatorio de dónde encontrar estas opciones? Consulte la [serie de guías de inicio rápido](view-applications-portal.md).
 
Una vez que configure una aplicación, asigne usuarios y grupos a la misma. Al asignar usuarios, puede controlar cuándo aparecerá la aplicación en [Aplicaciones](https://myapps.microsoft.com/) o en el iniciador de aplicaciones de Microsoft 365.

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de usuarios y grupos a la aplicación](./assign-user-or-group-access-portal.md)
- [Configuración del aprovisionamiento automático de cuentas de usuario](../app-provisioning/configure-automatic-user-provisioning-portal.md)
