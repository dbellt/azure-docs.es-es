---
title: Introducción a los agentes de supervisión de Azure | Microsoft Docs
description: En este artículo se proporciona una descripción detallada de los agentes de Azure disponibles que permiten supervisar las máquinas virtuales hospedadas en Azure o el entorno híbrido.
services: azure-monitor
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 01/12/2021
ms.openlocfilehash: a7b2e6ee4b54427f33ba9701ae13f139341a1941
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112030960"
---
# <a name="overview-of-azure-monitor-agents"></a>Información general sobre los agentes de Azure Monitor

Las máquinas virtuales y otros recursos de proceso requieren un agente para recopilar datos de supervisión necesarios para medir el rendimiento y la disponibilidad de sus cargas de trabajo y sistema operativo invitado. En este artículo se describen los agentes utilizados por Azure Monitor y se ayuda a determinar qué se necesita para cumplir con los requisitos del entorno concreto.

> [!NOTE]
> Actualmente, Azure Monitor tiene varios agentes, debido a la reciente consolidación de Azure Monitor y Log Analytics. Aunque puede haber superposición en sus características, cada uno tiene funcionalidades únicas. En función de los requisitos, puede que necesite uno o varios de los agentes en las máquinas. 

Puede tener un conjunto específico de requisitos que no se puede cumplir por completo con un solo agente para una máquina concreta. Por ejemplo, puede que quiera usar alertas de métricas que requieran la extensión de Azure Diagnostics, pero que también quieran aprovechar la funcionalidad de VM Insights que requieren el agente de Log Analytics y Dependency Agent. En este tipo de casos, puede usar varios agentes; se trata de un escenario común en clientes que requieren funcionalidad de cada uno de ellos.

## <a name="summary-of-agents"></a>Resumen de los agentes

En las tablas siguientes se proporciona una comparación rápida de los agentes de Azure Monitor para Windows y Linux. En la sección siguiente se proporcionan más detalles sobre cada uno de ellos.

> [!NOTE]
> El agente de Azure Monitor se encuentra actualmente en versión preliminar con funcionalidades limitadas. Esta tabla se actualizará. 

### <a name="windows-agents"></a>Agentes de Windows

| | Agente de Azure Monitor (versión preliminar) | Diagnóstico<br>extensión (WAD) | Log Analytics<br>agente | Dependencia<br>agente |
|:---|:---|:---|:---|:---|
| **Entornos compatibles** | Azure<br>Otra nube (Azure Arc)<br>Local (Azure Arc)  | Azure | Azure<br>Otra nube<br>Local | Azure<br>Otra nube<br>Local | 
| **Requisitos del agente**  | None | None | None | Requiere el agente de Log Analytics |
| **Datos recopilados** | Registros de eventos<br>Rendimiento | Registros de eventos<br>Eventos de ETW<br>Rendimiento<br>Registros basados en archivos<br>Registros IIS<br>Registros de aplicaciones .NET<br>Volcados de memoria<br>Registros de diagnósticos del agente | Registros de eventos<br>Rendimiento<br>Registros basados en archivos<br>Registros IIS<br>Conclusiones y soluciones<br>Otros servicios | Dependencias de procesos<br>Métricas de conexión de red |
| **Destinatario de los datos** | Registros de Azure Monitor<br>Métricas de Azure Monitor | Azure Storage<br>Métricas de Azure Monitor<br>Centro de eventos | Registros de Azure Monitor | Registros de Azure Monitor<br>(a través del agente de Log Analytics) |
| **Servicios y**<br>**features**<br>**Se admite** | Log Analytics<br>Explorador de métricas | Explorador de métricas | VM Insights<br>Log Analytics<br>Azure Automation<br>Azure Security Center<br>Azure Sentinel | VM Insights<br>Mapa de servicio |

### <a name="linux-agents"></a>Agentes de Linux

