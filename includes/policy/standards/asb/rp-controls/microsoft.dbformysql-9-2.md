---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 05/14/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 9c5703e348b407b73a0994b3e74a633d4ef9d296
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110165847"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La copia de seguridad con redundancia geográfica debe estar habilitada para Azure Database for MySQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F82339799-d096-41ae-8538-b108becf0970) |Azure Database for MySQL le permite elegir la opción de redundancia para el servidor de bases de datos. Se puede establecer en un almacenamiento de copia de seguridad con redundancia geográfica donde los datos no solo se almacenan dentro de la región en la que se hospeda el servidor, sino que también se replican en una región emparejada para proporcionar la opción de recuperación en caso de que se produzca un error en la región. La configuración de almacenamiento con redundancia geográfica para copia de seguridad solo se permite durante la creación del servidor. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/GeoRedundant_DBForMySQL_Audit.json) |
