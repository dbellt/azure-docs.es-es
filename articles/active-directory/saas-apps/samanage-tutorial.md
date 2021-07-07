---
title: 'Tutorial: Integración de Azure Active Directory con SolarWinds Service Desk (anteriormente Samanage) | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SolarWinds Service Desk.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/09/2021
ms.author: jeedes
ms.openlocfilehash: d10ff0f9ca4fde80557341c0b5d29b3f7c26c970
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111982843"
---
# <a name="tutorial-azure-active-directory-integration-with-solarwinds-service-desk-previously-samanage"></a>Tutorial: Integración de Azure Active Directory con SolarWinds Service Desk (anteriormente Samanage)

En este tutorial, aprenderá a integrar SolarWinds con Azure Active Directory (Azure AD). Al integrar SolarWinds con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a SolarWinds.
* Permitir que los usuarios inicien sesión automáticamente en SolarWinds con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en SolarWinds.

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SolarWinds admite SSO iniciado por **SP**.

## <a name="add-solarwinds-from-the-gallery"></a>Incorporación de SolarWinds desde la galería

Para configurar la integración de SolarWinds en Azure AD, es preciso agregar SolarWinds desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **SolarWinds** en el cuadro de búsqueda.
1. Seleccione **SolarWinds** en el panel de resultados y, luego, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-solarwinds"></a>Configuración y prueba del inicio de sesión único de Azure AD para SolarWinds

Configure y pruebe el inicio de sesión único de Azure AD con SolarWinds con un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de SolarWinds.

Para configurar y probar el inicio de sesión único de Azure AD con SolarWinds, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en SolarWinds](#configure-solarwinds-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en SolarWinds](#create-solarwinds-test-user)** : para tener un homólogo de B. Simon en SolarWinds vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **SolarWinds**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<Company Name>.samanage.com/saml_login/<Company Name>`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<Company Name>.samanage.com`

    > [!NOTE] 
    > Estos valores no son reales. Actualice estos valores con la dirección URL de inicio de sesión y el identificador reales, que se explican más adelante en el tutorial. Para más información, póngase en contacto con [equipo de soporte al cliente de Samanage](https://www.samanage.com/support). También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up SolarWinds** (Configurar SolarWinds), copie las direcciones URL que necesite.

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

En esta sección, va a conceder a B. Simon acceso a SolarWinds para que pueda usar el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **SolarWinds**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

<a name="configure-solarwinds-single-sign-on"></a>

## <a name="configure-solarwinds-sso"></a>Configuración del inicio de sesión único en SolarWinds

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la empresa SolarWinds.

2. Haga clic en **Dashboard** (Panel) y seleccione **Setup** (Configuración) en el panel de navegación izquierdo.
   
    ![Panel](./media/samanage-tutorial/tutorial-samanage-1.png "Panel")

3. Haga clic en **Inicio de sesión único**.
   
    ![Inicio de sesión único](./media/samanage-tutorial/tutorial-samanage-2.png "Inicio de sesión único")

4. Navegue hasta la sección **Login using SAML** (Iniciar sesión con SAML), realice los pasos siguientes:
   
    ![Login using SAML (Iniciar sesión con SAML)](./media/samanage-tutorial/tutorial-samanage-3.png "Login using SAML")
 
    a. Haga clic en **Enable Single Sign-On with SAML**(Habilitar inicio de sesión único con SAML).  
 
    b. En el cuadro de texto **Identity Provider URL** (Dirección URL del proveedor de identidades), escriba el valor como se muestra a continuación: `https://YourAccountName.samanage.com`.
 
    c. Confirme que **Login URL** (URL de inicio de sesión) coincide con el valor de **Dirección URL de inicio de sesión** de la sección **Configuración básica de SAML** de Azure Portal.
 
    d. En el cuadro de texto **Logout URL** (URL de cierre de sesión), pegue el valor de **URL de cierre de sesión** que copió de Azure Portal.
 
    e. En el cuadro de texto **SAML Issuer** (Emisor de SAML) escriba el URI del identificador de la aplicación establecido en el proveedor de identidades.
 
    f. En el Bloc de notas, abra el certificado codificado en base 64 descargado de Azure Portal, copie su contenido en el Portapapeles y péguelo en el cuadro de texto **Paste your Identity Provider x.509 Certificate below** (Pegue a continuación el certificado x.509 del proveedor de identidades).
 
    g. Haga clic en **Create users if they do not exist in SolarWinds** (Crear usuarios si no existen en SolarWinds).
 
    h. Haga clic en **Update**(Actualizar).

### <a name="create-solarwinds-test-user"></a>Creación de un usuario de prueba de SolarWinds

Para permitir que los usuarios de Azure AD inicien sesión en SolarWinds, debe aprovisionarlos en SolarWinds.  
En el caso de SolarWinds, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión como administrador en el sitio de la empresa SolarWinds.

2. Haga clic en **Dashboard** (Panel) y seleccione **Setup** (Configuración) en el panel de navegación izquierdo.
   
    ![Configuración](./media/samanage-tutorial/tutorial-samanage-1.png "Configurar")

3. Haga clic en la pestaña **Usuarios**.
   
    ![Usuarios](./media/samanage-tutorial/tutorial-samanage-6.png "Usuarios")

4. Haga clic en **Nuevo usuario**.
   
    ![New User](./media/samanage-tutorial/tutorial-samanage-7.png "Nuevo usuario") (Usuario nuevo)

5. Escriba el **nombre** y la **dirección de correo electrónico** de la cuenta de Azure Active Directory que desea aprovisionar y haga clic en **Create user** (Crear usuario).
   
    ![Crear usuario](./media/samanage-tutorial/tutorial-samanage-8.png "Crear usuario")
   
   >[!NOTE]
   >El titular de la cuenta de Azure Active Directory recibirá un mensaje de correo y seguirá un vínculo para confirmar su cuenta antes de que se active. Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de SolarWinds para aprovisionar las cuentas de usuario de Azure Active Directory.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la URL de inicio de sesión de SolarWinds, desde donde puede poner en marcha el flujo de inicio de sesión. 

* Vaya directamente a la URL de inicio de sesión de SolarWinds y comience el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de SolarWinds en Aplicaciones, se le redirigirá a la URL de inicio de sesión de la aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado SolarWinds, puede aplicar el control de sesión, que protege contra la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
