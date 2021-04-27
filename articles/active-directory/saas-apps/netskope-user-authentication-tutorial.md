---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Netskope User Authentication | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Netskope User Authentication.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/06/2021
ms.author: jeedes
ms.openlocfilehash: 78ef49740c3e74c4ad5a30838d9721969ca70b36
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107517004"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-netskope-user-authentication"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Netskope User Authentication

En este tutorial aprenderá a integrar Netskope User Authentication con Azure Active Directory (Azure AD). Al integrar Netskope User Authentication con Azure AD, podrá:

* Controlar en Azure AD quién tiene acceso a Netskope User Authentication.
* Permitir que los usuarios inicien sesión automáticamente en Netskope User Authentication con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Netskope User Authentication.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Netskope User Authentication admite el inicio de sesión único iniciado por **SP e IDP**.

## <a name="add-netskope-user-authentication-from-the-gallery"></a>Adición de Netskope User Authentication desde la galería

Para configurar la integración de Netskope User Authentication con Azure AD, debe agregarla desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Netskope User Authentication** en el cuadro de búsqueda.
1. Seleccione **Netskope User Authentication** del panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-netskope-user-authentication"></a>Configuración y prueba del inicio de sesión único de Azure AD para Netskope User Authentication

Configure y pruebe el inicio de sesión único de Azure AD con Netskope User Authentication mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Netskope User Authentication.

Para configurar y probar el inicio de sesión único de Azure AD con Netskope User Authentication, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Netskope User Authentication](#configure-netskope-user-authentication-sso)** : para configurar el inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Netskope User Authentication](#create-netskope-user-authentication-test-user)** : para tener un homólogo de B.Simon en Netskope User Authentication vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Netskope User Authentication**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<tenantname>.goskope.com/<customer entered string>`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<tenantname>.goskope.com/nsauth/saml2/http-post/<customer entered string>`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Estos valores se explican más adelante en el tutorial.

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<tenantname>.goskope.com`

    > [!NOTE]
    > El valor de la dirección URL de inicio de sesión no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico al cliente de Netskope User Authentication](mailto:support@netskope.com) para obtener el valor de URL de inicio de sesión. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Set up Netskope User Authentication** (Configurar Netskope User Authentication), copie las direcciones URL que necesite.

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

En esta sección va a permitir que B.Simon acceda a Netskope User Authentication mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Netskope User Authentication**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-netskope-user-authentication-sso"></a>Configuración del inicio de sesión único en Netskope User Authentication

1. Abra una nueva pestaña en el explorador e inicie sesión en el sitio de la empresa de Netskope User Authentication como administrador.

1. Haga clic en la pestaña **Active Platform** (Plataforma activa).

    ![Captura de pantalla que muestra la opción Active Platform (Plataforma activa) seleccionada en Settings (Configuración).](./media/netskope-user-authentication-tutorial/user-1.png)

1. Desplácese hacia abajo hasta **FORWARD PROXY** (PROXY DE REENVÍO) y seleccione **SAML**.

    ![Captura de pantalla que muestra la opción SAML seleccionada en Active Platform (Plataforma activa).](./media/netskope-user-authentication-tutorial/configuration.png)

1. En la página **SAML Settings** (Configuración de SAML), realice los pasos siguientes:

    ![Captura de pantalla que muestra la página SAML Settings (Configuración de SAML), donde puede especificar los valores descritos.](./media/netskope-user-authentication-tutorial/configure-copyurls.png)

    a. Copie el valor de **SAML Entity ID** (Id. de entidad SAML) y péguelo en el cuadro de texto **Identificador** de la sección **Configuración básica de SAML** en Azure Portal.

    b. Copie el valor de **SAML ACS URL** (URL de ACS de SAML) y péguelo en el cuadro de texto **URL de respuesta** de la sección **Configuración básica de SAML** en Azure Portal.

1. Haga clic en **ADD ACCOUNT** (AGREGAR CUENTA).

    ![Captura de pantalla que muestra la opción ADD ACCOUNT (AGREGAR CUENTA) seleccionada en el panel de SAML.](./media/netskope-user-authentication-tutorial/add-account.png)

1. En la página **Add SAML Account** (Agregar cuenta de SAML), realice los pasos siguientes:

    ![Captura de pantalla que muestra la página Add SAML Account (Agregar cuenta de SAML), donde puede especificar los valores descritos.](./media/netskope-user-authentication-tutorial/configure-settings.png)

    a. En el cuadro de texto **NAME** (NOMBRE), proporcione el nombre, por ejemplo, Azure AD.

    b. En el cuadro de texto **IDP URL** (URL de IDP), pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    c. En el cuadro de texto **IDENTIFICADOR DE LA ENTIDAD DE IDP**, pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    d. Abra el archivo de metadatos descargado en el bloc de notas, copie su contenido en el portapapeles y péguelo en el cuadro de texto **IDP CERTIFICATE** (CERTIFICADO DE IDP).

    e. Haga clic en **GUARDAR**.

### <a name="create-netskope-user-authentication-test-user"></a>Creación de un usuario de prueba en Netskope User Authentication

1. Abra una nueva pestaña en el explorador e inicie sesión en el sitio de la empresa de Netskope User Authentication como administrador.

1. Haga clic en la pestaña **Settings** (Configuración) en el panel de navegación izquierdo.

    ![Captura de pantalla de la pestaña Settings (Configuración) seleccionada.](./media/netskope-user-authentication-tutorial/configuration-settings.png)

1. Haga clic en la pestaña **Active Platform** (Plataforma activa).

    ![Captura de pantalla que muestra la opción Active Platform (Plataforma activa) seleccionada en Settings (Configuración).](./media/netskope-user-authentication-tutorial/user-1.png)

1. Haga clic en la pestaña **Users** (Usuarios).

    ![Captura de pantalla que muestra la opción Users (Usuarios) seleccionada en Active Platform (Plataforma activa).](./media/netskope-user-authentication-tutorial/add-user.png)

1. Haga clic en **ADD USERS** (AGREGAR USUARIOS).

    ![Captura de pantalla que muestra el cuadro de diálogo Users (Usuarios) donde puede seleccionar ADD USERS (AGREGAR USUARIOS).](./media/netskope-user-authentication-tutorial/user-add.png)

1. Escriba la dirección de correo electrónico del usuario al que desea agregar y haga clic en **ADD** (AGREGAR).

    ![Captura de pantalla que muestra la opción Add Users (Agregar usuarios) donde puede especificar una lista de usuarios.](./media/netskope-user-authentication-tutorial/add-user-popup.png)

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Netskope User Authentication, donde podrá iniciar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de Netskope User Authentication e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de Netskope User Authentication para la que configurara el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Netskope User Authentication en Aplicaciones, si tiene la configuración del modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para iniciar el flujo de inicio de sesión y, si tiene la configuración del modo IDP, debería iniciar sesión automáticamente en la instancia de Netskope User Authentication para la que configurara el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurada Netskope User Authentication, puede aplicar el control de sesión, que protege la información confidencial de la organización de la filtración y la infiltración en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).
