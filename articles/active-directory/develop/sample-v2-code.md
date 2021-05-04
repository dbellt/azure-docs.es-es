---
title: Ejemplos de código de la Plataforma de identidad de Microsoft
description: Proporciona un índice de los ejemplos de código disponibles para la plataforma de identidad de Microsoft organizados por escenario.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: sample
ms.workload: identity
ms.date: 11/04/2020
ms.author: marsma
ms.reviewer: jmprieur
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: dc8c498d80a25fce15577d6df96dc288c1201bf9
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108071818"
---
# <a name="microsoft-identity-platform-code-samples-v20-endpoint"></a>Ejemplos de código de la Plataforma de identidad de Microsoft (punto de conexión v2.0)

Use la plataforma de identidad de Microsoft para:

- Agregar autenticación y autorización a sus aplicaciones web y API web.
- Requerir un token de acceso para tener acceso a una API web protegida.

Este artículo describe brevemente vínculos a ejemplos y proporciona acceso a los mismos para la plataforma de identidad de Microsoft. Estos ejemplos le muestran cómo se hace y también proporciona fragmentos de código que puede usar en sus aplicaciones. En la página de ejemplos de código, encontrará temas Léame detallados que le ayudarán con los requisitos, la instalación y la configuración. Los comentarios dentro del código lo ayudarán a comprender las secciones críticas.

Para entender el escenario básico de cada tipo de ejemplo, consulte el artículo sobre los [tipos de aplicaciones para la plataforma de identidad de Microsoft](v2-app-types.md).

