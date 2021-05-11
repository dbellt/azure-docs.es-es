---
title: Aumento de la resistencia de las aplicaciones de autenticación y autorización desarrolladas
titleSuffix: Microsoft identity platform
description: Información general de nuestra guía de resistencia para el desarrollo de aplicaciones con Azure Active Directory y la plataforma de identidad de Microsoft
services: active-directory
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: how-to
author: knicholasa
ms.author: nichola
manager: martinco
ms.date: 11/23/2020
ms.openlocfilehash: 2076b2087c7cd90c83e1fe31c024d2c253109ee3
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108162192"
---
# <a name="increase-resilience-of-authentication-and-authorization-applications-you-develop"></a>Aumento de la resistencia de las aplicaciones de autenticación y autorización desarrolladas

La plataforma de identidad de Microsoft usa la autenticación y la autorización modernas basadas en tokens. Esto significa que una aplicación cliente adquiere tokens de un proveedor de identidades para autenticar al usuario y para autorizar a la aplicación a que llame a las API protegidas. Un servicio validará los tokens.

Un token es válido durante un tiempo determinado antes de que la aplicación deba adquirir uno nuevo. En raras ocasiones, las llamadas para recuperar un token puede que no funcionen debido a errores de la infraestructura o interrupciones del servicio de autenticación. En este documento, se describen los pasos que puede llevar a cabo un desarrollador para aumentar la resistencia en sus aplicaciones en caso de error de adquisición del token.

En estos artículos se proporcionan instrucciones sobre cómo aumentar la resistencia en las aplicaciones mediante la plataforma de identidad de Microsoft y Azure Active Directory. Hay instrucciones que van dirigidas tanto a las aplicaciones cliente y de servicio que funcionan en nombre de un usuario que ha iniciado sesión como a las aplicaciones de demonio que funcionan en su propio nombre. Contienen procedimientos recomendados para el uso de tokens, así como para la llamada a los recursos.

- [Aumento de la resistencia en aplicaciones que inician la sesión de los usuarios.](resilience-client-app.md)
- [Aumento de la resistencia en aplicaciones sin usuarios](resilience-daemon-app.md)
- [Compilación de resistencia en la infraestructura de administración de identidades y acceso](resilience-in-infrastructure.md)
- [Aumento de la resistencia en los sistemas CIAM](resilience-b2c.md)
- [Servicios de compilación que son resistentes a la actualización de metadatos](../develop/howto-build-services-resilient-to-metadata-refresh.md)
