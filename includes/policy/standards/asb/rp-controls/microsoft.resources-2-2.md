---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/14/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 21a0a62a2ed6afe2e0b45732144fc79bb2ec38d5
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107510530"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[El aprovisionamiento automático del agente de Log Analytics debe estar habilitado en la suscripción](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F475aae12-b88a-4572-8b36-9b712b2b3a17) |A fin de supervisar las amenazas y vulnerabilidades de seguridad, Azure Security Center recopila datos de las máquinas virtuales de Azure. El agente de Log Analytics, anteriormente conocido como Microsoft Monitoring Agent (MMA), recopila los datos al leer distintas configuraciones relacionadas con la seguridad y distintos registros de eventos de la máquina y copiar los datos en el área de trabajo de Log Analytics para analizarlos. Se recomienda habilitar el aprovisionamiento automático para implementar automáticamente el agente en todas las máquinas virtuales de Azure admitidas y en las nuevas que se creen. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_Automatic_provisioning_log_analytics_monitoring_agent.json) |
|[El perfil de registro de Azure Monitor debe recopilar los registros de las categorías "write", "delete" y "action"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1a4e592a-6a6e-44a5-9814-e36264ca96e7) |Esta directiva garantiza que un perfil de registro recopile registros para las categorías "write", "delete" y "action". |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_CaptureAllCategories.json) |
|[Azure Monitor debe recopilar los registros de actividad de todas las regiones](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F41388f1c-2db0-4c25-95b2-35d7f5ccbfa9) |Esta directiva audita el perfil de registro de Azure Monitor que no exporta actividades de todas las regiones admitidas de Azure, incluida la global. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_CaptureAllRegions.json) |
