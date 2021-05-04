---
title: 'Tutorial: Habilitación de la autenticación en una aplicación de página única'
titleSuffix: Azure AD B2C
description: En este tutorial, aprenderá a usar Azure Active Directory B2C para proporcionar inicio de sesión de usuario a una aplicación de página única (SPA).
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 04/04/2020
ms.custom: mvc, seo-javascript-september2019, devx-track-js
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: fb43415a6e5a32758dee976751436b140b4dfa03
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108071197"
---
# <a name="tutorial-enable-authentication-in-a-single-page-application-with-azure-ad-b2c"></a>Tutorial: Habilitación de la autenticación en una aplicación de página única mediante Azure AD B2C

En este tutorial se muestra cómo usar Azure Active Directory B2C (Azure AD B2C) para que los usuarios se registren e inicien sesión en una aplicación de página única (SPA) mediante el [flujo de código de autorización de OAuth 2.0](./authorization-code-flow.md) con [MSAL.js](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser).

Este tutorial es la primera parte de una serie de dos:

> [!div class="checklist"]
> * Agregue una dirección URL de respuesta a una aplicación registrada en el inquilino de Azure AD B2C.
> * Descargue un código de ejemplo de GitHub.
> * Modifique el código de ejemplo de la aplicación para que funcione con el inquilino.
> * Regístrese con el flujo de usuario de registro o inicio de sesión

El [siguiente tutorial](tutorial-single-page-app-webapi.md) de la serie habilita la parte de la API web del código de ejemplo.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

Antes de continuar con los pasos en este tutorial, necesita tener los siguientes recursos de Azure AD B2C:

* [Inquilino de Azure AD B2C](tutorial-create-tenant.md)
* [Aplicación registrada](tutorial-register-spa.md) en su inquilino
* [Flujos de usuario creados](tutorial-create-user-flows.md) en su inquilino

También necesita los siguientes elementos en su entorno de desarrollo:

