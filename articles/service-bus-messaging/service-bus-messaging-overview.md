---
title: Información general sobre la mensajería de Azure Service Bus | Microsoft Docs
description: En este artículo se proporciona información general de alto nivel sobre Azure Service Bus, un agente de mensajes de integración empresarial totalmente administrado.
ms.topic: overview
ms.date: 06/11/2021
ms.openlocfilehash: 932fde55bdd315993de5375706f7d5d3621cef6d
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112030720"
---
# <a name="what-is-azure-service-bus"></a>Qué es Azure Service Bus
Microsoft Azure Service Bus es un agente de mensajes empresarial totalmente administrado que incluye colas de mensajes y temas que se pueden publicar y a los que es posible suscribirse. Service Bus se usa para desacoplar aplicaciones de servicios, lo que proporciona las siguientes ventajas:

- Equilibrio de carga del trabajo entre trabajadores paralelos
- Enrutamiento y transferencia de datos de forma segura y control entre los límites de aplicaciones y servicios
- Coordinación del trabajo transaccional que requiere un alto grado de confiabilidad 

## <a name="overview"></a>Información general
Los datos se transfieren entre distintas aplicaciones y servicios mediante **mensajes**. Un mensaje es un contenedor decorado con metadatos que contiene datos. Los datos pueden ser cualquier tipo de información, como datos estructurados codificados con los formatos comunes, como los siguientes: JSON, XML, Apache Avro, texto sin formato.

Algunos escenarios de mensajería comunes son:

* **Mensajería**. Transferir datos empresariales, como pedidos de ventas o compras, diarios o movimientos de inventario.
* **Desacoplar aplicaciones**. Mejore la confiabilidad y la escalabilidad de las aplicaciones y los servicios. No es necesario que el productor y el consumidor estén en línea o disponibles al mismo tiempo. La [carga se redistribuye](/azure/architecture/patterns/queue-based-load-leveling), de tal forma que los picos de tráfico no sobrecarguen un servicio. 
* **Equilibrio de carga**. Permite que varios [consumidores paralelos](/azure/architecture/patterns/competing-consumers) lean una cola al mismo tiempo y que cada uno obtenga de forma segura la propiedad exclusiva de mensajes específicos. 
* **Temas y suscripciones**. Permite relaciones 1:*n* entre [publicadores y suscriptores](/azure/architecture/patterns/publisher-subscriber), lo que hace que los suscriptores puedan seleccionar mensajes concretos de una secuencia de mensajes publicados.
* **Transacciones**. Permite realizar varias operaciones, todas ellas en el ámbito de una transacción atómica. Por ejemplo, se pueden realizar las siguientes operaciones en el ámbito de una transacción.  

    1. Obtener un mensaje de una cola.
    2. Publicar resultados de procesamiento en una o varias colas diferentes.
    3. Mover el mensaje de entrada desde la cola original. 
    
    Los resultados se hacen visibles para los consumidores de nivel inferior solo si son correctos, incluida la liquidación correcta del mensaje de entrada, lo que permite una semántica de procesamiento de una sola vez. Este modelo de transacción es una base sólida para el patrón de [transacciones de compensación](/azure/architecture/patterns/compensating-transaction) en el contexto de una solución mayor. 
* **Sesiones de mensajes**. Implemente la coordinación a gran escala de flujos de trabajo y transferencias multiplexadas que requieran un ordenamiento o un aplazamiento de mensajes estricto.

Si está familiarizado con otros agentes de mensajes, como Apache ActiveMQ, los conceptos de Service Bus son parecidos a los que ya conoce. Como Service Bus es una oferta de plataforma como servicio (PaaS), una diferencia clave es que no es necesario preocuparse por las siguientes acciones. Azure se encarga de esas tareas. 

- Preocuparse por los errores de hardware. 
- Mantener revisados los sistemas operativos o productos.
- Colocar los registros y administrar el espacio en disco.
- Administrar las copias de seguridad.
- Conmutar por error a una máquina de reserva.

