---
author: mmacy
ms.author: marsma
ms.date: 11/25/2020
ms.service: active-directory
ms.subservice: develop
ms.topic: include
ms.openlocfilehash: 2ff600429cefbfe0c9a4f0522ee49274000029ca
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "98760663"
---
En este artículo se proporciona información general sobre los distintos tipos de errores y las recomendaciones para controlar los errores de inicio de sesión comunes.

## <a name="msal-error-handling-basics"></a>Aspectos básicos del control de errores en MSAL

Las excepciones de la Biblioteca de autenticación de Microsoft (MSAL) están diseñadas para que los desarrolladores de aplicaciones solucionen problemas y no para mostrarse a los usuarios finales. Los mensajes de excepción no están traducidos.

Cuando procesa excepciones y errores, puede usar el propio tipo de excepción y el código de error para distinguir entre las excepciones.  Para ver una lista con los códigos de error, consulte [Códigos de error de autenticación y autorización de Azure AD](../articles/active-directory/develop/reference-aadsts-error-codes.md).

Durante la experiencia de inicio de sesión, puede encontrar errores de consentimiento, de acceso condicional (MFA, administración de dispositivos, restricciones basadas en la ubicación), de emisión y canje de tokens, y de las propiedades del usuario.

En la sección siguiente se proporcionan más detalles sobre el control de errores de la aplicación.