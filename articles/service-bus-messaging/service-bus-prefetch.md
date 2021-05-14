---
title: Captura previa de mensajes de Azure Service Bus | Microsoft Docs
description: Mejore el rendimiento mediante la captura previa de mensajes de Azure Service Bus. Los mensajes están disponibles fácilmente para su recuperación local antes de que la aplicación los solicite.
ms.topic: article
ms.date: 04/23/2021
ms.openlocfilehash: 992fa9228d3cf8bafb3d2ea9110b515ff4b56498
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107989097"
---
# <a name="prefetch-azure-service-bus-messages"></a>Captura previa de mensajes de Azure Service Bus
Al habilitar la característica *Captura previa* para cualquiera de los clientes de Service Bus oficiales, el receptor adquiere más mensajes de los que la aplicación solicitó inicialmente, hasta el recuento de captura previa especificado. Tenga en cuenta que el cliente de JavaScript y TypeScript aún no admite esta característica.

A medida que se devuelven mensajes a la aplicación, el cliente adquiere más mensajes en segundo plano para rellenar el búfer de captura previa.

## <a name="enabling-prefetch"></a>Habilitación de la captura previa
Para habilitar la característica de captura previa, establezca el recuento de captura previa del cliente de cola o suscripción en un número mayor que cero. Si se establece el valor en cero, se desactivará la captura previa. 

