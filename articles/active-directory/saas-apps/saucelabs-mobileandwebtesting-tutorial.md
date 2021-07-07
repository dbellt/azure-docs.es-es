---
title: 'Tutorial: Integración de Azure Active Directory con Sauce Labs - Mobile and Web Testing | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y Sauce Labs - Mobile and Web Testing.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/20/2021
ms.author: jeedes
ms.openlocfilehash: 754fb3f80bfdf44205d43461c4b09b5ea7b20072
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110481683"
---
# <a name="tutorial-azure-active-directory-integration-with-sauce-labs---mobile-and-web-testing"></a>Tutorial: Integración de Azure Active Directory con Sauce Labs - Mobile and Web Testing

En este tutorial, obtendrá información sobre cómo integrar Sauce Labs - Mobile and Web Testing con Azure Active Directory (Azure AD). Al integrar Sauce Labs - Mobile and Web Testing con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a Sauce Labs - Mobile and Web Testing.
* Permitir que los usuarios inicien sesión automáticamente en Sauce Labs - Mobile and Web Testing con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con Sauce Labs - Mobile and Web Testing, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único de Sauce Labs - Mobile and Web Testing.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Sauce Labs - Mobile and Web Testing admite el inicio de sesión único iniciado por **IDP**.
* Sauce Labs - Mobile and Web Testing admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="add-sauce-labs---mobile-and-web-testing-from-the-gallery"></a>Adición de Sauce Labs - Mobile and Web Testing desde la galería

Para configurar la integración de Sauce Labs - Mobile and Web Testing en Azure AD, deberá agregar Sauce Labs - Mobile and Web Testing desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Sauce Labs - Mobile and Web Testing** en el cuadro de búsqueda.
1. Seleccione **Sauce Labs - Mobile and Web Testing en** el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-sauce-labs---mobile-and-web-testing"></a>Configuración y prueba del inicio de sesión único de Azure AD para Sauce Labs - Mobile and Web Testing

Configure y pruebe el inicio de sesión único de Azure AD con Sauce Labs - Mobile and Web Testing mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Sauce Labs - Mobile and Web Testing.

Para configurar el inicio de sesión único de Azure AD con Sauce Labs - Mobile and Web Testing, realice los pasos siguientes:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Sauce Labs - Mobile and Web Testing](#configure-sauce-labs---mobile-and-web-testing-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Sauce Labs - Mobile and Web Testing](#create-sauce-labs---mobile-and-web-testing-test-user)** : para tener un homólogo de B.Simon en Sauce Labs - Mobile and Web Testing que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Sauce Labs - Mobile and Web Testing**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, el usuario no tiene que realizar ningún paso porque la aplicación ya se ha integrado previamente con Azure.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Configurar Sauce Labs - Mobile and Web Testing**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, habilitará a B.Simon para que use el inicio de sesión único de Azure concediéndole acceso a Sauce Labs - Mobile and Web Testing.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Sauce Labs - Mobile and Web Testing**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-sauce-labs---mobile-and-web-testing-sso"></a>Configuración del inicio de sesión único de Sauce Labs - Mobile and Web Testing

1. En otra ventana del explorador web, inicie sesión en su sitio de la compañía de Sauce Labs - Mobile and Web Testing como administrador.

2. Haga clic en el **icono de usuario** y seleccione la pestaña **Administración del equipo**.

    ![Captura de pantalla que muestra el icono "User" (Usuario) y la lista desplegable "Team Management" (Administración del equipo) seleccionada.](./media/saucelabs-mobileandwebtesting-tutorial/user.png)

3. Escriba su **Nombre de dominio** en el cuadro de texto.

    ![Captura de pantalla que muestra un nombre de dominio de ejemplo en el cuadro de texto.](./media/saucelabs-mobileandwebtesting-tutorial/domain.png)

4. Haga clic en la pestaña **Configurar**.

    ![Captura de pantalla que muestra la pestaña "Configure" (Configurar) en "Single Sign On is Enabled" (Inicio de sesión único habilitado).](./media/saucelabs-mobileandwebtesting-tutorial/configure.png)

5. En la sección **Configurar inicio de sesión único**, siga estos pasos.

    ![Configurar inicio de sesión único](./media/saucelabs-mobileandwebtesting-tutorial/browse.png)

    a. Haga clic en **Examinar** y cargue el archivo de metadatos descargado de Azure AD.

    b. Seleccione la casilla **PERMITIR APROVISIONAMIENTO JUST-IN-TIME**.

    c. Haga clic en **Save**(Guardar).

### <a name="create-sauce-labs---mobile-and-web-testing-test-user"></a>Creación de un usuario de prueba de Sauce Labs - Mobile and Web Testing

En esta sección, se crea un usuario llamado Britta Simon en Sauce Labs - Mobile and Web Testing. Sauce Labs - Mobile and Web Testing admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si el usuario no existe en Sauce Labs - Mobile and Web Testing, se crea uno después de la autenticación.

> [!Note]
> Si necesita crear manualmente un usuario, póngase en contacto con el [equipo de soporte técnico de Sauce Labs - Mobile and Web Testing](mailto:support@saucelabs.com).

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

* Haga clic en Probar esta aplicación en Azure Portal; debería iniciar sesión automáticamente en la instancia de Sauce Labs - Mobile and Web Testing para la que ha configurado el inicio de sesión único.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Sauce Labs - Mobile and Web Testing en Aplicaciones, debería iniciar sesión automáticamente en la versión de Sauce Labs - Mobile and Web Testing para la que configurara el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Sauce Labs - Mobile and Web Testing, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
