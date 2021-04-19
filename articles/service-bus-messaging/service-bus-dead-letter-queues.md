---
title: Colas de mensajes fallidos de Service Bus | Microsoft Docs
description: Aquí se describen las colas de mensajes fallidos de Azure Service Bus. Las colas de Service Bus y las suscripciones a temas proporcionan una subcola secundaria que se denomina cola de mensajes fallidos.
ms.topic: article
ms.date: 04/08/2021
ms.custom: fasttrack-edit, devx-track-csharp
ms.openlocfilehash: 6459c8edd03427357810c1ad30161e87c18e059c
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107304331"
---
# <a name="overview-of-service-bus-dead-letter-queues"></a>Información general de colas de mensajes fallidos de Service Bus

Las colas de Azure Service Bus y las suscripciones a temas proporcionan una subcola secundaria, llamada *cola de mensajes fallidos* (DLQ). No es necesario crear explícitamente la cola de mensajes fallidos y no se puede eliminar ni administrar independientemente de la entidad principal.

En este artículo se describen las colas de mensajes fallidos de Service Bus. Gran parte de la descripción se muestra en el [ejemplo de colas de mensajes fallidos](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.Azure.ServiceBus/DeadletterQueue) en GitHub.
 
## <a name="the-dead-letter-queue"></a>Cola de mensajes fallidos

La finalidad de la cola de mensajes fallidos es mantener los mensajes que no se pueden entregar a ningún destinatario o los mensajes que no se pudieron procesar. Después, los mensajes se quitan de la cola de mensajes fallidos y se inspeccionan. Una aplicación podría, con ayuda de un operador, corregir los problemas y volver a enviar el mensaje, registrar el hecho de que se produjo un error y llevar a cabo medidas correctivas. 

Desde el punto de vista de la API y el protocolo, la cola de mensajes fallidos es muy similar a cualquier otra cola, salvo que los mensajes solo se pueden enviar a través de la operación de mensajes fallidos de la entidad principal. Además, no se observa el período de vida, y no puede tratar como fallido un mensaje desde una cola de mensajes fallidos. La cola de mensajes fallidos es totalmente compatible con las operaciones transaccionales y de entrega de bloqueo de información.

No hay limpieza automática de mensajes fallidos. Los mensajes permanecen en la cola de mensajes fallidos hasta que los recupere explícitamente de dicha cola y complete el mensaje con problemas de entrega.


## <a name="dlq-message-count"></a>Recuento de mensajes fallidos
No es posible obtener el número de mensajes de la cola de mensajes fallidos en el nivel de tema. Esto se debe a que los mensajes no se encuentran en el nivel de tema, a menos que Service Bus produzca un error interno. En su lugar, cuando un remitente envía un mensaje a un tema, el mensaje se reenvía a las suscripciones del tema en milisegundos y, por tanto, ya no reside en el nivel de tema. Por lo tanto, puede ver los mensajes en los mensajes fallidos asociados con la suscripción del tema. En el ejemplo siguiente, **Service Bus Explorer** muestra que hay 62 mensajes actualmente en los mensajes fallidos de la suscripción "test1". 

![Recuento de mensajes fallidos](./media/service-bus-dead-letter-queues/dead-letter-queue-message-count.png)

