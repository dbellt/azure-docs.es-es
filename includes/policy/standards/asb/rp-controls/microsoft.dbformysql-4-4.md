---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/21/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: ccb53768a98f6b29469d6a555aa60e72f26a91eb
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107880572"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Exigir una conexión SSL debe estar habilitado en los servidores de bases de datos MySQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe802a67a-daf5-4436-9ea6-f6d821dd0c5d) |Azure Database for MySQL permite conectar el servidor de Azure Database for MySQL con aplicaciones cliente mediante Capa de sockets seguros (SSL). La aplicación de conexiones SSL entre el servidor de bases de datos y las aplicaciones cliente facilita la protección frente a ataques de tipo "Man in the middle" al cifrar el flujo de datos entre el servidor y la aplicación. Esta configuración exige que SSL esté siempre habilitado para el acceso al servidor de bases de datos. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/MySQL_EnableSSL_Audit.json) |
