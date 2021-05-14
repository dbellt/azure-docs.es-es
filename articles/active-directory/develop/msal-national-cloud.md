---
title: Uso de MSAL en una aplicación en la nube nacional | Azure
titleSuffix: Microsoft identity platform
description: La Biblioteca de autenticación de Microsoft (MSAL) permite a los desarrolladores de aplicaciones adquirir tokens para llamar a las API web protegidas. Estas API web pueden ser Microsoft Graph, otras API de Microsoft, API web de terceros o su propia API web. MSAL admite varias plataformas y arquitecturas de aplicación.
services: active-directory
author: negoe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/22/2019
ms.author: negoe
ms.reviewer: marsma, nacanuma
ms.custom: aaddev
ms.openlocfilehash: c1ecf807d566fd6603f12ebc820c176edf96ec14
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108071853"
---
# <a name="use-msal-in-a-national-cloud-environment"></a>Uso de MSAL en un entorno de nube nacional

Las [nubes nacionales](authentication-national-cloud.md), también conocidas como nubes soberanas, son instancias físicamente aisladas de Azure. Estas regiones de Azure ayudan a asegurarse de que se cumplan los requisitos de residencia, soberanía y cumplimiento normativo de los datos dentro de fronteras geográficas.

Además de la nube mundial de Microsoft, la biblioteca de autenticación de Microsoft (MSAL) permite a los desarrolladores de aplicaciones en nubes nacionales adquirir tokens con el fin de autenticar e invocar API de web protegidas. Estas API web pueden Microsoft Graph u otras API de Microsoft.

Incluida la nube mundial, Azure Active Directory (Azure AD) se implementa en las siguientes nubes nacionales:  

- Azure Government
- Azure China 21Vianet
- Azure Alemania

