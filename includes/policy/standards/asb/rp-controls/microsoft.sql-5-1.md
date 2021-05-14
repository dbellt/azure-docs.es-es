---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 05/04/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: cf4cb251ea84c6913502b4d518a0266ec93b8399
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108791242"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La evaluación de vulnerabilidades debe estar habilitada en Instancia administrada de SQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1b7aa243-30e4-4c9e-bca8-d0d3022b634a) |Audita cada servicio SQL Managed Instance que no tiene habilitado los exámenes de evaluación de vulnerabilidades periódicos. La evaluación de vulnerabilidades permite detectar las vulnerabilidades potenciales de la base de datos, así como hacer un seguimiento y ayudar a corregirlas. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/VulnerabilityAssessmentOnManagedInstance_Audit.json) |
|[La evaluación de vulnerabilidades debe estar activada en sus servidores de SQL Server](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fef2a8f2a-b3d9-49cd-a8a8-9a3aaaf647d9) |Audita los servidores Azure SQL Server que no tienen habilitados los exámenes de evaluación de vulnerabilidades periódicos. La evaluación de vulnerabilidades permite detectar las vulnerabilidades potenciales de la base de datos, así como hacer un seguimiento y ayudar a corregirlas. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/VulnerabilityAssessmentOnServer_Audit.json) |
