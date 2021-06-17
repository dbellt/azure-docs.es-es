---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 06/11/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: ebc2928a8840c1234aa85f7ec6aeda55e3246458
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112041414"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Debe habilitarse una solución de evaluación de vulnerabilidades en sus máquinas virtuales](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F501541f7-f7e7-4cd6-868c-4190fdad3ac9) |Audita las máquinas virtuales para detectar si ejecutan una solución de evaluación de vulnerabilidades admitida. Un componente fundamental de cada programa de seguridad y riesgo cibernético es la identificación y el análisis de las vulnerabilidades. El plan de tarifa estándar de Azure Security Center incluye el análisis de vulnerabilidades de las máquinas virtuales sin costo adicional. Además, Security Center puede implementar automáticamente esta herramienta. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_ServerVulnerabilityAssessment_Audit.json) |
