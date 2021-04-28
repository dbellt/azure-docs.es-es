---
title: Supervisión de Azure Machine Learning | Microsoft Docs
description: Aprenda a usar Azure Monitor para ver, analizar y crear alertas de métricas de Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: larryfr
ms.author: aashishb
author: aashishb
ms.custom: subject-monitoring
ms.date: 10/01/2020
ms.openlocfilehash: e5fd0fdd5a6f9a4a7537a844b096efdfef253638
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107816861"
---
# <a name="monitor-azure-machine-learning"></a>Supervisión de Azure Machine Learning

Si tiene aplicaciones y procesos empresariales críticos que dependen de recursos de Azure, querrá supervisar esos recursos para su disponibilidad, rendimiento y funcionamiento. En este artículo se describen los datos de supervisión que ha generado Azure Machine Learning y cómo analizar y alertar sobre estos datos con Azure Monitor.

> [!TIP]
> La información de este documento va dirigida principalmente a los __administradores__, ya que describe la supervisión de Azure Machine Learning Service y de los servicios de Azure asociados. Los __científicos de datos__ o __desarrolladores__ que quieran supervisar información específica de las *ejecuciones de entrenamiento de un modelo* pueden consultar los siguientes documentos:
>
> * [Inicio, supervisión y cancelación de las ejecuciones de entrenamiento](how-to-track-monitor-analyze-runs.md)
> * [Métricas de registro de las ejecuciones de entrenamientos](how-to-log-view-metrics.md)
> * [Seguimiento de experimentos con MLflow](how-to-use-mlflow.md)
> * [Visualización de ejecuciones con TensorBoard](how-to-monitor-tensorboard.md)
>
> Si quiere supervisar la información generada por los modelos implementados como servicios web o módulos de IoT Edge, consulte [Recopilación de datos de modelos](how-to-enable-data-collection.md) y [Supervisión con Application Insights](how-to-enable-app-insights.md).

## <a name="what-is-azure-monitor"></a>¿Qué es Azure Monitor?

Azure Machine Learning crea los datos de supervisión mediante [Azure Monitor](../azure-monitor/overview.md), que es un servicio de supervisión de pila completo en Azure. Azure Monitor proporciona un conjunto completo de características para supervisar los recursos de Azure. También puede supervisar los recursos de otras nubes y de forma local.

Comience leyendo el artículo [Supervisión de recursos de Azure con Azure Monitor](../azure-monitor/essentials/monitor-azure-resource.md), en el que se describen los conceptos siguientes:

- ¿Qué es Azure Monitor?
- Costos asociados con la supervisión
- Datos de supervisión recopilados en Azure
- Configuración de la recolección de datos
- Herramientas estándar en Azure para analizar datos de supervisión y alertar sobre ellos

Las secciones siguientes complementan este artículo mediante la descripción de los datos específicos que se recopilan de Azure Machine Learning. En estas secciones también se proporcionan ejemplos para configurar la recopilación y el análisis de datos con Azure Tools.

