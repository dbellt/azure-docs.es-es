---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Samsung Knox and Business Services | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Samsung Knox and Business Services.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/11/2021
ms.author: jeedes
ms.openlocfilehash: 4c292838305baa44699b6309fcaaf7de7366ba94
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109753424"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-samsung-knox-and-business-services"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Samsung Knox and Business Services

En este tutorial, aprenderá a integrar Samsung Knox and Business Services con Azure Active Directory (Azure AD). Al integrar Samsung Knox and Business Services con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a Samsung Knox and Business Services.
* Permitir que los usuarios inicien sesión automáticamente en Samsung Knox and Business Services con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una cuenta de Samsung Knox.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Samsung Knox and Business Services admite el inicio de sesión único iniciado por **SP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-samsung-knox-and-business-services-from-the-gallery"></a>Adición de Samsung Knox and Business Services desde la galería

Para configurar la integración de Samsung Knox and Business Services en Azure AD, es preciso agregar Samsung Knox and Business Services desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la Galería**, escriba **Samsung Knox and Business Services** en el cuadro de búsqueda.
1. Seleccione **Samsung Knox and Business Services** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-samsung-knox-and-business-services"></a>Configuración y prueba del inicio de sesión único de Azure AD para Samsung Knox and Business Services

Configure y pruebe el inicio de sesión único de Azure AD con Samsung Knox and Business Services mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de [SamsungKnox.com](https://samsungknox.com/).

Para configurar y probar el inicio de sesión único de Azure AD con Samsung Knox and Business Services, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Samsung Knox and Business Services](#configure-samsung-knox-and-business-services-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación del usuario de prueba de Samsung Knox and Business Services](#create-samsung-knox-and-business-services-test-user)** , para tener un homólogo de B.Simon en Samsung Knox and Business Services que esté vinculado a la representación de Azure AD del usuario.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Samsung Knox and Business Services**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    * En el cuadro de texto **Dirección URL de inicio de sesión**, escriba la dirección URL: `https://www2.samsungknox.com/en/sso/login/ad`
    * En el cuadro de texto **URL de respuesta (URL del servicio de consumidor de aserciones)** , escriba la siguiente URL: `https://central.samsungknox.com/ams/ad/saml/acs`.
    
    ![Valores de Configuración básica de SAML](https://docs.samsungknox.com/assets/merge/ad-sso/basic-saml-configuration.png)

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar la **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

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

En esta sección, va a dar permiso a B.Simon para que pueda utilizar el inicio de sesión único de Azure concediéndole acceso a Samsung Knox and Business Services.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Samsung Knox and Business Services**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-samsung-knox-and-business-services-sso"></a>Configuración del inicio de sesión único de Samsung Knox and Business Services

1. En otra ventana del explorador web, inicie sesión en [SamsungKnox.com](https://samsungknox.com/) como administrador.

1. Haga clic en el **avatar** de la esquina superior derecha.

    ![Avatar de Samsung Knox](./media/samsung-knox-and-business-services-tutorial/avatar.png)

1. En la barra lateral izquierda, haga clic en **CONFIGURACIÓN DE ACTIVE DIRECTORY** y realice los pasos siguientes.

    ![CONFIGURACIÓN DE ACTIVE DIRECTORY](https://docs.samsungknox.com/assets/merge/ad-sso/ad-5.png)

    a. En el cuadro de texto **Identifier(entity ID)** (Identificador (ID de entidad)), introduzca el valor **Identificador**, que introdujo en Azure Portal.

    b. En el cuadro de texto **Dirección URL de metadatos de federación de aplicación**, pegue el valor de **Dirección URL de metadatos de federación de aplicación** que copió de Azure Portal.

    c. Haga clic en **CONNECT TO AD SSO** (CONECTARSE AL INICIO DE SESIÓN ÚNICO DE AD).

### <a name="create-samsung-knox-and-business-services-test-user"></a>Creación de un usuario de prueba de Samsung Knox and Business Services

En esta sección, creará un usuario llamado Britta Simon en Samsung Knox and Business Services. Consulte las guías de administración de [Knox Configure](https://docs.samsungknox.com/admin/knox-configure/Administrators.htm) o [Knox Mobile Enrollment](https://docs.samsungknox.com/admin/knox-mobile-enrollment/kme-add-an-admin.htm) para obtener instrucciones sobre cómo invitar a un subadministrador, o un usuario de prueba, a su organización de Samsung Knox. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta operación le redirigirá a [SamsungKnox.com](https://samsungknox.com/), donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a [SamsungKnox.com](https://samsungknox.com/) e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Samsung Knox and Business Services en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de [SamsungKnox.com](https://samsungknox.com/). Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Samsung Knox and Business Services, puede aplicar el control de sesión, que protege a la organización en tiempo real contra la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).