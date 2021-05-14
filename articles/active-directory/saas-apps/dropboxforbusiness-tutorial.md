---
title: 'Tutorial: Integración de Azure Active Directory con Dropbox Business | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Dropbox Business.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/17/2021
ms.author: jeedes
ms.openlocfilehash: 30c3ecb64b8fdb2ff1bf3569803b07d9f0646347
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108138092"
---
# <a name="tutorial-integrate-dropbox-business-with-azure-active-directory"></a>Tutorial: Integración de Azure Active Directory con Dropbox Business

En este tutorial, aprenderá a integrar Dropbox Business con Azure Active Directory (Azure AD). Si integra Dropbox Business con Azure AD, podrá hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Dropbox Business.
* Permitir que los usuarios inicien sesión automáticamente en Dropbox Business con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Dropbox Business.

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

* En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. Dropbox Business admite SSO iniciado por **SP**.

* Dropbox Business admite el [aprovisionamiento y desaprovisionamiento de usuarios de forma automática](dropboxforbusiness-tutorial.md).

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-dropbox-business-from-the-gallery"></a>Adición de Dropbox Business desde la galería

Para configurar la integración de Dropbox Business en Azure AD, es preciso agregar Dropbox Business desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Dropbox Business** en el cuadro de búsqueda.
1. Seleccione **Dropbox Business** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-dropbox-business"></a>Configuración y prueba del inicio de sesión único de Azure AD para Dropbox Business

Configure y pruebe el inicio de sesión único de Azure AD con Dropbox Business mediante el usuario de prueba **Britta Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Dropbox Business.

Para configurar el inicio de sesión único de Azure AD con Dropbox Business, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.    
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Dropbox Business](#configure-dropbox-business-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Dropbox Business](#create-dropbox-business-test-user)** , para tener un homólogo de Britta Simon en Dropbox Business que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Dropbox Business**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://www.dropbox.com/sso/<id>`
    
     b. En el cuadro de texto **Identificador (id. de entidad)** , escriba el valor: `Dropbox`
    
    c. En el campo **URL de respuesta**, escriba `https://www.dropbox.com/saml_login`.
    > [!NOTE]
    > El **identificador de inicio de sesión único de Dropbox** se puede encontrar en el sitio de Dropbox en Dropbox > Admin console > Settings > Single sign-on > SSO sign-in URL (Dropbox > Consola de administración > Configuración > Inicio de sesión único > Dirección URL de inicio de sesión único).

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up Dropbox Business** (Configurar Dropbox Business), copie las direcciones URL correctas según sus requisitos.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)


### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado Britta Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `Britta Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `BrittaSimon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, concederá acceso a B.Simon a Dropbox Business para que pueda usar el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Dropbox Business**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-dropbox-business-sso"></a>Configuración del inicio de sesión único de Dropbox Business

1. Para automatizar la configuración en Dropbox Business, debe instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Install the extension** (Instalar la extensión).

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Configurar Dropbox Business** para ir a la aplicación Dropbox Business. Desde allí, proporcione las credenciales del administrador para iniciar sesión en Dropbox Business. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 8.

    ![Configuración](common/setup-sso.png)

3. Si desea configurar Dropbox Business manualmente, abra una nueva ventana del explorador web, vaya al inquilino de Dropbox Business e inicie sesión en él. y realice los siguientes pasos:

    ![Captura de pantalla que muestra la página de inicio de sesión de Dropbox Business.](./media/dropboxforbusiness-tutorial/account.png "Configurar inicio de sesión único")

4. Haga clic en el **icono de usuario** y seleccione la pestaña **Settings** (Configuración).

    ![Captura de pantalla que muestra la acción de icono de usuario y la pestaña "Settings" (Configuración) seleccionada.](./media/dropboxforbusiness-tutorial/configure-1.png "Configurar inicio de sesión único")

5. En el panel de navegación de la izquierda, haga clic en **Admin Console**(Consola de administración).

    ![Captura de pantalla que muestra "Admin console" (Consola de administración) seleccionado.](./media/dropboxforbusiness-tutorial/configure-2.png "Configurar inicio de sesión único")

6. En **Admin Console** (Consola de administración), haga clic en la opción **Settings** (Configuración) del panel de navegación izquierdo.

    ![Captura de pantalla que muestra "Settings" (Configuración) seleccionado.](./media/dropboxforbusiness-tutorial/configure-3.png "Configurar inicio de sesión único")

7. Seleccione la opción **Single sign-on** (Inicio de sesión único) en la sección **Authentication** (Autenticación).

    ![Captura de pantalla que muestra la sección "Authentication" (Autenticación) con la opción "Single sign-on" (Inicio de sesión único) seleccionada.](./media/dropboxforbusiness-tutorial/configure-4.png "Configurar inicio de sesión único")

8. En la sección **Single sign-on** (Inicio de sesión único), siga estos pasos:  

    ![Captura de pantalla que muestra la configuración de "Single sign-on" (Inicio de sesión único).](./media/dropboxforbusiness-tutorial/configure-5.png "Configurar inicio de sesión único")

    a. Seleccione **Required** (Obligatorio) como opción en el menú desplegable para **Single sign-on** (Inicio de sesión único).

    b. Haga clic en **Add sign-in URL** (Agregar inicio de sesión único) y en el cuadro de texto **Identity provider sign-in URL** (Dirección URL de inicio de sesión del proveedor de identidades), pegue el valor de **Login URL** (Dirección URL de inicio de sesión) que ha copiado de Azure Portal y luego seleccione **Done** (Listo).

    ![Configuración del inicio de sesión único](./media/dropboxforbusiness-tutorial/sso.png "Configurar inicio de sesión único")

    c. Haga clic en **Upload certificate** (Cargar certificado) y, a continuación, vaya al **archivo de certificado codificado en Base64** que ha descargado de Azure Portal.

    d. Haga clic en **Copy link** (Copiar vínculo) y pegue el valor copiado en el cuadro de texto **Dirección URL de inicio de sesión** en la sección de **Dominio y direcciones URL de Dropbox Business** en Azure Portal.

    e. Haga clic en **Save**(Guardar).

### <a name="create-dropbox-business-test-user"></a>Creación de un usuario de prueba de Dropbox Business

En esta sección se crea un usuario llamado B.Simon en Dropbox Business. Dropbox Business admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en Dropbox Business, se crea uno nuevo después de la autenticación.

>[!Note]
>Si necesita crear un usuario de forma manual, póngase en contacto con el [equipo de soporte al cliente de Dropbox Business](https://www.dropbox.com/business/contact)

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Dropbox Business, donde podrá iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Dropbox Business e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Si hace clic en el icono de Dropbox Business de Aplicaciones, accederá a la dirección URL de inicio de sesión único de Dropbox Business. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Dropbox Business, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