También puede contribuir a los ejemplos en GitHub. Para saber cómo, vea [Ejemplos y documentación de Microsoft Azure Active Directory](https://github.com/Azure-Samples?page=3&query=active-directory).

## <a name="single-page-applications"></a>Aplicación de página única

En estos ejemplos se muestra cómo escribir una aplicación de página única protegida con la Plataforma de identidad de Microsoft. Estos ejemplos usan uno de los tipos de MSAL.js.

| Plataforma | Descripción | Vínculo |
| -------- | --------------------- | -------- |
| ![Esta imagen muestra el logotipo de JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (MSAL.js 2.0)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser) | SPA llama a Microsoft Graph mediante el flujo de código de autenticación con PKCE. |[javascript-v2](https://github.com/Azure-Samples/ms-identity-javascript-v2) |
| ![Esta imagen muestra el logotipo de JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (MSAL.js 2.0)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser) | SPA llama a B2C mediante el flujo de código de autenticación con PKCE. |[b2c-javascript-spa](https://github.com/Azure-Samples/ms-identity-b2c-javascript-spa) |
| ![Esta imagen muestra el logotipo de JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (MSAL.js 2.0)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser) | SPA llama a su propia API web que, a su vez, llama a Microsoft Graph.  | [ms-identity-javascript-tutorial-chapter4-obo](https://github.com/Azure-Samples/ms-identity-javascript-tutorial/tree/main/4-AdvancedGrants/1-call-api-graph) |
| ![Esta imagen muestra el logotipo de Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL Angular)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular)| SPA llama a Microsoft Graph.  | [active-directory-javascript-singlepageapp-angular](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-angular) |
| ![Esta imagen muestra el logotipo de Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL Angular 2.0)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular)| SPA llama a Microsoft Graph mediante el flujo de código de autenticación con PKCE. | [ms-identity-javascript-angular-spa](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa) |
| ![Esta imagen muestra el logotipo de Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL Angular 2.0)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular)| SPA llama a la API web personalizada. | [ms-identity-javascript-angular-spa-aspnetcore-webapi](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnetcore-webapi) |
| ![Esta imagen muestra el logotipo de Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL Angular)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular) | SPA llama a B2C. |[active-directory-b2c-javascript-angular-spa](https://github.com/Azure-Samples/active-directory-b2c-javascript-angular-spa) |
| ![Esta imagen muestra el logotipo de Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL Angular 2.0)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular) | SPA llama a la API web personalizada con roles de aplicación y grupos de seguridad. |[ms-identity-javascript-angular-spa-dotnetcore-webapi-roles-groups](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-dotnetcore-webapi-roles-groups) |
| ![Esta imagen muestra el logotipo de React](media/sample-v2-code/logo_react.png) [React (MSAL React)](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-react)| SPA llama a Microsoft Graph mediante el flujo de código de autenticación con PKCE. | [ms-identity-javascript-react-spa](https://github.com/Azure-Samples/ms-identity-javascript-react-spa) |
| ![Esta imagen muestra el logotipo de React](media/sample-v2-code/logo_react.png) [React (MSAL React)](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-react)| SPA llama a la API web personalizada. | [ms-identity-javascript-react-tutorial](https://github.com/Azure-Samples/ms-identity-javascript-react-tutorial/tree/main/3-Authorization-II/1-call-api) |
| ![Esta imagen muestra el logotipo de React](media/sample-v2-code/logo_react.png) [React (MSAL.js 2.0)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core)| SPA llama a su propia API web que, a su vez, llama a Microsoft Graph.  | [ms-identity-javascript-react-spa-dotnetcore-webapi-obo](https://github.com/Azure-Samples/ms-identity-javascript-react-spa-dotnetcore-webapi-obo) |
| ![Esta imagen muestra el logotipo de Blazor](media/sample-v2-code/logo-blazor.png) [Blazor WebAssembly (MSAL.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser) | Tutorial de Blazor WebAssembly sobre cómo iniciar la sesión de los usuarios y llamar a las API mediante Azure Active Directory |[ms-identity-blazor-wasm](https://github.com/Azure-Samples/ms-identity-blazor-wasm) |

## <a name="web-applications"></a>Aplicaciones web

Los ejemplos siguientes ilustran las aplicaciones web que inician la sesión de los usuarios. Algunos ejemplos también muestran la aplicación que llama a Microsoft Graph o a su propia API web con la identidad del usuario.

| Plataforma | Únicamente inicio de sesión del usuario | Inicio de sesión del usuario y llamada a Microsoft Graph |
| -------- | ------------------- | --------------------------------- |
| ![Esta imagen muestra el logotipo de ASP.NET Core](media/sample-v2-code/logo_NETcore.png)</p>ASP.NET Core | [ASP.NET Core WebApp signs-in users tutorial](https://aka.ms/aspnetcore-webapp-sign-in) (Tutorial de inicio de sesión para los usuarios de WebApp de ASP.NET Core) | Mismo ejemplo en la fase en la que la [aplicación web de ASP.NET Core llama a Microsoft Graph](https://aka.ms/aspnetcore-webapp-call-msgraph)</p>Ejemplo avanzado: [Acceso a la memoria caché de tokens del usuario que ha iniciado sesión desde aplicaciones en segundo plano, API y servicios](https://github.com/Azure-Samples/ms-identity-dotnet-advanced-token-cache) |
| ![Esta imagen muestra el logotipo de ASP.NET Framework](media/sample-v2-code/logo_NETframework.png)</p>ASP.NET Core | [Cuaderno de migración de aplicaciones de AD FS a Azure AD para desarrolladores](https://github.com/Azure-Samples/ms-identity-dotnet-adfs-to-aad), para aprender a migrar con seguridad las aplicaciones integradas con Servicios de federación de Active Directory (AD FS) a Azure Active Directory (Azure AD). | |
| ![Esta imagen muestra el logotipo de ASP.NET Framework](media/sample-v2-code/logo_NETframework.png)</p> ASP.NET | [Inicio rápido de ASP.NET](https://github.com/AzureAdQuickstarts/AppModelv2-WebApp-OpenIDConnect-DotNet) </p> [dotnet-webapp-openidconnect-v2](https://github.com/azure-samples/active-directory-dotnet-webapp-openidconnect-v2)  |  [dotnet-admin-restricted-scopes-v2](https://github.com/azure-samples/active-directory-dotnet-admin-restricted-scopes-v2) </p> [msgraph-training-aspnetmvcapp](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp) |
| ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png)  |[Tutorial de servlet de Java: capítulo 1.1](https://github.com/Azure-Samples/ms-identity-java-servlet-webapp-authentication/blob/main/README.md) Inicio de sesión con AAD| |
| ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png)  |[Tutorial de servlet de Java: capítulo 1.2](https://github.com/Azure-Samples/ms-identity-java-servlet-webapp-authentication/blob/main/README.md) Inicio de sesión con B2C |
| ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png)  | | [Tutorial de servlet de Java: capítulo 2.1](https://github.com/Azure-Samples/ms-identity-java-servlet-webapp-authentication/blob/main/README.md) Inicio de sesión con AAD y gráfico de llamadas|
| ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png)  |[Tutorial de servlet de Java: capítulo 3.1](https://github.com/Azure-Samples/ms-identity-java-servlet-webapp-authentication/blob/main/README.md) Inicio de sesión con AAD y control de acceso con notificación de roles| |
| ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png)  | | [Tutorial de servlet de Java: capítulo 3.2](https://github.com/Azure-Samples/ms-identity-java-servlet-webapp-authentication/blob/main/README.md) Inicio de sesión con AAD y control de acceso con notificación de grupos|
| ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png) | |[Tutorial de servlet de Java: capítulo 4.1](https://github.com/Azure-Samples/ms-identity-java-servlet-webapp-authentication/blob/main/README.md) Implementación en Azure App Service|
| ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png)  | | [ms-identity-java-webapp](https://github.com/Azure-Samples/ms-identity-java-webapp) |
| ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png)  | [ms-identity-b2c-java-servlet-webapp-authentication](https://github.com/Azure-Samples/ms-identity-b2c-java-servlet-webapp-authentication)|  |
| ![Esta imagen muestra el logotipo de Node.js](media/sample-v2-code/logo_nodejs.png)</p>Node.js (nodo MSAL) | [Tutorial de inicio de sesión para los usuarios de la aplicación web de Express](https://github.com/Azure-Samples/ms-identity-node) | |
| ![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo_python.png)  | [Tutorial de Python Flask: capítulo 1.1](https://github.com/Azure-Samples/ms-identity-python-flask-tutorial/blob/main/README.md) Inicio de sesión con AAD  |  |
| ![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo_python.png)  | [Tutorial de Python Flask: capítulo 1.2](https://github.com/Azure-Samples/ms-identity-python-flask-tutorial/blob/main/README.md) Inicio de sesión con B2C                    |  |
| ![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo_python.png)  | | [Tutorial de Python Flask: capítulo 2.1](https://github.com/Azure-Samples/ms-identity-python-flask-tutorial/blob/main/README.md) Inicio de sesión con AAD y gráfico de llamadas |
| ![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo_python.png)  | |[Tutorial de Python Flask: capítulo 3.1](https://github.com/Azure-Samples/ms-identity-python-flask-tutorial/blob/main/README.md) Implementación en Azure App Service  |
| ![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo_python.png)  | [Tutorial de Django de Python: capítulo 1.1](https://github.com/Azure-Samples/ms-identity-python-django-tutorial/blob/main/README.md) Inicio de sesión con AAD  | |
| ![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo_python.png)  | [Tutorial de Django de Python: capítulo 1.2](https://github.com/Azure-Samples/ms-identity-python-django-tutorial/blob/main/README.md) Inicio de sesión con B2C                    |  |
| ![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo_python.png)  | | [Tutorial de Django de Python: capítulo 2.1](https://github.com/Azure-Samples/ms-identity-python-django-tutorial/blob/main/README.md) Inicio de sesión con AAD y gráfico de llamadas|
| ![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo_python.png)  | | [Tutorial de Django de Python: capítulo 3.1](https://github.com/Azure-Samples/ms-identity-python-django-tutorial/blob/main/README.md) Implementación en Azure App Service                    |
| ![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo_python.png)  | | [Aplicación web de Python Flask](https://github.com/Azure-Samples/ms-identity-python-webapp) |
| ![Esta imagen muestra el logotipo de Ruby](media/sample-v2-code/logo_ruby.png) |                   | [msgraph-training-rubyrailsapp](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp) |
| ![Esta imagen muestra el logotipo de Blazor.](media/sample-v2-code/logo-blazor.png)</p>Servidor Blazor | [Tutorial de inicio de sesión de los usuarios de la aplicación Blazor Server](https://github.com/Azure-Samples/ms-identity-blazor-server/tree/main/WebApp-OIDC) | [Llamadas a Microsoft Graph de la aplicación Blazor Server](https://github.com/Azure-Samples/ms-identity-blazor-server/tree/main/WebApp-graph-user/Call-MSGraph)</p>Tutorial de Chapterwise: [La aplicación Blazor Server inicia la sesión de los usuarios y llama a las API con Azure Active Directory](https://github.com/Azure-Samples/ms-identity-blazor-server) |

## <a name="desktop-and-mobile-public-client-apps"></a>Aplicaciones de escritorio y móviles de cliente público

Los siguientes ejemplos muestran aplicaciones cliente público (aplicaciones de escritorio o móviles) que tienen acceso a Microsoft Graph API o a una API web en nombre de un usuario. Además del ejemplo de *Escritorio (consola) con WAM*, todas estas aplicaciones cliente usan la Biblioteca de autenticación de Microsoft (MSAL).

| Aplicación cliente | Plataforma | Flujo y concesión | Llama a Microsoft Graph | Llama a una API web de ASP.NET Core. |
| ------------------ | -------- |  ----------| ---------- | ------------------------- |
| Tutorial de escritorio (.NET Core), opcionalmente con:</p>- la memoria caché de tokens multiplataforma</p>- la interfaz de usuario web personalizada | ![Esta imagen muestra el logotipo de .NET/C#](media/sample-v2-code/logo_NETcore.png) | [Código de autorización](msal-authentication-flows.md#authorization-code)| [ms-identity-dotnet-desktop-tutorial](https://github.com/azure-samples/ms-identity-dotnet-desktop-tutorial) | |
| Escritorio (WPF)      | ![Esta imagen muestra el logotipo del escritorio de .NET y C#.](media/sample-v2-code/logo_NET.png) | [Código de autorización](msal-authentication-flows.md#authorization-code)| [dotnet-desktop-msgraph-v2](https://github.com/azure-samples/active-directory-dotnet-desktop-msgraph-v2) | [dotnet-native-aspnetcore-v2](https://aka.ms/msidentity-aspnetcore-webapi) |
| Escritorio (consola)   | ![Imagen que muestra el logotipo de .NET y C# (escritorio)](media/sample-v2-code/logo_NET.png) | [Autenticación integrada de Windows](msal-authentication-flows.md#integrated-windows-authentication) | [dotnet-iwa-v2](https://github.com/azure-samples/active-directory-dotnet-iwa-v2) |  |
| Escritorio (consola)   | ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png) | [Autenticación integrada de Windows](msal-authentication-flows.md#integrated-windows-authentication) |[ms-identity-java-desktop](https://github.com/Azure-Samples/ms-identity-java-desktop/) |  |
| Escritorio (consola)   | ![Este es el logotipo de .NET y C# (escritorio).](media/sample-v2-code/logo_NETcore.png) | [Nombre de usuario/contraseña](msal-authentication-flows.md#usernamepassword) |[dotnetcore-up-v2](https://github.com/azure-samples/active-directory-dotnetcore-console-up-v2) |  |
| Escritorio (consola) con WAM  | ![Este es el logotipo de .NET y C# (escritorio).](media/sample-v2-code/logo_NETcore.png) | Interactivo con el [Administrador de cuentas web](/windows/uwp/security/web-account-manager) (WAM) |[dotnet-native-uwp-wam](https://github.com/azure-samples/active-directory-dotnet-native-uwp-wam) |  |
| Escritorio (consola)   | ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png) | [Nombre de usuario/contraseña](msal-authentication-flows.md#usernamepassword) |[ms-identity-java-desktop](https://github.com/Azure-Samples/ms-identity-java-desktop/) |  |
| Escritorio (consola)   | ![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo_python.png) | [Nombre de usuario/contraseña](msal-authentication-flows.md#usernamepassword) |[ms-identity-python-desktop](https://github.com/Azure-Samples/ms-identity-python-desktop) |  |
| Escritorio (Electron)   | ![Esta imagen muestra el logotipo de Node.js](media/sample-v2-code/logo_nodejs.png)</p>Node.js (nodo MSAL) | [Código de autorización (PKCE)](msal-authentication-flows.md#authorization-code) |[ms-identity-javascript-nodejs-desktop](https://github.com/Azure-Samples/ms-identity-javascript-nodejs-desktop) |  |
| Mobile (Android, iOS, UWP)   | ![Esta imagen muestra el logotipo de .NET/C# (Xamarin)](media/sample-v2-code/logo_xamarin.png) | [Código de autorización](msal-authentication-flows.md#authorization-code) |[xamarin-native-v2](https://github.com/azure-samples/active-directory-xamarin-native-v2) |  |
| Mobile (iOS)       | ![Esta imagen muestra iOS/Objective-C o Swift](media/sample-v2-code/logo_iOS.png) | [Código de autorización](msal-authentication-flows.md#authorization-code) |[ios-swift-objc-native-v2](https://github.com/azure-samples/active-directory-ios-swift-native-v2) </p> [ios-native-nxoauth2-v2](https://github.com/azure-samples/active-directory-ios-native-nxoauth2-v2) |  |
| Escritorio (macOS)       | macOS | [Código de autorización](msal-authentication-flows.md#authorization-code) |[macOS-swift-objc-native-v2](https://github.com/Azure-Samples/ms-identity-macOS-swift-objc) |  |
| Móvil (Android-Java)   | ![Esta imagen muestra el logotipo de Android](media/sample-v2-code/logo_Android.png) | [Código de autorización](msal-authentication-flows.md#authorization-code) |  [android-Java](https://github.com/Azure-Samples/ms-identity-android-java) |  |
| Móvil (Android-Kotlin)   | ![Esta imagen muestra el logotipo de Android](media/sample-v2-code/logo_Android.png) | [Código de autorización](msal-authentication-flows.md#authorization-code) |  [android-Kotlin](https://github.com/Azure-Samples/ms-identity-android-kotlin) |  |

## <a name="daemon-applications"></a>Aplicaciones demonio

En el siguiente ejemplo se muestra una aplicación que accede a Microsoft Graph API con su propia identidad (sin ningún usuario).

| Aplicación cliente | Plataforma | Flujo y concesión | Llama a Microsoft Graph |
| ------------------ | -------- | ---------- | -------------------- |
| Consola | ![Esta imagen muestra el logotipo de .NET Core](media/sample-v2-code/logo_NETcore.png)</p> ASP.NET  | [Credenciales de cliente](msal-authentication-flows.md#client-credentials) | [dotnetcore-daemon-v2](https://github.com/azure-samples/active-directory-dotnetcore-daemon-v2) |
| Aplicación web | ![Captura de pantalla que muestra el logotipo de ASP.NET.](media/sample-v2-code/logo_NETframework.png)</p> ASP.NET  | [Credenciales de cliente](msal-authentication-flows.md#client-credentials) | [dotnet-daemon-v2](https://github.com/azure-samples/active-directory-dotnet-daemon-v2) |
| Consola | ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png) | [Credenciales de cliente](msal-authentication-flows.md#client-credentials) | [ms-identity-java-daemon](https://github.com/Azure-Samples/ms-identity-java-daemon) |
| Consola | ![Esta imagen muestra el logotipo de Node.js](media/sample-v2-code/logo_nodejs.png)</p>Node.js (nodo MSAL)| [Credenciales de cliente](msal-authentication-flows.md#client-credentials) | [ms-identity-javascript-nodejs-console](https://github.com/Azure-Samples/ms-identity-javascript-nodejs-console) |
| Consola | ![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo_python.png) | [Credenciales de cliente](msal-authentication-flows.md#client-credentials) | [ms-identity-python-daemon](https://github.com/Azure-Samples/ms-identity-python-daemon) |

## <a name="headless-applications"></a>Aplicaciones sin periféricos

El siguiente ejemplo muestra aplicaciones cliente públicas que se ejecutan en un dispositivo sin un explorador web. La aplicación puede ser una herramienta de línea de comandos, una aplicación que se ejecuta en Linux o Mac o una aplicación de IoT. El ejemplo incluye aplicaciones que acceden a Microsoft Graph API en nombre de un usuario que inicia sesión de manera interactiva en otro dispositivo (como un teléfono móvil). Esta aplicación cliente usa la Biblioteca de autenticación de Microsoft (MSAL).

| Aplicación cliente | Plataforma | Flujo y concesión | Llama a Microsoft Graph |
| ------------------ | -------- |  ----------| ---------- |
| Escritorio (consola)   | ![Esta imagen muestra el logotipo de .NET/C# (escritorio)](media/sample-v2-code/logo_NETcore.png) | [Flujo de código de dispositivo](msal-authentication-flows.md#device-code) |[dotnetcore-devicecodeflow-v2](https://github.com/azure-samples/active-directory-dotnetcore-devicecodeflow-v2) |
| Escritorio (consola)   | ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png) | [Flujo de código de dispositivo](msal-authentication-flows.md#device-code) |[ms-identity-java-devicecodeflow](https://github.com/Azure-Samples/ms-identity-java-devicecodeflow) |
| Escritorio (consola)   | ![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo_python.png) | [Flujo de código de dispositivo](msal-authentication-flows.md#device-code) |[ms-identity-python-devicecodeflow](https://github.com/Azure-Samples/ms-identity-python-devicecodeflow) |

## <a name="multi-tenant-saas-applications"></a>Aplicaciones SaaS multiinquilino

En los siguientes ejemplos se muestra cómo configurar la aplicación para que acepte inicios de sesión de cualquier inquilino de Azure Active Directory (Azure AD). La configuración de la aplicación para que sea *multiinquilino* significa que puede ofrecer una aplicación de **software como servicio** (SaaS) a muchas organizaciones, lo que permite a sus usuarios iniciar sesión en la aplicación después de dar su consentimiento.

| Plataforma | Descripción | Vínculo |
| -------- | --------------------- | -------- |
| ![Esta imagen muestra el logotipo de Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL Angular 2.0)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular) | SPA multiinquilino llama a la API web personalizada multiinquilino. |[ms-identity-javascript-angular-spa-aspnet-webapi-multitenant](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnet-webapi-multitenant/tree/master/Chapter2) |
| ![Esta imagen muestra el logotipo de ASP.NET Core](media/sample-v2-code/logo_NETcore.png) [.NET Core (MSAL.NET)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) | La aplicación web ASP.NET Core MVC llama a Graph API |[active-directory-aspnetcore-webapp-openidconnect-v2](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/2-WebApp-graph-user/2-3-Multi-Tenant) |
| ![Esta imagen muestra el logotipo de ASP.NET Core](media/sample-v2-code/logo_NETcore.png) [.NET Core (MSAL.NET)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) | La aplicación web ASP.NET Core MVC llama a la API web de ASP.NET Core |[active-directory-aspnetcore-webapp-openidconnect-v2](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/4-WebApp-your-API/4-3-AnyOrg) |

## <a name="web-apis"></a>API web

En los ejemplos siguientes se muestra cómo proteger una API web mediante la plataforma de identidad de Microsoft y cómo llamar a una API descendente desde la API web.

| Plataforma | Muestra |
| -------- | ------------------- |
| ![Esta imagen muestra el logotipo de ASP.NET Core](media/sample-v2-code/logo_NETcore.png)</p>ASP.NET Core | API web (servicio) de ASP.NET Core de [dotnet-native-aspnetcore-v2](https://aka.ms/msidentity-aspnetcore-webapi-calls-msgraph)  |
| ![Esta imagen muestra el logotipo de ASP.NET](media/sample-v2-code/logo_NET.png)</p>ASP.NET MVC | API web (servicio) de [ms-identity-aspnet-webapi-onbehalfof](https://github.com/Azure-Samples/ms-identity-aspnet-webapi-onbehalfof) |
| ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png) | Web API (servicio) de [ms-identity-java-webapi](https://github.com/Azure-Samples/ms-identity-java-webapi) |
| ![Esta imagen muestra el logotipo de Node.js](media/sample-v2-code/logo_nodejs.png)</p>Node.js (Passport.js)| API web (servicio) de [active-directory-javascript-nodejs-webapi-v2](https://github.com/Azure-Samples/active-directory-javascript-nodejs-webapi-v2) |
| ![Esta imagen muestra el logotipo de Node.js](media/sample-v2-code/logo_nodejs.png)</p>Node.js (Passport.js)| API web B2C (servicio) de [active-directory-b2c-javascript-nodejs-webapi](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi) |

## <a name="azure-functions-as-web-apis"></a>Azure Functions como API web

En los ejemplos siguientes se muestra cómo proteger una función de Azure mediante HttpTrigger y exponer una API web mediante la plataforma de identidad de Microsoft. También se muestra cómo llamar a una API descendente desde la API web.

| Plataforma | Muestra |
| -------- | ------------------- |
| ![Esta imagen muestra el logotipo de ASP.NET Core](media/sample-v2-code/logo_NETcore.png)</p>ASP.NET Core | Función de Azure de API web de ASP.NET Core (servicio) de [dotnet-native-aspnetcore-v2](https://github.com/Azure-Samples/ms-identity-dotnet-webapi-azurefunctions)  |
| ![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo_python.png)</p>Python | API web API (servicio) de [Python](https://github.com/Azure-Samples/ms-identity-python-webapi-azurefunctions) |
| ![Esta imagen muestra el logotipo de Node.js](media/sample-v2-code/logo_nodejs.png)</p>Node.js (Passport.js)| API web (servicio) de [Node.js y passport-azure-ad](https://github.com/Azure-Samples/ms-identity-nodejs-webapi-azurefunctions) |
| ![Esta imagen muestra el logotipo de Node.js](media/sample-v2-code/logo_nodejs.png)</p>Node.js (Passport.js)| API web (servicio) de [Node.js y passport-azure-ad en nombre de un usuario](https://github.com/Azure-Samples/ms-identity-nodejs-webapi-onbehalfof-azurefunctions) |

## <a name="other-microsoft-graph-samples"></a>Otros ejemplos de Microsoft Graph

Para ver [ejemplos](https://github.com/microsoftgraph/msgraph-community-samples/tree/master/samples#aspnet) y tutoriales que muestren los diferentes patrones de uso de Microsoft Graph API, incluida la autenticación con Azure AD, consulte [Microsoft Graph Community samples & tutorials](https://github.com/microsoftgraph/msgraph-community-samples) (Ejemplos y tutoriales de la comunidad de Microsoft Graph).

## <a name="see-also"></a>Consulte también

[Conceptos y referencia de Microsoft Graph API](/graph/use-the-api?context=graph%2fapi%2fbeta&view=graph-rest-beta&preserve-view=true)
