---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SignalFx | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SignalFx.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/25/2021
ms.author: jeedes
ms.openlocfilehash: f3fbab143fd330e017e9e51c74a403d9bfb687ff
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110455664"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-signalfx"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SignalFx

En este tutorial, aprenderá a integrar SignalFx con Azure Active Directory (Azure AD). Si integra SignalFx con Azure AD, podrá hacer lo siguiente:

* Controlar desde Azure AD quién tiene acceso a SignalFx.
* Permitir que los usuarios puedan iniciar sesión automáticamente en SignalFx con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en SignalFx.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SignalFx admite el SSO iniciado por **IDP**.
* SignalFx admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="step-1-add-the-signalfx-application-in-azure"></a>Paso 1: Incorporación de la aplicación SignalFx en Azure

Utilice estas instrucciones para agregar la aplicación SignalFx a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal.
1. En la ventana de navegación del lado izquierdo, seleccione **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, en el cuadro de búsqueda, escriba y seleccione **SignalFx**.
     * Es posible que tenga que esperar unos minutos para que la aplicación se agregue al inquilino.
1. Deje abierto Azure Portal y, a continuación, abra una nueva pestaña web.    

## <a name="step-2-begin-signalfx-sso-configuration"></a>Paso 2: Inicio de la configuración del inicio de sesión único de SignalFx

Siga estas instrucciones para comenzar el proceso de configuración del inicio de sesión único de SignalFx.

1. En la pestaña recién abierta, acceda a la interfaz de usuario de SignalFx e inicie sesión. 
1. En el menú superior, haga clic en **Integrations** (Integraciones). 
1. En el campo de búsqueda, escriba y seleccione **Azure Active Directory**.
1. Haga clic en **Create New Integration** (Crear nueva integración).
1. En **Name** (Nombre), escriba un nombre fácilmente reconocible que los usuarios puedan entender.
1. Marque **Show on Login Page** (Mostrar en la página de inicio de sesión).
    * Esta característica mostrará un botón personalizado en la página de inicio de sesión en el que los usuarios podrán hacer clic. 
    * La información que escribió en **Name** (Nombre) aparecerá en el botón. Por ello, escriba un valor para **Name** (Nombre) que los usuarios puedan reconocer. 
    * Esta opción solo funcionará si usa un subdominio personalizado para la aplicación SignalFx; por ejemplo, **nombredelaempresa.signalfx.com**. Para obtener un subdominio personalizado, póngase en contacto con el equipo de soporte técnico de SignalFx. 
1. Copie el valor de **Integration ID** (Identificador de integración). Necesitará esta información en un paso posterior. 
1. Deje abierta la interfaz de usuario de SignalFx. 

## <a name="step-3-configure-azure-ad-sso"></a>Paso 3: Configuración del inicio de sesión único de Azure AD

Siga estas instrucciones para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. Vuelva a Azure Portal y, en la página de integración de la aplicación **SignalFx**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos: 

    a. En **Identificador**, escriba la dirección URL `https://api.<realm>.signalfx.com/v1/saml/metadata` y reemplace `<realm>` por el dominio Kerberos de SignalFx. 

    b. En **URL de respuesta**, escriba la dirección URL `https://api.<realm>.signalfx.com/v1/saml/acs/<integration ID>` y reemplace `<realm>` por el dominio Kerberos de SignalFx y `<integration ID>` por el valor de **Integration ID** (Identificador de integración) que copió antes de la interfaz de usuario de SignalFx.

1. La aplicación SignalFx espera las aserciones de SAML en un formato específico. Para ello, es necesario agregar asignaciones de atributos personalizados a la configuración de los atributos del token SAML. 
    
