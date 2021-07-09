---
title: 'Tutorial: Integración de Azure Active Directory con LearnUpon | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y LearnUpon.
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
ms.openlocfilehash: 5f789baf7467af863c59a44ab9a256c9f6d7fd15
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110075276"
---
# <a name="tutorial-azure-active-directory-integration-with-learnupon"></a>Tutorial: Integración de Azure Active Directory con LearnUpon

En este tutorial, aprenderá a integrar LearnUpon con Azure Active Directory (Azure AD). Al integrar LearnUpon con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a LearnUpon.
* Permitir que los usuarios inicien sesión automáticamente en LearnUpon con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con LearnUpon, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en LearnUpon.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* LearnUpon admite SSO iniciado por **IDP.**

* LearnUpon admite el aprovisionamiento de usuarios **Just In Time**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-learnupon-from-the-gallery"></a>Incorporación de LearnUpon desde la galería

Para configurar la integración de LearnUpon en Azure AD, es preciso agregar LearnUpon desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **LearnUpon** en el cuadro de búsqueda.
1. Seleccione **LearnUpon** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-learnupon"></a>Configuración y prueba del SSO de Azure AD para LearnUpon

Configure y pruebe el SSO de Azure AD con LearnUpon mediante un usuario de prueba llamado **B.Simon**. Para que el SSO funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de LearnUpon.

Para configurar y probar el SSO de Azure AD con LearnUpon, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del SSO en LearnUpon](#configure-learnupon-sso)** : para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de LearnUpon](#create-learnupon-test-user)** : el objetivo es tener un homólogo de B.Simon en LearnUpon que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **LearnUpon**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.learnupon.com/saml/consumer`

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de respuesta real. Póngase en contacto con el [equipo de soporte técnico de LearnUpon](https://www.learnupon.com/features/support/) para obtener este valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, ubique la **huella digital**. Se agregará a su configuración de SAML de LearnUpon.

    ![Vínculo de descarga del certificado](common/certificateraw.png)

6. En la sección **Configurar LearnUpon**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a LearnUpon utilizando el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **LearnUpon**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-learnupon-sso"></a>Configuración del SSO de LearnUpon

1. Abra otra instancia del explorador e inicie sesión en LearnUpon con una cuenta de administrador.

1. Haga clic en la pestaña **settings** (Configuración).

    ![Captura de pantalla que muestra la pestaña de configuración.](./media/learnupon-tutorial/settings.png)

1. Haga clic en **Single Sign On - SAML** (Inicio de sesión único - SAML) y, después, en **General Settings** (Configuración general) para configurar SAML.
   
    ![Captura de pantalla que muestra Single Sign On - SAML (Inicio de sesión único - SAML) con la opción General settings (Configuración general) seleccionada.](./media/learnupon-tutorial/general-settings.png) 

1. En la sección **General Settings** (Configuración general), siga estos pasos:
   
    ![Captura de pantalla que muestra la sección General Settings (Configuración general), donde puede especificar los valores descritos.](./media/learnupon-tutorial/values.png)  
  
    a. Seleccione **Habilitado**.

    b. Seleccione la **versión** **2.0**.

    c. En **Skip conditions** (Omitir condiciones), haga clic en **No**.

    d. En el cuadro de texto **SAML Token Post param name** (Nombre de parámetro POST de token SAML), escriba el nombre del parámetro POST de la solicitud en la dirección URL del consumidor de SAML indicada anteriormente que contenga la aserción SAML que se va a comprobar y autenticar (por ejemplo, **SAMLResponse**).

    e. En el cuadro de texto **Name Identifier Format** (Formato de identificador de nombre), escriba el valor que indica en qué lugar de la aserción SAML reside el identificador del usuario (dirección de correo electrónico) (por ejemplo, `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`).
  
    f. En el cuadro de texto **Identify Provider Location** (Identificar la ubicación del proveedor), escriba el valor que indica el lugar al que se envían los usuarios si hacen clic en el icono cargado desde la pantalla de inicio de sesión de Azure Portal.
  
    g. En el cuadro de texto **Sign Out URL** (URL de cierre de sesión), pegue el valor de la **URL de cierre de sesión** que ha copiado de Azure Portal.

    h. Haga clic en **Manage finger prints**(Administrar huellas dactilares) y, a continuación, cargue la huella digital del certificado descargado.

1. Haga clic en **User Settings**(Configuración del usuario) y siga estos pasos:

     ![Captura de pantalla que muestra la sección User Settings (Configuración de usuario), donde puede especificar los valores descritos.](./media/learnupon-tutorial/user-settings.png)  

    a. En el cuadro de texto **First Name Identifier Format** (Formato de identificador de nombre), escriba el valor que nos indica el lugar de la aserción SAML en que reside el nombre de los usuarios (por ejemplo: `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`).
  
    b. En el cuadro de texto **Last Name Identifier Format** (Formato de identificador de apellido), escriba el valor que nos indica el lugar de la aserción SAML en que reside el apellido de los usuarios (por ejemplo: `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`).

### <a name="create-learnupon-test-user"></a>Creación de un usuario de prueba de LearnUpon

En esta sección, se crea un usuario llamado Britta Simon en LearnUpon. LearnUpon admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en LearnUpon, se crea otro después de la autenticación. Si necesita crear un usuario manualmente, es preciso que se ponga contacto con el [equipo de soporte técnico de LearnUpon](https://www.learnupon.com/features/support/).

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

* Haga clic en Probar esta aplicación en Azure Portal; debería iniciar sesión automáticamente en la instancia de LearnUpon para la que configuró el SSO.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de LearnUpon en Aplicaciones, debería iniciar sesión automáticamente en la versión de LearnUpon para la que configuró el SSO. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado LearnUpon, puede aplicar el control de sesión, que protege su organización, en tiempo real, frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
