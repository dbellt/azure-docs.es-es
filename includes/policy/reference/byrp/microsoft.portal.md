---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/31/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 5ad3af71227ae1f6d6701102bf29c042454a965c
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106094266"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Los paneles compartidos no deben tener iconos de Markdown con contenido alineado](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F04c655fe-0ac7-48ae-9a32-3a2e208c7624) |No permitir la creación de un panel compartido que tenga contenido alineado en iconos de Markdown y exigir que el contenido se almacene como un archivo Markdown hospedado en línea. Si usa el contenido insertado en el icono de Markdown, no podrá administrar el cifrado del contenido. Mediante la configuración de su propio almacenamiento, puede aplicar cifrado y cifrado doble, e incluso aportar sus propias claves. Al habilitar esta directiva, se impide que los usuarios usen 2020-09-01-preview o una versión anterior de la API de REST de paneles compartidos. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Portal/SharedDashboardInlineContent_Deny.json) |
