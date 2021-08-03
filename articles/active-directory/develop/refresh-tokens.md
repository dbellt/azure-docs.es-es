---
title: Tokens de actualización de la Plataforma de identidad de Microsoft | Azure
titleSuffix: Microsoft identity platform
description: Conozca los tokens de actualización emitidos por Azure AD.
services: active-directory
author: SHERMANOUKO
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 05/25/2021
ms.author: shermanouko
ms.reviewer: mmacy, hirsin
ms.custom: aaddev, identityplatformtop40, fasttrack-edit
ms.openlocfilehash: 8b8bda71c637419dbd5b2bf7b181288abd9b965c
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112075248"
---
# <a name="microsoft-identity-platform-refresh-tokens"></a>Tokens de actualización de la Plataforma de identidad de Microsoft

Cuando un cliente adquiere un token de acceso para acceder a un recurso protegido, recibe también un token de actualización. El token de actualización se usa para obtener nuevos pares de tokens de acceso/actualización cuando el token de acceso actual expira. Los tokens de actualización se usan también para adquirir tokens de acceso adicionales para otros recursos. Los tokens de actualización están enlazados a una combinación de usuario y cliente, pero no están asociados a ningún recurso ni inquilino. Esto permite que un cliente use un token de actualización para adquirir tokens de acceso en cualquier combinación de recurso e inquilino donde tenga permiso para hacerlo. Los tokens de actualización están cifrados y solo la Plataforma de identidad de Microsoft puede leerlos.

## <a name="prerequisites"></a>Prerrequisitos

Antes de leer este artículo, se recomienda que leer estos otros:

* [Tokens de id.](id-tokens.md) de la plataforma de identidad de Microsoft.
* [Tokens de acceso](access-tokens.md) de la Plataforma de identidad de Microsoft.

## <a name="refresh-token-lifetime"></a>Vigencia del token de actualización

Los tokens de actualización tienen una vigencia considerablemente superior a la de los tokens de acceso. La vigencia predeterminada de los tokens es de 90 días y se reemplazan automáticamente por un token nuevo después de cada uso. Esto significa que, cada vez que se usa un token de actualización para adquirir un nuevo token de acceso, también se emite un nuevo token de actualización. La Plataforma de identidad de Microsoft no revoca los tokens de actualización antiguos cuando se usan para obtener nuevos tokens de acceso. Elimine de forma segura el token de actualización antiguo después de adquirir uno nuevo. Los tokens de actualización deben almacenarse de forma segura, como los tokens de acceso o las credenciales de las aplicaciones. 

## <a name="refresh-token-expiration"></a>Expiración de los tokens de actualización

Los tokens de actualización se pueden revocar en cualquier momento, porque se agote el tiempo de espera o por revocaciones. Cuando esto sucede, la aplicación debe abordar los rechazos del servicio de inicio de sesión de forma controlada. Para ello, muestra al usuario un mensaje de inicio de sesión interactivo para que vuelva a iniciar sesión. 

### <a name="token-timeouts"></a>Tiempos de espera de token

La [configuración de la vigencia de los tokens](active-directory-configurable-token-lifetimes.md#refresh-and-session-token-lifetime-policy-properties) permite reducir o aumentar la vigencia de los tokens de actualización. Esta configuración cambia el período de tiempo que un token de actualización puede estar sin usarse. Por ejemplo, imagine un escenario en el que un usuario no abre una aplicación durante más de 90 días. Cuando la aplicación intente usar ese token de actualización de más de 90 días, encontrará que ha expirado. Además, un administrador puede requerir que se usen segundos factores con una cadencia periódica para obligar al usuario a iniciar sesión manualmente a intervalos específicos. Entre los escenarios se incluyen los siguientes:

* Inactividad: los tokens de actualización solo son válidos durante un período determinado por `MaxInactiveTime`.  Si el token no se usa (y no se reemplaza por un nuevo token) dentro de ese período de tiempo, ya no se podrá usar.
* Sesión expirada: si `MaxAgeSessionMultiFactor` o `MaxAgeSessionSingleFactor` se han establecido en un valor distinto de su valor predeterminado (hasta que se revoca), será necesario volver a autenticarse cuando transcurra el tiempo establecido en MaxAgeSession*.  Esto se usa para obligar a los usuarios a autenticarse de nuevo periódicamente con un primer o segundo factor. 
* Ejemplos:
  * El inquilino tiene establecido MaxInactiveTime en cinco días y el usuario ha estado de vacaciones durante una semana. Por tanto, Azure AD no ha recibido una nueva solicitud de token del usuario en siete días. La próxima vez que el usuario solicite un nuevo token, encontrará que su token de actualización se ha revocado, así que tendrá que escribir sus credenciales de nuevo.
  * Una aplicación confidencial tiene un valor de `MaxAgeSessionMultiFactor` de un día. Un usuario deberá usar MFA una vez más a través de un mensaje interactivo si vuelve a iniciar sesión después de un período de un día. Por ejemplo, si un usuario inicia sesión el lunes y, después, el martes cuando han transcurrido 25 horas. 

No todos los tokens de actualización siguen las reglas establecidas en la directiva de vigencia de los tokens. En concreto, los tokens de actualización que se usan en las [aplicaciones de una sola página](reference-third-party-cookies-spas.md) se limitan siempre a 24 horas de actividad, como si se les aplicara una directiva de `MaxAgeSessionSingleFactor` de 24 horas. 

### <a name="revocation"></a>Revocación

El servidor puede revocar los tokens de actualización por un cambio de credenciales o por una acción del usuario o del administrador.  Los tokens de actualización se dividen en dos clases: los emitidos para clientes confidenciales (columna del extremo derecho) y los emitidos para clientes públicos (el resto de columnas).

| Change | Cookie basada en contraseñas | Token basado en contraseñas | Cookie no basada en contraseñas | Token no basado en contraseñas | Token de cliente confidencial |
|---|-----------------------|----------------------|---------------------------|--------------------------|---------------------------|
| La contraseña expira | Permanece activa | Permanece activa | Permanece activa | Permanece activa | Permanece activa |
| Contraseña cambiada por el usuario | Revocada | Revocada | Permanece activa | Permanece activa | Permanece activa |
| Usuario realiza SSPR | Revocada | Revocada | Permanece activa | Permanece activa | Permanece activa |
| Administrador restablece la contraseña | Revocada | Revocada | Permanece activa | Permanece activa | Permanece activa |
| Usuario revoca sus tokens de actualización [a través de PowerShell](/powershell/module/azuread/revoke-azureadsignedinuserallrefreshtoken) | Revocada | Revocada | Revocada | Revocada | Revocada |
| El administrador revoca todos los tokens de actualización de un usuario [a través de PowerShell](/powershell/module/azuread/revoke-azureaduserallrefreshtoken) | Revocada | Revocada |Revocada | Revocada | Revocada |
| Cierre de sesión único [en la Web](v2-protocols-oidc.md#single-sign-out) | Revocada | Permanece activa | Revocada | Permanece activa | Permanece activa |

## <a name="next-steps"></a>Pasos siguientes

* Más información acerca de la [vigencia configurable de los tokens](active-directory-configurable-token-lifetimes.md).
* Consulte [Tokens de actualización principales](../devices/concept-primary-refresh-token.md) para obtener más detalles sobre estos tokens.
