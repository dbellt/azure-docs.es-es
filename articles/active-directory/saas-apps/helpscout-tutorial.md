---
title: 'Tutorial: Integración de Azure Active Directory con Help Scout | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y Help Scout.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/28/2021
ms.author: jeedes
ms.openlocfilehash: 0db3825a3e835010ee3c7b8e6203191653e6507f
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111572344"
---
# <a name="tutorial-azure-active-directory-integration-with-help-scout"></a>Tutorial: integración de Azure Active Directory con Help Scout

En este tutorial, aprenderá a integrar Help Scout con Azure Active Directory (Azure AD). Al integrar Help Scout con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Help Scout.
* Permitir que los usuarios inicien sesión automáticamente en Help Scout con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con Help Scout, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Help Scout.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Help Scout admite el inicio de sesión único iniciado por **SP e IDP**.
* Help Scout admite aprovisionamiento de usuarios **Just-In-Time**.

## <a name="add-help-scout-from-the-gallery"></a>Incorporación de Help Scout desde la galería

Para configurar la integración de Help Scout en Azure AD, deberá agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Help Scout** en el cuadro de búsqueda.
1. Seleccione **Help Scout** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-help-scout"></a>Configuración y prueba del inicio de sesión único de Azure AD para Help Scout

Configure y pruebe el inicio de sesión único de Azure AD con Help Scout mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Help Scout.

Para configurar y probar el inicio de sesión único de Azure AD con Help Scout, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Help Scout](#configure-help-scout-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Help Scout](#create-help-scout-test-user)** , para tener un homólogo de B.Simon en Help Scout que esté vinculado a su representación en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Help Scout**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en el modo iniciado por **IDP** siga estos pasos:

    a. **Identificador** es la opción **Audience URI (Service Provider Entity ID)** [URI de audiencia (Identificador de entidad del proveedor de servicios)] de Help Scout y empieza por `urn:`

    b. **URL de respuesta** es la opción **Post-back URL (Assertion Consumer Service URL)** [URL de devolución (URL del Servicio de consumidor de aserciones)] de Help Scout y empieza por `https://` 

    > [!NOTE]
    > Los valores de estas direcciones URL se muestran solo con fines demostrativos. Tiene que actualizar estos valores con el identificador y la dirección URL de respuesta reales. Estos valores se obtienen en la pestaña **Single Sign-On** (Inicio de sesión único), en la sección Authentication (Autenticación), que se explica más adelante en el tutorial.

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL: `https://secure.helpscout.net/members/login/`

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up Help Scout** (Configurar Help Scout), copie las direcciones URL adecuada según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a B.Simon para que use el inicio de sesión único de Azure concediéndole acceso a Help Scout.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Help Scout**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-help-scout-sso"></a>Configuración del inicio de sesión único de Help Scout

1. Para automatizar la configuración en Help Scout, debe instalar la **extensión de explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instale la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Configurar Help Scout** para ir a esta aplicación. En ella, escriba las credenciales de administrador para iniciar sesión en Help Scout. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 7.

    ![Configuración](common/setup-sso.png)

1. Si quiere configurar Help Scout manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Help Scout como administrador y lleve a cabo los siguientes pasos:

1. Haga clic en **Manage** (Administrar) en el menú superior y seleccione **Company** (Compañía) en el menú desplegable.

    ![Captura de pantalla que muestra el menú Manage (Administrar) con Company (Empresa) seleccionado.](./media/helpscout-tutorial/settings.png)

1. Seleccione **Autenticación** en el panel de navegación izquierdo.

    ![Captura de pantalla que muestra Authentication (Autenticación) seleccionado.](./media/helpscout-tutorial/authentication.png)

1. Esto le lleva a la sección de configuración de SAML, donde debe seguir estos pasos:

    ![Captura de pantalla que muestra la pestaña Single Sign-On (Inicio de sesión único) en la que se especifica la información especificada.](./media/helpscout-tutorial/configuration.png)

    a. Copie el valor de **Post-back URL (Assertion Consumer Service URL)** [URL de devolución (URL del Servicio de consumidor de aserciones)] y péguelo en el cuadro **URL de respuesta**, en la sección **Configuración básica de SAML** de Azure Portal.

    b. Copie el valor de **Audience URI (Service Provider Entity ID)** [URI de audiencia (Identificador de entidad del proveedor de servicios)] y péguelo en el cuadro **Identificador**, en la sección **Configuración básica de SAML** de Azure Portal.

1. Active **Enable SAML** (Habilitar SAML) y siga estos pasos:

    ![Captura de pantalla que muestra la pestaña Single Sign-On (Inicio de sesión único) para habilitar SAML y agregar más información.](./media/helpscout-tutorial/information.png)

    a. En el cuadro de texto **Single Sign-on URL** (Dirección URL de inicio de sesión único), pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    b. Haga clic en **Upload Certificate** (Cargar certificado) para cargar el **Certificado (Base64)** que descargó de Azure Portal.

    c. Especifique el dominio o dominios de correo electrónico de su organización, por ej. `contoso.com` en el cuadro de texto **Email Domains** (Dominios de correo electrónico). Puede separar varios dominios mediante comas. Siempre que un administrador o un usuario de Help Scout entre en ese dominio específico en la [página de inicio de sesión de Help Scout](https://secure.helpscout.net/members/login/), se le redirigirá al proveedor de identidades para que se autentique con sus credenciales.

    d. Por último, puede cambiar **Force SAML Sign-on** (Forzar inicio de sesión de SAML) si desea que los usuarios solo inicien sesión en Help Scout mediante este método. Si aún así desea dejar la opción para que puedan conectarse con sus credenciales de Help Scout, puede dejarla desactivada. Incluso si esta opción está habilitada, el propietario de la cuenta siempre podrá iniciar sesión en Help Scout con su contraseña de la cuenta.

    e. Haga clic en **Save**(Guardar).

### <a name="create-help-scout-test-user"></a>Creación de un usuario de prueba de Help Scout

En esta sección, se crea un usuario llamado B.Simon en Help Scout. Help Scout admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe en Help Scout, se crea otro después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la URL de inicio de sesión de Help Scout, desde donde podrá comenzar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de Help Scout e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de Help Scout para la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Help Scout en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y, si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de Help Scout para la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que se ha configurado Help Scout, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
