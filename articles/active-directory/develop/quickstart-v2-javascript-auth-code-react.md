---
title: 'Inicio rápido: Inicio de sesión de usuarios en aplicaciones de página única (SPA) de JavaScript React con código de autenticación y llamada a Microsoft Graph | Azure'
titleSuffix: Microsoft identity platform
description: En este inicio rápido, obtendrá información sobre la forma en que una aplicación de página única (SPA) de JavaScript React puede iniciar la sesión de usuarios de cuentas personales, profesionales y educativas mediante el flujo de código de autorización, ya llamar a Microsoft Graph.
services: active-directory
author: j-mantu
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 01/14/2021
ms.author: jamesmantu
ms.custom: aaddev, scenarios:getting-started, languages:JavaScript, devx-track-js
ms.openlocfilehash: 563f4bb3eeab6bc0329b4eb46ba06c1d4fedf259
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109655253"
---
# <a name="quickstart-sign-in-and-get-an-access-token-in-a-react-spa-using-the-auth-code-flow"></a>Inicio rápido: Inicio de sesión y obtención de un token de acceso en una SPA de React mediante el flujo de código de autorización

En este inicio rápido descargará y ejecutará un código de ejemplo que muestra cómo una aplicación de página única de JavaScript React puede realizar el inicio de sesión de usuarios y llamar a Microsoft Graph API mediante el flujo de código de autorización. En el ejemplo de código se muestra cómo obtener un token de acceso para llamar a Microsoft Graph API o a cualquier API web. 

Para ilustrar este tema, consulte el apartado en el que se explica el [funcionamiento del ejemplo](#how-the-sample-works).

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una de forma gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Node.js](https://nodejs.org/en/download/)
* [Visual Studio Code](https://code.visualstudio.com/download) u otro editor de código

> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-application"></a>Registro y descarga de la aplicación de inicio rápido
> Para iniciar la aplicación de inicio rápido, use cualquiera de las siguientes opciones.
>
>
> ### <a name="option-1-express-register-and-auto-configure-your-app-and-then-download-your-code-sample"></a>Opción 1 (Rápido): registrar y configurar de modo automático la aplicación y, a continuación, descargar el código de ejemplo
>
> 1. Vaya a la experiencia de inicio rápido <a href="https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade/quickStartType/JavascriptSpaQuickstartPage/sourceType/docs" target="_blank">Azure Portal: Registros de aplicaciones</a>.
> 1. Escriba un nombre para la aplicación.
> 1. En **Supported account types** (Tipos de cuenta compatibles), seleccione **Accounts in any organizational directory and personal Microsoft accounts** (Cuentas en cualquier directorio de organización y cuentas personales de Microsoft).
> 1. Seleccione **Registrar**.
> 1. Vaya al panel de inicio rápido y siga las instrucciones para descargar y configurar automáticamente su nueva aplicación.
>
> ### <a name="option-2-manual-register-and-manually-configure-your-application-and-code-sample"></a>Opción 2 (Manual): registrar y configurar manualmente la aplicación y el código de ejemplo
>
> #### <a name="step-1-register-your-application"></a>Paso 1: Registrar su aplicación
>
> 1. Inicie sesión en <a href="https://portal.azure.com/" target="_blank">Azure Portal</a>.
> 1. Si tiene acceso a varios inquilinos, use el filtro **Directorio + suscripción** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: del menú superior para seleccionar el inquilino en el que desea registrar una aplicación.
> 1. Busque y seleccione **Azure Active Directory**.
> 1. En **Administrar**, seleccione **Registros de aplicaciones** >  y, luego, **Nuevo registro**.
> 1. Cuando se abra la página **Registrar una aplicación**, escriba el nombre de su aplicación.
> 1. En **Supported account types** (Tipos de cuenta compatibles), seleccione **Accounts in any organizational directory and personal Microsoft accounts** (Cuentas en cualquier directorio de organización y cuentas personales de Microsoft).
> 1. Seleccione **Registrar**. En la página de **información general** de la aplicación, anote el valor del **Identificador de aplicación (cliente)** para su uso posterior.
> 1. En **Administrar**, seleccione **Autenticación**.
> 1. En **Configuraciones de plataforma**, seleccione **Agregar una plataforma**. En el panel que se abre, seleccione **Aplicación de página única**.
> 1. Establezca el valor de **URI de redirección** en `http://localhost:3000/`. Este es el puerto predeterminado que NodeJS escuchará en la máquina local. Devolveremos la respuesta de autenticación a este identificador URI después de autenticar correctamente al usuario. 
> 1. Seleccione **Configurar** para aplicar los cambios.
> 1. En **Configuraciones de plataforma** expanda **Aplicación de página única**.
> 1. Confirme que en **Tipos de concesión** ![Ya configurado](media/quickstart-v2-javascript/green-check.png) Su URI de redirección es válido para el flujo de código de autorización con PKCE.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-your-application-in-the-azure-portal"></a>Paso 1: Configuración de la aplicación en Azure Portal
> Para que el código de ejemplo de este inicio rápido funcione, agregue un **URI de redirección** de `http://localhost:3000/`.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Realizar estos cambios por mí]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Ya configurada](media/quickstart-v2-javascript/green-check.png) La aplicación está configurada con estos atributos.

