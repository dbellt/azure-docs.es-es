---
title: Supervisión y administración de máquinas virtuales de Azure VMware Solution
description: Aprenda a administrar todos los aspectos del ciclo de vida de las máquinas virtuales de Azure VMware Solution con herramientas nativas de Microsoft Azure.
ms.topic: how-to
ms.date: 05/04/2021
ms.openlocfilehash: 3cffba1147a52a48d5005aa22e3a08a33956d66e
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108769262"
---
# <a name="monitor-and-manage-azure-vmware-solution-vms"></a>Supervisión y administración de máquinas virtuales de Azure VMware Solution



Las herramientas nativas de Microsoft Azure le permiten supervisar y administrar las máquinas virtuales (VM) en el entorno de Azure. Sin embargo, también le permiten supervisar y administrar las máquinas virtuales en Azure VMware Solution y las máquinas virtuales locales. En este artículo, veremos la arquitectura de supervisión integrada que ofrece Azure y cómo puede usar sus herramientas nativas para administrar las máquinas virtuales de Azure VMware Solution a lo largo de su ciclo de vida.

## <a name="benefits"></a>Ventajas

- Los servicios nativos de Azure se pueden usar para administrar las máquinas virtuales en un entorno híbrido (Azure, Azure VMware Solution y local).
- Supervisión y visibilidad integradas de las máquinas virtuales de Azure, Azure VMware Solution y locales.
- Con Azure Update Management en Azure Automation, puede administrar las actualizaciones del sistema operativo de las máquinas Windows y Linux. 
- Azure Security Center proporciona Advanced Threat Protection, incluyendo lo siguiente:
    - Supervisión de la integridad de los archivos
    - Alertas de seguridad sin archivos
    - Evaluación de revisiones del sistema operativo
    - Evaluación de configuraciones de seguridad incorrectas
    - Evaluación de EndPoint Protection 
- Implemente con facilidad el agente de Log Analytics mediante la compatibilidad con la extensión de máquina virtual de servidores habilitados para Azure Arc para máquinas virtuales nuevas y existentes. 
- El área de trabajo de Log Analytics en Azure Monitor habilita la recopilación de registros y contadores de rendimiento mediante el agente de Log Analytics o extensiones. Recopile datos y registros en un único punto y presente dichos datos en diferentes servicios nativos de Azure. 
- Entre las ventajas agregadas de Azure Monitor se incluyen: 
    - Supervisión fluida 
    - Mejor visibilidad de la infraestructura 
    - Notificaciones instantáneas 
    - Resolución automática 
    - Rentabilidad 

## <a name="integrated-azure-monitoring-architecture"></a>Arquitectura de supervisión de Azure integrada

En el diagrama se muestra la arquitectura de supervisión integrada de las máquinas virtuales de Azure VMware Solution.

![Arquitectura de supervisión de Azure integrada](media/lifecycle-management-azure-vmware-solutions-virtual-machines/integrated-azure-monitoring-architecture.png)

## <a name="before-you-start"></a>Antes de comenzar

Si no está familiarizado con Azure o con ninguno de los servicios mencionados anteriormente, consulte los artículos siguientes:

- [Introducción a la autenticación de cuentas de Automation](../automation/automation-security-overview.md)
- [Diseño de la implementación de registros de Azure Monitor](../azure-monitor/logs/design-logs-deployment.md) y [Azure Monitor](../azure-monitor/overview.md)
- [Planeamiento](../security-center/security-center-planning-and-operations-guide.md) y [Plataformas compatibles](../security-center/security-center-os-coverage.md) para Azure Security Center
- [Información general sobre la habilitación de Azure Monitor para VM](../azure-monitor/vm/vminsights-enable-overview.md)
- [¿Qué son los servidores habilitados para Azure Arc?](../azure-arc/servers/overview.md) y [¿Qué es Kubernetes habilitado para Azure Arc?](../azure-arc/kubernetes/overview.md)
- [Introducción a Update Management](../automation/update-management/overview.md)

## <a name="integrate-and-deploy-azure-native-services"></a>Integración e implementación de servicios nativos de Azure

### <a name="enable-azure-update-management"></a>Habilitación de Azure Update Management

Azure Update Management en Azure Automation administra las actualizaciones del sistema operativo de las máquinas Windows y Linux en un entorno híbrido. Supervisa el cumplimiento de la aplicación de revisiones y reenvía alertas de desviación de la aplicación de revisiones a Azure Monitor para su corrección. Azure Update Management debe conectarse al área de trabajo de Log Analytics para usar datos almacenados con el fin de evaluar el estado de las actualizaciones en las máquinas virtuales.

1.  Antes de poder agregar Log Analytics a Azure Update Management, primero debe [crear una cuenta de Azure Automation](../automation/automation-create-standalone-account.md). Si prefiere crear su cuenta mediante una plantilla, consulte [Creación de una cuenta de Automation con una plantilla de Azure Resource Manager](../automation/quickstart-create-automation-account-template.md).

2. El **área de trabajo de Log Analytics** habilita la recopilación de registros y contadores de rendimiento mediante el agente de Log Analytics o extensiones. Para crear el área de trabajo de Log Analytics, consulte [Creación de un área de trabajo de Log Analytics en Azure Portal](../azure-monitor/logs/quick-create-workspace.md). Si lo prefiere, también puede crear un área de trabajo a través de la [CLI](../azure-monitor/logs/quick-create-workspace-cli.md), [PowerShell](../azure-monitor/logs/powershell-workspace-configuration.md) o una [plantilla de Azure Resource Manager](../azure-monitor/logs/resource-manager-workspace.md).

