---
title: Supervisión de Azure Container Registry
description: Aprenda a supervisar Azure Container Registry mediante las características de Azure Monitor.
author: dlepow
ms.author: danlep
ms.topic: how-to
ms.custom: subject-monitoring
ms.service: container-registry
ms.date: 03/19/2021
ms.openlocfilehash: 21cc4aeb5b8a643fcee85cfd60e2958ed0c72c72
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112060973"
---
# <a name="monitor-azure-container-registry"></a>Supervisión de Azure Container Registry

Si tiene aplicaciones y procesos empresariales críticos que dependen de recursos de Azure, querrá supervisar esos recursos para su disponibilidad, rendimiento y funcionamiento. En este artículo se describen los datos de supervisión generados por Azure Container Registry y cómo puede usar las características de Azure Monitor para analizar estos datos y generar alertas basadas en ellos:

## <a name="monitor-overview"></a>Información general de supervisión

La página de **información general** de Azure Portal para cada registro incluye una breve vista del uso y la actividad recientes del recurso como, por ejemplo, las operaciones de inserción y extracción. Esta información de alto nivel es útil, pero solo se muestra una pequeña cantidad de datos de supervisión. 

:::image type="content" source="media/monitor-service/metrics-overview.png" alt-text="Introducción a las métricas del registro"::: 

## <a name="what-is-azure-monitor"></a>¿Qué es Azure Monitor?

Azure Container Registry crea datos de supervisión mediante [Azure Monitor](../azure-monitor/overview.md), que es un servicio de supervisión de pila completo de Azure, que proporciona un conjunto completo de características para supervisar los recursos de Azure, además de los recursos locales y en otras nubes.

Comience leyendo el artículo [Supervisión de recursos de Azure con Azure Monitor](../azure-monitor/essentials/monitor-azure-resource.md), en el que se describen los conceptos siguientes:

- ¿Qué es Azure Monitor?
- Costos asociados con la supervisión
- Datos de supervisión recopilados en Azure
- Configuración de la recolección de datos
- Herramientas estándar en Azure para analizar datos de supervisión y alertar sobre ellos

Las secciones siguientes se basan en este artículo, donde se describen los datos específicos recopilados para Azure Container Registry, y se proporcionan ejemplos para configurar la recopilación de datos y el análisis de estos datos con herramientas de Azure.

## <a name="monitoring-data"></a>Supervisión de datos 

