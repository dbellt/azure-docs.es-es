---
title: Parámetros y encabezados comunes
description: Los parámetros y encabezados comunes a todas las operaciones que puede realizar relacionadas con los recursos de Key Vault.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 01/07/2019
ms.author: mbaldwin
ms.openlocfilehash: 616b6061b08258d465b09902556de6903b873199
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107749877"
---
# <a name="common-parameters-and-headers"></a>Parámetros y encabezados comunes

La siguiente información es común a todas las operaciones que puede realizar relacionadas con los recursos de Key Vault:

- El encabezado HTTP `Host` debe estar siempre presente y debe especificar el nombre de host del almacén. Ejemplo: `Host: contoso.vault.azure.net`. Tenga en cuenta que la mayoría de las tecnologías de cliente rellenan el encabezado `Host` del URI. Por ejemplo, `GET https://contoso.vault.azure.net/secrets/mysecret{...}` establecerá el `Host` como `contoso.vault.azure.net`. Esto significa que si tiene acceso a Key Vault mediante una dirección raw IP, como `GET https://10.0.0.23/secrets/mysecret{...}`, el valor automático del encabezado de `Host` será incorrecto y tendrá que asegurarse de que el encabezado de `Host` contiene el nombre de host del almacén de forma manual.
- Reemplace `{api-version}` por la versión de api del URI.
- Reemplace `{subscription-id}` por el identificador de suscripción del URI.
- Sustituya `{resource-group-name}` por el grupo de recursos. Para más información, consulte el artículo Uso de grupos de recursos para administrar los recursos de Azure.
- Reemplace `{vault-name}` por el nombre del almacén de claves del URI.
- Establezca el encabezado Content-Type en application/json.
- Establezca el encabezado Authorization en un token web de JSON que se obtiene de Azure Active Directory (AAD). Par más información, consulte [Solicitudes de autenticación de Azure Resource Manager](authentication-requests-and-responses.md).

## <a name="common-error-response"></a>Respuestas de errores habituales
El servicio usará códigos de estado HTTP para indicar el éxito o el error. Además, los errores contienen una respuesta en el formato siguiente:

```
   {  
     "error": {  
     "code": "BadRequest",  
     "message": "The key vault sku is invalid."  
     }  
   }  
```

|Nombre del elemento | Tipo | Descripción |
|---|---|---|
| código | string | El tipo de error que se produjo.|
| message | string | Una descripción de lo que produjo el error. |



## <a name="see-also"></a>Consulte también
 [Referencia de la API REST de Azure Key Vault](/rest/api/keyvault/)
