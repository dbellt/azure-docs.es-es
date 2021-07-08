---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Segment | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Segment.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/28/2021
ms.author: jeedes
ms.openlocfilehash: ab3afec6b849288a87ba8aa194d5b180a49dc184
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111557687"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-segment"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Segment

En este tutorial aprenderá a integrar Segment con Azure Active Directory (Azure AD). Al integrar Segment con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Segment.
* Permitir que los usuarios inicien sesión automáticamente en Segment con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Segment.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Segment admite el inicio de sesión único iniciado por **SP e IDP**.
* Segment admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="add-segment-from-the-gallery"></a>Adición de Segment desde la galería

Para configurar la integración de Segment en Azure AD, deberá agregar Segment desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Segment** en el cuadro de búsqueda.
1. Seleccione **Segment** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-segment"></a>Configuración y prueba del inicio de sesión único de Azure AD para Segment

Configure y pruebe el inicio de sesión único de Azure AD con Segment mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Segment.

Para configurar y probar el inicio de sesión único de Azure AD con Segment, haga lo siguiente:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Segment](#configure-segment-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Segment](#create-segment-test-user)** : para tener un homólogo de B.Simon en Segment vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Segment**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice los siguientes pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `urn:auth0:segment-prod:samlp-<CUSTOMER_VALUE>`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://segment-prod.auth0.com/login/callback?connection=<CUSTOMER_VALUE>`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL: `https://app.segment.com`

    > [!NOTE]
    > Estos valores son marcadores de posición. Debe usar los valores reales de identificador, dirección URL de respuesta o dirección URL de inicio de sesión. Los pasos para obtener estos valores se describen más adelante.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Segment**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Segment mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Segment**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-segment-sso"></a>Configuración del inicio de sesión único de Segment

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la compañía en Segment.

1. Haga clic en el **icono de configuración** y desplácese hacia abajo hasta **AUTHENTICATION** (Autenticación) y haga clic en **Connections** (Conexiones).

    ![Captura de pantalla que muestra el icono Settings "Configuración" seleccionado y la opción "Connections" (Conexiones) seleccionada en el menú "Authentication" (Autenticación).](./media/segment-tutorial/connections.PNG)

1. Haga clic en **Add new Connection** (Agregar nueva conexión).

    ![Captura de pantalla que muestra la sección "Connections" (Conexiones) con el botón "Add new Connection" (Agregar nueva conexión) seleccionado.](./media/segment-tutorial/new-connections.PNG)

1. Seleccione **SAML 2.0** como la conexión que va a configurar y haga clic en el botón **Select Connection** (Seleccionar conexión).

    ![Captura de pantalla que muestra la sección "Choose a Connection" (Elegir una conexión) con "S A M L 2.0" y el botón "Select Connection" (Seleccionar conexión) seleccionados.](./media/segment-tutorial/select-connections.PNG)

1. En la página siguiente, realice los pasos siguientes:

    ![Captura de pantalla que muestra la página "Configure Identity Provider" (Configurar proveedor de identidades) con los cuadros de texto "Single Sign-On U R L" (Dirección U R L de inicio de sesión) y "Audience U R L" (Dirección URL de audiencia) resaltados, y el botón "Next" (Siguiente) seleccionado.](./media/segment-tutorial/configure.PNG)

    a. Copie el valor de **Single Sign-On URL** (Dirección URL de inicio de sesión único) y péguelo en el cuadro **Dirección URL de respuesta** del cuadro de diálogo **Configuración básica de SAML** de Azure Portal.

    b. Copie el valor de ****Audience URL**** (Dirección URL de audiencia) y péguelo en el cuadro **URL del identificador** del cuadro de diálogo **Configuración básica de SAML** de Azure Portal.

    c. Haga clic en **Siguiente**.

    ![Configuración de Segment](./media/segment-tutorial/certificate.PNG)

1. En el cuadro **SAML 2.0 Endpoint URL** (URL de punto de conexión de SAML 2.0), pegue el valor de **Dirección URL de inicio de sesión** que copió de Azure Portal.

1. Abra en el Bloc de notas el archivo del **certificado (Base 64)** que descargó de Azure Portal, copie el contenido y, luego, péguelo en el cuadro de texto **Public Certificate** (Certificado público).

1. Haga clic en **Configure Connection** (Configurar conexión).

### <a name="create-segment-test-user"></a>Creación de un usuario de prueba de Segment

En esta sección, se crea un usuario llamado B.Simon en Segment. Segment admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si el usuario no existe en Segment, se crea uno después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Segment, donde puede poner en marcha el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de Segment y ponga en marcha el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de Segment para la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Segment en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión y, si se ha configurado en modo IDP, debería iniciar sesión automáticamente en la instancia de Segment para la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que ha configurado Segment, puede aplicar el control de sesión, que protege la información confidencial de la organización en tiempo real de posibles filtraciones e infiltraciones. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