Azure Container Registry recopila los mismos tipos de datos de supervisión que otros recursos de Azure que se describen en [Supervisión de datos de recursos de Azure](../azure-monitor/essentials/monitor-azure-resource.md#monitoring-data). 

Consulte [Referencia de supervisión de datos de Azure Container Registry](monitor-service-reference.md) para obtener información detallada sobre las métricas y los registros creados por Azure Container Registry.

## <a name="collection-and-routing"></a>Recopilación y enrutamiento

Las métricas de la plataforma y el registro de actividad se recopilan y almacenan de forma automática, pero se pueden enrutar a otras ubicaciones mediante una configuración de diagnóstico.  

Los registros de recursos no se recopilan ni almacenan hasta que se crea una configuración de diagnóstico y se enrutan a una o varias ubicaciones.

Consulte [Creación de una configuración de diagnóstico para recopilar registros de plataforma y métricas en Azure](../azure-monitor/essentials/diagnostic-settings.md) para ver el proceso detallado para crear una configuración de diagnóstico mediante el Azure Portal, la CLI o PowerShell. Cuando se crea una configuración de diagnóstico, se especifican las categorías de registros que se van a recopilar. Las categorías de Azure Container Registry se enumeran en la [referencia de datos de supervisión de Azure Container Registry](monitor-service-reference.md#resource-logs).

> [!TIP]
> Para crear la configuración de diagnóstico del registro puede ir también al registro en el portal. En el menú, seleccione **Configuración de diagnóstico** en **Supervisión**.

En la imagen siguiente se muestran las opciones que hay cuando habilita la configuración de diagnóstico de un registro.

:::image type="content" source="media/monitor-service/diagnostic-settings.png" alt-text="Configuración de diagnóstico de Container Registry":::

En las secciones siguientes se describen las métricas y los registros que se pueden recopilar.

## <a name="analyzing-metrics-preview"></a>Análisis de métricas (versión preliminar)

Puede analizar las métricas de Azure Container Registry con métricas de otros servicios de Azure mediante el explorador de métricas abriendo **Métricas** en el menú de **Azure Monitor**. Consulte [Introducción al explorador de métricas de Azure](../azure-monitor/essentials/metrics-getting-started.md) para más información sobre esta herramienta. 

> [!TIP]
> También puede ir al explorador de métricas. Para ello, vaya al registro en el portal. En el menú, seleccione **Métricas (versión preliminar)** en **Supervisión**.

Para obtener una lista de las métricas de plataforma recopiladas para Azure Container Registry, consulte [Supervisión de métricas de referencia de datos de Azure Container Registry](monitor-service-reference.md#metrics)  

Como referencia, puede ver una lista de [todas las métricas de recursos que se admiten en Azure Monitor](../azure-monitor/essentials/metrics-supported.md).

### <a name="azure-cli"></a>Azure CLI

Los siguientes comandos de la CLI de Azure se pueden usar para obtener información sobre las métricas de Azure Container Registry.

* [az acr show-usage](/cli/azure/acr/#az_acr_show_usage): muestra el almacenamiento actual que usa una instancia de Azure Container Registry
* [az monitor metrics list-definitions](/cli/azure/monitor/metrics#az_monitor_metrics_list_definitions): muestra las definiciones y dimensiones de las métricas
* [az monitor metrics list](/cli/azure/monitor/metrics#az_monitor_metrics_list): recupera los valores de las métricas

### <a name="rest-api"></a>API DE REST 

Puede usar la API REST de Azure Monitor para obtener información mediante programación sobre las métricas de Azure Container Registry.

* [Enumeración de las definiciones y dimensiones de las métricas](/rest/api/monitor/metricdefinitions/list)
* [Recuperación de los valores de métrica](/rest/api/monitor/metrics/list)

## <a name="analyzing-logs"></a>Análisis de datos

Los datos de los registros de Azure Monitor se almacenan en tablas, cada una con un conjunto propio de propiedades únicas.  

Todos los registros de recursos de Azure Monitor tienen los mismos campos seguidos de campos específicos del servicio. El esquema común se describe en [Esquema de registros de recursos de Azure Monitor](/azure/azure-monitor/essentials/resource-logs-schema#top-level-resource-logs-schema). El esquema de los registros de recursos de Azure Container Registry se encuentra en la [referencia de datos de Azure Container Registry](monitor-service-reference.md#schemas). 

El [registro de actividad](../azure-monitor/essentials/activity-log.md) es un registro de plataforma de Azure que proporciona conclusiones sobre los eventos del nivel de suscripción. Puede verlo de forma independiente o enrutarlo a registros de Azure Monitor, donde puede realizar consultas mucho más complejas mediante Log Analytics.  

Para obtener una lista de los tipos de recursos de registros recopilados para Azure Container Registry, consulte [Supervisión de la referencia de datos de Azure Container Registry](monitor-service-reference.md#resource-logs).  

Para obtener una lista de las tablas utilizadas por los registros de Azure Monitor y en la que Log Analytics puede realizar consultas, consulte [Supervisión de los datos de referencia de Azure Container Registry](monitor-service-reference.md#azure-monitor-logs-tables).  

### <a name="sample-kusto-queries"></a>Ejemplos de consultas de Kusto

> [!IMPORTANT]
> Al seleccionar **Registros** en el menú de **Azure Container Registry**, Log Analytics se abre con el ámbito de la consulta establecido en el registro actual. Esto significa que las consultas de registro solo incluirán datos de ese recurso. Si desea ejecutar una consulta que incluya datos de otros registros o de otros servicios de Azure, seleccione **Registros** en el menú **Azure Monitor**. Consulte [Ámbito e intervalo de tiempo de una consulta de registro en Log Analytics de Azure Monitor](/azure/azure-monitor/log-query/scope/) para obtener más información.

Por ejemplo, la consulta siguiente recupera las 24 horas de datos más recientes de la tabla **ContainerRegistryRepositoryEvents**:

```Kusto
ContainerRegistryRepositoryEvents
| where TimeGenerated > ago(1d) 
```

En la imagen siguiente se muestra la salida de ejemplo:

:::image type="content" source="media/monitor-service/azure-monitor-query.png" alt-text="Consulta de datos de registro":::

A continuación se muestran las consultas que puede usar para facilitar la supervisión del recurso de registro. 

### <a name="error-events-from-the-last-hour"></a>Eventos de error de la última hora

```Kusto
union Event, Syslog // Event table stores Windows event records, Syslog stores Linux records
| where TimeGenerated > ago(1h)
| where EventLevelName == "Error" // EventLevelName is used in the Event (Windows) records
    or SeverityLevel== "err" // SeverityLevel is used in Syslog (Linux) records
```

### <a name="100-most-recent-registry-events"></a>100 eventos de registro más recientes

```Kusto
ContainerRegistryRepositoryEvents
| union ContainerRegistryLoginEvents
| top 100 by TimeGenerated
| project TimeGenerated, LoginServer, OperationName, Identity, Repository, DurationMs, Region , ResultType
```

### <a name="identity-of-user-or-object-that-deleted-repository"></a>Identidad del usuario u objeto que eliminó el repositorio

```Kusto
ContainerRegistryRepositoryEvents
| where OperationName contains "Delete"
| project LoginServer, OperationName, Repository, Identity, CallerIpAddress
```

### <a name="identity-of-user-or-object-that-deleted-tag"></a>Identidad del usuario u objeto que eliminó la etiqueta

```Kusto
ContainerRegistryRepositoryEvents
| where OperationName contains "Untag"
| project LoginServer, OperationName, Repository, Tag, Identity, CallerIpAddress
```

### <a name="repository-level-operation-failures"></a>Errores de operación de nivel de repositorio

```kusto
ContainerRegistryRepositoryEvents 
| where ResultDescription contains "40"
| project TimeGenerated, OperationName, Repository, Tag, ResultDescription
```

### <a name="registry-authentication-failures"></a>Errores de autenticación del Registro

```kusto
ContainerRegistryLoginEvents 
| where ResultDescription != "200"
| project TimeGenerated, Identity, CallerIpAddress, ResultDescription
```


## <a name="alerts"></a>Alertas

Las alertas de Azure Monitor le informan de forma proactiva cuando se detectan condiciones importantes en los datos que se supervisan. Permiten identificar y solucionar las incidencias en el sistema antes de que los clientes puedan verlos. Puede establecer alertas en [métricas](../azure-monitor/alerts/alerts-metric-overview.md), [registros](../azure-monitor/alerts/alerts-unified-log.md) y el [registro de actividad](../azure-monitor/alerts/activity-log-alerts.md). Cada tipo de alerta tiene sus ventajas y desventajas.


<!-- only include next line if applications run on your service and work with App Insights. 

If you are creating or running an application which run on <*service*> [Azure Monitor Application Insights](../azure-monitor/overview.md#application-insights) may offer additional types of alerts.
-->

En la tabla siguiente se enumeran las reglas de alertas de métricas comunes y recomendadas para Azure Container Registry.

<!-- Fill in the table with metric and log alerts that would be valuable for your service. Change the format as necessary to make it more readable -->
| Tipo de alerta | Condición | Descripción  |
|:---|:---|:---|
| métrica | Señal: almacenamiento usado<br/>Operador: Mayor que<br/>Tipo de agregación: Promedio<br/>Valor umbral: 5 GB|  Alerta si el almacenamiento de registros utilizado supera un valor especificado.|
| | | |

### <a name="example-send-email-alert-when-registry-storage-used-exceeds-a-value"></a>Ejemplo: envío de una alerta por correo electrónico cuando el almacenamiento de registros utilizado supera un valor

1. En Azure Portal, vaya al registro.
1. Seleccione **Métricas (versión preliminar)** en **Supervisión**.
1. En el explorador de métricas, en **Métrica**, seleccione **Almacenamiento usado**.
1. Seleccione **Nueva regla de alertas**.
1. En **Ámbito**, confirme el recurso del registro para el que desea crear una regla de alerta.
1. En **Condición**, seleccione **Agregar condición**.
    1. En **Nombre de señal**, seleccione **Almacenamiento usado**.
    1. En **Período del gráfico**, seleccione **Durante las últimas 24 horas**.
    1. En **Lógica de alerta**, en **Valor del umbral**, seleccione un valor como *5*. En **Unidad**, seleccione un valor como *GB*.
    1. Acepte los valores predeterminados para la configuración restante y seleccione **Listo**.
1. En **Acciones**, seleccione **Adición de grupos de acciones** >  **+ Crear grupo de acciones**.
    1. Escriba los detalles del grupo de acciones.
    1. En la pestaña **Notificaciones**, seleccione **Correo electrónico/mensaje SMS/notificación push/voz** y escriba un destinatario como *admin@contoso.com* . Seleccione **Revisar + crear**.
1. Escriba un nombre y la descripción de la regla de alertas y seleccione el nivel de gravedad.
1. Seleccione **Crear regla de alertas**.

## <a name="next-steps"></a>Pasos siguientes

- Consulte [Supervisión de la referencia de datos de Azure Container Registry](monitor-service-reference.md) para obtener una referencia de las métricas, los registros y otros valores importantes creados por Azure Container Registry.
- Para más información sobre la supervisión de recursos de Azure, consulte [Supervisión de recursos de Azure con Azure Monitor](../azure-monitor/essentials/monitor-azure-resource.md).
