---
title: 'Tutorial: integración de Azure Active Directory con Workfront | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Workfront.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/23/2021
ms.author: jeedes
ms.openlocfilehash: f6d69d20684433cf8df8b121734458bacefda3b8
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108206842"
---
# <a name="tutorial-azure-active-directory-integration-with-workfront"></a>Tutorial: integración de Azure Active Directory con Workfront

En este tutorial, aprenderá a integrar Workfront con Azure Active Directory (Azure AD). Al integrar Workfront con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Workfront.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Workfront con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con Workfront, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Workfront.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Workfront admite SSO iniciado por **SP**.

## <a name="add-workfront-from-the-gallery"></a>Adición de Workfront desde la galería

Para configurar la integración de Workfront en Azure AD, será preciso que agregue Workfront desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Workfront** en el cuadro de búsqueda.
1. Seleccione **Workfront** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-workfront"></a>Configuración y prueba del inicio de sesión único de Azure AD para Workfront

Configure y pruebe el inicio de sesión único de Azure AD con Workfront mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Workfront.

Para configurar y probar el inicio de sesión único de Azure AD con Workfront, siga los pasos que se indican a continuación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Workfront](#configure-workfront-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Workfront](#create-workfront-test-user)** : para tener un homólogo de B. Simon en Workfront que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Workfront**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.attask-ondemand.com`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<companyname>.attasksandbox.com/SAML2`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte de cliente de Workfront](https://www.workfront.com/services-and-support) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Workfront** (Configurar Workfront), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Workfront mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Workfront**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-workfront-sso"></a>Configuración del inicio de sesión único de Workfront

1. Inicie sesión en su sitio de la empresa Workfront como administrador.

2. Vaya a **Single Sign On Configuration**(Configuración de inicio de sesión único).

3. En el cuadro de diálogo **Inicio de sesión único** , siga estos pasos.
    
    ![Configurar inicio de sesión único](./media/workfront-tutorial/single-sign-on.png)
   
    a. En **Tipo**, seleccione **SAML 2.0**.
   
    b. Seleccione **Service Provider ID** (Id. del proveedor de servicios).
   
    c. Pegue el valor de **Dirección URL de inicio de sesión** en el cuadro de texto **Login Portal URL** (URL de portal de inicio de sesión).
   
    d. Pegue la **dirección URL de cierre de sesión** en el cuadro de texto **Sign-Out URL** (URL de cierre de sesión).
   
    e. Pegue el valor de **Cambiar dirección URL de contraseña** en el cuadro de texto **Cambiar dirección URL de contraseña**.
   
    f. Haga clic en **Save**(Guardar).

### <a name="create-workfront-test-user"></a>Creación de un usuario de prueba de Workfront

El objetivo de esta sección es crear un usuario de prueba llamado Britta Simon en Workfront.

**Para crear un usuario llamado Britta Simon en Workfront, siga estos pasos:**

1. Inicie sesión en su sitio de la empresa Workfront como administrador.
 
2. En el menú de la parte superior, haga clic en **People**(Personas).
 
3. Haga clic en **New Person**(Nueva persona). 

4. En el cuadro de diálogo Nueva persona, realice los pasos siguientes:
   
    ![Crear un usuario de prueba en Workfront](./media/workfront-tutorial/add-person.png)
   
    a. En el cuadro de texto **First Name** (Nombre), escriba "Britta".
   
    b. En el cuadro de texto **Last Name** (Apellidos), escriba "Simon".
   
    c. En el cuadro de texto **E-mail Address** (Dirección de correo electrónico), escriba la dirección de correo electrónico de Britta Simon en Azure Active Directory.
   
    d. Haga clic en **Add Person**(Agregar persona).

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Workfront, desde donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Workfront e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Workfront en Aplicaciones, se le redirigirá a la URL de inicio de sesión de esa aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Workfront, puede aplicar el control de sesión, que protege a su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).