# <a name="net"></a>[.NET](#tab/dotnet)
Si usa la biblioteca Azure.Messaging.ServiceBus más reciente, puede establecer la propiedad de recuento de capturas previas en los objetos [ServiceBusReceiver](/dotnet/api/azure.messaging.servicebus.servicebusreceiver.prefetchcount#Azure_Messaging_ServiceBus_ServiceBusReceiver_PrefetchCount) y [ServiceBusProcessor](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.prefetchcount#Azure_Messaging_ServiceBus_ServiceBusProcessor_PrefetchCount). 

Si usa la biblioteca cliente de .NET anterior para Service Bus (Microsoft.Azure.ServiceBus), puede establecer la propiedad de recuento de capturas previa en [MessageReceiver](/dotnet/api/microsoft.servicebus.messaging.messagereceiver.prefetchcount), [QueueClient](/dotnet/api/microsoft.azure.servicebus.queueclient.prefetchcount#Microsoft_Azure_ServiceBus_QueueClient_PrefetchCount) o [SubscriptionClient](/dotnet/api/microsoft.azure.servicebus.subscriptionclient.prefetchcount).
 
# <a name="java"></a>[Java](#tab/java)
Si usa la biblioteca azure-messaging-servicebus más reciente, puede establecer la propiedad de recuento de capturas previas en los objetos [ServiceBusReceiver](/dotnet/api/azure.messaging.servicebus.servicebusreceiver.prefetchcount#Azure_Messaging_ServiceBus_ServiceBusReceiver_PrefetchCount) y [ServiceBusProcessor](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.prefetchcount#Azure_Messaging_ServiceBus_ServiceBusProcessor_PrefetchCount). 

Si usa la biblioteca cliente de Java anterior para Service Bus (azure-servicebus), puede establecer la propiedad de recuento de capturas previa en [MessageReceiver](/java/api/com.microsoft.azure.servicebus.imessagereceiver.setprefetchcount#com_microsoft_azure_servicebus_IMessageReceiver_setPrefetchCount_int_), [QueueClient](/java/api/com.microsoft.azure.servicebus.queueclient.setprefetchcount#com_microsoft_azure_servicebus_QueueClient_setPrefetchCount_int_) o [SubscriptionClient](/java/api/com.microsoft.azure.servicebus.subscriptionclient.setprefetchcount#com_microsoft_azure_servicebus_SubscriptionClient_setPrefetchCount_int_).
 
# <a name="python"></a>[Python](#tab/python)

Puede establecer **prefetch_count** en [azure.servicebus.ServiceBusReceiver](/python/api/azure-servicebus/azure.servicebus.servicebusreceiver) o [azure.servicebus.aio.ServiceBusReceiver](/python/api/azure-servicebus/azure.servicebus.aio.servicebusreceiver).

---

> [!NOTE]
> El SDK de Javascript no admite la característica **Captura previa**. 

Aunque los mensajes están disponibles en el búfer de captura previa, las llamadas de recepción subsiguientes se realizan directamente desde el búfer. Este se repone en segundo plano a medida que haya espacio disponible. Si no hay ningún mensaje disponible para la entrega, la operación de recepción vacía el búfer y, después, lo espera o bloquea, como esté previsto.

## <a name="why-is-prefetch-not-the-default-option"></a>¿Por qué la captura previa no es la opción predeterminada?
La captura previa acelera el flujo de mensajes al hacer que el mensaje esté fácilmente disponible para la recuperación local antes de que la aplicación lo solicite. Esta mejora de rendimiento es el resultado de un intercambio que el autor de la aplicación debe realizar explícitamente:

Con modo de recepción [receive-and-delete](message-transfers-locks-settlement.md#receiveanddelete), ya no están disponibles en la cola todos los mensajes que se adquieren en el búfer de captura previa. Los mensajes permanecen solo en el búfer de captura previa en memoria hasta que se reciben en la aplicación. Si la aplicación finaliza antes de que se reciban los mensajes en la aplicación, estos no se pueden recuperar (se pierden).

En el modo de recepción [peek-lock](message-transfers-locks-settlement.md#peeklock), los mensajes capturados en el búfer de captura previa se adquieren en el búfer en un estado bloqueado. Se selecciona el reloj de tiempo de espera para el bloqueo. Si el búfer de captura previa es grande y el procesamiento tarda tanto tiempo que expiran los bloqueos de ese mensaje mientras permanece en el búfer de captura previa o incluso mientras la aplicación está procesando el mensaje, puede haber algunos eventos confusos que aplicación tenga que controlar.

La aplicación podría adquirir un mensaje con un bloqueo que haya expirado o que esté a punto a punto de expirar. Si es así, la aplicación puede procesar el mensaje, pero, después, encuentra que no puede completarlo debido a la expiración de un bloqueo. La aplicación puede comprobar la propiedad `LockedUntilUtc` (que está sujeta a desfases del reloj entre el reloj del agente y de la máquina local). Si el bloqueo del mensaje ha expirado, la aplicación debe ignorar el mensaje y no debe realizarse ninguna llamada API en el mensaje. Si el mensaje no ha expirado pero la expiración es inminente, el bloqueo se puede renovar y ampliar durante otro período de bloqueo predeterminado.

Si el bloqueo expira en modo silencioso en el búfer de captura previa, el mensaje se trata como abandonado y de nuevo estará disponible para su recuperación desde la cola. Esto podría provocar que el mensaje se capturara en el búfer de captura previa y se colocara al final. Si el búfer de captura previa no se puede trabajar normalmente durante la expiración del mensaje, los mensajes se capturan previamente de forma repetida, pero nunca se entregan de forma eficaz en un estado utilizable (bloqueado válidamente), y, finalmente, se mueven a la cola de mensajes fallidos una vez que se supera el número máximo de entregas.

Si necesita un alto grado de confiabilidad para procesar los mensajes y el procesamiento requiere un trabajo y tiempo considerable, se recomienda usar la característica de captura previa de manera conservadora o no utilizarla en absoluto.

Si necesita un elevado rendimiento y el procesamiento de mensajes es normalmente barato, la captura previa aporta ventajas de rendimiento significativas.

El número máximo de captura previa y la duración del bloqueo configurado en la cola o suscripción deben equilibrarse de forma que el tiempo de espera del bloqueo supere al menos el tiempo de procesamiento de mensajes acumulativo para el tamaño máximo del búfer de captura previa, además de un mensaje. Al mismo tiempo, el tiempo de expiración de bloqueo no debe ser tan prolongado que los mensajes superen su período de vida máximo cuando se quitan accidentalmente, lo que requiere que su bloqueo expire antes de que se vaya a volver a entregar.

## <a name="next-steps"></a>Pasos siguientes

Pruebe los ejemplos en el lenguaje que prefiera para explorar las características de Azure Service Bus. 

- [Ejemplos de la biblioteca cliente de Azure Service Bus para .NET (versión más reciente)](/samples/azure/azure-sdk-for-net/azuremessagingservicebus-samples/) 
- [Ejemplos de la biblioteca cliente de Azure Service Bus para Java (versión más reciente)](/samples/azure/azure-sdk-for-java/servicebus-samples/)
- [Ejemplos de la biblioteca cliente de Azure Service Bus para Python](/samples/azure/azure-sdk-for-python/servicebus-samples/)
- [Ejemplos de la biblioteca cliente de Azure Service Bus para JavaScript](/samples/azure/azure-sdk-for-js/service-bus-javascript/)
- [Ejemplos de la biblioteca cliente de Azure Service Bus para TypeScript](/samples/azure/azure-sdk-for-js/service-bus-typescript/)

A continuación, encontrará ejemplos de las bibliotecas cliente de .NET y Java anteriores:
- [Ejemplos de la biblioteca cliente de Azure Service Bus para .NET (heredado)](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.Azure.ServiceBus/): consulte el ejemplo de **captura previa**. 
- [Ejemplos de la biblioteca cliente de Azure Service Bus para Java (heredado)](https://github.com/Azure/azure-service-bus/tree/master/samples/Java/azure-servicebus): consulte el ejemplo de **captura previa**. 
