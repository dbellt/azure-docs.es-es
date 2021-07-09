---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Evernote | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y Evernote.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/18/2021
ms.author: jeedes
ms.openlocfilehash: a7150ac0fc3d291019b9e53776ed8b92431f2d2b
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110451571"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-evernote"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Evernote

En este tutorial aprenderá a integrar Evernote con Azure Active Directory (Azure AD). Al integrar Evernote con Azure AD, se puede realizar lo siguiente:

* Controlar en Azure AD quién tiene acceso a Evernote.
* Permitir que los usuarios inicien sesión automáticamente en Evernote con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Evernote.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Evernote admite SSO iniciado por **SP e IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-evernote-from-the-gallery"></a>Incorporación de Evernote desde la galería

Para configurar la integración de Evernote en Azure AD, es preciso agregar Evernote desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Evernote** en el cuadro de búsqueda.
1. Seleccione **Evernote** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-evernote"></a>Configuración y prueba del SSO de Azure AD para Evernote

Configure y pruebe el inicio de sesión único de Azure AD con Evernote mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Evernote.

Para configurar el SSO de Azure AD con Evernote, realice los pasos siguientes:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Evernote](#configure-evernote-sso)** , para configurar los valores del inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Evernote](#create-evernote-test-user)** , para tener un homólogo de B.Simon en Evernote que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Evernote**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice los pasos siguientes: 

    En el cuadro de texto **Identificador**, escriba la dirección URL `https://www.evernote.com/saml2`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL: `https://www.evernote.com/Login.action`

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

7. Para modificar las opciones de **Firma**, haga clic en el botón **Editar** para abrir el cuadro de diálogo **Certificado de firma de SAML**.

    ![Captura de pantalla que muestra el cuadro de diálogo "Certificado de firma de S A M L" con el botón "Editar" seleccionado.](common/edit-certificate.png) 

    ![imagen](./media/evernote-tutorial/assertion.png)

    a. Seleccione **Firmar respuesta y aserción SAML** en **Opción de firma**.

    b. Haga clic en **Guardar**

1. En la sección **Configurar Evernote**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon use el inicio de sesión único de Azure concediéndole acceso a Evernote.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Evernote**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-evernote-sso"></a>Configuración del inicio de sesión único de Evernote

1. Para automatizar la configuración en Evernote, debe instalar la **extensión del explorador de inicio de sesión seguro de mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Configurar Evernote** para ir a la aplicación Evernote. En ella, escriba las credenciales de administrador para iniciar sesión en Evernote. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 6.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar Evernote manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Evernote como administrador y haga lo siguiente:

4. Vaya a **"Consola de administración"**

    ![Admin-Console](./media/evernote-tutorial/admin.png)

5. En la **"Consola de administración"** , vaya a **"Seguridad"** y seleccione **"Inicio de sesión único"**

    ![SSO-Setting](./media/evernote-tutorial/security.png)

6. Configure los valores siguientes:

    ![Certificate-Setting](./media/evernote-tutorial/certificate.png)
    
    a.  **Enable SSO** (Habilitar SSO): El SSO está habilitado de manera predeterminada (haga clic en **Disable Single Sign-on** [Deshabilitar el inicio de sesión único] para quitar el requisito de SSO).

    b. Pegue el valor del campo **Dirección URL de inicio de sesión** que copió de Azure Portal en el cuadro de texto **SAML HTTP Request URL** (Dirección URL de solicitud HTTP de SAML).

    c. Abra el certificado que se descargó de Azure AD en un bloc de notas y copie el contenido, incluido "BEGIN CERTIFICATE" y "END CERTIFICATE", y péguelo en el cuadro de texto **X.509 Certificate** (Certificado X.509). 

    d. Haga clic en **Guardar cambios**

### <a name="create-evernote-test-user"></a>Creación de un usuario de prueba en Evernote

Para permitir que los usuarios de Azure AD inicien sesión en Evernote, deben aprovisionarse en Evernote.  
En el caso de Evernote, el aprovisionamiento es una tarea manual.

**Para aprovisionar cuentas de usuario, realice estos pasos:**

1. Inicie sesión en el sitio de la empresa de Evernote como administrador.

2. Haga clic en la **"Consola de administración"** .

    ![Admin-Console](./media/evernote-tutorial/admin.png)

3. En la **"Consola de administración"** , vaya a **"Agregar usuarios"** .

    ![Captura de pantalla que muestra el menú "Usuarios" con la opción "Agregar usuarios" seleccionada.](./media/evernote-tutorial/create-user.png)

4. **Agregue miembros del equipo** en el cuadro de texto **Correo electrónico**, escriba la dirección de correo electrónico de la cuenta de usuario y haga clic en **Invitar**.

    ![Add-testUser](./media/evernote-tutorial/add-user.png)
    
5. Una vez que se envía la invitación, el titular de la cuenta de Azure Active Directory recibirá un correo electrónico para aceptar la invitación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Evernote, donde puede comenzar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de Evernote e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de Evernote para la que configuró el SSO. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Evernote en Aplicaciones, si seleccionó el modo SP en la configuración, debería acceder automáticamente a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión. Por el contrario, si seleccionó el modo IDP en la configuración, debería iniciar sesión automáticamente en la instancia de Evernote en la que configuró el SSO. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Evernote, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
