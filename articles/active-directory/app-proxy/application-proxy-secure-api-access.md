---
title: Acceso a las API en el entorno local con Azure Active Directory Application Proxy
description: Azure Active Directory Application Proxy permite a las aplicaciones nativas acceder de forma segura a las API y la lógica de negocios que hospeda en el entorno local o en las VM en la nube.
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-proxy
ms.workload: identity
ms.topic: how-to
ms.date: 04/27/2021
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: 660d76d03db3b3d918ad616da9d6ab7b4205860d
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108186766"
---
# <a name="secure-access-to-on-premises-apis-with-azure-active-directory-application-proxy"></a>Acceso seguro a las API en el entorno local con Azure Active Directory Application Proxy

Puede tener API de lógica de negocios ejecutándose en el entorno local u hospedadas en máquinas virtuales en la nube. Las aplicaciones nativas de Android, iOS, Mac o Windows necesitan interactuar con los puntos de conexión de la API para usar datos o proporcionar la interacción del usuario. Azure AD Application Proxy y la [bibliotecas de autenticación de Microsoft (MSAL)](../azuread-dev/active-directory-authentication-libraries.md) permiten a las aplicaciones nativas acceder de forma segura a las API en el entorno local. Azure Active Directory Application Proxy es una solución más rápida y segura que abrir puertos de firewall y controlar la autenticación y autorización en el nivel de aplicación.

En este artículo, se le guiará a lo largo de la configuración de una solución de Azure AD Application Proxy para hospedar un servicio de API web al que puedan acceder las aplicaciones nativas.

## <a name="overview"></a>Información general

En el siguiente diagrama se muestra una forma tradicional de publicar API en el entorno local. Este enfoque requiere abrir los puertos de entrada 80 y 443.

![Acceso a la API tradicional](./media/application-proxy-secure-api-access/overview-publish-api-open-ports.png)

En el diagrama siguiente se muestra cómo puede usar Azure AD Application Proxy para publicar API de forma segura sin tener que abrir ningún puerto de entrada:

![Acceso a API de Azure AD Application Proxy](./media/application-proxy-secure-api-access/overview-publish-api-app-proxy.png)

Azure AD Application Proxy es la base de la solución, funciona como punto de conexión público para el acceso a API y proporciona autenticación y autorización. Puede acceder a las API desde una amplia variedad de plataformas mediante las bibliotecas de tipo [Biblioteca de autenticación de Microsoft (MSAL)](../azuread-dev/active-directory-authentication-libraries.md).

Puesto que la autorización y autenticación de Azure AD Application Proxy se basan en Azure AD, puede usar el acceso condicional de Azure AD para asegurarse de que solo los dispositivos de confianza puedan tener acceso a las API publicadas mediante Application Proxy. Use Unión a Azure AD o Unión a Azure AD híbrido para escritorios e Intune Managed para dispositivos. También puede aprovechar las características de Azure Active Directory Premium, como Azure AD Multi-Factor Authentication, y la seguridad basada en el aprendizaje automático de [Azure Identity Protection](../identity-protection/overview-identity-protection.md).

## <a name="prerequisites"></a>Requisitos previos

Para seguir este tutorial, necesita:

- Acceso de administrador a un directorio de Azure con una cuenta que pueda crear y registrar aplicaciones
- La API web de ejemplo y aplicaciones de cliente nativas de [https://github.com/jeevanbisht/API-NativeApp-ADAL-SampleApp](https://github.com/jeevanbisht/API-NativeApp-ADAL-SampleApp)

## <a name="publish-the-api-through-application-proxy"></a>Publicar la API mediante Application Proxy

Para publicar una API fuera de la intranet mediante Application Proxy, se sigue el mismo patrón que para publicar aplicaciones web. Para más información, consulte el [Tutorial: Adición de una aplicación local para el acceso remoto mediante Application Proxy en Azure Active Directory](application-proxy-add-on-premises-application.md).

Para publicar la API web SecretAPI mediante Application Proxy:

1. Compile y publique el proyecto de ejemplo SecretAPI como aplicación web ASP.NET en el equipo local o la intranet. Asegúrese de que puede acceder a la aplicación web localmente.

1. En [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**. Después, seleccione **Aplicaciones empresariales**.

1. En la parte superior de la página **Aplicaciones empresariales - Todas las aplicaciones, seleccione** **Nueva aplicación**.

1. En la página **Agregar una aplicación**, seleccione **Aplicaciones locales**. Se abre la página **Agregar aplicación local propia**.

1. Si no tiene instalado un conector de Application Proxy, se le pedirá que lo instale. Seleccione **Descargar conector de Application Proxy** para descargar el conector e instalarlo.

1. Una vez instalado el conector de Application Proxy, en la página **Agregar aplicación local propia**:

   1. Junto a **Nombre**, escriba *SecretAPI*.

   1. Junto a **Dirección URL interna**, escriba la dirección URL que usa para acceder a la API desde la intranet.

   1. Asegúrese de que la **Autenticación previa** está establecida en **Azure Active Directory**.

   1. Seleccione **Agregar** en la parte superior de la página y espere a que la aplicación se cree.

   ![Agregar una aplicación de API](./media/application-proxy-secure-api-access/3-add-api-app.png)

1. En la página **Aplicaciones empresariales - Todas las aplicaciones**, seleccione la aplicación **SecretAPI**.

1. En la página **SecretAPI - Introducción**, seleccione **Propiedades** en el panel de navegación izquierdo.

1. No es recomendable que las API estén disponibles para los usuarios finales en el panel **Mis aplicaciones**, así que establezca **¿Es visible para los usuarios?** en **No** en la parte inferior de la página **Propiedades** y, a continuación, seleccione **Guardar**.

   ![No visible para los usuarios](./media/application-proxy-secure-api-access/5-not-visible-to-users.png)

Ha publicado la API web mediante Azure AD Application Proxy. Ahora, agregue los usuarios que pueden acceder a la aplicación.

1. En la página **SecretAPI - Introducción**, seleccione **Usuarios y grupos** en el panel de navegación izquierdo.

1. En la página **Usuarios y grupos**, seleccione **Agregar usuario**.

1. Después, en la página **Agregar asignación**, seleccione **Usuarios y grupos**.

1. En la página **Usuarios y grupos**, busque y seleccione los usuarios que pueden acceder a la aplicación, incluido, al menos, usted mismo. Después de seleccionar todos los usuarios, elija **Seleccionar**.

   ![Seleccionar y asignar el usuario](./media/application-proxy-secure-api-access/7-select-admin-user.png)

1. En la página **Agregar asignación**, seleccione **Asignar**.

> [!NOTE]
> Las API que usan la autenticación integrada de Windows pueden requerir [pasos adicionales](./application-proxy-configure-single-sign-on-with-kcd.md).

## <a name="register-the-native-app-and-grant-access-to-the-api"></a>Registrar la aplicación nativa y conceder acceso a la API

Las aplicaciones nativas son programas desarrollados para usarlos en un dispositivo o plataforma concreta. Para que la aplicación nativa pueda conectarse y acceder a una API, debe registrarla en Azure AD. Los pasos siguientes muestran cómo registrar una aplicación nativa y darle acceso a la API web publicada mediante Application Proxy.

Para registrar la aplicación nativa AppProxyNativeAppSample:

1. En la página **Introducción** de Azure Active Directory , seleccione **Registros de aplicaciones** y, en la parte superior del panel **Registros de aplicaciones**, seleccione **Nuevo registro** .

1. En la página **Registrar una aplicación**:

   1. En **Nombre**, escriba *AppProxyNativeAppSample*.

   1. En **Supported account types** (Tipos de cuenta compatibles), seleccione **Accounts in any organizational directory and personal Microsoft accounts** (Cuentas en cualquier directorio de organización y cuentas personales de Microsoft).

   1. En **URL de redireccionamiento**, despliegue y seleccione **Cliente público (móvil y escritorio)** y, a continuación, escriba *https://login.microsoftonline.com/common/oauth2/nativeclient* .

   1. Seleccione **Registrar** y espere a que la aplicación se registre correctamente.

      ![Nuevo registro de aplicaciones](./media/application-proxy-secure-api-access/8-create-reg-ga.png)

Ya ha registrado la aplicación AppProxyNativeAppSample en Azure Active Directory. Para dar a la aplicación nativa acceso a la API web SecretAPI:

1. En la página **Introducción** > **Registros de aplicaciones** de Azure Active Directory, seleccione la aplicación **AppProxyNativeAppSample**.

1. En la página **AppProxyNativeAppSample**, seleccione **Permisos de API** en el panel de navegación izquierdo.

1. En la página **Permisos de API**, seleccione **Agregar un permiso**.

1. En la primera página **Solicitud de permisos de API**, seleccione la pestaña **API usadas en mi organización** y, a continuación, busque y seleccione **SecretAPI**.

1. En la siguiente página **Solicitud de permisos de API**, seleccione la casilla situada junto a **user_impersonation** y, a continuación, seleccione **Agregar permisos**.

    ![Selección de una API](./media/application-proxy-secure-api-access/10-secretapi-added.png)

1. En la página **Permisos de API**, puede seleccionar **Conceder consentimiento de administrador para Contoso** para impedir que otros usuarios tengan que consentir individualmente la aplicación.

## <a name="configure-the-native-app-code"></a>Configurar el código de aplicación nativo

El último paso es configurar la aplicación nativa. El siguiente fragmento de código del archivo *Form1.cs* de la aplicación de ejemplo NativeClient hace que la biblioteca MSAL adquiera el token para solicitar la llamada API y que lo adjunte como portador al encabezado de la aplicación.

   ```
   // Acquire Access Token from AAD for Proxy Application
 IPublicClientApplication clientApp = PublicClientApplicationBuilder
.Create(<App ID of the Native app>)
.WithDefaultRedirectUri() // will automatically use the default Uri for native app
.WithAuthority("https://login.microsoftonline.com/{<Tenant ID>}")
.Build();

AuthenticationResult authResult = null;
var accounts = await clientApp.GetAccountsAsync();
IAccount account = accounts.FirstOrDefault();

IEnumerable<string> scopes = new string[] {"<Scope>"};

try
 {
    authResult = await clientApp.AcquireTokenSilent(scopes, account).ExecuteAsync();
 }
    catch (MsalUiRequiredException ex)
 {
     authResult = await clientApp.AcquireTokenInteractive(scopes).ExecuteAsync();                
 }
 
if (authResult != null)
 {
  //Use the Access Token to access the Proxy Application
  
  HttpClient httpClient = new HttpClient();
  HttpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", authResult.AccessToken);
  HttpResponseMessage response = await httpClient.GetAsync("<Proxy App Url>");
 }
```

Para configurar la aplicación nativa para que se conecte a Azure Active Directory y llame a la instancia de Application Proxy de la API, actualice los valores de marcador del archivo *App.config* de la aplicación de ejemplo NativeClient con valores de Azure AD:

- Pegue el **Id. de directorio (inquilino)** en el campo `<add key="ida:Tenant" value="" />`. Puede buscar y copiar este valor (GUID) en la página **Introducción** de cualquiera de sus aplicaciones.

- Pegue el **Id. de aplicación (cliente)** en el campo `<add key="ida:ClientId" value="" />`. Puede buscar y copiar este valor (GUID) desde la página **Introducción** de AppProxyNativeAppSample.

- Pegue el **URI de redireccionamiento** en el campo `<add key="ida:RedirectUri" value="" />`. Puede buscar y copiar este valor (URI) desde la página **Autenticación** de AppProxyNativeAppSample.

- Pegue el **URI de id. de aplicación** de SecretAPI en el campo `<add key="todo:TodoListResourceId" value="" />`. Puede buscar y copiar este valor (URI) en la página **Exponer una API**.

- Pegue la **URL de página principal** de SecretAPI en el campo `<add key="todo:TodoListBaseAddress" value="" />`. Puede buscar y copiar este valor (URL) en la página **Personalización de marca** de SecretAPI.

Después de configurar los parámetros, compile y ejecute la aplicación nativa. Al seleccionar el botón **Iniciar sesión**, la aplicación permite iniciar sesión y, a continuación, muestra una pantalla para confirmar que se conectó correctamente a SecretAPI.

![Captura de pantalla que muestra un mensaje de secreto de A P I correcto y el botón Aceptar.](./media/application-proxy-secure-api-access/success.png)

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Adición de una aplicación local para el acceso remoto mediante el proxy de aplicación en Azure Active Directory](application-proxy-add-on-premises-application.md)
- [Inicio rápido: Configuración de una aplicación cliente para tener acceso a las API web](../develop/quickstart-configure-app-access-web-apis.md)
- [Habilitación de las aplicaciones cliente nativas para interactuar con el proxy de aplicaciones](application-proxy-configure-native-client-application.md)