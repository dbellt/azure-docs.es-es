---
title: archivo de inclusión
description: archivo de inclusión
services: event-hubs
author: spelluru
ms.service: event-hubs
ms.topic: include
ms.date: 06/11/2021
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: aa3c541db838f9b33208dba7ba9ac33422d3b1a5
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112060817"
---
Event Hubs captura los registros de diagnóstico de las siguientes categorías:

| Category | Descripción | 
| -------- | ----------- | 
| Registros de archivo | Capturan información sobre las operaciones de [Event Hubs Capture](../articles/event-hubs/event-hubs-capture-overview.md), en concreto, registros relacionados con errores de captura. |
| Registros operativos | Capturan todas las operaciones de administración que se realizan en el espacio de nombres de Azure Event Hubs. Las operaciones de datos no se capturan debido al elevado volumen de este tipo de operaciones que se realizan en Azure Event Hubs. |
| Registros de escalabilidad automática | Capturan las operaciones de inflado automático realizadas en un espacio de nombres de Event Hubs. |
| Registros de coordinador de Kafka | Capturan las operaciones del coordinador de Kafka relacionadas con Event Hubs. |
| Registros de errores de usuario de Kafka | Capturan información sobre las API de Kafka llamadas en Event Hubs. |
| Evento de conexión de red virtual de Event Hubs | Captura información sobre las direcciones IP y las redes virtuales que envían tráfico a Event Hubs. |
| Registros de usuario de claves administradas por el cliente | Capturan las operaciones relacionadas con las claves administradas por el cliente. |


Todos los registros se almacenan en el formato de notación de objetos JavaScript (JSON). Cada entrada tiene campos de cadena que usan el formato descrito en las secciones siguientes.

### <a name="archive-logs-schema"></a>Esquema de registros de archivo

Las cadenas JSON de registros de archivo incluyen elementos enumerados en la tabla siguiente:

Nombre | Descripción
------- | -------
`TaskName` | La descripción de la tarea que produjo error
`ActivityId` | El identificador interno, usado con fines de seguimiento
`trackingId` | El identificador interno, usado con fines de seguimiento
`resourceId` | El identificador de recursos de Azure Resource Manager
`eventHub` | Nombre completo del centro de eventos (incluye el nombre del espacio de nombres)
`partitionId` | La partición del centro de eventos en la que se escribe.
`archiveStep` | Valores posibles: ArchiveFlushWriter, DestinationInit
`startTime` | Hora de inicio del error
`failures` | Número de veces que se ha producido el error.
`durationInSeconds` | La duración del error
`message` | Mensaje de error
`category` | ArchiveLogs

El código siguiente es un ejemplo de una cadena JSON de registro de archivo:

