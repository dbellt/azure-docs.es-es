---
title: Supervisión de Azure Event Hubs
description: Aprenda a usar Azure Monitor para ver, analizar y crear alertas sobre las métricas de Azure Event Hubs.
ms.topic: conceptual
ms.custom: subject-monitoring
ms.date: 06/13/2021
ms.openlocfilehash: 1a7ea209b3aaf47095723f8d2831fff4a94ffe54
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112063803"
---
# <a name="monitor-azure-event-hubs"></a>Supervisión de Azure Event Hubs
Si tiene aplicaciones y procesos empresariales críticos que dependen de recursos de Azure, querrá supervisar esos recursos para su disponibilidad, rendimiento y funcionamiento. En este artículo se describen los datos de supervisión que ha generado Azure Event Hubs y cómo analizar y generar alertas sobre estos datos con Azure Monitor.

## <a name="what-is-azure-monitor"></a>¿Qué es Azure Monitor?
Azure Event Hubs crea datos de supervisión mediante [Azure Monitor](../azure-monitor/overview.md), que es un servicio de supervisión de pila completa de Azure. Azure Monitor proporciona un conjunto completo de características para supervisar los recursos de Azure. También puede supervisar los recursos de otras nubes y de forma local.

Comience leyendo el artículo [Supervisión de recursos de Azure con Azure Monitor](../azure-monitor/essentials/monitor-azure-resource.md), en el que se describen los conceptos siguientes:

- ¿Qué es Azure Monitor?
- Costos asociados con la supervisión
- Datos de supervisión recopilados en Azure
- Configuración de la recolección de datos
- Herramientas estándar en Azure para analizar datos de supervisión y alertar sobre ellos

En las secciones siguientes se desarrolla este artículo mediante la descripción de los datos específicos que se recopilan para Azure Event Hubs. En estas secciones también se proporcionan ejemplos para configurar la recopilación y el análisis de datos con Azure Tools.

