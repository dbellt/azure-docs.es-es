---
title: 'Incorporación de Google como proveedor de identidades para B2B: Azure AD'
description: Federación con Google para permitir a los usuarios invitados iniciar sesión en sus aplicaciones de Azure AD con sus cuentas de Gmail
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 04/06/2021
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0cb3b3fd2010bf4ad1288b767d62d3d76f7b37e3
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106550972"
---
# <a name="add-google-as-an-identity-provider-for-b2b-guest-users"></a>Incorporación de Google como proveedor de identidades para los usuarios invitados de B2B

Si configura la federación con Google, puede permitir que los usuarios invitados puedan iniciar sesión en sus aplicaciones y recursos compartidos con sus propias cuentas de Google, sin tener que crear cuentas Microsoft.

Cuando haya agregado Google como una de las opciones de inicio de sesión en la página **Iniciar sesión**, el usuario solo tendrá que escribir el correo electrónico que utiliza para iniciar sesión en Google o seleccionar **Opciones de inicio de sesión** y elegir **Iniciar sesión con Google**. En los dos casos, accederá automáticamente a la página de inicio de sesión de Google, donde tendrá que autenticarse.

![Opciones de inicio de sesión para los usuarios de Google](media/google-federation/sign-in-with-google-overview.png)

> [!NOTE]
> La federación de Google está diseñada específicamente para los usuarios de Gmail. Para realizar la federación con los dominios de G Suite, use la [federación directa](direct-federation.md).

