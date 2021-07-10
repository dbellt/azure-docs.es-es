---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Pulse Secure Virtual Traffic Manager | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Pulse Secure Virtual Traffic Manager.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/18/2021
ms.author: jeedes
ms.openlocfilehash: 58f77c53798d10e667248f625df1517515ca1fa7
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110476495"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-pulse-secure-virtual-traffic-manager"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Pulse Secure Virtual Traffic Manager

En este tutorial, aprenderá a integrar Pulse Secure Virtual Traffic Manager con Azure Active Directory (Azure AD). Al integrar Pulse Secure Virtual Traffic Manager con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Pulse Secure Virtual Traffic Manager.
* Permitir que los usuarios inicien sesión automáticamente en Pulse Secure Virtual Traffic Manager con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Pulse Secure Virtual Traffic Manager.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Pulse Secure Virtual Traffic Manager admite el inicio de sesión único iniciado por **SP**.

## <a name="add-pulse-secure-virtual-traffic-manager-from-the-gallery"></a>Adición de Pulse Secure Virtual Traffic Manager desde la galería

Para configurar la integración de Pulse Secure Virtual Traffic Manager en Azure AD, es preciso agregar Pulse Secure Virtual Traffic Manager desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Pulse Secure Virtual Traffic Manager** en el cuadro de búsqueda.
1. Seleccione **Pulse Secure Virtual Traffic Manager** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-pulse-secure-virtual-traffic-manager"></a>Configuración y prueba del inicio de sesión único de Azure AD para Pulse Secure Virtual Traffic Manager

Configure y pruebe el inicio de sesión único de Azure AD con Pulse Secure Virtual Traffic Manager mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Pulse Secure Virtual Traffic Manager.

