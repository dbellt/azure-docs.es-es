---
title: 'Mensajería de Azure Service Bus: colas, temas y suscripciones'
description: En este artículo se proporciona información general sobre las entidades de mensajería (colas, temas y suscripciones) de Azure Service Bus.
ms.topic: conceptual
ms.date: 04/08/2021
ms.openlocfilehash: c3e50c5f8487a0f6dc26a50ec6bfb10f4136f899
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110479101"
---
# <a name="service-bus-queues-topics-and-subscriptions"></a>Colas, temas y suscripciones de Service Bus
Azure Service Bus admite un conjunto de tecnologías middleware orientadas a mensajes basadas en la nube, entre las que se incluyen una cola de mensajes de confianza y una mensajería de publicación/suscripción duradera. Estas funcionalidades de mensajería asincrónica pueden considerarse como características de mensajería desacopladas que admiten la publicación-suscripción, el desacoplamiento temporal y los escenarios de equilibrio de carga mediante la carga de trabajo de mensajería de Service Bus. La comunicación desacoplada ofrece muchas ventajas. Por ejemplo, los clientes y servidores pueden conectarse según sea necesario y realizar sus operaciones de manera asincrónica.

Las entidades de mensajería que forman el núcleo de las funcionalidades de mensajería de Service Bus son las **colas**, los **temas y suscripciones** y las reglas o acciones.

## <a name="queues"></a>Colas
Las colas ofrecen una entrega de mensajes según el modelo **primero en entrar, primero en salir (FIFO [PEPS])** a uno o más destinatarios de la competencia. Es decir, los receptores normalmente reciben y procesan los mensajes en el orden en el que se agregaron a la cola. Además, solo un destinatario de mensaje recibe y procesa cada uno de los mensajes. La principal ventaja del uso de colas es conseguir un **desacoplamiento temporal de los componentes de la aplicación**. En otras palabras, los productores (remitentes) y los consumidores (receptores) no tienen que enviar y recibir mensajes al mismo tiempo. Esto se debe a que los mensajes se almacenan de manera duradera en la cola. El productor no tiene que esperar una respuesta del destinatario para continuar el proceso y el envío de mensajes.

Una ventaja relacionada es la **nivelación de la carga**, lo que permite a los productores y consumidores enviar y recibir mensajes con distintas velocidades. En muchas aplicaciones, la carga del sistema varía con el tiempo. Sin embargo, el tiempo de procesamiento necesario para cada unidad de trabajo suele ser constante. La intermediación de productores y consumidores de mensajes con una cola implica que la aplicación consumidora solo necesita la capacidad de administrar una carga promedio, en lugar de una carga de mucha actividad. La profundidad de la cola aumenta y se contrae a medida que varíe la carga entrante, lo que permite ahorrar dinero directamente en función de la cantidad de infraestructura requerida para dar servicio a la carga de la aplicación. A medida que aumenta la carga, se pueden agregar más procesos de trabajo para que puedan leerse desde la cola. Cada mensaje se procesa únicamente por uno de los procesos de trabajo. Es más, este equilibrio de carga basado en la extracción permite el uso óptimo de los equipos de trabajo aunque estos equipos con capacidad de procesamiento extraigan mensajes a la frecuencia máxima. Este patrón con frecuencia se denomina patrón de **consumo de competidor**.

El uso de colas para intermediar entre los consumidores y productores de mensajes proporciona un acoplamiento no estricto inherente entre los componentes. Dado que los productores y consumidores no están relacionados entre sí, un consumidor puede actualizarse sin tener ningún efecto en el productor.

