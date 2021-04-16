---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/31/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 289a2fe7fecc3e28d646e98692c1d5ada4ea7fa8
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106097456"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Los servidores SQL Server con auditoría en el destino de la cuenta de almacenamiento se deben configurar con una retención de 90 días o superior.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F89099bee-89e0-4b26-a5f4-165451757743) |Con fines de investigación de incidentes, se recomienda establecer la retención de datos de auditoría de las instancias de SQL Server en el destino de la cuenta de almacenamiento en al menos 90 días. Confirme que cumple las reglas de retención necesarias para las regiones en las que trabaja. A veces, es necesario para cumplir con los estándares normativos. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServerAuditingRetentionDays_Audit.json) |
