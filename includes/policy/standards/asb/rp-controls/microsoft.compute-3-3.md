---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 05/14/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 43b7476a8a005a29d3c30ef7bd7e18f706a95d98
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110164212"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Auditar las máquinas Windows que no tengan ninguno de los miembros especificados en el grupo de administradores](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F30f71ea1-ac77-4f26-9fc5-2d926bbd4ba7) |Requiere que los requisitos previos se implementen en el ámbito de asignación de directivas. Para más información, consulte [https://aka.ms/gcpol](https://aka.ms/gcpol). Las máquinas no son compatibles si el grupo de administradores locales no contiene uno o más miembros de los que se enumeran en el parámetro de la directiva. |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AdministratorsGroupMembersToInclude_AINE.json) |
|[Auditar las máquinas Windows que tengan cuentas adicionales en el grupo de administradores](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3d2a3320-2a72-4c67-ac5f-caa40fbee2b2) |Requiere que los requisitos previos se implementen en el ámbito de asignación de directivas. Para más información, consulte [https://aka.ms/gcpol](https://aka.ms/gcpol). Las máquinas no son compatibles si el grupo de administradores locales contiene miembros que no se enumeran en el parámetro de la directiva. |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AdministratorsGroupMembers_AINE.json) |
|[Auditar las máquinas Windows que tengan los miembros especificados en el grupo de administradores](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F69bf4abd-ca1e-4cf6-8b5a-762d42e61d4f) |Requiere que los requisitos previos se implementen en el ámbito de asignación de directivas. Para más información, consulte [https://aka.ms/gcpol](https://aka.ms/gcpol). Las máquinas no son compatibles si el grupo de administradores locales contiene uno o varios de los miembros enumerados en el parámetro de la directiva. |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AdministratorsGroupMembersToExclude_AINE.json) |
