---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 05/04/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 4595e611d5a3d95b885f0ae4b1b6479d07f12187
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108762220"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[El punto de conexión de Bot Service debe ser un URI de HTTPS válido](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6164527b-e1ee-4882-8673-572f425f5e0a) |Los datos se pueden alterar durante la transmisión. Hay protocolos que proporcionan cifrado para solucionar problemas de uso indebido y manipulación. Para asegurarse de que los bots se comunican solo por canales cifrados, establezca el punto de conexión en un URI de HTTPS válido. Esto garantiza que se usa el protocolo HTTPS para cifrar los datos en tránsito; además, suele ser un requisito de cumplimiento de los estándares normativos o del sector. Visite: [https://docs.microsoft.com/azure/bot-service/bot-builder-security-guidelines](https://docs.microsoft.com/azure/bot-service/bot-builder-security-guidelines). |audit, deny, disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Bot%20Service/BotService_ValidEndpoint_Audit.json) |
|[Bot Service se debe cifrar con una clave administrada por el cliente](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F51522a96-0869-4791-82f3-981000c2c67f) |Azure Bot Service cifra automáticamente el recurso para proteger sus datos y satisfacer los compromisos de cumplimiento y seguridad de la organización. De forma predeterminada, se usan claves de cifrado administradas por Microsoft. Para una mayor flexibilidad en la administración de claves o el control del acceso a su suscripción, seleccione claves administradas por el cliente, también conocidas como Bring your own key (BYOK). Más información acerca del cifrado de Azure Bot Service: [https://docs.microsoft.com/azure/bot-service/bot-service-encryption](https://docs.microsoft.com/azure/bot-service/bot-service-encryption). |audit, deny, disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Bot%20Service/BotService_CMKEnabled_Audit.json) |
|[Bot Service debe tener el modo aislado habilitado](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F52152f42-0dda-40d9-976e-abb1acdd611e) |Los bots deben establecerse en el modo "solo aislado". Este valor configura los canales de Bot Service que requieren que se deshabilite el tráfico a través de la red pública de Internet. |audit, deny, disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Bot%20Service/BotService_NetworkIsolatedEnabled_Audit.json) |
