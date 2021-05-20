---
title: Publicación de una granja de servidores de SharePoint en el entorno local con Azure Active Directory Application Proxy
description: En este documento se explican los conceptos básicos sobre cómo integrar una granja de servidores de SharePoint en el entorno local con Azure Active Directory Application Proxy para SAML.
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-proxy
ms.workload: identity
ms.topic: how-to
ms.date: 04/27/2021
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: e23d8871b50827e76bb383c4a49475627a9ca4a9
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108764448"
---
# <a name="integrate-azure-active-directory-application-proxy-with-sharepoint-saml"></a>Integración de Azure Active Directory Application Proxy con SharePoint (SAML)

En esta guía paso a paso se explica cómo proteger el acceso al [entorno local de SharePoint (SAML) integrado de Azure Active Directory](../saas-apps/sharepoint-on-premises-tutorial.md) mediante Azure AD Application Proxy, donde los usuarios de su organización (Azure AD, B2B) se conectan a SharePoint a través de Internet.

> [!NOTE]
> Si no está familiarizado con Azure AD Application Proxy y quiere obtener más información, consulte [Acceso remoto a aplicaciones locales mediante Azure AD Application Proxy](./application-proxy.md).

Hay tres ventajas principales de esta configuración:

- Azure AD Application Proxy garantiza que el tráfico autenticado pueda alcanzar la red interna y SharePoint.
- Los usuarios pueden acceder a los sitios de SharePoint como de costumbre sin usar una VPN.
- Puede controlar el acceso por asignación de usuario en el nivel de Azure AD Application Proxy y puede aumentar la seguridad con características de Azure AD como el acceso condicional y la autenticación multifactor (MFA).

Este proceso requiere dos aplicaciones empresariales. Una es una instancia local de SharePoint que se publica desde la galería a la lista de aplicaciones SaaS administradas. La segunda es una aplicación local (aplicación ajena a la galería) que usará para publicar la primera aplicación empresarial de la galería.

## <a name="prerequisites"></a>Prerrequisitos

Para realizar la configuración, necesita los siguientes recursos:
 - Una granja de servidores de SharePoint 2013, o posterior. La granja de servidores de SharePoint se debe [integrar con Azure AD](../saas-apps/sharepoint-on-premises-tutorial.md).
 - Un inquilino de Azure AD con un plan que incluya el proxy de aplicación. Obtenga más información acerca de los [planes y precios de Azure AD](https://azure.microsoft.com/pricing/details/active-directory/).
 - Un [dominio comprobado personalizado ](../fundamentals/add-custom-domain.md) en el inquilino de Azure AD. El dominio comprobado debe coincidir con el sufijo de la dirección URL de SharePoint.
 - Se requiere un certificado SSL. Consulte los detalles en [Publicación de dominios personalizados](./application-proxy-configure-custom-domain.md).
 - Los usuarios de Active Directory local deben estar sincronizados con Azure AD Connect y deben estar configurados para [iniciar sesión en Azure](../hybrid/plan-connect-user-signin.md). 
 - En el caso de usuarios solo en la nube y usuarios B2B, debe [conceder acceso a una cuenta de invitado a la instancia de SharePoint local en Azure Portal](../saas-apps/sharepoint-on-premises-tutorial.md#manage-guest-users-access).
 - Un conector del proxy de aplicación instalado y en ejecución en una máquina que se encuentre dentro del dominio corporativo.


## <a name="step-1-integrate-sharepoint-on-premises-with-azure-ad"></a>Paso 1: Integración de SharePoint local con Azure AD

1. Configure la aplicación local de SharePoint. Para más información, consulte [Tutorial: Integración del inicio de sesión único de Azure Active Directory con SharePoint local](../saas-apps/sharepoint-on-premises-tutorial.md).
2. Valide la configuración antes de pasar al paso siguiente. Para realizar la validación, intente acceder a SharePoint local desde la red interna y confirme que es accesible internamente.


## <a name="step-2-publish-the-sharepoint-on-premises-application-with-application-proxy"></a>Paso 2: Publicación de la aplicación local de SharePoint con Application Proxy

En este paso, creará en su inquilino de Azure Active Directory una aplicación que usa Application Proxy. Establezca la dirección URL externa y especifique la dirección URL interna. Ambas direcciones se usarán más adelante en SharePoint.

> [!NOTE]
> Las direcciones URL internas y externas deben coincidir con la **dirección URL de inicio de sesión** de la configuración de la aplicación basada en SAML del paso 1.

   ![Captura de pantalla que muestra el valor de la URL de inicio de sesión.](./media/application-proxy-integrate-with-sharepoint-server/sso-url-saml.png)


 1. Cree una nueva aplicación de Azure AD Application Proxy con un dominio personalizado. Para obtener instrucciones paso a paso, consulte [Dominios personalizados en Azure AD Application Proxy](./application-proxy-configure-custom-domain.md).

    - Dirección URL interna: https://portal.contoso.com/
    - Dirección URL externa: https://portal.contoso.com/
    - Autenticación previa: Azure Active Directory
    - Traducción de direcciones URL en encabezados: no
    - Traducción de direcciones URL en el cuerpo de la aplicación: no

        ![Captura de pantalla que muestra las opciones que se usan para crear la aplicación.](./media/application-proxy-integrate-with-sharepoint-server/create-application-azure-active-directory.png)

2. Asigne los [mismos grupos](../saas-apps/sharepoint-on-premises-tutorial.md#grant-permissions-to-a-security-group) que asignó a la aplicación de la galería de SharePoint local.

3. Finalmente, vaya a la sección de **Propiedades** y establezca la opción **¿Es visible para los usuarios?** en **No**. Esta opción garantiza que solo el icono de la primera aplicación aparecerá en el portal Aplicaciones (https://myapplications.microsoft.com).

   ![Captura de pantalla que muestra dónde se puede establecer la opción ¿Es visible para los usuarios?](./media/application-proxy-integrate-with-sharepoint-server/configure-properties.png)
 
## <a name="step-3-test-your-application"></a>Paso 3: Prueba de la aplicación

Con un explorador de un equipo en una red externa, vaya a la dirección URL (el enlace que configuró durante el paso de publicación). Asegúrese de que puede iniciar sesión con la cuenta de prueba que configuró.
