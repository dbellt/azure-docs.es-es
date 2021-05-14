---
title: 'Tutorial: Integración del inicio de sesión único de Azure Active Directory con Exium | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Exium.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/16/2021
ms.author: jeedes
ms.openlocfilehash: bcbf68523fda8ccd8041d5147f9904f0b8520521
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108147974"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-exium"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Exium

En este tutorial aprenderá a integrar Exium con Azure Active Directory (Azure AD). Al integrar Exium con Azure AD, podrá hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Exium.
* Permitir que los usuarios inicien sesión automáticamente en Exium con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción habilitada para el inicio de sesión único en Exium.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Exium admite el inicio de sesión único iniciado por el **proveedor de servicios**.

## <a name="adding-exium-from-the-gallery"></a>Incorporación de Exium desde la galería

Para configurar la integración de Exium en Azure AD, deberá agregar Exium desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Exium** en el cuadro de búsqueda.
1. Seleccione **Exium** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-exium"></a>Configuración y prueba del inicio de sesión único de Azure AD para Exium

Configure y pruebe el inicio de sesión único de Azure AD con Exium mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Exium.

Para configurar y probar el inicio de sesión único de Azure AD con Exium, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Exium](#configure-exium-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Exium](#create-exium-test-user)** , para tener un homólogo de B.Simon en Exium que esté vinculado a la representación de este usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Exium**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://subapi.exium.net/saml/<WORKSPACE_ID>/metadata`
    
    b.  En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://subapi.exium.net/saml/<WORKSPACE_ID>/acs`

    c. En el cuadro de texto **Dirección URL de inicio de sesión**, escriba la dirección URL: `https://service.exium.net/sign-in`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Póngase en contacto con el [equipo de soporte técnico de Exium](mailto:support@exium.net) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

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

En esta sección, va a permitir que B.Simon acceda a Exium mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Exium**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-exium-sso"></a>Configuración del inicio de sesión único de Exium

1. Inicie sesión en el sitio de la compañía de Exium como administrador.

1. En la **consola de administración**, seleccione el panel **Perfil de la empresa**.

    ![captura de pantalla de la consola de administración](./media/exium-tutorial/company-profile.png)

1. En el **perfil**, haga clic en **SSO Settings** (Configuración de SSO) y **edítela**.

1. Realice los pasos siguientes en la sección **SSO Settings** (Configuración de SSO).

    ![captura de pantalla de la configuración de SSO](./media/exium-tutorial/update.png)

    a. Seleccione **SSO Type** (Tipo de SSO) como **AzureAD** en la lista desplegable.

    b. Pegue el valor de **Dirección URL de metadatos de federación de aplicación** en el campo **Dirección URL de metadatos de de SAML 2.0 IDP**.

    c. Copie el valor de **URL de SSO de SAML 2.0**, péguelo en el cuadro de texto **Dirección URL de respuesta**, en la sección **Configuración básica de SAML** de Azure Portal.

    d. Copie el valor del **identificador de entidad del SP de SAML 2.0** y péguelo en el cuadro de texto **Identificador de Azure AD** de la sección **Configuración básica de SAML** en Azure Portal.  

    e. Haga clic en **Update** (Actualizar).

### <a name="create-exium-test-user"></a>Creación de un usuario de prueba de Exium

1. Inicie sesión en el sitio de la compañía de Exium como administrador.

1. Vaya a **Administración de usuarios > Usuarios** y haga clic en **Agregar usuario**.

    ![captura de pantalla de la creación de un usuario de prueba](./media/exium-tutorial/add-user.png)

1. Especifique los campos obligatorios en la página siguiente y haga clic en **Save** (Guardar).

    ![captura de pantalla de los campos de creación de un usuario de prueba con el botón guardar](./media/exium-tutorial/add-user-2.png)

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Exium, donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Exium e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Exium en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de dicha aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Exium, puede aplicar el control de sesión, que protege a su organización, en tiempo real, frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).