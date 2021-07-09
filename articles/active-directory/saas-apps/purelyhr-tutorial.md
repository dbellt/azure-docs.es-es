---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con PurelyHR | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y PurelyHR.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/26/2021
ms.author: jeedes
ms.openlocfilehash: 84a3af1c8d61df9645adf98798bb55631d9edaf9
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111570860"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-purelyhr"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con PurelyHR

En este tutorial, aprenderá a integrar PurelyHR con Azure Active Directory (Azure AD). Al integrar PurelyHR con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a PurelyHR.
* Permitir que los usuarios inicien sesión automáticamente en PurelyHR con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en PurelyHR.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* PurelyHR admite el inicio de sesión único iniciado por **SP e IDP**.
* PurelyHR admite el aprovisionamiento de usuarios **Just-In-Time**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-purelyhr-from-the-gallery"></a>Adición de PurelyHR desde la galería

Para configurar la integración de PurelyHR en Azure AD, deberá agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **PurelyHR** en el cuadro de búsqueda.
1. Seleccione **PurelyHR** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-purelyhr"></a>Configuración y prueba del inicio de sesión único de Azure AD para PurelyHR

Configure y pruebe el inicio de sesión único de Azure AD con PurelyHR mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de PurelyHR.

Para configurar y probar el inicio de sesión único de Azure AD con PurelyHR, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en PurelyHR](#configure-purelyhr-sso)**, para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de PurelyHR](#create-purelyhr-test-user)**, para tener un homólogo de B. Simon en PurelyHR que esté vinculado a la representación de Azure AD de usuario.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **PurelyHR**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice el siguiente paso:

    En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<COMPANY_ID>.purelyhr.com/sso-consume`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<COMPANY_ID>.purelyhr.com/sso-initiate`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de PurelyHR](https://support.purelyhr.com/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar PurelyHR**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B. Simon acceda a PurelyHR mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **PurelyHR**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-purelyhr-sso"></a>Configuración del inicio de sesión único de PurelyHR

1. Para automatizar la configuración en PurelyHR, es preciso instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Configurar PurelyHR** para ir a la aplicación del mismo nombre. En ella, escriba las credenciales de administrador para iniciar sesión en PurelyHR. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 5.

    ![Configuración](common/setup-sso.png)

1. Si desea configurar PurelyHR manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de PurelyHR como administrador y haga lo siguiente:

1. Abra el **panel** en las opciones de la barra de herramientas y haga clic en **SSO Settings** (Configuración de SSO).

1. Pegue los valores en los cuadros tal y como se describe a continuación:

    ![Configurar inicio de sesión único](./media/purelyhr-tutorial/dashboard.png)    

    a. Abra el archivo **Certificate(Bas64)** que se ha descargado de Azure Portal en el bloc de notas y copie el valor del certificado. Pegue el valor copiado en el cuadro **X.509 Certificate** (Certificado X.509).

    b. En el cuadro **IdP Issuer URL** (URL del emisor de IdP), pegue el **identificador de Azure AD** que copió de Azure Portal.

    c. En el cuadro **IDP Endpoint URL** (URL de punto de conexión de IDP), pegue la **dirección URL de inicio de sesión** que copió de Azure Portal. 

    d. Active la casilla **Auto-Create Users** (Creación automática de usuarios) para habilitar el aprovisionamiento automático de usuarios en PurelyHR.

    e. Haga clic en **Guardar cambios** para guardar la configuración.

### <a name="create-purelyhr-test-user"></a>Creación de un usuario de prueba en PurelyHR

Este paso no suele ser necesario porque la aplicación admite aprovisionamiento de usuarios Just-In-Time. Si el aprovisionamiento automático de usuarios no se habilita, puede llevarse a cabo la creación manual de usuarios tal y como se describe a continuación.

Inicie sesión como administrador en su sitio de la compañía de Velpic SAML y realice los pasos siguientes:

1. Haga clic en la pestaña Manage (Administrar), vaya a la sección Users (Usuarios) y haga clic en el botón New (Nuevo) para agregar usuarios.

    ![agregar usuario](./media/purelyhr-tutorial/new-user.png)

2. En el cuadro de diálogo **"Create New User"** (Crear nuevo usuario), realice los pasos siguientes.

    ![usuario](./media/purelyhr-tutorial/create-user.png)

    a. En el cuadro de texto **First Name** (Nombre), escriba el nombre de B.

    b. En el cuadro de texto **Apellido**, escriba el apellido Simon.

    c. En el cuadro de texto **User Name** (Nombre de usuario), escriba el nombre del usuario de B.Simon.

    d. En el cuadro de texto **Correo electrónico**, escriba la dirección de correo electrónico de la cuenta B.Simon@contoso.com.

    e. El resto de la información es opcional, puede rellenarla si es necesario.

    f. Haga clic en **GUARDAR**.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de PurelyHR, donde puede iniciar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de PurelyHR e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de PurelyHR para la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de PurelyHR en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de PurelyHR para la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que configure PurelyHR, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
