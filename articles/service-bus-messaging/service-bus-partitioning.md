---
title: Creación de temas y colas con particiones de Azure Service Bus | Microsoft Docs
description: Describe cómo realizar la partición de los temas y las colas de Service Bus mediante el uso de varios agentes de mensajes.
ms.topic: article
ms.date: 04/26/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 36b3437d3f5e8b27f38ca414fc8a4b954226e608
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108136812"
---
# <a name="partitioned-queues-and-topics"></a>Temas y colas con particiones

Azure Service Bus emplea varios agentes de mensajes para procesar mensajes y varios almacenes de mensajería para almacenar mensajes. Un único agente de mensajes controla una cola o tema convencional, que se almacena en un almacén de mensajería. Las *particiones* de Service Bus permiten que las colas y temas, o *entidades de mensajería*, se dividan entre varios agentes de mensajes y almacenes de mensajería. Las particiones significan que el rendimiento general de una entidad particionada ya no está limitado por el rendimiento de un solo agente o almacén de mensajería. Además, una interrupción temporal de un almacén de mensajería no hace que una cola o tema con particiones deje de estar disponible. Las colas y los temas con particiones pueden contener todas las características avanzadas de Service Bus, como la compatibilidad con transacciones y sesiones.

> [!NOTE]
> La creación de particiones está disponible en la creación de la entidad para todas las colas y temas de las SKU básicas o estándar. Ya no está disponible para el SKU de mensajería prémium, pero las entidades con particiones que existían anteriormente y que eran compatibles con los espacios de nombres prémium seguirán funcionando tal como se esperaba.
 
No es posible cambiar la opción de particionamiento en una cola o tema existentes; solo puede establecer la opción cuando se crea la entidad.

## <a name="how-it-works"></a>Funcionamiento

Cada cola o tema con particiones consta de varias particiones. Cada partición se almacena en un almacén de mensajería diferente y la controla un agente de mensajes diferente. Cuando se envía un mensaje a una cola o un tema con particiones, Service Bus asigna el mensaje a una de las particiones. La selección se realiza de forma aleatoria mediante Service Bus o una clave de partición que el remitente puede especificar.

Cuando un cliente quiere recibir un mensaje de una cola con particiones, o de una suscripción a un tema con particiones, Service Bus consulta en todas las particiones si hay mensajes y luego devuelve al receptor el primer mensaje que se obtiene de cualquiera de los almacenes de mensajería. Service Bus almacena en caché los demás mensajes y los devuelve cuando recibe solicitudes de recepción adicionales. Un cliente de recepción no es consciente de las particiones; el comportamiento de cara al cliente de una cola o tema con particiones (por ejemplo, lectura, finalización, aplazamiento, mensajes fallidos y captura previa) es idéntico al comportamiento de una entidad regular.

La operación de inspección en una entidad sin particiones siempre devuelve el mensaje más antiguo, pero no en una entidad con particiones. En su lugar, devuelve el mensaje más antiguo en una de las particiones cuyo agente de mensajes respondió primero. No hay garantía de que el mensaje devuelto sea el más antiguo de todas las particiones. 

No hay costos adicionales cuando se envía un mensaje a una cola o tema con particiones o cuando se recibe un mensaje de ellos.

> [!NOTE]
> La operación de inspección devuelve el mensaje más antiguo de la partición en función de su número de secuencia. Para las entidades con particiones, el número de secuencia se emite con respecto a la partición. Para obtener más información, consulte [Secuenciación y marcas de tiempo de los mensajes](../service-bus-messaging/message-sequencing.md).

## <a name="use-of-partition-keys"></a>Uso de claves de partición

Cuando un mensaje se coloca en cola en una cola o tema con particiones, Service Bus comprueba la presencia de una clave de partición. Si encuentra una, selecciona la partición basándose en esa clave. Si no encuentra una clave de partición, selecciona la partición basándose en un algoritmo interno.

### <a name="using-a-partition-key"></a>Uso de una clave de partición

