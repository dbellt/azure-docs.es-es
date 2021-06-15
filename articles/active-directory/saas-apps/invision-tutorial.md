---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con InVision | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory e InVision.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/24/2020
ms.author: jeedes
ms.openlocfilehash: 429215dbd35368c7a2c8681e480b5256813171a1
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111753654"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-invision"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con InVision

En este tutorial, obtendrá información sobre cómo integrar InVision con Azure Active Directory (Azure AD). Al integrar InVision con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a InVision.
* Permitir que los usuarios puedan iniciar sesión automáticamente en InVision con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en InVision.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* InVision admite el inicio de sesión único iniciado por **SP e IDP**.

## <a name="adding-invision-from-the-gallery"></a>Adición de InVision desde la galería

Para configurar la integración de InVision en Azure AD, debe agregar InVision desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **InVision** en el cuadro de búsqueda.
1. Seleccione **InVision** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-invision"></a>Configuración y prueba del inicio de sesión único de Azure AD para InVision

Configure y pruebe el inicio de sesión único de Azure AD con InVision mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de InVision.

Para configurar y probar el inicio de sesión único de Azure AD con InVision, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en InVision](#configure-invision-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de InVision](#create-invision-test-user)** : para tener un homólogo de B.Simon en InVision vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **InVision**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<SUBDOMAIN>.invisionapp.com`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.invisionapp.com//sso/auth`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.invisionapp.com`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de InVision](mailto:support@invisionapp.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar InVision**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a InVision mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **InVision**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-invision-sso"></a>Configuración del inicio de sesión único de InVision

1. Para automatizar la configuración en InVision, debe instalar la **extensión del navegador de inicio de sesión seguro de Aplicaciones**. Para ello, haga clic en **Instale la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al navegador, haga clic en **Set up InVision** (Configurar InVision) para ir a la aplicación InVision. Ahí, escriba las credenciales de administrador para iniciar sesión en InVision. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 6.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar InVision manualmente, en otra ventana del explorador web, inicie sesión en el sitio de la empresa InVision como administrador.

1. Haga clic en **Team** (Equipo) y seleccione **Settings** (Configuración).

    ![Captura de pantalla que muestra la pestaña Team (Equipo) con la opción Settings (Configuración) seleccionada.](./media/invision-tutorial/config1.png)

1. Desplácese hacia abajo hasta **Single sign-on**(Inicio de sesión único) y, a continuación, haga clic en **Change** (Cambiar).

    ![Captura de pantalla que muestra el botón Change (Cambiar) para Single sign-on (Inicio de sesión único).](./media/invision-tutorial/config3.png)

1. En la página **Single sign-on** (Inicio de sesión único) realice los pasos siguientes:

    ![Captura de pantalla que muestra la página Single Sign-On (Inicio de sesión único) en la que se especifica la información especificada.](./media/invision-tutorial/config4.png)

    a. Cambie **Require SSO for every member of < account name >** (Requerir SSO para cada miembro de <nombre de cuenta>) a **On** (Activado).

    b. En el cuadro de texto **Name** (Nombre), escriba el nombre; por ejemplo, `azureadsso`.

    c. Escriba el valor de la dirección URL de inicio de sesión en el cuadro de texto **Sign-in URL** (Dirección URL de inicio de sesión).

    d. En el cuadro de texto **Sign-out URL** (Dirección URL de cierre de sesión), pegue el valor de **Dirección URL de cierre de sesión** que ha copiado de Azure Portal.

    e. En el cuadro de texto **SAML Certificate** (Certificado de SAML), abra el archivo **Certificado (base 64)** descargado en el Bloc de notas, copie el contenido y péguelo en el cuadro de texto SAML Certificate (Certificado de SAML).

    f. En el cuadro de texto **Name ID Format** (Formato del identificador de nombre), use `urn:oasis:names:tc:SAML:1.1:nameid-format:Unspecified` para el campo **Name ID Format** (Formato del identificador de nombre).

    g. Seleccione **SHA-256** en la lista desplegable **HASH Algorithm** (Algoritmo hash).

    h. Escriba el nombre adecuado para **SSO Button Label** (Etiqueta del botón de SSO).

    i. Cambie **Allow Just-in-Time provisioning** (Permitir aprovisionamiento Just-In-Time) a On (Activado).

    j. Haga clic en **Update**(Actualizar).

### <a name="create-invision-test-user"></a>Creación de un usuario de prueba de InVision

1. En otra ventana del explorador web, inicie sesión en el sitio web de InVision como administrador.

1. Haga clic en **Team** (Equipo) y seleccione **People** (Personas).

    ![Captura de pantalla que muestra la pestaña Team (Equipo) con la opción People (Personas) seleccionada.](./media/invision-tutorial/config2.png)

1. Haga clic en el **icono +** para agregar un nuevo usuario.

    ![Captura de pantalla que muestra el icono + para agregar un usuario.](./media/invision-tutorial/user1.png)

1. Escriba la dirección de correo electrónico del usuario y haga clic en **Next** (Siguiente).

    ![Captura de pantalla que muestra el cuadro de diálogo Invite (Invitar), donde puede escribir direcciones.](./media/invision-tutorial/user2.png)

1. Confirme la dirección de correo electrónico y haga clic en **Invite** (Invitar).

    ![Captura de pantalla que muestra el cuadro de diálogo Invite (Invitar), donde puede seleccionar Invite (Invitar) para continuar.](./media/invision-tutorial/user3.png)

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de InVision donde podrá iniciar el flujo de inicio de sesión.

* Vaya directamente a la dirección URL de inicio de sesión de InVision e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de InVision para la que configurara el inicio de sesión único.

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de InVision en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de InVision para la que configurara el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurada InVision, puede aplicar el control de sesión, que protege la información confidencial de la organización de la filtración y la infiltración en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