## <a name="compliance-with-standards-and-protocols"></a>Cumplimiento de estándares y protocolos

El protocolo de conexión principal de Service Bus es [Advanced Message Queuing Protocol (AMQP) 1.0](service-bus-amqp-overview.md), un estándar abierto ISO/IEC. Este protocolo permite a los clientes escribir aplicaciones que funcionan en agentes de Service Bus y locales, como ActiveMQ o RabbitMQ. En la [guía del protocolo AMQP](service-bus-amqp-protocol-guide.md) se proporciona información detallada en caso de que quiera crear una abstracción de este tipo.

[Service Bus Premium](service-bus-premium-messaging.md) es totalmente compatible con la API [Java Message Service (JMS) 2.0](how-to-use-java-message-service-20.md) de Java/Jakarta EE. Además, Service Bus Standard admite el subconjunto JMS 1.1 centrado en las colas. JMS es una abstracción común de los agentes de mensajes y se integra con muchas aplicaciones y marcos, incluido el conocido marco Spring. Para pasar de otros agentes a Azure Service Bus, solo tiene que volver a crear la topología de colas y temas y cambiar la configuración y las dependencias del proveedor del cliente. Puede encontrar un ejemplo en la [guía de migración de ActiveMQ](migrate-jms-activemq-to-servicebus.md).

## <a name="concepts-and-terminology"></a>Conceptos y terminología 
En esta sección se describen los conceptos y la terminología de Service Bus.

### <a name="queues"></a>Colas
Los mensajes se envían y se reciben desde **colas**. Las colas almacenan mensajes hasta que la aplicación receptora está disponible para recibirlos y procesarlos.

![Cola](./media/service-bus-messaging-overview/about-service-bus-queue.png)

Los mensajes de las colas se ordenan y se les asigna una marca de tiempo a su llegada. Una vez aceptados por el agente, el mensaje siempre se mantiene de forma duradera en un almacenamiento con redundancia triple, distribuido entre las zonas de disponibilidad si el espacio de nombres está habilitado para zonas. Service Bus nunca deja mensajes en memoria o almacenamiento volátil una vez que se han notificado al cliente como aceptados.

Los mensajes se entregan en modo de **extracción** y solo entregan mensajes cuando se solicitan. A diferencia del modelo de sondeo de estado ocupado de otras colas de nube, la operación de extracción puede ser de larga duración y completarse solamente una vez que un mensaje esté disponible. 

### <a name="topics"></a>Temas

También puede usar **temas** para enviar y recibir mensajes. Mientras que una cola se utiliza a menudo para la comunicación punto a punto, los temas son útiles en escenarios de publicación y suscripción.

![Tema](./media/service-bus-messaging-overview/about-service-bus-topic.png)

Los temas pueden tener varias suscripciones independientes asociadas; por lo demás, funcionan exactamente igual que las colas del lado receptor. Un suscriptor a un tema puede recibir una copia de cada mensaje enviado a ese tema. Las suscripciones son entidades con nombre. De forma predeterminada, las suscripciones son duraderas, pero se pueden configurar para que expiren y se eliminen luego automáticamente. Mediante la API de JMS, Service Bus Premium también permite crear suscripciones volátiles que existen mientras dura la conexión.

Puede definir reglas en una suscripción. Una regla de suscripción tiene un **filtro** para definir una condición del mensaje que se va a copiar en la suscripción, y una acción **opcional** que puede modificar los metadatos del mensaje. Para más información, consulte [Filtros y acciones de temas](topic-filters.md). Esta característica es útil en los escenarios siguientes:

- No quiere que una suscripción reciba todos los mensajes enviados a un tema.
- Quiere marcar los mensajes con metadatos adicionales cuando pasan a través de una suscripción.

### <a name="namespaces"></a>Espacios de nombres
Un espacio de nombres es un contenedor de todos los componentes de mensajería (colas y temas). Varias colas y temas pueden estar en un solo espacio de nombres, y los espacios de nombres suelen servir como contenedores de aplicación. 

