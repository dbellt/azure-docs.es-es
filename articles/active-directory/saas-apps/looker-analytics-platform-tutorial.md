---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Looker Analytics Platform | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Looker Analytics Platform.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/10/2020
ms.author: jeedes
ms.openlocfilehash: ed4cfb9362e20bb1ad8c25caa5579fddaa147e8d
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108764304"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-looker-analytics-platform"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Looker Analytics Platform

En este tutorial, aprenderá a integrar Looker Analytics Platform con Azure Active Directory (Azure AD). Al integrar la Looker Analytics Platform con Azure AD, podrá hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Looker Analytics Platform.
* Permitir que los usuarios inicien sesión automáticamente en Looker Analytics Platform con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) de Looker Analytics Platform.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Looker Analytics Platform admite el inicio de sesión único activado habilitado **SP e IDP**.
* Looker Analytics Platform admite el aprovisionamiento de usuarios **Just-In-Time**.


## <a name="adding-looker-analytics-platform-from-the-gallery"></a>Incorporación de Looker Analytics Platform desde la galería

Para configurar la integración de Looker Analytics Platform en Azure AD, es necesario agregar la aplicación desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Looker Analytics Platform** en el cuadro de búsqueda.
1. Seleccione **Looker Analytics Platform** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-looker-analytics-platform"></a>Configuración y prueba del inicio de sesión único de Azure AD para Looker Analytics Platform

Configure y pruebe el inicio de sesión único de Azure AD con Looker Analytics Platform mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente en Looker Analytics Platform.

Para configurar y probar el inicio de sesión único de Azure AD con Looker Analytics Platform, complete los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Looker Analytics Platform](#configure-looker-analytics-platform-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Looker Analytics Platform](#create-looker-analytics-platform-test-user)** , para tener un usuario equivalente a B.Simon en la aplicación que esté vinculado a su representación en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Looker Analytics Platform**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **SP Entity/IdP Audience** (Entidad SP/audiencia IdP), escriba una dirección URL con el siguiente patrón: `<SPN>_looker`.

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.looker.com/samlcallback`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.looker.com`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte al cliente de Looker Analytics Platform](mailto:support@looker.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar Looker Analytics Platform**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Looker Analytics Platform mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Looker Analytics Platform**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-looker-analytics-platform-sso"></a>Configuración del inicio de sesión único de Looker Analytics Platform

1. En otra ventana del explorador web, inicie sesión en el sitio web de Looker Analytics Platform como administrador.

1. Vaya a: **Administrador** -> **Autenticación** -> **SAML**

    ![captura de pantalla de la opción SAML](./media/looker-analytics-platform-tutorial/admin.png)

1. Pegue la información de **metadatos de federación** que copió de Azure Portal en el cuadro de texto de **metadatos de IDP** y haga clic en **Cargar**.

    ![captura de pantalla de la carga de metadatos](./media/looker-analytics-platform-tutorial/metadata.png)

1. Siga los pasos que se describen a continuación en la sección de **configuración de atributos del usuario**.

    ![captura de pantalla de la configuración de atributos del usuario](./media/looker-analytics-platform-tutorial/user-attribute-settings.png)

    a. Agregue el siguiente valor en el campo Email Attr (Atributo de correo electrónico): `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`

    b. Agregue el siguiente valor en el campo Fname Attr (Atributo de Fname): `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`

    c. Agregue el siguiente valor en el campo Lname Attr (Atributo de Lname): `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`

    d. En **Test User Authentication** (Probar autenticación del usuario), haga clic en **Test SAML Authentication** (Probar autenticación SAML). Si la página que se carga muestra el mensaje "Server Response Successfully Validated" (La respuesta del servidor se validó correctamente), quiere decir que ha configurado correctamente la instancia para la integración SAML.
    
    e. En **Save and Apply Settings** (Guardar y aplicar configuración), active la casilla **I have confirmed the configuration above and want to enable applying it globally** (He confirmado la configuración anterior y deseo aplicarla globalmente).

    f. Haga clic en el botón **Update Settings** (Actualizar configuración).

### <a name="create-looker-analytics-platform-test-user"></a>Creación de un usuario de prueba de Looker Analytics Platform

En esta sección, se creará una usuaria llamada Britta Simon en Looker Analytics Platform. Looker Analytics Platform admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si todavía no hay ningún usuario en Looker Analytics Platform, se creará uno después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la URL de inicio de sesión de Looker Analytics Platform, donde puede poner en marcha el flujo de inicio de sesión.  

* Acceda directamente a la URL de inicio de sesión de Looker Analytics Platform y ponga en marcha el flujo de inicio de sesión desde ahí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Debería iniciarse sesión automáticamente en la aplicación Looker Analytics Platform para la que configuró el inicio de sesión único. 

También puede usar el Panel de acceso de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Looker Analytics Platform en el panel de acceso, si se ha configurado el modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para poner en marcha el flujo de inicio de sesión y, si ha configurado el modo IDP, se debería iniciar sesión automáticamente en la aplicación Looker Analytics Platform para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Looker Analytics Platform, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
