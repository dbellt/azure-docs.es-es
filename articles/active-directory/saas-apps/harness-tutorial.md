---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Harness | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Harness.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/18/2021
ms.author: jeedes
ms.openlocfilehash: 9452358c7549209bd628e21d5eb702a07d6f39f1
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110473024"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-harness"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Harness

En este tutorial, obtendrá información sobre cómo integrar Harness con Azure Active Directory (Azure AD). Al integrar Harness con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Harness.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Harness con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Harness

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Harness admite el SSO iniciado por **SP e IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-harness-from-the-gallery"></a>Adición de Harness desde la galería

Para configurar la integración de Harness en Azure AD, será preciso que agregue Harness desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Harness** en el cuadro de búsqueda.
1. Seleccione **Harness** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-harness"></a>Configuración y prueba del inicio de sesión único de Azure AD para Harness

Configure y pruebe el inicio de sesión único de Azure AD con Harness mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Harness.

Para configurar y probar el inicio de sesión único de Azure AD con Harness, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Harness](#configure-harness-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Harness](#create-harness-test-user)** , para tener un homólogo de B. Simon en Harness que esté vinculado a su representación en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Harness**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice el siguiente paso:

    En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://app.harness.io/gateway/api/users/saml-login?accountId=<harness_account_id>`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL: `https://app.harness.io/`

    > [!NOTE]
    > El valor de dirección URL de respuesta no es real. Obtendrá la dirección URL de respuesta real en la sección **Configuración del inicio de sesión único de Harness**, que se explica más adelante en el tutorial. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar Harness**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B. Simon acceda a Harness mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Harness**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-harness-sso"></a>Configuración del inicio de sesión único de Harness

1. Para automatizar la configuración en Harness, debe instalar la **extensión del explorador de inicio de sesión seguro de Aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Configurar Harness** para ir a la aplicación Harness. En ella, escriba las credenciales de administrador para iniciar sesión en Harness. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 6.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar Harness manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Harness como administrador y haga lo siguiente:

4. En la parte superior derecha de la página, haga clic en **Continuous Security (Seguridad continua)**  > **Access Management (Administración de acceso)**  > **Authentication Settings (Configuración de autenticación)** .

    ![Captura de pantalla que muestra el menú "Continuous Security" (Seguridad continua) con las opciones "Access Management" (Administración de acceso) y "Authentication Settings" (Configuración de autenticación) seleccionadas.](./media/harness-tutorial/authentication.png)

5. En la sección **SSO Providers** (Proveedores de inicio de sesión único), haga clic en **+ Add SSO Providers (+ Agregar proveedores de inicio de sesión único)**  > **SAML**.

    ![Captura de pantalla que muestra la sección "S S O Providers" (Proveedores de inicio de sesión único) con la opción "+ Add S S O Providers - S A M L" (+ Agregar proveedores de inicio de sesión único: S A M L) seleccionada.](./media/harness-tutorial/providers.png)

6. En el menú desplegable **SAML Provider** (Proveedor de SAML), lleve a cabo estos pasos:

    ![Capturar de pantalla que muestra el elemento emergente "S A M L Provider" (Proveedor de S A M L) con los campos "U R L" (Dirección U R L) y "Display Name" (Nombre para mostrar) resaltados y los botones "Choose File" (Elegir archivo) y "Submit" (Enviar) seleccionados.](./media/harness-tutorial/file.png)

    a. Copie la instancia de **In your SSO Provider, please enable SAML-based login, then enter the following URL** (En su proveedor de SSO, habilite el inicio dirección basado en SAML y, después, escriba la siguiente dirección URL de respuesta) y péguelo en el cuadro de texto Dirección URL de respuesta en **Configuración básica de SAML** en Azure Portal.

    b. En el cuadro de texto **Nombre para mostrar**, escriba el nombre para mostrar.

    c. Haz clic en **Choose File** (Elegir archivo) para cargar el archivo XML de metadatos que haya descargado desde Azure AD.

    d. Haga clic en **ENVIAR**.

### <a name="create-harness-test-user"></a>Creación de un usuario de prueba de Harness

Para permitir que los usuarios de Azure AD inicien sesión en Harness, deben aprovisionarse en Harness. En Harness, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión como administrador en Harness.

1. En la parte superior derecha de la página, haga clic en **Continuous Security (Seguridad continua)**  > **Access Management (Administración de acceso)**  > **Users (Usuarios)** .

    ![Captura de pantalla que muestra el menú "Continuous Security" (Seguridad continua) con las opciones "Access Management" (Administración de acceso) y "Users" (Usuario) seleccionadas.](./media/harness-tutorial/users.png)

1. En el lado derecho de la página, haga clic en **+ Add User** (Agregar usuario).

    ![Captura de pantalla que muestra la página "Users" (Usuarios) con la acción "+ Add Users" (Agregar usuario) seleccionada.](./media/harness-tutorial/add-user.png)

1. En el elemento emergente **Add User** (Agregar usuario), lleve a cabo los pasos siguientes:

    ![Configuración de Harness](./media/harness-tutorial/configure.png)

    a. En el cuadro de texto **Email Address(es)** (Direcciones de correo electrónico), escriba el correo electrónico del usuario; por ejemplo, `B.simon@contoso.com`.

    b. Seleccione su **User Groups** (Grupos de usuarios).

    c. Haga clic en **Enviar**.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Harness, donde puede iniciar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de Harness e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de Harness para la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Harness en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de Harness para la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Harness, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
