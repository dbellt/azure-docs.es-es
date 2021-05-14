---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 05/04/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: acde4db609dedc9f2ccce409102a152e5d0ca425
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108792469"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Los almacenes de claves deben tener habilitada la protección contra operaciones de purga](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0b60c0b2-2dc2-4e1c-b5c9-abbed971de53) |La eliminación malintencionada de un almacén de claves puede provocar una pérdida de datos permanente. Un usuario malintencionado de la organización puede eliminar y purgar los almacenes de claves. La protección contra purgas le protege frente a ataques internos mediante la aplicación de un período de retención obligatorio para almacenes de claves eliminados temporalmente. Ningún usuario de su organización o Microsoft podrá purgar los almacenes de claves durante el período de retención de eliminación temporal. |Audit, Deny, Disabled |[1.1.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/KeyVault_Recoverable_Audit.json) |
