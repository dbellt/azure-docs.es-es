---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con GitHub AE | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y GitHub AE.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/18/2021
ms.author: jeedes
ms.openlocfilehash: 721c13ea31f812dd5f5b5c335373884766764c80
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108750786"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-github-ae"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con GitHub AE

En este tutorial aprenderá a integrar GitHub AE con Azure Active Directory (Azure AD). Al integrar GitHub AE con Azure AD, podrá hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a GitHub AE.
* Permitir que los usuarios inicien sesión automáticamente en GitHub AE con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* GitHub AE, listo para la [inicialización](https://docs.github.com/github-ae@latest/admin/configuration/initializing-github-ae).

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* GitHub AE admite el inicio de sesión único iniciado por **SP** e **IDP**.
* GitHub AE admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="adding-github-ae-from-the-gallery"></a>Incorporación de GitHub AE desde la galería

Para configurar la integración de GitHub AE en Azure AD, debe agregar GitHub AE desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **GitHub AE** en el cuadro de búsqueda.
1. Seleccione **GitHub AE** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-github-ae"></a>Configuración y prueba del inicio de sesión único de Azure AD para GitHub AE

Configure y pruebe el inicio de sesión único de Azure AD con GitHub AE mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario correspondiente de GitHub AE.

Para configurar y probar el inicio de sesión único de Azure AD con GitHub AE, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en GitHub AE](#configure-github-ae-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en GitHub AE](#create-github-ae-test-user)** : para tener un homólogo de B.Simon en GitHub AE vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **GitHub AE**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<YOUR-GITHUB-AE-HOSTNAME>`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<YOUR-GITHUB-AE-HOSTNAME>/saml/consume`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

     En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<YOUR-GITHUB-AE-HOSTNAME>/sso`

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de inicio de sesión, la dirección URL de respuesta y el identificador reales. Póngase en contacto con el [equipo de soporte al cliente de GitHub AE](mailto:support@github.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.


1. La aplicación GitHub AE espera las aserciones SAML en un formato específico. Para ello, es necesario agregar asignaciones de atributos personalizados a la configuración de atributos de token SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Edite **Atributos y notificaciones de usuario**.

1. Haga clic en **Agregar nueva notificación** y escriba el nombre como **administrador** en el cuadro de texto (el valor de **administrador** distingue mayúsculas de minúsculas).

1. Expanda **Condiciones de la notificación** y seleccione **Miembros** en **Tipo de usuario**.

1. Haga clic en **Seleccionar grupos** y busque el **grupo** en el que desee incluir esta notificación, donde sus miembros deben ser administradores de GHAE.

1. Seleccione **Atributo** para **Origen** y escriba **true** (sin comillas) para **Valor**. 

1. Haga clic en **Save**(Guardar).

    ![administración de notificaciones](./media/github-ae-tutorial/administrator.png)

    > [!NOTE]
    > Para ver las instrucciones sobre cómo agregar una notificación, consulte el [vínculo](https://docs.github.com/en/github-ae@latest/admin/authentication/configuring-authentication-and-provisioning-for-your-enterprise-using-azure-ad).

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up GitHub AE** (Configurar GitHub AE), copie las direcciones URL que necesite.

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

En esta sección va a permitir que B.Simon acceda a GitHub AE mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **GitHub AE**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-github-ae-sso"></a>Configuración del inicio de sesión único en GitHub AE

Para configurar el inicio de sesión único en GitHub AE, debe seguir las instrucciones mencionadas [aquí](https://docs.github.com/github-ae@latest/admin/authentication/configuring-saml-single-sign-on-for-your-enterprise#enabling-saml-sso).

### <a name="create-github-ae-test-user"></a>Creación de un usuario de prueba en GitHub AE

En esta sección se crea un usuario llamado B.Simon en GitHub AE. GitHub AE admite el aprovisionamiento de usuarios Just-In-Time, habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si todavía no existe un usuario en GitHub AE, se crea uno después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de GitHub AE, donde podrá iniciar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de GitHub AE e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de GitHub AE para la que configurara el inicio de sesión único. 

También puede usar el Panel de acceso de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de GitHub AE en el panel de acceso, si tiene la configuración del modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para iniciar el flujo de inicio de sesión. Si tiene la configuración del modo IDP, debería iniciarse sesión automáticamente en la instancia de GitHub AE para la que configurara el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

* [Configuración del aprovisionamiento de usuarios para la empresa](https://docs.github.com/github-ae@latest/admin/authentication/configuring-user-provisioning-for-your-enterprise).

* Una vez configurada la aplicación GitHub AE, podrá aplicar el control de sesión, que protege a su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
