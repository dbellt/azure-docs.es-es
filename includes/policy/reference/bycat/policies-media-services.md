---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 05/04/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: f6e4aaf143f077c40c6184fd8e55e437763ea2c6
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108762373"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Los trabajos de Azure Media Services con entradas HTTPS deben limitar los URI de entrada a patrones de URI permitidos](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe9914afe-31cd-4b8a-92fa-c887f847d477) |Restrinja las entradas HTTPS que usan los trabajos de Media Services a puntos de conexión conocidos. Las entradas de los puntos de conexión HTTPS se pueden deshabilitar completamente estableciendo una lista vacía de patrones de entrada de trabajos permitidos. Cuando las entradas del trabajo especifican un elemento "baseUri", los patrones se comparan con este valor; cuando no se establece "baseUri", el patrón se compara con la propiedad "files". |Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Media%20Services/Jobs_RestrictHttpInputs.json) |