En algunos escenarios, como sesiones o transacciones, es necesario que los mensajes se almacenen en una partición específica. Todos estos escenarios requieren el uso de una clave de partición. Todos los mensajes que usan la misma clave de partición se asignan a la misma partición. Si la partición no está disponible temporalmente, Service Bus devuelve un error.

En función del escenario, se usan diferentes propiedades de mensaje como clave de partición:

**SessionId**: si un mensaje tiene establecida la propiedad del id. de sesión, Service Bus la usa como clave de partición. De esta manera, todos los mensajes que pertenecen a la misma sesión se gestionan por el mismo agente de mensajes. Las sesiones permiten que Service Bus garantice la ordenación de los mensajes, así como la coherencia de los estados de la sesión.

**PartitionKey**: si un mensaje tiene establecida la propiedad de la clave de partición, pero no la propiedad de id. de sesión, Service Bus usa el valor de la propiedad de la clave de partición como clave de partición. Si el mensaje tiene establecidas las propiedades de id. de sesión y de la clave de partición, ambas propiedades deben ser idénticas. Si la propiedad de la clave de partición se establece en un valor distinto del de la propiedad del. id de sesión, Service Bus devuelve una excepción de operación no válida. La propiedad de la clave de partición debe usarse si un remitente envía mensajes transaccionales que no tienen en cuenta la sesión. La clave de partición asegura que todos los mensajes que se envían dentro de una transacción se gestionan por el mismo agente de mensajes.

**MessageId:** si la cola o el tema se crearon con la [característica de detección de duplicados](duplicate-detection.md) y no se han establecido las propiedades del id. de sesión o de la clave de partición, el valor de la propiedad del id. de mensaje actúa como clave de partición. (Las bibliotecas de cliente de Microsoft asignan automáticamente un id. de mensaje si la aplicación emisora no lo hace). En este caso, todas las copias del mismo mensaje se controlan mediante el mismo agente de mensajes. Este identificador permite que Service Bus detecte y elimine los mensajes duplicados. Si la característica de detección de duplicados no está habilitada, Service Bus no considera la propiedad del id. de mensaje como una clave de partición.

### <a name="not-using-a-partition-key"></a>Sin uso de una clave de partición

En ausencia de una clave de partición, Service Bus distribuye los mensajes en modo round-robin a todas las particiones de una cola o tema con particiones. Si la partición elegida no está disponible, Service Bus asigna el mensaje a una partición diferente. De esta manera, la operación de envío se realiza correctamente a pesar de la no disponibilidad temporal de un almacén de mensajería. Sin embargo, no obtendrá el orden garantizado que proporciona una clave de partición.

Para acceder a un análisis más detallado del equilibrio entre la disponibilidad (sin clave de partición) y la coherencia (con una clave de partición), consulte [Disponibilidad y coherencia en Event Hubs](../event-hubs/event-hubs-availability-and-consistency.md). Excepto por el id. de partición que no se expone a los usuarios, esta información se aplica igualmente a las entidades con particiones de Service Bus.

Para dar a Service Bus el tiempo suficiente para poner en cola el mensaje en otra partición, el valor del tiempo de expiración especificado por el cliente que envía el mensaje debe ser superior a 15 segundos. Se recomienda establecer el valor predeterminado de 60 segundos.

Una clave de partición “ancla” un mensaje a una partición específica. Si el almacén de mensajería que contiene esta partición no está disponible, Service Bus devuelve un error. En ausencia de una clave de partición, Service Bus puede elegir una partición diferente y la operación se realiza correctamente. Por lo tanto, se recomienda no suministrar una clave de partición a menos que sea necesario.

## <a name="advanced-topics"></a>Temas avanzados

### <a name="use-transactions-with-partitioned-entities"></a>Uso de transacciones con entidades con particiones

