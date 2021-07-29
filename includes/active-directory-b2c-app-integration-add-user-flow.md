---
author: msmimart
ms.service: active-directory-b2c
ms.subservice: B2C
ms.topic: include
ms.date: 06/11/2021
ms.author: mimart
ms.openlocfilehash: fdafb1320c40a44bcdd9a365bd53baec431627d3
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112071355"
---
Cuando un usuario intenta iniciar sesión en la aplicación, esta inicia una solicitud de autenticación para el punto de conexión de autorización a través de un [flujo de usuario](../articles/active-directory-b2c/user-flow-overview.md). El flujo de usuario define y controla la experiencia del usuario. Cuando el usuario completa el flujo de usuario, Azure AD B2C genera un token y, luego, redirige al usuario de vuelta a la aplicación.

Si aún no lo ha hecho, [cree un flujo de usuario o una directiva personalizada](../articles/active-directory-b2c/add-sign-up-and-sign-in-policy.md).
