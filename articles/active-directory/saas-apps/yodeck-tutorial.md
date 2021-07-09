---
title: 'Tutorial: Integración de Azure Active Directory con Yodeck | Microsoft Docs'
description: Obtenga información para configurar el inicio de sesión único entre Azure Active Directory y Yodeck.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/02/2021
ms.author: jeedes
ms.openlocfilehash: 79c71a0df5c08f314aace126cc0960636faa6a27
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111571236"
---
# <a name="tutorial-azure-active-directory-integration-with-yodeck"></a>Tutorial: Integración de Azure Active Directory con Yodeck

En este tutorial, aprenderá a integrar Yodeck con Azure Active Directory (Azure AD). La integración de Yodeck con Azure AD permite:

* Controlar en Azure AD quién tiene acceso a Yodeck.
* Habilitar que los usuarios inicien sesión automáticamente en Yodeck con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con Yodeck, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción habilitada para el inicio de sesión único en Yodeck.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Yodeck admite el inicio de sesión único iniciado por **SP** e **IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-yodeck-from-the-gallery"></a>Incorporación de Yodeck desde la galería

Para configurar la integración de Yodeck en Azure AD, debe agregar Yodeck desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Yodeck** en el cuadro de búsqueda.
1. Seleccione **Yodeck** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-yodeck"></a>Configuración y prueba del inicio de sesión único de Azure AD en Yodeck

Configure y pruebe el inicio de sesión único de Azure AD con Yodeck mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Yodeck.

Para configurar y probar el inicio de sesión único de Azure AD con Yodeck, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Yodeck](#configure-yodeck-sso)** : para configurar el inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Yodeck](#create-yodeck-test-user)** : para tener un homólogo de B. Simon en Yodeck que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Yodeck**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice el siguiente paso:

    En el cuadro de texto **Identificador**, escriba la dirección URL `https://app.yodeck.com/api/v1/account/metadata/`

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL: `https://app.yodeck.com/login`

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

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

En esta sección, va a permitir que B. Simon utilice el inicio de sesión único de Azure, ya que le concederá acceso a Yodeck.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Yodeck**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-yodeck-sso"></a>Configuración del inicio de sesión único de Yodeck

1. Para automatizar la configuración en **Yodeck**, debe instalar la **extensión del explorador de inicio de sesión seguro Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Captura de pantalla que muestra el botón de instalación de la extensión.](./media/target-process-tutorial/install_extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Configurar Yodeck** para ir a la aplicación Yodeck. En ella, escriba las credenciales de administrador para iniciar sesión en Yodeck. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 5.

    ![Configuración](common/setup-sso.png)

    **Si desea configurar la aplicación manualmente, realice los siguientes pasos:**

1. En otra ventana del explorador web, inicie sesión en el sitio de la empresa de Yodeck como administrador.

1. Haga clic en la opción **Configuración de usuario** de la esquina superior derecha de la página y seleccione **Configuración de cuenta**.

    ![Captura de pantalla que muestra la configuración de cuenta seleccionada para el usuario.](./media/yodeck-tutorial/account.png)

1. Seleccione **SAML** y realice los siguientes pasos:

    ![Captura de pantalla que muestra la pestaña SAML, desde donde puede realizar estos pasos.](./media/yodeck-tutorial/configure.png)

    a. Seleccione **Importar desde URL**.

    b. En el cuadro de texto **URL**, pegue el valor **Dirección URL de metadatos de federación de aplicación** que ha copiado de Azure Portal y haga clic en **Importar**.
    
    c. Después de importar **Dirección URL de metadatos de federación de aplicación**, los campos restantes se rellenan automáticamente.

    d. Haga clic en **Save**(Guardar).

### <a name="create-yodeck-test-user"></a>Creación de un usuario de prueba de Yodeck

Para permitir que los usuarios de Azure AD inicien sesión en Yodeck, tienen que aprovisionarse en Yodeck. En el caso de Yodeck, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en el sitio de la empresa de Yodeck como administrador.

2. Haga clic en la opción **Configuración de usuario** de la esquina superior derecha de la página y seleccione **Usuarios**.

    ![Captura de pantalla que muestra los usuarios seleccionados.](./media/yodeck-tutorial/user.png)

3. Haga clic en **+Usuario** para abrir la pestaña **Detalles del usuario**.

    ![Captura de pantalla que muestra el botón de usuarios.](./media/yodeck-tutorial/user-details.png)

4. En la página de diálogo **Detalles del usuario**, lleve a cabo los pasos siguientes:

    ![Captura de pantalla que muestra la pestaña Detalles del usuario, desde donde puede realizar estos pasos.](./media/yodeck-tutorial/user-page.png)

    a. En el cuadro de texto **Nombre**, escriba el nombre del usuario, en este caso, **Britta**.

    b. En el cuadro de texto **Apellidos**, escriba los apellidos del usuario, en este caso, **Simon**.

    c. En el cuadro de texto **Correo electrónico**, escriba la dirección de correo electrónico de un usuario, por ejemplo, brittasimon@contoso.com.

    d. Seleccione la opción **Permisos de cuenta** adecuada según los requisitos de la organización.
    
    e. Haga clic en **Save**(Guardar).

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Yodeck, donde puede comenzar el flujo de inicio de sesión.  

* Vaya a directamente a la dirección URL de inicio de sesión de Yodeck y comience el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de Yodeck para la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Yodeck en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; sin embargo, si se ha configurado en modo IDP, debería iniciar sesión automáticamente en la instancia de Yodeck para la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Tras configurar Yodeck, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
