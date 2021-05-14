---
title: 'Tutorial: Integración de Azure Active Directory con la plataforma de productividad ClickUp | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y la plataforma de productividad ClickUp.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/30/2021
ms.author: jeedes
ms.openlocfilehash: 617293cf3e74dc58b60fe090f6a71091bcc94684
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108144079"
---
# <a name="tutorial-azure-active-directory-integration-with-clickup-productivity-platform"></a>Tutorial: Integración de Azure Active Directory con la plataforma de productividad ClickUp

En este tutorial aprenderá a integrar la plataforma de productividad ClickUp con Azure Active Directory (Azure AD). Al integrar ClickUp Productivity Platform con Azure AD, puede realizar lo siguiente:

* Controlar en Azure AD quién tiene acceso a la plataforma de productividad ClickUp.
* Permitir que los usuarios inicien sesión automáticamente en ClickUp Productivity Platform con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción que permita el inicio de sesión único en ClickUp Productivity Platform.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* ClickUp Productivity Platform admite el inicio de sesión único iniciado por **SP**.

## <a name="add-clickup-productivity-platform-from-the-gallery"></a>Incorporación de ClickUp Productivity Platform desde la galería

Para configurar la integración de ClickUp en Azure AD, deberá agregar la aplicación de la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **ClickUp Productivity Platform** en el cuadro de búsqueda.
1. Seleccione **ClickUp Productivity Platform** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-clickup-productivity-platform"></a>Configuración y prueba del inicio de sesión único de Azure AD para ClickUp Productivity Platform

Configure y pruebe el inicio de sesión único de Azure AD con ClickUp Productivity Platform mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de ClickUp Productivity Platform.

Para configurar y probar el inicio de sesión único de Azure AD con ClickUp Productivity Platform, realice los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en ClickUp Productivity Platform](#configure-clickup-productivity-platform-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación del usuario de prueba de ClickUp](#create-clickup-productivity-platform-test-user)** : para tener un homólogo de B.Simon en ClickUp Productivity Platform vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **ClickUp Productivity Platform**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **Dirección URL de inicio de sesión**, escriba la dirección URL: `https://app.clickup.com/login/sso`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://api.clickup.com/v1/team/<team_id>/microsoft`

    > [!NOTE]
    > El valor del identificador no es real. Actualícelo con el valor del identificador real, que se explica más adelante en este tutorial.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

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

En esta sección, habilitará a B.Simon para que use el inicio de sesión único de Azure concediéndole acceso a ClickUp Productivity Platform.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **ClickUp**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-clickup-productivity-platform-sso"></a>Configuración del inicio de sesión único en ClickUp Productivity Platform

1. En otra ventana del explorador web, inicie sesión en su inquilino de ClickUp como administrador.

2. Haga clic en **User profile** (Perfil de usuario) y, a continuación, seleccione **Settings** (Configuración).

    ![Captura de pantalla que muestra el inquilino de ClickUp con el icono de configuración seleccionado.](./media/clickup-productivity-platform-tutorial/configure-0.png)

    ![Captura de pantalla que muestra Settings (Configuración).](./media/clickup-productivity-platform-tutorial/configure-1.png)

3. Como proveedor de inicio de sesión único (SSO), seleccione **Microsoft**.

    ![Captura de pantalla que muestra el panel Authentication (Autenticación) con Microsoft seleccionado.](./media/clickup-productivity-platform-tutorial/configure-2.png)

4. En la página **Configure Microsoft Single Sign On** (Configurar inicio de sesión único de Microsoft), realice los siguientes pasos:

    ![Captura de pantalla que muestra la página Configure Microsoft Single Sign On (Configurar inicio de sesión único de Microsoft), donde puede copiar el identificador de entidad y guardar la dirección U R L de los metadatos de federación de Azure.](./media/clickup-productivity-platform-tutorial/configure-3.png)

    a. Haga clic en **Copy** (Copiar) para copiar el identificador de entidad y pegarlo en el cuadro de texto **Identificador (id. de entidad)** de la sección **Configuración básica de SAML** en Azure Portal.

    b. En el cuadro de texto **Azure Federation Metadata URL** (Dirección URL de los metadatos de federación de Azure), pegue el valor de la URL de los metadatos de federación de la aplicación que copió de Azure Portal y haga clic en **Save** (Guardar).

5. Para completar la instalación, haga clic en **Authenticate With Microsoft to complete setup** (Autenticar con Microsoft para completar la instalación) y autentíquese con la cuenta Microsoft.

    ![Captura de pantalla que muestra el botón Authenticate with Microsoft to complete setup (Autenticar con Microsoft para completar la instalación).](./media/clickup-productivity-platform-tutorial/configure-4.png)


### <a name="create-clickup-productivity-platform-test-user"></a>Creación del usuario de prueba en ClickUp

1. En otra ventana del explorador web, inicie sesión en su inquilino de ClickUp como administrador.

2. Haga clic en **User profile** (Perfil de usuario) y, a continuación, seleccione **People** (Personas).

    ![Captura de pantalla que muestra el inquilino de ClickUp.](./media/clickup-productivity-platform-tutorial/configure-0.png)

    ![Captura de pantalla que muestra el vínculo People (Personas) seleccionado.](./media/clickup-productivity-platform-tutorial/user-1.png)

3. Escriba la dirección de correo electrónico del usuario en el cuadro de texto y haga clic en **Invite** (Invitar).

    ![Captura de pantalla que muestra la configuración de usuarios del equipo, donde puede invitar a personas por correo electrónico.](./media/clickup-productivity-platform-tutorial/user-2.png)

    > [!NOTE]
    > El usuario recibirá la notificación y tendrá que aceptar la invitación para activar la cuenta.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la URL de inicio de sesión de ClickUp Productivity Platform, donde podrá poner en marcha el flujo de inicio de sesión. 

* Acceda directamente a la URL de inicio de sesión de ClickUp Productivity Platform y ponga en marcha el flujo de inicio de sesión desde ahí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de ClickUp Productivity Platform en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de ClickUp Productivity Platform. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado ClickUp Productivity Platform, puede aplicar el control de sesión que protege la información confidencial de la organización de la filtración y la infiltración en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).