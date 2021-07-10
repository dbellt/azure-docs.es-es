---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con iSAMS | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y iSAMS.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/10/2021
ms.author: jeedes
ms.openlocfilehash: a43284ce4dca5b06a7c5cb32ac0d5c4691320350
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111985546"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-isams"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con iSAMS

En este tutorial, obtendrá información sobre cómo integrar iSAMS con Azure Active Directory (Azure AD). Al integrar iSAMS con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a iSAMS.
* Permitir que los usuarios puedan iniciar sesión automáticamente en iSAMS con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción habilitada para el inicio de sesión único (SSO) en iSAMS.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* iSAMS admite el inicio de sesión único iniciado por **SP e IDP**.

## <a name="add-isams-from-the-gallery"></a>Incorporación de iSAMS desde la galería

Para configurar la integración de iSAMS en Azure AD, deberá agregar iSAMS desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **iSAMS** en el cuadro de búsqueda.
1. Seleccione **iSAMS** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-isams"></a>Configuración y prueba del inicio de sesión único de Azure AD para iSAMS

Configure y pruebe el inicio de sesión único (SSO) de Azure AD con iSAMS mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de iSAMS.

Para configurar y probar el inicio de sesión único de Azure AD con iSAMS, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en iSAMS](#configure-isams-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de iSAMS](#create-isams-test-user)** , para tener un homólogo de B.Simon en iSAMS vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **iSAMS**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice los siguientes pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<SUBDOMAIN>.isams.cloud/main/sso/saml2`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.isams.cloud/main/sso/saml2/acs`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.isams.cloud/`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de iSAMS](mailto:support@isams.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar la **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

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

En esta sección, va a permitir que B.Simon acceda a iSAMS mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **iSAMS**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-isams-sso"></a>Configuración del inicio de sesión único de iSAMS

1. Inicie sesión en iSAMS como administrador.

1. Vaya al panel de control y abra el módulo **Autenticación**.

1. En el menú de la derecha, seleccione **Proveedores de identidades**.

    ![Captura de pantalla Configuración de Active Directory con la opción Proveedores de identidades seleccionada.](./media/isams-tutorial/click-identity-provider.png)

1. Seleccione **Agregar proveedor**.

    ![Captura de pantalla que muestra Proveedores de identidades con la opción Agregar proveedores seleccionada.](./media/isams-tutorial/add-identity-provider.png)

1. En la página siguiente, realice estos pasos:

    ![Captura de pantalla que muestra el asistente para proveedores de identidades, donde puede realizar los pasos descritos.](./media/isams-tutorial/configure-isams.png)

    a. En el cuadro de texto **Nombre**, proporcione un nombre válido, como `Saml2 Azure`. Este es el nombre que aparecerá en la página de inicio de sesión.

    b. En el cuadro URL de metadatos, pegue la **dirección URL de metadatos de federación de aplicación** que copió en Azure Portal.
    
    c. Presione **Importar**.
    
    d. En el cuadro de lista **Aplicaciones** de la sección **Enabled Client Applications** (Aplicaciones cliente habilitadas), seleccione todas las aplicaciones iSAMS en las que desee que aparezca el proveedor en la página de inicio de sesión.

    e. Haga clic en **Guardar y cerrar**.

### <a name="create-isams-test-user"></a>Creación de un usuario de prueba de iSAMS

1. Inicie sesión en iSAMS como administrador.

2.  Vaya a **Control Panel Home** -> **Security & Permissions** -> **User Accounts** -> **User Options & Tasks** -> **Modify User Properties** (Ventana principal del panel de control > Seguridad y permisos > Cuentas de usuario > Opciones y tareas de usuario > Modificar propiedades de usuario).

    ![Captura de pantalla que muestra la página User Accounts (Cuentas de usuario) con la opción Modify User Properties (Modificar propiedades de usuario) seleccionada.](./media/isams-tutorial/modify-user-properties.png)


3. En la ventana emergente resultante, seleccione la pestaña **Detalles de la cuenta** y cambie la **Autorización** a la del proveedor de identidades que acaba de crear.

    ![Captura de pantalla que muestra los detalles de la cuenta con un valor en Authorization (Autorización).](./media/isams-tutorial/account-details.png)

4. Haga clic en **Guardar y cerrar**.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de iSAMS, donde puede poner en marcha el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de iSAMS y ponga en marcha el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de iSAMS para la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de iSAMS en Mis aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de iSAMS para la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado iSAMS, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
