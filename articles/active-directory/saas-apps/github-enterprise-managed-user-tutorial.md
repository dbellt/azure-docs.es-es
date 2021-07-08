---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con GitHub Enterprise Managed User | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y GitHub Enterprise Managed User.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/15/2021
ms.author: jeedes
ms.openlocfilehash: ffdd6c30e279cc5df7f97e5ab5bb77a87c18dd8b
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110480528"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-github-enterprise-managed-user"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con GitHub Enterprise Managed User

En este tutorial, aprenderá a integrar GitHub Enterprise Managed User (EMU) con Azure Active Directory (Azure AD). Al integrar la cuenta de GitHub Enterprise Managed User con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a la cuenta de GitHub Enterprise Managed User.
* Permitir que los usuarios inicien sesión automáticamente en la cuenta de GitHub Enterprise Managed User con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en GitHub Enterprise Managed User.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* GitHub Enterprise Managed User admite el inicio de sesión único iniciado por **SP e IDP**.
* GitHub Enterprise Managed User admite el [aprovisionamiento **automatizado** de usuarios](./github-enterprise-managed-user-provisioning-tutorial.md).

## <a name="adding-github-enterprise-managed-user-from-the-gallery"></a>Adición de un usuario administrado de GitHub Enterprise Managed User desde la galería

Para configurar la integración de GitHub Enterprise Managed User en Azure AD, es preciso agregar GitHub Enterprise Managed User desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **GitHub Enterprise Managed User** en el cuadro de búsqueda.
1. Seleccione **GitHub Enterprise Managed User** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-github-enterprise-managed-user"></a>Configuración y prueba del inicio de sesión único (SSO) de Azure AD para GitHub Enterprise Managed User

Para configurar y probar el inicio de sesión único (SSO) de Azure AD con GitHub Enterprise Managed User, siga los pasos que se indican a continuación:

1. **[Configuración de SSO de Azure AD:](#configure-azure-ad-sso)** para habilitar el inicio de sesión único de SAML en el inquilino de AAD.
1. **[Configuración de SSO de GitHub Enterprise Managed User](#configure-github-enterprise-managed-user-sso)** : para configurar los valores de inicio de sesión único en GitHub Enterprise.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **GitHub Enterprise Managed User**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. Asegúrese de que tiene la dirección URL de Enterprise antes de empezar. El campo ENTITY que se menciona a continuación es el nombre de Enterprise de la dirección URL de Enterprise habilitada para EMU. Por ejemplo, https://github.com/enterprises/contoso - **contoso** es el valor de ENTITY. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://github.com/enterprises/<ENTITY>`
    
    > [!NOTE]
    > Tenga en cuenta que el formato del identificador es diferente del sugerido por la aplicación; siga el formato de arriba. Además, asegúrese de que el **identificador no contiene una barra diagonal final.
    
    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://github.com/enterprises/<ENTITY>/saml/consume`
    

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://github.com/enterprises/<ENTITY>/sso`

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificate-base64-download.png)

1. En la sección **Configurar GitHub Enterprise Managed User**, copie las direcciones URL siguientes y guárdelas para configurar GitHub a continuación.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, asignará su cuenta a GitHub Enterprise Managed User para completar la configuración de SSO.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **GitHub Enterprise Managed User**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione su cuenta en la lista de usuarios y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Seleccionar un rol**, seleccione el rol **Enterprise Owner** (Propietario de la empresa) y, a continuación, haga clic en el botón **Seleccionar** situado en la parte inferior de la pantalla. La cuenta se asigna como Enterprise Owner (Propietario de la empresa) para la instancia de GitHub cuando aprovisione la cuenta en el siguiente tutorial. 
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-github-enterprise-managed-user-sso"></a>Configuración del inicio de sesión único de GitHub Enterprise Managed User

Para configurar el inicio de sesión único en **GitHub Enterprise Managed User**, necesitará los siguientes elementos:

1. Las direcciones URL anteriores de la aplicación de usuarios administrados de empresa de AAD: dirección URL de inicio de sesión, identificador de Azure AD y dirección URL de cierre de sesión.
1. El nombre de cuenta y la contraseña del primer usuario administrador de GitHub Enterprise. Las credenciales se proporcionan mediante un correo electrónico de restablecimiento de contraseña que remite el contacto de ingeniería de soluciones de GitHub. 

### <a name="enable-github-enterprise-managed-user-saml-sso"></a>Habilitación de SSO de SAML para GitHub Enterprise Managed User

En esta sección, tomará la información anterior obtenida de AAD y la escribirá en la configuración de Enterprise para habilitar la compatibilidad con SSO.

1. Vaya a https://github.com.
1. Haga clic en Sign In (Iniciar sesión) en la esquina superior derecha.
1. Escriba las credenciales de la primera cuenta de usuario de administrador. El identificador de inicio de sesión debe tener el formato siguiente: `<your enterprise short code>_admin`.
1. Vaya a https://github.com/enterprises/ `<your enterprise name>`. Esta información se la debe proporcionar su contacto de ingeniería de soluciones.
1. En el menú de navegación de la izquierda, seleccione **Settings** (Configuración) y, luego, **Security** (Seguridad).
1. Haga clic en la casilla **Enable SAML authentication** (Habilitar autenticación SAML).
1. Escriba la dirección URL de inicio de sesión. Esta dirección URL es la dirección URL de inicio de sesión anterior que copió de AAD.
1. Escriba el emisor. Esta dirección URL es el identificador de Azure AD anterior que copió de AAD.
1. Escriba el certificado público. Abra el certificado base64 que descargó anteriormente y pegue el contenido de texto de ese archivo en este cuadro de diálogo.
1. Haga clic en **Probar configuración de SAML.** Se abrirá un cuadro de diálogo para que inicie sesión con sus credenciales de Azure AD para validar que el inicio de sesión único de SAML está configurado correctamente. Inicie sesión con sus credenciales de AAD. Recibirá un mensaje **Passed: Successfully authenticated your SAML SSO identity** (Superado: Autenticación correcta de la identidad de SSO de SAML) tras una validación correcta.
1. Haga clic en **Guardar** para conservar esta configuración.
1. Guarde (descargue, imprima o copie) los códigos de recuperación en un lugar seguro.
1. Haga clic en **Habilitar autenticación SAML**.
1. En este momento, solo las cuentas con SSO pueden iniciar sesión en Enterprise. Siga las instrucciones del documento siguiente sobre el aprovisionamiento para aprovisionar cuentas con inicio de sesión único.

## <a name="next-steps"></a>Pasos siguientes

GitHub Enterprise Managed User **requiere** que todas las cuentas se creen a través del aprovisionamiento automático de usuarios. Puede encontrar más detalles [aquí](./github-enterprise-managed-user-provisioning-tutorial.md) sobre cómo configurar el aprovisionamiento automático de usuarios.