| | Agente de Azure Monitor (versión preliminar) | Diagnóstico<br>extensión (WAD) | Telegraf<br>agente | Log Analytics<br>agente | Dependencia<br>agente |
|:---|:---|:---|:---|:---|:---|
| **Entornos admitidos** (consulte la tabla siguiente para los sistemas operativos compatibles) | Azure<br>Otra nube (Azure Arc)<br>Local (Azure Arc) | Azure | Azure<br>Otra nube<br>Local | Azure<br>Otra nube<br>Local | Azure<br>Otra nube<br>Local |
| **Requisitos del agente**  | None | None | None | None | Requiere el agente de Log Analytics |
| **Datos recopilados** | syslog<br>Rendimiento | syslog<br>Rendimiento | Rendimiento | syslog<br>Rendimiento| Dependencias de procesos<br>Métricas de conexión de red |
| **Destinatario de los datos** | Registros de Azure Monitor<br>Métricas de Azure Monitor | Azure Storage<br>Centro de eventos | Métricas de Azure Monitor | Registros de Azure Monitor | Registros de Azure Monitor<br>(a través del agente de Log Analytics) |
| **Servicios y**<br>**features**<br>**Se admite** | Log Analytics<br>Explorador de métricas | | Explorador de métricas | VM Insights<br>Log Analytics<br>Azure Automation<br>Azure Security Center<br>Azure Sentinel | VM Insights<br>Mapa de servicio |


## <a name="azure-monitor-agent-preview"></a>Agente de Azure Monitor (versión preliminar)

El [agente de Azure Monitor ](azure-monitor-agent-overview.md) se encuentra actualmente en versión preliminar y reemplazará al agente de Log Analytics y al agente de Telegraf para máquinas Windows y Linux. Puede enviar datos a los registros y métricas de Azure Monitor y usar las [reglas de recopilación de datos (DCR)](data-collection-rule-overview.md) que proporcionan un método más escalable para configurar la recopilación de datos y los destinos de cada agente.

Utilice el agente de Azure Monitor si necesita:

- Recopilar los registros y las métricas de los invitados de cualquier máquina en Azure, en otras nubes o en el entorno local. (Los [servidores habilitados para Azure Arc](../../azure-arc/servers/overview.md) son necesarios para las máquinas fuera de Azure). 
- Enviar datos a registros y métricas de Azure Monitor para su análisis con Azure Monitor. 
- Enviar datos a Azure Storage para su archivado.
- Enviar datos a herramientas de terceros mediante [Azure Event Hubs](./diagnostics-extension-stream-event-hubs.md).
- Administrar la seguridad de las máquinas mediante [Azure Security Center](../../security-center/security-center-introduction.md) o [Azure Sentinel](../../sentinel/overview.md). (No disponible en versión preliminar).

Las limitaciones del agente de Azure Monitor incluyen:

