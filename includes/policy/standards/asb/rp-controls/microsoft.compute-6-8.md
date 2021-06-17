---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 06/11/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 7ac67043249a6af7ed36b5f83e217d4fc2e7ca0a
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112039404"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Los controles de aplicaciones adaptables para definir aplicaciones seguras deben estar habilitados en las máquinas](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F47a6b606-51aa-4496-8bb7-64b11cf66adc) |Habilite controles de aplicaciones para definir la lista de aplicaciones seguras conocidas que se ejecutan en las máquinas, y recibir avisos cuando se ejecuten otras aplicaciones. Esta directiva también ayuda a proteger las máquinas frente al malware. Para simplificar el proceso de configuración y mantenimiento de las reglas, Security Center usa el aprendizaje automático para analizar las aplicaciones que se ejecutan en cada máquina y sugerir la lista de aplicaciones seguras conocidas. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_AdaptiveApplicationControls_Audit.json) |
