---
title: Ejemplos de código de Azure Active Directory B2C | Microsoft Docs
description: Ejemplos de código para aplicaciones móviles, de escritorio, web y de una sola página de Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 10/02/2020
ms.custom: mvc
ms.topic: sample
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: 4cc7bff05f7235843a05802dcd54d46e339318df
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110575432"
---
# <a name="azure-active-directory-b2c-code-samples"></a>Ejemplos de código de Azure Active Directory B2C

En las tablas siguientes se proporcionan vínculos a ejemplos para aplicaciones iOS, Android, .NET y Node.js.

## <a name="mobile-and-desktop-apps"></a>Aplicaciones móviles y de escritorio

| Muestra | Descripción |
|--------| ----------- |
| [ios-swift-native-msal](https://github.com/Azure-Samples/active-directory-b2c-ios-swift-native-msal) | Un ejemplo de iOS en Swift que autentica a los usuarios de Azure AD B2C y llama a una API mediante OAuth 2.0 |
| [android-native-msal](https://github.com/Azure-Samples/ms-identity-android-java#b2cmodefragment-class) | Una aplicación Android sencilla que muestra cómo usar MSAL para autenticar a los usuarios mediante Azure Active Directory B2C y acceder a una API web con los tokens resultantes. |
| [ios-native-appauth](https://github.com/Azure-Samples/active-directory-b2c-ios-native-appauth) | Un ejemplo que muestra cómo se puede usar una biblioteca de terceros para compilar una aplicación de iOS en Objective-C que autentica a los usuarios de identidades de Microsoft en nuestro servicio de identidad de Azure AD B2C. |
| [android-native-appauth](https://github.com/Azure-Samples/active-directory-b2c-android-native-appauth) | Un ejemplo que muestra cómo se puede usar una biblioteca de terceros para compilar una aplicación Android que autentica a los usuarios de identidades de Microsoft en nuestro servicio de identidad B2C y llama a una API web mediante tokens de acceso de OAuth 2.0. |
| [dotnet-desktop](https://github.com/Azure-Samples/active-directory-b2c-dotnet-desktop) | Un ejemplo que muestra cómo una aplicación .NET de Windows Desktop (WPF) puede iniciar la sesión de un usuario mediante Azure AD B2C, obtener un token de acceso mediante MSAL.NET y llamar a una API. |
| [xamarin-native](https://github.com/Azure-Samples/active-directory-b2c-xamarin-native) | Una aplicación de Xamarin Forms sencilla que muestra cómo usar MSAL para autenticar a los usuarios a través de Azure Active Directory B2C y acceder a una API web con los tokens resultantes. |

## <a name="web-apps-and-apis"></a>Web Apps y API

| Muestra | Descripción |
|--------| ----------- |
| [dotnet-webapp-and-webapi](https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi) | Un ejemplo combinado de una aplicación web .NET que llama a una API web. NET, ambas protegidas mediante Azure AD B2C. |
| [dotnetcore-webapp-openidconnect](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-5-B2C) | Una aplicación web ASP.NET Core que usa OpenID Connect para iniciar la sesión de los usuarios en Azure AD B2C. |
| [dotnetcore-webapp-msal-api](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/4-WebApp-your-API/4-2-B2C) | Una aplicación web ASP.NET Core que puede iniciar la sesión de un usuario mediante Azure AD B2C, obtener un token de acceso mediante MSAL.NET y llamar a una API. |
| [openidconnect-nodejs](https://github.com/AzureADQuickStarts/B2C-WebApp-OpenIDConnect-NodeJS) | Una aplicación Node.js que proporciona una manera rápida y sencilla de configurar una aplicación web con Express mediante OpenID Connect. |
| [javascript-nodejs-webapi](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi) | Una pequeña API web node.js para Azure AD B2C que muestra cómo proteger la API web y aceptar tokens de acceso B2C mediante passport.js. |
| [ms-identity-python-webapp](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/master/README_B2C.md) | Se muestra cómo integrar B2C de la Plataforma de identidad de Microsoft con una aplicación web en Python.  |

## <a name="single-page-apps"></a>Aplicaciones de una sola página

| Muestra | Descripción |
|--------| ----------- |
| [ms-identity-javascript-angular-tutorial](https://github.com/Azure-Samples/ms-identity-javascript-angular-tutorial/tree/main/3-Authorization-II/2-call-api-b2c) | Una aplicación de página única (SPA) de Angular que llama a una API web. La autenticación se realiza con Azure AD B2C mediante MSAL Angular. En este ejemplo se usa el flujo de código de autorización con PKCE. |
| [ms-identity-javascript-react-tutorial](https://github.com/Azure-Samples/ms-identity-javascript-react-tutorial/tree/main/3-Authorization-II/2-call-api-b2c) | Una aplicación de página única de React que llama a una API web. La autenticación se realiza con Azure AD B2C mediante MSAL React. En este ejemplo se usa el flujo de código de autorización con PKCE. |
| [ms-identity-b2c-javascript-spa](https://github.com/Azure-Samples/ms-identity-b2c-javascript-spa) | Una aplicación de página única de VanillaJS que llama a una API web. La autenticación se realiza con Azure AD B2C mediante MSAL.js. En este ejemplo se usa el flujo de código de autorización con PKCE. |
| [javascript-nodejs-management](https://github.com/Azure-Samples/ms-identity-b2c-javascript-nodejs-management/tree/main/Chapter1) | Una aplicación de página única de VanillaJS que llama a Microsoft Graph para administrar usuarios en un directorio de B2C. La autenticación se realiza con Azure AD B2C mediante MSAL.js. En este ejemplo se usa el flujo de código de autorización con PKCE.|

## <a name="consoledaemon-apps"></a>Aplicaciones de consola o de demonio

| Muestra | Descripción |
|--------| ----------- |
| [javascript-nodejs-management](https://github.com/Azure-Samples/ms-identity-b2c-javascript-nodejs-management/tree/main/Chapter2) | Una aplicación de Node.js y de demonio de consola rápida que llama a Microsoft Graph con su propia identidad para administrar usuarios en un directorio B2C. La autenticación se realiza con Azure AD B2C mediante MSAL Node. En este ejemplo se usa el flujo de código de autorización.|
| [dotnetcore-b2c-account-management](https://github.com/Azure-Samples/ms-identity-dotnetcore-b2c-account-management) | Una aplicación de consola de .NET Core que llama a Microsoft Graph con su propia identidad para administrar usuarios en un directorio B2C. La autenticación se realiza con Azure AD B2C mediante MSAL.NET. En este ejemplo se usa el flujo de código de autorización.|

## <a name="saml-test-application"></a>Aplicación de prueba SAML

| Muestra | Descripción |
|--------| ----------- |
| [saml-sp-tester](https://github.com/azure-ad-b2c/saml-sp-tester/tree/master/source-code) | Aplicación de prueba SAML para probar Azure AD B2C configurado para que actúe como proveedor de identidades SAML. |

## <a name="api-connectors"></a>Conectores de API

En las tablas siguientes se proporcionan vínculos a ejemplos de código para aprovechar las API web en los flujos de usuario con [conectores de API](api-connectors-overview.md).

### <a name="azure-function-quickstarts"></a>Inicios rápidos de Azure Functions

| Muestra                                                                                                                          | Descripción                                                                                                                                               |
| ------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [.NET Core](https://github.com/Azure-Samples/active-directory-dotnet-external-identities-api-connector-azure-function-validate) | Este ejemplo de Azure Functions con .NET Core muestra cómo limitar los registros a dominios de correo electrónico específicos y validar la información proporcionada por el usuario. |
| [Node.js](https://github.com/Azure-Samples/active-directory-nodejs-external-identities-api-connector-azure-function-validate)   | Este ejemplo de Azure Functions con Node.js muestra cómo limitar los registros a dominios de correo electrónico específicos y validar la información proporcionada por el usuario.  |
| [Python](https://github.com/Azure-Samples/active-directory-python-external-identities-api-connector-azure-function-validate)    | Este ejemplo de Azure Functions con Python muestra cómo limitar los registros a dominios de correo electrónico específicos y validar la información proporcionada por el usuario.    |


### <a name="automated-fraud-protection-services--captcha"></a>Servicios automatizados de protección contra el fraude y CAPTCHA
| Muestra                                                                                                            | Descripción                                                                                                                          |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| [Protección de Arkose Labs contra el fraude y el uso indebido](https://github.com/Azure-Samples/active-directory-b2c-node-sign-up-user-flow-arkose) | En este ejemplo se muestra cómo proteger los registros de usuario mediante el servicio de protección de Arkose Labs contra el fraude y el abuso. |
| [reCAPTCHA](https://github.com/Azure-Samples/active-directory-b2c-node-sign-up-user-flow-captcha) | En este ejemplo se muestra cómo proteger los registros de usuario mediante un desafío reCAPTCHA para evitar el abuso automatizado. |


### <a name="identity-verification"></a>Verificación de identidad

| Muestra                                                                                                            | Descripción                                                                                                                          |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| [IDology](https://github.com/Azure-Samples/active-directory-dotnet-external-identities-idology-identity-verification) | En este ejemplo se muestra cómo comprobar una identidad de usuario como parte de los flujos de registro mediante un conector de API para integrarse con IDology. |
| [Experian](https://github.com/Azure-Samples/active-directory-dotnet-external-identities-experian-identity-verification) | En este ejemplo se muestra cómo comprobar una identidad de usuario como parte de los flujos de registro mediante un conector de API para integrarse con Experian. |


### <a name="other"></a>Otros

| Muestra                                                                                                            | Descripción                                                                                                                          |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| [Código de invitación](https://github.com/Azure-Samples/active-directory-b2c-node-sign-up-user-flow-invitation-code) | Este ejemplo muestra cómo limitar los registros a públicos específicos mediante códigos de invitación.|
| [Ejemplos de la comunidad del conector de API](https://github.com/azure-ad-b2c/api-connector-samples) | Este repositorio incluye ejemplos de escenarios mantenidos por la comunidad habilitados por los conectores de API.|
