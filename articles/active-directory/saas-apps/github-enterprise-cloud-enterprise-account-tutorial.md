---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con GitHub Enterprise Cloud - Enterprise Account | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y GitHub Enterprise Cloud - Enterprise Account.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/25/2021
ms.author: jeedes
ms.openlocfilehash: 0ccd7886007177e32506ce0c3db489826a8a708d
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110479992"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-github-enterprise-cloud---enterprise-account"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con GitHub Enterprise Cloud - Enterprise Account

En este tutorial, aprenderá a integrar la cuenta de GitHub Enterprise Cloud - Enterprise Account con Azure Active Directory (Azure AD). Al integrar la cuenta de GitHub Enterprise Cloud - Enterprise Account con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a una cuenta de GitHub Enterprise y a cualquier organización de la cuenta de empresa.


## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una [cuenta de GitHub Enterprise](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-your-enterprise/about-enterprise-accounts).
* Una cuenta de usuario de GitHub que sea un propietario de la cuenta de empresa. 

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* GitHub Enterprise Cloud - Enterprise Account admite el SSO iniciado por **SP** e **IDP**.
* GitHub Enterprise Cloud - Enterprise Account admite el aprovisionamiento de usuarios **Just-in-Time**.

## <a name="adding-github-enterprise-cloud---enterprise-account-from-the-gallery"></a>Adición de GitHub Enterprise Cloud - Enterprise Account desde la galería

Para configurar la integración de GitHub Enterprise Cloud - Enterprise Account en Azure AD, es preciso agregar GitHub Enterprise Cloud - Enterprise Account desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la Galería**, escriba **GitHub Enterprise Cloud - Enterprise Account** en el cuadro de búsqueda.
1. Seleccione **GitHub Enterprise Cloud - Enterprise Account** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-github-enterprise-cloud---enterprise-account"></a>Configuración y prueba del inicio de sesión único (SSO) de Azure AD para GitHub Enterprise Cloud - Enterprise Account

Configure y pruebe el inicio de sesión único (SSO) de Azure AD en GitHub Enterprise Cloud - Enterprise Account con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una vinculación entre un usuario de Azure AD y el usuario correspondiente de GitHub Enterprise Cloud - Enterprise Account.

