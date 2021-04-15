---
title: Migración de API web basadas en OWIN a b2clogin.com o a un dominio personalizado
titleSuffix: Azure AD B2C
description: Obtenga información sobre cómo habilitar una API web de .NET para admitir tokens emitidos por varios emisores de token mientras migra las aplicaciones a b2clogin.com.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/15/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 2ab93a8742d210969ef6ad10683bed92221c1a57
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107256708"
---
# <a name="migrate-an-owin-based-web-api-to-b2clogincom-or-a-custom-domain"></a>Migración de una API web basada en OWIN a b2clogin.com o a un dominio personalizado

En este artículo se describe una técnica para habilitar la compatibilidad con varios emisores de tokens en API web que implementan la [interfaz web abierta para .NET (OWIN)](http://owin.org/). La compatibilidad con varios puntos de conexión de token es útil cuando se migran API de Azure Active Directory B2C (Azure AD B2C) y sus aplicaciones de un dominio a otro. Por ejemplo, de *login.microsoftonline.com* a *b2clogin.com*, o a un [dominio personalizado](custom-domain.md).

Al agregar compatibilidad en la API para aceptar tokens que ha emitido b2clogin.com, login.microsoftonline.com o un dominio personalizado, puede migrar las aplicaciones web de manera escalonada antes de quitar la compatibilidad con los tokens emitidos por login.microsoftonline.com de la API.

En las secciones siguientes se muestra un ejemplo de cómo habilitar varios emisores en una API web que usa los componentes de middleware de [Microsoft OWIN][katana] (Katana). Aunque los ejemplos de código son específicos del middleware de Microsoft OWIN, la técnica general debería ser aplicable a otras bibliotecas OWIN.

## <a name="prerequisites"></a>Prerrequisitos

Antes de continuar con los pasos de este artículo, necesita tener los siguientes recursos de Azure AD B2C:

* [Flujos de usuario](tutorial-create-user-flows.md?pivots=b2c-user-flow) o [directivas personalizadas](tutorial-create-user-flows.md?pivots=b2c-custom-policy) creadas en el inquilino

## <a name="get-token-issuer-endpoints"></a>Obtención de los puntos de conexión de emisores de tokens

Primero, debe obtener los URI del punto de conexión de emisor de token para cada emisor que quiera admitir en la API. Para obtener los puntos de conexión *b2clogin.com* y *login.microsoftonline.com* que se admiten en su inquilino de Azure AD B2C, use el siguiente procedimiento en Azure Portal.

Para empezar, seleccione uno de los flujos de usuario existentes:

1. Vaya a su inquilino de Azure AD B2C en [Azure Portal](https://portal.azure.com).
1. En **Directivas**, seleccione **Flujos de usuario (directivas)** .
1. Seleccione una directiva existente, por ejemplo *B2C_1_signupsignin1* y, luego, seleccione **Ejecutar flujo de usuario**.
1. En el encabezado **Ejecutar flujo de usuario** situado cerca de la parte superior de la página, seleccione el hipervínculo para ir hasta el punto de conexión de detección de OpenID Connect para ese flujo de usuario.

    ![Hipervínculo de URI conocido en la página Ejecutar ahora de Azure Portal](media/multi-token-endpoints/portal-01-policy-link.png)

1. En la página que se abre en el explorador, registre el valor `issuer`, por ejemplo:

    `https://your-b2c-tenant.b2clogin.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/`

1. Use la lista desplegable **Seleccionar dominio** para seleccionar el otro dominio y, después, vuelva a realizar los dos pasos anteriores y registre su valor `issuer`.

Ahora debería tener dos identificadores URI registrados similares a:

```
https://login.microsoftonline.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/
https://your-b2c-tenant.b2clogin.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/
```

### <a name="custom-policies"></a>Directivas personalizadas

Si tiene directivas personalizadas en lugar de flujos de usuario, puede usar un proceso similar para obtener los URI del emisor.

1. Ir al inquilino de Azure AD B2C
1. Seleccione **Marco de experiencia de identidad**.
1. Seleccione una de las directivas de usuario de confianza, por ejemplo, *B2C_1A_signup_signin*.
1. Use la lista desplegable **Seleccionar dominio** para seleccionar un dominio, por ejemplo, *yourtenant.b2clogin.com*.
1. Seleccione el hipervínculo que se muestra en **el punto de conexión de detección de OpenID Connect**.
1. Registre el valor de `issuer`.
1. Siga los pasos del 4 al 6 con el otro dominio, por ejemplo, *login.microsoftonline.com*.

## <a name="get-the-sample-code"></a>Obtención del código de ejemplo

Ahora que tiene ambos URI de punto de conexión de token, debe actualizar el código para especificar que ambos puntos de conexión son emisores válidos. Para examinar un ejemplo, descargue o clone la aplicación de ejemplo y, luego, actualice el ejemplo para admitir ambos puntos de conexión como emisores válidos.

Descargue el archivo: [active-directory-b2c-dotnet-webapp-and-webapi-master.zip][sample-archive]

```
git clone https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi.git
```

## <a name="enable-multiple-issuers-in-web-api"></a>Habilitación de varios emisores en API web

En esta sección, actualizará el código para especificar que ambos puntos de conexión de emisores de tokens son válidos.

1. Abra la solución **B2C-WebAPI-DotNet.sln** en Visual Studio.
1. En el proyecto **TaskService**, abra el archivo *TaskService\\App_Start\\ **Startup.Auth.cs**.* en el editor.
1. Agregue la siguiente directiva `using` al principio del archivo:

    `using System.Collections.Generic;`
1. Agregue la propiedad [`ValidIssuers`][validissuers] a la definición [`TokenValidationParameters`][tokenvalidationparameters] y especifique los dos identificadores URI que registró en la sección anterior:

    ```csharp
    TokenValidationParameters tvps = new TokenValidationParameters
    {
        // Accept only those tokens where the audience of the token is equal to the client ID of this app
        ValidAudience = ClientId,
        AuthenticationType = Startup.DefaultPolicy,
        ValidIssuers = new List<string> {
            "https://login.microsoftonline.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/",
            "https://{your-b2c-tenant}.b2clogin.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/"//,
            //"https://your-custom-domain/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/"
        }
    };
    ```

MSAL.NET proporciona `TokenValidationParameters` y lo consume el middleware OWIN en la siguiente sección de código de *Startup.Auth.cs.* . Si se especifican varios emisores válidos, la canalización de la aplicación OWIN es consciente de que ambos puntos de conexión de token son emisores válidos.

```csharp
app.UseOAuthBearerAuthentication(new OAuthBearerAuthenticationOptions
{
    // This SecurityTokenProvider fetches the Azure AD B2C metadata &  from the OpenID Connect metadata endpoint
    AccessTokenFormat = new JwtFormat(tvps, new tCachingSecurityTokenProvider(String.Format(AadInstance, ultPolicy)))
});
```

Como se mencionó anteriormente, otras bibliotecas OWIN proporcionan normalmente una instalación similar para admitir varios emisores. Aunque el objetivo de este artículo no es proporcionar ejemplos de cada biblioteca, puede usar una técnica similar con la mayoría de ellas.

## <a name="switch-endpoints-in-web-app"></a>Cambio de los puntos de conexión en la aplicación web

Ahora que ya se admiten ambos URI en la API web, el siguiente paso es actualizar la aplicación web para que recupere los tokens del punto de conexión b2clogin.com.

Por ejemplo, puede configurar la aplicación web de ejemplo para que use el nuevo punto de conexión modificando el valor `ida:AadInstance` del archivo *TaskWebApp\\**Web.config** _ del proyecto _* TaskWebApp**.

Cambie el valor `ida:AadInstance` en *Web.config* de TaskWebApp para que haga referencia a `{your-b2c-tenant-name}.b2clogin.com` y no a `login.microsoftonline.com`.

Antes:

```xml
<!-- Old value -->
<add key="ida:AadInstance" value="https://login.microsoftonline.com/tfp/{0}/{1}" />
```

Después, reemplace `{your-b2c-tenant}` por el nombre de su inquilino B2C:

```xml
<!-- New value -->
<add key="ida:AadInstance" value="https://{your-b2c-tenant}.b2clogin.com/tfp/{0}/{1}" />
```

Cuando las cadenas de punto de conexión se construyen durante la ejecución de la aplicación web, se usan los puntos de conexión basados en b2clogin.com cuando se solicitan tokens.

Al usar el dominio personalizado:

```xml
<!-- Custom domain -->
<add key="ida:AadInstance" value="https://custom-domain/{0}/{1}" />
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo se ha presentado un método para configurar una API web que implementa el middleware de Microsoft OWIN (Katana) para aceptar tokens de varios puntos de conexión de emisor. Como puede observar, hay otras diversas cadenas en los archivos *Web.Config* de los proyectos TaskService y TaskWebApp que deberían cambiarse si quiere compilar y ejecutar estos proyectos en su propio inquilino. Puede modificar los proyectos de forma adecuada si quiere verlos en acción; sin embargo, el objetivo de este artículo no es mostrar un tutorial completo de cómo hacerlo.

Para más información sobre los diferentes tipos de tokens de seguridad emitidos por Azure AD B2C, consulte [Introducción a los tokens en Active Directory B2C](tokens-overview.md).

<!-- LINKS - External -->
[sample-archive]: https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi/archive/master.zip
[sample-repo]: https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi

<!-- LINKS - Internal -->
[katana]: /aspnet/aspnet/overview/owin-and-katana/
[validissuers]: /dotnet/api/microsoft.identitymodel.tokens.tokenvalidationparameters.validissuers
[tokenvalidationparameters]: /dotnet/api/microsoft.identitymodel.tokens.tokenvalidationparameters
