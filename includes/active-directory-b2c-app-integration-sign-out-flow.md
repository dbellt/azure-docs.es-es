---
author: msmimart
ms.service: active-directory-b2c
ms.subservice: B2C
ms.topic: include
ms.date: 06/11/2021
ms.author: mimart
ms.openlocfilehash: f4677d9800f0c94d9071838b9addbeb3cd1231c9
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112071356"
---
El flujo de cierre de sesión consta de los siguientes pasos:

1. En la aplicación, el usuario selecciona **Cerrar sesión**.
1. La aplicación borra sus cookies de sesión y la biblioteca de autenticación borra su caché de tokens.
1. La aplicación redirige al usuario al punto de conexión de cierre de sesión de Azure AD B2C para finalizar la sesión de Azure AD B2C.
1. Se redirige al usuario a la aplicación.