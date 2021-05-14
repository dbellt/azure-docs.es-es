---
title: Información general de las características de Azure Event Hubs | Microsoft Docs
description: En este artículo se proporcionan detalles acerca de las características y la terminología de Azure Event Hubs.
ms.topic: article
ms.date: 03/15/2021
ms.openlocfilehash: 4fdcee27cd414069572e996f31de37d1ae641d13
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107988449"
---
# <a name="features-and-terminology-in-azure-event-hubs"></a>Características y terminología de Azure Event Hubs

Azure Event Hubs es un servicio escalable de procesamiento de eventos que recopila y procesa grandes volúmenes de eventos y datos, con una baja latencia y una alta fiabilidad. Consulte [¿Qué es Event Hubs?](./event-hubs-about.md) para obtener una introducción detallada.

Este artículo se basa en el contenido del [artículo de información general](./event-hubs-about.md) e incluye detalles técnicos y de implementación de las características y los componentes de Event Hubs.

> [!TIP]
> [La compatibilidad del protocolo con los clientes de **Apache Kafka**](event-hubs-for-kafka-ecosystem-overview.md) (versiones > = 1.0) proporciona puntos de conexión de red que permiten que las aplicaciones compiladas usen Apache Kafka con cualquier cliente para utilizar Event Hubs. La mayoría de las aplicaciones de Kafka existentes se pueden volver a configurar simplemente para que apunten a un espacio de nombres del centro de eventos en lugar de a un servidor de arranque del clúster de Kafka. 
>
>Desde la perspectiva del costo, el esfuerzo operativo y la confiabilidad, Azure Event Hubs es una excelente alternativa a la implementación y el uso de sus propios clústeres de Kafka y Zookeeper y a las ofertas de Kafka como servicio no nativas para Azure. 
>
> Además de conseguir la misma funcionalidad básica que la del agente de Apache Kafka, también se obtiene acceso a las características del centro de eventos de Azure, como el procesamiento por lotes y el archivado automáticos a través de [Event Hubs Capture](event-hubs-capture-overview.md), el equilibrio y el escalado automáticos, la recuperación ante desastres, la compatibilidad con zonas de disponibilidad sin costos adicionales, la integración de red flexible y segura y la compatibilidad con varios protocolos, como el protocolo compatible con firewall AMQP sobre WebSockets.


## <a name="namespace"></a>Espacio de nombres
Los espacios de nombres de Event Hubs proporcionan puntos de conexión de red integrados de DNS y una variedad de características de administración de control de acceso e integración de red, como [filtrado de IP](event-hubs-ip-filtering.md), [punto de conexión de red virtual](event-hubs-service-endpoints.md) y [Private Link](private-link-service.md), y son contenedor de administración de una de numerosas instancias de Event Hubs (o temas, en el lenguaje Kafka).

## <a name="event-publishers"></a>Publicadores de eventos

Toda entidad que envíe datos a un centro de eventos es un *publicador de eventos* (que se usa como sinónimo de *productor de eventos*). Los publicadores de eventos pueden publicar eventos mediante HTTPS o AMQP 1.0 o el protocolo de Kafka. Para obtener acceso de publicación, los publicadores de eventos usan la autorización basada en Azure Active Directory con tokens JWT emitidos por OAuth2 o un token de firma de acceso compartido (SAS) específico del centro de eventos.

### <a name="publishing-an-event"></a>Publicación de un evento