* [Visual Studio Code](https://code.visualstudio.com/) u otro editor de código
* [Node.js](https://nodejs.org/en/download/)

## <a name="update-the-application"></a>Actualizar la aplicación

En el [segundo tutorial](./tutorial-register-spa.md) que completó como parte de los requisitos previos, registró una aplicación de página única en Azure AD B2C. Para habilitar la comunicación con el código de ejemplo en este tutorial, agregue una dirección URL de respuesta (también denominado URI de redirección) al registro de aplicación.

Para actualizar una aplicación en su inquilino de Azure AD B2C, puede usar la nueva experiencia unificada **Registros de aplicaciones**, o bien la experiencia anterior **Aplicaciones (heredado)** . [Más información acerca de la nueva experiencia](./app-registrations-training-guide.md).

#### <a name="app-registrations"></a>[Registros de aplicaciones](#tab/app-reg-auth/)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En el menú de la izquierda, seleccione **Azure AD B2C**. O bien, seleccione **Todos los servicios** y busque y seleccione **Azure AD B2C**.
1. Seleccione **Registros de aplicaciones**, la pestaña **Aplicaciones propias** y, por último, la aplicación *spaapp1*.
1. En **Aplicación de página única**, seleccione el vínculo **Agregar URI** y escriba `http://localhost:6420`.
1. Seleccione **Guardar**.
1. Seleccione **Información general**.
1. Anote el **Id. de aplicación (cliente)** ; lo usará en un paso posterior al actualizar el código en la aplicación web de página única.

#### <a name="applications-legacy"></a>[Aplicaciones (heredado)](#tab/applications-legacy/)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Asegúrese de usar el directorio que contiene el inquilino de Azure AD B2C. Para ello, seleccione el filtro **Directorio y suscripción** en el menú superior y luego el directorio que contiene el inquilino.
1. Seleccione **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
1. Seleccione **Applications (Legacy)** (Aplicaciones [heredado]) y, a continuación, la aplicación *spaapp1*.
1. En **URL de respuesta**, agregue `http://localhost:6420`.
1. Seleccione **Guardar**.
1. En la página de propiedades, registre el **identificador de aplicación**. Dicho identificador se usará en un paso posterior para actualizar el código de la aplicación web de una página.

* * *

## <a name="get-the-sample-code"></a>Obtención del código de ejemplo

En este tutorial, va a configurar un código de ejemplo que puede descargar desde GitHub para que funcione con el inquilino de B2C. En este ejemplo se muestra cómo una aplicación de página única puede usar Azure AD B2C para el registro y el inicio de sesión de usuario, y llamar a una API web protegida (habilita la API web en el siguiente tutorial de la serie).

  [Descargue un archivo ZIP](https://github.com/Azure-Samples/ms-identity-b2c-javascript-spa/archive/main.zip) o clone el ejemplo de GitHub:

  ```
  git clone https://github.com/Azure-Samples/ms-identity-b2c-javascript-spa.git
  ```

## <a name="update-the-sample"></a>Actualización del ejemplo

Ahora que ha obtenido el ejemplo, actualice el código con el nombre del inquilino de Azure AD B2C y el identificador de aplicación que registró en el paso anterior.

1. Abra el archivo *authConfig.js* de la carpeta *App*.
1. En el objeto `msalConfig`, busque la asignación de `clientId` y reemplácela por el valor de **Id. de aplicación (cliente)** que anotó en un paso anterior.
1. Abra el archivo `policies.js`.
1. Busque las entradas de `names` y reemplace su asignación por el nombre de los flujos de usuario que creó en un paso anterior, por ejemplo, `B2C_1_signupsignin1`.
1. Busque las entradas de `authorities` y reemplácelas según corresponda por los nombres de los flujos de usuario que creó en un paso anterior, por ejemplo, `https://<your-tenant-name>.b2clogin.com/<your-tenant-name>.onmicrosoft.com/<your-sign-in-sign-up-policy>`.
1. Busque la asignación de `authorityDomain` y reemplácela por `<your-tenant-name>.b2clogin.com`.
1. Abra el archivo `apiConfig.js`.
1. Busque la asignación de `b2cScopes` y reemplace la dirección URL por la dirección URL de ámbito que creó para la API web, por ejemplo, `b2cScopes: ["https://<your-tenant-name>.onmicrosoft.com/helloapi/demo.read"]`.
1. Busque la asignación de `webApi` y reemplace la dirección URL actual por aquella donde implementó la API web en el paso 4, por ejemplo, `webApi: http://localhost:5000/hello`.


El código resultante debe tener el siguiente aspecto:

*authConfig.js*:

```javascript
const msalConfig = {
  auth: {
    clientId: "e760cab2-b9a1-4c0d-86fb-ff7084abd902",
    authority: b2cPolicies.authorities.signUpSignIn.authority,
    knownAuthorities: [b2cPolicies.authorityDomain],
  },
  cache: {
    cacheLocation: "localStorage",
    storeAuthStateInCookie: true
  }
};

const loginRequest = {
  scopes: ["openid", "profile"],
};

const tokenRequest = {
  scopes: apiConfig.b2cScopes // i.e. ["https://fabrikamb2c.onmicrosoft.com/helloapi/demo.read"]
};
```

*policies.js*:

```javascript
const b2cPolicies = {
    names: {
        signUpSignIn: "b2c_1_susi",
        forgotPassword: "b2c_1_reset",
        editProfile: "b2c_1_edit_profile"
    },
    authorities: {
        signUpSignIn: {
            authority: "https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_susi",
        },
        forgotPassword: {
            authority: "https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_reset",
        },
        editProfile: {
            authority: "https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_edit_profile"
        }
    },
    authorityDomain: "fabrikamb2c.b2clogin.com"
}
```

*apiConfig.js*:

```javascript
const apiConfig = {
  b2cScopes: ["https://fabrikamb2c.onmicrosoft.com/helloapi/demo.read"],
  webApi: "https://fabrikamb2chello.azurewebsites.net/hello"
};
```


## <a name="run-the-sample"></a>Ejecución del ejemplo

1. Abra una ventana de consola y vaya al directorio que contiene el ejemplo.

    ```console
    cd ms-identity-b2c-javascript-spa
    ```

1. Ejecute los comandos siguientes:

    ```console
    npm install && npm update
    npm start
    ```

    La ventana de consola muestra el número de puerto del servidor Node.js que se ejecuta localmente:

    ```console
    Listening on port 6420...
    ```
1. Vaya a `http://localhost:6420` para ver la aplicación web que se ejecuta en la máquina local.

    :::image type="content" source="media/tutorial-single-page-app/web-app-spa-01-not-logged-in.png" alt-text="Explorador web que muestra la aplicación de página única que se ejecuta localmente":::

### <a name="sign-up-using-an-email-address"></a>Registro con una dirección de correo electrónico

Esta aplicación de ejemplo admite el registro, el inicio de sesión y el restablecimiento de contraseña. En este tutorial se registra mediante una dirección de correo electrónico.

1. Seleccione **Iniciar sesión** para iniciar el flujo de usuario *B2C_1_signupsignin1* que ha especificado en un paso anterior.
1. Azure AD B2C presenta una página de inicio de sesión que incluye un vínculo de registro. Como no tiene aún una cuenta, seleccione el vínculo **Registrarse ahora**.
1. El flujo de trabajo del registro presenta una página para recopilar y verificar la identidad del usuario con una dirección de correo electrónico. El flujo de trabajo del registro también recopila la contraseña de usuario y los atributos requeridos definidos en el flujo de usuario.

    Utilice una dirección de correo electrónico válida y valídela mediante el código de verificación. Establezca una contraseña. Especifique valores para los atributos solicitados.

    :::image type="content" source="media/tutorial-single-page-app/user-flow-sign-up-workflow-01.png" alt-text="Página de registro que muestra el flujo de usuario de Azure AD B2C":::

1. Seleccione **Crear** para crear una cuenta local en el directorio de Azure AD B2C.

Al seleccionar **Crear**, la aplicación muestra el nombre del usuario que ha iniciado sesión.

:::image type="content" source="media/tutorial-single-page-app/web-app-spa-02-logged-in.png" alt-text="Explorador web que muestra la aplicación de página única con el usuario que ha iniciado sesión":::

Si desea probar el inicio de sesión, seleccione el botón **Cerrar sesión**, luego, **Iniciar sesión** e inicie sesión con la dirección de correo electrónico y la contraseña que especificó al registrarse.

### <a name="what-about-calling-the-api"></a>¿Qué pasa si se llama a la API?

Si selecciona el botón **Call API** (Llamar a la API) después de iniciar sesión, aparecerá la página del flujo de usuario del registro o inicio de sesión en lugar de los resultados de la llamada a la API. Esto se espera porque todavía no ha configurado la parte de la API de la aplicación para comunicarse con una aplicación de la API web registrada en *su* inquilino de Azure AD B2C.

En este momento, la aplicación sigue intentando comunicarse con la API registrada en el inquilino de demostración (fabrikamb2c.onmicrosoft.com) y, como no se ha autenticado en ese inquilino, se muestra la página de registro o inicio de sesión.

Continúe con el siguiente tutorial de la serie para habilitar la API protegida (consulte la sección [Pasos siguientes](#next-steps)).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha configurado una aplicación de página única a fin de que funcione con un flujo de usuario en el inquilino de Azure AD B2C para proporcionar la funcionalidad de registro e inicio de sesión. Ha completado estos pasos:

> [!div class="checklist"]
> * Ha agregado una dirección URL de respuesta a una aplicación registrada en el inquilino de Azure AD B2C.
> * Ha descargado un código de ejemplo desde GitHub.
> * Ha modificado el código de ejemplo de la aplicación para que funcione con el inquilino.
> * Se ha registrado con el flujo de usuario de registro o inicio de sesión.

Pase al siguiente tutorial de la serie para conceder acceso a una API web protegida desde la SPA:

> [!div class="nextstepaction"]
> [Tutorial: Protección y concesión de acceso a una API web desde una aplicación de página única >](tutorial-single-page-app-webapi.md)