---
title: Visualización de aplicaciones con el inquilino de Azure Active Directory para la administración de identidades
description: Comprenda cómo visualizar todas las aplicaciones con el inquilino de Azure Active Directory para la administración de identidades.
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: reference
ms.date: 01/07/2021
ms.author: mtillman
ms.openlocfilehash: 95a5264582087d748c1cb0e6b924f14f34728ab6
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112080712"
---
# <a name="viewing-apps-using-your-azure-ad-tenant-for-identity-management"></a>Visualización de las aplicaciones con el inquilino de Azure AD para la administración de identidades
En [Serie de guías de inicio rápido sobre la administración de aplicaciones](view-applications-portal.md) se explican los aspectos básicos. Aquí aprenderá cómo visualizar todas las aplicaciones con el inquilino de Azure AD para la administración de identidades. En este artículo se profundiza un poco más en los tipos de aplicaciones que encontrará.

## <a name="why-does-a-specific-application-appear-in-my-all-applications-list"></a>¿Por qué aparece una aplicación específica en mi lista de todas las aplicaciones?
Cuando se filtra por **todas las aplicaciones**, la **lista** **Todas las aplicaciones** muestra todos los objetos de entidad de servicio del inquilino. Los objetos de entidad de servicio pueden aparecer en esta lista de varias formas:
- Cuando agrega cualquier aplicación desde la galería de aplicaciones, incluidas:
   - **Azure AD: Aplicaciones empresariales**. Aplicaciones agregadas al inquilino mediante la opción **Aplicaciones empresariales** en el portal de Azure AD. Normalmente, las aplicaciones se integran con el estándar SAML.
   - **Azure AD: Registros de aplicaciones**. Aplicaciones agregadas al inquilino con la opción **Registros de aplicaciones** en el portal de Azure AD. Normalmente, aplicaciones desarrolladas de forma personalizada con los estándares Open ID Connect y OAuth.
   - **Aplicaciones de proxy de aplicación**: una aplicación que se ejecuta en su entorno local en la que quiere que se proporcione inicio de sesión seguro externamente.
- Al registrarse o iniciar sesión en una aplicación de terceros integrada con Azure Active Directory. Un ejemplo de esto es [Smartsheet](https://app.smartsheet.com/b/home) o [DocuSign](https://www.docusign.net/member/MemberLogin.aspx).
- Aplicaciones de Microsoft, como Microsoft 365.
- Al agregar un nuevo registro de aplicación mediante la creación de una aplicación desarrollada de forma personalizada con el [Registro de aplicaciones](../develop/quickstart-register-app.md).
- Al agregar un nuevo registro de aplicación mediante la creación de una aplicación desarrollada de forma personalizada con el [Portal de registro de aplicaciones V2.0](../develop/quickstart-register-app.md).
- Al agregar una aplicación, el desarrollo se lleva a cabo con los [métodos de autenticación de ASP.NET](https://www.asp.net/visual-studio/overview/2013/creating-web-projects-in-visual-studio#orgauthoptions) o los [servicios conectados](https://devblogs.microsoft.com/visualstudio/connecting-to-cloud-services/) de Visual Studio.
- Al crear un objeto de entidad de servicio mediante el [módulo de PowerShell de Azure AD](/powershell/azure/active-directory/install-adv2).
- Al [dar su consentimiento a una aplicación](../develop/howto-convert-app-to-be-multi-tenant.md) como administrador para que utilice datos de su inquilino.
- Cuando un [usuario da su consentimiento a una aplicación](../develop/howto-convert-app-to-be-multi-tenant.md) para que utilice datos de su inquilino.
- Al habilitar determinados servicios que almacenan datos en el inquilino. Un ejemplo de esto es Password Reset, que se modela como entidad de servicio para almacenar la directiva de restablecimiento de contraseña de forma segura.

Para averiguar cómo y por qué las aplicaciones se agregan a su directorio, lea [Cómo se agregan aplicaciones a Azure AD](../develop/active-directory-how-applications-are-added.md).

## <a name="next-steps"></a>Pasos siguientes
[Administración de aplicaciones con Azure Active Directory](what-is-application-management.md)