Puede publicar un evento a través de AMQP 1.0, el protocolo de Kafka o HTTPS. El servicio Event Hubs proporciona [API REST](/rest/api/eventhub/) y bibliotecas cliente para [.NET](event-hubs-dotnet-standard-getstarted-send.md), [Java](event-hubs-java-get-started-send.md), [Python](event-hubs-python-get-started-send.md), [JavaScript](event-hubs-node-get-started-send.md) y [Go](event-hubs-go-get-started-send.md) para publicar eventos en un centro de eventos. Para otras plataformas y tiempos de ejecución, puede usar cualquier cliente de AMQP 1.0, como [Apache Qpid](https://qpid.apache.org/). 

La opción de usar AMQP o HTTPS es específica para el escenario de uso. AMQP requiere el establecimiento de un socket bidireccional persistente, además de la seguridad de nivel de transporte (TLS) o SSL/TLS. AMQP tiene un mayor costo de red al inicializar la sesión, sin embargo, HTTPS requiere una sobrecarga de TLS adicional para cada solicitud. AMQP presenta un rendimiento bastante mayor para publicadores frecuentes y puede lograr latencias mucho menores cuando se usa con código de publicación asincrónico.

Los eventos se pueden publicar de forma individual o por lotes. Una sola publicación tiene un límite de 1 MB, con independencia de si es un evento único o un lote. La publicación de eventos que superen este umbral se rechazará. 

El rendimiento de Event Hubs se ajusta mediante particiones y asignaciones de unidades de rendimiento (consulte a continuación). Se recomienda que los publicadores no estén informados del modelo de particionamiento específico elegido para un centro de eventos y que solo especifiquen una *clave de partición* que se usa para asignar de forma coherente eventos relacionados a la misma partición.

![Claves de partición](./media/event-hubs-features/partition_keys.png)

Event Hubs garantiza que todos los eventos que comparten un valor de clave de partición se almacenen juntos y se entreguen en el orden de llegada. Si se usan claves de partición con directivas de publicador, la identidad del publicador y el valor de la clave de partición deben coincidir. De lo contrario, se produce un error.

### <a name="event-retention"></a>Retención de eventos

Los eventos publicados se quitan de un centro de eventos en función de una directiva de retención configurable basada en el tiempo. Estos son algunos puntos importantes:

- El valor **predeterminado** y el período de retención **más corto** posible es de **1 día (24 horas)** .
- En el caso de Event Hubs **estándar**, el período de retención máximo es de **7 días**. 
- En el caso de Event Hubs **dedicado**, el período de retención máximo es de **90 días**.
- Si cambia el período de retención, se aplica a todos los mensajes, incluidos los que ya están en el centro de eventos. 

Event Hubs retiene los eventos durante el tiempo de retención configurado, que se aplica a todas las particiones. Los eventos se eliminan automáticamente cuando se alcanza el período de retención. Si especifica un período de retención de un día, el evento dejará de estar disponible exactamente 24 horas después de que se haya aceptado. No se pueden eliminar eventos explícitamente. 

Si necesita archivar eventos más allá del período de retención permitido, puede hacer que se [almacenen automáticamente en Azure Storage o Azure Data Lake mediante la activación de la característica Capture de Event Hubs](event-hubs-capture-overview.md) y, si necesita buscar o analizar estos archivos, puede [importarlos fácilmente en Azure Synapse](store-captured-data-data-warehouse.md) u otros almacenes y plataformas de análisis similares. 

La razón del límite de Event Hubs en la retención de datos basada en el tiempo es evitar que grandes volúmenes de datos históricos del cliente se mantengan en un almacén que solo está indexado por una marca de tiempo y solo permite el acceso secuencial. La filosofía arquitectónica aquí es que los datos históricos necesitan una indexación más enriquecida y un acceso más directo que la interfaz de eventos en tiempo real que proporcionan Event Hubs o Kafka. Los motores de flujos de eventos no son adecuados para desempeñar el papel de los lagos de datos o los archivos a largo plazo para la creación de orígenes de eventos. 
 

> [!NOTE]
> Event Hubs es un motor de secuencia de eventos en tiempo real y no está diseñado para usarse en lugar de una base de datos o como almacén permanente para secuencias de eventos que se conservan por tiempo indefinido. 
> 
> Cuanto más profundo sea el historial de una secuencia de eventos, más se necesitarán índices auxiliares para encontrar un segmento histórico determinado de una secuencia dada. La inspección de cargas de eventos y la indexación no se encuentran dentro del ámbito de características de Event Hubs (o Apache Kafka). Las bases de datos y los almacenes y motores de análisis especializados, como [Azure Data Lake Store](../data-lake-store/data-lake-store-overview.md), [Azure Data Lake Analytics](../data-lake-analytics/data-lake-analytics-overview.md) y [Azure Synapse](../synapse-analytics/overview-what-is.md) son, por lo tanto, más adecuados para almacenar eventos históricos.
>
> [Event Hubs Capture](event-hubs-capture-overview.md) se integra directamente con Azure Blob Storage y Azure Data Lake Storage y, gracias a esa integración, también permite el [flujo de eventos directamente a Azure Synapse](store-captured-data-data-warehouse.md).
>
> Si quiere usar el patrón [Origen de eventos](/azure/architecture/patterns/event-sourcing) para la aplicación, debe alinear la estrategia de instantáneas con los límites de retención de Event Hubs. Su intención no debe ser la de recompilar las vistas materializadas de eventos sin procesar que se inician al principio. Probablemente lamentaría una estrategia así una vez que la aplicación esté en producción durante un tiempo y se use correctamente y el generador de proyecciones tenga que rebuscar entre años de eventos de cambios al intentar ponerse al día con los más recientes y continuos. 


### <a name="publisher-policy"></a>Directiva del publicador

Los Event Hubs permiten un control granular sobre los publicadores de eventos a través de las *directivas de publicador*. Las directivas de publicador son características de tiempo de ejecución diseñadas para facilitar grandes números de publicadores de eventos independientes. Con las directivas de publicador, cada publicador usa su propio identificador único al publicar los eventos en un centro de eventos mediante el mecanismo siguiente:

```http
//<my namespace>.servicebus.windows.net/<event hub name>/publishers/<my publisher name>
```

No tiene que crear nombres de publicador con antelación, pero deben coincidir con el token de SAS que se usa al publicar un evento, con el fin de garantizar las identidades de publicador independientes. Al usar directivas de publicador, el valor **PartitionKey** se establece como el nombre del publicador. Para que funcione correctamente, estos valores deben coincidir.

## <a name="capture"></a>Capturar

[Event Hubs Capture](event-hubs-capture-overview.md) permite capturar automáticamente los datos de transmisión de Event Hubs y guardarlos en una cuenta de Blob Storage o en una cuenta de servicio de Azure Data Lake. Puede habilitar Capture desde Azure Portal y especificar una ventana de tiempo y de tamaño mínimos para realizar la captura. Event Hubs Capture permite especificar una cuenta y un contenedor propios de Azure Blob Storage, o una cuenta de servicio de Azure Data Lake, uno de los cuales se usa para almacenar los datos capturados. Los datos capturados se escriben en el formato de Apache Avro.

## <a name="partitions"></a>Particiones
[!INCLUDE [event-hubs-partitions](../../includes/event-hubs-partitions.md)]


## <a name="sas-tokens"></a>Tokens de SAS

Event Hubs usa *firmas de acceso compartido* que están disponibles en el nivel del espacio de nombres y del centro de eventos. Un token de SAS se genera a partir de una clave de SAS y es un hash SHA de una dirección URL, codificado en un formato concreto. Con el nombre de la clave (directiva) y el token, Event Hubs puede volver a generar el hash y así autenticar al remitente. Normalmente, los tokens de SAS para publicadores de eventos se crean solo con privilegios de **envío** en un centro de eventos concreto. Este mecanismo de dirección URL del token de SAS es la base para la identificación del publicador introducida en la directiva del publicador. Para obtener más información sobre el funcionamiento con SAS, consulte [Autenticación con firma de acceso compartido en Service Bus](../service-bus-messaging/service-bus-sas.md).

## <a name="event-consumers"></a>Consumidores de eventos

Cualquier entidad que lea datos de eventos de un centro de eventos es un *consumidor de eventos*. Todos los consumidores de Event Hubs se conectan a través de la sesión de AMQP 1.0, y los eventos se entregan a través de la sesión a medida que están disponibles. El cliente no necesita realizar un sondeo de disponibilidad de los datos.

### <a name="consumer-groups"></a>Grupos de consumidores

El mecanismo de publicación y suscripción de Event Hubs se habilita a través de los *grupos de consumidores*. Un grupo de consumidores es una vista (estado, posición o desplazamiento) de un centro de eventos completo. Los grupos de consumidores habilitan varias aplicaciones consumidoras para que cada una tenga una vista separada del flujo de eventos y para que lean el flujo de forma independiente a su propio ritmo y con sus propios desplazamientos.

En una arquitectura de procesamiento de flujos, cada aplicación de bajada se corresponde con un grupo de consumidores. Si quiere escribir datos de eventos para el almacenamiento a largo plazo, esa aplicación de escritura de almacenamiento es un grupo de consumidores. Otro grupo de consumidores independiente puede realizar el procesamiento de eventos complejos. Solo puede obtener acceso a las particiones a través de un grupo de consumidores. Siempre hay un grupo de consumidores predeterminado en un centro de eventos y puede crear hasta 20 grupos de consumidores para un centro de eventos de nivel Estándar.

Como máximo, puede haber cinco lectores simultáneos en una partición por grupo de consumidores; pero **se recomienda que solo haya un receptor activo en una partición por grupo de consumidores**. Cada lector recibe todos los mensajes dentro de una sola partición. Si tiene varios lectores en la misma partición, procesará los mensajes duplicados. Debe controlar esto en su código, pues no puede ser trivial. Sin embargo, es un enfoque válido en algunos escenarios.

Algunos clientes que ofrecen los SDK de Azure son agentes de consumidor inteligentes que administran automáticamente los detalles para asegurarse de que cada partición tenga un lector único y que se estén leyendo todas las particiones para un centro de eventos. Esto permite que el código se centre en el procesamiento de los eventos que se leen desde el centro de eventos de modo que pueda omitir muchos detalles de las particiones. Para más información, consulte [Conexión a una partición](#connect-to-a-partition).

A continuación se muestran ejemplos de la convención de URI del grupo de consumidores:

```http
//<my namespace>.servicebus.windows.net/<event hub name>/<Consumer Group #1>
//<my namespace>.servicebus.windows.net/<event hub name>/<Consumer Group #2>
```

La siguiente ilustración muestra la arquitectura de procesamiento del flujo de Event Hubs:

![Arquitectura de Event Hubs](./media/event-hubs-about/event_hubs_architecture.svg)

### <a name="stream-offsets"></a>Desplazamientos de los flujos

Un *desplazamiento* es la posición de un evento dentro de una partición. Puede pensar en un desplazamiento como un cursor de lado cliente. El desplazamiento es una numeración de byte del evento. Este desplazamiento permite que un consumidor de eventos (lector) especifique un punto en el flujo de eventos desde el que quiere empezar a leer los eventos. Puede especificar el desplazamiento como una marca de tiempo o como un valor de desplazamiento. Los consumidores son responsables de almacenar sus propios valores de desplazamiento fuera del servicio de Event Hubs. Dentro de una partición, cada evento incluye un desplazamiento.

![Desplazamiento de partición](./media/event-hubs-features/partition_offset.png)

### <a name="checkpointing"></a>Puntos de control

*Puntos de control* es un proceso en el que los lectores marcan o confirman su posición dentro de la secuencia de eventos de una partición. La creación de puntos de comprobación es responsabilidad del consumidor y se realiza por partición dentro de un grupo de consumidores. Esta responsaibilidad significa que por cada grupo de consumidores, cada lector de la partición debe realizar un seguimiento de su posición actual en el flujo del evento y puede informar al servicio cuando considere que el flujo de datos se ha completado.

Si se desconecta un lector de una partición, cuando se vuelve a conectar comienza a leer en el punto de comprobación que envió previamente el último lector de esa partición en ese grupo de consumidores. Cuando se conecta el lector, pasa este desplazamiento al centro de eventos para especificar la ubicación en la que se va a empezar a leer. De este modo, puede usar puntos de comprobación para marcar eventos como "completados" por las aplicaciones de bajada y para ofrecer resistencia en caso de que se produzca una conmutación por error entre lectores que se ejecutan en máquinas distintas. Es posible volver a los datos más antiguos especificando un desplazamiento inferior desde este proceso de puntos de comprobación. Mediante este mecanismo, los puntos de comprobación permiten una resistencia a la conmutación por error y una reproducción del flujo de eventos.

> [!IMPORTANT]
> El servicio Event Hubs proporciona desplazamientos. Es responsabilidad del consumidor crear puntos de comprobación a medida que se procesan los eventos.

> [!NOTE]
> Si usa Azure Blob Storage como el almacén de puntos de comprobación en un entorno que admite una versión diferente del SDK de blobs de almacenamiento que las que normalmente están disponibles en Azure, tendrá que utilizar código para cambiar la versión de la API del servicio de almacenamiento a la versión admitida por ese entorno. Por ejemplo, si ejecuta [Event Hubs en una instancia de Azure Stack Hub versión 2002](/azure-stack/user/event-hubs-overview), la versión más alta disponible para el servicio Storage es 2017-11-09. En este caso, tendrá que usar código para establecer como destino la versión de la API del servicio Storage en 2017-11-09. Para obtener un ejemplo de cómo establecer como destino una versión específica de la API de Storage, vea estos ejemplos en GitHub: 
> - [.NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Azure.Messaging.EventHubs.Processor/samples/) 
> - [Java](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs-checkpointstore-blob/src/samples/java/com/azure/messaging/eventhubs/checkpointstore/blob/)
> - [JavaScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/javascript) o [TypeScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/typescript)
> - [Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub-checkpointstoreblob-aio/samples/)

### <a name="common-consumer-tasks"></a>Tareas comunes del consumidor

Todos los consumidores de Event Hubs se conectan a través de una sesión de AMQP 1.0, un canal de comunicación bidireccional con estado. Cada partición tiene una sesión de AMQP 1.0 que facilita el transporte de eventos que deben separarse por partición.

#### <a name="connect-to-a-partition"></a>Conexión a una partición

Es una práctica habitual al conectarse a particiones usar un mecanismo de concesiones para coordinar las conexiones del lector a particiones concretas. De este modo, es posible que cada partición de un grupo de consumidores solo tenga un lector activo. Los puntos de comprobación, la concesión y la administración de lectores se simplifican mediante el uso de los clientes de los SDK de Event Hubs, que actúan como agentes de consumidor inteligentes. Dichos componentes son:

- [EventProcessorClient](/dotnet/api/azure.messaging.eventhubs.eventprocessorclient) para .NET
- [EventProcessorClient](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs/src/main/java/com/azure/messaging/eventhubs/EventProcessorClient.java) para Java
- [EventHubConsumerClient](/python/api/azure-eventhub/azure.eventhub.aio.eventhubconsumerclient) para Python
- [EventHubSoncumerClient](/javascript/api/@azure/event-hubs/eventhubconsumerclient) para JavaScript o TypeScript

#### <a name="read-events"></a>Lectura de eventos

Después de abrir una sesión de AMQP 1.0 y el vínculo de una partición específica, el servicio de Centros de eventos entrega los eventos al cliente de AMQP 1.0. Este mecanismo de entrega permite un mayor procesamiento y una menor latencia que los mecanismos basados en extracción como HTTP GET. Los eventos se envían al cliente, cada instancia de datos de eventos contiene metadatos importantes, como el número de secuencia y el desplazamiento que se usan para facilitar la creación de puntos de comprobación en la secuencia de eventos.

Datos de evento:
* Offset
* Número de secuencia
* Body
* Propiedades de usuario
* Propiedades del sistema

Es su responsabilidad administrar el desplazamiento.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de Event Hubs, visite los vínculos siguientes:

- Introducción a Event Hubs
    - [.NET](event-hubs-dotnet-standard-getstarted-send.md)
    - [Java](event-hubs-java-get-started-send.md)
    - [Python](event-hubs-python-get-started-send.md)
    - [JavaScript](event-hubs-node-get-started-send.md)
* [Guía de programación de Event Hubs](event-hubs-programming-guide.md)
* [Disponibilidad y coherencia en Event Hubs](event-hubs-availability-and-consistency.md)
* [Preguntas más frecuentes sobre Event Hubs](event-hubs-faq.yml)
* [Ejemplos de Event Hubs](event-hubs-samples.md)
