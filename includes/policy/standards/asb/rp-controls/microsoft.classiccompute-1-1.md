---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/14/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: ddb285c31129ec0ac9a01979ace04f6d201798e0
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107510541"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Las máquinas virtuales accesibles desde Internet deben estar protegidas con grupos de seguridad de red](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff6de0be7-9a8a-4b8a-b349-43cf02d22f7c) |Proteja sus máquinas virtuales de posibles amenazas limitando el acceso a ellas con grupos de seguridad de red (NSG). Más información sobre cómo controlar el tráfico con los grupos de seguridad de red en [https://aka.ms/nsg-doc](https://aka.ms/nsg-doc). |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnInternetFacingVirtualMachines_Audit.json) |
|[El reenvío de IP en la máquina virtual debe estar deshabilitado](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fbd352bd5-2853-4985-bf0d-73806b4a5744) |Habilitar el reenvío de IP en la NIC de la máquina virtual permite que la máquina reciba tráfico dirigido a otros destinos. El reenvío de IP rara vez es necesario (por ejemplo, cuando se usa la máquina virtual como una aplicación virtual de red) y, por lo tanto, el equipo de seguridad de red debe revisarlo. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_IPForwardingOnVirtualMachines_Audit.json) |
|[Se deben cerrar los puertos de administración en las máquinas virtuales](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22730e10-96f6-4aac-ad84-9383d35b5917) |Los puertos de administración remota abiertos exponen la máquina virtual a un alto nivel de riesgo de recibir ataques basados en Internet. Estos ataques intentan averiguar las credenciales por medio de fuerza bruta a fin de obtener acceso de administrador a la máquina |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_OpenManagementPortsOnVirtualMachines_Audit.json) |
