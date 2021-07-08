---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con CakeHR | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y CakeHR.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/08/2021
ms.author: jeedes
ms.openlocfilehash: 45ee0770031eee14f9a5732a4057a8f032ccfc4f
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111893879"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-cakehr"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con CakeHR

En este tutorial, obtendrá información sobre cómo integrar CakeHR con Azure Active Directory (Azure AD). Al integrar CakeHR con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a CakeHR.
* Permitir a los usuarios iniciar sesión automáticamente en CakeHR con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción habilitada para el inicio de sesión único (SSO) en CakeHR.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* CakeHR admite el inicio de sesión único iniciado por **SP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-cakehr-from-the-gallery"></a>Adición de CakeHR desde la galería

Para configurar la integración de CakeHR en Azure AD, es preciso agregar CakeHR desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **CakeHR** en el cuadro de búsqueda.
1. Seleccione **CakeHR** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-cakehr"></a>Configuración y prueba del SSO de Azure AD para CakeHR

Configure y pruebe el inicio de sesión único de Azure AD con CakeHR mediante una usuaria de prueba llamada **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de CakeHR.

Para configurar y probar el inicio de sesión único de Azure AD con CakeHR, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en CakeHR](#configure-cakehr-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en CakeHR](#create-cakehr-test-user)** , para tener un homólogo de B. Simon en CakeHR que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **CakeHR**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<CAKE_DOMAIN>.cake.hr/`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<CAKE_DOMAIN>.cake.hr/services/saml/consume`
    
    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte para clientes de CakeHR](mailto:info@cake.hr) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la sección **Certificado de firma de SAML**, haga clic en el botón **Editar** para abrir el cuadro de diálogo **Certificado de firma de SAML**.

    ![Edición del certificado de firma de SAML](common/edit-certificate.png)

1. En la sección **Certificado de firma de SAML**, copie el valor de **HUELLA DIGITAL** y guárdelo en el Bloc de notas.

    ![Copia del valor de la huella digital](common/copy-thumbprint.png)

1. En la sección **Configurar CakeHR**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B. Simon use el inicio de sesión único de Azure, para lo que le concederá acceso a CakeHR.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **CakeHR**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-cakehr-sso"></a>Configuración del inicio de sesión único en CakeHR

1. Para automatizar la configuración en CakeHR, debe instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Set up CakeHR** (Configurar CakeHR) para ir a la aplicación del mismo nombre. En ella, escriba las credenciales de administrador para iniciar sesión en CakeHR. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 5.

    ![Configuración](common/setup-sso.png)

1. Si quiere configurar CakeHR manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de CakeHR como administrador y haga lo siguiente:

1. En la esquina superior derecha de la página, haga clic en su **perfil** y vaya a **Configuración**.

    ![Captura de pantalla que muestra Profile (Perfil) con Settings (Configuración) seleccionado.](./media/cakehr-tutorial/profile.png)

1. En la parte izquierda de la barra de menús, haga clic en **INTEGRATIONS**(INTEGRACIONES) > **SAML SSO** (SSO de SAML) y siga estos pasos:

    ![Captura de pantalla que muestra el panel Setting (Configuración) para realizar estos pasos.](./media/cakehr-tutorial/menu.png)

    a. En el cuadro de texto **Entity ID** (Id. de identidad), escriba `cake.hr`.

    b. En el cuadro de texto **Authentication URL** (Dirección URL de autenticación), pegue el valor de **URL de inicio de sesión**, que ha copiado de Azure Portal.

    c. En el cuadro de texto **Key fingerprint (SHA1 format)** [Huella digital clave (formato SHA1)], pegue el valor de **THUMBPRINT** (HUELLA DIGITAL) que copió de Azure Portal.

    d. Seleccione la casilla **Enable Single Sign On** (Habilitar inicio de sesión único).

    e. Haga clic en **Save**(Guardar).

### <a name="create-cakehr-test-user"></a>Creación de un usuario de prueba en CakeHR

Para permitir que los usuarios de Azure AD inicien sesión en CakeHR, es preciso que estén aprovisionados en CakeHR. En CakeHR, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en CakeHR como administrador de seguridad.

2. En el lado izquierdo de la barra de menús, haga clic en **COMPANY** (EMPRESA) > **ADD** (AGREGAR).

    ![Captura de pantalla que muestra CakeHR con COMPANY (EMPRESA) y ADD (AGREGAR) seleccionados.](./media/cakehr-tutorial/account.png)

3. En el elemento emergente **Add new employee** (Agregar nuevo empleado), siga estos pasos:

     ![Captura de pantalla que muestra Add new employee (Agregar nuevo empleado) para realizar estos pasos.](./media/cakehr-tutorial/add-account.png)

    a. En el cuadro de texto **Full name** (Nombre completo), escriba el nombre del usuario, B.Simon.

    b. En el cuadro de texto **Work email** (Correo electrónico del trabajo), escriba el correo electrónico del usuario; por ejemplo, `B.Simon@contoso.com`.

    c. Haga clic en **CREATE ACCOUNT** (CREAR CUENTA).

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la URL de inicio de sesión de CakeHR, desde donde podrá comenzar el flujo de inicio de sesión. 

* Acceda directamente a la URL de inicio de sesión de CakeHR y ponga en marcha el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de CakeHR en Aplicaciones, se le redirigirá a la URL de inicio de sesión de la aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado CakeHR, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
