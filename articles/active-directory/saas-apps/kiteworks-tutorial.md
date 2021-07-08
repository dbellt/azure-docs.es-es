---
title: 'Tutorial: Integración de Azure Active Directory con Kiteworks | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Kiteworks.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/12/2021
ms.author: jeedes
ms.openlocfilehash: 197caadb0c6aa091b646868b856b28f627a9dd66
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110452903"
---
# <a name="tutorial-integrate-kiteworks-with-azure-active-directory"></a>Tutorial: Integración de Kiteworks con Azure Active Directory

En este tutorial, obtendrá información sobre cómo integrar Kiteworks con Azure Active Directory (Azure AD). Al integrar Kiteworks con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Kiteworks.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Kiteworks con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para inicio de sesión único (SSO) en Kiteworks.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Kiteworks admite SSO iniciado por **SP**.
* Kiteworks admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="add-kiteworks-from-the-gallery"></a>Adición de Kiteworks desde la galería

Para configurar la integración de Kiteworks en Azure AD, es preciso agregar Kiteworks desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Kiteworks** en el cuadro de búsqueda.
1. Seleccione **Kiteworks** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-kiteworks"></a>Configuración y prueba del SSO de Azure AD para Kiteworks

Configure y pruebe el inicio de sesión único de Azure AD con Kiteworks utilizando un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Kiteworks.

Para configurar y probar el SSO de Azure AD con Kiteworks, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del SSO en Kiteworks](#configure-kiteworks-sso)** : para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Kiteworks](#create-kiteworks-test-user)** : para tener un homólogo de B.Simon en Kiteworks vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Kiteworks**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<kiteworksURL>.kiteworks.com`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<kiteworksURL>/sp/module.php/saml/sp/saml2-acs.php/sp-sso`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte de cliente de Kiteworks](https://accellion.com/support) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up Kiteworks** (Configurar Kiteworks), copie las direcciones URL que necesite.

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

En esta sección, va a permitir que B.Simon acceda a Kiteworks mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Kiteworks**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-kiteworks-sso"></a>Configuración del inicio de sesión único de Kiteworks

1. Inicie sesión en su sitio de la compañía de Kiteworks como administrador.

1. En la barra de herramientas de la parte superior, haga clic en el icono de **Configuración**.

    ![Captura de pantalla que muestra el icono "Settings" (Configuración) de la barra de herramientas seleccionado.](./media/kiteworks-tutorial/settings.png)

1. En la sección **Autenticación y autorización**, haga clic en **SSO Setup** (Instalación de SSO).

    ![Captura de pantalla que muestra la opción "S S O Setup" (Configuración del inicio de sesión único) seleccionada en la sección "Authentication and Authorization" (Autenticación y autorización).](./media/kiteworks-tutorial/authentication.png)

1. En la página Configuración de SSO, realice los pasos siguientes:

    ![Configurar inicio de sesión único](./media/kiteworks-tutorial/setup-page.png)

    a. Seleccione **Autenticar mediante SSO**.

    b. Seleccione **Iniciar AuthnRequest**.

    c. En el cuadro de texto **IDP Entity ID** (Identificador de la entidad de IdP), pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    d. En el cuadro de texto **Dirección URL del servicio de inicio de sesión único**, pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    e. En el cuadro de texto **URL del servicio de cierre de sesión único**, pegue el valor de **URL de cierre de sesión** que ha copiado de Azure Portal.

    f. Abra el certificado descargado en el Bloc de notas, copie el contenido y péguelo en el cuadro de texto **Certificado de clave pública RSA** .

    g. Haga clic en **Save**(Guardar).

### <a name="create-kiteworks-test-user"></a>Creación de un usuario de prueba en Kiteworks

En esta sección, va a crear un usuario llamado B.Simon en Kiteworks. Kiteworks admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en Kiteworks, se crea otro después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Kiteworks, donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Kiteworks e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Kiteworks en Aplicaciones, se le redirigirá a la URL de inicio de sesión de esa plataforma. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que se ha configurado Kiteworks, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
