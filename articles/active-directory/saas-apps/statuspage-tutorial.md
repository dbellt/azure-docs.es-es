---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con StatusPage | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y StatusPage.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/31/2021
ms.author: jeedes
ms.openlocfilehash: 6bc2508220a8c0193efb55eb6cd11f7e151c80be
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106284598"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-statuspage"></a>Tutorial: Integración del inicio de sesión único de Azure Active Directory con StatusPage

En este tutorial, aprenderá a integrar StatusPage con Azure Active Directory (Azure AD). Al integrar StatusPage con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a StatusPage.
* Permitir que los usuarios inicien sesión automáticamente en StatusPage con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con StatusPage, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en StatusPage.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* StatusPage admite el inicio de sesión único comenzado por **IDP**.

## <a name="add-statuspage-from-the-gallery"></a>Incorporación de StatusPage desde la galería

Para configurar la integración de StatusPage en Azure AD, deberá agregar StatusPage desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **StatusPage** en el cuadro de búsqueda.
1. Seleccione **StatusPage** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-statuspage"></a>Configuración y prueba del inicio de sesión único de Azure AD para StatusPage

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con StatusPage con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de StatusPage.

Para configurar el inicio de sesión único de Azure AD con StatusPage, realice los pasos siguientes:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en StatusPage](#configure-statuspage-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de StatusPage](#create-statuspage-test-user)** : para tener un homólogo de Britta Simon en StatusPage que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **AskYourTeam**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con uno de los siguientes patrones:

    | Identificador |
    |--------------|
    | `https://<subdomain>.statuspagestaging.com/` |
    | `https://<subdomain>.statuspage.io/` |
    |

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con uno de los siguientes patrones:

     | URL de respuesta |
    |--------------|
    | `https://<subdomain>.statuspagestaging.com/sso/saml/consume` |
    | `https://<subdomain>.statuspage.io/sso/saml/consume` |
    |

    > [!NOTE]
    > Póngase en contacto con el equipo de soporte técnico de StatusPage [SupportTeam@statuspage.io](mailto:SupportTeam@statuspage.io)a fin de solicitar los metadatos necesarios para configurar el inicio de sesión único. 
    >
    > a. En los metadatos, copie el valor de Emisor y luego péguelo en el cuadro de texto **Identificador** .
    >
    > b. En los metadatos, copie el valor de URL de respuesta y luego péguelo en el cuadro de texto **URL de respuesta** .

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up StatusPage** (Configurar StatusPage), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a StatusPage.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y, después, **StatusPage**.

2. En la lista de aplicaciones, seleccione **StatusPage**.

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-statuspage-sso"></a>Configuración del inicio de sesión único en StatusPage

1. Para automatizar la configuración en StatusPage, debe instalar la **extensión del navegador de inicio de sesión seguro de Aplicaciones**. Para ello, haga clic en **Instale la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al navegador, haga clic en **Set up StatusPage** (Configurar StatusPage) para ir a la aplicación del mismo nombre. Desde allí, proporcione las credenciales del administrador para iniciar sesión en StatusPage. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 6.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar StatusPage manualmente, en otra ventana del explorador web, inicie sesión en el sitio de la empresa StatusPage como administrador.

1. En la barra de herramientas principal, haga clic en **Administrar cuenta**.

    ![Captura de pantalla que muestra la opción de administrar cuenta seleccionada en el sitio de la compañía StatusPage.](./media/statuspage-tutorial/account.png)

1. Haga clic en la pestaña **Inicio de sesión único** .

    ![Captura de pantalla que muestra la pestaña Inicio de sesión único.](./media/statuspage-tutorial/tab.png)

1. En la página Configuración de SSO, realice los pasos siguientes:

    ![Captura de pantalla que muestra la página de configuración del inicio de sesión único, donde puede especificar los valores descritos.](./media/statuspage-tutorial/setup.png)

    ![Captura de pantalla que muestra el botón para guardar la configuración.](./media/statuspage-tutorial/configuration.png)

    a. En el cuadro de texto **SSO Target URL** (Dirección URL de destino de SSO), pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    b. Abra el certificado descargado en el Bloc de notas, copie el contenido y luego péguelo en el cuadro de texto **Certificado** .

    c. Haga clic en **GUARDAR CONFIGURACIÓN**.

### <a name="create-statuspage-test-user"></a>Creación de un usuario de prueba de StatusPage

El objetivo de esta sección es crear un usuario de prueba llamado Britta Simon en StatusPage.

StatusPage admite el aprovisionamiento Just-In-Time. Ya lo ha habilitado en [Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso).

**Para crear un usuario llamado Britta Simon en StatusPage, realice los pasos siguientes:**

1. Inicie sesión en su sitio de la empresa StatusPage como administrador.

1. En el menú de la parte superior, haga clic en **Administrar cuenta**.

    ![Captura de pantalla que muestra la opción de administrar cuenta seleccionada en el sitio de la compañía StatusPage.](./media/statuspage-tutorial/account.png)

1. Haga clic en la pestaña **Team Members** (Miembros del equipo).
  
    ![Captura de pantalla que muestra la pestaña Miembros del equipo.](./media/statuspage-tutorial/sandbox.png) 

1. Haga clic en **ADD TEAM MEMBER**(Agregar miembro del equipo).
  
    ![Captura de pantalla que muestra el botón para agregar un miembro del equipo.](./media/statuspage-tutorial/team.png) 

1. Escriba **Dirección de correo electrónico**, **Nombre** y **Apellido** en los cuadros de texto correspondientes para un usuario válido que quiera aprovisionar. 

    ![Captura de pantalla que muestra el cuadro de diálogo para agregar usuario, donde puede especificar los valores descritos.](./media/statuspage-tutorial/user.png) 

1. En **Rol**, elija **Administrador de clientes**.

1. Haga clic en **CREATE ACCOUNT** (CREAR CUENTA).

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

* Haga clic en Probar esta aplicación en Azure Portal. Se debería iniciar sesión automáticamente en la instancia de StatusPage para la que configurara el inicio de sesión único.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de StatusPage en Aplicaciones, debería iniciar sesión automáticamente en la instancia de StatusPage para la que configurara el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurada StatusPage, puede aplicar el control de sesión, que protege la información confidencial de la organización de la filtración y la infiltración en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).