> [!TIP]
> Para comprender los costos asociados a Azure Monitor, consulte [Uso y costos estimados](../azure-monitor//usage-estimated-costs.md). Para comprender el tiempo necesario para que los datos aparezcan en Azure Monitor, consulte [Tiempo de ingesta de datos de registro](../azure-monitor/logs/data-ingestion-time.md).

## <a name="monitoring-data-from-azure-event-hubs"></a>Supervisión de datos de Azure Event Hubs
Azure Event Hubs recopila los mismos tipos de datos de supervisión que otros recursos de Azure, que se describen en [Supervisión de datos de recursos de Azure](../azure-monitor/essentials/monitor-azure-resource.md#monitoring-data). 

Consulte [Referencia de datos de supervisión de Azure Event Hubs](monitor-event-hubs-reference.md) para una referencia detallada de los registros y las métricas creados por Azure Event Hubs.

## <a name="collection-and-routing"></a>Recopilación y enrutamiento
Tanto las métricas de la plataforma como el registro de actividad se recopilan y almacenan de forma automática, pero se pueden enrutar a otras ubicaciones mediante una configuración de diagnóstico.  

Los registros de recursos no se recopilan ni almacenan hasta que se crea una configuración de diagnóstico y se enrutan a una o varias ubicaciones.

Consulte [Creación de una configuración de diagnóstico para recopilar registros de plataforma y métricas en Azure](../azure-monitor/essentials/diagnostic-settings.md) para ver el proceso detallado para crear una configuración de diagnóstico mediante el Azure Portal, la CLI o PowerShell. Cuando se crea una configuración de diagnóstico, se especifican las categorías de registros que se van a recopilar. Las categorías de Azure Event Hubs se enumeran en la [referencia de datos de supervisión de Azure Event Hubs](monitor-event-hubs-reference.md#resource-logs).

Si usa **Azure Storage** para almacenar la información de los registros de diagnóstico, esta se almacena en los contenedores **insights-logs-operationlogs** e **insights-metrics-pt1m**. Dirección URL de ejemplo para un registro de operaciones: `https://<Azure Storage account>.blob.core.windows.net/insights-logs-operationallogs/resourceId=/SUBSCRIPTIONS/<Azure subscription ID>/RESOURCEGROUPS/<Resource group name>/PROVIDERS/MICROSOFT.SERVICEBUS/NAMESPACES/<Namespace name>/y=<YEAR>/m=<MONTH-NUMBER>/d=<DAY-NUMBER>/h=<HOUR>/m=<MINUTE>/PT1H.json`. La dirección URL de un registro de métricas es similar. 

Si usa **Azure Event Hubs** para almacenar la información de los registros de diagnóstico, esta se almacena en los centros de eventos **insights-logs-operationlogs** e **insights-metrics-pt1m**. También puede seleccionar su propio centro de eventos. 

Si usa **Log Analytics** para almacenar la información de los registros de diagnóstico, esta se almacena en tablas llamadas **AzureDiagnostics** y **AzureMetrics**. 

> [!IMPORTANT]
> La habilitación de esta configuración requiere servicios adicionales de Azure (cuenta de almacenamiento, centro de eventos o Log Analytics), lo que puede aumentar el costo. Para calcular un costo estimado, visite la [Calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator).

> [!NOTE]
> Cuando se habilitan las métricas en una configuración de diagnóstico, la información de dimensión no se incluye actualmente como parte de la información que se envía a una cuenta de almacenamiento, a un centro de eventos o al análisis de registros.

En las secciones siguientes se describen las métricas y los registros que se pueden recopilar.

## <a name="analyzing-metrics"></a>Análisis de métricas
Puede analizar las métricas de Azure Event Hubs, junto con las métricas de otros servicios de Azure, seleccionando **Métricas** en la sección **Azure Monitor** de la página principal del espacio de nombres de Event Hubs. Consulte [Introducción al explorador de métricas de Azure](../azure-monitor/essentials/metrics-getting-started.md) para más información sobre esta herramienta. Para ver una lista de las métricas de la plataforma recopiladas, consulte [Supervisión de las métricas de referencia de datos de Azure Event Hubs](monitor-event-hubs-reference.md#metrics).

![Explorador de métricas con el espacio de nombres de Event Hubs seleccionado](./media/monitor-event-hubs/metrics.png)

Como referencia, puede ver una lista de [todas las métricas de recursos que se admiten en Azure Monitor](../azure-monitor/essentials/metrics-supported.md).

> [!TIP]
> Los datos de métricas de Azure Monitor están disponibles durante 90 días. Sin embargo, cuando se crean gráficos, solo se pueden visualizar durante 30 días. Por ejemplo, si quiere visualizar un período de 90 días, debe dividirlo en tres gráficos de 30 días dentro de ese período.

### <a name="filtering-and-splitting"></a>Filtrar y dividir
En el caso de las métricas que admiten dimensiones, puede aplicar filtros con un valor de dimensión. Por ejemplo, agregue un filtro con `EntityName` establecido en el nombre de un centro de eventos. Puede dividir una métrica por dimensión para visualizar la comparación de unos segmentos de la métrica con otros. Para más información sobre el filtrado y la división, vea [Características avanzadas de Azure Monitor](../azure-monitor/essentials/metrics-charts.md).

:::image type="content" source="./media/monitor-event-hubs/metrics-filter-split.png" alt-text="Imagen que muestra el filtrado y la división de métricas":::

## <a name="analyzing-logs"></a>Análisis de datos
El uso de Log Analytics de Azure Monitor requiere la creación de una configuración de diagnóstico y habilitar __Enviar información a Log Analytics__. Para obtener más información, consulte la sección [Recolección y enrutamiento](#collection-and-routing). Los datos de los registros de Azure Monitor se almacenan en tablas, cada tabla tiene su propio conjunto de propiedades únicas. Azure Event Hubs almacena los datos en las tablas **AzureDiagnostics** y **AzureMetrics**.

> [!IMPORTANT]
> Al seleccionar **Registros** en el menú de Azure Event Hubs, se abre Log Analytics con el ámbito de la consulta establecido en el área de trabajo actual. Esto significa que las consultas de registro solo incluirán datos de ese recurso. Si desea ejecutar una consulta que incluya datos de otras bases de datos o de otros servicios de Azure, elija **Registros** en el menú **Azure Monitor**. Consulte [Ámbito e intervalo de tiempo de una consulta de registro en Log Analytics de Azure Monitor](../azure-monitor/logs/scope.md) para obtener más información.

Para encontrar una referencia detallada de los registros y las métricas, consulte [Referencia de datos de supervisión de Azure Event Hubs](monitor-event-hubs-reference.md).

### <a name="sample-kusto-queries"></a>Ejemplos de consultas de Kusto

> [!IMPORTANT]
> Al seleccionar **Registros** en el menú de Azure Event Hubs, se abre Log Analytics con el ámbito de la consulta establecido en el espacio de nombres actual de Azure Event Hubs. Esto significa que las consultas de registro solo incluirán datos de ese recurso. Si quiere ejecutar una consulta que incluya datos de otras áreas de trabajo o de otros servicios de Azure, seleccione **Registros** en el menú de **Azure Monitor**. Consulte [Ámbito e intervalo de tiempo de una consulta de registro en Log Analytics de Azure Monitor](../azure-monitor/logs/scope.md) para obtener más información.

A continuación, se muestran las consultas de ejemplo que se pueden usar para supervisar los recursos de Azure Event Hubs: 

+ Obtener errores de los últimos 7 días.

    ```Kusto
    AzureDiagnostics
    | where TimeGenerated > ago(7d)
    | where ResourceProvider =="MICROSOFT.EVENTHUB"
    | where Category == "OperationalLogs"
    | summarize count() by "EventName"
    ```

+ Obtener los intentos de acceso a un almacén de claves que provocaron el error "No se encontró la clave".

    ```Kusto
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.EVENTHUB" 
    | where Category == "Error" and OperationName == "wrapkey"
    | project Message
    ```

+ Obtener las operaciones realizadas con un almacén de claves para deshabilitar o restaurar la clave.

    ```Kusto
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.EVENTHUB"
    | where Category == "info" and OperationName == "disable" or OperationName == "restore"
    | project Message
    ```
+ Obtener errores de captura y su duración en segundos.

    ```kusto
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.EVENTHUB"
    | where Category == "ArchiveLogs"
    | summarize count() by "failures", "durationInSeconds"    
    ```
    
## <a name="alerts"></a>Alertas
Para acceder a las alertas de Azure Event Hubs, seleccione **Alertas** en la sección **Azure Monitor** de la página principal del espacio de nombres de Event Hubs. Vea [Creación, visualización y administración de alertas de métricas mediante Azure Monitor](../azure-monitor/alerts/alerts-metric.md) para más detalles sobre la creación de alertas.


## <a name="next-steps"></a>Pasos siguientes

- Para obtener una referencia de los registros y las métricas, consulte [Supervisión de referencia de datos de Azure Event Hubs](monitor-event-hubs-reference.md).
- Para más información sobre la supervisión de recursos de Azure, consulte [Supervisión de recursos de Azure con Azure Monitor](../azure-monitor/essentials/monitor-azure-resource.md).