> [!TIP]
> Para comprender los costos asociados a Azure Monitor, consulte [Uso y costos estimados](../azure-monitor//usage-estimated-costs.md). Para comprender el tiempo necesario para que los datos aparezcan en Azure Monitor, consulte [Tiempo de ingesta de datos de registro](../azure-monitor/logs/data-ingestion-time.md).

## <a name="monitoring-data-from-azure-machine-learning"></a>Supervisión de datos de Azure Machine Learning

Azure Machine Learning recopila los mismos tipos de datos de supervisión que otros recursos de Azure que se describen en [Supervisión de datos de recursos de Azure](../azure-monitor/essentials/monitor-azure-resource.md#monitoring-data). 

Consulte [Referencia de datos de supervisión de Azure Machine Learning](monitor-resource-reference.md) para obtener una referencia detallada de los registros y las métricas creados por Azure Machine Learning.

<a id="configuration"></a>

## <a name="collection-and-routing"></a>Recopilación y enrutamiento

Las métricas de la plataforma y el registro de actividad se recopilan y almacenan de forma automática, pero se pueden enrutar a otras ubicaciones mediante una configuración de diagnóstico.  

Los registros de recursos no se recopilan ni almacenan hasta que se crea una configuración de diagnóstico y se enrutan a una o varias ubicaciones.

Consulte [Creación de una configuración de diagnóstico para recopilar registros de plataforma y métricas en Azure](../azure-monitor/essentials/diagnostic-settings.md) para ver el proceso detallado para crear una configuración de diagnóstico mediante el Azure Portal, la CLI o PowerShell. Cuando se crea una configuración de diagnóstico, se especifican las categorías de registros que se van a recopilar. Las categorías de Azure Machine Learning se muestran en la [Referencia de datos de supervisión de Azure Machine Learning](monitor-resource-reference.md#resource-logs).

> [!IMPORTANT]
> La habilitación de esta configuración requiere servicios adicionales de Azure (cuenta de almacenamiento, centro de eventos o Log Analytics), lo que puede aumentar el costo. Para calcular un costo estimado, visite la [Calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator).

Puede configurar los siguientes registros para Azure Machine Learning:

| Category | Descripción |
|:---|:---|
| AmlComputeClusterEvent | Eventos de clústeres de proceso de Azure Machine Learning. |
| AmlComputeClusterNodeEvent | Eventos de nodos dentro de un clúster de proceso de Azure Machine Learning. |
| AmlComputeJobEvent | Eventos de trabajos en ejecución en un proceso de Azure Machine Learning. |

> [!NOTE]
> Cuando se habilitan las métricas en una configuración de diagnóstico, la información de dimensión no se incluye actualmente como parte de la información que se envía a una cuenta de almacenamiento, a un centro de eventos o al análisis de registros.

En las secciones siguientes se describen las métricas y los registros que se pueden recopilar.

## <a name="analyzing-metrics"></a>Análisis de métricas

Puede analizar las métricas de Azure Machine Learning con métricas de otros servicios de Azure mediante la opción **Métricas** del menú de **Azure Monitor**. Consulte [Introducción al explorador de métricas de Azure](../azure-monitor/essentials/metrics-getting-started.md) para más información sobre esta herramienta.

Para obtener una lista de las métricas de la plataforma recopiladas, consulte [Supervisión de las métricas de referencia de datos Azure Machine Learning](monitor-resource-reference.md#metrics).

Todas las métricas de Azure Machine Learning se encuentran en el espacio de nombres **Área de trabajo de servicio de Machine Learning**.

![Explorador de métricas con Machine Learning Service Workspace seleccionada](./media/monitor-azure-machine-learning/metrics.png)

Como referencia, puede ver una lista de [todas las métricas de recursos que se admiten en Azure Monitor](../azure-monitor/essentials/metrics-supported.md).

> [!TIP]
> Los datos de métricas de Azure Monitor están disponibles durante 90 días. Sin embargo, cuando se crean gráficos, solo se pueden visualizar durante 30 días. Por ejemplo, si quiere visualizar un período de 90 días, debe dividirlo en tres gráficos de 30 días dentro de ese período.
### <a name="filtering-and-splitting"></a>Filtrar y dividir

En el caso de las métricas que admiten dimensiones, puede aplicar filtros con un valor de dimensión. Por ejemplo, el filtrado de **Núcleos activos** para un **Nombre de clúster** de `cpu-cluster`. 

Puede dividir una métrica por dimensión para visualizar la comparación de unos segmentos de la métrica con otros. Por ejemplo, dividir el **Tipo de paso de canalización** para ver un recuento de los tipos de pasos que se usan en la canalización.

Para más información sobre el filtrado y la división, vea [Características avanzadas de Azure Monitor](../azure-monitor/essentials/metrics-charts.md).

<a id="analyzing-log-data"></a>
## <a name="analyzing-logs"></a>Análisis de datos

El uso de Log Analytics de Azure Monitor requiere la creación de una configuración de diagnóstico y habilitar __Enviar información a Log Analytics__. Para obtener más información, consulte la sección [Recolección y enrutamiento](#collection-and-routing).

Los datos de los registros de Azure Monitor se almacenan en tablas, cada tabla tiene su propio conjunto de propiedades únicas. Azure Machine Learning almacena los datos en las tablas siguientes:

| Tabla | Descripción |
|:---|:---|
| AmlComputeClusterEvent | Eventos de clústeres de proceso de Azure Machine Learning. |
| AmlComputeClusterNodeEvent | Eventos de nodos dentro de un clúster de proceso de Azure Machine Learning. |
| AmlComputeJobEvent | Eventos de trabajos en ejecución en un proceso de Azure Machine Learning. |

> [!IMPORTANT]
> Al seleccionar **Registros** en el menú Azure Machine Learning, Log Analytics se abre con el ámbito de la consulta establecido en el área de trabajo actual. Esto significa que las consultas de registro solo incluirán datos de ese recurso. Si desea ejecutar una consulta que incluya datos de otras bases de datos o de otros servicios de Azure, elija **Registros** en el menú **Azure Monitor**. Consulte [Ámbito e intervalo de tiempo de una consulta de registro en Log Analytics de Azure Monitor](../azure-monitor/logs/scope.md) para obtener más información.

Consulte [Referencia de datos de supervisión de Azure Machine Learning](monitor-resource-reference.md) para una referencia detallada de los registros y las métricas.

### <a name="sample-kusto-queries"></a>Ejemplos de consultas de Kusto

> [!IMPORTANT]
> Al seleccionar **Registros** del menú de [service-name], Log Analytics se abre con el ámbito de la consulta establecido en el área de trabajo actual de Azure Machine Learning. Esto significa que las consultas de registro solo incluirán datos de ese recurso. Si quiere ejecutar una consulta que incluya datos de otras áreas de trabajo o de otros servicios de Azure, seleccione **Registros** en el menú de **Azure Monitor**. Consulte [Ámbito e intervalo de tiempo de una consulta de registro en Log Analytics de Azure Monitor](../azure-monitor/logs/scope.md) para obtener más información.

A continuación se muestran las consultas que puede usar para supervisar los recursos de Azure Machine Learning: 

+ Obtener trabajos con errores en los últimos cinco días:

    ```Kusto
    AmlComputeJobEvent
    | where TimeGenerated > ago(5d) and EventType == "JobFailed"
    | project  TimeGenerated , ClusterId , EventType , ExecutionState , ToolType
    ```

+ Obtener registros para un nombre de trabajo específico:

    ```Kusto
    AmlComputeJobEvent
    | where JobName == "automl_a9940991-dedb-4262-9763-2fd08b79d8fb_setup"
    | project  TimeGenerated , ClusterId , EventType , ExecutionState , ToolType
    ```

+ Obtiene los eventos de clúster en los últimos cinco días para los clústeres en los que el tamaño de la máquina virtual es Standard_D1_V2:

    ```Kusto
    AmlComputeClusterEvent
    | where TimeGenerated > ago(4d) and VmSize == "STANDARD_D1_V2"
    | project  ClusterName , InitialNodeCount , MaximumNodeCount , QuotaAllocated , QuotaUtilized
    ```

+ Obtener nodos asignados en los últimos ocho días:

    ```Kusto
    AmlComputeClusterNodeEvent
    | where TimeGenerated > ago(8d) and NodeAllocationTime  > ago(8d)
    | distinct NodeId
    ```

## <a name="alerts"></a>Alertas

Puede acceder a las métricas de Azure Machine Learning abriendo **Alertas** en el menú **Azure Monitor**. Vea [Creación, visualización y administración de alertas de métricas mediante Azure Monitor](../azure-monitor/alerts/alerts-metric.md) para más detalles sobre la creación de alertas.

En la tabla siguiente se enumeran las reglas de alertas de métricas comunes y recomendadas para Azure Machine Learning:

| Tipo de alerta | Condición | Descripción |
|:---|:---|:---|
| Error al realizar la implementación de modelo | Tipo de agregación: Total, operador: Mayor que, valor de umbral: 0 | Cuando se han producido errores en una o varias implementaciones de modelos |
| Porcentaje de uso de la cuota | Tipo de agregación: Media, operador: Mayor que, valor de umbral: 90| Cuando el porcentaje de uso de la cuota es superior al 90 % |
| Nodos no utilizables | Tipo de agregación: Total, operador: Mayor que, valor de umbral: 0 | Cuando hay uno o más nodos no utilizables |

## <a name="next-steps"></a>Pasos siguientes

- Consulte [Supervisión de la referencia de datos de Azure Machine Learning](monitor-resource-reference.md) como referencia de los registros y las métricas.
- Para obtener información sobre cómo trabajar con cuotas relacionadas con Azure Machine Learning, consulte [Administración y solicitud de cuotas para recursos de Azure](how-to-manage-quotas.md).
- Para más información sobre la supervisión de recursos de Azure, consulte [Supervisión de recursos de Azure con Azure Monitor](../azure-monitor/essentials/monitor-azure-resource.md).
