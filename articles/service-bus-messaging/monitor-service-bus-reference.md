---
title: Referencia de datos para la supervisión de Azure Service Bus
description: Material de referencia importante necesario al supervisar Azure Service Bus.
ms.topic: reference
ms.custom: subject-monitoring
ms.date: 05/18/2021
ms.openlocfilehash: a32972b6400831a4045082fa5d0255b06ed66210
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112061297"
---
# <a name="monitoring-azure-service-bus-data-reference"></a>Referencia de datos para la supervisión de Azure Service Bus
Consulte [Supervisión de Azure Service Bus](monitor-service-bus.md) para más información sobre la recopilación y el análisis de datos de supervisión de Azure Service Bus.

## <a name="metrics"></a>Métricas
En esta sección se indican todas las métricas de la plataforma recopiladas automáticamente para Azure Service Bus. El proveedor de recursos para estas métricas es **Microsoft.ServiceBus/namespaces.**

### <a name="request-metrics"></a>Solicitud de métricas
Cuenta el número de solicitudes de operaciones de datos y administración.

| Nombre de la métrica |  Se puede exportar con la configuración de diagnóstico. | Unidad | Tipo de agregación |  Descripción | Dimensions | 
| ---------- | ---------- | ----- | --- | --- | --- | 
| Solicitudes entrantes| Sí | Count | Total | Número de solicitudes realizadas al servicio de Service Bus durante un período determinado. | EntityName | 
|Solicitudes correctas| No | Count | Total | Número de solicitudes correctas realizadas al servicio de Service Bus durante un período determinado. | Nombre de entidad<br/>OperationResult|
|Errores del servidor| No | Count | Total | Número de solicitudes no procesadas debido a un error en el servicio de Service Bus durante un período determinado. | Nombre de entidad<br/>OperationResult|
|Errores de usuario | No | Count | Total | Número de solicitudes no procesadas debido a errores de usuario durante un período determinado. | Nombre de entidad|
|Solicitudes limitadas| No | Count | Total | Número de solicitudes que se han limitado porque se superó el uso. |  Nombre de entidad|

Los dos tipos de errores siguientes se clasifican como **errores de usuario**:

1. Errores en el cliente (en HTTP, serían errores 400).
2. Errores que se producen al procesar los mensajes, como [MessageLockLostException](/dotnet/api/microsoft.azure.servicebus.messagelocklostexception).


### <a name="message-metrics"></a>Métricas de mensaje

| Nombre de la métrica |  Se puede exportar con la configuración de diagnóstico. | Unidad | Tipo de agregación |  Descripción | Dimensions | 
| ---------- | ---------- | ----- | --- | --- | --- | 
|Mensajes entrantes| Sí | Count | Total | Número de eventos o mensajes enviados a Service Bus durante un período determinado. Esta métrica no incluye mensajes que se reenvían automáticamente. | Nombre de entidad|
|Mensajes salientes| Sí | Count | Total | Número de eventos o mensajes recibidos de Service Bus durante un período determinado. | Nombre de entidad|
| error de Hadoop| No | Count | Average | Recuento de mensajes de una cola/tema. | Nombre de entidad |
| Mensajes activos| No | Count | Average | Recuento de mensajes activos de una cola/tema. | Nombre de entidad |
| Mensajes fallidos| No | Count | Average | Recuento de mensajes fallidos de una cola/tema.  | Nombre de entidad |
| Mensajes programados| No | Count | Average | Recuento de mensajes programados de una cola/tema. | Nombre de entidad |
| Size | No | Bytes | Average | Tamaño de una entidad (cola o tema) en bytes. | Nombre de entidad | 

> [!NOTE]
> Los valores de mensajes, activos, fallidos, programados, completados y abandonados son valores de un momento dado. Es posible que los mensajes entrantes que se consumieron inmediatamente después de ese momento dado no se reflejen en estas métricas. 

