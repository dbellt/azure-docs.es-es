---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 06/11/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: df4af57a55acb0e89bf6155de9c6a5aded9f9a46
ms.sourcegitcommit: 942a1c6df387438acbeb6d8ca50a831847ecc6dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112019851"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[\[Versión preliminar\]: Configurar almacenes de Azure Recovery Services para usar zonas de DNS privado](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F942bd215-1a66-44be-af65-6a1c0318dbe2) |Use zonas DNS privadas para invalidar la resolución DNS de un punto de conexión privado. Una zona DNS privada se vincula a la red virtual para resolver los almacenes de Recovery Services. Más información en: [https://aka.ms/privatednszone](https://aka.ms/privatednszone). |DeployIfNotExists, Disabled |[1.0.0-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Site%20Recovery/SiteRecovery_PrivateDnsZones_DeployIfNotExist.json) |
|[\[Versión preliminar\]: Configurar puntos de conexión privados en los almacenes de Azure Recovery Services](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe95a8a5c-0987-421f-84ab-df4d88ebf7d1) |Los puntos de conexión privados conectan la red virtual a los servicios de Azure sin una dirección IP pública en el origen o el destino. Al asignar puntos de conexión privados a los recursos de recuperación del sitio de los almacenes de Recovery Services, puede reducir los riesgos de pérdida de datos. Para usar vínculos privados, la identidad de servicio administrada debe asignarse a los almacenes de Recovery Services. Más información sobre los vínculos privados en [https://docs.microsoft.com/azure/site-recovery/azure-to-azure-how-to-enable-replication-private-endpoints](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-how-to-enable-replication-private-endpoints). |DeployIfNotExists, Disabled |[1.0.0-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Site%20Recovery/SiteRecovery_PrivateEndpoints_DeployIfNotExist.json) |
|[\[Versión preliminar\]: Los almacenes de Recovery Services deben usar un vínculo privado](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F11e3da8c-1d68-4392-badd-0ff3c43ab5b0) |Azure Private Link permite conectar la red virtual a servicios de Azure sin una dirección IP pública en el origen o el destino. La plataforma Private Link administra la conectividad entre el consumidor y los servicios a través de la red troncal de Azure. Al asignar puntos de conexión privados a Azure Recovery Services almacenes, se reduce el riesgo de pérdida de datos. Obtenga más información sobre los vínculos privados para Azure Site Recovery en [https://aka.ms/HybridScenarios-PrivateLink](https://aka.ms/HybridScenarios-PrivateLink) y [https://aka.ms/AzureToAzure-PrivateLink](https://aka.ms/AzureToAzure-PrivateLink). |Audit, Disabled |[1.0.0-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Site%20Recovery/RecoveryServices_SiteRecovery_PrivateEndpoint_Audit.json) |
