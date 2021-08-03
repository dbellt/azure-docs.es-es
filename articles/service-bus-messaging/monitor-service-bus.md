---
title: Supervisión de Azure Service Bus
description: Aprenda a usar Azure Monitor para ver, analizar y crear alertas sobre métricas de Azure Service Bus.
ms.topic: conceptual
ms.custom: subject-monitoring
ms.date: 05/18/2021
ms.openlocfilehash: 90930a5f76fd1aaccd7968febdf7dbdbd684ab28
ms.sourcegitcommit: a9f131fb59ac8dc2f7b5774de7aae9279d960d74
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110192261"
---
# <a name="monitor-azure-service-bus"></a>Supervisión de Azure Service Bus
Si tiene aplicaciones y procesos empresariales críticos que dependen de recursos de Azure, querrá supervisar esos recursos para su disponibilidad, rendimiento y funcionamiento. En este artículo se describen los datos de supervisión que ha generado Azure Service Bus y cómo analizar y generar alertas sobre estos datos con Azure Monitor.

## <a name="what-is-azure-monitor"></a>¿Qué es Azure Monitor?
Azure Service Bus crea datos de supervisión mediante [Azure Monitor](../azure-monitor/overview.md), que es un servicio de supervisión de pila completa de Azure. Azure Monitor proporciona un conjunto completo de características para supervisar los recursos de Azure. También puede supervisar los recursos de otras nubes y de forma local.

Comience leyendo el artículo [Supervisión de recursos de Azure con Azure Monitor](../azure-monitor/essentials/monitor-azure-resource.md), en el que se describen los conceptos siguientes:

- ¿Qué es Azure Monitor?
- Costos asociados con la supervisión
- Datos de supervisión recopilados en Azure
- Configuración de la recolección de datos
- Herramientas estándar en Azure para analizar datos de supervisión y alertar sobre ellos

En las secciones siguientes se desarrolla este artículo mediante la descripción de los datos específicos que se recopilan para Azure Service Bus. En estas secciones también se proporcionan ejemplos para configurar la recopilación y el análisis de datos con Azure Tools.

