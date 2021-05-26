---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 05/14/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 9c2335db77050a8571f2cea35c3917ac8339d9ec
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110074624"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Los dominios administrados de Azure Active Directory Domain Services solo deben usar el modo TLS 1.2.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3aa87b5a-7813-4b57-8a43-42dd9df5aaa7) |Use solo el modo TLS 1.2 para los dominios administrados. De manera predeterminada, Azure AD Domain Services permite el uso de cifrados tales como NTLM v1 y TLS v1. Aunque estos cifrados pueden ser necesarios para algunas aplicaciones heredadas, se consideran poco seguros y se pueden deshabilitar si no se necesitan. Cuando solo se habilita el modo TLS 1.2, cualquier cliente que realice una solicitud que no use TLS 1.2 producirá un error. Obtenga más información en [https://docs.microsoft.com/azure/active-directory-domain-services/secure-your-domain](https://docs.microsoft.com/azure/active-directory-domain-services/secure-your-domain). |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Azure%20Active%20Directory/AADDomainServices_TLS_Audit.json) |
