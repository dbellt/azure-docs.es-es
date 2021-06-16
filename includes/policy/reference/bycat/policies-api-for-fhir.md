---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 06/11/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: dd5ae3ebf0d6a231adb7656f64731e75bbade66f
ms.sourcegitcommit: 942a1c6df387438acbeb6d8ca50a831847ecc6dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112019561"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Azure API for FHIR debe usar una clave administrada por el cliente para cifrar los datos en reposo.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F051cba44-2429-45b9-9649-46cec11c7119) |Use claves administradas por el cliente para controlar el cifrado en reposo de los datos almacenados en Azure API for FHIR cuando exista un requisito normativo o de cumplimiento. Las claves administradas por el cliente también proporcionan cifrado doble, ya que agregan una segunda capa de cifrado además de la capa predeterminada que se creó mediante las claves administradas por el servicio. |audit, disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20for%20FHIR/HealthcareAPIs_EnableByok_Audit.json) |
|[Azure API for FHIR debe usar un vínculo privado.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1ee56206-5dd1-42ab-b02d-8aae8b1634ce) |Azure API for FHIR debe tener al menos una conexión de punto de conexión privado aprobada. Los clientes de una red virtual pueden acceder de forma segura a los recursos que tengan conexiones de punto de conexión privadas mediante vínculos privados. Para más información, visite [https://aka.ms/fhir-privatelink](https://aka.ms/fhir-privatelink). |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20for%20FHIR/HealthcareAPIs_PrivateLink_Audit.json) |
|[CORS no debe permitir que todos los dominios accedan a la API para FHIR](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0fea8f8a-4169-495d-8307-30ec335f387d). |El uso compartido de recursos entre orígenes (CORS) no debe permitir que todos los dominios accedan a API for FHIR. Para proteger la instancia de API for FHIR, elimine el acceso de todos los dominios y defina explícitamente los dominios que tienen permiso para conectarse. |audit, disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20for%20FHIR/HealthcareAPIs_RestrictCORSAccess_Audit.json) |
