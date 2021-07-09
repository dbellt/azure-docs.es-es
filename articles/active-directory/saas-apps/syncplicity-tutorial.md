---
title: 'Tutorial: integración de Azure Active Directory con Syncplicity | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Syncplicity.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/21/2021
ms.author: jeedes
ms.openlocfilehash: d0da89d8dfa4c577648429892dec731a08f1bed7
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110458182"
---
# <a name="tutorial-integrate-syncplicity-with-azure-active-directory"></a>Tutorial: Integración de Syncplicity con Azure Active Directory

En este tutorial, aprenderá a integrar Syncplicity con Azure Active Directory (Azure AD). Al integrar Syncplicity con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Syncplicity.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Syncplicity con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Syncplicity.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. 
* Syncplicity admite inicio de sesión único iniciado por **SP**.

## <a name="add-syncplicity-from-the-gallery"></a>Incorporación de Syncplicity desde la galería

Para configurar la integración de Syncplicity en Azure AD, deberá agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. En **Crear**, haga clic en **Aplicación empresarial**.
1. En la sección **Examinar la Galería de Azure AD**, escriba **Syncplicity** en el cuadro de búsqueda.
1. Seleccione **Syncplicity** en el panel de resultados y haga clic en **Crear** para agregar la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-syncplicity"></a>Configuración y prueba del SSO de Azure AD para Syncplicity

Configure y pruebe el inicio de sesión único de Azure AD con Syncplicity mediante un usuario de prueba llamado **B.Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Syncplicity.

Para configurar y probar el SSO de Azure AD con Syncplicity, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del SSO en Syncplicity](#configure-syncplicity-sso)** : para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Syncplicity](#create-syncplicity-test-user)**: para tener un homólogo de B.Simon en Syncplicity vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.
1. **[Actualización del SSO](#update-sso)** : para realizar los cambios necesarios en Syncplicity si ha cambiado la configuración del SSO en Azure AD.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Syncplicity**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<COMPANY_NAME>.syncplicity.com/sp`

    b. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<COMPANY_NAME>.syncplicity.com`
    
    c. En el cuadro de texto **URL de respuesta (URL del Servicio de consumidor de aserciones)** , escriba una URL con el siguiente patrón: `https://<COMPANY_NAME>.syncplicity.com/Auth/AssertionConsumerService.aspx`.

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de respuesta, la dirección URL de inicio de sesión y el identificador reales. Póngase en contacto con el [equipo de soporte de cliente de Syncplicity](https://www.syncplicity.com/contact-us) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Editar**. Después, en el cuadro de diálogo, haga clic en el botón de puntos suspensivos junto al certificado activo y seleccione **Descarga de certificados PEM**.

   ![Vínculo de descarga del certificado](common/certificatebase64.png)

    > [!NOTE]
    > Necesita el certificado PEM, ya que Syncplicity no acepta certificados en formato CER.

6. En la sección **Configuración de Syncplicity**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Syncplicity utilizando el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Syncplicity**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-syncplicity-sso"></a>Configuración del inicio de sesión único de Syncplicity

1. Inicie sesión en el inquilino de **Syncplicity**.

1. En el menú en la parte superior, haga clic en **Admin** (administración), seleccione **Settings** (configuración) y luego haga clic en **Custom domain and single sign-on** (dominio personalizado e inicio de sesión único).

    ![Syncplicity](./media/syncplicity-tutorial/admin.png "Syncplicity")

1. En la página del cuadro de diálogo **Inicio de sesión único (SSO)**, siga estos pasos:

    ![Inicio de sesión único \(SSO\)](./media/syncplicity-tutorial/configuration.png "Single Sign-On \\\(SSO\\\)")

    a. En el cuadro de texto **Dominio personalizado**, escriba el nombre del dominio.
  
    b. Seleccione **Habilitado** como **Estado de inicio de sesión único**.

    c. En el cuadro de texto **Identificador de entidad**, pegue el valor del **identificador (identificador de entidad)** que usó en la **configuración básica de SAML** de Azure Portal.

    d. En el cuadro de texto **Sign-in page URL** (dirección URL de página de inicio de sesión), pegue la **URL de inicio de sesión** que copió de Azure Portal.

    e. En el cuadro de texto **Logout page URL** (dirección URL de la página de cierre de sesión), pegue el valor de la **URL de cierre de sesión** que copió de Azure Portal.

    f. En **Identity Provider Certificate** (Certificado del proveedor de identidades), haga clic en **Elegir archivo** y cargue el certificado que ha descargado de Azure Portal.

    g. Haga clic en **GURDAR CAMBIOS**.

### <a name="create-syncplicity-test-user"></a>Creación de un usuario de prueba de Syncplicity

Para que los usuarios de Azure AD puedan iniciar sesión, estos deben aprovisionarse en la aplicación Syncplicity. En esta sección se describe cómo crear cuentas de usuario de Azure AD en Syncplicity.

**Para aprovisionar cuentas de usuario a Syncplicity, realice los siguientes pasos:**

1. Inicie sesión en su inquilino de **Syncplicity** (por ejemplo, `https://company.Syncplicity.com`).

1. Haga clic en **Admin** (administración), seleccione **User Accounts** (cuentas de usuario) y, a continuación, haga clic en **Add a User** (agregar un usuario).

    ![Administración de usuarios](./media/syncplicity-tutorial/users.png "Administrar usuarios")

1. Escriba las **Email addresses** (direcciones de correo electrónico) de una cuenta de Azure AD que quiera aprovisionar, seleccione **User** (usuario) como **Role** (rol) y haga clic en **Next** (siguiente).

    ![Información de cuenta](./media/syncplicity-tutorial/roles.png "Información de cuenta")

    > [!NOTE]
    > El titular de la cuenta de Azure AD recibirá un mensaje de correo electrónico junto con un vínculo para confirmar y activar la cuenta.

1. Seleccione un grupo de la empresa del que formará parte el nuevo usuario y, a continuación, haga clic en **Siguiente**.

    ![Pertenencia a grupos](./media/syncplicity-tutorial/group.png "Pertenencia a grupos")

    > [!NOTE]
    > Si no se muestra ningún grupo, haga clic en **Next**(siguiente).

1. Seleccione las carpetas que desea que Syncplicity controle en el equipo del usuario y, a continuación, haga clic en **Siguiente**.

    ![Carpetas de Syncplicity](./media/syncplicity-tutorial/folder.png "Carpetas de Syncplicity")

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de Syncplicity ofrecida por Syncplicity para aprovisionar cuentas de usuario de Azure AD.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. De este modo, accederá automáticamente a la dirección URL de inicio de sesión de Syncplicity, donde podrá comenzar el flujo de inicio de sesión. 

* Acceda directamente a la dirección URL de inicio de sesión de Syncplicity y ponga en marcha el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Syncplicity en Aplicaciones y se le redirigirá a la dirección URL de inicio de sesión de esa plataforma. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

### <a name="update-sso"></a>Actualización del inicio de sesión único

Siempre que necesite realizar cambios en el inicio de sesión único, debe comprobar el **Certificado de firma de SAML** que se está usando. Si el certificado ha cambiado, asegúrese de cargar el nuevo en Syncplicity, tal y como se describe en **[Configuración del inicio de sesión único de Syncplicity](#configure-syncplicity-sso)** .

Si usa la aplicación móvil de Syncplicity, póngase en contacto con el servicio de soporte al cliente de Syncplicity ( support@syncplicity.com) para obtener ayuda.

## <a name="next-steps"></a>Pasos siguientes

Una vez que se ha configurado Syncplicity, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
