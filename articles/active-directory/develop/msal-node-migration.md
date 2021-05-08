---
title: Migración de una aplicación Node.js de ADAL a MSAL | Azure
titleSuffix: Microsoft identity platform
description: En este artículo se explica cómo actualizar una aplicación Node.js con el objetivo de usar la biblioteca de autenticación de Microsoft (MSAL), para habilitar la autenticación y autorización, en lugar de la biblioteca de autenticación de Active Directory (ADAL).
services: active-directory
author: derisen
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 04/26/2021
ms.author: v-doeris
ms.openlocfilehash: e2b82976c84d838f8c774cfba39edb630cbceb61
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108124300"
---
# <a name="how-to-migrate-a-nodejs-app-from-adal-to-msal"></a>Migración de una aplicación Node.js de ADAL a MSAL

La [biblioteca de autenticación de Microsoft para Node](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) (nodo MSAL) es ahora el SDK recomendado para habilitar la autenticación y autorización en las aplicaciones registradas en la plataforma de identidad de Microsoft. En este artículo se explican cuáles son los pasos importantes que se deben seguir para migrar las aplicaciones de la biblioteca de autenticación de Active Directory para Node (nodo ADAL) al nodo MSAL.

## <a name="prerequisites"></a>Prerequisites

- Versión 10, 12 o 14 de Node. Consulte la [nota sobre la compatibilidad con versiones](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node#node-version-support).

## <a name="update-app-registration-settings"></a>Actualización de la configuración de registro de aplicaciones

Al trabajar con el nodo ADAL, es probable que se utilice el **punto de conexión de Azure AD v1.0**. También debería plantearse usar el **punto de conexión de Azure AD v2.0** para las aplicaciones que se migran de ADAL a MSAL.

1. Revise las [diferencias entre los puntos de conexión v1 y v2](https://docs.microsoft.com/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison).
1. Si es necesario, actualice en consecuencia los registros de aplicaciones que ya existan.

> [!NOTE]
> Para garantizar la compatibilidad con versiones anteriores, el nodo MSAL admite los puntos de conexión v1.0 y v2.0.

## <a name="install-and-import-msal"></a>Instalación e importación de MSAL

1. Instale el paquete del nodo MSAL mediante NPM:

```console
npm install @azure/msal-node
```

1. Después, importe el nodo MSAL en el código:

```javascript
const msal = require('@azure/msal-node');
```

1. Por último, desinstale el paquete del nodo ADAL y quite las referencias del código:

```console
npm uninstall adal-node
```

## <a name="initialize-msal"></a>Inicializar MSAL

En el nodo ADAL, se inicializa un objeto `AuthenticationContext`, que luego expone los métodos que se pueden usar en distintos flujos de autenticación (por ejemplo, `acquireTokenWithAuthorizationCode` para las aplicaciones web). Al inicializarse, el único parámetro obligatorio es el identificador **URI de autoridad**:

```javascript
var adal = require('adal-node');

var authorityURI = "https://login.microsoftonline.com/common";
var authenticationContex = new adal.AuthenticationContext(authorityURI);
```

En el nodo MSAL, tiene dos alternativas: si va a compilar una aplicación móvil o una aplicación de escritorio, se crea una instancia de un objeto `PublicClientApplication`. El constructor espera un [objeto de configuración](#configure-msal) que contiene el parámetro `clientId` como mínimo. MSAL establece `https://login.microsoftonline.com/common` como valor predeterminado del identificador URI de autoridad si no se especifica.

```javascript
const msal = require('@azure/msal-node');

const pca = new msal.PublicClientApplication({
        auth: {
            clientId = "YOUR_CLIENT_ID"
        }
    });
```

> [!NOTE]
> Si usa la autoridad `https://login.microsoftonline.com/common` en v2.0, permitirá a los usuarios iniciar sesión con una cuenta personal de Microsoft (MSA) o una de organización de Azure AD. En el nodo MSAL, si quiere restringir el inicio de sesión a cualquier cuenta de Azure AD (el mismo comportamiento que con el nodo ADAL), use `https://login.microsoftonline.com/organizations`.

Por otro lado, si va a compilar una aplicación web o una aplicación de demonio, cree una instancia de un objeto `ConfidentialClientApplication`. Con estas aplicaciones, también debe proporcionar una *credencial de cliente*, como un secreto de cliente o un certificado:

```javascript
const msal = require('@azure/msal-node');

const cca = new msal.ConfidentialClientApplication({
        auth: {
            clientId = "YOUR_CLIENT_ID",
            clientSecret = "YOUR_CLIENT_SECRET"
        }
    });
```

Tanto `PublicClientApplication` como `ConfidentialClientApplication`, a diferencia del objeto `AuthenticationContext` de ADAL, están enlazados a un identificador de cliente. Esto significa que si tiene distintos identificadores de cliente que quiere usar en la aplicación, debe crear una instancia de una nueva instancia de MSAL para cada uno de ellos. Para obtener más información, consulte [Inicialización del nodo MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-node/docs/initialize-confidential-client-application.md).

## <a name="configure-msal"></a>Configuración de MSAL

Al compilar aplicaciones en la plataforma de identidad de Microsoft, la aplicación contendrá muchos parámetros relacionados con la autenticación. En el nodo ADAL, el objeto `AuthenticationContext` tiene un número limitado de parámetros de configuración con los que se pueden crear instancias, mientras que el resto de los parámetros se quedan sin respuesta en el código (por ejemplo, *clientSecret*):

```javascript
var adal = require('adal-node');

var authority = "https://login.microsoftonline.com/YOUR_TENANT_ID"
var validateAuthority = true,
var cache = null;

var authenticationContext = new adal.AuthenticationContext(authority, validateAuthority, cache);
```

- `authority`: dirección URL que identifica una entidad de token.
- `validateAuthority`: una característica que impide que el código solicite tokens a una autoridad potencialmente malintencionada.
- `cache`: establece la caché de tokens que utiliza esta instancia de AuthenticationContext.  Si no se establece este parámetro, se usa una caché en memoria predeterminada.

Por otro lado, el nodo MSAL usa un objeto de configuración de tipo [Configuration](https://azuread.github.io/microsoft-authentication-library-for-js/ref/modules/_azure_msal_node.html#configuration). Este contiene las siguientes propiedades:

```javascript
const msal = require('@azure/msal-node');

const msalConfig = {
    auth: {
        clientId: "YOUR_CLIENT_ID",
        authority: "https://login.microsoftonline.com/YOUR_TENANT_ID",
        clientSecret: "YOUR_TENANT_ID",
        knownAuthorities: [], 
    },
    cache: {
        // your implementation of caching
    },
    system: {
        loggerOptions: { /** logging related options */ }
    }
}


const cca = new msal.ConfidentialClientApplication(msalConfig);
```

Como diferencia importante, MSAL no tiene una marca para deshabilitar la validación de autoridad, y las autoridades siempre se validan de forma predeterminada. MSAL compara la autoridad solicitada con una lista de autoridades conocidas por Microsoft o una lista de autoridades que se ha especificado en la configuración. Para obtener más información, consulte [Opciones de configuración](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-node/docs/configuration.md).

## <a name="enable-logging"></a>Habilitar registro

En el nodo ADAL, configure el registro por separado en cualquier lugar del código:

```javascript
var adal = require('adal-node');

//PII or OII logging disabled. Default Logger does not capture any PII or OII.
adal.logging.setLoggingOptions({
  log: function (level, message, error) {
    console.log(message);

    if (error) {
      console.log(error);
    }
  },
  level: logging.LOGGING_LEVEL.VERBOSE, // provide the logging level
  loggingWithPII: false  // Determine if you want to log personal identification information. The default value is false.
});
```

En el nodo MSAL, el registro forma parte de las opciones de configuración y se crea con la inicialización de la instancia del nodo MSAL:

```javascript
const msal = require('@azure/msal-node');

const msalConfig = {
    auth: {
        // authentication related parameters
    },
    cache: {
        // cache related parameters
    },
    system: {
        loggerOptions: {
            loggerCallback(loglevel, message, containsPii) {
                console.log(message);
            },
            piiLoggingEnabled: false,
            logLevel: msal.LogLevel.Verbose,
        }
    }
}

const cca = new msal.ConfidentialClientApplication(msalConfig);
```

## <a name="use-scopes-instead-of-resources"></a>Uso de ámbitos en lugar de recursos

Una diferencia importante entre los puntos de conexión v1.0 y v2.0 es cómo se accede a los recursos. En el nodo ADAL, primero registraría un permiso en el portal de registro de aplicaciones y, a continuación, solicitaría un token de acceso para un recurso (como Microsoft Graph), como se muestra a continuación:

```javascript
  authenticationContext.acquireTokenWithAuthorizationCode(
    req.query.code,
    redirectUri,
    resource, // e.g. 'https://graph.microsoft.com'
    clientId,
    clientSecret,
    function (err, response) {
      // do something with the authentication response
  );
```

El nodo MSAL admite los puntos de conexión **v1.0** y **v2.0**. El punto de conexión v2.0 emplea un modelo *centrado en el ámbito* para acceder a los recursos. Por lo tanto, cuando solicite un token de acceso para un recurso, también debe especificar el ámbito de ese recurso:

```javascript
    const tokenRequest = {
        code: req.query.code,
        scopes: ["https://graph.microsoft.com/User.Read"],
        redirectUri: REDIRECT_URI,
    };

    pca.acquireTokenByCode(tokenRequest).then((response) => {
        // do something with the authentication response
    }).catch((error) => {
        console.log(error);
    });
```

Una ventaja del modelo centrado en el ámbito es la posibilidad de usar *ámbitos dinámicos*. Al compilar aplicaciones con v1.0, era necesario registrar el conjunto completo de permisos (llamados *ámbitos estáticos*) que requería la aplicación para que el usuario diera su consentimiento en el momento del inicio de sesión. En v2.0, puede usar el parámetro de ámbito para solicitar los permisos en el momento en que lo desee (por eso se llaman *ámbitos dinámicos*). Esto permite que el usuario dé su **consentimiento incremental** a los ámbitos. Por lo tanto, si al inicio solo quería que el usuario iniciara sesión en la aplicación y no necesita ningún tipo de acceso, puede hacerlo. Si posteriormente necesita poder leer el calendario del usuario, podrá solicitar el ámbito de calendario en los métodos acquireToken y obtener el consentimiento del usuario. Para obtener más información, consulte [Recursos y ámbitos](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/resources-and-scopes.md).

## <a name="switch-to-msal-api"></a>Cambio a la API de MSAL

La mayoría de los métodos públicos del nodo ADAL tienen equivalentes en el nodo MSAL:

| ADAL                                | MSAL                              | Notas                             |
|-------------------------------------|-----------------------------------|-----------------------------------|
| `acquireToken`                      | `acquireTokenSilent`              | Se ha cambiado el nombre y ahora espera un objeto de [cuenta](https://azuread.github.io/microsoft-authentication-library-for-js/ref/modules/_azure_msal_common.html#accountinfo). |
| `acquireTokenWithAuthorizationCode` | `acquireByAuthorizationCode`      |                                   |
| `acquireTokenWithClientCredentials` | `acquireTokenByClientCredentials` |                                   |
| `acquireTokenWithRefreshToken`      | `acquireTokenByRefreshToken`      |                                   |
| `acquireTokenWithDeviceCode`        | `acquireTokenByDeviceCode`        | Ahora abstrae la adquisición de código de usuario (véase a continuación). |
| `acquireTokenWithUsernamePassword`  | `acquireTokenByUsernamePassword`  |                                   |

Sin embargo, algunos métodos del nodo ADAL están en desuso, mientras que el nodo MSAL ofrece nuevos métodos:

| ADAL                              | MSAL                            | Notas                             |
|-----------------------------------|---------------------------------|-----------------------------------|
| `acquireUserCode`                   | N/D                             | Se ha combinado con `acquireTokeByDeviceCode` (véase más arriba).|
| N/D                               | `acquireTokenOnBehalfOf`          | Nuevo método que abstrae el [flujo OBO](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow). |
| `acquireTokenWithClientCertificate` | N/D                             | Ya no es necesario, dado que ahora los certificados se asignan durante la inicialización (consulte las [opciones de configuración](#configure-msal)). |
| N/D                               | `getAuthCodeUrl`                  | Nuevo método que abstrae la construcción de URL de [puntos de conexión de autorización](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-protocols#endpoints). |

## <a name="use-promises-instead-of-callbacks"></a>Uso de promesas en lugar de devoluciones de llamada

En el nodo ADAL, las devoluciones de llamada se usan para cualquier operación después de que la autenticación se realice correctamente y se obtenga una respuesta:

```javascript
var context = new AuthenticationContext(authorityUrl, validateAuthority);

context.acquireTokenWithClientCredentials(resource, clientId, clientSecret, function(err, response) {
  if (err) {
    console.log(err);
  } else {
    // do something with the authentication response
  }
});
```

En el nodo MSAL, se usan promesas:

```javascript
    const cca = new msal.ConfidentialClientApplication(msalConfig);

    cca.acquireTokenByClientCredentials(tokenRequest).then((response) => {
        // do something with the authentication response
    }).catch((error) => {
        console.log(error);
    });
```

También puede usar la sintaxis **async/await** que se incluye con ES8:

```javascript
    try {
        const authResponse = await cca.acquireTokenByCode(tokenRequest);
    } catch (error) {
        console.log(error);
    }
```

## <a name="enable-token-caching"></a>Habilitación del almacenamiento en caché de tokens

En el nodo ADAL, tenía la opción de importar una caché de tokens en memoria. La caché de tokens se usa como parámetro al inicializar un objeto `AuthenticationContext`:

```javascript
var MemoryCache = require('adal-node/lib/memory-cache');

var cache = new MemoryCache();
var authorityURI = "https://login.microsoftonline.com/common";

var context = new AuthenticationContext(authorityURI, true, cache);
```

El nodo MSAL usa una caché de tokens en memoria de forma predeterminada. No es necesario importarla explícitamente; se expone como parte de los objetos `ConfidentialClientApplication` y `PublicClientApplication`.

```javascript
const msalTokenCache = publicClientApplication.getTokenCache();
```

También puede escribir la caché en el disco proporcionando su propio **complemento de caché**. El complemento de caché debe implementar la interfaz [ICachePlugin](https://azuread.github.io/microsoft-authentication-library-for-js/ref/interfaces/_azure_msal_common.icacheplugin.html). Al igual que el registro, el almacenamiento en caché forma parte de las opciones de configuración y se crea con la inicialización de la instancia del nodo MSAL:

```javascript
const msal = require('@azure/msal-node');

const msalConfig = {
    auth: {
        // authentication related parameters 
    },
    cache: {
        cachePlugin // your implementation of cache plugin
    },
    system: {
        // logging related options 
    }
}

const msalInstance = new ConfidentialClientApplication(msalConfig);
```

Se puede implementar un complemento de caché de ejemplo como se indica a continuación:

```javascript
const fs = require('fs');

// Call back APIs which automatically write and read into a .json file - example implementation
const beforeCacheAccess = async (cacheContext) => {
    cacheContext.tokenCache.deserialize(await fs.readFile(cachePath, "utf-8"));
};

const afterCacheAccess = async (cacheContext) => {
    if(cacheContext.cacheHasChanged) {
        await fs.writeFile(cachePath, cacheContext.tokenCache.serialize());
    }
};

// Cache Plugin
const cachePlugin = {
    beforeCacheAccess,
    afterCacheAccess
};
```

Si va a [desarrollar aplicaciones cliente públicas](https://docs.microsoft.com/azure/active-directory/develop/msal-client-applications), como aplicaciones de escritorio, las extensiones de autenticación de Microsoft para [Node](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/extensions/msal-node-extensions) ofrecen mecanismos seguros para que las aplicaciones cliente realicen la serialización y persistencia del almacenamiento en caché de tokens multiplataforma. Las plataformas admitidas son Windows, Mac y Linux.

> [!NOTE]
> Las [extensiones de autenticación de Microsoft para Node](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/extensions/msal-node-extensions) **no** se recomiendan para las aplicaciones web, ya que pueden provocar problemas de escalado y rendimiento. En su lugar, se recomienda que las aplicaciones web conserven la caché en la sesión.

## <a name="remove-logic-around-refresh-tokens"></a>Eliminación de la lógica en torno a los tokens de actualización

En el nodo ADAL, los tokens de actualización (RT) se han expuesto, lo que permite desarrollar soluciones en torno al uso de estos tokens mediante el almacenamiento en caché y el método `acquireTokenWithRefreshToken`. Escenarios típicos en los que los RT son especialmente relevantes:

- Servicios de larga duración que realizan acciones, como la actualización de paneles en nombre de los usuarios cuando estos ya no están conectados.
- Escenarios de WebFarm que permiten al cliente llevar el RT al servicio web (el almacenamiento en caché se realiza en el lado cliente, con la cookie cifrada, y no en el del servidor).

Por motivos de seguridad, el nodo MSAL no expone tokens de actualización. En su lugar, MSAL controla la actualización de tokens automáticamente. Por lo tanto, ya no es necesario crear lógica para esto. Aun así, si necesita usar los tokens de actualización adquiridos previamente y migrarlos, el nodo MSAL ofrece `acquireTokenByRefreshToken`, que es equivalente al método `acquireTokenWithRefreshToken` del nodo ADAL.

## <a name="run-the-app"></a>Ejecución la aplicación

Una vez realizados los cambios, ejecute la aplicación y pruebe el escenario de autenticación:

```console
npm start
```

## <a name="example-securing-web-apps-with-adal-node-vs-msal-node"></a>Ejemplo: Diferencias en la protección de aplicaciones web con el nodo ADAL y con el nodo MSAL

En el fragmento de código siguiente se muestra una aplicación web cliente confidencial en el marco Express.js. La aplicación está protegida con el nodo ADAL. Realiza un inicio de sesión cuando un usuario alcanza la ruta de autenticación `/auth`, adquiere un token de acceso para Microsoft Graph mediante la ruta `/redirect` y, a continuación, muestra el contenido de dicho token.

```javascript
// Import dependencies
var express = require('express');
var crypto = require('crypto');
var adal = require('adal-node');

// Authentication parameters
var clientId = 'Enter_the_Application_Id_Here';
var clientSecret = 'Enter_the_Client_Secret_Here';
var tenant = 'common';
var authorityUrl = 'https://login.microsoftonline.com/' + tenant;
var redirectUri = 'http://localhost:3000/redirect';
var resource = 'https://graph.microsoft.com';

// Configure logging
adal.Logging.setLoggingOptions({
    log: function (level, message, error) {
        console.log(message);
    },
    level: adal.Logging.LOGGING_LEVEL.VERBOSE,
    loggingWithPII: false
});

// Auth code request URL template
var templateAuthzUrl = 'https://login.microsoftonline.com/' + tenant + 
 '/oauth2/authorize?response_type=code&client_id=' + clientId + '&redirect_uri=' 
 + redirectUri + '&state=<state>&resource=' + resource;

// Initialize express
var app = express();

// State variable persists throughout the app lifetime
app.locals.state = "";

app.get('/auth', function(req, res) {

    // Create a random string as state parameter, which is used against XSRF
    crypto.randomBytes(48, function(ex, buf) {
        app.locals.state = buf.toString('base64').replace(/\//g, '_').replace(/\+/g, '-');
        
        // Construct auth code request URL
        var authorizationUrl = templateAuthzUrl.replace('<state>', app.locals.state);
        res.redirect(authorizationUrl);
    });
});

app.get('/redirect', function(req, res) {
    // Compare state parameter against XSRF
    if (app.locals.state !== req.query.state) {
        res.send('error: state does not match');
    }

    // Initialize an AuthenticationContext object
    var authenticationContext = new adal.AuthenticationContext(authorityUrl);
    
    // Exchange auth code for tokens
    authenticationContext.acquireTokenWithAuthorizationCode(
        req.query.code, 
        redirectUri, 
        resource, 
        clientId, 
        clientSecret,
        function(err, response) {
            res.send(response);
        }
    );
});

app.listen(3000, function() { console.log(`listening on port 3000!`); });
```

Una aplicación web con una funcionalidad equivalente se puede proteger con el nodo MSAL, como se muestra a continuación:

```javascript
// Import dependencies
const express = require("express");
const msal = require('@azure/msal-node');

// Authentication parameters
const config = {
    auth: {
        clientId: "Enter_the_Application_Id_Here",
        authority: "https://login.microsoftonline.com/common",
        clientSecret: "Enter_the_Client_Secret_Here"
    },
    system: {
        loggerOptions: {
            loggerCallback(loglevel, message, containsPii) {
                console.log(message);
            },
            piiLoggingEnabled: false,
            logLevel: msal.LogLevel.Verbose,
        }
    }
};

const REDIRECT_URI = "http://localhost:3000/redirect";

// Initialize MSAL Node object using authentication parameters
const cca = new msal.ConfidentialClientApplication(config);

// Initialize express
const app = express();

app.get('/auth', (req, res) => {
    
    // Construct a request object for auth code
    const authCodeUrlParameters = {
        scopes: ["user.read"],
        redirectUri: REDIRECT_URI,
    };

    // Request auth code, then redirect
    cca.getAuthCodeUrl(authCodeUrlParameters).then((response) => {
        res.redirect(response);
    }).catch((error) => res.send(error));
});

app.get('/redirect', (req, res) => {
    
    // Use the auth code in redirect request to construct
    // a token request object
    const tokenRequest = {
        code: req.query.code,
        scopes: ["user.read"],
        redirectUri: REDIRECT_URI,
    };

    // Exchange the auth code for tokens
    cca.acquireTokenByCode(tokenRequest).then((response) => {
        res.send(response);
    }).catch((error) => res.status(500).send(error));
});

app.listen(3000, () => console.log(`listening on port 3000!`));
```

## <a name="next-steps"></a>Pasos siguientes

- [Referencia de la API del nodo MSAL](https://azuread.github.io/microsoft-authentication-library-for-js/ref/modules/_azure_msal_node.html)
- [Ejemplos de código del nodo MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-node-samples)
