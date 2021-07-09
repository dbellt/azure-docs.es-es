---
title: Características de Azure Security Center según el sistema operativo, el tipo de máquina y la nube
description: Obtenga información acerca de las características de Azure Security Center que están disponibles según el sistema operativo, el tipo de máquina y la implementación en la nube.
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: overview
ms.date: 05/13/2021
ms.author: memildin
ms.openlocfilehash: b68a9946908697d6e9113af586a134b0b3d8784a
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110468346"
---
# <a name="feature-coverage-for-machines"></a>Cobertura de características para las máquinas

En las dos tablas siguientes se muestran las características de Azure Security Center disponibles para las máquinas virtuales y los servidores Windows y Linux.

## <a name="supported-features-for-virtual-machines-and-servers"></a>Características admitidas para máquinas virtuales y servidores <a name="vm-server-features"></a>

### <a name="windows-machines"></a>[**Máquinas Windows**](#tab/features-windows)

|**Característica**|**Azure Virtual Machines**|**Azure Virtual Machine Scale Sets**|**Máquinas habilitadas para Azure Arc**|**Azure Defender necesario**
|----|:----:|:----:|:----:|:----:|
|[Integración de Microsoft Defender for Endpoint](security-center-wdatp.md)|✔</br>✔ (en versiones admitidas)|✔</br>✔ (en versiones admitidas)|✔|Sí|
|[Análisis del comportamiento de máquinas virtuales (y alertas de seguridad)](alerts-reference.md)|✔|✔|✔|Sí|
|[Alertas de seguridad sin archivos](alerts-reference.md#alerts-windows)|✔|✔|✔|Sí|
|[Alertas de seguridad basadas en la red](other-threat-protections.md#network-layer)|✔|✔|-|Sí|
|[Acceso de máquina virtual Just-In-Time](security-center-just-in-time.md)|✔|-|-|Sí|
|[Evaluación de vulnerabilidades nativa](deploy-vulnerability-assessment-vm.md#overview-of-the-integrated-vulnerability-scanner)|✔|-|✔|Sí|
|[Supervisión de la integridad de los archivos](security-center-file-integrity-monitoring.md)|✔|✔|✔|Sí|
|[Controles de aplicación adaptables](security-center-adaptive-application.md)|✔|-|✔|Sí|
|[Mapa de red](security-center-network-recommendations.md#network-map)|✔|✔|-|Sí|
|[Protección de red adaptable](security-center-adaptive-network-hardening.md)|✔|-|-|Sí|
|[Panel e informes de cumplimiento normativo](security-center-compliance-dashboard.md)|✔|✔|✔|Sí|
|Recomendaciones y protección frente a amenazas en contenedores IaaS hospedados en Docker|-|-|-|Sí|
|Evaluación de revisiones de SO que faltan|✔|✔|✔|Azure: No<br><br>Habilitada para Arc: Sí|
|Evaluación de configuraciones de seguridad incorrectas|✔|✔|✔|Azure: No<br><br>Habilitada para Arc: Sí|
|[Evaluación de EndPoint Protection](security-center-services.md#supported-endpoint-protection-solutions-)|✔|✔|✔|Azure: No<br><br>Habilitada para Arc: Sí|
|Evaluación de Disk Encryption|✔</br>(para [escenarios admitidos](../virtual-machines/windows/disk-encryption-windows.md#unsupported-scenarios))|✔|-|No|
|Evaluación de vulnerabilidades de terceros|✔|-|✔|No|
|[Evaluación de la seguridad de red](security-center-network-recommendations.md)|✔|✔|-|No|


### <a name="linux-machines"></a>[**Máquinas Linux**](#tab/features-linux)

|**Característica**|**Azure Virtual Machines**|**Azure Virtual Machine Scale Sets**|**Máquinas habilitadas para Azure Arc**|**Azure Defender necesario**
|----|:----:|:----:|:----:|:----:|
|[Integración de Microsoft Defender for Endpoint](security-center-wdatp.md)|-|-|-|-|
|[Análisis del comportamiento de máquinas virtuales (y alertas de seguridad)](./azure-defender.md)|✔</br>✔ (en versiones admitidas)|✔</br>✔ (en versiones admitidas)|✔|Sí|
|[Alertas de seguridad sin archivos](alerts-reference.md#alerts-windows)|-|-|-|Sí|
|[Alertas de seguridad basadas en la red](other-threat-protections.md#network-layer)|✔|✔|-|Sí|
|[Acceso de máquina virtual Just-In-Time](security-center-just-in-time.md)|✔|-|-|Sí|
|[Evaluación de vulnerabilidades nativa](deploy-vulnerability-assessment-vm.md#overview-of-the-integrated-vulnerability-scanner)|✔|-|✔|Sí|
|[Supervisión de la integridad de los archivos](security-center-file-integrity-monitoring.md)|✔|✔|✔|Sí|
|[Controles de aplicación adaptables](security-center-adaptive-application.md)|✔|-|✔|Sí|
|[Mapa de red](security-center-network-recommendations.md#network-map)|✔|✔|-|Sí|
|[Protección de red adaptable](security-center-adaptive-network-hardening.md)|✔|-|-|Sí|
|[Panel e informes de cumplimiento normativo](security-center-compliance-dashboard.md)|✔|✔|✔|Sí|
|Recomendaciones y protección frente a amenazas en contenedores IaaS hospedados en Docker|✔|✔|✔|Sí|
|Evaluación de revisiones de SO que faltan|✔|✔|✔|Azure: No<br><br>Habilitada para Arc: Sí|
|Evaluación de configuraciones de seguridad incorrectas|✔|✔|✔|Azure: No<br><br>Habilitada para Arc: Sí|
|[Evaluación de EndPoint Protection](security-center-services.md#supported-endpoint-protection-solutions-)|-|-|-|No|
|Evaluación de Disk Encryption|✔</br>(para [escenarios admitidos](../virtual-machines/windows/disk-encryption-windows.md#unsupported-scenarios))|✔|-|No|
|Evaluación de vulnerabilidades de terceros|✔|-|✔|No|
|[Evaluación de la seguridad de red](security-center-network-recommendations.md)|✔|✔|-|No|

--- 


> [!TIP]
>Para experimentar con características que solo están disponibles con Azure Defender, puede inscribirse en una evaluación de treinta días. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/security-center/)para obtener más información.


## <a name="supported-endpoint-protection-solutions"></a>Soluciones de protección de punto de conexión compatibles <a name="endpoint-supported"></a>

En la tabla siguiente se proporciona una matriz de:

 - Si puede usar Azure Security Center para instalar cada solución para usted.
 - Qué soluciones de protección de punto de conexión puede detectar Security Center. Si se detecta una de las soluciones de protección de punto de conexión de esta lista, Security Center no recomendará instalar ninguna.

Para más información sobre cuándo se generan recomendaciones para cada una de estas protecciones, vea [Evaluación y recomendaciones de Endpoint Protection](security-center-endpoint-protection.md).

| Endpoint Protection| Plataformas | Instalación de Security Center | Detección de Security Center |
|------|------|-----|-----|
| Antivirus de Microsoft Defender| Windows Server 2016 o posterior| No, se integra en el sistema operativo.| Sí |
| System Center Endpoint Protection (Microsoft Antimalware) | Windows Server 2012 R2, 2012, 2008 R2 (consulte la nota que hay a continuación) | Mediante extensión | Sí |
| Trend Micro: Deep Security | Familia de Windows Server  | No | Sí |
| Symantec v12.1.1100+| Familia de Windows Server  | No | Sí |
| McAfee v10 o posterior | Familia de Windows Server  | No | Sí |
| McAfee v10 o posterior | Familia de Linux Server  | No | Sí |
| Sophos V9+| Familia de Linux Server  | No | Sí |

> [!NOTE]
> La detección de System Center Endpoint Protection (SCEP) en una máquina virtual de Windows Server 2008 R2 requiere que SCEP se instale después de PowerShell 0 (v3.0 o una versión superior).



## <a name="feature-support-in-government-clouds"></a>Compatibilidad de características en las nubes gubernamentales

| Servicio o característica | US Gov | China Gov |
|------|:----:|:----:|
|[Acceso a máquinas virtuales Just-In-Time](security-center-just-in-time.md) (1)|✔|✔|
|[Supervisión de la integridad de los archivos](security-center-file-integrity-monitoring.md) (1)|✔|✔|
|[Controles de aplicaciones adaptables](security-center-adaptive-application.md) (1)|✔|✔|
|[Protección de red adaptable](security-center-adaptive-network-hardening.md) (1)|-|-|
|[Protección de hosts de Docker](harden-docker-hosts.md) (1)|✔|✔|
|[Evaluación integrada de vulnerabilidades para las máquinas](deploy-vulnerability-assessment-vm.md) (1)|-|-|
|[Microsoft Defender para puntos de conexión](harden-docker-hosts.md) (1)|✔|-|
|[Conexión de cuentas de AWS](quickstart-onboard-aws.md) (1)|-|-|
|[Conexión de cuentas de GCP](quickstart-onboard-gcp.md) (1)|-|-|
|[Exportación continua](continuous-export.md)|✔|✔|
|[Automatización de flujos de trabajo (versión preliminar)](workflow-automation.md)|✔|✔|
|[Reglas de exención de recomendaciones](exempt-resource.md)|-|-|
|[Reglas de eliminación de alertas](alerts-suppression-rules.md)|✔|✔|
|[Notificaciones de correo electrónico para alertas de seguridad](security-center-provide-security-contact-details.md)|✔|✔|
|[Inventario de recursos](asset-inventory.md)|✔|✔|
|[Azure Defender para App Service](defender-for-app-service-introduction.md)|-|-|
|[Azure Defender para Storage](defender-for-storage-introduction.md)|✔|-|
|[Azure Defender para SQL](defender-for-sql-introduction.md)|✔|✔ (2)|
|[Azure Defender para Key Vault](defender-for-key-vault-introduction.md)|-|-|
|[Azure Defender para Resource Manager](defender-for-resource-manager-introduction.md)|-|-|
|[Azure Defender para DNS](defender-for-dns-introduction.md)|-|-|
|[Azure Defender para registros de contenedor](defender-for-container-registries-introduction.md)|✔ (2)|✔ (2)|
|[Azure Defender para Kubernetes](defender-for-kubernetes-introduction.md)|✔|✔|
|[Azure Defender para bases de datos relacionales de código abierto](defender-for-databases-introduction.md)|-|-|
|[Protección de cargas de trabajo de Kubernetes](kubernetes-workload-protections.md)|✔|✔|
|||

(1) Se requiere **Azure Defender para servidores**.

(2) Parcial.


## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo [Security Center recopila datos mediante el agente de Log Analytics](security-center-enable-data-collection.md).
- Obtenga información sobre cómo [Security Center administra y protege los datos](security-center-data-security.md).
- Revise las [plataformas compatibles con Security Center](security-center-os-coverage.md).