#### <a name="step-2-download-the-project"></a>Paso 2: Descarga del proyecto

> [!div renderon="docs"]
> Para ejecutar el proyecto con un servidor web con Node.js. [descargue los archivos principales del proyecto](https://github.com/Azure-Samples/ms-identity-javascript-react-spa/archive/main.zip).

> [!div renderon="portal" class="sxs-lookup"]
> Ejecutar el proyecto con un servidor Web mediante Node.js

> [!div renderon="portal" class="sxs-lookup" id="autoupdate" class="nextstepaction"]
> [Descargar el código de ejemplo](https://github.com/Azure-Samples/ms-identity-javascript-react-spa/archive/main.zip)

> [!div renderon="docs"]
> #### <a name="step-3-configure-your-javascript-app"></a>Paso 3: Configuración de la aplicación de JavaScript
>
> En la carpeta *src*, abra el archivo *authConfig.js* y actualice los valores `clientID`, `authority` y `redirectUri` del objeto `msalConfig`.
>
> ```javascript
> /**
> * Configuration object to be passed to MSAL instance on creation. 
> * For a full list of MSAL.js configuration parameters, visit:
> * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/configuration.md 
> */
> export const msalConfig = {
>    auth: {
>        clientId: "Enter_the_Application_Id_Here",
>        authority: "Enter_the_Cloud_Instance_Id_HereEnter_the_Tenant_Info_Here",
>        redirectUri: "Enter_the_Redirect_Uri_Here"
>    },
>    cache: {
>        cacheLocation: "sessionStorage", // This configures where your cache will be stored
>        storeAuthStateInCookie: false, // Set this to "true" if you are having issues on IE11 or Edge
>    },
> ```

> [!div renderon="portal" class="sxs-lookup"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`

> [!div renderon="docs"]
>
> Modifique los valores de la sección `msalConfig` como se describe aquí:
>
> - `Enter_the_Application_Id_Here` es el **identificador de aplicación (cliente)** de la aplicación que registró.
>
>    Para buscar el valor de **Identificador de aplicación (cliente)** , vaya a la página **Información general** del registro de la aplicación en Azure Portal.
> - `Enter_the_Cloud_Instance_Id_Here` es la instancia de la nube de Azure. En el caso de la nube principal o global de Azure, escriba `https://login.microsoftonline.com/`. Para nubes **nacionales** (por ejemplo, China), consulte [Nubes nacionales](authentication-national-cloud.md).
> - `Enter_the_Tenant_info_here` se establece en una de las opciones siguientes:
>   - Si la aplicación admite *cuentas de este directorio organizativo*, reemplace este valor por los valores de **Id. de inquilino** o **Nombre de inquilino**. Por ejemplo, `contoso.microsoft.com`.
>
>    Para encontrar el valor de **Identificador de directorio (inquilino)** , vaya a la página **Información general** del registro de la aplicación en Azure Portal.
>   - Si la aplicación admite *cuentas de cualquier directorio organizativo*, reemplace este valor por `organizations`.
>   - Si la aplicación admite *cuentas de cualquier directorio organizativo y cuentas Microsoft personales*, reemplace este valor por `common`. **En este inicio rápido**, use `common`.
>   - Para restringir la compatibilidad a *solo cuentas de Microsoft personales*, reemplace este valor por `consumers`.
>
>    Para buscar el valor de **Tipos de cuenta admitidos**, vaya a la página **Información general** del registro de la aplicación en Azure Portal.
> - `Enter_the_Redirect_Uri_Here` es `http://localhost:3000/`.
>
> El valor `authority` en *authConfig.js* debe ser similar al siguiente si usa la nube principal (global) de Azure:
>
> ```javascript
> authority: "https://login.microsoftonline.com/common",
> ```
>
> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-your-app-is-configured-and-ready-to-run"></a>Paso 3: La aplicación está configurada y lista para ejecutarse
> Hemos configurado el proyecto con los valores de las propiedades de su aplicación.

> [!div renderon="docs"]
>
> Desplácese hacia abajo en el mismo archivo y actualice `graphMeEndpoint`. 
> - Reemplace la cadena `Enter_the_Graph_Endpoint_Herev1.0/me` por `https://graph.microsoft.com/v1.0/me`.
> - `Enter_the_Graph_Endpoint_Herev1.0/me` es el punto de conexión en el que se realizarán las llamadas API. Como servicio principal de Microsoft Graph API (global), escriba `https://graph.microsoft.com/` (incluya la barra diagonal final). Para más información, consulte la [documentación](/graph/deployments).
>
>
>
> ```javascript
>   // Add here the endpoints for MS Graph API services you would like to use.
>    export const graphConfig = {
>        graphMeEndpoint: "Enter_the_Graph_Endpoint_Herev1.0/me"
>    };
> ```
>
>
#### <a name="step-4-run-the-project"></a>Paso 4: Ejecución del proyecto

Ejecute el proyecto con un servidor web mediante Node.js:

1. Para iniciar el servidor, ejecute los siguientes comandos desde el directorio del proyecto:
    ```console
    npm install
    npm start
    ```
1. Vaya a `http://localhost:3000/`.

1. Seleccione **Iniciar sesión** para iniciar el proceso de inicio de sesión y, luego, llame a Microsoft Graph API.

    La primera vez que inicie sesión, se le pedirá que dé su consentimiento para permitir que la aplicación acceda a su perfil e inicie sesión automáticamente. Una vez que haya iniciado sesión correctamente, haga clic en **Request Profile Information** (Solicitar información del perfil) para mostrar la información del perfil en la página.

## <a name="more-information"></a>Más información

### <a name="how-the-sample-works"></a>Funcionamiento del ejemplo

![Diagrama que muestra el flujo de código de autorización para una aplicación de página única.](media/quickstart-v2-javascript-auth-code/diagram-01-auth-code-flow.png)

### <a name="msaljs"></a>msal.js

La biblioteca MSAL.js inicia la sesión de los usuarios y solicita los tokens que se usan para acceder a una API protegida por la plataforma de identidad de Microsoft.

Si tiene Node.js instalado, también puede descargar la versión más reciente mediante el administrador de paquetes de Node.js (npm):

```console
npm install @azure/msal-browser @azure/msal-react
```

## <a name="next-steps"></a>Pasos siguientes

Si desea una guía paso a paso detallada sobre la compilación de la aplicación del flujo de código de autenticación mediante Vanilla JavaScript, consulte el siguiente tutorial:

> [!div class="nextstepaction"]
> [Tutorial para iniciar sesión y llamar a MS Graph](./tutorial-v2-javascript-auth-code.md)