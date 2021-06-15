---
title: 'Tutorial: Integración de Azure Active Directory con Carbonite Endpoint Backup | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Carbonite Endpoint Backup.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/03/2021
ms.author: jeedes
ms.openlocfilehash: 8aea830fed1c184f974260fddb9edda90df55329
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111964708"
---
# <a name="tutorial-integrate-carbonite-endpoint-backup-with-azure-active-directory"></a>Tutorial: Integración de Carbonite Endpoint Backup con Azure Active Directory

En este tutorial, aprenderá a integrar Carbonite Endpoint Backup con Azure Active Directory (Azure AD). Al integrar Carbonite Endpoint Backup con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a Carbonite Endpoint Backup.
* Permitir que los usuarios inicien sesión automáticamente en Carbonite Endpoint Backup con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción habilitada para el inicio de sesión único (SSO) de Carbonite Endpoint Backup.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Carbonite Endpoint Backup admite el inicio de sesión único iniciado por **SP e IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-carbonite-endpoint-backup-from-the-gallery"></a>Incorporación de Carbonite Endpoint Backup desde la galería

Para configurar la integración de Carbonite Endpoint Backup en Azure AD, deberá agregar la aplicación desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Carbonite Endpoint Backup** en el cuadro de búsqueda.
1. Seleccione **Carbonite Endpoint Backup** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-carbonite-endpoint-backup"></a>Configuración y prueba del inicio de sesión único de Azure AD para Carbonite Endpoint Backup

Configure y pruebe el inicio de sesión único de Azure AD con Carbonite Endpoint Backup mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Carbonite Endpoint Backup.

Para configurar y probar el inicio de sesión único de Azure AD con Carbonite Endpoint Backup, es preciso seguir los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Carbonite Endpoint Backup](#configure-carbonite-endpoint-backup-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación del usuario de prueba de Carbonite Endpoint Backup](#create-carbonite-endpoint-backup-test-user)** : para tener un homólogo de Britta Simon en Carbonite Endpoint Backup vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Carbonite Endpoint Backup**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una de las siguientes direcciones URL:

    ```http
    https://red-us.mysecuredatavault.com
    https://red-apac.mysecuredatavault.com
    https://red-fr.mysecuredatavault.com
    https://red-emea.mysecuredatavault.com
    https://kamino.mysecuredatavault.com
    ```

    b. En el cuadro de texto **URL de respuesta**, escriba una de las siguientes direcciones URL:

    ```http
    https://red-us.mysecuredatavault.com/AssertionConsumerService.aspx
    https://red-apac.mysecuredatavault.com/AssertionConsumerService.aspx
    https://red-fr.mysecuredatavault.com/AssertionConsumerService.aspx
    https://red-emea.mysecuredatavault.com/AssertionConsumerService.aspx
    ```

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una de las siguientes direcciones URL:

    ```http
    https://red-us.mysecuredatavault.com/
    https://red-apac.mysecuredatavault.com/
    https://red-fr.mysecuredatavault.com/
    https://red-emea.mysecuredatavault.com/
    ```

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up Carbonite Endpoint Backup** (Configurar Carbonite Endpoint Backup), copie las direcciones URL que necesite.

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

En esta sección va a conceder acceso a B.Simon a Carbonite Endpoint Backup para permitirle usar el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Carbonite Endpoint Backup**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-carbonite-endpoint-backup-sso"></a>Configuración del inicio de sesión único en Carbonite Endpoint Backup

1. Para automatizar la configuración en Carbonite Endpoint Backup, debe instalar la **extensión del explorador de inicio de sesión seguro de Aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Tras agregar la extensión al explorador, haga clic en **Configurar Carbonite Endpoint Backup**; se le dirigirá a la aplicación Carbonite Endpoint Backup. Desde allí, proporcione las credenciales de administrador para iniciar sesión en Carbonite Endpoint Backup. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 7.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar manualmente Carbonite Endpoint Backup, abra una nueva ventana del explorador web, inicie sesión en el sitio de la compañía de Carbonite Endpoint Backup como administrador y realice los pasos siguientes:

4. Haga clic en **Company** (Compañía) en el panel izquierdo.

    ![Captura de pantalla que muestra Carbonite Endpoint con la opción Company (Empresa) seleccionada.](media/carbonite-endpoint-backup-tutorial/company.png)

5. Haga clic en **Single sign-on** (Inicio de sesión único).

    ![Captura de pantalla que muestra Company (Empresa) con la opción Single sign-on (Inicio de sesión único) seleccionada.](media/carbonite-endpoint-backup-tutorial/single-sign-on.png)

6. Haga clic en **Enable** (Habilitar) y en **Edit settings** (Editar configuración) para la configuración.

    ![Captura de pantalla que muestra la pestaña Single sign-on (Inicio de sesión único) con las opciones Enable (Habilitar) y Edit settings (Editar configuración) seleccionadas.](media/carbonite-endpoint-backup-tutorial/settings.png)

7. En la página de configuración **Single sign-on** (Inicio de sesión único), siga estos pasos:

    ![Captura de pantalla que muestra la pestaña Single sign-on (Inicio de sesión único) con la información que se describe en este paso.](media/carbonite-endpoint-backup-tutorial/save.png)

    1. En el cuadro de texto **Identity provider name** (Nombre del proveedor de identidades), pegue el valor de **Azure AD Identifier** (Identificador de Azure AD) que ha copiado de Azure Portal.

    1. En el cuadro de texto **Identity Provider URL** (Dirección URL del proveedor de identidades), pegue el valor de **Login URL** (Dirección URL de inicio de sesión) que ha copiado de Azure Portal.

    1. Haga clic en **Elegir archivo** para cargar el archivo **Certificado (Base64)** que ha descargado de Azure Portal.

    1. Haga clic en **Save**(Guardar).

### <a name="create-carbonite-endpoint-backup-test-user"></a>Creación del usuario de prueba en Carbonite Endpoint Backup

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la compañía de Carbonite Endpoint Backup.

1. Haga clic en **Usuarios** en el panel izquierdo y en **Agregar usuario**.

    ![Captura de pantalla que muestra la página Carbonite Endpoint con las opciones Users (Usuarios) y Add users (Agregar usuarios) seleccionadas.](media/carbonite-endpoint-backup-tutorial/add-user-1.png)

1. En la página **Agregar usuario**, siga estos pasos:

    ![Captura de pantalla que muestra la página Add user (Agregar usuario), donde puede realizar los pasos que se describen aquí.](media/carbonite-endpoint-backup-tutorial/add-user-2.png)

    1. Escriba el **correo electrónico**, el **nombre** y los **apellidos** del usuario y proporciónele los permisos necesarios los requisitos de la organización.

    1. Haga clic en **Agregar usuario**.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Carbonite Endpoint Backup, donde puede iniciar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de Carbonite Endpoint Backup e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de Carbonite Endpoint Backup para la que ha configurado el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Carbonite Endpoint Backup en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, debería iniciar sesión automáticamente en la instancia de Carbonite Endpoint Backup para la que haya configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Carbonite Endpoint Backup, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).