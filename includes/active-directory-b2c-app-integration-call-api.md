---
author: msmimart
ms.service: active-directory-b2c
ms.subservice: B2C
ms.topic: include
ms.date: 06/11/2021
ms.author: mimart
ms.openlocfilehash: fe128e7eee99d2b396c758979f0efff936ec6785
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112073064"
---
Después de la autenticación, el usuario navega a una página que invoca una API web protegida con autenticación de token de portador. El token de portador es el token de acceso que se obtiene de Azure AD B2C. La aplicación pasa el token en el encabezado de autorización de la solicitud HTTPS. 
    
```http
Authorization: Bearer <token>
```

Si el ámbito del token de acceso no coincide con los ámbitos de la API web, la biblioteca de autenticación obtiene un nuevo token de acceso con los ámbitos correctos.
s