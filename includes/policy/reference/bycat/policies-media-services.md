---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 06/11/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 887d2784d4949a70763e6c08ce3ec72aaddd0bcd
ms.sourcegitcommit: 942a1c6df387438acbeb6d8ca50a831847ecc6dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112019776"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Las cuentas de Azure Media Services deben usar una API que admita Private Link](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa77d8bb4-8d22-4bc1-a884-f582a705b480) |Las cuentas de Media Services se deben crear con una API que admita vínculo privado. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Media%20Services/MediaServices_RequirePrivateLinkSupport_Audit.json) |
|[Se deben bloquear las cuentas de Azure Media Services que permitan el acceso a la v2 API heredada](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fccf93279-9c91-4143-a841-8d1f21505455) |La API v2 (heredada) de Media Services permite solicitudes que no se pueden administrar mediante Azure Policy. Los recursos de Media Services creados mediante la API 2020-05-01 o posterior bloquean el acceso a la API v2 heredada. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Media%20Services/MediaServices_BlockRestV2_Audit.json) |
|[Las directivas de clave de contenido de Azure Media Services deben usar la autenticación por tokens](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fdaccf7e4-9808-470c-a848-1c5b582a1afb) |Las directivas de clave de contenido definen las condiciones que se deben cumplir para acceder a las claves de contenido. Una restricción de token garantiza que solo los usuarios que tengan tokens válidos de un servicio de autenticación puedan acceder a las claves de contenido, por ejemplo, Azure Active Directory. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Media%20Services/ContentKeyPolicies_RequireTokenAuth_Audit.json) |
|[Los trabajos de Azure Media Services con entradas HTTPS deben limitar los URI de entrada a patrones de URI permitidos](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe9914afe-31cd-4b8a-92fa-c887f847d477) |Restrinja las entradas HTTPS que usan los trabajos de Media Services a puntos de conexión conocidos. Las entradas de los puntos de conexión HTTPS se pueden deshabilitar completamente estableciendo una lista vacía de patrones de entrada de trabajos permitidos. Cuando las entradas del trabajo especifican un elemento "baseUri", los patrones se comparan con este valor; cuando no se establece "baseUri", el patrón se compara con la propiedad "files". |Deny, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Media%20Services/Jobs_RestrictHttpInputs.json) |
