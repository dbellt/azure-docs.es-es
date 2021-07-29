---
title: Habilitación de las opciones de una aplicación web mediante Azure Active Directory B2C
description: Habilite el uso de las opciones de una aplicación web de varias maneras.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 05/25/2021
ms.author: mimart
ms.subservice: B2C
ms.custom: b2c-support
ms.openlocfilehash: 3335e035a2d36cc7830d8bc93db82a7d318d26b3
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110482853"
---
# <a name="configure-authentication-in-a-sample-web-application-using-azure-active-directory-b2c-options"></a>Configuración de la autenticación en una aplicación web de ejemplo mediante opciones de Azure Active Directory B2C

En este artículo se describen métodos para personalizar y mejorar la experiencia de autenticación de Azure Active Directory B2C (Azure AD B2C) para una aplicación web. Antes de empezar, familiarícese con los siguientes artículos: [Configuración de la autenticación en una aplicación web de ejemplo](configure-authentication-sample-web-app.md) o [Habilitación de la autenticación en su propia aplicación web](enable-authentication-web-application.md).

## <a name="use-a-custom-domain"></a>Uso de un dominio personalizado

El uso de un [dominio personalizado](custom-domain.md) en la URL de redireccionamiento de su aplicación proporciona una experiencia de usuario más fluida. Desde la perspectiva del usuario, permanece en el dominio durante el proceso de inicio de sesión, en lugar de redirigirse al dominio predeterminado de Azure AD B2C, que es .b2clogin.com.

Para usar un dominio personalizado, siga las instrucciones del artículo [Habilitación de dominios personalizados](custom-domain.md). En la carpeta raíz del proyecto, abra el archivo `appsettings.json`. Este archivo contiene información sobre el proveedor de identidades de Azure AD B2C. Actualice la entrada `Instance` con el dominio personalizado.

El siguiente JSON muestra la configuración de la aplicación antes del cambio: 

```JSon
"AzureAdB2C": {
  "Instance": "https://contoso.b2clogin.com",
  ...
}
```  

El siguiente JSON muestra la configuración de la aplicación después del cambio: 

```JSon
"AzureAdB2C": {
  "Instance": "https://login.contoso.com",
  ...
}
``` 

## <a name="use-your-tenant-id"></a>Uso del id. del inquilino

Puede reemplazar el nombre del inquilino de B2C en la dirección URL por el GUID del identificador de inquilino a fin de quitar todas las referencias a "b2c" en la dirección URL.  Por ejemplo, puede cambiar `https://account.contosobank.co.uk/contosobank.onmicrosoft.com/` por `https://account.contosobank.co.uk/<tenant ID GUID>/`.

