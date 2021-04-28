---
title: 'Disponibilidad y coherencia: Azure Event Hubs | Microsoft Docs'
description: Cómo proporcionar el máximo nivel de disponibilidad y coherencia con Azure Event Hubs mediante el uso de particiones.
ms.topic: article
ms.date: 03/15/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 02f26aaf369db177d77d27f3274b34fe1c400419
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107988953"
---
# <a name="availability-and-consistency-in-event-hubs"></a>Disponibilidad y coherencia en Event Hubs
En este artículo se proporciona información sobre la disponibilidad y la coherencia que admite Azure Event Hubs. 

## <a name="availability"></a>Disponibilidad
Azure Event Hubs extiende el riesgo de errores catastróficos de máquinas individuales o incluso de bastidores completos en clústeres que abarcan varios dominios de error dentro de un centro de recursos. Implementa la detección transparente de errores y los mecanismos de conmutación por error para que el servicio siga funcionando dentro de los niveles de servicio garantizados y, normalmente, sin interrupciones perceptibles cuando se producen estos errores. 

Si se creó un espacio de nombres de Event Hubs con [zonas de disponibilidad](../availability-zones/az-overview.md) habilitadas, el riesgo de interrupción se reparte entre tres instalaciones separadas físicamente, y el servicio tiene suficientes reservas de capacidad para hacer frente de inmediato a la pérdida completa y catastrófica de toda la instalación. Para más información, consulte [Azure Event Hubs: recuperación ante desastres geográfica](event-hubs-geo-dr.md).

Cuando una aplicación cliente envía eventos a un centro de eventos sin especificar una partición, los eventos se distribuyen automáticamente entre las particiones del centro de eventos. Si una partición no está disponible por algún motivo, los eventos se distribuyen entre las particiones restantes. Este comportamiento permite disfrutar del máximo tiempo de actividad. En los casos de uso que requieren el tiempo de actividad máximo, se prefiere este modelo en lugar de enviar eventos a una partición específica. 

## <a name="consistency"></a>Coherencia
En algunos escenarios, el orden de los eventos puede ser importante. Por ejemplo, puede que prefiera el sistema back-end para procesar un comando de actualización antes que un comando de eliminación. En este escenario, una aplicación cliente envía eventos a una partición específica para que se conserve el orden. Cuando una aplicación de consumidor consume estos eventos de la partición, se leen por orden. 

Con esta configuración, tenga en cuenta que si la partición concreta a la que se realiza el envío no se encuentra disponible, recibirá una respuesta de error. Como punto de comparación, si no tiene una afinidad para una sola partición, el servicio Event Hubs envía el evento a la siguiente partición disponible.