Para configurar y probar el inicio de sesión único (SSO) de Azure AD con GitHub Enterprise Cloud - Enterprise Account, siga los pasos que se indican a continuación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de Azure AD y la cuenta de usuario de prueba a la aplicación GitHub](#assign-your-azure-ad-user-and-the-test-user-account-to-the-github-app)** , para permitir que su cuenta de usuario y el usuario de prueba `B.Simon` usen el inicio de sesión único de Azure AD.
1. **[Habilitación y prueba de SAML para la cuenta de empresa y sus organizaciones](#enable-and-test-saml-for-the-enterprise-account-and-its-organizations)** , con el fin de configurar las opciones de inicio de sesión único en la aplicación.
    1. **[Prueba del inicio de sesión único con otro propietario de la cuenta de empresa o cuenta de miembro de la organización](#test-sso-with-another-enterprise-account-owner-or-organization-member-account)** , para determinar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **GitHub Enterprise Cloud - Enterprise Account**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://github.com/orgs/<ENTERPRISE-SLUG>`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://github.com/orgs/<ENTERPRISE-SLUG>/saml/consume`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

     En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://github.com/orgs/<ENTERPRISE-SLUG>/sso`

    > [!NOTE]
    > Reemplace `<ENTERPRISE-SLUG>` por el nombre real de su cuenta de GitHub Enterprise.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificateBase64.png)

1. En la sección **Configurar GitHub Enterprise Cloud - Enterprise Account**, copie las direcciones URL adecuadas en función de sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, creará un usuario de prueba en Azure Portal llamado `B.Simon`.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

<a name="assign-the-azure-ad-test-user"></a>

### <a name="assign-your-azure-ad-user-and-the-test-user-account-to-the-github-app"></a>Asignación del usuario de Azure AD y la cuenta de usuario de prueba a la aplicación GitHub

En esta sección, va a permitir que `B.Simon` y su cuenta de usuario utilicen el inicio de sesión único de Azure al conceder acceso a GitHub Enterprise Cloud - Enterprise Account.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **GitHub Enterprise Cloud - Enterprise Account**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** y la cuenta de usuario en la lista de usuarios. A continuación, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="enable-and-test-saml-for-the-enterprise-account-and-its-organizations"></a>Habilitación y prueba de SAML para la cuenta de empresa y sus organizaciones

Para configurar el inicio de sesión único en **GitHub Enterprise Cloud - Enterprise Account**, siga los pasos indicados en la [documentación de GitHub](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-your-enterprise/enforcing-security-settings-in-your-enterprise-account#enabling-saml-single-sign-on-for-organizations-in-your-enterprise-account). 
1. Inicie sesión en GitHub.com con una cuenta de usuario que sea un [propietario de la cuenta de empresa](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-your-enterprise/roles-in-an-enterprise#enterprise-owner). 
1. Copie el valor del campo `Login URL` en la aplicación desde Azure Portal y péguelo en el campo `Sign on URL` de la configuración de SAML para la cuenta de GitHub Enterprise. 
1. Copie el valor del campo `Azure AD Identifier` en la aplicación del Azure Portal y péguelo en el campo `Issuer` de la configuración de SAML para la cuenta de GitHub Enterprise. 
1. Copie el contenido del archivo **Certificate (Base64)** que descargó en los pasos anteriores en Azure Portal y péguelos en el campo correspondiente de la configuración de SAML para la cuenta de GitHub Enterprise. 
1. Haga clic en `Test SAML configuration` y confirme que puede autenticarse correctamente con la cuenta de GitHub Enterprise en Azure AD.
1. Una vez que la comprobación sea satisfactoria, guarde la configuración. 
1. Después de autenticarse mediante SAML por primera vez desde la cuenta de GitHub Enterprise, se creará una _identidad externa vinculada_ en la cuenta de GitHub Enterprise que asocia la cuenta de usuario de GitHub que ha iniciado sesión a la cuenta de usuario de Azure AD.  
 
Después de habilitar el inicio de sesión único de SAML para su cuenta de GitHub Enterprise, el inicio de sesión único de SAML estará habilitado de forma predeterminada para todas las organizaciones pertenecientes a su cuenta de empresa. Todos los miembros deben autenticarse mediante el inicio de sesión único de SAML para acceder a las organizaciones a las que pertenecen; y los propietarios de empresa deben autenticarse mediante el inicio de sesión único de SAML al acceder a una cuenta de empresa.

<a name="test-sso"></a>

## <a name="test-sso-with-another-enterprise-account-owner-or-organization-member-account"></a>Prueba del inicio de sesión único con otro propietario de la cuenta de empresa o cuenta de miembro de la organización

Una vez configurada la integración de SAML para la cuenta de GitHub Enterprise (que también se aplica a las organizaciones de GitHub de la cuenta de empresa), otros propietarios de cuentas de empresa que estén asignados a la aplicación en Azure AD deberían poder acceder a la URL de la cuenta de GitHub Enterprise (`https://github.com/orgs/<enterprise account>`), autenticarse mediante SAML y acceder a las directivas y la configuración de la cuenta de GitHub Enterprise. 

El propietario de una organización para una cuenta de empresa debería poder [invitar a un usuario a unirse a su organización de GitHub](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-organizations-and-teams/inviting-users-to-join-your-organization). Inicie sesión en GitHub.com con una cuenta de propietario de organización y siga los pasos del artículo para invitar a `B.Simon` a la organización. Es necesario crear una cuenta de usuario de GitHub para `B.Simon`, si aún no dispone de una. 

Para comprobar el acceso a la organización de GitHub en la cuenta de empresa con la cuenta de usuario de prueba de `B.Simon`, siga estos pasos:
1. Invite a `B.Simon` a una organización de la cuenta de empresa como propietario de la organización. 
1. Inicie sesión en GitHub.com con la cuenta de usuario que desea vincular a la cuenta de usuario de Azure AD de `B.Simon`.
1. Inicie sesión en Azure AD con la cuenta de usuario `B.Simon`.
1. Acceda a la organización de GitHub. Se solicitará al usuario que se autentique mediante SAML. Una vez realizada correctamente la autenticación SAML, `B.Simon` debe poder acceder a los recursos de la organización. 

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado GitHub Enterprise Cloud - Enterprise Account, puede aplicar el control de sesión, que protege contra la filtración e infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
