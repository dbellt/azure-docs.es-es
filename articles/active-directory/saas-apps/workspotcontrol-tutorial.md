---
title: 'Tutorial: integración de Azure Active Directory con Workspot Control | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Workspot Control.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/25/2021
ms.author: jeedes
ms.openlocfilehash: c8b241c8d6068cdf1ea6d7e36e872f48ca12b9ea
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110469070"
---
# <a name="tutorial-azure-active-directory-integration-with-workspot-control"></a>Tutorial: integración de Azure Active Directory con Workspot Control

En este tutorial, aprenderá a integrar Workspot Control con Azure Active Directory (Azure AD). Al integrar Workspot Control con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Workspot Control.
* Permitir que los usuarios inicien sesión automáticamente en Workspot Control con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Workspot Control, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).

* Una suscripción habilitada para el inicio de sesión único en Workspot Control.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Workspot Control admite el inicio de sesión único iniciado por SP e iniciado por IDP.

## <a name="add-workspot-control-from-the-gallery"></a>Adición de Workspot Control desde la galería

Para configurar la integración de Workspot Control en Azure AD, será preciso que agregue Workspot Control desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Workspot Control** en el cuadro de búsqueda.
1. Seleccione **Workspot Control** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-workspot-control"></a>Configuración y prueba del inicio de sesión único de Azure AD para Workspot Control

Configure y pruebe el inicio de sesión único de Azure AD con Workspot Control mediante un usuario de prueba llamado **B.Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Workspot Control.

Para configurar y probar el inicio de sesión único de Azure AD con Workspot Control, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Workspot Control](#configure-workspot-control-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Workspot Control](#create-workspot-control-test-user)** : para tener un homólogo de B.Simon en Workspot Control que esté vinculado a su representación en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Workspot Control**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por IDP, siga estos pasos:

    1. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón:<br/>
    `https://<<i></i>INSTANCENAME>-saml.workspot.com/saml/metadata`

    1. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón:<br/>
    `https://<<i></i>INSTANCENAME>-saml.workspot.com/saml/assertion`

5. Seleccione **Establecer direcciones URL adicionales** si desea configurar la aplicación en el modo iniciado por SP.

    En el cuadro de texto **Dirección URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón:<br/>
    `https://<<i></i>INSTANCENAME>-saml.workspot.com/`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico al cliente de Workspot Control](mailto:support@workspot.com) para obtener estos valores. También puede hacer referencia a los patrones de la sección **Configuración básica de SAML** de Azure Portal.

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione **Descargar** para descargar el **certificado (Base 64)** de las opciones disponibles según sus requisitos. Guárdelo en el equipo.

    ![Vínculo de descarga del certificado (Base64)](common/certificatebase64.png)

7. En la sección **Configurar Workspot Control**, copie las direcciones URL adecuadas según sus necesidades:

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

En esta sección va a permitir que B.Simon acceda a Workspot Control mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Workspot Control**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-workspot-control-sso"></a>Configuración del inicio de sesión único de Workspot Control

1. En otra ventana del explorador web, inicie sesión en Workspot Control como administrador de seguridad.

2. En la barra de herramientas de la parte superior de la página, seleccione **Setup** (Configurar) y, a continuación, **SAML**.

    ![Opciones de configuración](./media/workspotcontrol-tutorial/setup.png)

3. En la ventana **Security Assertion Markup Language Configuration** (Configuración de Lenguaje de marcado de aserción de seguridad), realice los pasos siguientes:
 
    ![Ventana de Configuración de Lenguaje de marcado de aserción de seguridad](./media/workspotcontrol-tutorial/security.png)

    1. En el cuadro **Entity ID** (Identificador de entidad), pegue el **identificador de Azure AD** que copió de Azure Portal.

    1. En el cuadro **Signon Service URL** (Dirección URL del servicio de inicio de sesión), pegue la **Dirección URL de inicio de sesión** que copió de Azure Portal.

    1. En el cuadro **Logout Service URL** (Dirección URL del servicio de cierre de sesión), pegue la **Dirección URL de cierre de sesión** que copió de Azure Portal.

    1. Seleccione **Update File** (Actualizar archivo) para cargar en el certificado X.509 el certificado codificado en base 64 que descargó de Azure Portal.

    1. Seleccione **Guardar**.

### <a name="create-workspot-control-test-user"></a>Creación de un usuario de prueba de Workspot Control

Para permitir que los usuarios de Azure AD inicien sesión en Workspot Control, tienen que aprovisionarse en Workspot Control. El aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, siga estos pasos:**

1. Inicie sesión en Workspot Control como administrador de seguridad.

2. En la barra de herramientas de la parte superior de la página, seleccione **Users** (Usuarios) y, a continuación, **Add User** (Agregar usuario).

    ![Opciones de "Usuarios"](./media/workspotcontrol-tutorial/user.png)

3. En la ventana **Add a New User** (Agregar un nuevo usuario), siga estos pasos:

    ![Ventana "Agregar un nuevo usuario"](./media/workspotcontrol-tutorial/new-user.png)

    1. En el cuadro de texto **First Name** (Nombre), escriba el nombre del usuario, en este caso **Britta**.

    1. En el cuadro de texto **Last Name** (Apellidos), escriba los apellidos del usuario, en este caso **Simon**.

    1. En el cuadro de texto **Email** (Correo electrónico), escriba la dirección de correo electrónico del usuario, como **Brittasimon@contoso.<i></i>com**.

    1. Seleccione el rol de usuario correspondiente de la lista desplegable **Role** (Rol).

    1. Seleccione el grupo de usuarios correspondiente de la lista desplegable **Group** (Grupo).

    1. Seleccione **Agregar usuario**.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Workspot Control, donde puede iniciar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de Workspot Control e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de Workspot Control para la que ha configurado el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Workspot Control en Aplicaciones, si está configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para iniciar el flujo de inicio de sesión y, si está configurado en modo IDP, debería iniciar sesión automáticamente en la instancia de Workspot Control para la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Workspot Control, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
