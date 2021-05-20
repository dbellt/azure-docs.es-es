---
title: Información general del agente de Azure Monitor
description: Información general del agente de Azure Monitor (AMA), que recopila datos de supervisión del sistema operativo invitado de máquinas virtuales.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/16/2021
ms.custom: references_regions
ms.openlocfilehash: 248f070c37e32cb0d90c3e31eebddc6245446828
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108765672"
---
# <a name="azure-monitor-agent-overview-preview"></a>Información general del agente de Azure Monitor (versión preliminar)
El agente de Azure Monitor (AMA) recopila datos de supervisión del sistema operativo invitado de máquinas virtuales de Azure y los entrega a Azure Monitor. En este artículo se proporciona información general sobre el agente de Azure Monitor, incluido cómo instalarlo y cómo configurar la recopilación de datos.

## <a name="relationship-to-other-agents"></a>Relación con otros agentes
El agente de Azure Monitor reemplaza a los siguientes agentes utilizados actualmente por Azure Monitor para recopilar datos de invitado de máquinas virtuales:

- [Agente de Log Analytics](./log-analytics-agent.md): envía datos al área de trabajo de Log Analytics y admite VM Insights y soluciones de supervisión.
- [Extensión de Diagnostics](./diagnostics-extension-overview.md): Envía datos a Azure Storage, a las métricas de Azure Monitor (solo en Windows), a Event Hubs y a Azure Storage.
- [Agente de Telegraf](../essentials/collect-custom-metrics-linux-telegraf.md): Envía datos a las métricas de Azure Monitor (solo Linux).

Además de consolidar esta funcionalidad en un solo agente, el agente de Azure Monitor proporciona las siguientes ventajas frente a los agentes existentes:

- Ámbito de supervisión. Configure de forma centralizada la recopilación para diferentes conjuntos de datos de diferentes conjuntos de VM.  
- Hospedaje múltiple en Linux: Envíe datos de VM Linux a varias áreas de trabajo.
- Filtrado de eventos de Windows: Use consultas de XPATCH para filtrar los eventos de Windows que se recopilan.
- Administración mejorada de las extensiones: El agente de Azure Monitor utiliza un nuevo método de control de la extensibilidad que es más transparente y controlable que los módulos de administración y los complementos de Linux en los agentes de Log Analytics actuales.

### <a name="changes-in-data-collection"></a>Cambios en la recopilación de datos
Los métodos para definir la recopilación de datos de los agentes existentes son distintos entre sí, y cada uno de ellos tiene desafíos que se abordan con el agente de Azure Monitor.

- El agente de Log Analytics obtiene su configuración de un área de trabajo de Log Analytics. Esto es fácil de configurar de forma centralizada, pero es difícil establecer definiciones independientes para diferentes máquinas virtuales. Solo puede enviar datos a un área de trabajo de Log Analytics.

- La extensión de Diagnostics tiene una configuración para cada máquina virtual. Resulta fácil establecer definiciones independientes para diferentes máquinas virtuales, pero difícil de administrar de forma centralizada. Solo puede enviar datos a métricas de Azure Monitor, Azure Event Hubs o Azure Storage. En el caso de los agentes de Linux, se necesita el agente de Telegraf de código abierto para enviar datos a las métricas de Azure Monitor.

El agente de Azure Monitor utiliza las [reglas de recopilación de datos (DCR)](data-collection-rule-overview.md) para configurar los datos que se van a recopilar de cada agente. Las reglas de recopilación de datos permiten la capacidad de administración de la configuración de la recopilación a escala y, al mismo tiempo, habilitan configuraciones únicas con ámbito para subconjuntos de máquinas. Son independientes del área de trabajo e independientes de la máquina virtual, lo que permite definirlas una vez y reutilizarlas en distintas máquinas y entornos. Consulte [Configuración de la recopilación de datos para el agente de Azure Monitor (versión preliminar)](data-collection-rule-azure-monitor-agent.md).

## <a name="should-i-switch-to-azure-monitor-agent"></a>¿Debo cambiar al agente de Azure Monitor agente?
El agente de Azure Monitor coexiste con los [agentes disponibles con carácter general para Azure Monitor ](agents-overview.md), pero puede considerar la posibilidad de realizar la transición de las máquinas virtuales fuera de los agentes actuales durante el período de versión preliminar pública del agente de Azure Monitor. Tenga en cuenta los siguientes factores al realizar esta determinación.