- Actualmente en versión preliminar pública. Consulte las [limitaciones actuales](azure-monitor-agent-overview.md#current-limitations) para obtener una lista de las limitaciones durante la versión preliminar pública.

## <a name="log-analytics-agent"></a>Agente de Log Analytics

El [agente de Log Analytics](./log-analytics-agent.md) recopila datos de supervisión del sistema operativo invitado y de las cargas de trabajo de las máquinas virtuales de Azure, otros proveedores de nube y las máquinas del entorno local. Envía datos a un área de trabajo de Log Analytics. El agente de Log Analytics es el mismo agente que se usa en System Center Operations Manager y en los equipos de agente de host múltiple para comunicarse con el grupo de administración y Azure Monitor simultáneamente. Este agente también es necesario en ciertas conclusiones de Azure Monitor y otros servicios de Azure.

> [!NOTE]
> El agente de Log Analytics para Windows a menudo se conoce como Microsoft Monitoring Agent (MMA). El agente de Log Analytics para Linux se conoce a menudo como agente de OMS.

Use el agente de Log Analytics si necesita:

* Recopilar registros y datos de rendimiento de las máquinas virtuales de Azure o de las máquinas híbridas hospedadas fuera de Azure.
* Enviar datos a un área de trabajo de Log Analytics para aprovechar las características compatibles con los [registros de Azure Monitor](../logs/data-platform-logs.md), como las [consultas de registro](../logs/log-query-overview.md).
* Use [VM Insights](../vm/vminsights-overview.md), que le permite supervisar las máquinas a escala, y supervisa sus procesos y dependencias en otros recursos y procesos externos.  
* Administrar la seguridad de las máquinas mediante [Azure Security Center](../../security-center/security-center-introduction.md) o [Azure Sentinel](../../sentinel/overview.md).
* Usar [Update Management de Azure Automation](../../automation/update-management/overview.md), [State Configuration de Azure Automation](../../automation/automation-dsc-overview.md) o [Seguimiento de cambios e inventario de Azure Automation](../../automation/change-tracking/overview.md) para ofrecer una administración completa de las VM de Azure y las que no son de Azure.
* Usar diferentes [soluciones](../monitor-reference.md#insights-and-core-solutions) para supervisar un servicio o una aplicación determinados.

Las limitaciones del agente de Log Analytics incluyen:

- No se pueden enviar datos a métricas de Azure Monitor, Azure Storage ni Azure Event Hubs.
- Dificultad para configurar definiciones de supervisión únicas para agentes individuales.
- Dificultad para administrar a escala, ya que cada máquina virtual tiene una configuración única.

## <a name="azure-diagnostics-extension"></a>Extensión de Diagnósticos de Azure

La [extensión Azure Diagnostics](./diagnostics-extension-overview.md) recopila datos de supervisión del sistema operativo invitado y de las cargas de trabajo de las máquinas virtuales de Azure y otros recursos de proceso. Principalmente, recopila datos en Azure Storage, pero también permite definir receptores de datos para enviar datos a otros destinos, como métricas de Azure Monitor y Azure Event Hubs.

Use la extensión Azure Diagnostics si necesita:

- Enviar datos a Azure Storage para archivarlos o analizarlos con herramientas como [Explorador de Azure Storage](../../vs-azure-tools-storage-manage-with-storage-explorer.md).
- Enviar datos a [métricas de Azure Monitor](../essentials/data-platform-metrics.md) para analizarlos con el [explorador de métricas](../essentials/metrics-getting-started.md) y para aprovechar las característica, como las [alertas de métricas](../alerts/alerts-metric-overview.md) casi en tiempo real y la [escalabilidad automática](../autoscale/autoscale-overview.md) (solo Windows).
- Enviar datos a herramientas de terceros mediante [Azure Event Hubs](./diagnostics-extension-stream-event-hubs.md).
- Recopilar [diagnósticos de arranque](/troubleshoot/azure/virtual-machines/boot-diagnostics) para investigar los problemas de arranque de VM.

Las limitaciones de la extensión Azure Diagnostics incluyen:

- Solo se puede utilizar con recursos de Azure.
- Tiene capacidad limitada para enviar datos a los registros de Azure Monitor.

## <a name="telegraf-agent"></a>Agente Telegraf

El [agente de InfluxData Telegraf](../essentials/collect-custom-metrics-linux-telegraf.md) se usa para recopilar datos de rendimiento de equipos Linux en métricas de Azure Monitor.

Use el agente Telegraf si necesita:

* Enviar datos a [métricas de Azure Monitor](../essentials/data-platform-metrics.md) para analizarlos con el [explorador de métricas](../essentials/metrics-getting-started.md) y para aprovechar las característica, como las [alertas de métricas](../alerts/alerts-metric-overview.md) casi en tiempo real y la [escalabilidad automática](../autoscale/autoscale-overview.md) (solo Linux).

## <a name="dependency-agent"></a>Dependency Agent

Dependency Agent recopila datos detectados acerca de los procesos que se ejecutan en las dependencias de las máquinas y los procesos externos. 

Use Dependency Agent si necesita:

* Usar la característica de asignación para [VM Insights](../vm/vminsights-overview.md) o la solución [Service Map](../vm/service-map.md).

Tenga en cuenta lo siguiente al usar Dependency Agent:

- Dependency Agent requiere que el agente de Log Analytics se instale en la misma máquina.
- En los equipos Linux, el agente de Log Analytics debe instalarse antes que la extensión Azure Diagnostics.

## <a name="virtual-machine-extensions"></a>Extensiones de máquina virtual

La extensión Log Analytics para [Windows](../../virtual-machines/extensions/oms-windows.md) y [Linux](../../virtual-machines/extensions/oms-linux.md) instala el agente de Log Analytics en máquinas virtuales de Azure. La extensión Dependency de Azure Monitor para [Windows](../../virtual-machines/extensions/agent-dependency-windows.md) y [Linux](../../virtual-machines/extensions/agent-dependency-linux.md) instala Dependency Agent en máquinas virtuales de Azure. Se trata de los mismos agentes descritos anteriormente, pero permiten administrarlos a través de [extensiones de máquinas virtuales](../../virtual-machines/extensions/overview.md). Debe usar extensiones para instalar y administrar los agentes siempre que sea posible.

En las máquinas híbridas, use [servidores habilitados para Azure Arc](../../azure-arc/servers/manage-vm-extensions.md) para implementar las extensiones de máquina virtual de Log Analytics y dependencias de Azure Monitor.

## <a name="supported-operating-systems"></a>Sistemas operativos admitidos

En las tablas siguientes se enumeran los sistemas operativos compatibles con los agente de Azure Monitor. Consulte la documentación de cada agente para conocer las consideraciones únicas y el proceso de instalación. Consulte la documentación de Telegraf para obtener información sobre los sistemas operativos admitidos. Se supone que todos los sistemas operativos son x64. No se admite x86 para ningún sistema operativo.

### <a name="windows"></a>Windows

| Sistema operativo | Agente de Azure Monitor | Agente de Log Analytics | Dependency Agent | Extensión Diagnostics | 
|:---|:---:|:---:|:---:|:---:|
| Windows Server 2019                                      | X | X | X | X |
| Windows Server 2019 Core                                 | X |   |   |   |
| Windows Server 2016                                      | X | X | X | X |
| Windows Server 2016 Core                                 | X |   |   | X |
| Windows Server 2012 R2                                   | X | X | X | X |
| Windows Server 2012                                      | X | X | X | X |
| Windows Server 2008 R2 SP1                               | X | X | X | X |
| Windows Server 2008 R2                                   |   | X | X | X |
| Windows 10 Enterprise<br>(incluida la sesión múltiple) y Pro<br>(Solo para escenarios de servidor)  | X | X | X | X |
| Windows 8 Enterprise y Pro<br>(Solo para escenarios de servidor)  |   | X | X |   |
| Windows 7 SP1<br>(Solo para escenarios de servidor)                 |   | X | X |   |

### <a name="linux"></a>Linux

| Sistema operativo | Agente de Azure Monitor <sup>1</sup> | Agente de Log Analytics <sup>1</sup> | Dependency Agent | Extensión Diagnostics <sup>2</sup>| 
|:---|:---:|:---:|:---:|:---:
| Amazon Linux 2017.09                                        |   | X |   |   |
| CentOS Linux 8                                              | X <sup>3</sup> | X | X |   |
| CentOS Linux 7                                              | X | X | X | X |
| CentOS Linux 6                                              |   | X |   |   |
| CentOS Linux 6.5+                                           |   | X | X | X |
| Debian 10 <sup>1</sup>                                      | X |   |   |   |
| Debian 9                                                    | X | X | x | X |
| Debian 8                                                    |   | X | X |   |
| Debian 7                                                    |   |   |   | X |
| OpenSUSE 13.1+                                              |   |   |   | X |
| Oracle Linux 8                                              | X <sup>3</sup> | X |   |   |
| Oracle Linux 7                                              | X | X |   | X |
| Oracle Linux 6                                              |   | X |   |   |
| Oracle Linux 6.4+                                           |   | X |   | X |
| Red Hat Enterprise Linux Server 8.3                         | X <sup>3</sup> | X | X |   |
| Red Hat Enterprise Linux Server 8                           | X <sup>3</sup> | X | X |   |
| Red Hat Enterprise Linux Server 7                           | X | X | X | X |
| Red Hat Enterprise Linux Server 6                           |   | X | X |   |
| Red Hat Enterprise Linux Server 6.7+                        |   | X | X | X |
| SUSE Linux Enterprise Server 15.2                           | X <sup>3</sup> |   |   |   |
| SUSE Linux Enterprise Server 15.1                           | X <sup>3</sup> | X |   |   |
| SUSE Linux Enterprise Server 15 SP1                         | X | X | X |   |
| SUSE Linux Enterprise Server 15                             | X | X | X |   |
| SUSE Linux Enterprise Server 12 SP5                         | X | X | X | X |
| SUSE Linux Enterprise Server 12                             | X | X | X | X |
| Ubuntu 20.04 LTS                                            | X | X | X |   |
| Ubuntu 18.04 LTS                                            | X | X | X | X |
| Ubuntu 16.04 LTS                                            | X | X | X | X |
| Ubuntu 14.04 LTS                                            |   | X |   | X |

<sup>1</sup> Requiere que Python (2 o 3) esté instalado en la máquina.

<sup>3</sup> Problema conocido al recopilar eventos de Syslog en versiones anteriores a la 1.9.0
#### <a name="dependency-agent-linux-kernel-support"></a>Compatibilidad del kernel de Linux con Dependency Agent

Dado que Dependency Agent funciona en el nivel de kernel, la compatibilidad también depende de la versión del kernel. En la tabla siguiente se enumeran la versión principal y secundaria de los sistemas operativos Linux y las versiones de kernel admitidas para Dependency Agent.

| Distribución | Versión del SO | Versión del kernel |
|:---|:---|:---|
|  Red Hat Linux 8   | 8,2     | 4.18.0-193.\*el8_2.x86_64 |
|                    | 8.1     | 4.18.0-147.\*el8_1.x86_64 |
|                    | 8.0     | 4.18.0-80.\*el8.x86_64<br>4.18.0-80.\*el8_0.x86_64 |
|  Red Hat Linux 7   | 7.9     | 3.10.0-1160 |
|                    | 7.8     | 3.10.0-1136 |
|                    | 7,7     | 3.10.0-1062 |
|                    | 7.6     | 3.10.0-957  |
|                    | 7.5     | 3.10.0-862  |
|                    | 7.4     | 3.10.0-693  |
| Red Hat Linux 6    | 6.10    | 2.6.32-754 |
|                    | 6.9     | 2.6.32-696  |
| CentOS Linux 8     | 8,2     | 4.18.0-193.\*el8_2.x86_64 |
|                    | 8.1     | 4.18.0-147.\*el8_1.x86_64 |
|                    | 8.0     | 4.18.0-80.\*el8.x86_64<br>4.18.0-80.\*el8_0.x86_64 |
| CentOS Linux 7     | 7.9     | 3.10.0-1160 |
|                    | 7.8     | 3.10.0-1136 |
|                    | 7,7     | 3.10.0-1062 |
| CentOS Linux 6     | 6.10    | 2.6.32-754.3.5<br>2.6.32-696.30.1 |
|                    | 6.9     | 2.6.32-696.30.1<br>2.6.32-696.18.7 |
| Ubuntu Server      | 20.04   | 5.4\* |
|                    | 18,04   | 5.3.0-1020<br>5.0 (incluye kernel optimizado para Azure)<br>4.18 *<br>4.15* |
|                    | 16.04.3 | 4.15.\* |
|                    | 16.04   | 4.13.\*<br>4.11.\*<br>4.10.\*<br>4.8.\*<br>4.4.\* |
| SUSE Linux 12 Enterprise Server | 15     | 4.12.14-150\*
|                                 | 12 SP4 | 4.12.* (incluye kernel optimizado para Azure) |
|                                 | 12 SP3 | 4.4.* |
|                                 | 12 SP2 | 4.4.* |
| Debian                          | 9      | 4,9  | 

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cada uno de los agentes en los siguientes artículos:

- [Introducción al agente de Log Analytics](./log-analytics-agent.md)
- [Introducción a la extensión Azure Diagnostics](./diagnostics-extension-overview.md)
- [Recopilación de métricas personalizadas para una máquina virtual Linux con el agente de InfluxData Telegraf](../essentials/collect-custom-metrics-linux-telegraf.md)