Para configurar y probar el inicio de sesión único de Azure AD con Pulse Secure Virtual Traffic Manager, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Pulse Secure Virtual Traffic Manager](#configure-pulse-secure-virtual-traffic-manager-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Pulse Secure Virtual Traffic Manager](#create-pulse-secure-virtual-traffic-manager-test-user)** , para tener un homólogo de B.Simon en Pulse Secure Virtual Traffic Manager que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Pulse Secure Virtual Traffic Manager**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<PUBLISHED VIRTUAL SERVER FQDN>/saml/consume`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<PUBLISHED VIRTUAL SERVER FQDN>/saml/metadata`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<PUBLISHED VIRTUAL SERVER FQDN>/saml/consume`

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de inicio de sesión, la dirección URL de respuesta y el identificador reales. Póngase en contacto con el [equipo de soporte técnico de Pulse Secure Virtual Traffic Manager](mailto:support@pulsesecure.net) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Pulse Secure Virtual Traffic Manager**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Pulse Secure Virtual Traffic Manager mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Pulse Secure Virtual Traffic Manager**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-pulse-secure-virtual-traffic-manager-sso"></a>Configuración del inicio de sesión único de Pulse Secure Virtual Traffic Manager

En esta sección se describe la configuración necesaria para habilitar la autenticación de SAML de Azure AD en Pulse Virtual Traffic Manager. Todos los cambios de configuración se realizan en Pulse Virtual Traffic Manager mediante la interfaz de usuario web de administración. 

### <a name="create-a-saml-trusted-identity-provider"></a>Creación de un proveedor de identidades de confianza de SAML

a. Vaya a la página **Pulse Virtual Traffic Manager Appliance Admin UI > Catalog > SAML > Trusted Identity Providers Catalog** (Interfaz de usuario del administrador del dispositivo Pulse Virtual Traffic Manager > Catálogo > SAML > Catálogo de proveedores de identidades de confianza) y haga clic en **Edit** (Editar).

![Página de catálogos de SAML](./media/pulse-secure-virtual-traffic-manager-tutorial/saml-catalogs.png)

b. Agregue los detalles para el nuevo proveedor de identidades de confianza de SAML, copie la información de la aplicación Azure AD Enterprise en la página Single sign-on settings (Configuración de inicio de sesión único) y, a continuación, haga clic en **Create New Trusted Identity Provider** (Crear un proveedor de identidades de confianza).

![Creación de un proveedor de identidades de confianza](./media/pulse-secure-virtual-traffic-manager-tutorial/identity-provider.png)

* En el cuadro de texto **Name** (Nombre), escriba un nombre para el proveedor de identidades de confianza. 

* En el cuadro de texto **Entity_id** (Id. de entidad), pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.  

* En el cuadro de texto **url**, pegue el valor de **Dirección URL de inicio de sesión** que copió de Azure Portal. 

* Abra el archivo del **certificado** que descargó de Azure Portal en el Bloc de notas, copie el contenido y, luego, péguelo en el cuadro de texto **Certificate** (Certificado).

c. Compruebe que el nuevo proveedor de identidades de SAML se ha creado correctamente. 

![Verificación del proveedor de identidades de confianza](./media/pulse-secure-virtual-traffic-manager-tutorial/verify-identity-provider.png)

### <a name="configure-the-virtual-server-to-use-azure-ad-authentication"></a>Configuración del servidor virtual para usar la autenticación de Azure AD

a. Vaya a la página **Pulse Virtual Traffic Manager Appliance Admin UI > Services > Virtual Servers** (Interfaz de usuario de administración del dispositivo de Pulse Virtual Traffic Manager > Servicios > Servidores virtuales) y haga clic en el botón **Edit** (Editar) junto al servidor virtual creado anteriormente.

![Edición de servidores virtuales](./media/pulse-secure-virtual-traffic-manager-tutorial/virtual-servers.png)

b. En la sección **Authentication** (Autenticación), haga clic en **Edit** (Editar). 

![Sección Authentication](./media/pulse-secure-virtual-traffic-manager-tutorial/authentication.png)

c. Configure las siguientes opciones de autenticación para el servidor virtual: 

1. Authentication (Autenticación):

    ![configuración de autenticación para el servidor virtual](./media/pulse-secure-virtual-traffic-manager-tutorial/authentication-1.png)

    a. En **auth!type**, seleccione **SAML Service Provider** (Proveedor de servicios de SAML). 

    b. Establezca **auth!verbose** en "Yes" (Sí) para solucionar los problemas de autenticación; de lo contrario, deje el valor predeterminado "No". 

2. Authentication Session Management (Administración de la sesión de autenticación):

    ![Sección Authentication Session Management](./media/pulse-secure-virtual-traffic-manager-tutorial/authentication-session.png)

    a. En **auth!session!cookie_name**, deje el valor predeterminado "VS_SamlSP_Auth". 

    b. En **auth!session!timeout**, deje el valor predeterminado "7200". 

    c. Establezca **auth!session!log_external_state** en "Yes" (Sí) para solucionar los problemas de autenticación; de lo contrario, deje el valor predeterminado "No". 

    d. Cambie el valor de **auth!session!cookie_attributes** a "HTTPOnly".

3. SAML Service Provider (Proveedor de servicios de SAML):

    ![Sección SAML Service Provider](./media/pulse-secure-virtual-traffic-manager-tutorial/service-provider.png)

    a. Establezca el valor del cuadro de texto **auth!saml!sp_entity_id** en la misma dirección URL usada como identificador de configuración de inicio de sesión único de Azure AD (Id. de entidad). Por ejemplo, `https://pulseweb.labb.info/saml/metadata`. 

    b. Establezca **auth!saml!sp_acs_url** en la misma dirección URL usada como la dirección URL de reproducción de la configuración de inicio de sesión único (URL del Servicio de consumidor de aserciones). Por ejemplo, `https://pulseweb.labb.info/saml/consume`. 

    c. En **auth!saml!idp**, seleccione el **proveedor de identidades confianza** que creó en el paso anterior. 

    d. En auth!saml!time_tolerance, deje el valor predeterminado "5" segundos. 

    e. En auth!saml!nameid_format, seleccione **unspecified** (sin especificar).

    f. Para aplicar los cambios, haga clic en **Update** (Actualizar) en la parte inferior de la página.
    
### <a name="create-pulse-secure-virtual-traffic-manager-test-user"></a>Creación de un usuario de prueba de Pulse Secure Virtual Traffic Manager

En esta sección, creará una usuaria llamada Britta Simon en Pulse Secure Virtual Traffic Manager. Trabaje con el [equipo de soporte técnico de Pulse Secure Virtual Traffic Manager](mailto:support@pulsesecure.net) para agregar los usuarios a la plataforma de Pulse Secure Virtual Traffic Manager. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Pulse Secure Virtual Traffic Manager, donde puede iniciar el flujo de inicio de sesión. 

* Vaya a la dirección URL de inicio de sesión de Pulse Secure Virtual Traffic Manager directamente e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Pulse Secure Virtual Traffic Manager en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de Pulse Secure Virtual Traffic Manager. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Pulse Secure Virtual Traffic Manager, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de su organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
