---
title: 'Tutorial: Integración de Azure Active Directory con Andromeda | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Andromeda.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/28/2020
ms.author: jeedes
ms.openlocfilehash: 45d7939840f1fd3055f29d3aee677e105890a94d
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108802097"
---
# <a name="tutorial-azure-active-directory-integration-with-andromeda"></a>Tutorial: Integración de Azure Active Directory con Andromeda

En este tutorial, aprenderá a integrar Andromeda con Azure Active Directory (Azure AD).
La integración de Andromeda con Azure AD proporciona las siguientes ventajas:

- Puede controlar en Azure AD quién tiene acceso a Andromeda.
- Puede permitir que los usuarios inicien sesión automáticamente en Andromeda (inicio de sesión único) con sus cuentas de Azure AD.
- Puede administrar sus cuentas en una ubicación central: Azure Portal.

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con Andromeda, se necesitan los siguientes elementos:

- Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
- Una suscripción que permita el inicio de sesión único en Andromeda

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

- Andromeda admite el inicio de sesión único iniciado por **SP e IDP**
- Andromeda admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="adding-andromeda-from-the-gallery"></a>Adición de Andromeda desde la galería

Para configurar la integración de Andromeda en Azure AD, será preciso que agregue Andromeda desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Andromeda** en el cuadro de búsqueda.
1. Seleccione **Andromeda** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-andromeda"></a>Configuración y prueba del inicio de sesión único de Azure AD para Andromeda

Configure y pruebe el inicio de sesión único de Azure AD con Andromeda mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Andromeda.

Para configurar y probar el inicio de sesión único de Azure AD con Andromeda, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
   1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
   1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