### <a name="connection-metrics"></a>Métricas de conexión

| Nombre de la métrica |  Se puede exportar con la configuración de diagnóstico. | Unidad | Tipo de agregación |  Descripción | Dimensions | 
| ---------- | ---------- | ----- | --- | --- | --- | 
|Conexiones activas| No | Count | Total | Número de conexiones activas en un espacio de nombres y en una entidad en el espacio de nombres. El valor de esta métrica es un valor de un momento dado. Es posible que las conexiones que estaban activas inmediatamente después de ese momento dado no se reflejen en esta métrica. | |
|Conexiones abiertas | No | Count | Average | Número de conexiones abiertas. | Nombre de entidad|
|Conexiones cerradas | No | Count | Average | Número de conexiones cerradas. | Nombre de entidad|

### <a name="resource-usage-metrics"></a>Métricas de uso de recursos

> [!NOTE] 
> Estas métricas están disponibles solo con el nivel **Premium**. 
> 
> Estas son las métricas importantes que supervisar en busca de posibles interrupciones de un espacio de nombres de nivel Premium: **Uso de CPU por espacio de nombres** y **Uso del tamaño de memoria por espacio de nombres**. [Configure alertas](../azure-monitor/alerts/alerts-metric.md) para estas métricas usando Azure Monitor.
> 
> La otra métrica que se podría supervisar es: **Solicitudes limitadas**. Con todo, esto no debería ser un problema siempre y cuando el espacio de nombres permanezca dentro de sus límites de memoria, CPU y conexiones desacopladas correspondientes. Para más información, vea [Limitación en el nivel Prémium de Azure Service Bus](service-bus-throttling.md#throttling-in-azure-service-bus-premium-tier).

| Nombre de la métrica |  Se puede exportar con la configuración de diagnóstico. | Unidad | Tipo de agregación |  Descripción | Dimensions | 
| ---------- | ---------- | ----- | --- | --- | --- | 
|Uso de CPU por espacio de nombres| No | CPU | Percent | Porcentaje de uso de CPU del espacio de nombres. | Réplica |
|Uso de tamaño de memoria por espacio de nombres| No | Uso de la memoria | Percent | Porcentaje de uso de memoria del espacio de nombres. | Réplica |

### <a name="error-metrics"></a>Métricas de error
| Nombre de la métrica |  Se puede exportar con la configuración de diagnóstico. | Unidad | Tipo de agregación |  Descripción | Dimensions |
| ------------------- | ----------------- | --- | --- | --- | --- | 
|Errores del servidor| No | Count | Total | Número de solicitudes no procesadas debido a un error en el servicio de Service Bus durante un período determinado. | Nombre de entidad<br/><br/>Resultado de la operación |
|Errores de usuario | No | Count | Total | Número de solicitudes no procesadas debido a errores de usuario durante un período determinado. | Nombre de entidad<br/><br/>Resultado de la operación|

## <a name="metric-dimensions"></a>Dimensiones de métricas

Azure Service Bus admite las siguientes dimensiones para las métricas de Azure Monitor. La adición de dimensiones a las métricas es opcional. Si no agrega dimensiones, las métricas se especifican en el nivel de espacio de nombres. 

|Nombre de dimensión|Descripción|
| ------------------- | ----------------- |
|Nombre de entidad| Service Bus admite las entidades de mensajería en el espacio de nombres.|

## <a name="resource-logs"></a>Registros del recurso
En esta sección se enumeran los tipos de registros de recursos que se pueden recopilar para Azure Service Bus.

- Registros operativos
- Registros de filtrado de IP y red virtual

### <a name="operational-logs"></a>Registros operativos
Las entradas de registros operativos incluyen los elementos que aparecen en la tabla siguiente:

| Nombre | Descripción |
| ------- | ------- |
| ActivityId | Identificador interno que se usa para identificar la actividad especificada. |
| EventName | Nombre de la operación |
| ResourceId | El identificador de recursos de Azure Resource Manager |
| SubscriptionId | Id. de suscripción |
| EventTimeString | Hora de la operación |
| EventProperties | Propiedades de la operación |
| Estado | Estado de la operación |
| Autor de llamada | Autor de la llamada de la operación (Azure Portal o Management Client) |
| Category | OperationalLogs |

Este es un ejemplo de una cadena JSON de registro operativo:

```json
{
  "ActivityId": "0000000000-0000-0000-0000-00000000000000",
  "EventName": "Create Queue",
  "resourceId": "/SUBSCRIPTIONS/<AZURE SUBSCRPTION ID>/RESOURCEGROUPS/<RESOURCE GROUP NAME>/PROVIDERS/MICROSOFT.SERVICEBUS/NAMESPACES/<SERVICE BUS NAMESPACE NAME>",
  "SubscriptionId": "0000000000-0000-0000-0000-00000000000000",
  "EventTimeString": "9/28/2016 8:40:06 PM +00:00",
  "EventProperties": "{\"SubscriptionId\":\"0000000000-0000-0000-0000-00000000000000\",\"Namespace\":\"mynamespace\",\"Via\":\"https://mynamespace.servicebus.windows.net/f8096791adb448579ee83d30e006a13e/?api-version=2016-07\",\"TrackingId\":\"5ee74c9e-72b5-4e98-97c4-08a62e56e221_G1\"}",
  "Status": "Succeeded",
  "Caller": "ServiceBus Client",
  "category": "OperationalLogs"
}
```

### <a name="events-and-operations-captured-in-operational-logs"></a>Eventos y operaciones capturados en registros operativos
Los registros operativos capturan todas las operaciones de administración que se realizan en el espacio de nombres de Azure Service Bus. Las operaciones de datos no se capturan debido al elevado volumen de este tipo de operaciones que se realizan en Azure Service Bus.

> [!NOTE]
> Para que sea más fácil hacer un seguimiento de las operaciones de datos, se recomienda usar el seguimiento del lado cliente.

Estas operaciones de administración se capturan en los registros operativos: 

| Ámbito | Operación|
|-------| -------- |
| Espacio de nombres | <ul> <li> Crear espacio de nombres</li> <li> Actualizar espacio de nombres </li> <li> Eliminar espacio de nombres </li> <li> Actualizar la directiva SharedAccess del espacio de nombres </li> </ul> | 
| Cola | <ul> <li> Crear cola</li> <li> Actualizar cola</li> <li> Eliminar cola </li> <li> Eliminar cola automáticamente (AutoDelete) </li> </ul> | 
| Tema | <ul> <li> Crear tema </li> <li> Actualizar tema </li> <li> Eliminar tema </li> <li> Eliminar tema automáticamente (AutoDelete) </li> </ul> |
| Subscription | <ul> <li> Crear suscripción </li> <li> Actualizar suscripción </li> <li> Eliminar suscripción </li> <li> Eliminar suscripción automáticamente (AutoDelete) </li> </ul> |

> [!NOTE]
> Actualmente, no se realiza un seguimiento de las operaciones de *lectura* en los registros operativos.

## <a name="azure-monitor-logs-tables"></a>Tablas de registros de Azure Monitor
Azure Service Bus usa tablas de Kusto de registros de Azure Monitor. Puede consultar estas tablas con Log Analytics. Para ver una lista de las tablas de Kusto que usa el servicio, consulte [Referencia de las tablas de Azure Monitor](/azure/azure-monitor/reference/tables/tables-resourcetype#service-bus).


## <a name="next-steps"></a>Pasos siguientes
- Para más información sobre la supervisión de Azure Service Bus, consulte [Supervisión Azure Service Bus](monitor-service-bus.md).
- Para más información sobre la supervisión de recursos de Azure, consulte [Supervisión de recursos de Azure con Azure Monitor](../azure-monitor/essentials/monitor-azure-resource.md).
