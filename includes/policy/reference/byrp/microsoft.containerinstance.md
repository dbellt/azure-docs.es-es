---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/14/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: b9720165ce5ff27e1e67e074afa64d21a78efaa6
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107504170"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[El grupo de contenedores de la instancia de Azure Container Instances se debe implementar en una red virtual](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8af8f826-edcb-4178-b35f-851ea6fea615) |Proteja la comunicación entre los contenedores con redes virtuales de Azure. Cuando se especifica una red virtual, los recursos de la red virtual pueden comunicarse entre sí de forma segura y privada. |Audit, Disabled, Deny |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Container%20Instance/ContainerInstance_VNET_Audit.json) |
|[El grupo de contenedores de la instancia de Azure Container Instances debe usar una clave administrada por el cliente para el cifrado](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0aa61e00-0a01-4a3c-9945-e93cffedf0e6) |Proteja los contenedores con mayor flexibilidad mediante el uso de claves administradas por el cliente. Cuando se especifica una clave administrada por el cliente, esa clave se usa para proteger y controlar el acceso a la clave que cifra los datos. El uso de claves administradas por el cliente proporciona funcionalidades adicionales para controlar la rotación de la clave de cifrado de claves o para borrar datos mediante criptografía. |Audit, Disabled, Deny |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Container%20Instance/ContainerInstance_CMK_Audit.json) |