```json
{
   "TaskName": "EventHubArchiveUserError",
   "ActivityId": "000000000-0000-0000-0000-0000000000000",
   "trackingId": "0000000-0000-0000-0000-00000000000000000",
   "resourceId": "/SUBSCRIPTIONS/000000000-0000-0000-0000-0000000000000/RESOURCEGROUPS/<Resource Group Name>/PROVIDERS/MICROSOFT.EVENTHUB/NAMESPACES/<Event Hubs Namespace Name>",
   "eventHub": "<Event Hub full name>",
   "partitionId": "1",
   "archiveStep": "ArchiveFlushWriter",
   "startTime": "9/22/2016 5:11:21 AM",
   "failures": 3,
   "durationInSeconds": 360,
   "message": "Microsoft.WindowsAzure.Storage.StorageException: The remote server returned an error: (404) Not Found. ---> System.Net.WebException: The remote server returned an error: (404) Not Found.\r\n   at Microsoft.WindowsAzure.Storage.Shared.Protocol.HttpResponseParsers.ProcessExpectedStatusCodeNoException[T](HttpStatusCode expectedStatusCode, HttpStatusCode actualStatusCode, T retVal, StorageCommandBase`1 cmd, Exception ex)\r\n   at Microsoft.WindowsAzure.Storage.Blob.CloudBlockBlob.<PutBlockImpl>b__3e(RESTCommand`1 cmd, HttpWebResponse resp, Exception ex, OperationContext ctx)\r\n   at Microsoft.WindowsAzure.Storage.Core.Executor.Executor.EndGetResponse[T](IAsyncResult getResponseResult)\r\n   --- End of inner exception stack trace ---\r\n   at Microsoft.WindowsAzure.Storage.Core.Util.StorageAsyncResult`1.End()\r\n   at Microsoft.WindowsAzure.Storage.Core.Util.AsyncExtensions.<>c__DisplayClass4.<CreateCallbackVoid>b__3(IAsyncResult ar)\r\n--- End of stack trace from previous location where exception was thrown ---\r\n   at System.",
   "category": "ArchiveLogs"
}
```

### <a name="operational-logs-schema"></a>Esquema de registros operativos

Las cadenas JSON de registros operativos incluyen elementos enumerados en la tabla siguiente:

Nombre | Descripción
------- | -------
`ActivityId` | Identificador interno, usado con fines de seguimiento. |
`EventName` | Nombre de la operación. Para consultar una lista de los valores de este elemento, consulte [Nombres de evento.](#event-names) |
`resourceId` | El identificador de recursos de Azure Resource Manager |
`SubscriptionId` | Id. de suscripción |
`EventTimeString` | Hora de la operación |
`EventProperties` |Propiedades de la operación. Este elemento proporciona más información sobre el evento, como se muestra en el ejemplo siguiente. |
`Status` | Estado de la operación. El valor puede ser **Succeeded** o **Failed**.  |
`Caller` | Autor de la llamada de la operación (Azure Portal o Management Client) |
`Category` | OperationalLogs |

El código siguiente es un ejemplo de una cadena JSON de registro operativo:

```json
Example:
{
   "ActivityId": "00000000-0000-0000-0000-00000000000000",
   "EventName": "Create EventHub",
   "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-0000000000000/RESOURCEGROUPS/<Resource Group Name>/PROVIDERS/MICROSOFT.EVENTHUB/NAMESPACES/<Event Hubs namespace name>",
   "SubscriptionId": "000000000-0000-0000-0000-000000000000",
   "EventTimeString": "9/28/2016 8:40:06 PM +00:00",
   "EventProperties": "{\"SubscriptionId\":\"0000000000-0000-0000-0000-000000000000\",\"Namespace\":\"<Namespace Name>\",\"Via\":\"https://<Namespace Name>.servicebus.windows.net/f8096791adb448579ee83d30e006a13e/?api-version=2016-07\",\"TrackingId\":\"5ee74c9e-72b5-4e98-97c4-08a62e56e221_G1\"}",
   "Status": "Succeeded",
   "Caller": "ServiceBus Client",
   "category": "OperationalLogs"
}
```

#### <a name="event-names"></a>Nombres de eventos
El nombre del evento se rellena como el tipo de operación y el tipo de recurso de las siguientes enumeraciones. Por ejemplo, `Create Queue`, `Retrieve Event Hu` o `Delete Rule`. 

| Tipo de operación | Tipo de recurso | 
| -------------- | ------------- | 
| <ul><li>Crear</li><li>Actualizar</li><li>Eliminar</li><li>Recuperar</li><li>Unknown</li></ul> | <ul><li>Espacio de nombres</li><li>Cola</li><li>Tema</li><li>Subscription</li><li>EventHub</li><li>EventHubSubscription</li><li>NotificationHub</li><li>NotificationHubTier</li><li>SharedAccessPolicy</li><li>UsageCredit</li><li>NamespacePnsCredentials</li>Regla</li>ConsumerGroup</li> |

### <a name="autoscale-logs-schema"></a>Esquema de registros de escalabilidad automática
Las cadenas JSON del registro de escalabilidad automática incluyen los elementos enumerados en la tabla siguiente:

| Nombre | Descripción |
| ---- | ----------- | 
| `TrackingId` | Identificador interno, que se usa con fines de seguimiento. |
| `ResourceId` | Identificador de recursos de Azure Resource Manager. |
| `Message` | Mensaje informativo, que proporciona detalles sobre la acción de inflado automático. El mensaje contiene el valor anterior y actual de la unidad de procesamiento de un espacio de nombres determinado y lo que desencadenó el inflado de la unidad de rendimiento. |

Este es un evento de escalado automático de ejemplo: 

```json
{
    "TrackingId": "fb1b3676-bb2d-4b17-85b7-be1c7aa1967e",
    "Message": "Scaled-up EventHub TUs (UpdateStartTimeUTC: 5/13/2020 7:48:36 AM, PreviousValue: 1, UpdatedThroughputUnitValue: 2, AutoScaleReason: 'IncomingMessagesPerSecond reached 2170')",
    "ResourceId": "/subscriptions/0000000-0000-0000-0000-000000000000/resourcegroups/testrg/providers/microsoft.eventhub/namespaces/namespace-name"
}
```

### <a name="kafka-coordinator-logs-schema"></a>Esquema de registros de coordinador de Kafka
Las cadenas JSON del registro de coordinador de Kafka incluyen los elementos enumerados en la tabla siguiente:

| Nombre | Descripción |
| ---- | ----------- | 
| `RequestId` | Identificador de solicitud, que se usa con fines de seguimiento. |
| `ResourceId` | El identificador de recursos de Azure Resource Manager |
| `Operation` | Nombre de la operación que se realiza durante la coordinación de grupos. |
| `ClientId` | Id. de cliente |
| `NamespaceName` | Nombre del espacio de nombres | 
| `SubscriptionId` | Identificador de suscripción de Azure |
| `Message` | Mensaje informativo o de advertencia, que proporciona detalles sobre las acciones realizadas durante la coordinación de grupos. |

#### <a name="example"></a>Ejemplo

```json
{
    "RequestId": "FE01001A89E30B020000000304620E2A_KafkaExampleConsumer#0",
    "Operation": "Join.Start",
    "ClientId": "KafkaExampleConsumer#0",
    "Message": "Start join group for new member namespace-name:c:$default:I:KafkaExampleConsumer#0-cc40856f7f3c4607915a571efe994e82, current group size: 0, API version: 2, session timeout: 10000ms, rebalance timeout: 300000ms.",
    "SubscriptionId": "0000000-0000-0000-0000-000000000000",
    "NamespaceName": "namespace-name",
    "ResourceId": "/subscriptions/0000000-0000-0000-0000-000000000000/resourcegroups/testrg/providers/microsoft.eventhub/namespaces/namespace-name",
    "Category": "KafkaCoordinatorLogs"
}
```

### <a name="kafka-user-error-logs-schema"></a>Esquema de registros de error de usuario de Kafka
Las cadenas JSON del registro de errores de Kafka incluyen los elementos enumerados en la tabla siguiente:

| Nombre | Descripción |
| ---- | ----------- |
| `TrackingId` | Identificador de seguimiento, que se usa con fines de seguimiento. |
| `NamespaceName` | Nombre del espacio de nombres |
| `Eventhub` | Nombre del centro de eventos |
| `PartitionId` | Id. de partición |
| `GroupId` | Identificador de grupo |
| `ClientId` | Id. de cliente |
| `ResourceId` | Identificador de recursos de Azure Resource Manager. |
| `Message` | Mensaje informativo, que proporciona detalles sobre un error. |

### <a name="event-hubs-virtual-network-connection-event-schema"></a>Esquema de eventos de conexión de red virtual de Event Hubs
Las cadenas JSON del evento de conexión de red virtual (VNet) de Event Hubs contienen los elementos enumerados en la tabla siguiente:

| Nombre | Descripción |
| ---  | ----------- | 
| `SubscriptionId` | Identificador de suscripción de Azure |
| `NamespaceName` | Nombre del espacio de nombres |
| `IPAddress` | Dirección IP de un cliente que se conecta al servicio de Event Hubs. |
| `Action` | Acción realizada por el servicio Event Hubs al evaluar las solicitudes de conexión. Las acciones admitidas son **Accept Connection** (Aceptar conexión) y **Deny Connection** (Denegar conexión). |
| `Reason` | Proporciona el motivo de que se haya realizado la acción. |
| `Count` | Número de repeticiones de una acción dada. |
| `ResourceId` | Identificador de recursos de Azure Resource Manager. |

Los registros de red virtual solo se generan si el espacio de nombres permite el acceso desde **redes seleccionadas** o desde **direcciones IP específicas** (reglas de filtro de IP). Si no quiere restringir el acceso al espacio de nombres mediante estas características y quiere obtener registros de red virtual para realizar el seguimiento de las direcciones IP de los clientes que se conectan al espacio de nombres de Event Hubs, puede usar la siguiente alternativa. [Habilite el filtrado de IP](../articles/event-hubs/event-hubs-ip-filtering.md) y agregue el intervalo IPv4 direccionable total (1.0.0.0/1 - 255.0.0.0/1). El filtrado de IP de Event Hubs no admite intervalos IPv6. Tenga en cuenta que puede ver las direcciones de un punto de conexión privado en formato IPv6 en el registro. 

#### <a name="example"></a>Ejemplo

```json
{
    "SubscriptionId": "0000000-0000-0000-0000-000000000000",
    "NamespaceName": "namespace-name",
    "IPAddress": "1.2.3.4",
    "Action": "Deny Connection",
    "Reason": "IPAddress doesn't belong to a subnet with Service Endpoint enabled.",
    "Count": "65",
    "ResourceId": "/subscriptions/0000000-0000-0000-0000-000000000000/resourcegroups/testrg/providers/microsoft.eventhub/namespaces/namespace-name",
    "Category": "EventHubVNetConnectionEvent"
}
```

### <a name="customer-managed-key-user-logs-schema"></a>Esquema de registros de usuario de claves administradas por el cliente
Las cadenas JSON del registro de usuario de claves administradas por el cliente incluyen los elementos enumerados en la tabla siguiente:

| Nombre | Descripción |
| ---- | ----------- | 
| `Category` | Tipo de categoría de un mensaje. Corresponde a uno de los siguientes valores: **error** e **info** |
| `ResourceId` | Identificador de recurso interno, que incluye el identificador de suscripción y el nombre del espacio de nombres de Azure. |
| `KeyVault` | Nombre del recurso de Key Vault. |
| `Key` | Nombre de la clave de Key Vault. |
| `Version` | Versión de la clave de Key Vault. |
| `Operation` | El nombre de una operación realizada para atender solicitudes. |
| `Code` | Código de estado |
| `Message` | Mensaje, que proporciona detalles sobre un error o un mensaje informativo. |