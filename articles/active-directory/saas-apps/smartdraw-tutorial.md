---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SmartDraw | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SmartDraw.
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
ms.openlocfilehash: 623fc1d74f7cebc6a7773805c19f5994949fdabb
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110455451"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-smartdraw"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SmartDraw

En este tutorial, obtendrá información sobre cómo integrar SmartDraw con Azure Active Directory (Azure AD). Al integrar SmartDraw con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a SmartDraw.
* Permitir que los usuarios puedan iniciar sesión automáticamente en SmartDraw con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en SmartDraw.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SmartDraw admite el SSO iniciado tanto por **SP como por IDP**.
* SmartDraw admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="add-smartdraw-from-the-gallery"></a>Incorporación de SmartDraw desde la galería

Para configurar la integración de SmartDraw en Azure AD, es preciso agregar SmartDraw desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **SmartDraw** en el cuadro de búsqueda.
1. Seleccione **SmartDraw** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-smartdraw"></a>Configuración y prueba del SSO de Azure AD para SmartDraw

Configure y pruebe el inicio de sesión único de Azure AD con SmartDraw mediante un usuario de prueba llamado **B.Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de SmartDraw.

Para configurar y probar el SSO de Azure AD con SmartDraw, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en SmartDraw](#configure-smartdraw-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de SmartDraw](#create-smartdraw-test-user)** : para tener un homólogo de B.Simon en SmartDraw que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **SmartDraw**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, el usuario no tiene que realizar ningún paso porque la aplicación ya se ha integrado previamente con Azure.

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://cloud.smartdraw.com/sso/saml/login/<DOMAIN>`

    > [!NOTE]
    > El valor de la dirección URL de inicio de sesión no es real. El valor de la dirección URL de inicio de sesión se actualizará con el real, lo que se explica más adelante en el tutorial. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. Haga clic en **Save**(Guardar).

1. La aplicación SmartDraw espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación SmartDraw espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

    | Nombre | Atributo de origen|
    | ---------------| --------------- |
    | Nombre | user.givenname |
    | Apellidos | user.surname |
    | Email | user.mail |
    | Grupos | user.groups |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar SmartDraw**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección va a permitir que B.Simon acceda a SmartDraw mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **SmartDraw**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-smartdraw-sso"></a>Configuración del inicio de sesión único de SmartDraw

1. Para automatizar la configuración en SmartDraw, debe instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**; para ello, haga clic en **Instalar la aplicación**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Configurar SmartDraw** para ir a la aplicación SmartDraw. Desde allí, proporcione las credenciales del administrador para iniciar sesión en SmartDraw. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 5.

    ![Configuración](common/setup-sso.png)

1. Si quiere configurar SmartDraw manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de SmartDraw como administrador y haga lo siguiente:

1. Haga clic en **Inicio de sesión único** en Manage your SmartDraw License (Administrar su licencia de SmartDraw).

    ![Captura de pantalla que muestra el cuadro de diálogo para administrar la licencia de SmartDraw, donde puede seleccionar la opción de inicio de sesión único.](./media/smartdraw-tutorial/single-sign-on.png)

1. En la página Configuración, siga estos pasos:

    ![Captura de pantalla que muestra la página de configuración, donde puede especificar los valores descritos.](./media/smartdraw-tutorial/configuration.png)

    a. En el cuadro de texto **Su dominio (como acme.com)** , escriba el dominio.

    b. Copie el valor de **Your SP Initiated Login Url will be** (Su dirección URL de inicio de sesión iniciado por SP será) de la instancia y péguelo en el cuadro de texto Sign-on URL (Dirección URL de inicio de sesión) en **Configuración básica de SAML** en Azure Portal.

    c. En el cuadro de texto **Security Groups to Allow SmartDraw Access** (Grupos de seguridad para permitir el acceso a SmartDraw), escriba **Everyone** (Todos).

    d. En el cuadro de texto **Your SAML Issuer URL** (Dirección URL del emisor de SAML), pegue el valor del **identificador de Azure AD** que ha copiado de Azure Portal.

    e. En el Bloc de notas, abra el archivo XML de metadatos que descargó de Azure Portal, copie el contenido y luego péguelo en el cuadro **Your SAML MetaData** (Sus metadatos de SAML).

    f. Haga clic en **Guardar configuración**. 

### <a name="create-smartdraw-test-user"></a>Creación de usuario de prueba de SmartDraw

En esta sección, se crea un usuario llamado B.Simon en SmartDraw. SmartDraw admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe en SmartDraw, se crea otro después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de SmartDraw, desde donde puede comenzar el flujo de inicio de sesión.  

* Acceda directamente a la URL de inicio de sesión de SmartDraw y ponga en marcha el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de SmartDraw para la que configuró el SSO. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de SmartDraw de Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de SmartDraw para la que ha configurado el SSO. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que se ha configurado SmartDraw, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