> [!TIP]
> Para comprender los costos asociados a Azure Monitor, consulte [Uso y costos estimados](../azure-monitor//usage-estimated-costs.md). Para comprender el tiempo necesario para que los datos aparezcan en Azure Monitor, consulte [Tiempo de ingesta de datos de registro](../azure-monitor/logs/data-ingestion-time.md).

## <a name="monitoring-data-from-azure-service-bus"></a>Supervisión de datos de Azure Service Bus
Azure Service Bus recopila los mismos tipos de datos de supervisión que otros recursos de Azure que se describen en [Supervisión de datos de recursos de Azure](../azure-monitor/essentials/monitor-azure-resource.md#monitoring-data). 

Consulte [Referencia de datos de supervisión de Azure Service Bus](monitor-service-bus-reference.md) para encontrar una referencia detallada de los registros y las métricas creados por Azure Service Bus.

## <a name="collection-and-routing"></a>Recopilación y enrutamiento
Tanto las métricas de la plataforma como el registro de actividad se recopilan y almacenan de forma automática, pero se pueden enrutar a otras ubicaciones mediante una configuración de diagnóstico.  

Los registros de recursos no se recopilan ni almacenan hasta que se crea una configuración de diagnóstico y se enrutan a una o varias ubicaciones.

Consulte [Creación de una configuración de diagnóstico para recopilar registros de plataforma y métricas en Azure](../azure-monitor/essentials/diagnostic-settings.md) para ver el proceso detallado para crear una configuración de diagnóstico mediante el Azure Portal, la CLI o PowerShell. Cuando se crea una configuración de diagnóstico, se especifican las categorías de registros que se van a recopilar. Las categorías de Azure Service Bus se enumeran en [Referencia de datos de supervisión de Azure Service Bus](monitor-service-bus-reference.md#resource-logs).

### <a name="azure-storage"></a>Azure Storage 
La información de registro de diagnóstico se almacena en los contenedores **insights-logs-operationlogs** e **insights-metrics-pt1m**.

Dirección URL de ejemplo para un registro de operaciones: `https://<Azure Storage account>.blob.core.windows.net/insights-logs-operationallogs/resourceId=/SUBSCRIPTIONS/<Azure subscription ID>/RESOURCEGROUPS/<Resource group name>/PROVIDERS/MICROSOFT.SERVICEBUS/NAMESPACES/<Namespace name>/y=<YEAR>/m=<MONTH-NUMBER>/d=<DAY-NUMBER>/h=<HOUR>/m=<MINUTE>/PT1H.json`. La dirección URL de un registro de métricas es similar. 

### <a name="azure-event-hubs"></a>Azure Event Hubs
La información de registro de diagnóstico se almacena en los centros de eventos **insights-logs-operationlogs** e **insights-metrics-pt1m**. También puede seleccionar su propio centro de eventos. 

### <a name="log-analytics"></a>Log Analytics 
La información de registro de diagnóstico se almacena en las tablas **AzureDiagnostics** y **AzureMetrics**. 

### <a name="sample-operational-log-output-formatted"></a>Salida de registros operativos de ejemplo (con formato)

```json
{
    "Environment": "PROD",
    "Region": "East US",
    "ScaleUnit": "PROD-BL2-002",
    "ActivityId": "a097a88a-33e5-4c9c-9c64-20f506ec1375",
    "EventName": "Retrieve Namespace",
    "resourceId": "/SUBSCRIPTIONS/<Azure subscription ID>/RESOURCEGROUPS/SPSBUS0213RG/PROVIDERS/MICROSOFT.SERVICEBUS/NAMESPACES/SPSBUS0213NS",
    "SubscriptionId": "<Azure subscription ID>",
    "EventTimeString": "5/18/2021 3:25:55 AM +00:00",
    "EventProperties": "{\"SubscriptionId\":\"<Azure subscription ID>\",\"Namespace\":\"spsbus0213ns\",\"Via\":\"https://spsbus0213ns.servicebus.windows.net/$Resources/topics?api-version=2017-04&$skip=0&$top=100\",\"TrackingId\":\"a097a88a-33e5-4c9c-9c64-20f506ec1375_M8CH3_M8CH3_G8\"}",
    "Status": "Succeeded",
    "Caller": "rpfrontdoor",
    "category": "OperationalLogs"
}
```

### <a name="sample-metric-log-output-formatted"></a>Salida de registros de métricas de ejemplo (con formato)

```json
{
    "count": 1,
    "total": 4,
    "minimum": 4,
    "maximum": 4,
    "average": 4,
    "resourceId": "/SUBSCRIPTIONS/<Azure subscription ID>/RESOURCEGROUPS/SPSBUS0213RG/PROVIDERS/MICROSOFT.SERVICEBUS/NAMESPACES/SPSBUS0213NS",
    "time": "2021-05-18T03:27:00.0000000Z",
    "metricName": "IncomingMessages",
    "timeGrain": "PT1M"
}
```

> [!IMPORTANT]
> La habilitación de esta configuración requiere servicios adicionales de Azure (cuenta de almacenamiento, centro de eventos o Log Analytics), lo que puede aumentar el costo. Para calcular un costo estimado, visite la [Calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator).

> [!NOTE]
> Cuando se habilitan las métricas en una configuración de diagnóstico, la información de dimensión no se incluye actualmente como parte de la información que se envía a una cuenta de almacenamiento, a un centro de eventos o al análisis de registros.

En las secciones siguientes se describen las métricas y los registros que se pueden recopilar.

## <a name="analyzing-metrics"></a>Análisis de métricas
Puede analizar las métricas de Azure Service Bus, junto con las métricas de otros servicios de Azure, seleccionando **Métricas** en la sección **Azure Monitor** de la página principal del espacio de nombres de Service Bus. Consulte [Introducción al explorador de métricas de Azure](../azure-monitor/essentials/metrics-getting-started.md) para más información sobre esta herramienta. Para ver una lista de las métricas de la plataforma recopiladas, consulte [Supervisión de las métricas de referencia de datos de Azure Service Bus](monitor-service-bus-reference.md#metrics).

![Explorador de métricas con el espacio de nombres de Service Bus seleccionado](./media/monitor-service-bus/metrics.png)

Como referencia, puede ver una lista de [todas las métricas de recursos que se admiten en Azure Monitor](../azure-monitor/essentials/metrics-supported.md).

> [!TIP]
> Los datos de métricas de Azure Monitor están disponibles durante 90 días. Sin embargo, cuando se crean gráficos, solo se pueden visualizar durante 30 días. Por ejemplo, si quiere visualizar un período de 90 días, debe dividirlo en tres gráficos de 30 días dentro de ese período.

### <a name="filtering-and-splitting"></a>Filtrar y dividir
En el caso de las métricas que admiten dimensiones, puede aplicar filtros con un valor de dimensión. Por ejemplo, agregue un filtro con `EntityName` establecido en el nombre de una cola o un tema. Puede dividir una métrica por dimensión para visualizar la comparación de unos segmentos de la métrica con otros. Para más información sobre el filtrado y la división, vea [Características avanzadas de Azure Monitor](../azure-monitor/essentials/metrics-charts.md).

## <a name="analyzing-logs"></a>Análisis de datos
El uso de Log Analytics de Azure Monitor requiere la creación de una configuración de diagnóstico y habilitar __Enviar información a Log Analytics__. Para obtener más información, consulte la sección [Recolección y enrutamiento](#collection-and-routing). Los datos de los registros de Azure Monitor se almacenan en tablas, cada tabla tiene su propio conjunto de propiedades únicas. Azure Service Bus alacena los datos en las tablas **AzureDiagnostics** y **AzureMetrics**.

> [!IMPORTANT]
> Al seleccionar **Registros** en el menú de Azure Service Bus, se abre Log Analytics con el ámbito de la consulta establecido en el área de trabajo actual. Esto significa que las consultas de registro solo incluirán datos de ese recurso. Si desea ejecutar una consulta que incluya datos de otras bases de datos o de otros servicios de Azure, elija **Registros** en el menú **Azure Monitor**. Consulte [Ámbito e intervalo de tiempo de una consulta de registro en Log Analytics de Azure Monitor](../azure-monitor/logs/scope.md) para obtener más información.

Para encontrar una referencia detallada de los registros y las métricas, consulte [Referencia de datos de supervisión de Azure Service Bus](monitor-service-bus-reference.md).

### <a name="sample-kusto-queries"></a>Ejemplos de consultas de Kusto

> [!IMPORTANT]
> Al seleccionar **Registros** en el menú de Azure Service Bus, se abre Log Analytics con el ámbito de la consulta establecido en el espacio de nombres actual de Azure Service Bus. Esto significa que las consultas de registro solo incluirán datos de ese recurso. Si quiere ejecutar una consulta que incluya datos de otras áreas de trabajo o de otros servicios de Azure, seleccione **Registros** en el menú de **Azure Monitor**. Consulte [Ámbito e intervalo de tiempo de una consulta de registro en Log Analytics de Azure Monitor](../azure-monitor/logs/scope.md) para obtener más información.

A continuación, se muestran las consultas de ejemplo que se pueden usar para supervisar los recursos de Azure Service Bus: 

+ Obtener las operaciones de administración de los últimos 7 días. 

    ```Kusto
    AzureDiagnostics
    | where TimeGenerated > ago(7d)
    | where ResourceProvider =="MICROSOFT.SERVICEBUS"
    | where Category == "OperationalLogs"
    | summarize count() by EventName_s, _ResourceId
    ```

+ Obtener los intentos de acceso a un almacén de claves que provocaron el error "No se encontró la clave".

    ```Kusto
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.SERVICEBUS" 
    | where Category == "Error" and OperationName == "wrapkey"
    | project Message, _ResourceId
    ```

+ Obtener errores de los últimos 7 días.

    ```Kusto
    AzureDiagnostics
    | where TimeGenerated > ago(7d)
    | where ResourceProvider =="MICROSOFT.SERVICEBUS"
    | where Category == "Error" 
    | summarize count() by EventName_s, _ResourceId
    ```

+ Obtener las operaciones realizadas con un almacén de claves para deshabilitar o restaurar la clave.

    ```Kusto
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.SERVICEBUS"
    | where (Category == "info" and (OperationName == "disable" or OperationName == "restore"))
    | project Message, _ResourceId
    ```

+ Obtener todas las entidades que se han eliminado automáticamente.

    ```kusto
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.SERVICEBUS"
    | where Category == "OperationalLogs"
    | where EventName_s startswith "AutoDelete"
    | summarize count() by EventName_s, _ResourceId    
    ```
    
## <a name="alerts"></a>Alertas
Para acceder a las alertas de Azure Service Bus, seleccione **Alertas** en la sección **Azure Monitor** de la página principal del espacio de nombres de Service Bus. Vea [Creación, visualización y administración de alertas de métricas mediante Azure Monitor](../azure-monitor/alerts/alerts-metric.md) para más detalles sobre la creación de alertas.


## <a name="next-steps"></a>Pasos siguientes

- Para obtener una referencia de los registros y las métricas, consulte [Supervisión de referencia de datos de Azure Service Bus](monitor-service-bus-reference.md).
- Para más información sobre la supervisión de recursos de Azure, consulte [Supervisión de recursos de Azure con Azure Monitor](../azure-monitor/essentials/monitor-azure-resource.md).