Un espacio de nombres puede compararse con un servidor en la terminología de otros agentes, pero los conceptos no son directamente equivalentes. Un espacio de nombres de Service Bus es su propio segmento de capacidad de un clúster grande compuesto por docenas de máquinas virtuales activas. Opcionalmente, puede abarcar tres [zonas de disponibilidad de Azure](../availability-zones/az-overview.md). Por lo tanto, obtiene todas las ventajas de disponibilidad y solidez de la ejecución del agente de mensajes a gran escala. Además, no es necesario preocuparse por las complejidades subyacentes. Service Bus es mensajería sin servidor.

## <a name="advanced-concepts"></a>Conceptos avanzados
Service Bus incluye características avanzadas, como sesiones de mensajes, entrega programada y transacciones que permiten resolver problemas de mensajería más complejos. Para más información, consulte [Características avanzadas de Azure Service Bus](advanced-features-overview.md).

## <a name="client-libraries"></a>Bibliotecas de clientes

Todas bibliotecas cliente de Service Bus están disponibles a través de Azure SDK.

- [Azure Service Bus para .NET](/dotnet/api/overview/azure/service-bus?preserve-view=true)
- [Bibliotecas de Azure Service Bus para Java](/java/api/overview/azure/servicebus?preserve-view=true)
- [Proveedor de Azure Service Bus para Java JMS 2.0](how-to-use-java-message-service-20.md)
- [Módulos de Azure Service Bus para JavaScript y TypeScript](/javascript/api/overview/azure/service-bus?preserve-view=true)
- [Bibliotecas de Azure Service Bus para Python](/python/api/overview/azure/servicebus?preserve-view=true)

El [protocolo principal de Azure Service Bus es AMQP 1.0](service-bus-amqp-overview.md) y se puede usar desde cualquier cliente compatible con este protocolo. Varios clientes de AMQP de código abierto tienen ejemplos que demuestran explícitamente la interoperabilidad de Service Bus. Revise la [guía del protocolo AMQP 1.0](service-bus-amqp-protocol-guide.md) para aprender a usar las características de Service Bus directamente con los clientes de AMQP 1.0.

[!INCLUDE [messaging-oss-amqp-stacks.md](../../includes/messaging-oss-amqp-stacks.md)]

## <a name="integration"></a>Integración

Service Bus se integra totalmente con muchos servicios de Microsoft y Azure, por ejemplo:

* [Event Grid](service-bus-to-event-grid-integration-example.md)
* [Logic Apps](../connectors/connectors-create-api-servicebus.md)
* [Funciones de Azure](../azure-functions/functions-bindings-service-bus.md)
* [Power Platform](../connectors/connectors-create-api-servicebus.md)
* [Dynamics 365](/dynamics365/fin-ops-core/dev-itpro/business-events/how-to/how-to-servicebus)
* [Azure Stream Analytics](../stream-analytics/stream-analytics-define-outputs.md)

## <a name="next-steps"></a>Pasos siguientes

Para comenzar a trabajar con la mensajería de Service Bus, consulte los siguientes artículos:

* Para comparar los servicios de mensajería de Azure, consulte [Comparación de los servicios](../event-grid/compare-messaging-services.md?toc=%2fazure%2fservice-bus-messaging%2ftoc.json&bc=%2fazure%2fservice-bus-messaging%2fbreadcrumb%2ftoc.json).
* Pruebe las guías de inicio rápido para [.NET](service-bus-dotnet-get-started-with-queues.md), [Java](service-bus-java-how-to-use-queues.md) o [JMS](service-bus-java-how-to-use-jms-api-amqp.md).
* Para administrar los recursos de Service Bus, consulte [Explorador de Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer/releases).
* Más información sobre los niveles Estándar y Premium y sus precios, consulte [precios de Service Bus](https://azure.microsoft.com/pricing/details/service-bus/).
* Para más información sobre el rendimiento y la latencia del nivel Premium, consulte [Mensajería Premium](https://techcommunity.microsoft.com/t5/Service-Bus-blog/Premium-Messaging-How-fast-is-it/ba-p/370722).