Los mensajes que se envían como parte de una transacción deben especificar una clave de partición. La clave puede ser una de las siguientes propiedades: id. de sesión, clave de partición o id. de mensaje. Todos los mensajes que se envían como parte de la misma transacción deben especificar la misma clave de partición. Si intenta enviar un mensaje sin una clave de partición dentro de una transacción, Service Bus devuelve una excepción de operación no válida. Si intenta enviar varios mensajes dentro de la misma transacción que tienen claves de partición diferentes, Service Bus devuelve una excepción de operación no válida. Por ejemplo:

```csharp
CommittableTransaction committableTransaction = new CommittableTransaction();
using (TransactionScope ts = new TransactionScope(committableTransaction))
{
    Message msg = new Message("This is a message");
    msg.PartitionKey = "myPartitionKey";
    await messageSender.SendAsync(msg); 
    await ts.CompleteAsync();
}
committableTransaction.Commit();
```

Si se establece alguna de las propiedades que sirven de clave de partición, Service Bus ancla el mensaje a una partición. Este comportamiento se produce independientemente de si se usa una transacción o no. Se recomienda que no especifique una clave de partición si no es necesario.

### <a name="use-sessions-with-partitioned-entities"></a>Uso de sesiones con entidades con particiones

Para enviar un mensaje transaccional a un tema o cola que tiene en cuenta la sesión, el mensaje debe tener establecida la propiedad del id. de sesión. Si también se especifica la propiedad de la clave de partición, esta debe ser idéntica a la propiedad del id. de sesión. Si difieren, Service Bus devuelve una excepción de operación no válida.

A diferencia de las colas o temas regulares (sin particiones), no es posible usar una única transacción para enviar varios mensajes a diferentes sesiones. Si se intenta, Service Bus devuelve una excepción de operación no válida. Por ejemplo:

```csharp
CommittableTransaction committableTransaction = new CommittableTransaction();
using (TransactionScope ts = new TransactionScope(committableTransaction))
{
    Message msg = new Message("This is a message");
    msg.SessionId = "mySession";
    await messageSender.SendAsync(msg); 
    await ts.CompleteAsync();
}
committableTransaction.Commit();
```

### <a name="automatic-message-forwarding-with-partitioned-entities"></a>Reenvío automático de mensajes en entidades con particiones

Service Bus de Azure admite el reenvío automático de mensajes desde entidades con particiones, así como a ellas y entre ellas. Puede habilitar esta característica al crear o actualizar colas y suscripciones. Para obtener más información, consulte [Habilitación del desvío de mensajes](enable-auto-forward.md). Si el mensaje especifica una clave de partición (id. de sesión, clave de partición o id. de mensaje), esa clave de partición se usa para la entidad de destino.

