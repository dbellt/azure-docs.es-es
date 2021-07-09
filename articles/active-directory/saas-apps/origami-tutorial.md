---
title: 'Tutorial: Integración de Azure Active Directory con Origami | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Origami.
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
ms.openlocfilehash: cb00f7e5789e7c5ff4a1e3a7fc8265b1ce7edadf
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111888108"
---
# <a name="tutorial-azure-active-directory-integration-with-origami"></a>Tutorial: Integración de Azure Active Directory con Origami

En este tutorial, aprenderá a integrar Origami con Azure Active Directory (Azure AD). La integración de Origami con Azure AD permite:

* Controlar en Azure AD quién tiene acceso a Origami.
* Habilitar que los usuarios inicien sesión automáticamente en Origami con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Origami.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Origami admite el inicio de sesión único iniciado por **SP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-origami-from-the-gallery"></a>Adición de Origami desde la galería

Para configurar la integración de Origami en Azure AD, deberá agregar Origami desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Origami** en el cuadro de búsqueda.
1. Seleccione **Origami** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-origami"></a>Configuración y prueba del inicio de sesión único de Azure AD para Origami

Configure y pruebe el inicio de sesión único de Azure AD con Origami mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es preciso vincular un usuario de Azure AD y el usuario de Origami correspondiente.

Para configurar y probar el inicio de sesión único de Azure AD con Origami, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Origami](#configure-origami-sso)** : para configurar los valores del inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Origami](#create-origami-test-user)** : el objetivo es tener un homólogo de B. Simon en Origami que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Origami**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://live.origamirisk.com/origami/account/login?account=<COMPANY_NAME>`

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de atención al cliente de Origami ](https://wordpress.org/support/theme/origami) para obtener este valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Origami** (Configurar Origami), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B. Simon acceda a Origami utilizando el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Origami**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-origami-sso"></a>Configuración del inicio de sesión único de Origami

1. Inicie sesión en la cuenta de Origami con derechos de administrador.

2. En el menú de la parte superior, haga clic en **Administrador**.
   
    ![Captura de pantalla que muestra la página principal de Origami con la opción "Admin" (Administrador) seleccionada.](./media/origami-tutorial/admin.png)

3. En la página del cuadro de diálogo Single Sign On Setup (Configuración de inicio de sesión único), siga estos pasos:
   
    ![Captura de pantalla que muestra la página "Single Sign On Setup" (Configuración de inicio de sesión único) con la opción "Enable Single Sign-on" (Habilitar inicio de sesión único) seleccionada y los cuadros de texto resaltados.](./media/origami-tutorial/configuration.png)

    a. Seleccione **Enable Single Sign On**(Habilitar el inicio de sesión único).

    b. En el cuadro de texto **Identity Provider’s Sign-On URL** (Dirección URL de la página de inicio de sesión del proveedor de identidades), pegue el valor de **Dirección URL de inicio de sesión**  que ha copiado de Azure Portal.

    c. En el cuadro de texto **Identity Provider’s Single Sign-out URL** (Dirección URL de la página de cierre de sesión del proveedor de identidades), pegue el valor de **Dirección URL de cierre de sesión**  que ha copiado de Azure Portal.

    d. Haga clic en **Browse** (Examinar) para cargar el certificado que ha descargado de Azure Portal.

    e. Haga clic en **Guardar cambios**.

### <a name="create-origami-test-user"></a>Creación de un usuario de prueba en Origami

En esta sección, creará un usuario llamado Britta Simon en Origami. 

1. Inicie sesión en la cuenta de Origami con derechos de administrador.

2. En el menú de la parte superior, haga clic en **Administrador**.
   
    ![Captura de pantalla que muestra la página principal de la cuenta de Origami con la opción "Admin" (Administrador) seleccionada.](./media/origami-tutorial/admin.png)

3. En el cuadro de diálogo **Users and Security** (Usuarios y seguridad), haga clic en **Users** (Usuarios).
   
    ![Captura de pantalla que muestra el cuadro de diálogo "Users and Security" (Usuarios y seguridad) con la opción "Users" (Usuarios) seleccionada.](./media/origami-tutorial/user.png)

4. Haga clic en **Add New User**(Agregar nuevo usuario).
   
    ![Captura de pantalla que muestra el botón "Add New User" (Agregar nuevo usuario) seleccionado.](./media/origami-tutorial/add-user.png)

5. En el cuadro de diálogo Add New User (Agregar nuevo usuario), realice los pasos siguientes:
   
    ![Captura de pantalla que muestra el cuadro de diálogo "Add New User" (Agregar nuevo usuario) con los cuadros de texto "User Name" (Nombre de usuario), "First Name" (Nombre) y "Last Name" (Apellidos) resaltados.](./media/origami-tutorial/new-user.png)

    a. En el cuadro de texto **User Name** (Nombre de usuario), escriba el correo electrónico del usuario; por ejemplo, **brittasimon\@contoso.com**.

    b. En el cuadro de texto **Password** (Contraseña), escriba una contraseña.

    c. En el cuadro de texto **Confirm Password** (Confirmar contraseña), escriba la contraseña de nuevo.

    d. En el cuadro de texto **First Name** (Nombre), escriba el nombre de usuario, en este caso **Britta**.

    e. En el cuadro de texto **Last Name** (Apellidos), escriba el apellido del usuario, en este caso **Simon**.

    f. Haga clic en **Save**(Guardar).
   
    ![Captura de pantalla que muestra el botón "Save" (Guardar) seleccionado.](./media/origami-tutorial/save.png)

6. Asigne **User Roles** (Roles de usuario) y **Client Access** (Acceso de cliente) al usuario. 
   
    ![Configurar inicio de sesión único](./media/origami-tutorial/user-roles.png)

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Origami, desde donde podrá comenzar el flujo de inicio de sesión. 

* Acceda directamente a la dirección URL de inicio de sesión de Origami y ponga en marcha el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Origami en Aplicaciones, se le redirigirá a la URL de inicio de sesión de la aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Tras configurar Origami, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
