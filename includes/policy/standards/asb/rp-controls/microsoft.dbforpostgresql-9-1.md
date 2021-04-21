---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/14/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 18cbe3f50ee107d710b21e95f6f4774ecd65263c
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107511258"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La copia de seguridad con redundancia geográfica debe estar habilitada para Azure Database for PostgreSQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F48af4db5-9b8b-401c-8e74-076be876a430) |Azure Database for PostgreSQL le permite elegir la opción de redundancia para el servidor de bases de datos. Se puede establecer en un almacenamiento de copia de seguridad con redundancia geográfica donde los datos no solo se almacenan dentro de la región en la que se hospeda el servidor, sino que también se replican en una región emparejada para proporcionar la opción de recuperación en caso de que se produzca un error en la región. La configuración de almacenamiento con redundancia geográfica para copia de seguridad solo se permite durante la creación del servidor. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/GeoRedundant_DBForPostgreSQL_Audit.json) |
