---
title: 'Tutorial: integración de Azure Active Directory con el software Cezanne HR | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y el software Cezanne HR.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/11/2021
ms.author: jeedes
ms.openlocfilehash: 6318bfc659a307043f7339875644df33ff344a66
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110064118"
---
# <a name="tutorial-azure-active-directory-integration-with-cezanne-hr-software"></a>Tutorial: Integración de Azure Active Directory con el software Cezanne HR

En este tutorial, obtendrá información sobre cómo integrar Cezanne HR Software con Azure Active Directory (Azure AD). Al integrar Cezanne HR Software con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a Cezanne HR Software.
* Permitir que los usuarios inicien sesión automáticamente en Cezanne HR Software con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Cezanne HR Software.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Cezanne HR Software admite el inicio de sesión único iniciado por **SP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-cezanne-hr-software-from-the-gallery"></a>Adición de Cezanne HR Software desde la galería

Para configurar la integración del software Cezanne HR en Azure AD, deberá agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Cezanne HR Software** en el cuadro de búsqueda.
1. Seleccione **Cezanne HR Software** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-cezanne-hr-software"></a>Configuración y prueba de inicio de sesión único de Azure AD para Cezanne HR Software

Configure y pruebe el inicio de sesión único de Azure AD con Cezanne HR Software con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Cezanne HR Software.

