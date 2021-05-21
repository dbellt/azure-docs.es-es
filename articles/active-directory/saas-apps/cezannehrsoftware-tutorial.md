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
ms.date: 02/12/2019
ms.author: jeedes
ms.openlocfilehash: faecba93fbbc9e25ea85b644e0254f07747611dc
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109786166"
---
# <a name="tutorial-azure-active-directory-integration-with-cezanne-hr-software"></a>Tutorial: Integración de Azure Active Directory con el software Cezanne HR

En este tutorial, obtendrá información sobre cómo integrar Cezanne HR Software con Azure Active Directory (Azure AD).
La integración del software Cezanne HR con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso al software Cezanne HR.
* Puede permitir que los usuarios inicien sesión automáticamente en Cezanne HR Software (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con el software Cezanne HR, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Suscripción habilitada para el inicio de sesión único en Cezanne HR Software

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Cezanne HR Software admite el inicio de sesión único iniciado por **SP**.

## <a name="adding-cezanne-hr-software-from-the-gallery"></a>Adición del software Cezanne HR desde la galería

Para configurar la integración del software Cezanne HR en Azure AD, deberá agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar el software Cezanne HR desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Cezanne HR Software**, seleccione **Cezanne HR Software** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

    ![Cezanne HR Software en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Cezanne HR Software con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Cezanne HR Software.

Para configurar y probar el inicio de sesión único de Azure AD con el software Cezanne HR, debe completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Cezanne HR Software](#configure-cezanne-hr-software-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Cezanne HR Software](#create-cezanne-hr-software-test-user)** : para tener en Cezanne HR Software un homólogo de Britta Simon que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Cezanne HR Software, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Cezanne HR Software**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Cezanne HR Software](common/sp-identifier-reply.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://w3.cezanneondemand.com/CezanneOnDemand/-/<tenantidentifier>`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba la dirección URL: `https://w3.cezanneondemand.com/CezanneOnDemand/`.

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://w3.cezanneondemand.com:443/cezanneondemand/-/<tenantidentifier>/Saml/samlp`.
    
    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de URL de respuesta y URL de inicio de sesión. Para obtener estos valores, póngase en contacto con el [equipo de soporte técnico de Cezanne HR Software Client](https://cezannehr.com/services/support/).

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Cezanne HR Software** (Configurar Cezanne HR Software), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-cezanne-hr-software-single-sign-on"></a>Configuración del inicio de sesión único de Cezanne HR Software

1. En otra ventana del explorador web, inicie sesión en el inquilino del software Cezanne HR como administrador.

2. En el menú lateral, haga clic en **Administration** (Administración). A continuación, vaya a **Security Settings** (Configuración de seguridad) y haga clic en **Single Sign-On** (Inicio de sesión único).

    ![Captura de pantalla que muestra el inquilino del software Cezanne HR con Security Settings (Configuración de seguridad) y Single Sign-On Configuration (Configuración de inicio de sesión único) seleccionados.](https://user-images.githubusercontent.com/80324891/115692888-4c266900-a357-11eb-867d-7408b0ef16aa.png)

3. En el panel **Allow users to log in using the following Single Sign-On (SSO) Service** (Permitir a los usuarios iniciar sesión mediante el siguiente servicio de inicio de sesión único), marque la casilla **SAML 2.0** y seleccione la opción **Advanced Configuration** (Configuración avanzada).

    ![Captura de pantalla que muestra el panel Allow users (Permitir usuarios) con SAML 2.0 y Advanced Configuration (Configuración avanzada) seleccionados.](https://user-images.githubusercontent.com/80324891/115693054-72e49f80-a357-11eb-93c7-9986770ac17e.png)

4. Haga clic en el botón **Add New** (Agregar nuevo).

    ![Captura de pantalla que muestra el botón Add New (Agregar nuevo).](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_002.png)

5. Escriba los siguientes campos en la sección **SAML 2.0 IDENTITY PROVIDERS** (Proveedores de identidades de SAML 2.0) y haga clic en **OK** (Aceptar).

    ![Captura de pantalla que muestra un panel para escribir los valores que se describen en este paso.](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_003.png)

    a. **Display Name** (Nombre para mostrar): escriba el nombre del proveedor de identidades como el nombre para mostrar.

    b. **Entity Identifier** (Identificador de entidad): pegue el valor del identificador de Azure AD que ha copiado en Azure Portal en este cuadro de texto.

    c. **SAML Binding** (Enlace de SAML): cambie el enlace de SAML a "POST".

    d. **Security Token Service Endpoint** (Punto de conexión del servicio de token de seguridad): pegue el valor de la dirección URL de inicio de sesión que ha copiado de Azure Portal en este cuadro de texto.

    e. **User ID Attribute Name** (Nombre de atributo del identificador de usuario): escriba http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress en este cuadro de texto.

    f. **Public Key Certificate** (Certificado de clave pública): haga clic en el icono de carga para cargar el certificado que ha descargado de Azure Portal.

6. Haga clic en Aceptar.

7. Haga clic en el botón Guardar . 


### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba **brittasimon\@yourcompanydomain.extension**.  
    Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Cezanne HR Software.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **Cezanne HR Software**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Software Cezanne HR**.

    ![Vínculo de Cezanne HR Software en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-cezanne-hr-software-test-user"></a>Creación de un usuario de prueba de Cezanne HR Software

Para permitir que los usuarios de Azure AD inicien sesión en el software Cezanne HR, tienen que aprovisionarse en él. En el caso del software Cezanne HR, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en su sitio de la compañía del software Cezanne HR como administrador.

2. En el menú lateral, haga clic en **Administration** (Administración). A continuación, vaya a **Users** (Usuarios) y haga clic en **Add New User** (Agregar nuevo usuario).

    ![Captura de pantalla que muestra el inquilino del software Cezanne HR con Manage Users (Administrar usuarios) y Add New User (Agregar nuevo usuario) seleccionados.](https://user-images.githubusercontent.com/80324891/115694050-6ad92f80-a358-11eb-81be-148de665e185.png)

3. En la sección **PERSON DETAILS** (Datos de la persona), siga estos pasos:

    ![Captura de pantalla que muestra la sección PERSON DETAILS (DATOS DE LA PERSONA) para escribir los valores que se describen en este paso.](https://user-images.githubusercontent.com/80324891/115694321-a70c9000-a358-11eb-8325-de2582d135ec.png)

    a. En **Internal User** (Usuario interno), seleccione OFF (Desactivado).

    b. Escriba el nombre.   

    c. Escriba los apellidos.

    d. Escriba la dirección de correo electrónico.

4. En la sección **Account Information** (Información de la cuenta), siga estos pasos:

    ![Captura de pantalla que muestra ACCOUNT INFORMATION (INFORMACIÓN DE CUENTA) para escribir los valores que se describen en este paso.](https://user-images.githubusercontent.com/80324891/115694501-d3c0a780-a358-11eb-8873-0fc778b43775.png)

    a. En el cuadro de texto **Nombre de usuario**, escriba el correo electrónico de un usuario, por ejemplo, Brittasimon@contoso.com.

    b. En el cuadro de texto **Contraseña**, escriba la contraseña del usuario.

    c. Seleccione **HR Professional** (Profesional de RR.HH.) como **Security Role** (Rol de seguridad).

    d. Haga clic en **OK**.
    ![Captura de pantalla que muestra el botón Aceptar.](https://user-images.githubusercontent.com/80324891/115694644-f6eb5700-a358-11eb-9b23-a87a24921052.png)

5. Navegue a la pestaña **Single Sign-On** (Inicio de sesión único) y seleccione **Add New** (Agregar nuevo) en el área **SAML 2.0 Identifiers** (Identificadores SAML 2.0).

    ![Captura de pantalla que muestra la pestaña Single Sign-On (Inicio de sesión único) para seleccionar Add New (Agregar nuevo).](https://user-images.githubusercontent.com/80324891/115694716-0b2f5400-a359-11eb-9192-d31f6c9d3e3e.png)

6. Elija el proveedor de identidades en **Identity Provider** (Proveedor de identidades) y, en el cuadro de texto **User Identifier** (Identificador de usuario), escriba la dirección de correo electrónico del usuario.

    ![Captura de pantalla que muestra SAML 2.0 Identifiers (Identificadores de SAML 2.0) ara seleccionar el proveedor de identidades y el identificador de usuario.](https://user-images.githubusercontent.com/80324891/115694865-28fcb900-a359-11eb-9cd3-496a93124cc4.png)

7. Haga clic en el botón **Guardar** .

    ![Captura de pantalla que muestra el botón Save (Guardar) para User Settings (Configuración de usuario).](https://user-images.githubusercontent.com/80324891/115694880-3023c700-a359-11eb-85d4-83d057660cfb.png)

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Cezanne HR Software en el panel de acceso, debería iniciar sesión automáticamente en la aplicación Cezanne HR Software para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](./tutorial-list.md)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [¿Qué es el acceso condicional en Azure Active Directory?](../conditional-access/overview.md)