Por lo tanto, si la alta disponibilidad es más importante, no fije como destino una partición específica (mediante la clave o el identificador de la partición). El uso de una clave o un identificador de la partición degrada la disponibilidad de un centro de eventos en el nivel de partición. En este escenario, se realiza una selección explícita entre disponibilidad (sin identificador ni clave de partición) y coherencia (anclaje de eventos a una partición específica). Para obtener información detallada sobre las particiones en Event Hubs, vea [Particiones](event-hubs-features.md#partitions).

## <a name="appendix"></a>Apéndice

### <a name="send-events-without-specifying-a-partition"></a>Envío de eventos sin especificar ninguna partición
Se recomienda enviar eventos a un centro de eventos sin establecer la información de la partición para permitir que el servicio Event Hubs equilibre la carga entre las particiones. Consulte las siguientes guías de inicio rápido para aprender a hacerlo en distintos lenguajes de programación. 

- [Envío de eventos mediante .NET](event-hubs-dotnet-standard-getstarted-send.md)
- [Envío de eventos mediante Java](event-hubs-java-get-started-send.md)
- [Envío de eventos mediante JavaScript](event-hubs-python-get-started-send.md)
- [Envío de eventos mediante Python](event-hubs-python-get-started-send.md)


### <a name="send-events-to-a-specific-partition"></a>Envío de eventos a una partición específica
En esta sección aprenderá a enviar eventos a una partición específica mediante distintos lenguajes de programación. 

### <a name="net"></a>[.NET](#tab/dotnet)
Para enviar eventos a una partición específica, cree el lote mediante el método [EventHubProducerClient.CreateBatchAsync](/dotnet/api/azure.messaging.eventhubs.producer.eventhubproducerclient.createbatchasync#Azure_Messaging_EventHubs_Producer_EventHubProducerClient_CreateBatchAsync_Azure_Messaging_EventHubs_Producer_CreateBatchOptions_System_Threading_CancellationToken_). Para ello, especifique `PartitionId` o `PartitionKey` en [CreateBatchOptions](/dotnet/api/azure.messaging.eventhubs.producer.createbatchoptions). El código siguiente envía un lote de eventos a una partición específica mediante la especificación de una clave de partición. Event Hubs garantiza que todos los eventos que comparten un valor de clave de partición se almacenen juntos y se entreguen en el orden de llegada.

```csharp
var batchOptions = new CreateBatchOptions { PartitionKey = "cities" };
using var eventBatch = await producer.CreateBatchAsync(batchOptions);
```

También puede usar el método [EventHubProducerClient.SendAsync](/dotnet/api/azure.messaging.eventhubs.producer.eventhubproducerclient.sendasync#Azure_Messaging_EventHubs_Producer_EventHubProducerClient_SendAsync_System_Collections_Generic_IEnumerable_Azure_Messaging_EventHubs_EventData__Azure_Messaging_EventHubs_Producer_SendEventOptions_System_Threading_CancellationToken_). Para ello, especifique **PartitionId** o **PartitionKey** en [SendEventOptions](/dotnet/api/azure.messaging.eventhubs.producer.sendeventoptions).

```csharp
var sendEventOptions  = new SendEventOptions { PartitionKey = "cities" };
// create the events array
producer.SendAsync(events, sendOptions)
```


### <a name="java"></a>[Java](#tab/java)
Para enviar eventos a una partición específica, cree el lote con el método [createBatch](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs/src/main/java/com/azure/messaging/eventhubs/EventHubProducerClient.java). Para ello, especifique el **identificador de partición** o la **clave de partición** en [createBatchOptions](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs/src/main/java/com/azure/messaging/eventhubs/models/CreateBatchOptions.java). El código siguiente envía un lote de eventos a una partición específica mediante la especificación de una clave de partición. 

```java
CreateBatchOptions batchOptions = new CreateBatchOptions();
batchOptions.setPartitionKey("cities");
```

También puede usar el método [EventHubProducerClient.send](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs/src/main/java/com/azure/messaging/eventhubs/EventHubProducerClient.java). Para ello, especifique el **identificador de partición** o la **clave de partición** en [SendOptions](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs/src/main/java/com/azure/messaging/eventhubs/models/SendOptions.java).

```java
List<EventData> events = Arrays.asList(new EventData("Melbourne"), new EventData("London"), new EventData("New York"));
SendOptions sendOptions = new SendOptions();
sendOptions.setPartitionKey("cities");
producer.send(events, sendOptions);
```


### <a name="python"></a>[Python](#tab/python) 
Para enviar eventos a una partición específica, al crear un lote mediante el método [`EventHubProducerClient.create_batch`](/python/api/azure-eventhub/azure.eventhub.eventhubproducerclient#create-batch---kwargs-), especifique `partition_id` o `partition_key`. A continuación, use el método [`EventHubProducerClient.send_batch`](/python/api/azure-eventhub/azure.eventhub.aio.eventhubproducerclient#send-batch-event-data-batch--typing-union-azure-eventhub--common-eventdatabatch--typing-list-azure-eventhub-) para enviar el lote a la partición del centro de eventos. 

```python
event_data_batch = await producer.create_batch(partition_key='cities')
```

También puede utilizar el método [EventHubProducerClient.send_batch](/python/api/azure-eventhub/azure.eventhub.eventhubproducerclient#send-batch-event-data-batch----kwargs-). Para ello, especifique los valores de los parámetros `partition_id` o `partition_key`.

```python
producer.send_batch(event_data_batch, partition_key="cities")
```

### <a name="javascript"></a>[JavaScript](#tab/javascript)
Para enviar eventos a una partición específica, [cree un lote](/javascript/api/@azure/event-hubs/eventhubproducerclient#createBatch_CreateBatchOptions_) mediante el objeto [EventHubProducerClient.CreateBatchOptions](/javascript/api/@azure/event-hubs/eventhubproducerclient#createBatch_CreateBatchOptions_). Para ello, especifique `partitionId` o `partitionKey`. Después, envíe el lote al centro de eventos mediante el método [EventHubProducerClient.SendBatch](/javascript/api/@azure/event-hubs/eventhubproducerclient#sendBatch_EventDataBatch__OperationOptions_). 

Consulte el ejemplo siguiente.

```javascript
const batchOptions = { partitionKey = "cities"; };
const batch = await producer.createBatch(batchOptions);
```

También puede usar el método [EventHubProducerClient.sendBatch](/javascript/api/@azure/event-hubs/eventhubproducerclient#sendBatch_EventData____SendBatchOptions_). Para ello, especifique el **identificador de partición** o la **clave de partición** en [SendBatchOptions](/javascript/api/@azure/event-hubs/sendbatchoptions).

```javascript
const sendBatchOptions = { partitionKey = "cities"; };
// prepare events
producer.sendBatch(events, sendBatchOptions);
```

---



## <a name="next-steps"></a>Pasos siguientes
Para más información acerca de Event Hubs, visite los vínculos siguientes:

- [Información general sobre el servicio Event Hubs](./event-hubs-about.md)
- [Terminología de Event Hubs](event-hubs-features.md)