También puede obtener el recuento de mensajes fallidos mediante el comando de la CLI de Azure: [`az servicebus topic subscription show`](/cli/azure/servicebus/topic/subscription#az-servicebus-topic-subscription-show). 

## <a name="moving-messages-to-the-dlq"></a>Movimiento de mensajes a la cola de mensajes fallidos
Hay varias actividades en Service Bus que provocan que los mensajes se inserten en la cola de mensajes fallidos desde dentro del propio motor de mensajería. Una aplicación también puede mover mensajes explícitamente a la cola de mensajes fallidos. Las dos propiedades siguientes (motivo del problema de entrega y descripción del problema de entrega) se agregan a los mensajes con problemas de entrega. Las aplicaciones pueden definir sus propios códigos para la propiedad de motivo del problema de entrega, pero el sistema establece los valores siguientes.

| Motivo del problema de entrega | Descripción del error del problema de entrega |
| --- | --- |
|HeaderSizeExceeded |Se superó la cuota de tamaño de esta transmisión. |
|TTLExpiredException |El mensaje expiró y se consideró fallido. Consulte la sección [Período de vida](#time-to-live) para ver los detalles. |
|El identificador de sesión es null. |La entidad habilitada por sesión no permite un mensaje cuyo identificador de sesión es null. |
|MaxTransferHopCountExceeded | Número máximo de saltos permitidos al reenviar contenido entre colas. El valor se establece en 4. |
| MaxDeliveryCountExceededExceptionMessage | El mensaje no se pudo usar después de que transcurriera el número máximo de intentos de entrega. Consulte la sección [Número máximo de entregas](#maximum-delivery-count) para más información. |

## <a name="maximum-delivery-count"></a>Número máximo de entregas
Hay un límite en el número de intentos de entregar mensajes para las colas y suscripciones de Service Bus. El valor predeterminado es 10. Cuando un mensaje se entrega bajo un bloqueo de inspección, pero se abandona explícitamente o el bloqueo expira, el número de entregas del mensaje se incrementa. Cuando el número de entregas supera el límite, el mensaje se mueve a la cola de mensajes fallidos. El motivo del problema de entrega del mensaje en la cola de mensajes fallidos se establece en: MaxDeliveryCountExceeded. Este comportamiento no se puede deshabilitar, pero puede establecer el número máximo de entregas en un número grande.

## <a name="time-to-live"></a>Período de vida
Cuando se habilitan los mensajes con problemas de entrega en colas o suscripciones, todos los mensajes que expiran se mueven a la cola de mensajes fallidos. El código del motivo del problema de entrega se establece en: TTLExpiredException.

Los mensajes expirados solo se purgan y se mueven a la cola de mensajes fallidos cuando hay, al menos, un receptor activo que extrae contenido de la cola o suscripción principal. Los mensajes diferidos tampoco se purgarán ni moverán a la cola de mensajes fallidos tras expirar. Este comportamiento es así por diseño.

## <a name="errors-while-processing-subscription-rules"></a>Errores al procesar reglas de suscripción
Si habilita los mensajes con problemas de entrega en las excepciones de evaluación de filtro, los errores que se producen mientras se ejecuta la regla de filtro SQL de una suscripción se capturan en la cola de mensajes fallidos junto con el mensaje infractor. No use esta opción en un entorno de producción en el que no todos los tipos de mensaje tienen suscriptores.

## <a name="application-level-dead-lettering"></a>Mensajes fallidos de nivel de aplicación
Además de las características de mensajes fallidos proporcionadas por el sistema, las aplicaciones pueden usar la cola de mensajes fallidos para rechazar explícitamente mensajes inaceptables. Esto puede incluir mensajes que no se pueden procesar correctamente debido a cualquier tipo de problema del sistema, mensajes que mantienen cargas útiles con un formato incorrecto o mensajes que no superan la autenticación cuando se utiliza algún esquema de seguridad de nivel de mensaje.

## <a name="dead-lettering-in-forwardto-or-sendvia-scenarios"></a>Colas de mensajes fallidos en escenarios ForwardTo o SendVia
Los mensajes se enviarán a la cola de mensajes fallidos de transferencia en las siguientes condiciones:

- Un mensaje pasa a través de más de cuatro colas o temas que están [encadenados](service-bus-auto-forwarding.md).
- La cola de destino o el tema se han deshabilitado o eliminado.
- El tema o la cola de destino superan el tamaño máximo de entidad.

## <a name="path-to-the-dead-letter-queue"></a>Ruta de acceso para la cola de mensajes fallidos
Puede tener acceso a la cola de mensajes fallidos mediante la sintaxis siguiente:

```
<queue path>/$deadletterqueue
<topic path>/Subscriptions/<subscription path>/$deadletterqueue
```


## <a name="next-steps"></a>Pasos siguientes

Para más información sobre colas de Service Bus, consulte los siguientes artículos:

* [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Comparación de colas de Azure y colas de Service Bus](service-bus-azure-and-service-bus-queues-compared-contrasted.md)

