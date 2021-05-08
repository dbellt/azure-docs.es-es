---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/28/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: b9b7eef56c122ec9bc3a59f1442b8ad935a8c71b
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108212469"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Los almacenes de claves deben tener habilitada la protección contra operaciones de purga](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0b60c0b2-2dc2-4e1c-b5c9-abbed971de53) |La eliminación malintencionada de un almacén de claves puede provocar una pérdida de datos permanente. Un usuario malintencionado de la organización puede eliminar y purgar los almacenes de claves. La protección contra purgas le protege frente a ataques internos mediante la aplicación de un período de retención obligatorio para almacenes de claves eliminados temporalmente. Ningún usuario de su organización o Microsoft podrá purgar los almacenes de claves durante el período de retención de eliminación temporal. |Audit, Deny, Disabled |[1.1.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/KeyVault_Recoverable_Audit.json) |