1. Revise y compruebe que las siguientes notificaciones se asignan a los atributos de origen que se rellenan en Active Directory. 

    | Nombre |  Atributo de origen|
    | ------------------- | -------------------- |
    | User.FirstName  | user.givenname |
    | User.email  | user.mail |
    | PersonImmutableID       | user.userprincipalname    |
    | User.LastName       | user.surname    |

    > [!NOTE]
    > Este proceso requiere que Active Directory esté configurado con al menos un dominio personalizado comprobado, además de tener acceso a las cuentas de correo electrónico de este dominio. Si no está seguro o necesita ayuda con esta configuración, póngase en contacto con el equipo de soporte técnico de SignalFx.  

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base 64)** y, a continuación, seleccione **Descargar**. Descargue el certificado y guárdelo en el equipo. A continuación, copie el valor de **Dirección URL de metadatos de federación de aplicación**; necesitará esta información en un paso posterior en la interfaz de usuario de SignalFx. 

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar SignalFx**, copie el valor de **Identificador de Azure AD**. Necesitará esta información en un paso posterior en la interfaz de usuario de SignalFx. 

## <a name="step-4-create-an-azure-ad-test-user"></a>Paso 4: Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

## <a name="step-5-assign-the-azure-ad-test-user"></a>Paso 5: Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir que B.Simon acceda a SignalFx utilizando el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **SignalFx**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="step-6-complete-the-signalfx-sso-configuration"></a>Paso 6: Finalización de la configuración del inicio de sesión único de SignalFx 

1. Abra la pestaña anterior y vuelva a la interfaz de usuario de SignalFx para ver la página de integración de Azure Active Directory actual. 
1. Junto a **Certificate (Base64)** (Certificado [Base 64]), haga clic en **Upload File** (Cargar archivo) y, a continuación, busque el archivo de **certificado codificado en Base 64** que descargó anteriormente de Azure Portal.
1. Junto a **Azure AD Identifier** (Identificador de Azure AD), pegue el valor de **Identificador de Azure AD** que copió antes de Azure Portal. 
1. Junto a **Federation Metadata URL** (Dirección URL de metadatos de federación), pegue el valor de **Dirección URL de metadatos de federación de aplicación** que copió antes de Azure Portal. 
1. Haga clic en **Save**(Guardar).

## <a name="step-7-test-sso"></a>Paso 7: Prueba de SSO

Revise la siguiente información sobre cómo probar el inicio de sesión único, así como las expectativas para iniciar sesión en SignalFx por primera vez. 

### <a name="test-logins"></a>Prueba de los inicios de sesión

* Para probar el inicio de sesión, debe usar una ventana privada o de incógnito o bien puede cerrar sesión en Azure Portal. De no ser así, las cookies para el usuario que configuró la aplicación interfieren y evitan un inicio de sesión correcto con el usuario de prueba.

* Cuando un nuevo usuario de prueba inicia sesión por primera vez, Azure forzará un cambio de contraseña. Cuando esto ocurra, el proceso de inicio de sesión de SSO no se completará; se dirigirá al usuario de prueba a Azure Portal. Para solucionar el problema, el usuario de prueba debe cambiar su contraseña e ir a la página de inicio de sesión de SignalFx o a Aplicaciones y volver a intentarlo.
    * Al hacer clic en el icono de SignalFx en Aplicaciones, debería iniciar sesión automáticamente en la instancia de SignalFx. 
        * Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

* Se puede acceder a la aplicación SignalFx desde Aplicaciones o a mediante una página de inicio de sesión personalizada asignada a la organización. El usuario de prueba debe probar la integración a partir de cualquiera de estas ubicaciones.
    * El usuario de prueba puede usar las credenciales creadas anteriormente en este proceso para **b.simon\@contoso.com**.

### <a name="first-time-logins"></a>Inicios de sesión por primera vez

* Cuando un usuario inicia sesión en SignalFx desde el inicio de sesión único de SAML por primera vez, el usuario recibirá un correo electrónico de SignalFx con un vínculo. El usuario debe hacer clic en el vínculo para fines de autenticación. Esta validación de correo electrónico solo tendrá lugar para los usuarios por primera vez. 

* SignalFx admite la creación de usuarios **Just-in-Time**, lo que significa que si un usuario no existe en SignalFx, se creará la cuenta del usuario tras el primer intento de inicio de sesión.

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado SignalFx, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