Para configurar el inicio de sesión único de Azure AD con Cezanne HR Software, realice los pasos siguientes:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Cezanne HR Software](#configure-cezanne-hr-software-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Cezanne HR Software](#create-cezanne-hr-software-test-user)** , para tener un homólogo de B.Simon en Cezanne HR Software que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Cezanne HR Software**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://w3.cezanneondemand.com/CezanneOnDemand/-/<tenantidentifier>`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba la dirección URL: `https://w3.cezanneondemand.com/CezanneOnDemand/`.

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://w3.cezanneondemand.com:443/cezanneondemand/-/<tenantidentifier>/Saml/samlp`.
    
    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de URL de respuesta y URL de inicio de sesión. Para obtener estos valores, póngase en contacto con el [equipo de soporte técnico de Cezanne HR Software Client](https://cezannehr.com/services/support/).

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Cezanne HR Software** (Configurar Cezanne HR Software), copie las direcciones URL que necesite.

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

En esta sección, va a permitir que B.Simon use el inicio de sesión único de Azure concediéndole acceso a Cezanne HR Software.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Software Cezanne HR**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-cezanne-hr-software-sso"></a>Configuración del inicio de sesión único de Cezanne HR Software

1. En otra ventana del explorador web, inicie sesión en el inquilino del software Cezanne HR como administrador.

2. En el menú lateral, haga clic en **Administration** (Administración). A continuación, vaya a **Security Settings** (Configuración de seguridad) y haga clic en **Single Sign-On** (Inicio de sesión único).

    ![Captura de pantalla que muestra el inquilino del software Cezanne HR con Security Settings (Configuración de seguridad) y Single Sign-On Configuration (Configuración de inicio de sesión único) seleccionados.](./media/cezannehrsoftware-tutorial/settings.png)

3. En el panel **Allow users to log in using the following Single Sign-On (SSO) Service** (Permitir a los usuarios iniciar sesión mediante el siguiente servicio de inicio de sesión único), marque la casilla **SAML 2.0** y seleccione la opción **Advanced Configuration** (Configuración avanzada).

    ![Captura de pantalla que muestra el panel Allow users (Permitir usuarios) con SAML 2.0 y Advanced Configuration (Configuración avanzada) seleccionados.](./media/cezannehrsoftware-tutorial/configuration.png)

4. Haga clic en el botón **Add New** (Agregar nuevo).

    ![Captura de pantalla que muestra el botón Add New (Agregar nuevo).](./media/cezannehrsoftware-tutorial/new-button.png)

5. Escriba los siguientes campos en la sección **SAML 2.0 IDENTITY PROVIDERS** (Proveedores de identidades de SAML 2.0) y haga clic en **OK** (Aceptar).

    ![Captura de pantalla que muestra un panel para escribir los valores que se describen en este paso.](./media/cezannehrsoftware-tutorial/identity-provider.png)

    a. **Display Name** (Nombre para mostrar): escriba el nombre del proveedor de identidades como el nombre para mostrar.

    b. **Entity Identifier** (Identificador de entidad): pegue el valor del identificador de Azure AD que ha copiado en Azure Portal en este cuadro de texto.

    c. **SAML Binding** (Enlace de SAML): cambie el enlace de SAML a "POST".

    d. **Security Token Service Endpoint** (Punto de conexión del servicio de token de seguridad): pegue el valor de la dirección URL de inicio de sesión que ha copiado de Azure Portal en este cuadro de texto.

    e. **User ID Attribute Name** (Nombre de atributo de id. de usuario): escriba "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress" en este cuadro de texto.

    f. **Public Key Certificate** (Certificado de clave pública): haga clic en el icono de carga para cargar el certificado que ha descargado de Azure Portal.

6. Haga clic en Aceptar.

7. Haga clic en el botón Guardar . 

    ![Captura de pantalla que muestra el botón Save (Guardar) para Single Sign-On Configuration (configuración de inicio de sesión único).](./media/cezannehrsoftware-tutorial/save-button.png)

### <a name="create-cezanne-hr-software-test-user"></a>Creación de un usuario de prueba de Cezanne HR Software

Para permitir que los usuarios de Azure AD inicien sesión en el software Cezanne HR, tienen que aprovisionarse en él. En el caso del software Cezanne HR, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en su sitio de la compañía del software Cezanne HR como administrador.

2. En el menú lateral, haga clic en **Administration** (Administración). A continuación, vaya a **Users** (Usuarios) y haga clic en **Add New User** (Agregar nuevo usuario).

    ![Captura de pantalla que muestra el inquilino del software Cezanne HR con Manage Users (Administrar usuarios) y Add New User (Agregar nuevo usuario) seleccionados.](./media/cezannehrsoftware-tutorial/manage-users.png "Nuevo usuario")

3. En la sección **PERSON DETAILS** (Datos de la persona), siga estos pasos:

    ![Captura de pantalla que muestra la sección PERSON DETAILS (DATOS DE LA PERSONA) para escribir los valores que se describen en este paso.](./media/cezannehrsoftware-tutorial/details.png "Nuevo usuario")

    a. En **Internal User** (Usuario interno), seleccione OFF (Desactivado).

    b. Escriba el nombre.   

    c. Escriba los apellidos.

    d. Escriba la dirección de correo electrónico.

4. En la sección **Account Information** (Información de la cuenta), siga estos pasos:

    ![Captura de pantalla que muestra ACCOUNT INFORMATION (INFORMACIÓN DE CUENTA) para escribir los valores que se describen en este paso.](./media/cezannehrsoftware-tutorial/account.png "Nuevo usuario")

    a. En el cuadro de texto **Nombre de usuario**, escriba el correo electrónico de un usuario, por ejemplo, Brittasimon@contoso.com.

    b. En el cuadro de texto **Contraseña**, escriba la contraseña del usuario.

    c. Seleccione **HR Professional** (Profesional de RR.HH.) como **Security Role** (Rol de seguridad).

    d. Haga clic en **OK**.
    ![Captura de pantalla que muestra el botón Aceptar.](https://user-images.githubusercontent.com/80324891/115694644-f6eb5700-a358-11eb-9b23-a87a24921052.png)

5. Navegue a la pestaña **Single Sign-On** (Inicio de sesión único) y seleccione **Add New** (Agregar nuevo) en el área **SAML 2.0 Identifiers** (Identificadores SAML 2.0).

    ![Captura de pantalla que muestra la pestaña Single Sign-On (Inicio de sesión único) para seleccionar Add New (Agregar nuevo).](./media/cezannehrsoftware-tutorial/single-sign-on.png "Usuario")

6. Elija el proveedor de identidades en **Identity Provider** (Proveedor de identidades) y, en el cuadro de texto **User Identifier** (Identificador de usuario), escriba la dirección de correo electrónico del usuario.

    ![Captura de pantalla que muestra SAML 2.0 Identifiers (Identificadores de SAML 2.0) ara seleccionar el proveedor de identidades y el identificador de usuario.](./media/cezannehrsoftware-tutorial/user-identifier.png "Usuario")

7. Haga clic en el botón **Guardar** .

    ![Captura de pantalla que muestra el botón Save (Guardar) para User Settings (Configuración de usuario).](./media/cezannehrsoftware-tutorial/save.png "Usuario")

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Cezanne HR Software, donde puede comenzar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Cezanne HR Software y comience el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Cezanne HR Software en Mis aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de Cezanne HR Software. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Cezanne HR Software, puede aplicar el control de sesión que protege la exfiltración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
