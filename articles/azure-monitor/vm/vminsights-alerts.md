---
title: Alertas de VM Insights
description: Describe cómo crear reglas de alerta a partir de datos de rendimiento recopilados por VM Insights.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 11/10/2020
ms.openlocfilehash: c921637fc0c50125f6cc657dc9329e2107e8e774
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111894675"
---
# <a name="how-to-create-alerts-from-vm-insights"></a>Creación de alertas a partir de VM Insights
Las [alertas de Azure Monitor](../alerts/alerts-overview.md) notifican al usuario proactivamente sobre datos y patrones interesantes en los datos de supervisión. VM Insights no incluye reglas de alerta preconfiguradas, pero puede crear las suyas propias en función de los datos que recopila. En este artículo se proporcionan instrucciones sobre cómo crear reglas de alerta, incluido un conjunto de consultas de ejemplo.

> [!IMPORTANT]
> Las alertas descritas en este artículo se basan en las consultas de registro de los datos recopilados por VM Insights. Son diferentes de las alertas creadas por el [estado de invitado de Azure Monitor para VM](vminsights-health-overview.md), que es una característica que se encuentra actualmente en versión preliminar pública. A medida que esta característica se acerque a la disponibilidad general, se consolidarán las instrucciones para las alertas.


## <a name="alert-rule-types"></a>Tipos de reglas de alerta
Azure Monitor tiene [distintos tipos de reglas de alerta](../alerts/alerts-overview.md#what-you-can-alert-on) en función de los datos que se usan para crear la alerta. Todos los datos recopilados por VM Insights se almacenan en registros de Azure Monitor que admiten [alertas de registro](../alerts/alerts-log.md). Actualmente no puede usar [alertas de métricas](../alerts/alerts-log.md) con los datos de rendimiento recopilados de VM Insights porque los datos no se recopilan en métricas de Azure Monitor. Para recopilar datos para alertas de métricas, instale la [extensión de diagnósticos](../agents/diagnostics-extension-overview.md) para máquinas virtuales Windows o el [agente Telegraf](../essentials/collect-custom-metrics-linux-telegraf.md) para máquinas virtuales Linux para recopilar datos de rendimiento en métricas.

Hay dos tipos de alertas de registro en Azure Monitor:

- [Alertas de número de resultados](../alerts/alerts-unified-log.md#count-of-the-results-table-rows), que crean una única alerta cuando una consulta devuelve al menos un número especificado de registros. Son ideales para datos no numéricos, como los eventos de Windows y Syslog recopilados por el [agente de Log Analytics](../agents/log-analytics-agent.md) o para analizar las tendencias de rendimiento en varios equipos.
- [Alertas de medición de métricas](../alerts/alerts-unified-log.md#calculation-of-measure-based-on-a-numeric-column-such-as-cpu-counter-value), que crean una alerta independiente para cada registro de una consulta que tenga un valor que supere un umbral definido en la regla de alerta. Estas reglas de alerta son ideales para los datos de rendimiento recopilados por VM Insights, ya que pueden crear alertas individuales para cada equipo.


## <a name="alert-rule-walkthrough"></a>Tutorial de reglas de alerta
Esta sección le guía en la creación de una regla de alerta de medición de métricas mediante datos de rendimiento de VM Insights. Puede usar este proceso básico con diversas consultas de registro para alertar sobre diferentes contadores de rendimiento.

Empiece por crear una nueva regla de alerta siguiendo el procedimiento descrito en [Creación, visualización y administración de alertas de registro mediante Azure Monitor](../alerts/alerts-log.md). En el **recurso** seleccione el área de trabajo de Log Analytics que Azure Monitor para VM usa en la suscripción. Puesto que el recurso de destino para las reglas de alerta de registro es siempre un área de trabajo de Log Analytics, la consulta de registro debe incluir cualquier filtro para máquinas virtuales concretas o conjuntos de escalado de máquinas virtuales. 

Para la **condición** de la regla de alerta, use una de las consultas de la [sección que aparece a continuación](#sample-alert-queries) como la **consulta de búsqueda**. La consulta debe devolver una propiedad numérica denominada *AggregatedValue*. Debe resumir los datos por equipo para que pueda crear una alerta independiente para cada máquina virtual que supere el umbral.

En **Lógica de alerta**, seleccione **Unidades métricas** y proporcione un valor en **Valor del umbral**. En **Desencadenar alerta según**, especifique el número de veces que se debe superar el umbral antes de que se cree una alerta. Por ejemplo, es probable que no le interese el escenario en el que el procesador supera un umbral una vez y luego vuelve a la normalidad, pero sí si sigue superando el umbral en varias medidas consecutivas.

La sección **Se evaluó basándose en** define la frecuencia con la que se ejecuta la consulta y el período de tiempo de esta. En el ejemplo que se muestra a continuación, la consulta se ejecutará cada quince minutos y evaluará los valores de rendimiento recopilados durante los quince minutos anteriores.


![Regla de alerta para medición de métricas](media/vminsights-alerts/metric-measurement-alert.png)

## <a name="sample-alert-queries"></a>Consultas de alerta de ejemplo
Las siguientes consultas se pueden usar con una regla de alerta de medición de métricas con los datos de rendimiento recopilados por VM Insights. Cada una resume los datos por equipo para que se cree una alerta para cada equipo con un valor que supere el umbral.

### <a name="cpu-utilization"></a>Uso de CPU

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms" 
| where Namespace == "Processor" and Name == "UtilizationPercentage" 
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId
```

### <a name="available-memory-in-mb"></a>Memoria disponible en MB

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Memory" and Name == "AvailableMB"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId
```

### <a name="available-memory-in-percentage"></a>Memoria disponible en porcentaje

```kusto
InsightsMetrics 
| where Origin == "vm.azm.ms" 
| where Namespace == "Memory" and Name == "AvailableMB" 
| extend TotalMemory = toreal(todynamic(Tags)["vm.azm.ms/memorySizeMB"])
| extend AvailableMemoryPercentage = (toreal(Val) / TotalMemory) * 100.0 
| summarize AggregatedValue = avg(AvailableMemoryPercentage) by bin(TimeGenerated, 15m), Computer, _ResourceId 
```

### <a name="logical-disk-used---all-disks-on-each-computer"></a>Disco lógico usado: todos los discos de cada equipo

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "LogicalDisk" and Name == "FreeSpacePercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId 
```

### <a name="logical-disk-used---individual-disks"></a>Disco lógico usado: discos individuales

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "LogicalDisk" and Name == "FreeSpacePercentage"
| extend Disk=tostring(todynamic(Tags)["vm.azm.ms/mountId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId, Disk
```

### <a name="logical-disk-iops"></a>IOPS de disco lógico

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms" 
| where Namespace == "LogicalDisk" and Name == "TransfersPerSecond"
| extend Disk=tostring(todynamic(Tags)["vm.azm.ms/mountId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId, Disk
```

### <a name="logical-disk-data-rate"></a>Velocidad de datos de disco lógico

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms" 
| where Namespace == "LogicalDisk" and Name == "BytesPerSecond"
| extend Disk=tostring(todynamic(Tags)["vm.azm.ms/mountId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId, Disk
```

### <a name="network-interfaces-bytes-received---all-interfaces"></a>Bytes de interfaces de red recibidos: todas las interfaces

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Network" and Name == "ReadBytesPerSecond"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId 
```

### <a name="network-interfaces-bytes-received---individual-interfaces"></a>Bytes de interfaces de red recibidos: interfaces individuales

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Network" and Name == "ReadBytesPerSecond"
| extend NetworkInterface=tostring(todynamic(Tags)["vm.azm.ms/networkDeviceId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId, NetworkInterface
```

### <a name="network-interfaces-bytes-sent---all-interfaces"></a>Bytes de interfaces de red enviados: todas las interfaces

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Network" and Name == "WriteBytesPerSecond"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId
```

### <a name="network-interfaces-bytes-sent---individual-interfaces"></a>Bytes de interfaces de red enviados: interfaces individuales

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Network" and Name == "WriteBytesPerSecond"
| extend NetworkInterface=tostring(todynamic(Tags)["vm.azm.ms/networkDeviceId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId, NetworkInterface
```

### <a name="virtual-machine-scale-set"></a>Conjunto de escalado de máquina virtual
Realice la modificación por el identificador de suscripción, el grupo de recursos y el nombre del conjunto de escalado de máquinas virtuales.

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where _ResourceId startswith "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachineScaleSets/my-vm-scaleset" 
| where Namespace == "Processor" and Name == "UtilizationPercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), _ResourceId
```

### <a name="specific-virtual-machine"></a>Máquina virtual específica
Realice la modificación por el identificador de suscripción, el grupo de recursos y el nombre de la máquina virtual.

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where _ResourceId =~ "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachines/my-vm" 
| where Namespace == "Processor" and Name == "UtilizationPercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m)
```

### <a name="cpu-utilization-for-all-compute-resources-in-a-subscription"></a>Utilización de la CPU para todos los recursos de proceso en una suscripción
Realice la modificación por el identificador de suscripción.

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where _ResourceId startswith "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" and (_ResourceId contains "/providers/Microsoft.Compute/virtualMachines/" or _ResourceId contains "/providers/Microsoft.Compute/virtualMachineScaleSets/")
| where Namespace == "Processor" and Name == "UtilizationPercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), _ResourceId
```

### <a name="cpu-utilization-for-all-compute-resources-in-a-resource-group"></a>Utilización de la CPU para todos los recursos de proceso en un grupo de recursos
Realice la modificación por el identificador de suscripción y el grupo de recursos.

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where _ResourceId startswith "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachines/"
or _ResourceId startswith "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachineScaleSets/" 
| where Namespace == "Processor" and Name == "UtilizationPercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), _ResourceId

```

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre [alertas en Azure Monitor](../alerts/alerts-overview.md).
- Obtenga más información sobre [consultas de registro con datos de VM Insights](vminsights-log-search.md).
