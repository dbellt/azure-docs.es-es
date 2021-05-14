---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Saba Cloud | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Saba Cloud.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/22/2021
ms.author: jeedes
ms.openlocfilehash: 493ec8ccc46ea5c2763f3a0159891fe9cbea142c
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108209254"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-saba-cloud"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Saba Cloud

En este tutorial aprenderá a integrar Saba Cloud con Azure Active Directory (Azure AD). Al integrar Saba Cloud con Azure AD, podrá hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Saba Cloud.
* Permitir a los usuarios iniciar sesión automáticamente en Saba Cloud con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Saba Cloud.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Saba Cloud admite el inicio de sesión único iniciado por **SP e IDP**.
* Saba Cloud admite el aprovisionamiento de usuarios **Just-In-Time**.
* Ahora se puede configurar la aplicación Saba Cloud con Azure AD para habilitar el inicio de sesión único. En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

## <a name="adding-saba-cloud-from-the-gallery"></a>Incorporación de Saba Cloud desde la galería

Para configurar la integración de Saba Cloud en Azure AD, necesita agregar Saba Cloud desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Saba Cloud** en el cuadro de búsqueda.
1. Seleccione **Saba Cloud** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-saba-cloud"></a>Configuración y prueba del inicio de sesión único de Azure AD para Saba Cloud

Configure y pruebe el inicio de sesión único de Azure AD con Saba Cloud mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario correspondiente de Saba Cloud.

Para configurar y probar el inicio de sesión único de Azure AD con Saba Cloud, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Saba Cloud](#configure-saba-cloud-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Saba Cloud](#create-saba-cloud-test-user)** : para tener un homólogo de B.Simon en Saba Cloud vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.
1. **[Prueba del inicio de sesión único para Saba Cloud (móvil)](#test-sso-for-saba-cloud-mobile)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Saba Cloud**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `<CUSTOMER_NAME>_SPLN_PRINCIPLE`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<SIGN-ON URL>/Saba/saml/SSO/alias/<ENTITY_ID>`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<CUSTOMER_NAME>.sabacloud.com`

    b. En el cuadro de texto **Estado de la retransmisión**, escriba una dirección URL con el siguiente patrón: `IDP_INIT---SAML_SSO_SITE=<SITE_ID> ` o, en caso de que SAML esté configurado para un micrositio, una dirección URL con el siguiente patrón: `IDP_INIT---SAML_SSO_SITE=<SITE_ID>---SAML_SSO_MICRO_SITE=<MicroSiteId>`.

    > [!NOTE]
    > Para más información sobre la configuración de Estado de la retransmisión, consulte [este](https://help.sabacloud.com/sabacloud/help-system/topics/help-system-idp-and-sp-initiated-sso-for-a-microsite.html) vínculo.

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con el identificador real, la dirección URL de respuesta, la dirección URL de inicio de sesión y el estado de la retransmisión. Póngase en contacto con el [equipo de soporte técnico de Saba Cloud](mailto:support@saba.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Set up Saba Cloud** (Configurar Saba Cloud), copie las direcciones URL que necesite.

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

En esta sección va a permitir que B.Simon acceda a Saba Cloud mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Saba Cloud**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-saba-cloud-sso"></a>Configuración del inicio de sesión único en Saba Cloud

1. Inicie sesión como administrador en el sitio de la empresa Saba Cloud.
1. Haga clic en el icono **Menu** (Menú) y en **Admin** (Administración), y seleccione la pestaña **System Admin** (Administrador del sistema).

    ![captura de pantalla del administrador del sistema](./media/saba-cloud-tutorial/system.png)

1. En **Configure System** (Configurar sistema), seleccione **SAML SSO Setup** (Configurar SSO de SAML) y haga clic en el botón **SETUP SAML SSO** (CONFIGURAR SSO DE SAML). 

    ![captura de pantalla de configuración](./media/saba-cloud-tutorial/configure.png)

1. En la ventana emergente, seleccione **Microsite** (Micrositio) en la lista desplegable y haga clic en **ADD AND CONFIGURE** (AGREGAR Y CONFIGURAR).

    ![captura de pantalla de agregar sitio/micrositio](./media/saba-cloud-tutorial/microsite.png)

1. En la sección **Configure IDP** (Configurar IDP), haga clic en **BROWSE** (EXAMINAR) para cargar el archivo **XML de metadatos de federación** que descargó de Azure Portal. Active la casilla **Site Specific IDP** (IDP específico del sitio) y haga clic en **IMPORT** (IMPORTAR).

    ![captura de pantalla para la importación de certificados](./media/saba-cloud-tutorial/certificate.png) 

1. En la sección **Configure SP** (Configurar SP), copie el valor de **Entity Alias** (Alias de la entidad) y péguelo en el cuadro **Identifier (Entity ID)** (Identificador [Id. de entidad]) de la sección **Configuración básica de SAML** en Azure Portal. Haga clic en **GENERATE** (GENERAR).

    ![captura de pantalla para la configuración de SP](./media/saba-cloud-tutorial/generate-metadata.png) 

1. En la sección **Configurar propiedades**, compruebe los campos rellenados y haga clic en **GUARDAR**. 

    ![captura de pantalla de Configurar propiedades](./media/saba-cloud-tutorial/configure-properties.png) 

### <a name="create-saba-cloud-test-user"></a>Creación de un usuario de prueba en Saba Cloud

En esta sección se crea un usuario llamado a B.Simon en Saba Cloud. Saba Cloud admite el aprovisionamiento de usuarios Just-In-Time, habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si el usuario aún no existe en Saba Cloud, se crea después de la autenticación.

> [!NOTE]
> Para habilitar el aprovisionamiento de usuarios Just-in-Time de SAML con Saba Cloud, consulte [esta](https://help.sabacloud.com/sabacloud/help-system/topics/help-system-user-provisioning-with-saml.html) documentación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Saba Cloud, donde podrá comenzar el flujo de inicio de sesión.  

* Acceda directamente a la dirección URL de inicio de sesión de Saba Cloud y ponga en marcha el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de Saba Cloud para la que configurara el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Saba Cloud de Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y, si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de Saba Cloud para la que se configurara el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

> [!NOTE]
> Si la dirección URL de inicio de sesión no se rellena en Azure AD, se trata a la aplicación como en modo iniciado por IDP; si se rellena la dirección URL de inicio de sesión, Azure AD siempre redirigirá al usuario a la aplicación de la nube Saba para el flujo iniciado por el proveedor de servicios.

## <a name="test-sso-for-saba-cloud-mobile"></a>Prueba del inicio de sesión único para Saba Cloud (móvil)

1. Abra la aplicación móvil Saba Cloud, asígnele un valor a **Site Name** (Nombre del sitio) en el cuadro de texto y haga clic en **Enter** (Entrar).

    ![Captura de pantalla de Site name (nombre del sitio).](./media/saba-cloud-tutorial/site-name.png)

1. Escriba la **dirección de correo electrónico** y haga clic en **Next** (Siguiente).

    ![Captura de pantalla de la dirección de correo electrónico.](./media/saba-cloud-tutorial/email-address.png)

1. Por último, después de iniciar sesión correctamente, la página principal de la aplicación aparecerá.

    ![Captura de pantalla de un inicio de sesión correcto.](./media/saba-cloud-tutorial/dashboard.png)

## <a name="next-steps"></a>Pasos siguientes

Una vez configurada Saba Cloud, podrá aplicar el control de sesión, que protege la información confidencial de la organización de la filtración y la infiltración en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).