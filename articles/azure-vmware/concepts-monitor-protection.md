---
title: 'Conceptos: supervisión y protección'
description: Obtenga información sobre los servicios nativos de Azure que ayudan a asegurar y proteger sus cargas de trabajo de Azure VMware Solution.
ms.topic: conceptual
ms.date: 06/14/2021
ms.openlocfilehash: d735e705ffbccaa0e5fb38951c09b01a22818a0c
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111758324"
---
# <a name="monitor-and-protect-azure-vmware-solution-workloads"></a>Supervisión y protección de cargas de trabajo de Azure VMware Solution

Los servicios nativos de Microsoft Azure permiten supervisar, administrar y proteger máquinas virtuales (VM) en Azure VMware Solution y máquinas virtuales locales. Los servicios nativos de Azure que puede integrar con Azure VMware Solution incluyen:

- **Un área de trabajo de Log Analytics** es un entorno único para almacenar los datos de registro. Cada área de trabajo tiene su propio repositorio y configuración de datos. Los orígenes de datos y las soluciones se configuran para almacenar sus datos en un área de trabajo específica. Implemente con facilidad el agente de Log Analytics mediante la compatibilidad con la extensión de máquina virtual de servidores habilitados para Azure Arc para máquinas virtuales nuevas y existentes. 
- **Azure Security Center** es un sistema de administración de la seguridad de infraestructura unificada. Fortalece la seguridad de los centros de datos y proporciona protección contra amenazas avanzada en las cargas de trabajo híbridas tanto en la nube como a nivel local. Evalúa la vulnerabilidad de las máquinas virtuales de Azure VMware Solution y genera alertas según sea necesario. Estas alertas de seguridad se pueden reenviar a Azure Monitor para su resolución. Puede definir las directivas de seguridad en Azure Security Center. Para más información, consulte [Integración de Azure Security Center con Azure VMware Solution](azure-security-integration.md).
- **[Azure Monitor](../azure-monitor/vm/vminsights-enable-overview.md)** es una solución completa para recopilar, analizar y actuar en la telemetría desde los entornos local y en la nube. No requiere ninguna implementación. Con Azure Monitor, puede supervisar el rendimiento del sistema operativo invitado y descubrir y asignar dependencias de aplicaciones para máquinas virtuales de Azure VMware Solution o locales. El área de trabajo de Log Analytics en Azure Monitor habilita la recopilación de registros y contadores de rendimiento mediante el agente de Log Analytics o extensiones. Recopile datos y registros en un único punto y presente dichos datos en diferentes servicios nativos de Azure.
- **Azure Arc** lleva la administración de Azure a cualquier infraestructura, incluidos Azure VMware Solution, entornos locales u otras plataformas en la nube. Los [servidores habilitados para Azure Arc](../azure-arc/servers/overview.md) permiten administrar servidores físicos y máquinas virtuales Windows y Linux hospedados *fuera* de Azure, en la red corporativa o en otro proveedor de nube. Puede adjuntar un clúster de Kubernetes hospedado en el entorno de Azure VMware Solution mediante [Kubernetes habilitado para Azure Arc](../azure-arc/kubernetes/overview.md). 
- **[Azure Update Management](../automation/update-management/overview.md)** en Azure Automation administra las actualizaciones del sistema operativo de las máquinas Windows y Linux en un entorno híbrido. Supervisa el cumplimiento de la aplicación de revisiones y reenvía alertas de desviación de la aplicación de revisiones a Azure Monitor para su corrección. Azure Update Management debe conectarse al área de trabajo de Log Analytics para usar datos almacenados con el fin de evaluar el estado de las actualizaciones en las máquinas virtuales. 
 


## <a name="topology"></a>Topología

En el diagrama se muestra la arquitectura de supervisión integrada de las máquinas virtuales de Azure VMware Solution.

:::image type="content" source="media/lifecycle-management-azure-vmware-solutions-virtual-machines/integrated-azure-monitoring-architecture.png" alt-text="Diagrama que muestra la arquitectura de supervisión integrada de Azure." border="false":::

El agente de Log Analytics habilita la recopilación de datos de registro de Azure, Azure VMware Solution y las máquinas virtuales locales. Los datos de registro se envían a los registros de Azure Monitor y se almacenan en un área de trabajo de Log Analytics. Puede implementar el agente de Log Analytics mediante la [compatibilidad con la extensión de máquina virtual](../azure-arc/servers/manage-vm-extensions.md) de servidores habilitados para Arc para máquinas virtuales nuevas y existentes. 

Una vez que los registros se recopilan en el área de trabajo de Log Analytics, puede configurar el área de trabajo de Log Analytics con Azure Security Center. Azure Security Center evalúa el estado de vulnerabilidades de las máquinas virtuales de Azure VMware Solution y generará una alerta para cualquier vulnerabilidad crítica. Por ejemplo, se evalúan las revisiones del sistema operativo que faltan, los errores de configuración de seguridad y [Endpoint Protection](../security-center/security-center-services.md).

Puede configurar el área de trabajo de Log Analytics con Azure Sentinel para detección de alertas, visibilidad de amenazas, búsqueda y respuesta a amenazas. En el diagrama anterior, Azure Security Center se conecta a Azure Sentinel mediante el conector de Azure Security Center. Azure Security Center reenviará la vulnerabilidad del entorno a Azure Sentinel para crear un incidente y asignarlo a otras amenazas. También puede crear la consulta de reglas programadas para detectar la actividad no deseada y convertirla en incidentes.


## <a name="next-steps"></a>Pasos siguientes

Ahora que ha tratado estos conceptos de red e interconectividad de Azure VMware Solution, puede que quiera obtener información sobre:

- [Integración de servicios nativos de Azure en Azure VMware Solution](integrate-azure-native-services.md)
- [Integración de Azure Security Center con Azure VMware Solution](azure-security-integration.md)
- [Autenticación de cuentas de Automation](../automation/automation-security-overview.md)
- [Diseño de la implementación de registros de Azure Monitor](../azure-monitor/logs/design-logs-deployment.md) y [Azure Monitor](../azure-monitor/overview.md)
- [Planeamiento de Azure Security Center](../security-center/security-center-planning-and-operations-guide.md) y [Plataformas compatibles con Security Center](../security-center/security-center-os-coverage.md)