En esta guía se muestra cómo iniciar sesión en cuentas profesionales y educativas, obtener un token de acceso y llamar a la API de Microsoft Graph en el entorno de la [nube de Azure Government](https://azure.microsoft.com/global-infrastructure/government/).

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar, asegúrese de que cumpla estos requisitos previos.

### <a name="choose-the-appropriate-identities"></a>Elección de las identidades adecuadas

Las aplicaciones de [Azure Government](../../azure-government/index.yml) pueden usar identidades de Azure AD Government e identidades pública de Azure AD para autenticar a los usuarios. Dado que puede usar cualquiera de estas identidades, decida qué punto de conexión de autoridad debe elegir para su escenario:

- Azure AD público: se suele usar si la organización ya tiene un inquilino público de Azure AD para admitir Microsoft 365 (público o GCC) u otra aplicación.
- Azure AD Government: se suele usar si la organización ya tiene un inquilino de Azure AD Government para admitir Office 365 (GCC High o DoD) o está creando un nuevo inquilino en Azure AD Government.

Una vez decidido, debe tener en cuenta también dónde se realiza el registro de la aplicación. Si elige identidades de Azure AD público para la aplicación de Azure Government, debe registrar la aplicación en el inquilino de Azure AD público.

### <a name="get-an-azure-government-subscription"></a>Obtener una suscripción a Azure Government

Para obtener una suscripción a Azure Government, consulte [Administración y conexión a su suscripción en Azure Government](../../azure-government/compare-azure-government-global-azure.md).

Si no tiene una suscripción a Azure Government, cree una [cuenta gratuita](https://azure.microsoft.com/global-infrastructure/government/request/) antes de comenzar.

Para obtener más información sobre el uso de una nube nacional con un lenguaje de programación determinado, elija la pestaña que coincida con su lenguaje:

## <a name="net"></a>[.NET](#tab/dotnet)

Puede usar MSAL.NET para iniciar la sesión de los usuarios, adquirir tokens y llamar a la API de Microsoft Graph en nubes nacionales.

En los siguientes tutoriales se muestra cómo compilar una aplicación web MVC de .NET Core 2.2. La aplicación usa OpenID Connect para iniciar la sesión de los usuarios con una cuenta profesional y educativa en una organización que pertenece a una nube nacional.

- Para iniciar la sesión de los usuarios y adquirir tokens, siga este tutorial: [Cree una aplicación web de ASP.NET Core que permita iniciar la sesión de usuarios en nubes soberanas con la Plataforma de identidad de Microsoft](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-4-Sovereign#build-an-aspnet-core-web-app-signing-in-users-in-sovereign-clouds-with-the-microsoft-identity-platform).
- Para llamar a la API de Microsoft Graph, siga este tutorial: [Uso de la Plataforma de identidad de Microsoft para llamar a Microsoft Graph API desde una aplicación web ASP.NET Core 2.x en nombre de un usuario que inicia sesión con su cuenta profesional y educativa en Microsoft National Cloud](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/2-WebApp-graph-user/2-4-Sovereign-Call-MSGraph#using-the-microsoft-identity-platform-to-call-the-microsoft-graph-api-from-an-an-aspnet-core-2x-web-app-on-behalf-of-a-user-signing-in-using-their-work-and-school-account-in-microsoft-national-cloud).

## <a name="javascript"></a>[JavaScript](#tab/javascript)

Para habilitar la aplicación MSAL.js para nubes soberanas:

- Registre la aplicación en un portal específico, en función de la nube. Para más información sobre cómo elegir el portal, consulte [Puntos de conexión de registro de aplicaciones.](authentication-national-cloud.md#app-registration-endpoints)
- Use cualquiera de las [muestras](https://github.com/Azure-Samples/ms-identity-javascript-tutorial) del repositorio con algunos cambios en la configuración, en función de la nube, que se menciona a continuación.
- Use una autoridad específica, en función de la nube en la que haya registrado la aplicación. Para obtener más información sobre las autoridades de distintas nubes, consulte [Puntos de conexión de Autenticación de Azure AD](authentication-national-cloud.md#azure-ad-authentication-endpoints).
- La llamada a Microsoft Graph API requiere una dirección URL de punto de conexión específica de la nube que se está usando. Para buscar los puntos de conexión de Microsoft Graph de todas las nubes nacionales, consulte [Puntos de conexión de raíz de servicio de Microsoft Graph y Graph Explorer](/graph/deployments#microsoft-graph-and-graph-explorer-service-root-endpoints).

Este es un ejemplo de autoridad:

```json
"authority": "https://login.microsoftonline.us/Enter_the_Tenant_Info_Here"
```

A continuación, se muestra un ejemplo de un punto de conexión de Microsoft Graph con el ámbito:

```json
"endpoint" : "https://graph.microsoft.us/v1.0/me"
"scope": "User.Read"
```

Este es el mínimo de código para autenticar a un usuario con una nube soberana y llamar a Microsoft Graph:

```javascript
const msalConfig = {
    auth: {
        clientId: "Enter_the_Application_Id_Here",
        authority: "https://login.microsoftonline.us/Enter_the_Tenant_Info_Here",
        redirectUri: "/",
    }
};

// Initialize MSAL
const msalObj = new PublicClientApplication(msalConfig);

// Get token using popup experience
try {
    const graphToken = await msalObj.acquireTokenPopup({
        scopes: ["User.Read"]
    });
} catch(error) {
    console.log(error)
}

// Call the Graph API
const headers = new Headers();
const bearer = `Bearer ${graphToken}`;

headers.append("Authorization", bearer);

fetch("https://graph.microsoft.us/v1.0/me", {
    method: "GET",
    headers: headers
})
```


## <a name="python"></a>[Python](#tab/python)

Para habilitar la aplicación MSAL para Python para nubes soberanas:

- Registre la aplicación en un portal específico, en función de la nube. Para más información sobre cómo elegir el portal, consulte [Puntos de conexión de registro de aplicaciones.](authentication-national-cloud.md#app-registration-endpoints)
- Use cualquiera de las [muestras](https://github.com/AzureAD/microsoft-authentication-library-for-python/tree/dev/sample) del repositorio con algunos cambios en la configuración, en función de la nube, que se menciona a continuación.
- Use una autoridad específica, en función de la nube en la que haya registrado la aplicación. Para obtener más información sobre las autoridades de distintas nubes, consulte [Puntos de conexión de Autenticación de Azure AD](authentication-national-cloud.md#azure-ad-authentication-endpoints).

    Este es un ejemplo de autoridad:

    ```json
    "authority": "https://login.microsoftonline.us/Enter_the_Tenant_Info_Here"
    ```

- La llamada a Microsoft Graph API requiere una dirección URL de punto de conexión específica de la nube que se está usando. Para buscar los puntos de conexión de Microsoft Graph de todas las nubes nacionales, consulte [Puntos de conexión de raíz de servicio de Microsoft Graph y Graph Explorer](/graph/deployments#microsoft-graph-and-graph-explorer-service-root-endpoints).

    A continuación, se muestra un ejemplo de un punto de conexión de Microsoft Graph con el ámbito:

    ```json
    "endpoint" : "https://graph.microsoft.us/v1.0/me"
    "scope": "User.Read"
    ```

## <a name="java"></a>[Java](#tab/java)

Para habilitar la aplicación MSAL para Java para nubes soberanas:

- Registre la aplicación en un portal específico, en función de la nube. Para más información sobre cómo elegir el portal, consulte [Puntos de conexión de registro de aplicaciones.](authentication-national-cloud.md#app-registration-endpoints)
- Use cualquiera de las [muestras](https://github.com/AzureAD/microsoft-authentication-library-for-java/tree/dev/src/samples) del repositorio con algunos cambios en la configuración, en función de la nube, que se menciona a continuación.
- Use una autoridad específica, en función de la nube en la que haya registrado la aplicación. Para obtener más información sobre las autoridades de distintas nubes, consulte [Puntos de conexión de Autenticación de Azure AD](authentication-national-cloud.md#azure-ad-authentication-endpoints).

Este es un ejemplo de autoridad:

```json
"authority": "https://login.microsoftonline.us/Enter_the_Tenant_Info_Here"
```

- La llamada a Microsoft Graph API requiere una dirección URL de punto de conexión específica de la nube que se está usando. Para buscar los puntos de conexión de Microsoft Graph de todas las nubes nacionales, consulte [Puntos de conexión de raíz de servicio de Microsoft Graph y Graph Explorer](/graph/deployments#microsoft-graph-and-graph-explorer-service-root-endpoints).

A continuación se muestra un ejemplo de un punto de conexión de Graph con el ámbito:

```json
"endpoint" : "https://graph.microsoft.us/v1.0/me"
"scope": "User.Read"
```

## <a name="objective-c"></a>[Objective-C](#tab/objc)

MSAL para iOS y macOS se puede usar para adquirir tokens en nubes nacionales, pero se requiere una configuración adicional al crear `MSALPublicClientApplication`.

Por ejemplo, si quiere que la aplicación sea una aplicación de varios inquilinos en una nube nacional (en la Administración Pública de Estados Unidos), podría escribir lo siguiente:

```objc
MSALAADAuthority *aadAuthority =
                [[MSALAADAuthority alloc] initWithCloudInstance:MSALAzureUsGovernmentCloudInstance
                                                   audienceType:MSALAzureADMultipleOrgsAudience
                                                      rawTenant:nil
                                                          error:nil];

MSALPublicClientApplicationConfig *config =
                [[MSALPublicClientApplicationConfig alloc] initWithClientId:@"<your-client-id-here>"
                                                                redirectUri:@"<your-redirect-uri-here>"
                                                                  authority:aadAuthority];

NSError *applicationError = nil;
MSALPublicClientApplication *application =
                [[MSALPublicClientApplication alloc] initWithConfiguration:config error:&applicationError];
```

## <a name="swift"></a>[Swift](#tab/swift)

MSAL para iOS y macOS se puede usar para adquirir tokens en nubes nacionales, pero se requiere una configuración adicional al crear `MSALPublicClientApplication`.

Por ejemplo, si quiere que la aplicación sea una aplicación de varios inquilinos en una nube nacional (en la Administración Pública de Estados Unidos), podría escribir lo siguiente:

```swift
let authority = try? MSALAADAuthority(cloudInstance: .usGovernmentCloudInstance, audienceType: .azureADMultipleOrgsAudience, rawTenant: nil)

let config = MSALPublicClientApplicationConfig(clientId: "<your-client-id-here>", redirectUri: "<your-redirect-uri-here>", authority: authority)
if let application = try? MSALPublicClientApplication(configuration: config) { /* Use application */}
```

---

## <a name="next-steps"></a>Pasos siguientes

Consulte [Puntos de conexión de autenticación de nubes nacionales](authentication-national-cloud.md) para ver una lista de las direcciones URL de Azure Portal y los puntos de conexión de token para cada nube.

Documentación de la nube nacional:

- [Azure Government](../../azure-government/index.yml)
- [Azure China 21Vianet](/azure/china/)
- [Azure Alemania](../../germany/index.yml)