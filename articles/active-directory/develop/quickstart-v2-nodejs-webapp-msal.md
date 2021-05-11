---
title: 'Inicio rápido: Adición de autenticación a una aplicación web de Node.js con MSAL Node | Azure'
titleSuffix: Microsoft identity platform
description: En este inicio rápido, aprenderá a implementar la autenticación con una aplicación web de Node.js y la biblioteca de autenticación de Microsoft (MSAL) para Node.js.
services: active-directory
author: mmacy
manager: celested
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 10/22/2020
ms.author: marsma
ms.custom: aaddev, scenarios:getting-started, languages:js, devx-track-js
ms.openlocfilehash: af6d4388c9ca0af5301bc018854d9370c20d7cbe
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108176457"
---
# <a name="quickstart-sign-in-users-and-get-an-access-token-in-a-node-web-app-using-the-auth-code-flow"></a>Inicio rápido: Inicio de sesión de los usuarios y obtención de un token de acceso en una aplicación web de Node mediante el flujo de código de autorización

En este inicio rápido, descargará y ejecutará un código de ejemplo que muestra de qué forma puede una aplicación web de Node.js realizar el inicio de sesión de los usuarios mediante el flujo de código de autorización. En el ejemplo de código se muestra cómo obtener un token de acceso para llamar a Microsoft Graph API.

