---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SolarWinds Orion | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SolarWinds Orion.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/01/2021
ms.author: jeedes
ms.openlocfilehash: b503697972bfe7fbd8fc18b77d6e0206dc055b67
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108206140"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-solarwinds-orion"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SolarWinds Orion

En este tutorial, aprenderá a integrar SolarWinds Orion con Azure Active Directory (Azure AD). Al integrar SolarWinds Orion con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a SolarWinds Orion.
* Permitir que los usuarios inicien sesión automáticamente en SolarWinds Orion con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en SolarWinds Orion.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SolarWinds Orion admite el inicio de sesión único iniciado por **SP y IDP**.

## <a name="add-solarwinds-orion-from-the-gallery"></a>Adición de SolarWinds Orion desde la galería

Para configurar la integración de SolarWinds Orion en Azure AD, será preciso que agregue SolarWinds Orion de la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **SolarWinds Orion** en el cuadro de búsqueda.
1. Seleccione **SolarWinds Orion** en el panel de resultados y, luego, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-solarwinds-orion"></a>Configuración y prueba del inicio de sesión único de Azure AD para SolarWinds Orion

Configure y pruebe el inicio de sesión único de Azure AD con SolarWinds Orion con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de SolarWinds Orion.

Para configurar y probar el inicio de sesión único de Azure AD con SolarWinds Orion, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en SolarWinds Orion](#configure-solarwinds-orion-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en SolarWinds Orion](#create-solarwinds-orion-test-user)** : para tener un homólogo de B.Simon en SolarWinds Orion vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **SolarWinds Orion**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<ORION-HOSTNAME-OR-EXTERNAL-URL>`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<ORION-HOSTNAME-OR-EXTERNAL-URL>/Orion/SAMLLogin.aspx`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<ORION-HOSTNAME-OR-EXTERNAL-URL>/Orion/Login.aspx`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de atención al cliente de SolarWinds Orion](mailto:technicalsupport@solarwinds.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación SolarWinds Orion espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación SolarWinds Orion espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.
    
    | Nombre |  Atributo de origen|
    | ----------- | --------- |
    | Nombre | user.givenname |
    | Apellidos | user.surname |
    | Email |user.mail |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar SolarWinds Orion**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Evergreen mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Evergreen**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-solarwinds-orion-sso"></a>Configuración del inicio de sesión único en SolarWinds Orion

1. Inicie sesión en SolarWinds Orion y vaya a **Settings** -> **All Settings** (Configuración > Toda la configuración).

    ![Captura de pantalla que muestra la opción de toda la configuración seleccionada en la configuración.](./media/solarwinds-orion-tutorial/settings.png)

1. En la sección **USER ACCOUNTS** (CUENTAS DE USUARIO), seleccione **SAML Configuration** (Configuración de SAML).

    ![Captura de pantalla que muestra la opción de configuración de SAML seleccionada en las cuentas de usuario.](./media/solarwinds-orion-tutorial/configure-user-accounts.png)

1. Haga clic en **ADD IDENTITY PROVIDER** (AGREGAR PROVEEDOR DE IDENTIDADES).

    ![Captura de pantalla que muestra la configuración de SAML, donde puede seleccionar la opción para agregar un proveedor de identidades.](./media/solarwinds-orion-tutorial/configure-add-identity-provider.png)

1. Realice los pasos siguientes en la página **Add Identity Provider** (Agregar proveedor de identidades):

    ![Captura de pantalla que muestra la página para agregar un proveedor de identidades, donde puede especificar los valores descritos.](./media/solarwinds-orion-tutorial/configure-solarwinds.png)

    a. Vaya a la pestaña **Configure** (Configurar).

    b. En el cuadro **Identity Provider Name** (Nombre del proveedor de identidad), proporcione un nombre válido como `My SSO service`.

    c. En el cuadro de texto **SSO Target URL** (Dirección URL de destino de SSO), pegue el valor de **Login URL** (URL de inicio de sesión) que ha copiado de Azure Portal.

    d.  En el cuadro de texto **Issuer URL** (Dirección URL del emisor), pegue el valor del **Identificador de Azure AD** que ha copiado de Azure Portal.

    e. En Azure Portal, abra el **certificado (Base64)** en el Bloc de notas y pegue el contenido en el cuadro de texto **X.509 Signing Certificate** (Certificado de firma de X.509).

    f. Haga clic en **Guardar**.

### <a name="create-solarwinds-orion-test-user"></a>Creación de un usuario de prueba en SolarWinds Orion

1. Inicie sesión en el sitio web de SolarWinds Orion y vaya a **Settings** -> **All Settings** (Configuración > Toda la configuración).

    ![Captura de pantalla que muestra la opción de toda la configuración seleccionada en la configuración.](./media/solarwinds-orion-tutorial/settings.png)

1. En la sección **USER ACCOUNTS** (CUENTAS DE USUARIO), seleccione **Manage Accounts** (Administrar cuentas).

    ![Captura de pantalla que muestra la opción de configuración de SAML seleccionada.](./media/solarwinds-orion-tutorial/user-accounts.png)

1. En la pestaña **INDIVIDUAL ACCOUNTS** (CUENTAS INDIVIDUALES), haga clic en **ADD NEW ACCOUNT** (AGREGAR NUEVA CUENTA).

    ![Captura de pantalla que muestra la opción para agregar una nueva cuenta seleccionada en la administración de cuentas.](./media/solarwinds-orion-tutorial/create-user.png)

1. Seleccione el tipo de cuenta que necesita para crear usuarios individuales o grupos de SAML.

    ![Captura de pantalla que muestra la opción para agregar una nueva cuenta, donde puede seleccionar el tipo de cuenta.](./media/solarwinds-orion-tutorial/create-user-new-account.png)

1.  En el cuadro de texto **NAME ID** (ID. DE NOMBRE), escriba el nombre, que debe coincidir con el mismo nombre de usuario o nombre de grupo que en Azure AD.

1.  Haga clic en **Next** (Siguiente) y, luego, envíe la página.

    ![Captura de pantalla que muestra la opción para agregar una nueva cuenta, en la que puede escribir el nombre I D de Azure A D.](./media/solarwinds-orion-tutorial/create-user-name-id.png)

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la URL de inicio de sesión de SolarWinds Orion, desde donde puede poner en marcha el flujo de inicio de sesión.  

* Vaya directamente a la URL de inicio de sesión de SolarWinds Orion y comience el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de SolarWinds Orion para la que configurara el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de SolarWinds Orion en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de SolarWinds Orion para la que configurara el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado SolarWinds Orion, puede aplicar el control de sesión, que protege contra la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).