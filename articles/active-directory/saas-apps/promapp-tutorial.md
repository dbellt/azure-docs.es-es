---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Nintex Promapp | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Nintex Promapp.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/10/2021
ms.author: jeedes
ms.openlocfilehash: 348d71bec02f70da656b2cc5af262ab31c64d673
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111983325"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-nintex-promapp"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Nintex Promapp

En este tutorial, aprenderá a integrar Nintex Promapp Azure Active Directory (Azure AD). Al integrar Nintex Promapp con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Nintex Promapp.
* Permitir que los usuarios inicien sesión automáticamente en Nintex Promapp con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para inicio de sesión único (SSO) en Nintex Promapp.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Nintex Promapp admite el inicio de sesión único iniciado por **SP e IDP**.
* Nintex Promapp admite el aprovisionamiento de usuarios **Just-In-Time**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-nintex-promapp-from-the-gallery"></a>Incorporación de Nintex Promapp desde la galería

Para configurar la integración de Nintex Promapp en Azure AD, deberá agregar Nintex Promapp desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Nintex Promapp** en el cuadro de búsqueda.
1. Seleccione **Nintex Promapp** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-nintex-promapp"></a>Configuración y prueba del inicio de sesión único de Azure AD para Nintex Promapp

Configure y pruebe el inicio de sesión único de Azure AD con Nintex Promapp mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Nintex Promapp.

Para configurar y probar el inicio de sesión único de Azure AD con Nintex Promapp, haga lo siguiente:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Nintex Promapp](#configure-nintex-promapp-sso)**, para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Nintex Promapp](#create-nintex-promapp-test-user)**, para tener un homólogo de B.Simon en Nintex Promapp que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Nintex Promapp**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice los siguientes pasos:

    1. En el cuadro de texto **Identificador**, escriba una de las siguientes direcciones URL:
        
        | URL del identificador |
        |-----|
        |`https://go.promapp.com/TENANTNAME/`|
        |`https://au.promapp.com/TENANTNAME/`|
        |`https://us.promapp.com/TENANTNAME/`|
        |`https://eu.promapp.com/TENANTNAME/`|
        |`https://ca.promapp.com/TENANTNAME/`|

       > [!NOTE]
       > La integración de Azure AD con Nintex Promapp está configurada actualmente solo para la autenticación iniciada por el servicio (es decir, si va a una dirección URL de Nintex Promapp se inicia el proceso de autenticación). Sin embargo, el campo **Dirección URL de respuesta** es obligatorio.

    1. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<DOMAIN_NAME>.promapp.com/TENANTNAME/saml/authenticate.aspx`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **Dirección URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<DOMAIN_NAME>.promapp.com/TENANTNAME/saml/authenticate`.

    > [!NOTE]
    > Estos valores son marcadores de posición. Debe usar los valores reales de identificador, dirección URL de respuesta o dirección URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de Nintex Promapp](https://www.promapp.com/about-us/contact-us/) para obtener los valores. También puede consultar los patrones que se muestran en el cuadro de diálogo **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Nintex Promapp**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección va a permitir que B.Simon acceda a Nintex Promapp mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Nintex Promapp**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-nintex-promapp-sso"></a>Configuración del inicio de sesión único de Nintex Promapp

1. Inicie sesión en el sitio de la empresa Nintex Promapp como administrador.

2. En el menú de la parte superior de la ventana, seleccione **Admin** (Administrador):

    ![Seleccionar Admin (Administrador)](./media/promapp-tutorial/admin.png)

3. Seleccione **Configure** (Configurar):

    ![Seleccionar Configure (Configurar)](./media/promapp-tutorial/configuration.png)

4. En el cuadro de diálogo **Security** (Seguridad), siga los pasos que se indican a continuación.

    ![Cuadro de diálogo Security (Seguridad)](./media/promapp-tutorial/certificate.png)

    1. Pegue el valor de **Dirección URL de inicio de sesión** que copió de Azure Portal en el cuadro **SSO-Login URL** (URL de inicio de sesión único).

    1. En la lista **SSO - Single Sign-on Mode** (Modo Inicio de sesión único), seleccione **Optional** (Opcional). Seleccione **Guardar**.

       > [!NOTE]
       > El modo opcional es solo para pruebas. Una vez que esté satisfecho con la configuración, seleccione **Required** (Re requiere) en la lista **SSO - Single Sign-on Mode** (Modo Inicio de sesión único) para forzar a todos los usuarios a autenticarse en Azure AD.

    1. En el Bloc de notas, abra el certificado que descargó en la sección anterior. Copie el contenido del certificado sin la primera línea (**---BEGIN CERTIFICATE---**) o la última línea (**---END CERTIFICATE---**). Pegue el contenido del certificado en el cuadro **SSO-certificado x.509** y, a continuación, seleccione **Save** (Guardar).

### <a name="create-nintex-promapp-test-user"></a>Creación de un usuario de prueba en Nintex Promapp

En esta sección se crea un usuario llamado B.Simon en Nintex Promapp. Nintex Promapp admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe ya en Nintex Promapp, se crea después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Nintex Promapp, donde podrá comenzar el flujo de inicio de sesión.  

* Vaya a directamente a la dirección URL de inicio de sesión de Nintex Promapp y comience el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación en Azure Portal**; debería iniciar sesión automáticamente en la instancia de Nintex Promapp para la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Nintex Promapp en Mis aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de Nintex Promapp para la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que se ha configurado Nintex Promapp, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
