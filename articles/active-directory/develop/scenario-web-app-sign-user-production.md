---
title: Paso a producción de la aplicación web que inicia la sesión de los usuarios | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo compilar una aplicación web que permita iniciar sesión a los usuarios (paso a producción)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/17/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: a026d1c0c70fb210a1a24dd4a8d85beba97b9612
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108124220"
---
# <a name="web-app-that-signs-in-users-move-to-production"></a>Aplicación web que inicia sesión de usuarios: Paso a producción

Ahora que sabe cómo obtener un token para llamar a las API web, aquí tiene algunos aspectos que deben tenerse en cuenta al pasar la aplicación a producción.

[!INCLUDE [Common steps to move to production](../../../includes/active-directory-develop-scenarios-production.md)]

## <a name="troubleshooting"></a>Solución de problemas
Cuando los usuarios inicien sesión en la aplicación web por primera vez, deberán dar su consentimiento. Sin embargo, en algunas organizaciones, los usuarios pueden ver un mensaje similar al siguiente: *NombreAplicación necesita permisos para acceder a recursos de su organización que solo un administrador puede conceder. Pida a un administrador que conceda permiso a esta aplicación para poder usarla.*
Esto se debe a que el administrador de inquilinos ha **deshabilitado** la capacidad de los usuarios de dar su consentimiento. En ese caso, póngase en contacto con los administradores de inquilinos para que creen un consentimiento de administrador para los ámbitos que requiere la aplicación.

## <a name="same-site"></a>Mismo sitio

Asegúrese de que comprende los posibles problemas de las nuevas versiones del explorador Chrome: [Control de los cambios de cookies de SameSite en el explorador Chrome](howto-handle-samesite-cookie-changes-chrome-browser.md)

El paquete de NuGet Microsoft.Identity.Web controla los problemas de SameSite más comunes.

## <a name="deep-dive-aspnet-core-web-app-tutorial"></a>En profundidad: Tutorial de una aplicación web de ASP.NET Core

Obtenga información sobre otras maneras de iniciar la sesión de los usuarios con este tutorial de ASP.NET Core: 

[Habilitación de aplicaciones web para iniciar la sesión de los usuarios y llamar a las API con la Plataforma de identidad de Microsoft para desarrolladores](https://github.com/Azure-Samples/ms-identity-aspnetcore-webapp-tutorial)

Este tutorial progresivo incluye código listo para producción para una aplicación web, e incluye la forma de agregar el inicio de sesión con cuentas en:

- su organización,
- varias organizaciones,
- cuentas profesionales o educativas o cuentas personales de Microsoft,
- [Azure AD B2C](../../active-directory-b2c/overview.md)
- Nubes nacionales

## <a name="tutorial-nodejs-web-app"></a>Tutorial: Aplicación web de Node.js

Obtenga más información sobre la aplicación web de Node.js en este tutorial:

[Tutorial: Inicio de sesión de los usuarios en una aplicación web de Node.js y Express](./tutorial-v2-nodejs-webapp-msal.md)

## <a name="sample-code-java-web-app"></a>Ejemplo de código: Aplicaciones web de Java

Obtenga más información sobre la aplicación web de Java en el ejemplo de GitHub: 

[Aplicación web de Java mediante la que los usuarios inician sesión con la plataforma de identidad de Microsoft y que llama a Microsoft Graph](https://github.com/Azure-Samples/ms-identity-java-webapp)

## <a name="next-steps"></a>Pasos siguientes

Una vez que la aplicación web inicie la sesión de los usuarios, puede llamar a las API web en nombre de los usuarios con sesión iniciada. Las llamadas a las API web desde la aplicación web es el objeto del escenario siguiente: [Aplicación web que llama a las API web](scenario-web-app-call-api-overview.md)