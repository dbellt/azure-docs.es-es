---
title: 'Azure Service Bus: aplazamiento de mensajes'
description: En este artículo se explica cómo diferir la entrega de mensajes de Azure Service Bus. El mensaje permanece en la cola o la suscripción, pero se reserva.
ms.topic: conceptual
ms.date: 04/21/2021
ms.openlocfilehash: 71446f7989a235e79ca3ab208579edd0043dfeb2
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107987261"
---
# <a name="message-deferral"></a>Aplazamiento de mensajes
Cuando un cliente de cola o suscripción recibe un mensaje que se desea procesar, pero cuyo procesamiento no es posible en ese momento debido a circunstancias especiales, tiene la opción de "aplazar" la recuperación del mensaje para un momento posterior. El mensaje permanece en la cola o la suscripción, pero se reserva.

> [!NOTE]
> Los mensajes aplazados no se moverán de forma automática a la cola de mensajes fallidos [tras expirar](./service-bus-dead-letter-queues.md#time-to-live). Este comportamiento es así por diseño.

## <a name="sample-scenarios"></a>Escenarios de ejemplo
El aplazamiento es una característica creada específicamente para escenarios de procesamiento de flujo de trabajo. Los marcos de los flujos de trabajo pueden requerir que ciertas operaciones se procesen en un orden particular. Es posible que tengan que posponer el procesamiento de algunos mensajes recibidos hasta que se haya completado el trabajo previo indicado por otros mensajes.

Un ejemplo sencillo es una secuencia de procesamiento de pedidos en la que una notificación de pago de un proveedor de pago externo aparece en un sistema antes de haberse propagado el pedido de compra correspondiente desde el escaparate al sistema de suministro. En ese caso, el sistema de suministro puede aplazar el procesamiento de la notificación de pago hasta que haya un pedido con el que se vaya a asociar. En escenarios de encuentro, donde los mensajes de orígenes diferentes conducen a un desvío de flujo de trabajo, el orden de ejecución en tiempo real puede, de hecho, ser correcto pero es posible que los mensajes reflejen que los resultados llegan sin orden.

En última instancia, el aplazamiento contribuye a volver a ordenar los mensajes de su orden de llegada a un orden en el que se pueden procesar, lo que permite dejar los mensajes de forma segura en el almacén de mensajes cuyo procesamiento tiene que posponerse.

Si no se puede procesar un mensaje porque un recurso concreto para controlar ese mensaje no está disponible temporalmente, pero el procesamiento de mensajes no debe suspenderse sumariamente, una forma de apartar ese mensaje durante unos minutos es recordar el número de secuencia de un [mensaje programado](message-sequencing.md) que se vaya a publicar en unos minutos, y volver a recuperar el mensaje aplazado cuando llega el mensaje programado. Tenga en cuenta que, si un controlador de mensajes depende de una base de datos para todas las operaciones y esa base de datos no está disponible temporalmente, esta no debe usar el aplazamiento, sino que, en su lugar, debe suspender la recepción de mensajes por completo hasta que la base de datos vuelva a estar disponible. 

## <a name="retrieving-deferred-messages"></a>Recuperación de mensajes aplazados
Los mensajes aplazados permanecen en la cola principal junto con todos los demás mensajes activos (a diferencia de los mensajes fallidos que están activos en una subcola), pero ya no se reciben mediante las funciones de recepción habituales. Los mensajes aplazados se pueden detectar a través de la [exploración de mensajes](message-browsing.md) si una aplicación pierde su pista.

Para recuperar un mensaje aplazado, su propietario es responsable de recordar la propiedad el número de secuencia cuando se aplaza. Cualquier receptor que conozca el número de secuencia de un mensaje aplazado puede recibir el mensaje más adelante mediante métodos de recepción que toman el número de secuencia como parámetro. Para obtener más información sobre los números de secuencia, consulte [Secuenciación y marcas de tiempo de los mensajes](message-sequencing.md).

## <a name="next-steps"></a>Pasos siguientes
Pruebe los ejemplos en el lenguaje que prefiera para explorar las características de Azure Service Bus. 

- [Ejemplos de la biblioteca cliente de Azure Service Bus para .NET (versión más reciente)](/samples/azure/azure-sdk-for-net/azuremessagingservicebus-samples/): consulte el ejemplo de **resolución de mensajes**. 
- [Ejemplos de la biblioteca cliente de Azure Service Bus para Java (versión más reciente)](/samples/azure/azure-sdk-for-java/servicebus-samples/)
- [Ejemplos de la biblioteca cliente de Azure Service Bus para Python](/samples/azure/azure-sdk-for-python/servicebus-samples/): consulte el ejemplo de **receive_deferred_message_queue.py**. 
- [Ejemplos de la biblioteca cliente de Azure Service Bus para JavaScript](/samples/azure/azure-sdk-for-js/service-bus-javascript/): consulte el ejemplo de **advanced/deferral.js**. 
- [Ejemplos de la biblioteca cliente de Azure Service Bus para TypeScript](/samples/azure/azure-sdk-for-js/service-bus-typescript/): consulte el ejemplo de **advanced/deferral.ts**. 

A continuación, encontrará ejemplos de las bibliotecas cliente de .NET y Java anteriores:
- [Ejemplos de la biblioteca cliente de Azure Service Bus para .NET (versión heredada)](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.Azure.ServiceBus/): consulte el ejemplo de **aplazamiento**. 
- [Ejemplos de la biblioteca cliente de Azure Service Bus para Java (versión heredada)](https://github.com/Azure/azure-service-bus/tree/master/samples/Java/azure-servicebus/MessageBrowse)
