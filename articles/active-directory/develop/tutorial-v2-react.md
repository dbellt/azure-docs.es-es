---
title: 'Tutorial: Creación de una aplicación de página única de React que usa el flujo de código de autorización | Azure'
titleSuffix: Microsoft identity platform
description: En este tutorial, creará una aplicación de página única de React con la que los usuarios pueden iniciar sesión y usar el flujo de código de autenticación para obtener un token de acceso de la Plataforma de identidad de Microsoft y llamar a Microsoft Graph API.
services: active-directory
author: j-mantu
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 04/16/2021
ms.author: jamesmantu
ms.custom: aaddev, devx-track-js
ms.openlocfilehash: 1b8e822c72bdf2af8b20950944fb93686524b797
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112072350"
---
# <a name="tutorial-sign-in-users-and-call-the-microsoft-graph-api-from-a-react-single-page-app-spa-using-auth-code-flow"></a>Tutorial: Inicio de sesión de usuarios y llamada a Microsoft Graph API desde una aplicación de página única (SPA) de React mediante el flujo de código de autorización

En este tutorial, creará una aplicación de página única (SPA) de React que inicia la sesión de los usuarios y llama a Microsoft Graph mediante el flujo de código de autorización con PKCE. La aplicación de página única que cree usa la biblioteca de autenticación de Microsoft (MSAL) para React.

En este tutorial:
> [!div class="checklist"]
> * Crear un proyecto de React con `npm`
> * Registrar la aplicación en Azure Portal
> * Agregar código para admitir el inicio y el cierre de sesión de usuario
> * Agregar código para llamar a Microsoft Graph API
> * Prueba de la aplicación

MSAL React admite el flujo de código de autorización en el explorador en lugar del flujo de concesión implícito. MSAL React **NO** admite el flujo implícito.

## <a name="prerequisites"></a>Requisitos previos

