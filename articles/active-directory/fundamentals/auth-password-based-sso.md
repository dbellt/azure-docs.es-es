---
title: Autenticación basada en contraseñas con Azure Active Directory
description: Guía de arquitectura para lograr la autenticación basada en contraseñas con Azure Active Directory.
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 10/10/2020
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 737a073a0360842d2ddd8010970e8a3461193742
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111966037"
---
# <a name="password-based-authentication-with-azure-active-directory"></a>Autenticación basada en contraseñas con Azure Active Directory

El inicio de sesión único (SSO) basado en contraseñas usa el proceso de autenticación existente para la aplicación. Cuando habilita el inicio de sesión único basado en contraseñas, Azure Active Directory (Azure AD) recopila, cifra y almacena de forma segura las credenciales de usuario en el directorio. Azure AD proporciona el nombre de usuario y la contraseña a la aplicación cuando el usuario intenta iniciar sesión.

Elija el inicio de sesión único basado en contraseñas cuando una aplicación se autentique con un nombre de usuario y una contraseña en lugar de hacerlo con encabezados y tokens de acceso. El inicio de sesión único basado en contraseñas admite cualquier aplicación basada en la nube cuya página de inicio de sesión esté basada en HTML. 

## <a name="use-when"></a>Cuándo se utiliza

Debe protegerse con la autenticación previa y proporcionar inicio de sesión único a través de almacenamiento de contraseñas para las aplicaciones web.

![Diagrama de la arquitectura](./media/authentication-patterns/password-based-sso-auth.png)


## <a name="components-of-system"></a>Componentes del sistema

* **Usuario**: accede a una aplicación basada en formularios desde Aplicaciones o visitando directamente el sitio. 

* **Explorador web** : el componente con el que el usuario interactúa para acceder a la dirección URL externa de la aplicación. El usuario accede a la aplicación basada en formularios a través de la extensión MyApps. 

* **Extensión MyApps**: identifica la aplicación de SSO basada en contraseñas configurada y proporciona las credenciales al formulario de inicio de sesión. La extensión MyApps se instala en el explorador web. 

* **Azure AD**: autentica al usuario.

## <a name="implement-password-based-sso-with-azure-ad"></a>Implementación de SSO basado en contraseñas con Azure AD

* [Qué es el inicio de sesión único basado en contraseñas](../manage-apps/what-is-single-sign-on.md) 

* [Configuración del inicio de sesión único basado en contraseñas para aplicaciones en la nube](../manage-apps/configure-password-single-sign-on-non-gallery-applications.md)

* [Configuración del inicio de sesión único basado en contraseñas para aplicaciones locales con Application Proxy](../app-proxy/application-proxy-configure-single-sign-on-password-vaulting.md)