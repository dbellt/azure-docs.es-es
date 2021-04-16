---
title: Opciones avanzadas de firma de certificados de token SAML para aplicaciones de Azure AD
description: Aprenda a usar opciones avanzadas de firma de certificados en el token SAML para aplicaciones previamente integradas en Azure Active Directory
services: active-directory
author: iantheninja
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 03/25/2019
ms.author: iangithinji
ms.reviewer: jeedes
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1bb58f59b3796311f52d78ef87c8918f9888c152
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107377877"
---
# <a name="advanced-certificate-signing-options-in-the-saml-token-for-gallery-apps-in-azure-active-directory"></a>Opciones avanzadas de firma de certificados en el token SAML para aplicaciones previamente integradas en Azure Active Directory

En la actualidad Azure Active Directory (Azure AD) es compatible con miles de aplicaciones previamente integradas de la galería de aplicaciones de Azure Active Directory. Más de 500 de estas aplicaciones admiten el inicio de sesión único mediante el protocolo [Lenguaje de marcado de aserción de seguridad](https://wikipedia.org/wiki/Security_Assertion_Markup_Language) (SAML) 2.0, como la aplicación [NetSuite](https://azuremarketplace.microsoft.com/marketplace/apps/aad.netsuite). Cuando un cliente se autentica en una aplicación a través de Azure AD con SAML, Azure AD envía un token a la aplicación (mediante HTTP POST). Después, la aplicación valida y usa el token para iniciar la sesión del cliente, en lugar de solicitar un nombre de usuario y una contraseña. Estos tokens SAML se firman con el certificado único que se genera en Azure AD y mediante algoritmos estándar específicos.

Azure AD usa algunos de los valores predeterminados para las aplicaciones de la galería. Los valores predeterminados se configuran según los requisitos de la aplicación.

En Azure AD, puede configurar las opciones de firma de certificado y el algoritmo correspondiente.

## <a name="certificate-signing-options"></a>Opciones de firma de certificado

Azure AD admite tres opciones avanzadas de firma de certificado:

* **Firmar aserción SAML** Esta opción predeterminada se establece para la mayoría de las aplicaciones de la galería. Si se selecciona esta opción, Azure AD como proveedor de identidades (IdP) firma la aserción SAML y el certificado con el certificado [X509](https://wikipedia.org/wiki/X.509) de la aplicación.

* **Firmar respuesta SAML** Si se selecciona esta opción, Azure AD como IdP firma la respuesta SAML con el certificado X509 de la aplicación.

* **Firmar respuesta y aserción SAML** Si se selecciona esta opción, Azure AD como IdP firma todo el token SAML con el certificado X509 de la aplicación.

## <a name="certificate-signing-algorithms"></a>Algoritmos de firma de certificado

Azure AD admite dos algoritmos de firmas (o algoritmos hash seguros) para firmar la respuesta SAML:

* **SHA-256**. Azure AD utiliza este algoritmo predeterminado para firmar la respuesta SAML. Es el algoritmo más reciente y se considera más seguro que SHA-1. La mayoría de las aplicaciones admiten el algoritmo SHA-256. Si una aplicación solo admite SHA-1 como algoritmo de firma, puede cambiarlo. En caso contrario, se recomienda utilizar el algoritmo SHA-256 para firmar la respuesta SAML.

* **SHA-1**. Es un algoritmo más antiguo y se considera menos seguro que SHA-256. Si la aplicación admite solo este algoritmo de firma, puede seleccionar esta opción en la lista desplegable **Algoritmo de firma**. Azure AD firma entonces la respuesta SAML con el algoritmo SHA-1.

## <a name="change-certificate-signing-options-and-signing-algorithm"></a>Cambio de las opciones de firma de certificados y del algoritmo de firma

Para cambiar las opciones de firma de certificados SAML y el algoritmo de firma de certificados de una aplicación, seleccione la aplicación en cuestión:

1. En el [portal de Azure Active Directory](https://aad.portal.azure.com/), inicie sesión en su cuenta. Aparecerá la página del **Centro de administración de Azure Active Directory**.
1. En el panel izquierdo, seleccione **Aplicaciones empresariales**. Aparecerá una lista de las aplicaciones empresariales de su cuenta.
1. Seleccione una aplicación. Aparecerá una página de información general de la aplicación.

   ![Ejemplo: Página Información general de aplicación](./media/certificate-signing-options/application-overview-page.png)

Después, cambie las opciones de firma de certificados en el token SAML de esa aplicación:

1. En el panel izquierdo de la página de información general de la aplicación, seleccione **Inicio de sesión único**.
1. Si se muestra la página **Configurar el inicio de sesión único con SAML (versión preliminar)**, vaya al paso 5.
1. Si no aparece la página **Seleccione un método de inicio de sesión único**, seleccione **Cambiar los modos de inicio de sesión único** para que se muestre.
1. En la página **Seleccione un método de inicio de sesión único**, haga clic en **SAML**, si está disponible. (Si **SAML** no está disponible, la aplicación no es compatible con SAML, por lo que puede omitir el resto de este procedimiento y el artículo).
1. En la página **Configurar el inicio de sesión único con SAML (versión preliminar)** , busque el encabezado **Certificado de firma de SAML** y seleccione el icono **Editar** (con forma de lápiz). Aparecerá la página **Certificado de firma de SAML**.

   ![Ejemplo: Página Certificado de firma de SAML](./media/certificate-signing-options/saml-signing-page.png)

1. En la lista desplegable **Opción de firma**, seleccione **Firmar respuesta SAML**, **Firmar aserción SAML** o **Firmar respuesta y aserción SAML**. Las descripciones de estas opciones aparecen en la sección [Opciones de firma de certificado](#certificate-signing-options), anteriormente en este artículo.
1. En la lista desplegable **Algoritmo de firma**, elija **SHA-1** o **SHA-256**. Las descripciones de estas opciones aparecen en la sección [Algoritmos de firma de certificado](#certificate-signing-algorithms), anteriormente en este artículo.
1. Si está satisfecho con las opciones que ha seleccionado, haga clic en **Guardar** para aplicar la nueva configuración del certificado de firma de SAML. En caso contrario, seleccione **X** para descartar los cambios.

## <a name="next-steps"></a>Pasos siguientes

* [Configuración del inicio de sesión único en aplicaciones que no están en la galería de aplicaciones de Azure Active Directory](./configure-saml-single-sign-on.md)
* [Solución de problemas del inicio de sesión único basado en SAML](./debug-saml-sso-issues.md)