- **Requisitos de entorno.** El agente de Azure Monitor tiene un conjunto más limitado de requisitos de red, entornos y sistemas operativos compatibles que los agentes actuales. Lo más probable es que la compatibilidad con el entorno futuro, como las nuevas versiones de sistema operativo y los tipos de requisitos de red, se proporcione solo en el agente de Azure Monitor. Debe evaluar si el entorno el agente de Azure Monitor admite el entorno. Si no lo admite, tendrá que permanecer con el agente actual. Si el agente de Azure Monitor admite el entorno actual, debería considerar la posibilidad de realizar la transición a él.
- **Tolerancia de riesgos de la versión preliminar pública.** Mientras el agente de Azure Monitor se ha probado exhaustivamente para los escenarios admitidos actualmente, el agente todavía se encuentra en versión preliminar pública. Las actualizaciones de versión y las mejoras de funcionalidad se producirán con frecuencia y pueden presentar errores. Debe evaluar el riesgo de un error en el agente en las máquinas virtuales que podría detener la recopilación de datos. Si un intervalo de recopilación de datos no va a tener un impacto significativo en los servicios, entonces continúe con el agente de Azure Monitor. Si tiene una tolerancia baja para cualquier inestabilidad, debe permanecer con los agentes disponibles con carácter general hasta que el agente de Azure Monitor alcance este estado.
- **Requisitos de características actuales y nuevas.** El agente de Azure Monitor presenta varias funcionalidades nuevas, como el filtrado, el ámbito y el hospedaje múltiple, pero aún no está a la par con los agentes actuales para otras funcionalidades, como la recopilación de registros personalizados y la integración con las soluciones. La mayoría de las nuevas funcionalidades de Azure Monitor solo estarán disponibles con el agente de Azure Monitor, por lo que, con el tiempo, las funcionalidades adicionales solo estarán disponibles en el nuevo agente. Debe considerar si el agente de Azure Monitor tiene las características que necesita y si hay algunas características de las que puede prescindir temporalmente para obtener otras características importantes en el nuevo agente. Si el agente de Azure Monitor tiene todas las funcionalidades principales que necesita, considere la posibilidad de realizar la transición a él. Si hay características críticas que necesita, continúe con el agente actual hasta que el agente de Azure Monitor alcance la paridad.
- **Tolerancia al reprocesamiento.** Si está configurando un nuevo entorno con recursos como scripts de implementación y plantillas de incorporación, debe considerar si podrá reprocesarlos cuando el agente de Azure Monitor esté disponible con carácter general. Si el esfuerzo de este reprocesamiento es mínimo, siga con los agentes actuales por ahora. Si va a implicar una cantidad significativa de trabajo, considere la posibilidad de configurar el nuevo entorno con el nuevo agente. Se espera que el agente de Azure Monitor esté disponible con carácter general y una fecha de desuso publicada para los agentes de Log Analytics en 2021. Los agentes actuales se admitirán durante varios años una vez que se inicie el desuso.



## <a name="current-limitations"></a>Limitaciones actuales
Durante la versión preliminar pública del agente de Azure Monitor se aplican las siguientes limitaciones:

- El agente de Azure Monitor no admite soluciones y conclusiones como VM Insights y Azure Security Center. El único escenario que se admite actualmente es la recopilación de datos con las reglas de recopilación de datos que usted configure. 
- Las reglas de recopilación de datos se deben crear en la misma región que cualquier área de trabajo de Log Analytics que se use como destino.
- Actualmente se admiten máquinas virtuales de Azure, conjuntos de escalado de máquinas virtuales y servidores habilitados para Azure Arc. Azure Kubernetes Service y otros tipos de recursos de proceso no se admiten en este momento.
- La máquina virtual debe tener acceso a los siguientes puntos de conexión HTTPS:
  - *.ods.opinsights.azure.com
  - *.ingest.monitor.azure.com
  - *.control.monitor.azure.com


## <a name="supported-regions"></a>Regiones admitidas
El agente de Azure Monitor actualmente admite recursos en las siguientes regiones:

- Este de Asia
- Sudeste de Asia
- Centro de Australia
- Este de Australia
- Sudeste de Australia
- Centro de Canadá
- Norte de Europa
- Oeste de Europa
- Centro de Francia
- Centro-oeste de Alemania
- Centro de la India
- Japón Oriental
- Centro de Corea del Sur
- Norte de Sudáfrica
- Norte de Suiza
- Sur de Reino Unido
- Oeste de Reino Unido
- Centro de EE. UU.
- Este de EE. UU.
- Este de EE. UU. 2
- Centro-Norte de EE. UU
- Centro-sur de EE. UU.
- Oeste de EE. UU.
- Oeste de EE. UU. 2
- Centro-Oeste de EE. UU.

## <a name="coexistence-with-other-agents"></a>Coexistencia con otros agentes
El agente de Azure Monitor puede coexistir con los agentes existentes para que pueda seguir usando su funcionalidad existente durante la evaluación o la migración. Esto es especialmente importante debido a las limitaciones de la versión preliminar pública para admitir las soluciones existentes. Sin embargo, debe tener cuidado al recopilar datos duplicados, ya que esto podría sesgar los resultados de las consultas y generar cargos adicionales por la ingesta y retención de datos.

Por ejemplo, VM Insights usa el agente de Log Analytics para enviar datos de rendimiento a un área de trabajo de Log Analytics. También puede haber configurado el área de trabajo para recopilar eventos de Windows y eventos de Syslog de los agentes. Si instala el agente de Azure Monitor y crea una regla de recopilación de datos para estos mismos eventos y datos de rendimiento, se generarán datos duplicados.


## <a name="costs"></a>Costos
El agente de Azure Monitor no cuesta nada, pero puede incurrir en gastos por los datos ingeridos. Consulte [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/) para obtener más información sobre la recopilación y retención de datos de Log Analytics, y para las métricas de los clientes.

## <a name="data-sources-and-destinations"></a>Orígenes y destinos de los datos
En la tabla siguiente se enumeran los tipos de datos que se pueden recopilar actualmente con el agente de Azure Monitor mediante reglas de recopilación de datos y a dónde se pueden enviar los datos. Consulte [¿Qué supervisa Azure Monitor?](../monitor-reference.md) para ver una lista con información detallada, soluciones y otros servicios que usan el agente de Azure Monitor para recopilar otros tipos de datos.


El agente de Azure Monitor envía datos a métricas de Azure Monitor o a un área de trabajo de Log Analytics que admite registros de Azure Monitor.

| Origen de datos | Destinations | Descripción |
|:---|:---|:---|
| Rendimiento        | Métricas de Azure Monitor<br>Área de trabajo de Log Analytics | Valores numéricos que miden el rendimiento de diferentes aspectos del sistema operativo y las cargas de trabajo. |
| Registros de eventos de Windows | Área de trabajo de Log Analytics | Información enviada al sistema de registro de eventos de Windows. |
| syslog             | Área de trabajo de Log Analytics | Información enviada al sistema de registro de eventos de Windows. |


## <a name="supported-operating-systems"></a>Sistemas operativos admitidos
Consulte [Sistemas operativos admitidos](agents-overview.md#supported-operating-systems) para obtener una lista de las versiones de sistemas operativos Windows y Linux compatibles actualmente con el agente de Azure Monitor.



## <a name="security"></a>Seguridad
El agente de Azure Monitor no requiere ninguna clave, pero en su lugar requiere una [identidad administrada asignada por el sistema](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md#system-assigned-managed-identity). Debe tener una identidad administrada asignada por el sistema habilitada en cada máquina virtual antes de implementar el agente.

## <a name="networking"></a>Redes
El agente de Azure Monitor admite etiquetas de servicio de Azure (se requieren tanto etiquetas AzureMonitor como AzureResourceManager), pero todavía no funciona en ámbitos de Private Link de Azure Monitor o proxies directos.


## <a name="next-steps"></a>Pasos siguientes

- [Instale el agente de Azure Monitor](azure-monitor-agent-install.md) en máquinas virtuales Windows y Linux.
- [Cree una regla de recopilación de datos](data-collection-rule-azure-monitor-agent.md) para recopilar datos del agente y enviarlos a Azure Monitor.