> [!IMPORTANT]
> **A partir del 4 de enero de 2021**, Google [retira la compatibilidad con el inicio de sesión en WebView](https://developers.googleblog.com/2020/08/guidance-for-our-effort-to-block-less-secure-browser-and-apps.html). Si usa la federación de Google o el registro de autoservicio con Gmail, debería [comprobar la compatibilidad de las aplicaciones nativas de línea de negocio](google-federation.md#deprecation-of-webview-sign-in-support).

## <a name="what-is-the-experience-for-the-google-user"></a>¿Cuál es la experiencia del usuario de Google?

Cuando un usuario de Google canjee su invitación, su experiencia variará en función de si ya ha iniciado sesión o no en Google:

- Se pedirá a los usuarios invitados que no hayan iniciado sesión en Google que lo hagan.
- Se pedirá a los usuarios invitados que ya hayan iniciado sesión en Google que elijan la cuenta que desean usar. Deben elegir la cuenta donde hayan recibido la invitación.

Los usuarios invitados que ven un error "encabezado demasiado largo" pueden eliminar sus cookies o abrir una ventana privada o de incógnito e intentar iniciar sesión de nuevo.

![Captura de pantalla en la que se muestra la página de inicio de sesión de Google.](media/google-federation/google-sign-in.png)

## <a name="sign-in-endpoints"></a>Puntos de conexión de inicio de sesión

Ahora, los usuarios invitados de Google pueden iniciar sesión en sus aplicaciones multiinquilino o en las aplicaciones propias de Microsoft utilizando un [punto de conexión común](redemption-experience.md#redemption-and-sign-in-through-a-common-endpoint) (en otras palabras, una dirección URL general de la aplicación que no incluya el contexto del inquilino). Durante el proceso de inicio de sesión, el usuario invitado elige **Opciones de inicio de sesión** y después selecciona **Sign in to an organization** (Iniciar sesión en una organización). A continuación, el usuario escribe el nombre de la organización y continúa iniciando sesión con las credenciales de Google.

Los usuarios invitados de Google también pueden usar puntos de conexión de la aplicación que incluyan la información del inquilino; por ejemplo:

  * `https://myapps.microsoft.com/?tenantid=<your tenant ID>`
  * `https://myapps.microsoft.com/<your verified domain>.onmicrosoft.com`
  * `https://portal.azure.com/<your tenant ID>`

También puede proporcionar a los usuarios invitados de Google un vínculo directo a una aplicación o recurso que incluya la información del inquilino; por ejemplo, `https://myapps.microsoft.com/signin/Twitter/<application ID?tenantId=<your tenant ID>`.

## <a name="deprecation-of-webview-sign-in-support"></a>Desuso de la compatibilidad con el inicio de sesión en WebView

A partir del 4 de enero de 2021, Google [retira la compatibilidad con el inicio de sesión en WebView](https://developers.googleblog.com/2020/08/guidance-for-our-effort-to-block-less-secure-browser-and-apps.html). Si usa la federación de Google o el [registro de autoservicio con Gmail](identity-providers.md), debería comprobar la compatibilidad de las aplicaciones nativas de línea de negocio. Si las aplicaciones incluyen contenido de WebView que requiere autenticación, los usuarios de Gmail de Google no podrán autenticarse. Estos son escenarios conocidos que afectarán a los usuarios de Gmail:

- Aplicaciones de Windows que usan WebView insertado o WebAccountManager (WAM) en versiones anteriores de Windows.
- Otras aplicaciones nativas que haya desarrollado que usen un marco de explorador insertado para la autenticación.

Este cambio no afecta a:

- Aplicaciones de Windows que usan WebView insertado o WebAccountManager (WAM) en las versiones más recientes de Windows
- Aplicaciones de Microsoft iOS
- Identidades de G Suite; por ejemplo, cuando se usa la [federación directa](direct-federation.md) basada en SAML con G Suite

Seguimos probando varias plataformas y escenarios y actualizaremos este artículo en consecuencia.
### <a name="to-test-your-apps-for-compatibility"></a>Para probar la compatibilidad de las aplicaciones:

1. Siga la [guía de Google](https://developers.googleblog.com/2020/08/guidance-for-our-effort-to-block-less-secure-browser-and-apps.html) para determinar si sus aplicaciones se verán afectadas por este cambio.
2. Con Fiddler u otra herramienta de prueba, inserte un encabezado durante el inicio de sesión y use una identidad externa de Google para probar el inicio de sesión:

   1. Agregue Google-Accounts-Check-OAuth-Login:true a sus encabezados de solicitud HTTP cuando se envíen las solicitudes a accounts.google.com.
   1. Intente iniciar sesión en la aplicación escribiendo una dirección de Gmail en la página de inicio de sesión de accounts.google.com.
   1. Si se produce un error de inicio de sesión y ve un error que dice que es posible que el explorador o la aplicación no sean seguros, se bloqueará el inicio de sesión para las identidades externas de Google.

3. Para resolver este problema, haga lo siguiente:

   - Si la aplicación de Windows usa WebView insertado o WebAccountManager (WAM) en una versión anterior de Windows, actualice a la versión más reciente de esta plataforma.
   - Modifique sus aplicaciones para que usen el explorador del sistema para el inicio de sesión. Para más información, consulte [Interfaz de usuario web del sistema frente a insertada](../develop/msal-net-web-browsers.md#embedded-vs-system-web-ui) en la documentación de MSAL.NET.  


## <a name="step-1-configure-a-google-developer-project"></a>Paso 1: configuración de un proyecto de desarrollador de Google
En primer lugar, cree un proyecto en la consola de desarrolladores de Google para obtener un identificador y un secreto de cliente que pueda agregar después a Azure Active Directory (Azure AD). 
1. Vaya a las API de Google de https://console.developers.google.com e inicie sesión con su cuenta de Google. Se recomienda utilizar una cuenta de Google compartida con el equipo.
2. Si se le solicita, acepte los términos del servicio.
3. Cree un nuevo proyecto: en la esquina superior izquierda de la página, seleccione la lista de proyectos y, en la página **Seleccionar un proyecto**, haga clic en **Nuevo proyecto**.
4. En la página **Nuevo proyecto**, escriba un nombre para el proyecto (por ejemplo, **Azure AD B2B**) y seleccione **Crear**: 
   
   ![Captura en la que se muestra una página Nuevo proyecto.](media/google-federation/google-new-project.png)

4. En la página **API y servicios**, seleccione **Ver** en el nuevo proyecto.

5. Seleccione **Go to APIs overview** (Ir a la información general de las API) en la tarjeta de API. Seleccione **OAuth consent screen** (Pantalla de consentimiento de OAuth).

6. Seleccione **Externo** y después **Crear**. 

7. En la **Pantalla de consentimiento de OAuth**, especifique un **nombre de aplicación**:

   ![Captura de pantalla en la que se muestra la pantalla de consentimiento de OAuth de Google.](media/google-federation/google-oauth-consent-screen.png)

8. Desplácese hasta la sección **Dominios autorizados** y escriba **microsoftonline.com**:

   ![Captura de pantalla en la que se muestra la sección de dominios autorizados.](media/google-federation/google-oauth-authorized-domains.PNG)

9. Seleccione **Guardar**.

10. Seleccione **Credenciales**. En el menú **Crear credenciales**, seleccione **Id. del cliente de OAuth**.

    ![Captura de pantalla en la que se muestra el menú Crear credenciales de las API de Google.](media/google-federation/google-api-credentials.png)

11. En **Application type** (Tipo de aplicación), seleccione **Web application** (Aplicación web). Asigne un nombre adecuado a la aplicación, como **Azure AD B2B**. En **URI de redirección autorizados**, escriba los siguientes URI:
    - `https://login.microsoftonline.com`
    - `https://login.microsoftonline.com/te/<tenant ID>/oauth2/authresp` <br>(donde `<tenant ID>` es el id. de inquilino)
   
    > [!NOTE]
    > Para buscar el identificador de inquilino, vaya a [Azure Portal](https://portal.azure.com). En **Azure Active Directory**, seleccione **Propiedades** y copie el **Id. del inquilino**.

    ![Captura de pantalla en la que se muestra la sección de identificadores URI de redirección autorizados.](media/google-federation/google-create-oauth-client-id.png)

12. Seleccione **Crear**. Copie el identificador de cliente y del secreto de cliente. Los usará cuando agregue el proveedor de identidades en Azure Portal.

    ![Captura de pantalla en la que se muestra el Id. de cliente de OAuth y el secreto de cliente.](media/google-federation/google-auth-client-id-secret.png)

## <a name="step-2-configure-google-federation-in-azure-ad"></a>Paso 2: configuración de la federación de Google en Azure AD 
Defina ahora el identificador y el secreto de cliente de Google. Para ello, puede usar Azure Portal o PowerShell. No olvide probar la configuración de la federación de Google; para ello, invítese a usted mismo. Use una dirección de Gmail e intente canjear la invitación con su cuenta de Google invitada. 

**Para configurar la federación de Google en Azure Portal** 
1. Vaya a [Azure Portal](https://portal.azure.com). En el panel izquierdo, seleccione **Azure Active Directory**. 
2. Seleccione **External Identities**.
3. Seleccione **Todos los proveedores de identidades** y haga clic en el botón de **Google**.
4. Escriba el identificador y el secreto de cliente que obtuvo anteriormente. Seleccione **Guardar**: 

   ![Captura de pantalla en la que se muestra la página para agregar un proveedor de identidades de Google.](media/google-federation/google-identity-provider.png)

**Configuración de la federación de Google con PowerShell**
1. Instale la versión más reciente de Azure AD PowerShell para el módulo Graph ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview)).
2. Ejecute este comando: `Connect-AzureAD`
3. En el símbolo del sistema, inicie sesión con la cuenta de administrador global administrada.  
4. Ejecute el siguiente comando: 
   
   `New-AzureADMSIdentityProvider -Type Google -Name Google -ClientId <client ID> -ClientSecret <client secret>`
 
   > [!NOTE]
   > Use el identificador y secreto de cliente de la aplicación que creó en el "Paso 1: configuración de un proyecto de desarrollador de Google". Para más información, vea [New-AzureADMSIdentityProvider](/powershell/module/azuread/new-azureadmsidentityprovider?view=azureadps-2.0-preview&preserve-view=true). 
 
## <a name="how-do-i-remove-google-federation"></a>¿Cómo se quita la federación de Google?
La configuración de la federación de Google se puede eliminar. Si lo hace, los usuarios invitados de Google que ya hayan canjeado su invitación no podrán iniciar sesión. Sin embargo, puede concederles acceso de nuevo a sus recursos al [restablecer el estado de canje](reset-redemption-status.md).
 
**Para eliminar la federación de Google desde el portal de Azure AD**
1. Vaya a [Azure Portal](https://portal.azure.com). En el panel izquierdo, seleccione **Azure Active Directory**. 
2. Seleccione **External Identities**.
3. Seleccione **Todos los proveedores de identidades**.
4. En la línea de **Google**, seleccione el botón de puntos suspensivos ( **...** ) y, a continuación, seleccione **Eliminar**. 
   
   ![Captura de pantalla en la que se muestra el botón de eliminación del proveedor de identidades sociales.](media/google-federation/google-social-identity-providers.png)

1. Seleccione **Sí** para confirmar la eliminación. 

**Para eliminar la federación de Google mediante PowerShell** 
1. Instale la versión más reciente de Azure AD PowerShell para el módulo Graph ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview)).
2. Ejecute `Connect-AzureAD`.  
4. En el símbolo del sistema, inicie sesión con la cuenta de administrador global administrada.  
5. Escriba el comando siguiente:

    `Remove-AzureADMSIdentityProvider -Id Google-OAUTH`

   > [!NOTE]
   > Para más información, consulte [Remove-AzureADMSIdentityProvider](/powershell/module/azuread/Remove-AzureADMSIdentityProvider?view=azureadps-2.0-preview&preserve-view=true).
