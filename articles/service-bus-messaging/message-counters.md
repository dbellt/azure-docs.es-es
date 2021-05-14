---
title: Número de mensajes de Azure Service Bus
description: Recupere el número de mensajes incluidos en las colas y las suscripciones mediante las API NamespaceManager de Azure Resource Manager y Azure Service Bus.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 3ef2e50521b57fc45846acecf9e776e3ff24c5d4
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107987333"
---
# <a name="get-message-counters"></a>Obtención de contadores de mensajes
En este artículo se muestran diferentes maneras de obtener los siguientes recuentos de mensajes para una cola o suscripción. Conocer el número de mensajes activos resulta de utilidad para determinar si una cola crea un trabajo pendiente que requiere procesar más recursos que los que se han implementado actualmente. 

| Contador | Descripción |
| ----- | ---------- | 
| ActiveMessageCount | Número de mensajes de la cola o la suscripción que se encuentran en estado activo y listo para la entrega. |
| ScheduledMessageCount | Número de mensajes en estado programado. |
| DeadLetterMessageCount | Número de mensajes en la cola de mensajes fallidos. |
| TransferMessageCount | Número de mensajes pendientes de transferencia en otra cola o tema. |
| TransferDeadLetterMessageCount | Número de mensajes que no se pudieron transferir a otra cola u otro tema y se han movido a la cola de mensajes fallidos de transferencia. |

Si una aplicación desea escalar recursos en función de la longitud de la cola, debe hacerlo con un ritmo medido. La adquisición de los contadores de mensajes es una operación costosa en el agente de mensajes y ejecutarla con frecuencia afecta de forma directa y adversa al rendimiento de la entidad.

> [!NOTE]
> Los mensajes que se envían a un tema de Service Bus se reenvían a las suscripciones de ese tema. Por lo tanto, el número de mensajes activos en el propio tema es 0, ya que esos mensajes se han reenviado correctamente a la suscripción. Obtenga el número de mensajes en la suscripción y compruebe que es mayor que 0. Aunque vea mensajes en la suscripción, realmente se almacenan en un almacenamiento que pertenece al tema. Si observa las suscripciones, tendrán un número de mensajes distinto de cero (hasta un total de 323 MB de espacio para esta entidad completa).


## <a name="using-azure-portal"></a>Uso de Azure Portal
Vaya al espacio de nombres y seleccione la cola. Verá contadores de mensajes en la página **Información general** de la cola.

:::image type="content" source="./media/message-counters/queue-overview.png" alt-text="Contadores de mensajes en la página de información general de la cola":::

Vaya al espacio de nombres, seleccione el tema y, a continuación, seleccione la suscripción para el tema. Verá contadores de mensajes en la página **Información general** de la cola.

:::image type="content" source="./media/message-counters/subscription-overview.png" alt-text="Contadores de mensajes en la página de información general de la suscripción":::

## <a name="using-azure-cli"></a>Uso de la CLI de Azure
Use el comando [`az servicebus queue show`](/cli/azure/servicebus/queue#az_servicebus_queue_show) para obtener los detalles del recuento de mensajes de una cola, como se muestra en el ejemplo siguiente. 

```azurecli-interactive
az servicebus queue show --resource-group myresourcegroup \
    --namespace-name mynamespace \
    --name myqueue \
    --query countDetails
```

Este es una salida de ejemplo:

```bash
ActiveMessageCount    DeadLetterMessageCount    ScheduledMessageCount    TransferMessageCount    TransferDeadLetterMessageCount
--------------------  ------------------------  -----------------------  ----------------------  --------------------------------
0                     0                         0                        0                       0
```

Use el comando [`az servicebus topic subscription show`](/cli/azure/servicebus/topic/subscription#az_servicebus_topic_subscription_show) para obtener los detalles del recuento de mensajes de una suscripción, como se muestra en el ejemplo siguiente. 

```azurecli-interactive
az servicebus topic subscription show --resource-group myresourcegroup \
    --namespace-name mynamespace \
    --topic-name mytopic \
    --name mysub \
    --query countDetails
```

## <a name="using-azure-powershell"></a>Uso de Azure PowerShell
Con PowerShell, puede obtener los detalles del recuento de mensajes de una cola como se indica a continuación:

```azurepowershell-interactive
$queueObj=Get-AzServiceBusQueue -ResourceGroup myresourcegroup `
                    -NamespaceName mynamespace `
                    -QueueName myqueue 

$queueObj.CountDetails
```

Esta es la salida de ejemplo:

```bash
ActiveMessageCount             : 7
DeadLetterMessageCount         : 1
ScheduledMessageCount          : 3
TransferMessageCount           : 0
TransferDeadLetterMessageCount : 0
```

puede obtener los detalles del recuento de mensajes de una suscripción de la siguiente manera:

```azurepowershell-interactive
$topicObj= Get-AzServiceBusSubscription -ResourceGroup myresourcegroup `
                -NamespaceName mynamespace `
                -TopicName mytopic `
                -SubscriptionName mysub

$topicObj.CountDetails
```

El objeto `MessageCountDetails` devuelto tiene las siguientes propiedades: `ActiveMessageCount`, `DeadLetterMessageCount`, `ScheduledMessageCount`, `TransferDeadLetterMessageCount`, `TransferMessageCount`. 

## <a name="next-steps"></a>Pasos siguientes

Pruebe los ejemplos en el lenguaje que prefiera para explorar las características de Azure Service Bus. 

- [Ejemplos de la biblioteca cliente de Azure Service Bus para .NET (versión más reciente)](/samples/azure/azure-sdk-for-net/azuremessagingservicebus-samples/) 
- [Ejemplos de la biblioteca cliente de Azure Service Bus para Java (versión más reciente)](/samples/azure/azure-sdk-for-java/servicebus-samples/)
- [Ejemplos de la biblioteca cliente de Azure Service Bus para Python](/samples/azure/azure-sdk-for-python/servicebus-samples/)
- [Ejemplos de la biblioteca cliente de Azure Service Bus para JavaScript](/samples/azure/azure-sdk-for-js/service-bus-javascript/)
- [Ejemplos de la biblioteca cliente de Azure Service Bus para TypeScript](/samples/azure/azure-sdk-for-js/service-bus-typescript/)

A continuación, encontrará ejemplos de las bibliotecas cliente de .NET y Java anteriores:
- [Ejemplos de la biblioteca cliente de Azure Service Bus para .NET (versión heredada)](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.Azure.ServiceBus/)
- [Ejemplos de la biblioteca cliente de Azure Service Bus para Java (versión heredada)](https://github.com/Azure/azure-service-bus/tree/master/samples/Java/azure-servicebus)
