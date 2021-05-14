---
title: 'Tutorial: Integración de Azure Active Directory con Clarizen One | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Clarizen One.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/08/2021
ms.author: jeedes
ms.openlocfilehash: 62cd714fc20cb7620def8a1aaa3cbccba0b13d57
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108144118"
---
# <a name="tutorial-azure-active-directory-integration-with-clarizen-one"></a>Tutorial: Integración de Azure Active Directory con Clarizen One

En este tutorial aprenderá a integrar Clarizen One con Azure Active Directory (Azure AD). Al integrar Clarizen One con Azure AD, podrá:

* Controlar en Azure AD quién tiene acceso a Clarizen One.
* Permitir que los usuarios inicien sesión automáticamente en Clarizen One con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Clarizen One.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Clarizen One admite inicio de sesión único iniciado por **IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-clarizen-one-from-the-gallery"></a>Incorporación de Clarizen One desde la galería

Para configurar la integración de Clarizen One en Azure AD, es preciso agregar Clarizen One desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Clarizen One** en el cuadro de búsqueda.
1. Seleccione **Clarizen One** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-clarizen-one"></a>Configuración y comprobación del inicio de sesión único de Azure AD con Clarizen One

Configure y pruebe el inicio de sesión único de Azure AD con Clarizen One mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario correspondiente de Clarizen One.

Para configurar y probar el inicio de sesión único de Azure AD con Clarizen One, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Clarizen One](#configure-clarizen-one-sso)** : para configurar el inicio de sesión único en la aplicación.
    1. **[Creación del usuario de prueba en Clarizen One](#create-clarizen-one-test-user)** :para tener un homólogo de B.Simon en Clarizen One vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Clarizen One**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    a. En el cuadro de texto **Identificador**, escriba el valor: `Clarizen`

    b. En el cuadro de texto **URL de respuesta**, escriba la dirección URL: `https://.clarizen.com/Clarizen/Pages/Integrations/SAML/SamlResponse.aspx`

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Clarizen One** (Configurar Clarizen One), copie las direcciones URL que necesite.

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

En esta sección, va a permitir que B.Simon acceda a Clarizen One mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Clarizen One**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-clarizen-one-sso"></a>Configuración del inicio de sesión único en Clarizen One

1. En otra ventana del explorador web, inicie sesión en el sitio de la empresa Clarizen One como administrador.

1. Haga clic en el nombre de usuario y luego haga clic en **Settings** (Configuración).

    ![Clic en "Settings" (Configuración) debajo del nombre de usuario](./media/clarizen-tutorial/setting.png "Configuración")

1. Haga clic en la pestaña **Global Settings** (Configuración global). A continuación, junto a **Federated Authentication** (Autenticación federada), haga clic en **edit** (Editar).

    ![Pestaña Global Settings (Configuración global)](./media/clarizen-tutorial/authentication.png "Configuración global")

1. En el cuadro de diálogo **Federated Authentication** (Autenticación federada), realice los pasos siguientes:

    ![Cuadro de diálogo "Federated Authentication" (Autenticación federada)](./media/clarizen-tutorial/federated-authentication.png "Federated Authentication")

    a. Seleccione **Habilitar autenticación federada**.

    b. Haga clic en **Cargar** para cargar el certificado descargado.

    c. En el cuadro **URL de inicio de sesión**, escriba el valor de **Dirección URL de inicio de sesión** de la ventana de configuración de aplicaciones de Azure AD.

    d. En el cuadro **Sign-out URL** (Dirección URL de cierre de sesión), escriba el valor de **Logout URL** (Dirección URL de cierre de sesión) de la ventana de configuración de aplicaciones de Azure AD.

    e. Seleccione **Usar POST**.

    f. Haga clic en **Save**(Guardar).

### <a name="create-clarizen-one-test-user"></a>Creación de un usuario de prueba en Clarizen One

El objetivo de esta sección es crear un usuario llamado B.Simon en Clarizen One.

**Para crear un usuario manualmente, siga los pasos siguientes:**

Para permitir que los usuarios de Azure AD inicien sesión en Clarizen One, debe aprovisionar las cuentas de usuario. En el caso de Clarizen One, el aprovisionamiento es una tarea manual.

1. Inicie sesión en el sitio de la compañía Clarizen One como administrador.

2. Haga clic en **Contactos**.

    ![Clic en "People" (Personas)](./media/clarizen-tutorial/people.png "Personas")

3. Haga clic en **Invitar usuario**.

    ![Botón "Invite user" (Invitar a usuario)](./media/clarizen-tutorial/user.png "Invitar a usuarios")

1. En el cuadro de diálogo **Invite People** (Invitar a personas), siga estos pasos:

    ![Cuadro de diálogo "Invite People" (Invitar a personas)](./media/clarizen-tutorial/invite-people.png "Invitar a contactos")

    a. En el cuadro **Email** (Correo electrónico), escriba la dirección de correo electrónico de la cuenta de Britta Simon.

    b. Haga clic en **Invitar**.

    > [!NOTE]
    > El titular de la cuenta de Azure Active Directory recibirá un mensaje de correo y seguirá un vínculo para confirmar su cuenta antes de que se active.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

* En Azure Portal, haga clic en Probar esta aplicación. Al hacerlo, debería iniciar sesión automáticamente en la instancia de Clarizen One en la que configurara el inicio de sesión único.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Clarizen One en Aplicaciones, se debería iniciar sesión automáticamente en la versión de Clarizen One para la que configurara el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurada Clarizen One, podrá aplicar el control de sesión, que protege la información confidencial de la organización de la filtración y la infiltración en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).