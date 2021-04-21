---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/14/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 49074cd5d1ea69b39e76472cde04578f913adef1
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107499076"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Permite administrar los id. de inquilino que se incorporarán mediante Azure Lighthouse](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7a8a51a3-ad87-4def-96f3-65a1839242b6) |La restricción de las delegaciones de Azure Lighthouse a inquilinos de administración concretos aumenta la seguridad, ya que limita los usuarios que pueden administrar los recursos de Azure. |deny |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Lighthouse/AllowCertainManagingTenantIds_Deny.json) |
|[Auditar la delegación de ámbitos a un inquilino de administración](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F76bed37b-484f-430f-a009-fd7592dff818). |Audita la delegación de los ámbitos a un inquilino de administración a través de Azure Lighthouse. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Lighthouse/Lighthouse_Delegations_Audit.json) |
