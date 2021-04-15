---
author: mmacy
ms.author: marsma
ms.date: 11/25/2020
ms.service: active-directory
ms.subservice: develop
ms.topic: include
ms.openlocfilehash: d0377e3e918a8431c35c1f5b15a1c4b54eeae3b4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "98760703"
---
## <a name="retrying-after-errors-and-exceptions"></a>Nuevos intentos después de errores y excepciones

Se espera que implemente sus propias directivas de reintentos al llamar a MSAL. MSAL realiza llamadas HTTP al servicio de Azure AD y, en ocasiones, pueden producirse errores. Por ejemplo, la red puede dejar de funcionar o el servidor sobrecargarse.  

### <a name="http-429"></a>HTTP 429

Cuando el servidor de token de servicio (STS) se sobrecarga con demasiadas solicitudes, devuelve un error HTTP 429 con una sugerencia sobre el tiempo de espera necesario antes de volver a intentarlo en el campo de respuesta `Retry-After`.