## <a name="considerations-and-guidelines"></a>Consideraciones e instrucciones
* **Características de alta coherencia**: si una entidad utiliza características como sesiones, detección de duplicados o control explícito de la clave de creación de particiones, las operaciones de mensajería se enrutarán siempre a particiones específicas. Si cualquiera de estas particiones experimenta un tráfico elevado o el almacén subyacente es incorrecto, se producirá un error en las operaciones y se reducirá la disponibilidad. En general, la coherencia es todavía mucho mayor que en las entidades sin particiones ya que solo un subconjunto del tráfico tiene problemas, en lugar de todo el tráfico. Para obtener más información, consulte este [análisis sobre la disponibilidad y la coherencia](../event-hubs/event-hubs-availability-and-consistency.md).
* **Administración**: las operaciones Create, Update y Delete se deben realizar en todas las particiones de la entidad. Si alguna partición es incorrecta, podría provocar errores en estas operaciones. Para la operación Get, la información, como el número de mensajes, se debe agregar desde todas las particiones. Si alguna partición es incorrecta, se notifica el estado de disponibilidad de la entidad como limitado.
* **Escenarios de bajo volumen de mensajes**: para tales escenarios, especialmente cuando se usa el protocolo HTTP, tendrá que realizar varias operaciones de recepción con el fin de obtener todos los mensajes. Para las solicitudes de recepción, el equipo front-end realiza una operación de recepción en todos las particiones y almacena en caché todas las respuestas recibidas. Una solicitud de recepción posterior en la misma conexión se beneficiaría de este almacenamiento en caché y las latencias de recepción serían menores. Sin embargo, si tiene varias conexiones o utiliza el protocolo HTTP, se establecerá una conexión nueva para cada solicitud. Por lo tanto, no hay ninguna garantía de que llegará al mismo nodo. Si todos los mensajes existentes están bloqueados y almacenados en caché en otro equipo front-end, la operación de recepción devolverá un valor **null**. Finalmente, los mensajes caducan y podrá volver a recibirlos. Se recomienda mantener la conexión HTTP. Cuando se usa la creación de particiones en escenarios de bajo volumen, las operaciones de recepción pueden tardar más de lo esperado. Por lo tanto, se recomienda no usar la creación de particiones en estos escenarios. Elimine las entidades con particiones existentes y vuelva a crearlas con la creación de particiones deshabilitada para mejorar el rendimiento.
* **Examinar o inspeccionar mensajes:** la operación de inspección no siempre devuelve el número de mensajes solicitados. Este comportamiento tiene dos motivos habituales. Uno de ellos es que el tamaño agregado de la colección de mensajes supera el tamaño máximo de 256 KB. Otro motivo es que, en las colas o temas con particiones, es posible que una partición no tenga suficientes mensajes para devolver el número solicitado de mensajes. En general, si una aplicación quiere examinar o inspeccionar un número específico de mensajes, debe llamar a la operación de inspección repetidamente hasta que obtenga ese número de mensajes o no habrá más mensajes para inspeccionar. Para obtener más información, incluidos ejemplos de código, consulte [Exploración de mensajes](message-browsing.md).

## <a name="partitioned-entities-limitations"></a>Limitaciones de las entidades con particiones
En su implementación actual, Service Bus impone las siguientes limitaciones en colas y temas con particiones:

* Los temas y las colas con particiones no se admiten en el nivel de mensajería prémium. Se admiten las sesiones en el nivel premier mediante el uso de SessionId. 
* Las colas y los temas con particiones no admiten el envío de mensajes que pertenecen a sesiones diferentes en una sola transacción.
* Actualmente Service Bus permite hasta 100 colas o temas particionados por espacio de nombres. Cada cola o tema con particiones se tiene en cuenta para la cuota de 10 000 entidades por espacio de nombres (no se aplica al nivel prémium).

## <a name="other-features"></a>Otras características
- La adición o eliminación de una regla se admite con entidades con particiones, pero no en transacciones. 
- Asimismo, se admite AMQP para enviar mensajes a una entidad con particiones o recibirlos de ella.
- También se admite AMQP para las siguientes operaciones: Batch Send, Batch Receive, Receive by Sequence Number, Peek, Renew Lock, Schedule Message, Cancel Scheduled Message, Add Rule, Remove Rule, Session Renew Lock, Set Session State, Get Session State y Enumerate Sessions.



## <a name="next-steps"></a>Pasos siguientes
Puede habilitar la partición mediante Azure Portal, PowerShell, CLI, la plantilla de Resource Manager, .NET, Java, Python y JavaScript. Para obtener más información, consulte [Habilitar la partición](enable-partitions.md). 

Obtenga información sobre los conceptos básicos de la especificación de mensajería de AMQP 1.0 en la [guía del protocolo AMQP 1.0](service-bus-amqp-protocol-guide.md).

[Azure portal]: https://portal.azure.com
[QueueDescription.EnablePartitioning]: /dotnet/api/microsoft.servicebus.messaging.queuedescription.enablepartitioning
[TopicDescription.EnablePartitioning]: /dotnet/api/microsoft.servicebus.messaging.topicdescription.enablepartitioning
[QueueDescription.ForwardTo]: /dotnet/api/microsoft.servicebus.messaging.queuedescription.forwardto
[AMQP 1.0 support for Service Bus partitioned queues and topics]: ./service-bus-amqp-protocol-guide.md
