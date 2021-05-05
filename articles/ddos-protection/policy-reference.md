---
title: Definiciones de directivas integradas para Azure DDoS Protection estándar
description: Enumera las definiciones de directivas integradas de Azure Policy para Azure DDoS Protection estándar. Estas definiciones de directivas integradas proporcionan enfoques comunes para administrar los recursos de Azure.
services: ddos-protection
documentationcenter: na
author: aletheatoh
ms.service: ddos-protection
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/28/2021
ms.author: yitoh
ms.custom: subject-policy-reference
ms.topic: include
ms.openlocfilehash: ecb4c8546f68607a54591ec8062222e5ec843150
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108177933"
---
# <a name="azure-policy-built-in-definitions-for-azure-ddos-protection-standard"></a>Definiciones de directivas integradas de Azure Policy para Azure DDoS Protection estándar

Esta página es un índice de las definiciones de directivas integradas de [Azure Policy](../governance/policy/overview.md) para Azure DDoS Protection estándar. Puede encontrar elementos integrados adicionales de Azure Policy para otros servicios en [Definiciones de elementos integrados de Azure Policy](../governance/policy/samples/built-in-policies.md).

El nombre de cada definición de directiva integrada se vincula a la definición de directiva en Azure Portal. Use el vínculo de la columna **Versión** para ver el origen en el [repositorio de GitHub de Azure Policy](https://github.com/Azure/azure-policy).

## <a name="azure-ddos-protection-standard"></a>Protección contra DDoS de Azure estándar

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Las redes virtuales deben protegerse con el estándar Azure DDoS Protection](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F94de2ad3-e0c1-4caf-ad78-5d47bbc83d3d).|Proteja sus redes virtuales contra ataques volumétricos y de protocolo con Azure DDoS Protection Estándar. Para más información, visite [https://aka.ms/ddosprotectiondocs](./ddos-protection-overview.md).|Modificación, auditoría y deshabilitación|[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkDdosStandard_Audit.json)|
|[Las direcciones IP públicas deben tener los registros de recursos habilitados para el estándar Azure DDoS Protection](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F752154a7-1e0f-45c6-a880-ac75a7e4f648).|Habilite los registros de recursos de las IP públicas en la configuración de diagnóstico para transmitirlos a un área de trabajo de Log Analytics. Obtenga visibilidad detallada sobre el tráfico de ataque y las acciones realizadas para mitigar los ataques DDoS mediante las notificaciones, informes y registros de flujo.|AuditIfNotExists, DeployIfNotExists, Disabled|[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/PublicIpDdosLogging_Audit.json)|


## <a name="next-steps"></a>Pasos siguientes

- Los elementos integrados se pueden encontrar en el [repositorio de GitHub de Azure Policy](https://github.com/Azure/azure-policy).
- Revise la [estructura de definición de Azure Policy](../governance/policy/concepts/definition-structure.md).
- Vea la [Descripción de los efectos de directivas](../governance/policy/concepts/effects.md).
