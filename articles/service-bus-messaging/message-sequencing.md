---
title: Marcas de tiempo y secuenciación de mensajes de Azure Service Bus | Microsoft Docs
description: En este artículo se explica cómo conservar la secuenciación y el orden (con marcas de tiempo) de los mensajes de Azure Service Bus.
ms.topic: article
ms.date: 04/14/2021
ms.openlocfilehash: 3d5300568232afae1238445113d60eda8cdb2f1b
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107497105"
---
# <a name="message-sequencing-and-timestamps"></a>Secuenciación y marcas de tiempo de los mensajes

La secuenciación y las marcas de tiempo son dos características que siempre están habilitadas en todas las entidades y la superficie de Service Bus mediante las propiedades `SequenceNumber` y `EnqueuedTimeUtc` de los mensajes recibidos o examinados.

En aquellos casos en los que sea importante el orden absoluto de los mensajes o en los que un consumidor necesite un identificador único de confianza para los mensajes, el agente marca los mensajes con un número de secuencia creciente, sin espacios, relativo a la cola o el tema. Para las entidades con particiones, el número de secuencia se emite con respecto a la partición.

El valor **SequenceNumber** es un entero de 64 bits único asignado a un mensaje, que el agente acepta y almacena y funciona como su identificador interno. Para entidades con particiones, los 16 bits superiores reflejan el identificador de la partición. Los números de secuencia se ponen a cero cuando se agota el intervalo de 48/64 bits.

El número de secuencia se puede considerar un identificador único de confianza dado que lo asigna una autoridad central y neutral y no los clientes. También representa el verdadero orden de llegada y es más preciso como criterio de orden que una marca de tiempo, porque las marcas de tiempo no pueden tener una resolución lo suficientemente alta a velocidades de mensaje extremas y pueden estar sujetas a sesgos de reloj (no obstante, mínimos) en situaciones donde la propiedad del agente pasa de un nodo a otro.

El orden de llegada absoluto importa, por ejemplo, en escenarios empresariales en los que se sirve un número limitado de mercancías por orden de llegada mientras duran los suministros; la venta de entradas para conciertos es un ejemplo.

La funcionalidad de marca de tiempo actúa como autoridad neutral y de confianza que captura de forma precisa la hora UTC de llegada de un mensaje, reflejada en la propiedad **EnqueuedTimeUtc**. El valor es útil si un escenario empresarial depende de fechas límite, por ejemplo, si un elemento de trabajo se envió en una fecha determinada antes de medianoche, pero el procesamiento está muy por detrás del trabajo pendiente de la cola.

## <a name="scheduled-messages"></a>Mensajes programados

Puede enviar mensajes a una cola o tema para su procesamiento retrasado; por ejemplo, para programar un trabajo de forma que esté disponible para que lo procese el sistema a una hora determinada. Esta funcionalidad logra un programador basado en el tiempo distribuido confiable.

Los mensajes programados no se materializan en la cola hasta la hora de puesta en cola definida. Antes de esa hora, pueden cancelarse. La cancelación elimina el mensaje.

Puede programar mensajes mediante cualquiera de nuestros clientes de dos maneras:
- Use la API de envío normal, pero establezca la propiedad `ScheduledEnqueueTimeUtc` en el mensaje antes de enviarlo.
- Use la API de programación de mensajes y pase el mensaje normal y la hora programada. Esto devuelve el valor de **SequenceNumber** del mensaje programado, que luego puede usarse para cancelar dicho mensaje si es necesario. 

Los mensajes programados y sus números de secuencia también se pueden detectar mediante la [exploración de mensajes](message-browsing.md).

El valor de **SequenceNumber** de un mensaje programado solo es válido mientras el mensaje está en este estado. A medida que el mensaje pasa al estado activo, se anexa a la cola como si se hubiera puesto en cola en el instante actual, lo que incluye asignar un nuevo **SequenceNumber**.

Dado que la característica está anclada en mensajes individuales y los mensajes solo pueden ponerse en cola una vez, Service Bus no admite las programaciones recurrentes de los mensajes.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la mensajería de Service Bus, consulte los siguientes temas:

* [Colas, temas y suscripciones de Service Bus](service-bus-queues-topics-subscriptions.md)
* [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Uso de temas y suscripciones de Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)
