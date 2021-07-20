---
title: Características de Azure Security Center según el sistema operativo, el tipo de máquina y la nube
description: Obtenga información acerca de las características de Azure Security Center que están disponibles según el sistema operativo, el tipo de máquina y la implementación en la nube.
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: overview
ms.date: 06/17/2021
ms.custom: references_regions
ms.author: memildin
ms.openlocfilehash: 05b535a0aabe0108c0a9abcedef166293541d407
ms.sourcegitcommit: a038863c0a99dfda16133bcb08b172b6b4c86db8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/29/2021
ms.locfileid: "113005481"
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



## <a name="feature-support-in-government-and-sovereign-clouds"></a>Compatibilidad de características en las nubes gubernamentales y soberanas

| Característica o servicio                                                                                                                                                             | Azure          | US Gov                  | Azure China   |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------|--------------------------------|---------------|
| **Características gratuitas de Security Center**                                                                                                                                           |                |                                |               |
| - [Exportación continua](/azure/security-center/continuous-export)                                                                                                             | GA             | GA                             | GA            |
| - [Automatización de flujos de trabajo](/azure/security-center/continuous-export)                                                                                                           | GA             | GA                             | GA            |
| - [Reglas de exención de recomendaciones](/azure/security-center/exempt-resource)                                                                                                  | Vista previa pública | No disponible                  | No disponible |
| - [Reglas de eliminación de alertas](/azure/security-center/alerts-suppression-rules)                                                                                                | GA             | GA                             | GA            |
| - [Notificaciones de correo electrónico para alertas de seguridad](/azure/security-center/security-center-provide-security-contact-details)                                                        | GA             | GA                             | GA            |
| - [Aprovisionamiento automático de agentes y extensiones](/azure/security-center/security-center-enable-data-collection)                                                              | GA             | GA                             | GA            |
| - [Inventario de recursos](/azure/security-center/asset-inventory)                                                                                                                 | GA             | GA                             | GA            |
| - [Informes de libros de Azure Monitor en la galería de libros de Azure Security Center](/azure/security-center/custom-dashboards-azure-workbooks)                                  | GA             | GA                             | GA            |
| **Planes y extensiones de Azure Defender**                                                                                                                                     |                |                                |               |
| - [Azure Defender para servidores](/azure/security-center/defender-for-servers-introduction)                                                                                    | GA             | GA                             | GA            |
| - [Azure Defender para App Service](/azure/security-center/defender-for-app-service-introduction)                                                                            | GA             | No disponible                  | No disponible |
| - [Azure Defender para DNS](/azure/security-center/defender-for-dns-introduction)                                                                                            | GA             | No disponible                  | No disponible |
| - [Azure Defender para registros de contenedor](/azure/security-center/defender-for-container-registries-introduction) <sup>[1](#footnote1)</sup>                               | Disponibilidad general             | GA  <sup>[2](#footnote2)</sup> | GA  <sup>[2](#footnote2)</sup> |
| - [Azure Defender para registros de contenedor que examinan imágenes en flujos de trabajo de CI/CD](/azure/security-center/defender-for-container-registries-cicd) <sup>[3](#footnote3)</sup> | Vista previa pública | No disponible                  | No disponible |
| - [Azure Defender para Kubernetes](/azure/security-center/defender-for-kubernetes-introduction) <sup>[4](#footnote4)</sup>                                                   | GA             | GA                             | GA            |
| - [Extensión de Azure Defender para clústeres de Kubernetes habilitados para Azure Arc](/azure/security-center/defender-for-kubernetes-azure-arc) <sup>[5](#footnote5)</sup>                 | Vista previa pública | No disponible                  | No disponible |
| - [Azure Defender para servidores de Azure SQL Database](/azure/security-center/defender-for-sql-introduction)                                                                     | GA             | GA                             | GA  <sup>[9](#footnote9)</sup> |
| - [Azure Defender para servidores SQL en máquinas](/azure/security-center/defender-for-sql-introduction)                                                                        | GA             | GA                             | No disponible |
| - [Azure Defender para bases de datos relacionales de código abierto](/azure/security-center/defender-for-databases-introduction)                                                         | GA             | No disponible                  | No disponible |
| - [Azure Defender para Key Vault](/azure/security-center/defender-for-key-vault-introduction)                                                                                | GA             | No disponible                  | No disponible |
| - [Azure Defender para Resource Manager](/azure/security-center/defender-for-resource-manager-introduction)                                                                  | Disponibilidad general             | Vista previa pública                 | No disponible |
| - [Azure Defender para Storage](/azure/security-center/defender-for-storage-introduction) <sup>[6](#footnote6)</sup>                                                         | GA             | GA                             | No disponible |
| - [Protección contra amenazas para Cosmos DB](/azure/security-center/other-threat-protections.md#threat-protection-for-azure-cosmos-db-preview)                                       | Vista previa pública | No disponible                  | No disponible |
| - [Protección de cargas de trabajo de Kubernetes](/azure/security-center/kubernetes-workload-protections)                                                                                  | GA             | GA                             | GA            |
| **Azure Defender para características de servidores** <sup>[7](#footnote7)</sup>                                                                                                          |                |                                |               |
| - [Acceso de máquina virtual Just-In-Time](/azure/security-center/security-center-just-in-time)                                                                                             | GA             | GA                             | GA            |
| - [Supervisión de la integridad de los archivos](/azure/security-center/security-center-file-integrity-monitoring)                                                                             | GA             | GA                             | GA            |
| - [Controles de aplicaciones adaptables](/azure/security-center/security-center-adaptive-application)                                                                              | GA             | GA                             | GA            |
| - [Protección de red adaptable](/azure/security-center/security-center-adaptive-network-hardening)                                                                           | GA             | No disponible                  | No disponible |
| - [Protección de hosts de Docker](/azure/security-center/harden-docker-hosts)                                                                                                       | GA             | GA                             | GA            |
| - [Evaluación integrada de vulnerabilidades para las máquinas](/azure/security-center/deploy-vulnerability-assessment-vm)                                                             | GA             | No disponible                  | No disponible |
| - [Panel e informes de cumplimiento normativo](/azure/security-center/security-center-compliance-dashboard) <sup>[8](#footnote8)</sup>                                       | GA             | GA                             | GA            |
| - [Implementación de Microsoft Defender para punto de conexión y licencia integrada](/azure/security-center/security-center-wdatp)                                                         | GA             | GA                             | No disponible |
| - [Conexión de cuentas de AWS](/azure/security-center/quickstart-onboard-aws)                                                                                                      | GA             | No disponible                  | No disponible |
| - [Conexión de cuentas de GCP](/azure/security-center/quickstart-onboard-gcp)                                                                                                      | GA             | No disponible                  | No disponible |
|                                                                                                                                                                             |                |                                |

<sup> <a name="footnote1" /> </a> 1</sup> Parcialmente en disponibilidad general: la capacidad de deshabilitar los resultados específicos de los exámenes de vulnerabilidades está en versión preliminar pública.

<sup><a name="footnote2" /></a>2</sup> Los exámenes de vulnerabilidades de los registros de contenedor en Azure Gov solo se pueden realizar con la característica de examen en inserción.

<sup><a name="footnote3" /></a>3</sup> Requiere Azure Defender para registros de contenedor.

<sup><a name="footnote4" /></a>4</sup> Parcialmente en disponibilidad general: la compatibilidad con clústeres habilitados para Arc está en versión preliminar pública y no está disponible en Azure Government.

<sup><a name="footnote5" /></a>5</sup> Requiere Azure Defender para Kubernetes.

<sup><a name="footnote6" /></a> 6</sup> Parcialmente en disponibilidad general: algunas de las alertas de protección contra amenazas de Azure Defender para Storage están en versión preliminar pública.

<sup><a name="footnote7" /></a>7</sup> Todas estas características requieren [Azure Defender para servidores](/azure/security-center/defender-for-servers-introduction).

<sup><a name="footnote8" /></a>8</sup> Puede haber diferencias en los estándares ofrecidos por cada tipo de nube.
 
<sup><a name="footnote9" /></a>9</sup> Parcialmente en disponibilidad general: subconjunto de alertas y evaluación de vulnerabilidades en servidores SQL. No están disponibles las protecciones contra amenazas de comportamiento.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo [Security Center recopila datos mediante el agente de Log Analytics](security-center-enable-data-collection.md).
- Obtenga información sobre cómo [Security Center administra y protege los datos](security-center-data-security.md).
- Revise las [plataformas compatibles con Security Center](security-center-os-coverage.md).