---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/14/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 024d5a8b71d269da478bc8a9e3559b1eb4b92c63
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107505468"
---
## <a name="cmmc-level-3"></a>CMMC nivel 3

Para ver cómo se corresponden las integraciones de Azure Policy disponibles para todos los mapas de servicio de Azure con este estándar de cumplimiento, consulte [Detalles de la iniciativa integrada de cumplimiento normativo CMMC nivel 3](../../../../articles/governance/policy/samples/cmmc-l3.md).
Para más información sobre este estándar de cumplimiento, consulte [Certificación del modelo de madurez de ciberseguridad (CMMC)](https://www.acq.osd.mil/cmmc/docs/CMMC_Model_Main_20200203.pdf).

|Domain |Id. de control |Título de control |Directiva<br /><sub>(Azure Portal)</sub> |Versión de la directiva<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Control de acceso |AC.1.001 |Limita el acceso del sistema de información a los usuarios autorizados, los procesos que actúan en nombre de los usuarios autorizados y los dispositivos (incluidos otros sistemas de información). |[CORS no debe permitir que todos los dominios accedan a la API para FHIR](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0fea8f8a-4169-495d-8307-30ec335f387d). |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20for%20FHIR/HealthcareAPIs_RestrictCORSAccess_Audit.json) |
|Control de acceso |AC.1.002 |Limita el acceso del sistema de información a los tipos de transacciones y funciones que los usuarios autorizados pueden ejecutar. |[CORS no debe permitir que todos los dominios accedan a la API para FHIR](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0fea8f8a-4169-495d-8307-30ec335f387d). |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20for%20FHIR/HealthcareAPIs_RestrictCORSAccess_Audit.json) |
|Control de acceso |AC.2.016 |Controla el flujo de CUI de acuerdo con las autorizaciones aprobadas. |[CORS no debe permitir que todos los dominios accedan a la API para FHIR](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0fea8f8a-4169-495d-8307-30ec335f387d). |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20for%20FHIR/HealthcareAPIs_RestrictCORSAccess_Audit.json) |
|Administración de la configuración |CM.3.068 |Restringe, deshabilita o impide el uso de programas, funciones, puertos, protocolos y servicios no esenciales. |[CORS no debe permitir que todos los dominios accedan a la API para FHIR](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0fea8f8a-4169-495d-8307-30ec335f387d). |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20for%20FHIR/HealthcareAPIs_RestrictCORSAccess_Audit.json) |
|Protección del sistema y de las comunicaciones |SC.3.177 |Emplea una criptografía validada por FIPS para proteger la confidencialidad de CUI. |[Azure API for FHIR debe usar una clave administrada por el cliente para cifrar los datos en reposo.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F051cba44-2429-45b9-9649-46cec11c7119) |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20for%20FHIR/HealthcareAPIs_EnableByok_Audit.json) |
|Protección del sistema y de las comunicaciones |SC.3.183 |Deniega el tráfico de las comunicaciones de red de forma predeterminada y solo permite el tráfico de las comunicaciones de red de forma excepcional (es decir, deniega todo el tráfico y permite las excepciones). |[CORS no debe permitir que todos los dominios accedan a la API para FHIR](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0fea8f8a-4169-495d-8307-30ec335f387d). |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20for%20FHIR/HealthcareAPIs_RestrictCORSAccess_Audit.json) |

