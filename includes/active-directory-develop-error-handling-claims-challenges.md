---
author: mmacy
ms.author: marsma
ms.date: 11/25/2020
ms.service: active-directory
ms.subservice: develop
ms.topic: include
ms.openlocfilehash: e54301e01d25fb075325ef34522daa5f1b691dd5
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "98760650"
---
## <a name="conditional-access-and-claims-challenges"></a>Acceso condicional y desafíos de notificaciones

Al obtener los tokens de forma automática, la aplicación puede recibir errores si una API a la que está intentando acceder requiere un [desafío de notificaciones de acceso condicional](../articles/active-directory/develop/v2-conditional-access-dev-guide.md) como, por ejemplo, una directiva de autenticación multifactor.

El patrón para controlar este error consiste en adquirir interactivamente un token mediante MSAL. De esta forma, se avisa al usuario y le da la oportunidad de cumplir con la directiva de acceso condicional requerida.

En algunos casos, al llamar a una API que requiere acceso condicional, puede recibir un desafío de notificaciones en el error de la API. Por ejemplo, si la directiva de acceso condicional consiste en tener un dispositivo administrado (Intune) el error puede ser parecido a este: [AADSTS53000: Your device is required to be managed to access this resource](../articles/active-directory/develop/reference-aadsts-error-codes.md) (Se requiere que su dispositivo sea administrado para acceder a este recurso) o algo parecido. En este caso, puede pasar las notificaciones de la llamada de adquisición del token de manera que se le pida al usuario que cumpla con la directiva adecuada.
