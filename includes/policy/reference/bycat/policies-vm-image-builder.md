---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 06/11/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 68e6877d0efed3acae4bb5289e49c2cf650a841c
ms.sourcegitcommit: 942a1c6df387438acbeb6d8ca50a831847ecc6dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112019980"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Las plantillas de VM Image Builder deben usar Private Link](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2154edb9-244f-4741-9970-660785bccdaa) |Azure Private Link permite conectar la red virtual a servicios de Azure sin una dirección IP pública en el origen o el destino. La plataforma Private Link administra la conectividad entre el consumidor y los servicios a través de la red troncal de Azure. Mediante la asignación de puntos de conexión privados a los recursos de creación del generador de imágenes de máquina virtual, se reducen los riesgos de pérdida de datos. Más información sobre los vínculos privados en [https://docs.microsoft.com/azure/virtual-machines/linux/image-builder-networking#deploy-using-an-existing-vnet](https://docs.microsoft.com/azure/virtual-machines/linux/image-builder-networking#deploy-using-an-existing-vnet). |Audit, Disabled, Deny |[1.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/VM%20Image%20Builder/PrivateLinkEnabled_Audit.json) |
