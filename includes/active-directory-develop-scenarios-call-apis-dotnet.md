---
title: archivo de inclusión
description: archivo de inclusión
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.custom: include file
ms.openlocfilehash: 3d4e45d1bf53bab4d1f9c45367f9d051f1668e2b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "76308902"
---
### <a name="authenticationresult-properties-in-msalnet"></a>Propiedades de AuthenticationResult en MSAL.NET

Los métodos para adquirir tokens devuelven `AuthenticationResult`. En el caso de los métodos asincrónicos, se devuelve `Task<AuthenticationResult>`.

En MSAL.NET, `AuthenticationResult` expone:

- `AccessToken` para que la API web acceda a los recursos. Este parámetro es una cadena, normalmente un JWT con codificación en Base 64. El cliente nunca debe mirar dentro del token de acceso. No se garantiza que el formato permanezca estable y se pueda cifrar para el recurso. La escritura de código que depende del contenido de un token de acceso del cliente constituye una de las fuentes de errores y de interrupciones de la lógica de cliente más habituales. Para más información, consulte [Tokens de acceso](../articles/active-directory/develop/access-tokens.md).
- `IdToken` del usuario. Este parámetro es un JWT codificado. Para más información, consulte [Tokens de id.](../articles/active-directory/develop/id-tokens.md)
- `ExpiresOn` indica el día y la hora a la que expira el token.
- `TenantId` indica el inquilino en el que se encontró el usuario. Para los usuarios invitados en escenarios B2B de Azure Active Directory, el identificador de inquilino es el inquilino invitado, no el inquilino único.
Cuando se entrega el token para un usuario, `AuthenticationResult` también contiene información sobre dicho usuario. Para los flujos de cliente confidenciales en los que se solicitan tokens sin usuario para la aplicación, esta información de usuario está vacía.
- Los `Scopes` para los que se emitió el token.
- El identificador único del usuario.

### <a name="iaccount"></a>IAccount

MSAL.NET define la noción de una cuenta mediante la interfaz `IAccount`. Este cambio importante proporciona la semántica correcta. El mismo usuario puede tener varias cuentas en directorios de Azure AD diferentes. También MSAL.NET proporciona mejor información en el caso de escenarios de invitado ya que se proporciona información de la cuenta doméstica.
En el siguiente diagrama se muestra la estructura de la interfaz `IAccount`.

![Estructura de la interfaz IAccount](https://user-images.githubusercontent.com/13203188/44657759-4f2df780-a9fe-11e8-97d1-1abbffade340.png)

La clase `AccountId` identifica una cuenta en un inquilino específico con las propiedades que se muestran en la tabla siguiente.

| Propiedad | Descripción |
|----------|-------------|
| `TenantId` | Representación de una cadena en un GUID, que es el identificador del inquilino donde reside la cuenta. |
| `ObjectId` | Representación de una cadena en un GUID, que es el identificador del usuario a quien pertenece la cuenta en el inquilino. |
| `Identifier` | Identificador único de la cuenta. `Identifier` es la concatenación de `ObjectId` y `TenantId`, separados por una coma. No están codificados en Base 64. |

La interfaz `IAccount` representa información sobre una sola cuenta. El mismo usuario puede estar presente en distintos inquilinos, es decir, un usuario puede tener varias cuentas. Sus miembros se muestran en la siguiente tabla.

| Propiedad | Descripción |
|----------|-------------|
| `Username` | Una cadena que contiene el valor que se puede mostrar en formato UserPrincipalName (UPN), por ejemplo, john.doe@contoso.com. Esta cadena puede ser NULL, a diferencia de HomeAccountId y HomeAccountId.Identifier que no serán NULL. Esta propiedad reemplaza la propiedad `DisplayableId` de `IUser` en versiones anteriores de MSAL.NET. |
| `Environment` | Una cadena que contiene el proveedor de identidades de esta cuenta, por ejemplo, `login.microsoftonline.com`. Esta propiedad reemplaza la propiedad `IdentityProvider` de `IUser`, salvo que `IdentityProvider` también tenía información sobre el inquilino además del entorno de nube. Aquí, el valor es solo el host. |
| `HomeAccountId` | El identificador de la cuenta de inicio del usuario. Esta propiedad identifica unívocamente al usuario a través de los inquilinos de Azure AD. |

### <a name="use-the-token-to-call-a-protected-api"></a>Uso del token para llamar a una API protegida

Después de que MSAL devuelva `AuthenticationResult` en `result`, agréguelo al encabezado de autorización HTTP antes de realizar la llamada para acceder a la API web protegida.

```csharp
httpClient = new HttpClient();
httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);

// Call the web API.
HttpResponseMessage response = await _httpClient.GetAsync(apiUri);
...
}
```