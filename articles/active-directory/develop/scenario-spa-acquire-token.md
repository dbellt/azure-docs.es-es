---
title: Obtención de un token para llamar a una API web (aplicaciones de una página) | Azure
titleSuffix: Microsoft identity platform
description: Aprenda a crear una aplicación de página única (adquirir un token para llamar a una API).
services: active-directory
author: negoe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 04/2/2021
ms.author: negoe
ms.custom: aaddev
ms.openlocfilehash: 2e4369c729fc5497f615f64c1f7235d19c293f98
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108227747"
---
# <a name="single-page-application-acquire-a-token-to-call-an-api"></a>Aplicación de página única: Adquisición de un token para llamar a una API

El patrón para la adquisición de tokens de API con [MSAL.js](https://github.com/AzureAD/microsoft-authentication-library-for-js) consiste en intentar realizar primero una solicitud de token silenciosa utilizando el método `acquireTokenSilent`. Cuando se llama a este método, la biblioteca comprueba primero la caché en el almacenamiento del explorador para ver si existe un token válido y, a continuación, lo devuelve. Cuando no hay ningún token válido en la memoria caché, intenta usar su token de actualización para obtener el token. Si la vigencia del token de actualización de 24 horas ha expirado, MSAL.js abrirá un iframe oculto para solicitar de forma silenciosa un nuevo código de autorización, que intercambiará por un nuevo token de actualización válido. Para más información sobre los valores de inicio de sesión único y duración del token en Azure AD, consulte la [vigencia de los tokens](active-directory-configurable-token-lifetimes.md).

Las solicitudes de token silenciosas a Azure AD pueden producir un error por motivos como un cambio de contraseña o actualizaciones en las directivas de acceso condicional.  Con más frecuencia, los errores se deben al vencimiento de la vigencia de 24 horas del token de actualización y a que [el explorador bloquea las cookies de terceros](reference-third-party-cookies-spas.md), lo que impide el uso de iframes ocultos para seguir autenticando al usuario.  En estos casos, tiene que invocar uno de los métodos interactivos para adquirir tokens (lo que puede pedirse al usuario):

* [Ventana emergente](#acquire-a-token-with-a-pop-up-window), mediante `acquireTokenPopup`
* [Redirección](#acquire-a-token-with-a-redirect), mediante `acquireTokenRedirect`

## <a name="choose-between-a-pop-up-or-redirect-experience"></a>Decisión entre una experiencia de ventana emergente o redirección

La elección de una u otra experiencia depende del flujo de la aplicación:

* Si no quiere que los usuarios salgan de la página principal de la aplicación durante la autenticación, se recomienda el método de ventana emergente. Dado que la redirección de la autenticación tiene lugar en una ventana emergente, se conserva el estado de la aplicación principal.

* Si los usuarios tienen restricciones del explorador o directivas donde las ventanas emergentes están deshabilitadas, puede usar el método de redirección. Use el método de redirección con Internet Explorer, ya que hay [problemas conocidos con las ventanas emergentes en este explorador](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/internet-explorer.md#popups).

Al crear la solicitud de token de acceso, puede establecer los ámbitos de API que quiere que incluya el token de acceso. Tenga en cuenta que puede que no se concedan todos los ámbitos solicitados en el token de acceso. Depende del consentimiento del usuario.

## <a name="acquire-a-token-with-a-pop-up-window"></a>Adquisición de un token con una ventana emergente

# <a name="javascript-msaljs-v2"></a>[JavaScript (MSAL.js v2)](#tab/javascript2)

En el código siguiente se combina el patrón descrito anteriormente con los métodos de una experiencia de ventana emergente:

```javascript
// MSAL.js v2 exposes several account APIs, logic to determine which account to use is the responsibility of the developer
const account = publicClientApplication.getAllAccounts()[0];

const accessTokenRequest = {
    scopes: ["user.read"],
    account: account
}

publicClientApplication.acquireTokenSilent(accessTokenRequest).then(function(accessTokenResponse) {
    // Acquire token silent success
    let accessToken = accessTokenResponse.accessToken;
    // Call your API with token
    callApi(accessToken);
}).catch(function (error) {
    //Acquire token silent failure, and send an interactive request
    if (error instanceof InteractionRequiredAuthError) {
        publicClientApplication.acquireTokenPopup(accessTokenRequest).then(function(accessTokenResponse) {
            // Acquire token interactive success
            let accessToken = accessTokenResponse.accessToken;
            // Call your API with token
            callApi(accessToken);
        }).catch(function(error) {
            // Acquire token interactive failure
            console.log(error);
        });
    }
    console.log(error);
});
```

# <a name="javascript-msaljs-v1"></a>[JavaScript (MSAL.js v1)](#tab/javascript1)

En el código siguiente se combina el patrón descrito anteriormente con los métodos de una experiencia de ventana emergente:

```javascript
const accessTokenRequest = {
    scopes: ["user.read"]
}

userAgentApplication.acquireTokenSilent(accessTokenRequest).then(function(accessTokenResponse) {
    // Acquire token silent success
    // Call API with token
    let accessToken = accessTokenResponse.accessToken;
}).catch(function (error) {
    //Acquire token silent failure, and send an interactive request
    if (error.errorMessage.indexOf("interaction_required") !== -1) {
        userAgentApplication.acquireTokenPopup(accessTokenRequest).then(function(accessTokenResponse) {
            // Acquire token interactive success
        }).catch(function(error) {
            // Acquire token interactive failure
            console.log(error);
        });
    }
    console.log(error);
});
```

# <a name="angular-msaljs-v2"></a>[Angular (MSAL.js v2)](#tab/angular2)

El contenedor MSAL Angular proporciona el interceptor HTTP, que adquiere de forma automática y silenciosa los tokens de acceso y los asocia a las solicitudes HTTP de las API.

Puede especificar los ámbitos de las API en la opción de configuración `protectedResourceMap`. `MsalInterceptor` solicitará estos ámbitos cuando adquiera automáticamente tokens.

```javascript
// In app.module.ts
import { PublicClientApplication, InteractionType } from '@azure/msal-browser';
import { MsalInterceptor, MsalModule } from '@azure/msal-angular';

@NgModule({
    declarations: [
        // ...
    ],
    imports: [
        // ...
        MsalModule.forRoot( new PublicClientApplication({
        auth: {
            clientId: 'Enter_the_Application_Id_Here',
        },
        cache: {
            cacheLocation: 'localStorage',
            storeAuthStateInCookie: isIE,
        }
        }), {
            interactionType: InteractionType.Popup,
            authRequest: {
                scopes: ['user.read']
            }
        }, {
            interactionType: InteractionType.Popup,
            protectedResourceMap: new Map([ 
                ['https://graph.microsoft.com/v1.0/me', ['user.read']]
            ])
        })
    ],
    providers: [
        {
            provide: HTTP_INTERCEPTORS,
            useClass: MsalInterceptor,
            multi: true
        }
    ],
    bootstrap: [AppComponent]
})
export class AppModule { }
```

Para conocer el éxito o fracaso de la adquisición de tokens silenciosa, MSAL Angular proporciona eventos a los que puede suscribirse. También es importante recordar esta información si quiere cancelar la suscripción.

```javascript
import { MsalBroadcastService } from '@azure/msal-angular';
import { EventMessage, EventType } from '@azure/msal-browser';

// In app.component.ts
export class AppComponent implements OnInit {
    private readonly _destroying$ = new Subject<void>();

    constructor(private broadcastService: MsalBroadcastService) { }

    ngOnInit() {
        this.broadcastService.msalSubject$
        .pipe(
            filter((msg: EventMessage) => msg.eventType === EventType.ACQUIRE_TOKEN_SUCCESS),
            takeUntil(this._destroying$)
        )
        .subscribe((result: EventMessage) => {
            // Do something with event payload here
        });
    }

    ngOnDestroy(): void {
        this._destroying$.next(undefined);
        this._destroying$.complete();
    }
}
```

Como alternativa, también puede adquirir tokens de forma explícita mediante los métodos de adquisición de token, tal como se describe en la biblioteca principal MSAL.js.

# <a name="angular-msaljs-v1"></a>[Angular (MSAL.js v1)](#tab/angular1)
El contenedor MSAL Angular proporciona el interceptor HTTP, que adquiere de forma automática y silenciosa los tokens de acceso y los asocia a las solicitudes HTTP de las API.
Puede especificar los ámbitos de las API en la opción de configuración `protectedResourceMap`. `MsalInterceptor` solicitará estos ámbitos cuando adquiera automáticamente tokens.

```javascript
// app.module.ts
@NgModule({
  declarations: [
    // ...
  ],
  imports: [
    // ...
    MsalModule.forRoot({
      auth: {
        clientId: 'Enter_the_Application_Id_Here',
      }
    },
    {
      popUp: !isIE,
      consentScopes: [
        'user.read',
        'openid',
        'profile',
      ],
      protectedResourceMap: [
        ['https://graph.microsoft.com/v1.0/me', ['user.read']]
      ]
    })
  ],
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: MsalInterceptor,
      multi: true
    }
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Para conocer el éxito o fracaso de la adquisición de tokens silenciosa, MSAL Angular proporciona devoluciones de llamada a las que puede suscribirse. También es importante recordar esta información si quiere cancelar la suscripción.

```javascript
// In app.component.ts
 ngOnInit() {
    this.subscription=  this.broadcastService.subscribe("msal:acquireTokenFailure", (payload) => {
    });
}
ngOnDestroy() {
   this.broadcastService.getMSALSubject().next(1);
   if (this.subscription) {
     this.subscription.unsubscribe();
   }
 }
```

Como alternativa, también puede adquirir tokens de forma explícita mediante los métodos de adquisición de token, tal como se describe en la biblioteca principal MSAL.js.

# <a name="react"></a>[React](#tab/react)

En el código siguiente se combina el patrón descrito anteriormente con los métodos de una experiencia de ventana emergente:

```javascript
import { InteractionRequiredAuthError, InteractionStatus } from "@azure/msal-browser";
import { AuthenticatedTemplate, useMsal } from "@azure/msal-react";

function ProtectedComponent() {
    const { instance, inProgress, accounts } = useMsal();
    const [apiData, setApiData] = useState(null);


    useEffect(() => {
        if (!apiData && inProgress === InteractionStatus.None) {
            const accessTokenRequest = {
                scopes: ["user.read"],
                account: accounts[0]
            }
            instance.acquireTokenSilent(accessTokenRequest).then((accessTokenResponse) => {
                // Acquire token silent success
                let accessToken = accessTokenResponse.accessToken;
                // Call your API with token
                callApi(accessToken).then((response) => { setApiData(response) });
            }).catch((error) => {
                if (error instanceof InteractionRequiredAuthError) {
                    instance.acquireTokenPopup(accessTokenRequest).then(function(accessTokenResponse) {
                        // Acquire token interactive success
                        let accessToken = accessTokenResponse.accessToken;
                        // Call your API with token
                        callApi(accessToken).then((response) => { setApiData(response) });
                    }).catch(function(error) {
                        // Acquire token interactive failure
                        console.log(error);
                    });
                }
                console.log(error);
            })
        }
    }, [instance, accounts, inProgress, apiData]);

    return <p>Return your protected content here: {apiData}</p>
}

function App() {
    return (
        <AuthenticatedTemplate>
            <ProtectedComponent />
        </ AuthenticatedTemplate>
    )
}
```

Como alternativa, si necesita adquirir un token fuera de un componente de React, puede llamar a `acquireTokenSilent`, pero no debe recurrir a la interacción si se produce un error. Toda la interacción debe tener lugar bajo el componente `MsalProvider` en el árbol de componentes.

```javascript
// MSAL.js v2 exposes several account APIs, logic to determine which account to use is the responsibility of the developer
const account = publicClientApplication.getAllAccounts()[0];

const accessTokenRequest = {
    scopes: ["user.read"],
    account: account
}

// Use the same publicClientApplication instance provided to MsalProvider
publicClientApplication.acquireTokenSilent(accessTokenRequest).then(function(accessTokenResponse) {
    // Acquire token silent success
    let accessToken = accessTokenResponse.accessToken;
    // Call your API with token
    callApi(accessToken);
}).catch(function (error) {
    //Acquire token silent failure
    console.log(error);
});
```

---

## <a name="acquire-a-token-with-a-redirect"></a>Adquisición de un token con una redirección

# <a name="javascript-msaljs-v2"></a>[JavaScript (MSAL.js v2)](#tab/javascript2)

El siguiente patrón es como se describió anteriormente, pero se muestra con un método de redirección para adquirir tokens de manera interactiva. Tendrá que llamar a `handleRedirectPromise` y esperar cuando se cargue la página.

```javascript
const redirectResponse = await publicClientApplication.handleRedirectPromise();
if (redirectResponse !== null) {
    // Acquire token silent success
    let accessToken = redirectResponse.accessToken;
    // Call your API with token
    callApi(accessToken);
} else {
    // MSAL.js v2 exposes several account APIs, logic to determine which account to use is the responsibility of the developer
    const account = publicClientApplication.getAllAccounts()[0];
    
    const accessTokenRequest = {
        scopes: ["user.read"],
        account: account
    }
    
    publicClientApplication.acquireTokenSilent(accessTokenRequest).then(function(accessTokenResponse) {
        // Acquire token silent success
        // Call API with token
        let accessToken = accessTokenResponse.accessToken;
        // Call your API with token
        callApi(accessToken);
    }).catch(function (error) {
        //Acquire token silent failure, and send an interactive request
        console.log(error);
        if (error instanceof InteractionRequiredAuthError) {
            publicClientApplication.acquireTokenRedirect(accessTokenRequest);
        }
    });
}
```

# <a name="javascript-msaljs-v1"></a>[JavaScript (MSAL.js v1)](#tab/javascript1)

El siguiente patrón es como se describió anteriormente, pero se muestra con un método de redirección para adquirir tokens de manera interactiva. Como se mencionó anteriormente, tendrá que registrar la devolución de llamada de redirección.

```javascript
function authCallback(error, response) {
    // Handle redirect response
}

userAgentApplication.handleRedirectCallback(authCallback);

const accessTokenRequest: AuthenticationParameters = {
    scopes: ["user.read"]
}

userAgentApplication.acquireTokenSilent(accessTokenRequest).then(function(accessTokenResponse) {
    // Acquire token silent success
    // Call API with token
    let accessToken = accessTokenResponse.accessToken;
}).catch(function (error) {
    //Acquire token silent failure, and send an interactive request
    console.log(error);
    if (error.errorMessage.indexOf("interaction_required") !== -1) {
        userAgentApplication.acquireTokenRedirect(accessTokenRequest);
    }
});
```

## <a name="request-optional-claims"></a>Solicitud de notificaciones opcionales

Puede usar notificaciones opcionales con los siguientes fines:

- Incluir notificaciones adicionales en los tokens de la aplicación.
- Cambiar el comportamiento de ciertas notificaciones que Azure AD devuelve en tokens.
- Agregar notificaciones personalizadas para la aplicación y acceder a ellas.

Para solicitar notificaciones opcionales en `IdToken`, puede enviar un objeto de notificaciones en cadena al campo `claimsRequest` de la clase `AuthenticationParameters.ts`.

```javascript
"optionalClaims":
   {
      "idToken": [
            {
                  "name": "auth_time",
                  "essential": true
             }
      ],

var request = {
    scopes: ["user.read"],
    claimsRequest: JSON.stringify(claims)
};

myMSALObj.acquireTokenPopup(request);
```

Para más información, consulte [Notificaciones opcionales](active-directory-optional-claims.md).

# <a name="angular-msaljs-v2"></a>[Angular (MSAL.js v2)](#tab/angular2)

Este código es el mismo que se describió anteriormente, salvo que se recomienda arrancar `MsalRedirectComponent` para controlar los redireccionamientos. También se pueden cambiar las configuraciones de `MsalInterceptor` para usar los redireccionamientos.

```javascript
// In app.module.ts
import { PublicClientApplication, InteractionType } from '@azure/msal-browser';
import { MsalInterceptor, MsalModule, MsalRedirectComponent } from '@azure/msal-angular';

@NgModule({
    declarations: [
      // ...
    ],
    imports: [
        // ...
        MsalModule.forRoot( new PublicClientApplication({
            auth: {
                clientId: 'Enter_the_Application_Id_Here',
            },
            cache: {
                cacheLocation: 'localStorage',
                storeAuthStateInCookie: isIE,
            }
        }), {
            interactionType: InteractionType.Redirect,
            authRequest: {
                scopes: ['user.read']
            }
        }, {
            interactionType: InteractionType.Redirect,
            protectedResourceMap: new Map([ 
                ['https://graph.microsoft.com/v1.0/me', ['user.read']]
            ])
        })
    ],
    providers: [
        {
            provide: HTTP_INTERCEPTORS,
            useClass: MsalInterceptor,
            multi: true
        }
    ],
    bootstrap: [AppComponent, MsalRedirectComponent]
})
export class AppModule { }
```

# <a name="angular-msaljs-v1"></a>[Angular (MSAL.js v1)](#tab/angular1)
Este código es igual al que se ha descrito anteriormente.

# <a name="react"></a>[React](#tab/react)

Si se produce un error en `acquireTokenSilent`, recurra a `acquireTokenRedirect`. Este método iniciará un redireccionamiento de marco completo y la respuesta se controlará al volver a la aplicación. Cuando este componente se representa después de volver del redireccionamiento, `acquireTokenSilent` ahora debe ejecutarse correctamente, ya que los tokens se extraen de la memoria caché.

```javascript
import { InteractionRequiredAuthError, InteractionStatus } from "@azure/msal-browser";
import { AuthenticatedTemplate, useMsal } from "@azure/msal-react";

function ProtectedComponent() {
    const { instance, inProgress, accounts } = useMsal();
    const [apiData, setApiData] = useState(null);


    useEffect(() => {
        const accessTokenRequest = {
            scopes: ["user.read"],
            account: accounts[0]
        }
        if (!apiData && inProgress === InteractionStatus.None) {
            instance.acquireTokenSilent(accessTokenRequest).then((accessTokenResponse) => {
                // Acquire token silent success
                let accessToken = accessTokenResponse.accessToken;
                // Call your API with token
                callApi(accessToken).then((response) => { setApiData(response) });
            }).catch((error) => {
                if (error instanceof InteractionRequiredAuthError) {
                    instance.acquireTokenRedirect(accessTokenRequest);
                }
                console.log(error);
            })
        }
    }, [instance, accounts, inProgress, apiData]);

    return <p>Return your protected content here: {apiData}</p>
}

function App() {
    return (
        <AuthenticatedTemplate>
            <ProtectedComponent />
        </ AuthenticatedTemplate>
    )
}
```

Como alternativa, si necesita adquirir un token fuera de un componente de React, puede llamar a `acquireTokenSilent`, pero no debe recurrir a la interacción si se produce un error. Toda la interacción debe tener lugar bajo el componente `MsalProvider` en el árbol de componentes.

```javascript
// MSAL.js v2 exposes several account APIs, logic to determine which account to use is the responsibility of the developer
const account = publicClientApplication.getAllAccounts()[0];

const accessTokenRequest = {
    scopes: ["user.read"],
    account: account
}

// Use the same publicClientApplication instance provided to MsalProvider
publicClientApplication.acquireTokenSilent(accessTokenRequest).then(function(accessTokenResponse) {
    // Acquire token silent success
    let accessToken = accessTokenResponse.accessToken;
    // Call your API with token
    callApi(accessToken);
}).catch(function (error) {
    //Acquire token silent failure
    console.log(error);
});
```

---

## <a name="next-steps"></a>Pasos siguientes

Avance al siguiente artículo de este escenario, [Llamada a una API web](scenario-spa-call-api.md).
