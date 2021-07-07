---
title: 'Tutorial: Integración de Azure Active Directory con Brightidea | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Brightidea.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/19/2021
ms.author: jeedes
ms.openlocfilehash: f020d66238cb34f457627a0fda78176678bdf1d0
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110463903"
---
# <a name="tutorial-azure-active-directory-integration-with-brightidea"></a>Tutorial: Integración de Azure Active Directory con Brightidea

En este tutorial, aprenderá a integrar Brightidea con Azure Active Directory (Azure AD). Al integrar Brightidea con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Brightidea.
* Permitir que los usuarios inicien sesión automáticamente en Brightidea con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Brightidea.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Brightidea admite el SSO iniciado por **SP e IDP**.
* Brightidea admite el aprovisionamiento de usuarios **Just-In-Time**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-brightidea-from-the-gallery"></a>Incorporación de Brightidea desde la galería

Para configurar la integración de Brightidea en Azure AD, es preciso agregar Brightidea desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Brightidea** en el cuadro de búsqueda.
1. Seleccione **Brightidea** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-brightidea"></a>Configuración y prueba del SSO de Azure AD para Brightidea

Configure y pruebe el SSO de Azure AD con Brightidea mediante un usuario de prueba llamado **B.Simon**. Para que el SSO funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Brightidea.

Para configurar y probar el SSO de Azure AD con Brightidea, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del SSO en Brightidea](#configure-brightidea-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Brightidea](#create-brightidea-test-user)** : para tener un homólogo de B.Simon en Brightidea que esté vinculado a la representación de ella en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Brightidea**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si tiene el **archivo de metadatos del proveedor de servicios** y desea realizar la configuración en el modo iniciado por **IDP**, realice los pasos siguientes:

    a. Haga clic en **Cargar el archivo de metadatos**.

    ![Carga del archivo de metadatos](common/upload-metadata.png)

    b. Haga clic en el **logotipo de la carpeta** para seleccionar el archivo de metadatos y luego en **Cargar**.

    ![Elección del archivo de metadatos](common/browse-upload-metadata.png)

    c. Una vez que se haya cargado correctamente el archivo de metadatos, los valores **Identificador** y **URL de respuesta** se rellenan automáticamente en el cuadro de texto de la sección Brightidea:

    > [!Note]
    > Si los valores **Identificador** y **Dirección URL de respuesta** no se rellenan automáticamente, hágalo manualmente según sus necesidades.

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.brightidea.com`

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up Brightidea** (Configurar Brightidea), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, habilitará a B.Simon para que use el inicio de sesión único de Azure concediéndole acceso a Brightidea.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Brightidea**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-brightidea-sso"></a>Configuración del SSO en Brightidea

1. En una ventana diferente del explorador web, inicie sesión en Brightidea mediante las credenciales de administrador.

2. Para ir a la característica de inicio de sesión único en el sistema Brightidea, vaya a **Enterprise Setup** -> **Authentication Tab** (Configuración de empresa > pestaña Autenticación). Habrá dos subpestañas: Auth Selection (Selección automática) y SAML Profiles (Perfiles de SAML)

    ![Captura de pantalla que muestra el sitio de Brightidea con la pestaña Authentication (Autenticación) seleccionada.](./media/brightidea-tutorial/authentication.png)

3. Seleccione **Auth Selection** (Selección automática). De forma predeterminada, solo se muestran dos métodos estándar: Brightidea Login (Inicio de sesión en Brightidea) y Registration (Registro). Cuando se agrega un método de inicio de sesión único, aparece en la lista.

    ![Captura de pantalla que muestra la pestaña Authentication (Autenticación) de Brightidea con la opción Auth Selection (Selección de autenticación) seleccionada.](./media/brightidea-tutorial/selection.png)

4. Seleccione **SAML Profiles** (Perfiles de SAML) y realice los siguientes pasos:

    ![Captura de pantalla que muestra la pestaña Authentication (Autenticación) de Brightidea con SAML Profiles (Perfiles de SAML) seleccionado, que proporciona las opciones Download Metadata (Descargar metadatos) y Add New (Agregar nuevo).](./media/brightidea-tutorial/profile.png)

    a. Haga clic en **Download Metadata** (Descargar metadatos) y cargue la sección **Basic SAML Configuration** (Configuración básica de SAML) en Azure Portal.

    b. Haga clic en el botón **Add New** (Agregar nuevo) en **Identity Provider Setting** (Configuración del proveedor de identidades), realice los pasos siguientes:

    ![Captura de pantalla que muestra Identity Provider Setting (Configuración del proveedor de identidades) de Brightidea para escribir información.](./media/brightidea-tutorial/metadata.png)

   * Escriba el **nombre del perfil de SAML**, por ejemplo, `Azure Ad SSO`.

   * Para **cargar metadatos**, haga clic para elegir el archivo y cargue el archivo de metadatos descargado de Azure Portal.

     > [!NOTE]
     > Después de cargar el archivo de metadatos, los campos restantes **Single Sign-on Service (Servicio de inicio de sesión único), Identity Provider Issuer (Emisor del proveedor de identidades), Upload Public Key (Cargar clave pública)** se rellenan automáticamente.

   * En el cuadro de texto **Email** (Correo electrónico), escriba el valor como `mail`.

   * En el cuadro de texto **Screen Name** (Nombre de pantalla), escriba el valor como `givenName`.

   * Haga clic en **Guardar cambios**. 

### <a name="create-brightidea-test-user"></a>Creación de un usuario de prueba de Brightidea

En esta sección, se crea un usuario llamado a Britta Simon en Brightidea. Brightidea admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si el usuario no existe ya en Brightidea, se crea uno después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Brightidea, donde puede iniciar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de Brightidea y comience el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* En Azure Portal, haga clic en **Probar esta aplicación**. Al hacerlo, debería iniciar sesión automáticamente en la instancia de Brightidea en la que configuró el SSO. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Cuando haga clic en el icono de Brightidea en Aplicaciones, si seleccionó el modo SP en la configuración, debería acceder automáticamente a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión. Por el contrario, si seleccionó el modo IDP en la configuración, debería iniciar sesión en la instancia de Brightidea en la que configuró el SSO. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Brightidea, puede aplicar el control de sesión, que protege a su organización, en tiempo real, frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
