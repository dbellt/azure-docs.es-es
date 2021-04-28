---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/21/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: e721c25b3374e908c70fcec463625daeea0165a0
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107876137"
---
## <a name="azure-security-benchmark"></a>Prueba comparativa de la seguridad de Azure

[Azure Security Benchmark](/security/benchmark/azure/introduction) proporciona recomendaciones sobre cómo puede proteger sus soluciones de nube en Azure. Para ver cómo este servicio se asigna por completo a Azure Security Benchmark, consulte los [archivos de asignación de Az Security Benchmark](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

Para revisar el modo en que las integraciones de Azure Policy disponibles para todos los servicios de Azure se asignan a este estándar de cumplimiento, consulte [Cumplimiento normativo de Azure Policy: Azure Security Benchmark](../../../../articles/governance/policy/samples/azure-security-benchmark.md).

|Domain |Id. de control |Título de control |Directiva<br /><sub>(Azure Portal)</sub> |Versión de la directiva<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Administración de identidades |IM-1 |Unificación en Azure Active Directory como sistema central de identidad y autenticación |[Los clústeres de Service Fabric solo deben usar Azure Active Directory para la autenticación de cliente](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb54ed75b-3e1a-44ac-a333-05ba39b99ff0) |[1.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Service%20Fabric/ServiceFabric_AuditADAuth_Audit.json) |
|Protección de datos |DP-5 |Cifrado de datos confidenciales en reposo |[Se debe establecer la propiedad ClusterProtectionLevel en EncryptAndSign en los clústeres de Service Fabric](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F617c02be-7f02-4efd-8836-3180d47b6c68) |[1.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Service%20Fabric/ServiceFabric_AuditClusterProtectionLevel_Audit.json) |

## <a name="azure-security-benchmark-v1"></a>Azure Security Benchmark v1

[Azure Security Benchmark](/security/benchmark/azure/introduction) proporciona recomendaciones sobre cómo puede proteger sus soluciones de nube en Azure. Para ver cómo este servicio se asigna por completo a Azure Security Benchmark, consulte los [archivos de asignación de Az Security Benchmark](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

Para revisar el modo en que las integraciones de Azure Policy disponibles para todos los servicios de Azure se asignan a este estándar de cumplimiento, consulte [Cumplimiento normativo de Azure Policy: Azure Security Benchmark](../../../../articles/governance/policy/samples/azure-security-benchmark.md).

|Domain |Id. de control |Título de control |Directiva<br /><sub>(Azure Portal)</sub> |Versión de la directiva<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Identidad y Access Control |3.9 |Uso de Azure Active Directory |[Los clústeres de Service Fabric solo deben usar Azure Active Directory para la autenticación de cliente](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb54ed75b-3e1a-44ac-a333-05ba39b99ff0) |[1.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Service%20Fabric/ServiceFabric_AuditADAuth_Audit.json) |
|Protección de datos |4.8 |Cifrado de información confidencial en reposo |[Se debe establecer la propiedad ClusterProtectionLevel en EncryptAndSign en los clústeres de Service Fabric](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F617c02be-7f02-4efd-8836-3180d47b6c68) |[1.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Service%20Fabric/ServiceFabric_AuditClusterProtectionLevel_Audit.json) |

## <a name="cmmc-level-3"></a>CMMC nivel 3

Para ver cómo se corresponden las integraciones de Azure Policy disponibles para todos los mapas de servicio de Azure con este estándar de cumplimiento, consulte [Detalles de la iniciativa integrada de cumplimiento normativo CMMC nivel 3](../../../../articles/governance/policy/samples/cmmc-l3.md).
Para más información sobre este estándar de cumplimiento, consulte [Certificación del modelo de madurez de ciberseguridad (CMMC)](https://www.acq.osd.mil/cmmc/docs/CMMC_Model_Main_20200203.pdf).

|Domain |Id. de control |Título de control |Directiva<br /><sub>(Azure Portal)</sub> |Versión de la directiva<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Protección del sistema y de las comunicaciones |SC.3.177 |Emplea criptografía validada mediante FIPS cuando se usa para proteger la confidencialidad de CUI. |[Se debe establecer la propiedad ClusterProtectionLevel en EncryptAndSign en los clústeres de Service Fabric](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F617c02be-7f02-4efd-8836-3180d47b6c68) |[1.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Service%20Fabric/ServiceFabric_AuditClusterProtectionLevel_Audit.json) |
|Protección del sistema y de las comunicaciones |SC.3.191 |Protege la confidencialidad de CUI en reposo. |[Se debe establecer la propiedad ClusterProtectionLevel en EncryptAndSign en los clústeres de Service Fabric](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F617c02be-7f02-4efd-8836-3180d47b6c68) |[1.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Service%20Fabric/ServiceFabric_AuditClusterProtectionLevel_Audit.json) |

## <a name="iso-270012013"></a>ISO 27001:2013

Si desea ver la correspondencia que existe entre las integraciones de Azure Policy disponibles para todos los servicios de Azure y este estándar de cumplimiento, consulte este artículo sobre el [cumplimiento normativo de Azure Policy: ISO 27001:2013](../../../../articles/governance/policy/samples/iso-27001.md).
Para más información sobre este estándar de cumplimiento, consulte [ISO 27001:2013](https://www.iso.org/isoiec-27001-information-security.html).

|Domain |Id. de control |Título de control |Directiva<br /><sub>(Azure Portal)</sub> |Versión de la directiva<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Control de acceso |9.2.3 |Administración de los derechos de acceso con privilegios |[Los clústeres de Service Fabric solo deben usar Azure Active Directory para la autenticación de cliente](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb54ed75b-3e1a-44ac-a333-05ba39b99ff0) |[1.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Service%20Fabric/ServiceFabric_AuditADAuth_Audit.json) |
|Criptografía |10.1.1 |Directiva sobre el uso de controles criptográficos |[Se debe establecer la propiedad ClusterProtectionLevel en EncryptAndSign en los clústeres de Service Fabric](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F617c02be-7f02-4efd-8836-3180d47b6c68) |[1.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Service%20Fabric/ServiceFabric_AuditClusterProtectionLevel_Audit.json) |

## <a name="new-zealand-ism-restricted"></a>ISM restringido de Nueva Zelanda

Para consultar la correspondencia que existe entre las integraciones de Azure Policy disponibles para todos los servicios de Azure y este estándar de cumplimiento, consulte este artículo sobre el [cumplimiento normativo de Azure Policy y la restricción de ISM en Nueva Zelanda](../../../../articles/governance/policy/samples/new-zealand-ism.md).
Para más información acerca de este estándar de cumplimiento, consulte [ISM restringido de Nueva Zelanda](https://www.nzism.gcsb.govt.nz/).

|Domain |Id. de control |Título de control |Directiva<br /><sub>(Azure Portal)</sub> |Versión de la directiva<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Control de acceso y contraseñas |AC-2 |16.1.32 Identificación del usuario del sistema |[Los clústeres de Service Fabric solo deben usar Azure Active Directory para la autenticación de cliente](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb54ed75b-3e1a-44ac-a333-05ba39b99ff0) |[1.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Service%20Fabric/ServiceFabric_AuditADAuth_Audit.json) |

## <a name="nist-sp-800-53-r4"></a>NIST SP 800-53 R4

Para revisar el modo en que las integraciones de Azure Policy disponibles para todos los servicios de Azure se corresponden a este estándar de cumplimiento, consulte [Cumplimiento normativo de Azure Policy: NIST SP 800-53 R4](../../../../articles/governance/policy/samples/nist-sp-800-53-r4.md).
Para más información acerca de este estándar normativo, consulte [NIST SP 800-53 R4](https://nvd.nist.gov/800-53).

|Domain |Id. de control |Título de control |Directiva<br /><sub>(Azure Portal)</sub> |Versión de la directiva<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Control de acceso |AC-2 (7) |Administración de cuentas \| Esquemas basados en roles |[Los clústeres de Service Fabric solo deben usar Azure Active Directory para la autenticación de cliente](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb54ed75b-3e1a-44ac-a333-05ba39b99ff0) |[1.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Service%20Fabric/ServiceFabric_AuditADAuth_Audit.json) |