2. **[Configuración del inicio de sesión único en Andromeda](#configure-andromeda-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
   1. **[Creación de un usuario de prueba de Andromeda](#create-andromeda-test-user)** : para tener un homólogo de Britta Simon en Andromeda que esté vinculado a la representación del usuario en Azure AD.
3. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Andromeda**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

   a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<tenantURL>.ngcxpress.com/`

   b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<tenantURL>.ngcxpress.com/SAMLConsumer.aspx`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

   ![Captura de pantalla que muestra Establecer direcciones U R L adicionales donde puede escribir una U R L de inicio de sesión.](common/metadata-upload-additional-signon.png)

   En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<tenantURL>.ngcxpress.com/SAMLLogon.aspx`

   > [!NOTE]
   > Estos valores no son reales. El valor se actualizará con la dirección URL de inicio de sesión, la dirección URL de respuesta y el identificador reales, que se explican más adelante en el tutorial.

1. La aplicación Andromeda espera las aserciones de SAML en un formato específico. Configure las siguientes notificaciones para esta aplicación. Puede administrar los valores de estos atributos en la sección **Atributos de usuario** de la página de integración de aplicaciones. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el botón **Editar** para abrir el cuadro de diálogo **Atributos de usuario**.

   ![Captura de pantalla que muestra User Attributes (Atributos de usuario) con atributos como emailaddress user.mail y givenname user.givenname.](common/edit-attribute.png)

   > [!Important]
   > Desactive las definiciones de espacio de nombres al realizar esta configuración.

1. En la sección **Notificaciones del usuario** del cuadro de diálogo **Atributos de usuario**, edite las notificaciones mediante el **icono Editar** o agregue notificaciones mediante **Agregar nueva notificación** para configurar el atributo Token SAML como muestra la imagen anterior y realice los siguientes pasos:

   | Nombre    | Atributo de origen  |
   | ------- | ----------------- |
   | rol    | Rol específico de la aplicación |
   | type    | Tipo de aplicación          |
   | company | CompanyName       |

   > [!NOTE]
   > Andromeda espera roles para los usuarios asignados a la aplicación. Configure estos roles en Azure AD para que se puedan asignar los roles correspondientes a los usuarios. Para aprender a configurar roles en Azure AD, consulte [este vínculo](../develop/howto-add-app-roles-in-azure-ad-apps.md#app-roles-ui).

   a. Haga clic en **Agregar nueva notificación** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

   ![Captura de pantalla que muestra User claims (Reclamaciones de usuario) con las opciones Add new claim (Agregar nueva reclamación) y Save (Guardar).](common/new-save-attribute.png)

   ![Captura de pantalla que muestra Manage user claims (Administrar reclamaciones de usuario) para escribir los valores que se describen en este paso.](common/new-attribute-details.png)

   b. En el cuadro de texto **Nombre**, escriba el nombre que se muestra para la fila.

   c. Deje **Espacio de nombres** en blanco.

   d. Seleccione **Atributo** como origen.

   e. En la lista **Atributo de origen**, escriba el valor de atributo que se muestra para esa fila.

   f. Haga clic en **Aceptar**.

   g. Haga clic en **Save**(Guardar).

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

   ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up Andromeda** (Configurar Andromeda), copie las direcciones URL adecuada según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Andromeda mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Andromeda**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si ha configurado los roles tal y como se explica en el apartado anterior, puede seleccionarlo en la lista desplegable **Seleccionar un rol**.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="configure-andromeda-sso"></a>Configuración del inicio de sesión único de Andromeda

1. Inicie sesión como administrador en el sitio de la empresa Andromeda.

2. En la parte superior de la barra de menús, haga clic en **Admin** (Administrador) y vaya a **Administration** (Administración).

   ![Administración de Andromeda](./media/andromedascm-tutorial/tutorial_andromedascm_admin.png)

3. En el lado izquierdo de la barra de herramientas, en la sección **Interfaces** (Interfaces), haga clic en **SAML Configuration** (Configuración de SAML).

   ![SAML de Andromeda](./media/andromedascm-tutorial/tutorial_andromedascm_saml.png)

4. En la página **SAML Configuration** (Configuración de SAML), realice los siguientes pasos:

   ![Configuración de Andromeda](./media/andromedascm-tutorial/tutorial_andromedascm_config.png)

   a. Marque **Enable SSO with SAML** (Habilitar SSO con SAML).

   b. En la sección **Andromeda Information** (Información de Andromeda), copie el valor **SP Identity** (Identidad de SP) y péguelo en el cuadro de texto **Identificador** de la sección **Configuración básica de SAML**.

   c. Copie el valor de **Consumer URL** (Dirección URL de consumidor) y péguelo en el cuadro de texto **Reply URL** (URL de respuesta) de la sección **Configuración básica de SAML**.

   d. Copie el valor de **Logon URL** (Dirección URL de inicio de sesión) y péguelo en el cuadro de texto **Sign-on URL** (URL de inicio de sesión) de la sección **Configuración básica de SAML**.

   e. En la sección **SAML Identity Provider** (Proveedor de identidades SAML), escriba el nombre del proveedor de identidades.

   f. En el cuadro de texto **Single Sign On End Point** (Punto de conexión del inicio de sesión único), pegue el valor de **Login URL** (URL de inicio de sesión) que ha copiado de Azure Portal.

   g. Abra el **certificado codificado en Base64** descargado desde Azure Portal en el Bloc de notas y péguelo en el cuadro de texto **X 509 Certificate** (Certificado X509).

   h. Asigne los siguientes atributos con su respectivo valor para facilitar el inicio de sesión único desde Azure AD. El atributo **User ID** (Id. de usuario) es necesario para iniciar sesión. Para el aprovisionamiento, es necesario proporcionar los valores **Email** (Correo electrónico), **Company** (Compañía), **UserType** (Tipo de usuario) y **Role** (Rol). En esta sección, se define la asignación de atributos (nombre y valores) que se corresponden con los que se establecen en Azure Portal.

   ![Asignación de atributos de Andromeda](./media/andromedascm-tutorial/tutorial_andromedascm_attbmap.png)

   i. Haga clic en **Save**(Guardar).

### <a name="create-andromeda-test-user"></a>Creación de un usuario de prueba de Andromeda

En esta sección, se crea un usuario llamado Britta Simon en Andromeda. Andromeda admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si el usuario no existe ya en Andromeda, se crea uno después de la autenticación. Si necesita crear manualmente un usuario, es preciso que se ponga contacto con el [equipo de soporte técnico de Andromeda](https://www.ngcsoftware.com/support/).

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

#### <a name="sp-initiated"></a>Iniciado por SP:

- Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Andromeda, donde puede iniciar el flujo de inicio de sesión.

- Vaya directamente a la dirección URL de inicio de sesión de Andromeda e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

- Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de Andromeda para la que ha configurado el inicio de sesión único.

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Andromeda en Mis aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, debería iniciar sesión automáticamente en la instancia de Andromeda para la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Andromeda, puede aplicar el control de sesión, que protege su organización, en tiempo real, frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
