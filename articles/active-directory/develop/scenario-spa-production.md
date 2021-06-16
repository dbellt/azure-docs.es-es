---
title: Traslado de una aplicación de página única a producción
titleSuffix: Microsoft identity platform
description: Aprenda a compilar una aplicación de página única (paso a producción)
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: mtillman
ms.custom: aaddev
ms.openlocfilehash: 48f9b90ef72d79510747cba82a59386d4f3a4372
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112075176"
---
# <a name="single-page-application-move-to-production"></a>Aplicación de página única: Paso a producción

Ahora que sabe cómo adquirir un token para llamar a las API web, aquí tiene algunos aspectos que deben tenerse en cuenta al trasladar la aplicación a producción.

[!INCLUDE [Common steps to move to production](../../../includes/active-directory-develop-scenarios-production.md)]

## <a name="deploy-your-app"></a>Implementación de la aplicación

Vea un [ejemplo de implementación](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnet-webapi-multitenant/tree/master/Chapter3) para aprender a implementar los proyectos de SPA y API web con Azure Storage y Azure App Services, respectivamente. 

## <a name="code-samples"></a>Ejemplos de código

En estos ejemplos de código se muestran varias operaciones clave para una aplicación de página única.
- [SPA con un back-end de ASP.NET](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnetcore-webapi): cómo obtener tokens para una API web de back-end (ASP.NET Core) propia mediante **MSAL.js**.

- [API web de Node.js (Azure AD)](https://github.com/Azure-Samples/active-directory-javascript-nodejs-webapi-v2): cómo validar tokens de acceso para la API web de back-end (Node.js) mediante **passport-azure-ad**.

- [SPA con Azure AD B2C](https://github.com/Azure-Samples/ms-identity-b2c-javascript-spa): cómo usar **MSAL.js** para iniciar la sesión de los usuarios en una aplicación registrada en **Azure Active Directory B2C** (Azure AD B2C).

- [API web de Node.js (Azure AD B2C)](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi): cómo usar **passport-azure-ad** para validar tokens de acceso para aplicaciones registradas con **Azure Active Directory B2C** (Azure AD B2C).

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial de SPA de JavaScript](./tutorial-v2-javascript-auth-code.md): profundice sobre cómo iniciar la sesión para los usuarios y obtener un token de acceso para llamar a **Microsoft Graph API** mediante **MSAL.js**.