---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con BlogIn | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y BlogIn.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/09/2021
ms.author: jeedes
ms.openlocfilehash: bba164ea19f1a4d4f70fb9151ae4f833ecd01515
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111891138"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-blogin"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con BlogIn

En este tutorial aprenderá a integrar BlogIn con Azure Active Directory (Azure AD). Al integrar BlogIn con Azure AD, podrá:

* Controlar en Azure AD quién tiene acceso a BlogIn.
* Permitir que los usuarios inicien sesión automáticamente en BlogIn con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en BlogIn.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* BlogIn admite el inicio de sesión único iniciado por **SP e IDP**.
* BlogIn admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="add-blogin-from-the-gallery"></a>Adición de BlogIn desde la galería

Para configurar la integración de BlogIn en Azure AD, será preciso agregar BlogIn desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **BlogIn** en el cuadro de búsqueda.
1. Seleccione **BlogIn** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-blogin"></a>Configuración y prueba del inicio de sesión único de Azure AD para BlogIn

Configure y pruebe el inicio de sesión único de Azure AD con BlogIn mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de BlogIn.

Para configurar y probar el inicio de sesión único de Azure AD con BlogIn, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en BlogIn](#configure-blogin-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en BlogIn](#create-blogin-test-user)** , para tener un homólogo de B.Simon en BlogIn vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **BlogIn**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, siga los pasos a continuación:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<SUBDOMAIN>.blogin.co/`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.blogin.co/sso/saml/callback`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.blogin.co/`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Puede obtener los valores exactos de estos campos en la página **Settings** (Configuración) de BlogIn, en la pestaña **User Authentication** > **Configure SSO and User Provisioning** (Autenticación de usuarios > Configurar SSO y el aprovisionamiento de usuarios). También puede ponerse en contacto con el [equipo de soporte técnico al cliente de BlogIn](mailto:support@blogin.co) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación BlogIn espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación BlogIn espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.
    
    | Nombre | Atributo de origen |
    | ------ | --------- |
    | title |user.jobtitle |
    

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

En esta sección, va a permitir que B.Simon acceda a BlogIn mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **BlogIn**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-blogin-sso"></a>Configuración del inicio de sesión único de BlogIn

Para configurar el inicio de sesión único en **BlogIn**, inicie sesión en su cuenta de BlogIn y siga estos pasos:

1. Vaya a **Settings** > **User Authentication** > **Configure SSO & User provisioning** (Configuración > Autenticación de usuarios > Configurar SSO y el aprovisionamiento de usuarios).
2. En la siguiente pantalla, cambie el estado de inicio de sesión único a **On** (Activado) y elija un nombre personalizado para el botón de inicio de sesión de SSO que se mostrará en la pantalla de inicio de sesión.

3. Si guardó el valor de **App Federation Metadata Url** (Dirección URL de metadatos de federación de la aplicación) en el último paso de la sección anterior, elija el método de configuración **URL Metadata** (Dirección URL de metadatos) y pegue el valor guardado en el campo URL Metadata (Dirección URL de metadatos). De lo contrario, cambie el método de configuración a **Manual**, rellene manualmente **Identity Provider SSO URL (Login URL)** (Dirección URL de SSO del proveedor de identidades [dirección URL de inicio de sesión)] e **Identity Provider Issuer (entity ID)** (Emisor del proveedor de identidades [identificador de entidad]), y cargue el **certificado (Base64)** que ha recibido de Azure AD.

4. Elija el rol de usuario predeterminado para los nuevos usuarios que se unan a BlogIn mediante el inicio de sesión único de usuario.

5. Seleccione **Save changes** (Guardar los cambios).

Para obtener una explicación más detallada de cómo configurar el inicio de sesión único en BlogIn, consulte [Configuración del inicio de sesión único para Microsoft Azure AD en BlogIn](https://blogin.co/blog/how-to-set-up-single-sign-on-sso-for-microsoft-azure-active-directory-azure-ad-267/). No dude en ponerse en contacto con el [equipo de soporte técnico de BlogIn](mailto:support@blogin.co) en cualquier momento si tiene alguna pregunta o necesita ayuda.

### <a name="create-blogin-test-user"></a>Creación del usuario de prueba de BlogIn

En esta sección, se crea un usuario llamado B.Simon en BlogIn. BlogIn admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si no existe todavía un usuario en BlogIn, se crea uno nuevo después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de BlogIn, donde puede iniciar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de BlogIn e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* En Azure Portal, haga clic en **Probar esta aplicación**. Al hacerlo, debería iniciar sesión automáticamente en la instancia de BlogIn en la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de BlogIn en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de BlogIn para la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado BlogIn, puede aplicar el control de sesión, que protege a la organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