### <a name="create-queues"></a>Creación de colas
Puede crear colas mediante [Azure Portal](service-bus-quickstart-portal.md), [PowerShell](service-bus-quickstart-powershell.md), la [CLI](service-bus-quickstart-cli.md) o [plantillas de Resource Manager](service-bus-resource-manager-namespace-queue.md). A continuación, envíe y reciba mensajes mediante clientes escritos en [C#](service-bus-dotnet-get-started-with-queues.md), [Java](service-bus-java-how-to-use-queues.md), [Python](service-bus-python-how-to-use-queues.md), [JavaScript](service-bus-nodejs-how-to-use-queues.md) y [PHP](service-bus-php-how-to-use-queues.md). 

### <a name="receive-modes"></a>Modos de recepción
Puede especificar dos modos distintos en los que Service Bus recibe los mensajes.

- **Recepción y eliminación**. En este modo, cuando Service Bus recibe la solicitud del consumidor, marca el mensaje como consumido y lo devuelve a la aplicación del consumidor. Este modo es el modelo más sencillo. Funciona mejor para los escenarios en los que la aplicación puede tolerar no procesar un mensaje en caso de error. Para entender este escenario, pongamos una situación en la que un consumidor emite la solicitud de recepción que se bloquea antes de procesarla. Ya que Service Bus marca el mensaje como consumido, la aplicación comienza a consumir mensajes al reiniciarse. Se omitirá el mensaje que se consumió antes del bloqueo.
- **Inspección y bloqueo**. En este modo, la operación de recepción se convierte en una operación de dos fases que hace posible admitir aplicaciones que no toleran la pérdida de mensajes. 
    1. Busca el siguiente mensaje que se va a consumir, lo **bloquea** para impedir que otros consumidores lo reciban y, a continuación, lo devuelve a la aplicación. 
    1. Una vez que la aplicación termina de procesar el mensaje, solicita al servicio Service Bus que complete la segunda fase del proceso de recepción. A continuación, el servicio **marca el mensaje como consumido**. 

        Si la aplicación no puede procesar el mensaje por alguna razón, puede solicitar al servicio Service Bus que **abandone** el mensaje. Service Bus **desbloquea** el mensaje y hace que esté disponible para que el mismo consumidor u otro vuelvan a recibirlo. En segundo lugar, hay un **tiempo de espera** asociado con el bloqueo. Si la aplicación no puede procesar el mensaje antes de que finalice el tiempo de espera de bloqueo, Service Bus desbloquea el mensaje y hace que esté disponible para que pueda volver a recibirse.

        Si la aplicación se bloquea tras procesar el mensaje, pero antes de solicitar al servicio Service Bus que complete el mensaje, Service Bus volverá a entregar el mensaje a la aplicación cuando se reinicie. Este proceso se suele denominar procesamiento de **al menos una vez**. Es decir, cada mensaje se procesa al menos una vez. Sin embargo, en determinadas situaciones, es posible que se vuelva a entregar el mismo mensaje. Si el escenario no tolera el procesamiento duplicado, agregue lógica adicional en la aplicación para detectar duplicados. Para más información, consulte [Detección de duplicados](duplicate-detection.md). Esta característica se conoce como procesamiento **exactamente una vez**.

        > [!NOTE]
        > Para más información acerca de estos dos modos, consulte [Liquidación de las operaciones de recepción](message-transfers-locks-settlement.md#settling-receive-operations).

## <a name="topics-and-subscriptions"></a>Temas y suscripciones
Una cola permite que un único consumidor procese un mensaje. En comparación con las colas, los temas y suscripciones proporcionan una vía de comunicación uno a varios en un patrón de **publicación y suscripción**. Resulta útil para escalar a un gran número de destinatarios. Cada mensaje publicado se pone a disposición de todas las suscripciones registradas en el tema. El publicador envía un mensaje a un tema y uno o varios suscriptores reciben una copia del mensaje, en función de las reglas de filtro establecidas en las suscripciones. Las suscripciones pueden usar filtros adicionales para restringir los mensajes que desean recibir. Los publicadores envían mensajes a un tema de la misma manera en que envían mensajes a una cola. No obstante, los consumidores no reciben mensajes directamente del tema. En su lugar, los reciben de las suscripciones del tema. Una suscripción al tema se parece a una cola virtual en que recibe copias de los mensajes que se envían al tema. Los consumidores reciben los mensajes de una suscripción exactamente de la misma manera en que se reciben de una cola.

La funcionalidad de envío de mensajes de una cola los asigna directamente a un tema y su funcionalidad de recepción de mensajes a una suscripción. Entre otras cosas, esta característica significa que las suscripciones admiten los mismos patrones que se han descrito en esta misma sección con respecto a las colas: consumidor en competencia, desacoplamiento temporal, nivelación de carga y equilibrio de la carga.

### <a name="create-topics-and-subscriptions"></a>Creación de temas y suscripciones
La creación de un tema es similar a la creación de una cola, como se describe en la sección anterior. Puede crear temas y suscripciones mediante [Azure Portal](service-bus-quickstart-topics-subscriptions-portal.md), [PowerShell](service-bus-quickstart-powershell.md), la [CLI](service-bus-tutorial-topics-subscriptions-cli.md) o [plantillas de Resource Manager](service-bus-resource-manager-namespace-topic.md). A continuación, envíe mensajes a un tema y recíbalos de las suscripciones mediante clientes escritos en [C#](service-bus-dotnet-how-to-use-topics-subscriptions.md), [Java](service-bus-java-how-to-use-topics-subscriptions.md), [Python](service-bus-python-how-to-use-topics-subscriptions.md), [JavaScript](service-bus-nodejs-how-to-use-topics-subscriptions.md) y [PHP](service-bus-php-how-to-use-topics-subscriptions.md). 

### <a name="rules-and-actions"></a>Reglas y acciones
En muchos escenarios, los mensajes que tienen características específicas deben procesarse de maneras diferentes. Para permitir este procesamiento, puede configurar suscripciones para buscar los mensajes que tengan las propiedades deseadas y, después, realizar determinadas modificaciones en dichas propiedades. Mientras que las suscripciones del Service Bus ven todos los mensajes enviados al tema, solo se puede copiar un subconjunto de dichos mensajes en la cola de suscripción virtual. Este filtrado se consigue mediante los filtros de suscripción. Dichas modificaciones se denominan **acciones de filtrado**. Cuando se cree una suscripción, podrá proporcionar una expresión de filtro que opere en las propiedades del mensaje. Las propiedades pueden ser del sistema (por ejemplo, **etiqueta**) y propiedades de la aplicación personalizadas (por ejemplo, **StoreName**). En este caso, la expresión de filtro SQL es opcional. Sin una expresión de filtro SQL, todas las acciones de filtro definidas en una suscripción se realizarán en todos los mensajes de dicha suscripción.

Si desea ver un ejemplo funcional completo, consulte el [ejemplo TopicSubscriptionWithRuleOperationsSample](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/GettingStarted/Microsoft.Azure.ServiceBus/TopicSubscriptionWithRuleOperationsSample) en GitHub.

Para más información acerca de los filtros, consulte [Filtros y acciones de temas](topic-filters.md).

## <a name="java-message-service-jms-20-entities"></a>Entidades de Java Message Service (JMS) 2.0
Se puede acceder a las siguientes entidades a través de la API de Java Message Service (JMS) 2.0.

  * Colas temporales
  * Temas temporales
  * Suscripciones duraderas compartidas
  * Suscripciones duraderas no compartidas
  * Suscripciones no duraderas compartidas
  * Suscripciones no duraderas no compartidas

Obtenga más información sobre las [entidades de JMS 2.0](java-message-service-20-entities.md) y sobre cómo [usarlas](how-to-use-java-message-service-20.md).

## <a name="next-steps"></a>Pasos siguientes

Para más información y ejemplos de cómo usar la mensajería de Service Bus, consulte estos temas avanzados:

* [Introducción a la mensajería de Service Bus](service-bus-messaging-overview.md)
* [Inicio rápido: Envío y recepción de mensajes mediante Azure Portal y .NET](service-bus-quickstart-portal.md)
* [Tutorial: Actualización del inventario mediante Azure Portal y temas y suscripciones](service-bus-tutorial-topics-subscriptions-portal.md)


