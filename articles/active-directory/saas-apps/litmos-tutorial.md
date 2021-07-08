---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Litmos | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Litmos.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/12/2021
ms.author: jeedes
ms.openlocfilehash: 535e1b103e44074021f23ae9527f849082d24a7f
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110467842"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-litmos"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Litmos

En este tutorial, obtendrá información sobre cómo integrar Litmos con Azure Active Directory (Azure AD). Al integrar Litmos con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Litmos.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Litmos con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Litmos.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Litmos admite el inicio de sesión único iniciado por **IDP**.
* Litmos admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="add-litmos-from-the-gallery"></a>Incorporación de Litmos desde la galería

Para configurar la integración de Litmos en Azure AD, deberá agregar Litmos desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Litmos** en el cuadro de búsqueda.
1. Seleccione **Litmos** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-litmos"></a>Configuración y prueba del inicio de sesión único de Azure AD para Litmos

Configure y pruebe el inicio de sesión único de Azure AD con Litmos mediante una usuaria de prueba llamada **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Litmos.

Para configurar y probar el inicio de sesión único de Azure AD con Litmos, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Litmos](#configure-litmos-sso)**, para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Litmos](#create-litmos-test-user)**, para tener un homólogo de B.Simon en Litmos que esté vinculado a la representación de ella en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Litmos**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<companyname>.litmos.com/account/Login`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.litmos.com/integration/samllogin`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador y Dirección URL de respuesta, que se explican más adelante en el tutorial, o bien póngase en contacto con el [equipo de soporte técnico al cliente de Litmos](https://www.litmos.com/contact-us) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Litmos**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Litmos mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Litmos**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-litmos-sso"></a>Configuración del inicio de sesión único en Litmos

1. En otra ventana del explorador, inicie sesión en su sitio de la compañía de Litmos como administrador.

2. En la barra de navegación del lado izquierdo, haga clic en el icono de **cuentas**.

    ![Sección Cuentas en la aplicación](./media/litmos-tutorial/account.png)

3. Haga clic en la pestaña **Integraciones** .

    ![Pestaña Integración](./media/litmos-tutorial/integrate.png)

4. En la pestaña **Integraciones**, desplácese hacia abajo hasta **3rd Party Integrations (Integraciones de terceros)** y, después, haga clic en la pestaña **SAML 2.0**.

    ![Sección SAML 2.0](./media/litmos-tutorial/third-party.png)

5. Copie el valor situado bajo **El punto de conexión de SAML para Litmos es:** y péguelo en el cuadro de texto **Dirección URL de respuesta** en la sección **Dominio y direcciones URL de Litmos** en Azure Portal.

    ![Punto de conexión de SAML](./media/litmos-tutorial/certificate.png)

6. En la aplicación **Litmos** , realice los pasos siguientes:

    ![Aplicación Litmos](./media/litmos-tutorial/application.png)

    a. Haga clic en **Enable SAML**(Habilitar SAML).

    b. Abra el certificado codificado en base 64 en el Bloc de notas, copie el contenido del mismo en el Portapapeles y, a continuación, péguelo en el cuadro de texto **Certificado X.509 de SAML** .

    c. Haga clic en **Guardar cambios**.

### <a name="create-litmos-test-user"></a>Creación de un usuario de prueba en Litmos

En esta sección se crea un usuario llamado B.Simon en Litmos. Litmos admite el aprovisionamiento de usuarios Just-In-Time, habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si todavía no existe ningún usuario en Litmos, se creará uno después de la autenticación.

**Para crear un usuario llamado Britta Simon en Litmos, realice los pasos siguientes:**

1. En otra ventana del explorador, inicie sesión en su sitio de la compañía de Litmos como administrador.

2. En la barra de navegación del lado izquierdo, haga clic en el icono de **cuentas**.

    ![Sección Cuentas en la aplicación](./media/litmos-tutorial/account.png)

3. Haga clic en la pestaña **Integraciones** .

    ![Pestaña Integraciones](./media/litmos-tutorial/integrate.png)

4. En la pestaña **Integraciones**, desplácese hacia abajo hasta **3rd Party Integrations (Integraciones de terceros)** y, después, haga clic en la pestaña **SAML 2.0**.

    ![SAML 2.0](./media/litmos-tutorial/third-party.png)

5. Seleccione **Autogenerate Users**(Generar usuarios automáticamente)
  
    ![Generar usuarios automáticamente](./media/litmos-tutorial/users.png)

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

* Haga clic en Probar esta aplicación en Azure Portal; debería iniciar sesión automáticamente en la instancia de Litmos para la que configurara el inicio de sesión único.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Litmos de Aplicaciones, debería iniciar sesión automáticamente en la versión de Litmos para la que configurara el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurada Litmos, podrá aplicar el control de sesión para proteger la información confidencial de su organización de la filtración y la infiltración en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
