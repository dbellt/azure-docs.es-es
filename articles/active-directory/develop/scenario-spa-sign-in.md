---
title: Inicio y cierre de sesión de una aplicación de página única
titleSuffix: Microsoft identity platform
description: Aprenda a crear una aplicación de página única (inicio de sesión)
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 02/11/2020
ms.author: nacanuma
ms.custom: aaddev
ms.openlocfilehash: bf1fde475227ae1735a19016e6ecb69d20fb9281
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107947526"
---
# <a name="single-page-application-sign-in-and-sign-out"></a>Aplicación de página única: Inicio y cierre de sesión

Aprenda a agregar el inicio de sesión al código de la aplicación de página única.

Antes de obtener tokens para acceder a las API de la aplicación, necesita un contexto de usuario autenticado. Puede iniciar la sesión de los usuarios en la aplicación con MSAL.js de dos maneras:

* [Ventana emergente](#sign-in-with-a-pop-up-window), mediante el método `loginPopup`
* [Redirección](#sign-in-with-redirect), mediante el método `loginRedirect`

Opcionalmente, también puede pasar los ámbitos de las API para las que necesita que el usuario dé su consentimiento en el momento de iniciar sesión.

> [!NOTE]
> Si su aplicación ya tiene acceso a un contexto de usuario autenticado o un token de identificación, puede omitir el paso de inicio de sesión y adquirir los tokens directamente. Para más información, consulte [Inicio de sesión único sin inicio de sesión en MSAL.js](msal-js-sso.md#sso-without-msaljs-login).

## <a name="choosing-between-a-pop-up-or-redirect-experience"></a>Elegir entre una experiencia de elemento emergente o redireccionamiento

La elección de una u otra experiencia depende del flujo de la aplicación:

* Si no quiere que los usuarios salgan de la página principal de la aplicación durante la autenticación, se recomienda el método de ventana emergente. Dado que la redirección de la autenticación tiene lugar en una ventana emergente, se conserva el estado de la aplicación principal.

* Si los usuarios tienen restricciones del explorador o directivas donde las ventanas emergentes están deshabilitadas, puede usar el método de redirección. Use el método de redirección con Internet Explorer, ya que hay [problemas conocidos con las ventanas emergentes en este explorador](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/internet-explorer.md#popups).

## <a name="sign-in-with-a-pop-up-window"></a>Inicio de sesión con una ventana emergente

# <a name="javascript-msaljs-v2"></a>[JavaScript (MSAL.js v2)](#tab/javascript2)

```javascript

const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const loginRequest = {
    scopes: ["User.ReadWrite"]
}

let username = "";

const myMsal = new PublicClientApplication(config);

myMsal.loginPopup(loginRequest)
    .then(function (loginResponse) {
        //login success

        // In case multiple accounts exist, you can select
        const currentAccounts = myMsal.getAllAccounts();
    
        if (currentAccounts === null) {
            // no accounts detected
        } else if (currentAccounts.length > 1) {
            // Add choose account code here
        } else if (currentAccounts.length === 1) {
            username = currentAccounts[0].username;
        }
    
    }).catch(function (error) {
        //login failure
        console.log(error);
    });
```

# <a name="javascript-msaljs-v1"></a>[JavaScript (MSAL.js v1)](#tab/javascript1)

```javascript

const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const loginRequest = {
    scopes: ["User.ReadWrite"]
}

const myMsal = new UserAgentApplication(config);

myMsal.loginPopup(loginRequest)
    .then(function (loginResponse) {
        //login success
    }).catch(function (error) {
        //login failure
        console.log(error);
    });
```

# <a name="angular-msaljs-v2"></a>[Angular (MSAL.js v2)](#tab/angular2)

El contenedor MSAL Angular le permite proteger rutas específicas de la aplicación con solo agregar el valor `MsalGuard` a la definición de ruta. Esta protección invocará al método para iniciar sesión cuando se tenga acceso a esa ruta.

```javascript
// In app-routing.module.ts
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { ProfileComponent } from './profile/profile.component';
import { MsalGuard } from '@azure/msal-angular';
import { HomeComponent } from './home/home.component';

const routes: Routes = [
    {
        path: 'profile',
        component: ProfileComponent,
        canActivate: [
            MsalGuard
        ]
    },
    {
        path: '',
        component: HomeComponent
    }
];

@NgModule({
    imports: [RouterModule.forRoot(routes, { useHash: false })],
    exports: [RouterModule]
})
export class AppRoutingModule { }
```

Para obtener una experiencia de ventana emergente, establezca la configuración de `interactionType` en `InteractionType.Popup` en la configuración de Guard. También puede pasar los ámbitos que requieren el consentimiento tal como se detalla a continuación:

```javascript
// In app.module.ts
import { PublicClientApplication, InteractionType } from '@azure/msal-browser';
import { MsalModule } from '@azure/msal-angular';

@NgModule({
    imports: [
        MsalModule.forRoot( new PublicClientApplication({
            auth: {
                clientId: 'Enter_the_Application_Id_Here',
            },
            cache: {
                cacheLocation: 'localStorage',
                storeAuthStateInCookie: isIE,
            }
        }), {
            interactionType: InteractionType.Popup, // Msal Guard Configuration
            authRequest: {
                scopes: ['user.read']
            }
        }, null)
    ]
})
export class AppModule { }
```

# <a name="angular-msaljs-v1"></a>[Angular (MSAL.js v1)](#tab/angular1)
El contenedor MSAL Angular le permite proteger rutas específicas de la aplicación con solo agregar el valor `MsalGuard` a la definición de ruta. Esta protección invocará al método para iniciar sesión cuando se tenga acceso a esa ruta.
```javascript
// In app-routing.module.ts
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { ProfileComponent } from './profile/profile.component';
import { MsalGuard } from '@azure/msal-angular';
import { HomeComponent } from './home/home.component';
const routes: Routes = [
  {
    path: 'profile',
    component: ProfileComponent,
    canActivate: [
      MsalGuard
    ]
  },
  {
    path: '',
    component: HomeComponent
  }
];
@NgModule({
  imports: [RouterModule.forRoot(routes, { useHash: false })],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

Para usar la experiencia de ventana emergente, habilite la opción de configuración `popUp`. También puede pasar los ámbitos que requieren el consentimiento tal como se detalla a continuación:

```javascript
// In app.module.ts
@NgModule({
    imports: [
        MsalModule.forRoot({
            auth: {
                clientId: 'your_app_id',
            }
        }, {
            popUp: true,
            consentScopes: ["User.ReadWrite"]
        })
    ]
})
```

# <a name="react"></a>[React](#tab/react)

El contenedor React de MSAL permite proteger componentes específicos al encapsularlos en el componente `MsalAuthenticationTemplate`. Este componente invocará el inicio de sesión si un usuario aún no ha iniciado sesión o representará los componentes secundarios en caso contrario.

```javascript
import { InteractionType } from "@azure/msal-browser";
import { MsalAuthenticationTemplate, useMsal } from "@azure/msal-react";

function WelcomeUser() {
    const { accounts } = useMsal();
    const username = accounts[0].username;
    
    return <p>Welcome, {username}</p>
}

// Remember that MsalProvider must be rendered somewhere higher up in the component tree
function App() {
    return (
        <MsalAuthenticationTemplate interactionType={InteractionType.Popup}>
            <p>This will only render if a user is signed-in.</p>
            <WelcomeUser />
        </MsalAuthenticationTemplate>
      )
};
```

También puede usar las API de `@azure/msal-browser` directamente para invocar un inicio de sesión emparejado con los componentes `AuthenticatedTemplate` y/o `UnauthenticatedTemplate` para representar contenido específico para los usuarios que han iniciado sesión o que la han cerrado, respectivamente. Este es el enfoque recomendado si necesita invocar el inicio de sesión como resultado de la interacción del usuario, como un clic de botón.

```javascript
import { useMsal, AuthenticatedTemplate, UnauthenticatedTemplate } from "@azure/msal-react";

function signInClickHandler(instance) {
    instance.loginPopup();
}

// SignInButton Component returns a button that invokes a popup login when clicked
function SignInButton() {
    // useMsal hook will return the PublicClientApplication instance you provided to MsalProvider
    const { instance } = useMsal();

    return <button onClick={() => signInClickHandler(instance)}>Sign In</button>
};

function WelcomeUser() {
    const { accounts } = useMsal();
    const username = accounts[0].username;
    
    return <p>Welcome, {username}</p>
}

// Remember that MsalProvider must be rendered somewhere higher up in the component tree
function App() {
    return (
        <>
            <AuthenticatedTemplate>
                <p>This will only render if a user is signed-in.</p>
                <WelcomeUser />
            </AuthenticatedTemplate>
            <UnauthenticatedTemplate>
                <p>This will only render if a user is not signed-in.</p>
                <SignInButton />
            </UnauthenticatedTemplate>
        </>
    )
}
```

---

## <a name="sign-in-with-redirect"></a>Inicio de sesión con redirección

# <a name="javascript-msaljs-v2"></a>[JavaScript (MSAL.js v2)](#tab/javascript2)

```javascript

const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const loginRequest = {
    scopes: ["User.ReadWrite"]
}

let username = "";

const myMsal = new PublicClientApplication(config);

function handleResponse(response) {
    //handle redirect response

    // In case multiple accounts exist, you can select
    const currentAccounts = myMsal.getAllAccounts();

    if (currentAccounts === null) {
        // no accounts detected
    } else if (currentAccounts.length > 1) {
        // Add choose account code here
    } else if (currentAccounts.length === 1) {
        username = currentAccounts[0].username;
    }
}

myMsal.handleRedirectPromise().then(handleResponse);

myMsal.loginRedirect(loginRequest);
```

# <a name="javascript-msaljs-v1"></a>[JavaScript (MSAL.js v1)](#tab/javascript1)

Los métodos de redirección no devuelven una promesa, dado que se abandona la aplicación principal. Para procesar los tokens devueltos y acceder a ellos, registre las devoluciones de llamada correctas y con error antes de llamar a los métodos de redirección.

```javascript

const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const loginRequest = {
    scopes: ["User.ReadWrite"]
}

const myMsal = new UserAgentApplication(config);

function authCallback(error, response) {
    //handle redirect response
}

myMsal.handleRedirectCallback(authCallback);

myMsal.loginRedirect(loginRequest);
```

# <a name="angular-msaljs-v2"></a>[Angular (MSAL.js v2)](#tab/angular2)

El código aquí es el mismo que se describió anteriormente en la sección sobre el inicio de sesión con una ventana emergente, salvo que `interactionType` está establecido en `InteractionType.Redirect` para la configuración de MsalGuard y se arranca `MsalRedirectComponent` para controlar los redireccionamientos.

```javascript
// In app.module.ts
import { PublicClientApplication, InteractionType } from '@azure/msal-browser';
import { MsalModule, MsalRedirectComponent } from '@azure/msal-angular';

@NgModule({
    imports: [
        MsalModule.forRoot( new PublicClientApplication({
            auth: {
                clientId: 'Enter_the_Application_Id_Here',
            },
            cache: {
                cacheLocation: 'localStorage',
                storeAuthStateInCookie: isIE,
            }
        }), {
            interactionType: InteractionType.Redirect, // Msal Guard Configuration
            authRequest: {
                scopes: ['user.read']
            }
        }, null)
    ],
    bootstrap: [AppComponent, MsalRedirectComponent]
})
export class AppModule { }
```

# <a name="angular-msaljs-v1"></a>[Angular (MSAL.js v1)](#tab/angular1)

El código aquí es el mismo que el que se describió anteriormente en la sección sobre el inicio de sesión con una ventana emergente. El flujo predeterminado es de redirección.

# <a name="react"></a>[React](#tab/react)

El contenedor React de MSAL permite proteger componentes específicos al encapsularlos en el componente `MsalAuthenticationTemplate`. Este componente invocará el inicio de sesión si un usuario aún no ha iniciado sesión o representará los componentes secundarios en caso contrario.

```javascript
import { InteractionType } from "@azure/msal-browser";
import { MsalAuthenticationTemplate, useMsal } from "@azure/msal-react";

function WelcomeUser() {
    const { accounts } = useMsal();
    const username = accounts[0].username;
    
    return <p>Welcome, {username}</p>
}

// Remember that MsalProvider must be rendered somewhere higher up in the component tree
function App() {
    return (
        <MsalAuthenticationTemplate interactionType={InteractionType.Redirect}>
            <p>This will only render if a user is signed-in.</p>
            <WelcomeUser />
        </MsalAuthenticationTemplate>
      )
};
```

También puede usar las API de `@azure/msal-browser` directamente para invocar un inicio de sesión emparejado con los componentes `AuthenticatedTemplate` y/o `UnauthenticatedTemplate` para representar contenido específico para los usuarios que han iniciado sesión o que la han cerrado, respectivamente. Este es el enfoque recomendado si necesita invocar el inicio de sesión como resultado de la interacción del usuario, como un clic de botón.

```javascript
import { useMsal, AuthenticatedTemplate, UnauthenticatedTemplate } from "@azure/msal-react";

function signInClickHandler(instance) {
    instance.loginRedirect();
}

// SignInButton Component returns a button that invokes a popup login when clicked
function SignInButton() {
    // useMsal hook will return the PublicClientApplication instance you provided to MsalProvider
    const { instance } = useMsal();

    return <button onClick={() => signInClickHandler(instance)}>Sign In</button>
};

function WelcomeUser() {
    const { accounts } = useMsal();
    const username = accounts[0].username;
    
    return <p>Welcome, {username}</p>
}

// Remember that MsalProvider must be rendered somewhere higher up in the component tree
function App() {
    return (
        <>
            <AuthenticatedTemplate>
                <p>This will only render if a user is signed-in.</p>
                <WelcomeUser />
            </AuthenticatedTemplate>
            <UnauthenticatedTemplate>
                <p>This will only render if a user is not signed-in.</p>
                <SignInButton />
            </UnauthenticatedTemplate>
        </>
    )
}
```

---

## <a name="sign-out-with-a-popup-window"></a>Cierre de sesión con una ventana emergente

MSAL.js v2 proporciona un método `logoutPopup` que borra la memoria caché en el almacenamiento del explorador y abre una ventana emergente a la página de cierre de sesión de Azure Active Directory (Azure AD). Después de cerrar la sesión, Azure AD redirige el elemento emergente a la aplicación y MSAL.js lo cierra.

Puede configurar el URI al que Azure AD debe realizar el redireccionamiento tras el cierre de sesión. Para ello, establezca `postLogoutRedirectUri`. Este URI se debe registrar como un URI de redireccionamiento en el registro de la aplicación.

También puede configurar `logoutPopup` para redirigir la ventana principal a otra página, como la página principal o la página de inicio de sesión, una vez completado el cierre de sesión mediante el paso de `mainWindowRedirectUri` como parte de la solicitud.

# <a name="javascript-msaljs-v2"></a>[JavaScript (MSAL.js v2)](#tab/javascript2)

```javascript
const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", // defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const myMsal = new PublicClientApplication(config);

// you can select which account application should sign out
const logoutRequest = {
    account: myMsal.getAccountByHomeId(homeAccountId),
    mainWindowRedirectUri: "your_app_main_window_redirect_uri"
}

await myMsal.logoutPopup(logoutRequest);
```
# <a name="javascript-msaljs-v1"></a>[JavaScript (MSAL.js v1)](#tab/javascript1)

No se admite el cierre de sesión con una ventana emergente en MSAL.js v1

# <a name="angular-msaljs-v2"></a>[Angular (MSAL.js v2)](#tab/angular2)

```javascript
// In app.module.ts
@NgModule({
    imports: [
        MsalModule.forRoot( new PublicClientApplication({
            auth: {
                clientId: 'your_app_id',
                postLogoutRedirectUri: 'your_app_logout_redirect_uri'
            }
        }), null, null)
    ]
})

// In app.component.ts
logout() {
    this.authService.logoutPopup({
        mainWindowRedirectUri: "/"
    });
}
```

# <a name="angular-msaljs-v1"></a>[Angular (MSAL.js v1)](#tab/angular1)

No se admite el cierre de sesión con una ventana emergente en MSAL Angular v1

# <a name="react"></a>[React](#tab/react)

```javascript
import { useMsal, AuthenticatedTemplate, UnauthenticatedTemplate } from "@azure/msal-react";

function signOutClickHandler(instance) {
    const logoutRequest = {
        account: instance.getAccountByHomeId(homeAccountId),
        mainWindowRedirectUri: "your_app_main_window_redirect_uri",
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
    instance.logoutPopup(logoutRequest);
}

// SignOutButton Component returns a button that invokes a popup logout when clicked
function SignOutButton() {
    // useMsal hook will return the PublicClientApplication instance you provided to MsalProvider
    const { instance } = useMsal();

    return <button onClick={() => signOutClickHandler(instance)}>Sign Out</button>
};

// Remember that MsalProvider must be rendered somewhere higher up in the component tree
function App() {
    return (
        <>
            <AuthenticatedTemplate>
                <p>This will only render if a user is signed-in.</p>
                <SignOutButton />
            </AuthenticatedTemplate>
            <UnauthenticatedTemplate>
                <p>This will only render if a user is not signed-in.</p>
            </UnauthenticatedTemplate>
        </>
    )
}
```

---

## <a name="sign-out-with-a-redirect"></a>Cierre de sesión con un redireccionamiento

MSAL.js proporciona un método `logout` en v1 (y un método `logoutRedirect` en v2), que borra la memoria caché en el almacenamiento del explorador y redirige la ventana a la página de cierre de sesión de Azure Active Directory (Azure AD). Después de cerrar sesión, Azure AD realiza el redireccionamiento a la página que invocó el cierre de sesión de forma predeterminada.

Puede configurar el URI de redirección tras el cierre de sesión mediante el establecimiento del valor `postLogoutRedirectUri`. Este URI se debe registrar como un URI de redireccionamiento en el registro de la aplicación.

# <a name="javascript-msaljs-v2"></a>[JavaScript (MSAL.js v2)](#tab/javascript2)

```javascript
const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const myMsal = new PublicClientApplication(config);

// you can select which account application should sign out
const logoutRequest = {
    account: myMsal.getAccountByHomeId(homeAccountId)
}

myMsal.logoutRedirect(logoutRequest);
```

# <a name="javascript-msaljs-v1"></a>[JavaScript (MSAL.js v1)](#tab/javascript1)

```javascript
const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const myMsal = new UserAgentApplication(config);

myMsal.logout();
```

# <a name="angular-msaljs-v2"></a>[Angular (MSAL.js v2)](#tab/angular2)

```javascript
// In app.module.ts
@NgModule({
    imports: [
        MsalModule.forRoot( new PublicClientApplication({
            auth: {
                clientId: 'your_app_id',
                postLogoutRedirectUri: 'your_app_logout_redirect_uri'
            }
        }), null, null)
    ]
})

// In app.component.ts
logout() {
    this.authService.logoutRedirect();
}
```

# <a name="angular-msaljs-v1"></a>[Angular (MSAL.js v1)](#tab/angular1)

```javascript
//In app.module.ts
@NgModule({
    imports: [
        MsalModule.forRoot({
            auth: {
                clientId: 'your_app_id',
                postLogoutRedirectUri: "your_app_logout_redirect_uri"
            }
        })
    ]
})
// In app.component.ts
this.authService.logout();
```

# <a name="react"></a>[React](#tab/react)

```javascript
import { useMsal, AuthenticatedTemplate, UnauthenticatedTemplate } from "@azure/msal-react";

function signOutClickHandler(instance) {
    const logoutRequest = {
        account: instance.getAccountByHomeId(homeAccountId),
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
    instance.logoutRedirect(logoutRequest);
}

// SignOutButton Component returns a button that invokes a redirect logout when clicked
function SignOutButton() {
    // useMsal hook will return the PublicClientApplication instance you provided to MsalProvider
    const { instance } = useMsal();

    return <button onClick={() => signOutClickHandler(instance)}>Sign Out</button>
};

// Remember that MsalProvider must be rendered somewhere higher up in the component tree
function App() {
    return (
        <>
            <AuthenticatedTemplate>
                <p>This will only render if a user is signed-in.</p>
                <SignOutButton />
            </AuthenticatedTemplate>
            <UnauthenticatedTemplate>
                <p>This will only render if a user is not signed-in.</p>
            </UnauthenticatedTemplate>
        </>
    )
}
```

---

## <a name="next-steps"></a>Pasos siguientes

Avance al siguiente artículo de este escenario, [Obtención de un token para la aplicación](scenario-spa-acquire-token.md).