3. Para habilitar Azure Update Management para las máquinas virtuales, consulte [Habilitación de Update Management desde una cuenta de Automation](../automation/update-management/enable-from-automation-account.md). En el proceso, vinculará el área de trabajo de Log Analytics con la cuenta de Automation. 
 
4. Una vez que haya agregado máquinas virtuales a Azure Update Management, puede [implementar actualizaciones en máquinas virtuales y revisar los resultados](../automation/update-management/deploy-updates.md). 

### <a name="enable-azure-security-center"></a>Habilitación de Azure Security Center

Azure Security Center proporciona Advanced Threat Protection en todas las cargas de trabajo híbridas de la nube y del entorno local. Evaluará la vulnerabilidad de las máquinas virtuales de Azure VMware Solution y generará alertas según sea necesario. Estas alertas de seguridad se pueden reenviar a Azure Monitor para su resolución.

Azure Security Center no requiere implementación. Para obtener más información, consulte una lista de [características admitidas para máquinas virtuales](../security-center/security-center-services.md).

1. Para agregar máquinas virtuales de Azure VMware Solution y otras que no sean de Azure a Security Center, vea [Inicio rápido: Configuración de Azure Security Center](../security-center/security-center-get-started.md). 

2. Después de agregar máquinas virtuales de Azure VMware Solution o de un entorno que no sea de Azure, habilite Azure Defender en Security Center. Security Center evaluará las máquinas virtuales en busca de posibles problemas de seguridad. También proporciona recomendaciones en la pestaña Información general. Para más información, consulte [Recomendaciones de seguridad de Azure Security Center](../security-center/security-center-recommendations.md).

3. Puede definir las directivas de seguridad en Azure Security Center. Para obtener información sobre cómo configurar las directivas de seguridad, consulte [Uso de directivas de seguridad](../security-center/tutorial-security-policy.md).

### <a name="onboard-vms-to-azure-arc-enabled-servers"></a>Incorporación de máquinas virtuales a servidores habilitados para Azure Arc

Azure Arc amplía la administración de Azure a cualquier infraestructura, incluidos Azure VMware Solution, el entorno local u otras plataformas en la nube.

- Para información sobre la habilitación de servidores habilitados para Azure Arc para varias máquinas virtuales Windows o Linux, vea [Conexión de máquinas híbridas a Azure a gran escala](../azure-arc/servers/onboard-service-principal.md).

### <a name="onboard-hybrid-kubernetes-clusters-with-arc-enabled-kubernetes"></a>Incorporación de clústeres híbridos de Kubernetes con Kubernetes habilitado para Azure Arc

Puede adjuntar un clúster de Kubernetes hospedado en el entorno de Azure VMware Solution mediante Kubernetes habilitado para Azure Arc. 

- Para más información, consulte [Creación de una entidad de servicio de incorporación habilitada para Azure Arc](../azure-arc/kubernetes/create-onboarding-service-principal.md).

### <a name="deploy-the-log-analytics-agent"></a>Implementación del agente de Log Analytics

Las máquinas virtuales de Azure VMware Solution se pueden supervisar a través del agente de Log Analytics (también conocido como Microsoft Monitoring Agent (MMA) o agente de Linux de OMS). Ya ha creado un área de trabajo Log Analytics al habilitar Update Management de Azure Automation.

- Implemente el agente de Log Analytics con [compatibilidad con la extensión de máquina virtual de servidores habilitados para Azure Arc](../azure-arc/servers/manage-vm-extensions.md).

### <a name="enable-azure-monitor"></a>Habilitación de Azure Monitor

Azure Monitor es una solución completa para recopilar, analizar y actuar en la telemetría desde los entornos local y en la nube. No requiere ninguna implementación. Con Azure Monitor, puede supervisar el rendimiento del sistema operativo invitado y descubrir y asignar dependencias de aplicaciones para máquinas virtuales de Azure VMware Solution o locales.

- Azure Monitor le permite recopilar datos de diferentes orígenes para su supervisión y análisis. Para obtener más información, consulte [Orígenes de datos de supervisión para Azure Monitor](../azure-monitor/agents/data-sources.md).

- Recopile otros tipos de datos para su análisis, visualización y alertas. Para más información, consulte [Plataforma de datos de Azure Monitor](../azure-monitor/data-platform.md).

- Para configurar Azure Monitor con el área de trabajo de Log Analytics, consulte [Configuración del área de trabajo de Log Analytics para Azure Monitor para VM](../azure-monitor/vm/vminsights-configure-workspace.md).

- Puede crear reglas de alerta para identificar problemas en su entorno, como, por ejemplo, uso elevado de recursos, revisiones que faltan, espacio en disco bajo y latido de las máquinas virtuales. También puede establecer una respuesta automatizada a eventos detectados enviando una alerta a las herramientas de Administración de servicios de TI (ITSM). La notificación de detección de alertas también se puede enviar por correo electrónico. Para crear dichas reglas, consulte:
    - [Creación, visualización y administración de alertas de métricas mediante Azure Monitor](../azure-monitor/alerts/alerts-metric.md).
    - [Creación, visualización y administración de alertas de registro mediante Azure Monitor](../azure-monitor/alerts/alerts-log.md).
    - [Reglas de acción](../azure-monitor/alerts/alerts-action-rules.md) para establecer acciones y notificaciones automatizadas.
    - [Conexión de Azure a las herramientas de ITSM mediante el Conector de Administración de servicios de TI](../azure-monitor/alerts/itsmc-overview.md).