* [Node.js](https://nodejs.org/en/download/) para ejecutar un servidor web local
* [Visual Studio Code](https://code.visualstudio.com/download) u otro editor de código

## <a name="how-the-tutorial-app-works"></a>Cómo funciona la aplicación tutorial

:::image type="content" source="media/tutorial-v2-javascript-auth-code/diagram-01-auth-code-flow.png" alt-text="Diagrama que muestra el flujo de código de autorización para una aplicación de página única":::

La aplicación que se crea en este tutorial permite que una aplicación de página única de React consulte Microsoft Graph API mediante la adquisición de tokens de seguridad de la Plataforma de identidad de Microsoft. Usa la Biblioteca de autenticación de Microsoft (MSAL) para React, un contenedor de la biblioteca de MSAL.js v2. MSAL React permite que las aplicaciones creadas con React 16, y las versiones posteriores, autentiquen a usuarios empresariales mediante Azure Active Directory, a usuarios de cuentas de Microsoft y a usuarios de identidades de redes sociales como Facebook, Google y LinkedIn. La biblioteca también permite que las aplicaciones obtengan acceso a los servicios en la nube de Microsoft y a Microsoft Graph.

En este escenario, después de que un usuario inicia sesión, se solicita un token de acceso y se agrega a las solicitudes HTTP mediante el encabezado de autorización. La adquisición y la renovación de los tokens se realiza por medio de la biblioteca de autenticación de Microsoft para React (MSAL React).

### <a name="libraries"></a>Bibliotecas

En este tutorial se usan las siguientes bibliotecas:

|Biblioteca|Descripción|
|---|---|
|[MSAL React](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-react)|Biblioteca de autenticación de Microsoft para el contenedor de React de JavaScript|
|[Explorador MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)|Biblioteca de autenticación de Microsoft para el paquete del explorador de JavaScript v2|

## <a name="get-the-completed-code-sample"></a>Obtención del ejemplo de código completado

¿Prefiere descargar en su lugar este proyecto de ejemplo completado del tutorial? Para ejecutar el proyecto mediante un servidor web local, como Node.js, clone el repositorio [ms-identity-javascript-react-spa](https://github.com/Azure-Samples/ms-identity-javascript-react-spa):

`git clone https://github.com/Azure-Samples/ms-identity-javascript-react-spa`

Después, para configurar el ejemplo de código antes de ejecutarlo, vaya al [paso de configuración](#register-your-application).

Para continuar con el tutorial y crear la aplicación, vaya a la sección siguiente, [Prerrequisitos](#prerequisites).

## <a name="create-your-project"></a>Creación del proyecto

Una vez que [Node.js](https://nodejs.org/en/download/) esté instalado, abra una ventana del terminal y ejecute los siguientes comandos:

```console
npx create-react-app msal-react-tutorial # Create a new React app
cd msal-react-tutorial # Change to the app directory
npm install @azure/msal-browser @azure/msal-react # Install the MSAL packages
npm install react-bootstrap bootstrap # Install Bootstrap for styling
```

Ya ha arrancado un pequeño proyecto de React mediante [Create React App](https://create-react-app.dev/docs/getting-started). Este será el punto de partida en el que se basará el resto del tutorial. Si desea ver los cambios que se producen en la aplicación a medida que avanza en el tutorial, puede ejecutar el siguiente comando: 

```console
npm start
```

Se debe abrir automáticamente una ventana del explorador para la aplicación. Si no es así, abra el explorador y vaya a http://localhost:3000. Cada vez que guarde un archivo con código actualizado, la página se volverá a cargar para reflejar los cambios.

## <a name="register-your-application"></a>Registro de la aplicación

Siga los pasos que encontrará en [Aplicación de una sola página: Registro de aplicación](./scenario-spa-app-registration.md) para crear un registro de aplicación para la aplicación de página única desde Azure Portal.

En el paso [URI de redireccionamiento: MSAL.js 2.0 con flujo de código de autenticación](scenario-spa-app-registration.md#redirect-uri-msaljs-20-with-auth-code-flow), escriba `http://localhost:3000`, que es la ubicación predeterminada en la que create-react-app servirá a su aplicación.

### <a name="configure-your-javascript-spa"></a>Configuración de JavaScript SPA

1. Cree en la carpeta *src* un archivo llamado *authConfig.js* que contenga los parámetros de configuración de la autenticación y agregue el código siguiente:

    ```javascript
    export const msalConfig = {
      auth: {
        clientId: "Enter_the_Application_Id_Here",
        authority: "Enter_the_Cloud_Instance_Id_Here/Enter_the_Tenant_Info_Here",
        redirectUri: "Enter_the_Redirect_Uri_Here",
      },
      cache: {
        cacheLocation: "sessionStorage", // This configures where your cache will be stored
        storeAuthStateInCookie: false, // Set this to "true" if you are having issues on IE11 or Edge
      }
    };
    
    // Add scopes here for ID token to be used at Microsoft identity platform endpoints.
    export const loginRequest = {
     scopes: ["User.Read"]
    };
    
    // Add the endpoints here for Microsoft Graph API services you'd like to use.
    export const graphConfig = {
        graphMeEndpoint: "Enter_the_Graph_Endpoint_Here/v1.0/me"
    };
    ```

1. Modifique los valores de la sección `msalConfig` como se describe aquí:
    
    |Nombre del valor| Información|
    |----------|------|
    |`Enter_the_Application_Id_Here`| el **identificador de la aplicación (cliente)** de la aplicación que registró.|
    |`Enter_the_Cloud_Instance_Id_Here`| Instancia en la nube de Azure en la que se registra la aplicación. En el caso de la nube principal (o *global*) de Azure, escriba `https://login.microsoftonline.com`. En el caso de las nubes **nacionales** (por ejemplo, China), puede encontrar los valores adecuados en las [nubes nacionales](authentication-national-cloud.md).
    |`Enter_the_Tenant_Info_Here`| Seleccione una de las siguientes opciones: Si la aplicación admite las *cuentas de este directorio organizativo*, reemplace este valor por el identificador del directorio (inquilino) o por el nombre del inquilino (por ejemplo, **contoso.microsoft.com**). Si la aplicación admite *cuentas en cualquier directorio organizativo*, reemplace este valor por **organizaciones**. Si la aplicación admite *cuentas en cualquier directorio organizativo y cuentas Microsoft personales*, reemplace este valor por **común**. Para restringir la compatibilidad a *Personal Microsoft accounts only* (Solo cuentas Microsoft personales), reemplace este valor por **consumidores**. |
    |`Enter_the_Redirect_Uri_Here`|Reemplace este valor por **http://localhost:3000**.|
    |`Enter_the_Graph_Endpoint_Here`| La instancia de Microsoft Graph API con la que la aplicación debe comunicarse. En el caso del punto de conexión de Microsoft Graph API **global**, reemplace ambas instancias de esta cadena por `https://graph.microsoft.com`. Para obtener información sobre los puntos de conexión en las implementaciones de nubes **nacionales**, consulte [Implementaciones de nube nacionales](/graph/deployments) en la documentación de Microsoft Graph.|
    
    Para más información acerca de las opciones configurables disponibles, consulte [Inicializar aplicaciones cliente](msal-js-initializing-client-applications.md).

1. Abra el archivo *src/index.js* y agregue las siguientes instrucciones import:

    ```javascript
    import "bootstrap/dist/css/bootstrap.min.css";
    import { PublicClientApplication } from "@azure/msal-browser";
    import { MsalProvider } from "@azure/msal-react";
    import { msalConfig } from "./authConfig";
    ```

1. Debajo de las instrucciones import de *src/index.js* cree una instancia de `PublicClientApplication` mediante la configuración del paso 1.

    ```javascript
    const msalInstance = new PublicClientApplication(msalConfig);
    ``` 

1. Busque el componente `<App />` en *src/index.js* y encapsúlelo en el componente `MsalProvider`. La función de representación debe ser similar a esta:

    ```jsx
    ReactDOM.render(
        <React.StrictMode>
            <MsalProvider instance={msalInstance}>
                <App />
            </MsalProvider>
        </React.StrictMode>,
        document.getElementById("root")
    );
    ``` 


## <a name="sign-in-users"></a>Inicio de sesión de usuarios

Cree una carpeta en *src* denominada *components*  y cree en ella el archivo *SignInButton.jsx*. Agregue el código de cualquiera de las secciones siguientes para invocar el inicio de sesión mediante una ventana emergente o un redireccionamiento de marco completo:

### <a name="sign-in-using-popups"></a>Inicie sesión mediante elementos emergentes

Agregue el siguiente código a *src/components/SignInButton.jsx* para crear un componente botón que invoque un inicio de sesión con elementos emergentes al seleccionarlo:

```jsx
import React from "react";
import { useMsal } from "@azure/msal-react";
import { loginRequest } from "../authConfig";
import Button from "react-bootstrap/Button";

function handleLogin(instance) {
    instance.loginPopup(loginRequest).catch(e => {
        console.error(e);
    });
}

/**
 * Renders a button which, when selected, will open a popup for login
 */
export const SignInButton = () => {
    const { instance } = useMsal();

    return (
        <Button variant="secondary" className="ml-auto" onClick={() => handleLogin(instance)}>Sign in using Popup</Button>
    );
}
```

### <a name="sign-in-using-redirects"></a>Inicio de sesión mediante redireccionamientos

Agregue el siguiente código a *src/components/SignInButton.jsx* para crear un componente botón que invoque un inicio de sesión mediante redireccionamiento al seleccionarlo:

```jsx
import React from "react";
import { useMsal } from "@azure/msal-react";
import { loginRequest } from "../authConfig";
import Button from "react-bootstrap/Button";

function handleLogin(instance) {
    instance.loginRedirect(loginRequest).catch(e => {
        console.error(e);
    });
}

/**
 * Renders a button which, when selected, will redirect the page to the login prompt
 */
export const SignInButton = () => {
    const { instance } = useMsal();

    return (
        <Button variant="secondary" className="ml-auto" onClick={() => handleLogin(instance)}>Sign in using Redirect</Button>
    );
}
```

### <a name="add-the-sign-in-button"></a>Incorporación del botón para iniciar sesión

1. Cree otro archivo en la carpeta *components* denominado *PageLayout.jsx* y agregue el código siguiente para crear un componente de barra de navegación que contendrá el botón de inicio de sesión que acaba de crear:

    ```jsx
    import React from "react";
    import Navbar from "react-bootstrap/Navbar";
    import { useIsAuthenticated } from "@azure/msal-react";
    import { SignInButton } from "./SignInButton";
    
    /**
     * Renders the navbar component with a sign-in button if a user is not authenticated
     */
    export const PageLayout = (props) => {
        const isAuthenticated = useIsAuthenticated();
    
        return (
            <>
                <Navbar bg="primary" variant="dark">
                    <a className="navbar-brand" href="/">MSAL React Tutorial</a>
                    { isAuthenticated ? <span>Signed In</span> : <SignInButton /> }
                </Navbar>
                <h5><center>Welcome to the Microsoft Authentication Library For React Tutorial</center></h5>
                <br />
                <br />
                {props.children}
            </>
        );
    };
    ```

2. Ahora abra *src/App.js* y reemplace el contenido existente por el código siguiente: 

    ```jsx
    import React from "react";
    import { PageLayout } from "./components/PageLayout";
    
    function App() {
      return (
          <PageLayout>
              <p>This is the main app content!</p>
          </PageLayout>
      );
    }
    
    export default App;
    ```

La aplicación ahora tiene un botón para iniciar sesión que solo se muestra a los usuarios no autenticados.

La primera vez que un usuario selecciona el botón **Sign in using Popup** (Iniciar sesión mediante elemento emergente) o **Sign in using Redirect** (Iniciar sesión mediante redireccionamiento), el controlador `onClick` llama a `loginPopup` (o a `loginRedirect`) para iniciar la sesión del usuario. Este método `loginPopup` hace que se abra una ventana emergente con el *punto de conexión de la Plataforma de identidad de Microsoft* para pedir y validar las credenciales del usuario. Después de un inicio de sesión correcto, *msal.js* inicia el [flujo de código de autorización](v2-oauth2-auth-code-flow.md).

En este momento, se envía un código de autorización protegido por PKCE al punto de conexión del token protegido con CORS y se intercambia por tokens. Se reciben un token de identificador, un token de acceso y un token de actualización por la aplicación y *msal.js* los procesa, y la información contenida en los tokens se almacena en caché.

## <a name="sign-users-out"></a>Cierre de sesión de usuarios

En *src/components*, cree un archivo denominado *SignOutButton.jsx*. Agregue el código de cualquiera de las secciones siguientes para invocar el cierre de sesión mediante una ventana emergente o un redireccionamiento de marco completo:

### <a name="sign-out-using-popups"></a>Cierre de sesión mediante elementos emergentes

Agregue el siguiente código a *src/components/SignOutButton.jsx* para crear un componente botón que invoque un cierre de sesión con elementos emergentes al seleccionarlo:

```jsx
import React from "react";
import { useMsal } from "@azure/msal-react";
import Button from "react-bootstrap/Button";

function handleLogout(instance) {
    instance.logoutPopup().catch(e => {
        console.error(e);
    });
}

/**
 * Renders a button which, when selected, will open a popup for logout
 */
export const SignOutButton = () => {
    const { instance } = useMsal();

    return (
        <Button variant="secondary" className="ml-auto" onClick={() => handleLogout(instance)}>Sign out using Popup</Button>
    );
}
```

### <a name="sign-out-using-redirects"></a>Cierre de sesión mediante redireccionamientos

Agregue el siguiente código a *src/components/SignOutButton.jsx* para crear un componente botón que invoque un cierre de sesión con direccionamientos al seleccionarlo:

```jsx
import React from "react";
import { useMsal } from "@azure/msal-react";
import Button from "react-bootstrap/Button";

function handleLogout(instance) {
    instance.logoutRedirect().catch(e => {
        console.error(e);
    });
}

/**
 * Renders a button which, when selected, will redirect the page to the logout prompt
 */
export const SignOutButton = () => {
    const { instance } = useMsal();

    return (
        <Button variant="secondary" className="ml-auto" onClick={() => handleLogout(instance)}>Sign out using Redirect</Button>
    );
}
```

### <a name="add-the-sign-out-button"></a>Adición del botón para cerrar sesión

Actualice el componente `PageLayout` en *src/components/PageLayout.jsx* para representar el nuevo componente `SignOutButton` para los usuarios autenticados. El código debe ser similar al siguiente:

```jsx
import React from "react";
import Navbar from "react-bootstrap/Navbar";
import { useIsAuthenticated } from "@azure/msal-react";
import { SignInButton } from "./SignInButton";
import { SignOutButton } from "./SignOutButton";

/**
 * Renders the navbar component with a sign-in button if a user is not authenticated
 */
export const PageLayout = (props) => {
    const isAuthenticated = useIsAuthenticated();

    return (
        <>
            <Navbar bg="primary" variant="dark">
                <a className="navbar-brand" href="/">MSAL React Tutorial</a>
                { isAuthenticated ? <SignOutButton /> : <SignInButton /> }
            </Navbar>
            <h5><center>Welcome to the Microsoft Authentication Library For React Tutorial</center></h5>
            <br />
            <br />
            {props.children}
        </>
    );
};
```

## <a name="conditionally-render-components"></a>Representación condicional de componentes

Para representar determinados componentes solo para usuarios autenticados o no autenticados, utilice `AuthenticateTemplate` o `UnauthenticatedTemplate`, como se muestra a continuación.

1. Agregue la siguiente instrucción import a *src/App.js*:

    ```javascript
    import { AuthenticatedTemplate, UnauthenticatedTemplate } from "@azure/msal-react";
    ```
    
1. Para representar determinados componentes solo para usuarios autenticados, actualice la función `App` de *src/App.js* con el código siguiente: 

    ```jsx
    function App() {
        return (
            <PageLayout>
                <AuthenticatedTemplate>
                    <p>You are signed in!</p>
                </AuthenticatedTemplate>
            </PageLayout>
        );
    }
    ```

1. Para representar determinados componentes solo para usuarios autenticados, como una sugerencia de inicio de sesión, actualice la función `App` de *src/App.js* con el código siguiente: 

    ```jsx
    function App() {
        return (
            <PageLayout>
                <AuthenticatedTemplate>
                    <p>You are signed in!</p>
                </AuthenticatedTemplate>
                <UnauthenticatedTemplate>
                    <p>You are not signed in! Please sign in.</p>
                </UnauthenticatedTemplate>
            </PageLayout>
        );
    }
    ```

## <a name="acquire-a-token"></a>Adquisición de un token

1. Antes de llamar a una API, como Microsoft Graph, deberá adquirir un token de acceso. Agregue un nuevo componente a *src/App.js* llamado `ProfileContent` con el código siguiente:

    ```jsx
    function ProfileContent() {
        const { instance, accounts, inProgress } = useMsal();
        const [accessToken, setAccessToken] = useState(null);
    
        const name = accounts[0] && accounts[0].name;
    
        function RequestAccessToken() {
            const request = {
                ...loginRequest,
                account: accounts[0]
            };
    
            // Silently acquires an access token which is then attached to a request for Microsoft Graph data
            instance.acquireTokenSilent(request).then((response) => {
                setAccessToken(response.accessToken);
            }).catch((e) => {
                instance.acquireTokenPopup(request).then((response) => {
                    setAccessToken(response.accessToken);
                });
            });
        }
    
        return (
            <>
                <h5 className="card-title">Welcome {name}</h5>
                {accessToken ? 
                    <p>Access Token Acquired!</p>
                    :
                    <Button variant="secondary" onClick={RequestAccessToken}>Request Access Token</Button>
                }
            </>
        );
    };
    ```

1. Actualice las instrucciones import en *src/App.js* para que coincidan con lo siguiente:

    ```js
    import React, { useState } from "react";
    import { PageLayout } from "./components/PageLayout";
    import { AuthenticatedTemplate, UnauthenticatedTemplate, useMsal } from "@azure/msal-react";
    import { loginRequest } from "./authConfig";
    import Button from "react-bootstrap/Button";
    ```

1. Por último, agregue el nuevo componente `ProfileContent` como elemento secundario de `AuthenticatedTemplate` en el componente `App` de *src/App.js*. El componente `App` debería ser similar al siguiente:

    ```javascript
    function App() {
      return (
          <PageLayout>
              <AuthenticatedTemplate>
                  <ProfileContent />
              </AuthenticatedTemplate>
              <UnauthenticatedTemplate>
                  <p>You are not signed in! Please sign in.</p>
              </UnauthenticatedTemplate>
          </PageLayout>
      );
    }
    ```

El código anterior representará un botón para los usuarios que han iniciado sesión, lo que les permitirá solicitar un token de acceso para Microsoft Graph cuando se selecciona el botón.

Una vez que un usuario inicia sesión, la aplicación no debería solicitarle que se vuelva a autenticar cada vez que necesite acceder a un recurso protegido (es decir, solicitar un token). Para evitar estas solicitudes de reautenticación, llame a `acquireTokenSilent`, que primero buscará un token de acceso que esté almacenado en la caché y no haya expirado y después, si fuera necesario, use el token de actualización para obtener un nuevo token de acceso. Sin embargo, puede haber situaciones en las que sea necesario hacer que los usuarios interactúen con la Plataforma de identidad de Microsoft. Por ejemplo:

- Los usuarios necesitan volver a escribir las credenciales porque la contraseña ha expirado.
- El token de actualización ha expirado.
- La aplicación solicita acceso a un recurso para el que se necesita el consentimiento del usuario.
- Se requiere la autenticación en dos fases.

La llamada a `acquireTokenPopup` abre una ventana emergente (o `acquireTokenRedirect` redirige a los usuarios a la Plataforma de identidad de Microsoft). En esa ventana, los usuarios tienen que interactuar confirmando sus credenciales, dándole el consentimiento al recurso requerido o completando la autenticación en dos fases.

> [!NOTE]
> Si usa Internet Explorer, se recomienda usar los métodos `loginRedirect` y `acquireTokenRedirect` debido a un [problema conocido](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/internet-explorer.md#popups) con Internet Explorer y ventanas emergentes.

## <a name="call-the-microsoft-graph-api"></a>Llamar a Microsoft Graph API

1. Cree un archivo denominado *graph.js* en la carpeta *src* y agregue el código siguiente para realizar llamadas de REST a Microsoft Graph API:

    ```javascript
    import { graphConfig } from "./authConfig";
    
    /**
     * Attaches a given access token to a Microsoft Graph API call. Returns information about the user
     */
    export async function callMsGraph(accessToken) {
        const headers = new Headers();
        const bearer = `Bearer ${accessToken}`;
    
        headers.append("Authorization", bearer);
    
        const options = {
            method: "GET",
            headers: headers
        };
    
        return fetch(graphConfig.graphMeEndpoint, options)
            .then(response => response.json())
            .catch(error => console.log(error));
    }
    ```

1. A continuación, cree el archivo *ProfileData.jsx en* *src/components* y agregue el código siguiente:

    ```javascript
    import React from "react";
    
    /**
     * Renders information about the user obtained from Microsoft Graph
     */
    export const ProfileData = (props) => {
        return (
            <div id="profile-div">
                <p><strong>First Name: </strong> {props.graphData.givenName}</p>
                <p><strong>Last Name: </strong> {props.graphData.surname}</p>
                <p><strong>Email: </strong> {props.graphData.userPrincipalName}</p>
                <p><strong>Id: </strong> {props.graphData.id}</p>
            </div>
        );
    };
    ```

1. A continuación, abra *src/App.js* y agréguelos a las instrucciones import:
    
    ```javascript
    import { ProfileData } from "./components/ProfileData";
    import { callMsGraph } from "./graph";
    ```

1. Por último, actualice el componente `ProfileContent` en *src/App.js* para llamar a Microsoft Graph y mostrar los datos del perfil después de adquirir el token. El componente `ProfileContent` debería ser similar al siguiente:

    ```javascript
    function ProfileContent() {
        const { instance, accounts } = useMsal();
        const [graphData, setGraphData] = useState(null);
    
        const name = accounts[0] && accounts[0].name;
    
        function RequestProfileData() {
            const request = {
                ...loginRequest,
                account: accounts[0]
            };
    
            // Silently acquires an access token which is then attached to a request for Microsoft Graph data
            instance.acquireTokenSilent(request).then((response) => {
                callMsGraph(response.accessToken).then(response => setGraphData(response));
            }).catch((e) => {
                instance.acquireTokenPopup(request).then((response) => {
                    callMsGraph(response.accessToken).then(response => setGraphData(response));
                });
            });
        }
    
        return (
            <>
                <h5 className="card-title">Welcome {name}</h5>
                {graphData ? 
                    <ProfileData graphData={graphData} />
                    :
                    <Button variant="secondary" onClick={RequestProfileData}>Request Profile Information</Button>
                }
            </>
        );
    };
    ```

En los cambios que se han realizado, el método `callMSGraph()` se usa para realizar una solicitud HTTP `GET` a un recurso protegido que requiere un token. A continuación, el método devuelve el contenido al autor de la llamada. Este método agrega el token adquirido al *encabezado de autorización HTTP*. En este ejemplo de aplicación creada en este tutorial, el recurso protegido es el punto de conexión *me* de Microsoft Graph API, que muestra información del perfil del usuario que ha iniciado sesión.

## <a name="test-your-application"></a>Prueba de la aplicación

Ha completado la creación de la aplicación y ya está listo para iniciar el servidor web y probar la funcionalidad de la aplicación.

1. Sirva la aplicación, para lo que debe ejecutar el siguiente comando desde la raíz de la carpeta del proyecto:

   ```console
   npm start
   ```
1. Se debe abrir automáticamente una ventana del explorador para la aplicación. Si no es así, abra el explorador y vaya a `http://localhost:3000`. Debería ver una página similar a la siguiente.

    :::image type="content" source="media/tutorial-v2-react/react-01-unauthenticated.png" alt-text="Explorador web que muestra el cuadro de diálogo de inicio de sesión":::

1. Seleccione el botón de inicio de sesión para iniciar sesión.

### <a name="provide-consent-for-application-access"></a>Consentimiento para el acceso a la aplicación

La primera vez que inicie sesión en la aplicación, se le pedirá que le conceda acceso a su perfil e inicie su sesión:

:::image type="content" source="media/tutorial-v2-javascript-auth-code/spa-02-consent-dialog.png" alt-text="Cuadro de diálogo de contenido que se muestra en el explorador web":::

Si da su consentimiento a los permisos solicitados, las aplicaciones web muestran el nombre, lo que significa que el inicio de sesión se ha realizado correctamente:

:::image type="content" source="media/tutorial-v2-react/react-02-authenticated.png" alt-text="Resultados de un inicio de sesión correcto en el explorador web":::

### <a name="call-the-graph-api"></a>Llamada a Graph API

Después de iniciar sesión, seleccione **Ver perfil** para ver la información de perfil de usuario devuelta en la respuesta de la llamada a Microsoft Graph API:

:::image type="content" source="media/tutorial-v2-react/react-03-graph-data.png" alt-text="Información de perfil de Microsoft Graph que se muestra en el explorador":::

### <a name="more-information-about-scopes-and-delegated-permissions"></a>Más información sobre los ámbitos y permisos delegados

Microsoft Graph API requiere el ámbito *user.read* para leer el perfil del usuario. De forma predeterminada, este ámbito se agrega automáticamente en todas las aplicaciones que se registran en Azure Portal. Otras API de Microsoft Graph, así como las API personalizadas para el servidor back-end, pueden requerir ámbitos adicionales. Por ejemplo, Microsoft Graph API requiere el ámbito *Mail.Read* para mostrar el correo electrónico del usuario.

A medida que se agregan ámbitos, puede que se pida a los usuarios que proporcionen consentimiento adicional para los ámbitos agregados.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Pasos siguientes

Si quiere profundizar más en el desarrollo de aplicaciones de página única de JavaScript en la Plataforma de identidad de Microsoft, consulte nuestra serie de escenarios dividida en varias partes:

> [!div class="nextstepaction"]
> [Escenario: Aplicación de una sola página](scenario-spa-overview.md)