Para ilustrar este tema, consulte el apartado en el que se explica el [funcionamiento del ejemplo](#how-the-sample-works).

En este inicio rápido se usa la biblioteca de autenticación de Microsoft para Node.js (MSAL Node) con el flujo del código de autorización.

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure. [Creación de una suscripción de Azure gratis](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Node.js](https://nodejs.org/en/download/)
* [Visual Studio Code](https://code.visualstudio.com/download) u otro editor de código

> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-application"></a>Registro y descarga de la aplicación de inicio rápido
>
> #### <a name="step-1-register-your-application"></a>Paso 1: Registrar su aplicación
>
> 1. Inicie sesión en <a href="https://portal.azure.com/" target="_blank">Azure Portal</a>.
> 1. Si tiene acceso a varios inquilinos, use el filtro **Directorio + suscripción** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: para seleccionar el inquilino en el que desea registrar una aplicación.
> 1. En **Administrar**, seleccione **Registros de aplicaciones** >  y, luego, **Nuevo registro**.
> 1. Escriba el **nombre** de la aplicación. Los usuarios de la aplicación pueden ver este nombre, el cual se puede cambiar más tarde.
> 1. En **Supported account types** (Tipos de cuenta compatibles), seleccione **Accounts in any organizational directory and personal Microsoft accounts** (Cuentas en cualquier directorio de organización y cuentas personales de Microsoft).
> 1. Establezca el valor de **URI de redireccionamiento** en `http://localhost:3000/redirect`.
> 1. Seleccione **Registrar**.
> 1. En la página de **información general** de la aplicación, anote el valor del **Identificador de aplicación (cliente)** para su uso posterior.
> 1. En **Administrar**, seleccione **Certificados y secretos** > **Nuevo secreto de cliente**.  Deje la descripción en blanco y la expiración predeterminada y, luego, seleccione **Agregar**.
> 1. Anote el valor de **Secreto de cliente** para usarlo posteriormente.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-the-application-in-azure-portal"></a>Paso 1: Configuración de la aplicación en Azure Portal
> Para que funcione el código de ejemplo de este inicio rápido, es preciso crear un secreto de cliente y agregar la siguiente dirección URL de respuesta: `http://localhost:3000/redirect`.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Hacer este cambio por mí]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Ya configurada](media/quickstart-v2-windows-desktop/green-check.png) La aplicación está configurada con estos atributos.

#### <a name="step-2-download-the-project"></a>Paso 2: Descarga del proyecto

> [!div renderon="docs"]
> Para ejecutar el proyecto con un servidor web con Node.js. [descargue los archivos principales del proyecto](https://github.com/Azure-Samples/ms-identity-node/archive/main.zip).

> [!div renderon="portal" class="sxs-lookup"]
> Ejecute el proyecto con un servidor web mediante Node.js.

> [!div renderon="portal" class="sxs-lookup" id="autoupdate" class="nextstepaction"]
> [Descargar el código de ejemplo](https://github.com/Azure-Samples/ms-identity-node/archive/main.zip)

> [!div renderon="docs"]
> #### <a name="step-3-configure-your-node-app"></a>Paso 3: Configuración de la aplicación de Node
>
> Extraiga el proyecto, abra la carpeta *ms-identity-node-main* y, después, abra el archivo *index.js*.
>
> Establezca el valor `clientID` con el identificador de aplicación (cliente) y, después, establezca el valor `clientSecret` con el secreto de cliente.
>
>```javascript
>const config = {
>    auth: {
>        clientId: "Enter_the_Application_Id_Here",
>        authority: "https://login.microsoftonline.com/common",
>        clientSecret: "Enter_the_Client_Secret_Here"
>    },
>    system: {
>        loggerOptions: {
>            loggerCallback(loglevel, message, containsPii) {
>                console.log(message);
>            },
>            piiLoggingEnabled: false,
>            logLevel: msal.LogLevel.Verbose,
>        }
>    }
>};
> ```

> [!div renderon="docs"]
>
> Modifique los valores en la sección `config`:
>
> - `Enter_the_Application_Id_Here` es el identificador (cliente) de la aplicación que ha registrado.
>
>    Para buscar el identificador de aplicación (cliente), vaya a la página **Información general** del registro de la aplicación en Azure Portal.
> - `Enter_the_Client_Secret_Here` es el secreto de cliente de la aplicación que ha registrado.
>
>    Para recuperar o generar un secreto de cliente, en **Administrar**, seleccione **Certificados y secretos**.
>
> El valor de `authority` predeterminado representa la nube de Azure principal (global):
>
> ```javascript
> authority: "https://login.microsoftonline.com/common",
> ```
>
> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-your-app-is-configured-and-ready-to-run"></a>Paso 3: La aplicación está configurada y lista para ejecutarse
>
> [!div renderon="docs"]
>
> #### <a name="step-4-run-the-project"></a>Paso 4: Ejecución del proyecto

Ejecute el proyecto mediante Node.js.

1. Para iniciar el servidor, ejecute los siguientes comandos desde el directorio del proyecto:

    ```console
    npm install
    npm start
    ```

1. Vaya a `http://localhost:3000/`.

1. Seleccione **Iniciar sesión** para comenzar el proceso de inicio de sesión.

    La primera vez que inicie sesión, se le pedirá que dé su consentimiento para permitir que la aplicación acceda a su perfil e inicie sesión automáticamente. Tras iniciar sesión correctamente, verá un mensaje de registro en la línea de comandos.

## <a name="more-information"></a>Más información

### <a name="how-the-sample-works"></a>Funcionamiento del ejemplo

El ejemplo hospeda un servidor web en localhost, puerto 3000. Cuando un explorador web accede a este sitio, el ejemplo redirige de inmediato al usuario a una página de autenticación de Microsoft. Por eso el ejemplo no contiene elementos HTML ni de presentación. Si el resultado de la autenticación es satisfactorio, se muestra el mensaje "OK" (Correcto).

### <a name="msal-node"></a>MSAL Node

La biblioteca MSAL Node inicia la sesión de los usuarios y solicita los tokens que se usan para acceder a una API protegida por la plataforma de identidad de Microsoft. Puede descargar la versión más reciente mediante el administrador de paquetes de Node.js (npm):

```console
npm install @azure/msal-node
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Adición de autenticación a una aplicación web existente: ejemplo de código de GitHub >](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-node-samples/auth-code)
