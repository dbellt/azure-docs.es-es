---
title: Configuración de la supervisión de VP con Container insights | Microsoft Docs
description: En este artículo se describe cómo puede configurar la supervisión de clústeres de Kubernetes con volúmenes persistentes mediante Container Insights.
ms.topic: conceptual
ms.date: 03/03/2021
ms.openlocfilehash: 593ee36bf01a7f2979b7db9ef8b28e0f805077ad
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110065663"
---
# <a name="configure-pv-monitoring-with-container-insights"></a>Configuración de la supervisión de VP con Container insights

A partir de la versión del agente *ciprod10052020*, el agente integrado de Container Insights ahora admite la supervisión del uso de VP (volúmenes persistentes). Con la versión del agente *ciprod01112021*, el agente admite la supervisión de inventario de VP, incluida información sobre el estado, la clase de almacenamiento, el tipo, los modos de acceso y otros detalles.
## <a name="pv-metrics"></a>Métricas de VP

Container Insights inicia automáticamente la supervisión del uso de PV mediante la recopilación de las métricas siguientes a intervalos de 60 segundos y su almacenamiento en la tabla **InsightMetrics**.

| Nombre de métrica | Dimensión de métrica (etiquetas) | Descripción de la métrica |
|-----|-----------|----------|
| `pvUsedBytes`| `podUID`, `podName`, `pvcName`, `pvcNamespace`, `capacityBytes`, `clusterId`, `clusterName`| Espacio usado en bytes para un volumen persistente específico con una notificación utilizada por un pod concreto. `capacityBytes` se pliega como una dimensión en el campo Etiquetas para reducir el costo de ingesta de datos y simplificar las consultas.|

Obtenga más información sobre la configuración de las métricas de VP recopiladas [aquí](./container-insights-agent-config.md).

## <a name="pv-inventory"></a>Inventario de VP

Container Insights para contenedores inicia automáticamente la supervisión de los volúmenes persistentes mediante la recopilación de la siguiente información a intervalos de 60 insights y su almacenamiento en la tabla **KubePVInventory**.

|data |Origen de datos| Tipo de datos| Campos|
|-----|-----------|----------|-------|
|Inventario de volúmenes persistentes en un clúster de Kubernetes |API de Kube |`KubePVInventory` |    `PVName`, `PVCapacityBytes`, `PVCName`, `PVCNamespace`, `PVStatus`, `PVAccessModes`, `PVType`, `PVTypeInfo`, `PVStorageClassName`, `PVCreationTimestamp`, `TimeGenerated`, `ClusterId`, `ClusterName`, `_ResourceId` |

## <a name="monitor-persistent-volumes"></a>Supervisión de volúmenes persistentes

Container Insights incluye gráficos preconfigurados para esta métrica de uso e información sobre el inventario en plantillas de libro para cada clúster. También puede habilitar una alerta recomendada sobre el uso de VP y consultar estas métricas en Log Analytics.  

### <a name="workload-details-workbook"></a>Libro Detalles de la carga de trabajo

Puede buscar gráficos de uso para cargas de trabajo específicas en la pestaña de volumen persistente del libro **Detalles de la carga de trabajo** directamente desde un clúster de AKS. Para ello, seleccione Libros en el panel izquierdo, en la lista desplegable **Ver libros** del panel Información o en la pestaña **Informes (versión preliminar)** del panel Información.


:::image type="content" source="./media/container-insights-persistent-volumes/pv-workload-example.PNG" alt-text="Ejemplo de libro de cargas de trabajo de VP de Azure Monitor":::

### <a name="persistent-volume-details-workbook"></a>Libro Detalles del volumen persistente

Puede encontrar información general sobre el inventario de volumen persistente en el libro **Detalles del volumen persistente** directamente desde un clúster de AKS. Para ello, seleccione Libros en el panel izquierdo. Este libro también se puede abrir desde la lista desplegable **Ver libros** del panel Información o desde la pestaña **Informes** del panel Información.


:::image type="content" source="./media/container-insights-persistent-volumes/pv-details-workbook-example.PNG" alt-text="Ejemplo de libro de detalles de volumen persistente de Azure Monitor":::

### <a name="persistent-volume-usage-recommended-alert"></a>Alerta recomendada de uso de volumen persistente
Puede habilitar una alerta recomendada que le avise cuando el uso medio de volumen persistente de un pod esté por encima del 80 %. Obtenga más información sobre las alertas [aquí](./container-insights-metric-alerts.md) y sobre cómo invalidar el umbral predeterminado [aquí](./container-insights-metric-alerts.md#configure-alertable-metrics-in-configmaps).
## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre las métricas de VP recopiladas [aquí](./container-insights-agent-config.md).