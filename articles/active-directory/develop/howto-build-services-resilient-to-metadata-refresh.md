---
title: 'Procedimiento: Crear servicios resistentes a la actualización de metadatos de OpenID Connect de Azure AD | Azure'
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo asegurarse de que la aplicación web o la API web sean resistentes a la actualización de metadatos de OpenID Connect de Azure AD.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: how-to
ms.date: 04/21/2021
ms.author: jmprieur
ms.reviewer: marsma, shermanouko
ms.custom: aaddev
ms.openlocfilehash: 8eb9db4d89f6d3c517afb57d6343387343ac5325
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108167133"
---
# <a name="how-to-build-services-that-are-resilient-to-azure-ads-openid-connect-metadata-refresh"></a>Procedimiento: Crear servicios resistentes a la actualización de metadatos de OpenID Connect de Azure AD

Las API web protegidas deben validar los tokens de acceso. Las aplicaciones web también validan los tokens de identificador. La validación de tokens tiene varias partes, y se comprueba si el token pertenece a la aplicación, si lo ha emitido un proveedor de identidades (IDP) de confianza, si su duración todavía está en el intervalo y si no se ha alterado. También puede haber validaciones especiales. Por ejemplo, la aplicación debe validar la firma y comprobar que las claves de firma (cuando se insertan en un token) sean de confianza y que el token no se esté reproduciendo. Cuando las claves de firma no se insertan en el token, deben capturarse desde el proveedor de identidades (detección o metadatos). A veces también es necesario obtener las claves de forma dinámica en tiempo de ejecución.

Las aplicaciones web y las API web deben actualizar los metadatos de OpenID Connect obsoletos para que sean resistentes. Este artículo proporciona una guía para lograr aplicaciones resistentes. Se aplica a ASP.NET Core, ASP.NET clásico y Microsoft.IdentityModel.

## <a name="aspnet-core"></a>ASP.NET Core

Use la versión más reciente de [Microsoft.IdentityModel.*](https://www.nuget.org/packages?q=Microsoft.IdentityModel) y siga manualmente las instrucciones siguientes.

En el método `ConfigureServices` de Startup.cs, asegúrese de que `JwtBearerOptions.RefreshOnIssuerKeyNotFound` está establecido en true y de que usa la biblioteca de Microsoft.IdentityModel.* más reciente. Esta propiedad debe estar habilitada de forma predeterminada.

```csharp
  services.Configure<JwtBearerOptions>(AzureADDefaults.JwtBearerAuthenticationScheme, options =>
  {
    …
    // shouldn’t be necessary as it’s true by default
    options.RefreshOnIssuerKeyNotFound = true;
    …
   };
```

## <a name="aspnet-owin"></a>OWIN de ASP.NET/

Microsoft recomienda pasar a ASP.NET Core, ya que el desarrollo se ha detenido en ASP.NET. 

Si usa ASP.NET clásico, use la versión más reciente de [Microsoft.IdentityModel.*](https://www.nuget.org/packages?q=Microsoft.IdentityModel).

OWIN tiene un intervalo de actualización automática de 24 horas para el objeto `OpenIdConnectConfiguration`. Esta actualización solo se desencadenará si se recibe una solicitud una vez transcurrido el intervalo de tiempo de 24 horas. Por lo que sabemos, no hay ninguna manera de cambiar este valor o desencadenar una actualización anticipada, aparte de reiniciar la aplicación.

## <a name="microsoftidentitymodel"></a>Microsoft.IdentityModel

Si valida el token usted mismo, por ejemplo, en una instancia de Azure Functions, use la versión más reciente de [Microsoft.IdentityModel.*](https://www.nuget.org/packages?q=Microsoft.IdentityModel) y siga las instrucciones de metadatos que se muestran en los fragmentos de código siguientes.

```csharp
ConfigurationManager<OpenIdConnectConfiguration> configManager = 
  new ConfigurationManager<OpenIdConnectConfiguration>("http://someaddress.com", 
                                                       new OpenIdConnectConfigurationRetriever());
OpenIdConnectConfiguration config = await configManager.GetConfigurationAsync().ConfigureAwait(false);
TokenValidationParameters validationParameters = new TokenValidationParameters()
{
  …
  IssuerSigningKeys = config.SigningKeys;
  …
}

JsonWebTokenHandler tokenHandler = new JsonWebTokenHandler();
result = Handler.ValidateToken(jwtToken, validationParameters);
if (result.Exception != null && result.Exception is SecurityTokenSignatureKeyNotFoundException)
{
  configManager.RequestRefresh();
  config = await configManager.GetConfigurationAsync().ConfigureAwait(false);
  validationParameters = new TokenValidationParameters()
  {
    …
    IssuerSigningKeys = config.SigningKeys,
    …
  };
  // attempt to validate token again after refresh
  result = Handler.ValidateToken(jwtToken, validationParameters);
}
```

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte [Validación de tokens en una API web protegida](scenario-protected-web-api-app-configuration.md#token-validation).
