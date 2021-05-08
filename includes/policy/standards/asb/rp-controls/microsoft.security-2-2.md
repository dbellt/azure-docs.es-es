---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/28/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 14e5c1cd5df9ffe7b21755dba11d388a10430a28
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108212451"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[El aprovisionamiento automático del agente de Log Analytics debe estar habilitado en la suscripción](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F475aae12-b88a-4572-8b36-9b712b2b3a17) |A fin de supervisar las amenazas y vulnerabilidades de seguridad, Azure Security Center recopila datos de las máquinas virtuales de Azure. El agente de Log Analytics, anteriormente conocido como Microsoft Monitoring Agent (MMA), recopila los datos al leer distintas configuraciones relacionadas con la seguridad y distintos registros de eventos de la máquina y copiar los datos en el área de trabajo de Log Analytics para analizarlos. Se recomienda habilitar el aprovisionamiento automático para implementar automáticamente el agente en todas las máquinas virtuales de Azure admitidas y en las nuevas que se creen. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_Automatic_provisioning_log_analytics_monitoring_agent.json) |