Para usar el identificador de inquilino, siga las instrucciones del artículo [Habilitación de dominios personalizados](custom-domain.md#optional-use-tenant-id). En la carpeta raíz del proyecto, abra el archivo `appsettings.json`. Este archivo contiene información sobre el proveedor de identidades de Azure AD B2C. Actualice la entrada `Domain` con el dominio personalizado.

El siguiente JSON demuestra la configuración de la aplicación antes del cambio: 

```JSon
"AzureAdB2C": {
  "Domain": "tenant-name.onmicrosoft.com",
  ...
}
```  

El siguiente JSON demuestra la configuración de la aplicación después del cambio:

```JSon
"AzureAdB2C": {
  "Domain": "00000000-0000-0000-0000-000000000000",
  ...
}
``` 

## <a name="support-advanced-scenarios"></a>Compatibilidad con escenarios avanzados

El método `AddMicrosoftIdentityWebAppAuthentication` de la API de la plataforma de identidad de Microsoft permite a los desarrolladores agregar código para escenarios de autenticación avanzada o suscribirse a eventos de OpenIDConnect. Por ejemplo, puede suscribirse a OnRedirectToIdentityProvider, que le permite personalizar la solicitud de autenticación que la aplicación envía a Azure AD B2C.

Para admitir escenarios avanzados, abra el archivo `Startup.cs` y, en la función `ConfigureServices`, reemplace `AddMicrosoftIdentityWebAppAuthentication` por el siguiente fragmento de código: 

```csharp
// Configuration to sign-in users with Azure AD B2C

//services.AddMicrosoftIdentityWebAppAuthentication(Configuration, "AzureAdB2C");

services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
        .AddMicrosoftIdentityWebApp(options =>
{
    Configuration.Bind("AzureAdB2C", options);
    options.Events ??= new OpenIdConnectEvents();
    options.Events.OnRedirectToIdentityProvider += OnRedirectToIdentityProviderFunc;
});
```

El código anterior agrega el evento OnRedirectToIdentityProvider con una referencia al método *OnRedirectToIdentityProviderFunc*. Agregue el siguiente fragmento de código a la clase `Startup.cs`.

```csharp
private async Task OnRedirectToIdentityProviderFunc(RedirectContext context)
{
    // Custom code here
    
    // Don't remove this line
    await Task.CompletedTask.ConfigureAwait(false);
}
```

Puede pasar parámetros entre el controlador y la función *OnRedirectToIdentityProvider* mediante parámetros de contexto. 


## <a name="prepopulate-the-sign-in-name"></a>Rellenar previamente el nombre de inicio de sesión

Durante el recorrido de inicio de sesión de un usuario, la aplicación puede tener como destino un usuario específico. Cuando el destino es un usuario, una aplicación puede especificar, en la solicitud de autorización, el parámetro de consulta `login_hint` con el nombre de inicio de sesión de usuario. Azure AD B2C rellena automáticamente el nombre de inicio de sesión y el usuario solo necesita proporcionar la contraseña. 

Para rellenar previamente el nombre de inicio de sesión, siga estos pasos:

1. Complete el procedimiento [Compatibilidad con escenarios avanzados](#support-advanced-scenarios).
1. Si usa una directiva personalizada, agregue la notificación de entrada necesaria como se describe en [Configuración del inicio de sesión directo](direct-signin.md#prepopulate-the-sign-in-name). 
1. Agregue la siguiente línea de código a la función *OnRedirectToIdentityProvider*:
    
    ```csharp
    private async Task OnRedirectToIdentityProviderFunc(RedirectContext context)
    {
      context.ProtocolMessage.LoginHint = "emily@contoso.com";
      
      // More code
      await Task.CompletedTask.ConfigureAwait(false);
    }
    ```

## <a name="redirect-sign-in-to-an-external-identity-provider"></a>Redirección del inicio de sesión a un proveedor de identidades externo

Si se ha configurado el recorrido de inicio de sesión para que la aplicación incluya cuentas de redes sociales como Facebook, LinkedIn o Google, puede especificar el parámetro `domain_hint`. Este parámetro de consulta proporciona una sugerencia a Azure AD B2C acerca del proveedor de identidades sociales que debe usarse para iniciar sesión. Por ejemplo, si la aplicación especifica `domain_hint=facebook.com`, el flujo de inicio de sesión va directamente a la página de inicio de sesión de Facebook. 

Para redirigir el inicio de sesión a un proveedor de identidades externo, siga estos pasos:

1. Complete el procedimiento [Compatibilidad con escenarios avanzados](#support-advanced-scenarios).
1. Compruebe el nombre de dominio del proveedor de identidades externo. Para más información, consulte [Redirección del inicio de sesión a un proveedor social](direct-signin.md#redirect-sign-in-to-a-social-provider). 
1. En la función *OnRedirectToIdentityProviderFunc*, agregue la siguiente línea de código a la función *OnRedirectToIdentityProvider*:
    
    ```csharp
    private async Task OnRedirectToIdentityProviderFunc(RedirectContext context)
    {
      context.ProtocolMessage.DomainHint = "facebook.com";
      
      // More code
      await Task.CompletedTask.ConfigureAwait(false);
    }
    ```

## <a name="specify-the-ui-language"></a>Especificación del idioma de la interfaz de usuario

La personalización del idioma en Azure AD B2C permite que el flujo de usuario albergue distintos idiomas a fin de satisfacer las necesidades de los clientes. Para obtener más información, consulte [Personalización de idioma](language-customization.md).

Para establecer el idioma preferido, siga estos pasos:

1. Complete el procedimiento [Compatibilidad con escenarios avanzados](#support-advanced-scenarios).
1. Agregue la siguiente línea de código a la función *OnRedirectToIdentityProvider*:

    ```csharp
    private async Task OnRedirectToIdentityProviderFunc(RedirectContext context)
    {
      context.ProtocolMessage.UiLocales = "es";

      // More code
      await Task.CompletedTask.ConfigureAwait(false);
    }
    ```

## <a name="pass-a-custom-query-string-parameter"></a>Paso de un parámetro de cadena de consulta personalizado

Con las directivas personalizadas puede pasar un parámetro de cadena de consulta personalizado. Por ejemplo, cuando desea [cambiar dinámicamente el contenido de la página](customize-ui-with-html.md?pivots=b2c-custom-policy#configure-dynamic-custom-page-content-uri).


Para pasar un parámetro de cadena de consulta personalizado, siga estos pasos:

1. Complete el procedimiento [Compatibilidad con escenarios avanzados](#support-advanced-scenarios).
1. Agregue la siguiente línea de código a la función *OnRedirectToIdentityProvider*:
    
    ```csharp
    private async Task OnRedirectToIdentityProviderFunc(RedirectContext context)
    {
      context.ProtocolMessage.Parameters.Add("campaignId", "123");

      // More code
      await Task.CompletedTask.ConfigureAwait(false);
    }
    ```

## <a name="pass-id-token-hint"></a>Paso de una sugerencia de token de identificador

Azure AD B2C permite a las aplicaciones de usuario de confianza enviar un token JWT de entrada como parte de la solicitud de autorización de OAuth2. El token JWT se puede emitir mediante una aplicación de usuario de confianza o un proveedor de identidades, y puede pasar una sugerencia sobre el usuario o la solicitud de autorización. Azure AD B2C valida la firma, el nombre del emisor y la audiencia del token, y extrae la notificación del token de entrada.

Para incluir una sugerencia de token de identificador en la solicitud de autenticación, siga estos pasos: 

1. Complete el procedimiento [Compatibilidad con escenarios avanzados](#support-advanced-scenarios).
1. En la directiva personalizada, defina un [perfil técnico de sugerencias de token de identificador](id-token-hint.md).
1. Agregue la siguiente línea de código a la función *OnRedirectToIdentityProvider*:
    
    ```csharp
    private async Task OnRedirectToIdentityProviderFunc(RedirectContext context)
    {
      // The idTokenHint variable holds your ID token 
      context.ProtocolMessage.IdTokenHint = idTokenHint

      // More code
      await Task.CompletedTask.ConfigureAwait(false);
    }
    ```
    
## <a name="account-controller"></a>Controlador de la cuenta

Si quiere personalizar las acciones de **inicio de sesión**, **registro** o **cierre de sesión**, se recomienda crear su propio controlador. Tener su propio controlador le permite pasar parámetros entre el controlador y la biblioteca de autenticación. `AccountController` forma parte del paquete NuGet `Microsoft.Identity.Web.UI`, que controla las acciones de inicio y cierre de sesión. Puede encontrar su implementación en la [biblioteca web de identidad de Microsoft](https://github.com/AzureAD/microsoft-identity-web/blob/master/src/Microsoft.Identity.Web.UI/Areas/MicrosoftIdentity/Controllers/AccountController.cs). 

El siguiente fragmento de código muestra un `MyAccountController` personalizado con la acción **SignIn**. La acción pasa un parámetro denominado `campaign_id` a la biblioteca de autenticación.

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.OpenIdConnect;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;


namespace mywebapp.Controllers
{
    [AllowAnonymous]
    [Area("MicrosoftIdentity")]
    [Route("[area]/[controller]/[action]")]
    public class MyAccountController : Controller
    {

        [HttpGet("{scheme?}")]
        public IActionResult SignIn([FromRoute] string scheme)
        {
            scheme ??= OpenIdConnectDefaults.AuthenticationScheme;
            var redirectUrl = Url.Content("~/");
            var properties = new AuthenticationProperties { RedirectUri = redirectUrl };
            properties.Items["campaign_id"] = "1234";
            return Challenge(properties, scheme);
        }

    }
}

```

En la vista `_LoginPartial.cshtml`, cambie el vínculo de inicio de sesión al controlador.

```
<form method="get" asp-area="MicrosoftIdentity" asp-controller="MyAccount" asp-action="SignIn">
```

En la función `OnRedirectToIdentityProvider`, en las llamadas `Startup.cs`, puede leer el parámetro personalizado:

```csharp
private async Task OnRedirectToIdentityProviderFunc(RedirectContext context)
{
    // Read the custom parameter
    var campaign_id = (context.Properties.Items.ContainsKey("campaign_id"))
    
    // Add your custom code here
    
    await Task.CompletedTask.ConfigureAwait(false);
}
```

## <a name="role-based-access-control"></a>Control de acceso basado en rol

Con la [autorización en ASP.NET Core](/aspnet/core/security/authorization/introduction), puede usar la [autorización basada en roles](/aspnet/core/security/authorization/roles), la [autorización basada en notificaciones](/aspnet/core/security/authorization/claims) o la [autorización basada en directivas](/aspnet/core/security/authorization/policies) para comprobar si el usuario está autorizado para acceder a un recurso protegido.

En el método *ConfigureServices*, agregue el método *AddAuthorization*, que agrega el modelo de autorización. En el ejemplo siguiente se crea una directiva llamada `EmployeeOnly`. La directiva comprueba que existe una notificación `EmployeeNumber`. El valor de la notificación debe ser uno de los siguientes identificadores: 1, 2, 3, 4 o 5.

```csharp
services.AddAuthorization(options =>
    {
        options.AddPolicy("EmployeeOnly", policy =>
              policy.RequireClaim("EmployeeNumber", "1", "2", "3", "4", "5"));
    });
```

La autorización en ASP.NET Core se controla con [AuthorizeAttribute](/aspnet/core/security/authorization/simple) y sus distintos parámetros. En su forma más básica, la aplicación del atributo `[Authorize]` a un controlador, una acción o una Razor Page limita el acceso para los usuarios autenticados de ese componente.

Las directivas se aplican a los controladores mediante el atributo `[Authorize]` con el nombre de directiva. El código siguiente limita el acceso a la acción `Claims` para los usuarios autorizados por la directiva `EmployeeOnly`:

```csharp
[Authorize(Policy = "EmployeeOnly")]
public IActionResult Claims()
{
    return View();
}
```

## <a name="next-steps"></a>Pasos siguientes

- Más información: [Introducción a la autorización en ASP.NET Core](/aspnet/core/security/authorization/introduction)