---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con desknets NEO | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y desknets NEO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/08/2021
ms.author: jeedes
ms.openlocfilehash: ef3e024e1b704cc9f8711d9ca78ee916acf72680
ms.sourcegitcommit: d3bcd46f71f578ca2fd8ed94c3cdabe1c1e0302d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107580707"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-desknets-neo"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con desknets NEO

En este tutorial aprenderá a integrar desknets NEO con Azure Active Directory (Azure AD). Al integrar desknets NEO con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a desknets NEO.
* Permitir que los usuarios puedan iniciar sesión automáticamente en desknets NEO con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en desknets NEO.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* desknets NEO admite SSO iniciado por **SP.**

## <a name="adding-desknets-neo-from-the-gallery"></a>Incorporación de desknets NEO desde la galería

Para configurar la integración de desknets NEO en Azure AD, deberá agregar desknets NEO desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **desknets NEO** en el cuadro de búsqueda.
1. Seleccione **desknets NEO** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-desknets-neo"></a>Configuración y prueba del inicio de sesión único de Azure AD para desknets NEO

Configure y pruebe el inicio de sesión único de Azure AD con desknets NEO, y utilice para ello un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de desknets NEO.

Para configurar y probar el inicio de sesión único de Azure AD con desknets NEO, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en desknets NEO](#configure-desknets-neo-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de desknets NEO](#create-desknets-neo-test-user)** , para tener un homólogo de B. Simon en desknets NEO que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicaciones **desknets NEO**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<CUSTOMER_NAME>.dn-cloud.com`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<CUSTOMER_NAME>.dn-cloud.com/cgi-bin/dneo/zsaml.cgi`

    c. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<CUSTOMER_NAME>.dn-cloud.com/cgi-bin/dneo/dneo.cgi`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y las direcciones URL de inicio de sesión y de respuesta reales. Para obtener estos valores, póngase en contacto con el [equipo de soporte técnico al cliente de desknets NEO](mailto:cloudsupport@desknets.com). También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configuración de desknets NEO**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a desknets NEO mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **desknets NEO**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-desknets-neo-sso"></a>Configuración del inicio de sesión único de desknets NEO

1. Inicie sesión en su sitio de la empresa de desknets NEO como administrador.

1. En el menú, haga clic en el icono **SAML authentication link settings** (Configuración del vínculo de autenticación de SAML).

    ![Captura de pantalla de SAML authentication link settings (Configuración del vínculo de autenticación de SAML).](./media/desknets-neo-tutorial/saml-authentication-icon.png)

1. En **Common settings** (Configuración común), haga clic en **use** (Usar) en SAML Authentication Collaboration (Colaboración de autenticación SAML).

    ![Captura de pantalla del uso de la autenticación SAML.](./media/desknets-neo-tutorial/saml-authentication-use.png)

1. Realice los pasos siguientes en la sección **SAML authentication link settings** (Configuración del vínculo de autenticación SAML).

    ![Captura de pantalla de la sección SAML authentication link settings (Configuración del vínculo de autenticación de SAML).](./media/desknets-neo-tutorial/saml-authentication.png)

    a. En el cuadro de texto **Access URL** (URL de acceso), pegue el valor de **Dirección URL de inicio de sesión** que copió de Azure Portal.

    b. En el cuadro de texto **SP Entity ID** (Id. de entidad de proveedor de servicios), pegue el valor de **Identificador** que ha copiado de Azure Portal.

    c. Haga clic en **Elegir archivo** para cargar el archivo de **certificado (Base64)** descargado de Azure Portal en el cuadro de texto **Certificado x.509.**

    d. Haga clic en **change** (cambiar).

### <a name="create-desknets-neo-test-user"></a>Creación de un usuario de prueba en desknets NEO

1. Inicie sesión en su sitio de la empresa de desknets NEO como administrador.

1. En el **menú**, haga clic en el icono **Configuración de administrador**.

    ![Captura de pantalla de Configuración de administrador.](./media/desknets-neo-tutorial/administrator-settings.png)

1. Haga clic en el icono de **configuración** y seleccione **Administración de usuarios** en **Configuración personalizada**.

    ![Captura de pantalla de la configuración de Administración de usuarios.](./media/desknets-neo-tutorial/user-management.png)

1. Haga clic en **Create user information** (Crear información del usuario).

    ![Captura de pantalla del botón Información del usuario.](./media/desknets-neo-tutorial/create-new-user.png)

1. Rellene los campos obligatorios en la página siguiente y haga clic en **creation** (creación).

    ![Captura de pantalla de la sección Creación de usuario.](./media/desknets-neo-tutorial/create-new-user-2.png)

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de desknets NEO, donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de desknets NEO e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de desknets NEO en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de desknets NEO. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).


## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado desknets NEO, puede aplicar el control de sesión, que protege a su organización, en tiempo real, frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).


