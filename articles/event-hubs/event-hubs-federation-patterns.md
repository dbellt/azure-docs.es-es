---
title: 'Patrones de tareas de replicación de eventos: Azure Event Hubs | Microsoft Docs'
description: En este artículo se proporcionan instrucciones detalladas para implementar patrones de tareas de replicación de eventos específicos.
ms.topic: article
ms.date: 12/12/2020
ms.openlocfilehash: adaa124470e6b3cc94bab647967f08b63a1c4f49
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110462859"
---
# <a name="event-replication-tasks-patterns"></a>Patrones de tareas de replicación de eventos

En las páginas de [información general sobre la federación](event-hubs-federation-overview.md) e [información general sobre las funciones del replicador](event-hubs-federation-replicator-functions.md) se explica la lógica y los elementos básicos de las tareas de replicación, y se recomienda que se familiarice con ellos antes de continuar con este artículo.

En este artículo, se detallan las instrucciones de implementación de algunos de los patrones resaltados en la sección de información general.

## <a name="replication"></a>Replicación

El patrón de replicación copia los eventos de un centro de eventos al siguiente, o de un centro de eventos a otro destino, como una cola de Service Bus. Los eventos se reenvían sin realizar ninguna modificación en la carga del evento.

La implementación de este patrón está incluida en los ejemplos de [replicación de eventos entre Event Hubs](https://github.com/Azure-Samples/azure-messaging-replication-dotnet/tree/main/functions/config/EventHubCopy) y [replicación de eventos entre Event Hubs y Service Bus](https://github.com/Azure-Samples/azure-messaging-replication-dotnet/tree/main/functions/config/EventHubCopyToServiceBus) y en el tutorial [Uso de Apache Kafka MirrorMaker con Event Hubs](event-hubs-kafka-mirror-maker-tutorial.md) para el caso específico de replicación de datos de un agente de Apache Kafka en Event Hubs.

### <a name="streams-and-order-preservation"></a>Secuencias y conservación del orden

La replicación, ya sea a través de Azure Functions o Azure Stream Analytics, no pretende garantizar la creación de clones exactos de 1:1 de un centro de eventos de origen en un centro de eventos de destino, sino que se centra en conservar el orden relativo de los eventos en los que la aplicación lo requiere. La aplicación se comunica mediante la agrupación de eventos relacionados con la misma clave de partición y [Event Hubs organiza los mensajes con la misma clave de partición secuencialmente en la misma partición](event-hubs-features.md#partitions).

> [!IMPORTANT] 
> La información de "desplazamiento" es única para cada centro de eventos y los desplazamientos de los mismos eventos variarán en las instancias del centro de eventos. Para buscar una posición en una secuencia de eventos copiada, use desplazamientos basados en el tiempo y haga referencia a los [metadatos asignados al servicio propagado](#service-assigned-metadata).
>
> Los desplazamientos basados en el tiempo inician el receptor en un momento específico:
> - *EventPosition.FromStart()* : leer todos los datos retenidos de nuevo.
> - *EventPosition.FromEnd()* : leer todos los datos nuevos desde el momento de la conexión.
> - *EventPosition.FromEnqueuedTime(dateTime)* : todos los datos a partir de una fecha y hora determinadas.
>
> En EventProcessor, establezca la posición a través de InitialOffsetProvider en EventProcessorOptions. Con las otras API del receptor, la posición se pasa a través del constructor. 


Los asistentes de funciones de replicación precompiladas [proporcionados como ejemplos](https://github.com/Azure-Samples/azure-messaging-replication-dotnet/tree/main/src/Azure.Messaging.Replication) que se usan en la guía basada en Azure Functions, garantizan que las secuencias de eventos con la misma clave de partición recuperada de una partición de origen se envíen al centro de eventos de destino como un lote en la secuencia original y con la misma clave de partición.

Si el número de particiones del centro de eventos de origen y de destino es idéntico, todas las secuencias del destino se asignarán a las mismas particiones que en el origen.
Si el número de particiones es diferente, lo que es importante en algunos de los patrones que se describen a continuación, la asignación será diferente, pero las secuencias siempre se mantienen juntas y en orden.

El orden relativo de los eventos que pertenecen a diferentes secuencias o de eventos independientes sin una clave de partición en una partición de destino puede ser siempre diferente de la partición de origen.

### <a name="service-assigned-metadata"></a>Metadatos asignados por el servicio

Los metadatos asignados por el servicio de un evento obtenido del centro de eventos de origen, el tiempo de puesta en cola original, el número de secuencia y el desplazamiento, se sustituyen por nuevos valores asignados por el servicio en el centro de eventos de destino, pero con las [funciones auxiliares](https://github.com/Azure-Samples/azure-messaging-replication-dotnet/tree/main/src/Azure.Messaging.Replication), las tareas de replicación que se proporcionan en nuestros ejemplos, los valores originales se conservan en las propiedades del usuario: `repl-enqueue-time` (cadena ISO8601), `repl-sequence`, `repl-offset`.

Dichas propiedades son de tipo cadena y contienen el valor en cadena de las propiedades originales respectivas. Si el evento se reenvía varias veces, los metadatos asignados por el servicio del origen inmediato se anexan a las propiedades ya existentes, con los valores separados por punto y coma.

### <a name="failover"></a>Conmutación por error

Si usa la replicación para la recuperación ante desastres, para protegerse de los eventos de disponibilidad regional en el servicio de Event Hubs, o de interrupciones de la red, cualquier escenario de error requiere la realización de una conmutación por error de un centro de eventos al siguiente, lo que indica a los productores o consumidores que usen el punto de conexión secundario.

En todos los escenarios de conmutación por error, se supone que los elementos necesarios de los espacios de nombres son estructuralmente idénticos, lo que significa que Event Hubs y los grupos de consumidores tienen un nombre idéntico y que las reglas de firma de acceso compartido o las reglas de control de acceso basado en roles se configuran de la misma manera. Para crear (y actualizar) un espacio de nombres secundario, siga la [guía para mover espacios de nombres](move-across-regions.md) y omita el paso de limpieza.

Para obligar a los productores y consumidores a realizar el cambio, debe proporcionar la información sobre qué espacios de nombres usar para realizar la búsqueda en una ubicación con la que se pueda establecer contacto fácilmente y que sea fácil de actualizar. Si los productores o los consumidores encuentran errores frecuentes o persistentes, deben consultar esa ubicación y ajustar su configuración. Hay muchas maneras de compartir esa configuración, pero se indican dos a continuación: DNS y recursos compartidos.

#### <a name="dns-based-failover-configuration"></a>Configuración de conmutación por error basada en DNS

Un posible enfoque consiste en mantener la información de los registros SRV de DNS en un DNS que controla y apunta a los puntos de conexión respectivos del centro de eventos. 

> [!IMPORTANT] 
> Tenga en cuenta que Event Hubs no permite que sus puntos de conexión tengan alias directamente con registros CNAME, lo que significa que usará DNS como mecanismo de búsqueda resistente para las direcciones de punto de conexión y no para resolver directamente la información de dirección IP.

Supongamos que posee el dominio `example.com` y, para su aplicación, una zona `test.example.com`. En el caso de dos instancias de Event Hubs alternativas, ahora creará dos zonas anidadas adicionales y un registro SRV en cada una.

A los registros SRV, siguiendo la convención común, se les aplica el prefijo `_azure_eventhubs._amqp` y contienen dos registros de punto de conexión: uno para AMQP mediante TLS en el puerto 5671 y otro para AMQP mediante WebSockets en el puerto 443, ambos apuntando al punto de conexión de Event Hubs del espacio de nombres correspondiente a la zona.

| Zona                   | Registro SRV                                                                                                                                                            |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `eh1.test.example.com` | **`_azure_servicebus._amqp.eh1.test.example.com`**<br>`1 1 5671 eh1-test-example-com.servicebus.windows.net`<br>`2 2 443 eh1-test-example-com.servicebus.windows.net` |
| `eh2.test.example.com` | **`_azure_servicebus._amqp.eh2.test.example.com`**<br>`1 1 5671 eh2-test-example-com.servicebus.windows.net`<br>`2 2 443 eh2-test-example-com.servicebus.windows.net` |

En la zona de la aplicación, creará una entrada CNAME que apunte a la zona subordinada correspondiente al centro de eventos principal:

| Registro CNAME                | Alias                    |
| --------------------------- | ------------------------ |
| `eventhub.test.example.com` | `eh1.test.example.com`   |

El uso de un cliente DNS que permite consultar los registros CNAME y SRV explícitamente (los clientes integrados de Java y .NET solo permiten la resolución simple de nombres en direcciones IP), puede resolver el punto de conexión deseado. Con [DnsClient.NET](https://dnsclient.michaco.net/), por ejemplo, la función de búsqueda es:

```C#
static string GetEventHubName(string aliasName)
{
    const string SrvRecordPrefix = "_azure_eventhub._amqp.";
    LookupClient lookup = new LookupClient();

    return (from CNameRecord alias in (lookup.Query(aliasName, QueryType.CNAME).Answers)
            from SrvRecord srv in lookup.Query(SrvRecordPrefix + alias.CanonicalName, QueryType.SRV).Answers
            where srv.Port == 5671
            select srv.Target).FirstOrDefault()?.Value.TrimEnd('.');
}
```

La función devuelve el nombre de host de destino registrado para el puerto 5671 de la zona que tiene un alias actualmente con el CNAME como se mostró anteriormente.

Para realizar una conmutación por error es necesario editar el registro CNAME y apuntarlo a la zona alternativa.

La ventaja de usar DNS y, en concreto [Azure DNS](../dns/dns-overview.md), es que la información de Azure DNS se replica globalmente y, por tanto, es resistente a las interrupciones de una sola región.

Este procedimiento es similar al funcionamiento de la [recuperación ante desastres geográfica de Event Hubs](event-hubs-geo-dr.md), pero totalmente bajo su propio control y también funciona con escenarios activos/activos.

#### <a name="file-share-based-failover-configuration"></a>Configuración de conmutación por error basada en recursos compartidos

La alternativa más sencilla a usar DNS para compartir la información del punto de conexión es poner el nombre del punto de conexión principal en un archivo de texto sin formato y servir el archivo desde una infraestructura que sea sólida frente a las interrupciones y, además, permitir actualizaciones.

Si ya ha ejecutado una infraestructura de sitios web de alta disponibilidad con replicación de contenido y disponibilidad global, agregue dicho archivo y vuelva a publicarlo si se necesita un cambio.

> [!CAUTION]
> Solo debe publicar el nombre del punto de conexión de esta manera, no una cadena de conexión completa que incluya los secretos.

#### <a name="extra-considerations-for-failing-over-consumers"></a>Consideraciones adicionales para la conmutación por error de los consumidores

En el caso de los consumidores del centro de eventos, las consideraciones adicionales para la estrategia de conmutación por error dependen de las necesidades del procesador de eventos.

Si se produce un desastre que requiere recompilar un sistema, incluidas las bases de datos, desde los datos de copia de seguridad, y las bases de datos se alimentan directamente o a través del procesamiento intermedio de los eventos contenidos en el centro de eventos, debe restaurar la copia de seguridad y, a continuación, iniciar la reproducción de eventos en el sistema desde el momento en que se creó la copia de seguridad de la base de datos y no desde el momento en que se destruyó el sistema original.

Si un error solo afecta a un segmento de un sistema o, en realidad, a un solo centro de eventos, que se ha vuelto inaccesible, es probable que quiera seguir procesando eventos aproximadamente desde la misma posición en la que se interrumpió el procesamiento.

Para llevar a cabo cualquier escenario y usar el procesador de eventos de la instancia respectiva de Azure SDK, [creará un nuevo almacén de puntos de control](event-processor-balance-partition-load.md#checkpointing) y proporcionará una posición inicial de la partición en función de la _marca de tiempo_ de la que quiere reanudar el procesamiento.

Si todavía tiene acceso al almacén de puntos de control del centro de eventos desde el que se va a cambiar, los [metadatos propagados](#service-assigned-metadata) descritos anteriormente le ayudarán a omitir los eventos que ya se han controlado y se reanudarán precisamente desde donde se quedó por última vez.

## <a name="merge"></a>Merge

El patrón de combinación tiene una o varias tareas de replicación que apuntan a un destino, posiblemente al mismo tiempo que productores normales que también envían eventos al mismo destino.

Estas son las variaciones de estos patrones:

- Dos o más funciones de replicación que adquieren al mismo tiempo eventos de orígenes independientes y los envían al mismo destino.
- Una función de replicación más que adquiere eventos de un origen mientras que el destino también lo usan directamente los productores.
- El patrón anterior, salvo los eventos reflejados entre dos o más instancias de Event Hubs, lo que genera las instancias de Event Hubs que contienen las mismas secuencias, independientemente de dónde se produzcan los eventos.

Las dos primeras variaciones de patrón son triviales y no difieren de las tareas de replicación sin formato.

El último escenario requiere la exclusión de la replicación de los eventos que ya se han replicado. La técnica se muestra y se explica en el ejemplo [EventHubToEventHubMerge](https://github.com/Azure-Samples/azure-messaging-replication-dotnet/tree/main/functions/code/EventHubMerge).

## <a name="editor"></a>Editor

El patrón de editor se basa en el patrón de [replicación](#replication), pero los mensajes se modifican antes de reenviarse. 

Estos son algunos ejemplos de esas modificaciones:

- **_Transcodificación_**: si el contenido del evento (también conocido como "cuerpo" o "carga") llega del origen codificado mediante el formato _Apache Avro_, o algún formato de serialización propietario, pero la expectativa de que el sistema que posee el destino es que el contenido tenga codificación _JSON_, lo primero que hará una tarea de replicación de transcodificación será deserializar la carga de _Apache Avro_ en un gráfico de objetos en memoria y, después, serializará ese gráfico en el formato _JSON_ para el mensaje que se reenvía. La transcodificación también incluye las tareas de descompresión y **compresión del contenido**.
- **_Transformación_**: es posible que los eventos que contienen datos con estructura necesiten volver a dar forma a dichos datos para facilitar su consumo por parte de los consumidores de bajada, lo que puede obligar a realizar trabajos como el acoplamiento de estructuras anidadas, la eliminación de elementos de datos extraños o la remodelación de la carga para ajustarse exactamente a un esquema determinado.
- **_Procesamiento por lotes_**: los eventos se pueden recibir en lotes (varios eventos en una única transferencia) desde un origen, pero se deben reenviar uno a uno a su destino, o viceversa. Por tanto, un tarea puede reenviar varios eventos según una única transferencia de eventos de entrada, o bien agregar un conjunto de eventos que, posteriormente, se transfieren juntos.
- **_Validación_**: a menudo, hay que comprobar si los datos de los eventos de orígenes externos cumplen un conjunto de reglas antes de reenviarlos. Las reglas se pueden expresar mediante esquemas o un código. Los eventos que se detecte que no las cumplen se pueden eliminar, e indicar el problema en los registros, o se pueden reenviar a un destino especial para tratarlos más.
- **_Enriquecimiento_**: es posible que los datos de eventos que provienen de algunos orígenes deban enriquecerse con mayor contexto, con el fin de que se puedan usar en los sistemas de destino, lo que puede implicar la búsqueda de datos de referencia y la inserción de datos con el evento, o la incorporación de información sobre el origen que se conoce en la tarea de replicación, pero que no se incluye en los eventos.
- **_Filtro_**: es posible que algunos eventos que lleguen de un origen tengan que retenerse en el destino debido a alguna regla. Un filtro prueba el evento con una regla y lo elimina si no la cumple. El filtrado de eventos duplicados mediante la observación de determinados criterios y la eliminación de los eventos posteriores con los mismos valores es una forma de filtrado.
- **_Criptografía_**: puede que una tarea de replicación tenga que descifrar el contenido que llega del origen o cifrar el contenido que se reenvía a un destino, o puede que tenga que comprobar la integridad del contenido y los metadatos, en relación con una firma que incluye en el evento, o bien adjuntar dicha firma.
- **_Atestación_**: una tarea de replicación puede adjuntar metadatos, potencialmente protegidos por una firma digital, a un evento que atestigua que el evento se ha recibido a través de un canal concreto o en un momento determinado.
- **_Encadenamiento_**: una tarea de replicación puede aplicar firmas a secuencias de eventos, de modo que la integridad de la secuencia esté protegida y se puedan detectar los eventos que faltan.

Normalmente, los patrones de transformación, procesamiento por lotes y enriquecimiento se implementan mejor con los trabajos de [Azure Stream Analytics](../stream-analytics/stream-analytics-introduction.md). 

Todos esos patrones se pueden implementar mediante Azure Functions, y usar el [desencadenador de Event Hubs](../azure-functions/functions-bindings-event-hubs-trigger.md) para adquirir eventos y el [enlace de salida del centro de eventos](../azure-functions/functions-bindings-event-hubs-output.md) para entregarlos.

## <a name="routing"></a>Enrutamiento

El patrón de enrutamiento se basa en el patrón de [replicación](#replication), pero en lugar de tener un origen y un destino, la tarea de replicación tiene varios destinos, que se muestran aquí en C#:

```csharp
[FunctionName("EH2EH")]
public static async Task Run(
    [EventHubTrigger("source", Connection = "EventHubConnectionAppSetting")] EventData[] events,
    [EventHub("dest1", Connection = "EventHubConnectionAppSetting")] EventHubClient output1,
    [EventHub("dest2", Connection = "EventHubConnectionAppSetting")] EventHubClient output2,
    ILogger log)
{
    foreach (EventData eventData in events)
    {
        // send to output1 and/or output2 based on criteria
        EventHubReplicationTasks.ConditionalForwardToEventHub(input, output1, log, (eventData) => {
            return ( inputEvent.SystemProperties.SequenceNumber%2==0 ) ? inputEvent : null;
        });
        EventHubReplicationTasks.ConditionalForwardToEventHub(input, output2, log, (eventData) => {
            return ( inputEvent.SystemProperties.SequenceNumber%2!=0 ) ? inputEvent : null;
        });
    }
}
```

La función de enrutamiento considerará los metadatos del mensaje o la carga del mensaje y, después, elegirá uno de los destinos disponibles a los que realizar el envío.

En Azure Stream Analytics, puede lograr lo mismo con la definición de varias salidas y la ejecución de una consulta por salida.

```sql
select * into dest1Output from inputSource where Info = 1
select * into dest2Output from inputSource where Info = 2
```


## <a name="log-projection"></a>Proyección de registros

El patrón de proyección de registros simplifica la secuencia de eventos en una base de datos indexada, con eventos que se convierten en registros en la base de datos. Normalmente, los eventos se agregan a la misma colección o tabla, y la clave de partición del centro de eventos se convierte en la parte de la clave principal que busca convertir el registro en único.

La proyección de registros puede generar un historial de series temporales de los datos de eventos o una vista compacta, en la que solo se conserva el evento más reciente para cada clave de partición. La forma de la base de datos de destino depende en última instancia de usted y de las necesidades de la aplicación. Este patrón también se conoce como "aprovisionamiento de eventos".

> [!TIP]
> Puede crear fácilmente proyecciones de registros en [Azure SQL Database](../stream-analytics/sql-database-output.md) y [Azure Cosmos DB](../stream-analytics/azure-cosmos-db-output.md) en Azure Stream Analytics, que debe ser la opción preferente.

Los siguientes proyectos de Azure Functions proyectan el contenido de un centro de eventos compactado en una colección de Azure Cosmos DB.

```C#
[FunctionName("Eh1ToCosmosDb1Json")]
[ExponentialBackoffRetry(-1, "00:00:05", "00:05:00")]
public static async Task Eh1ToCosmosDb1Json(
    [EventHubTrigger("eh1", ConsumerGroup = "Eh1ToCosmosDb1", Connection = "Eh1ToCosmosDb1-source-connection")] EventData[] input,
    [CosmosDB(databaseName: "SampleDb", collectionName: "foo", ConnectionStringSetting = "CosmosDBConnection")] IAsyncCollector<object> output,
    ILogger log)
{
    foreach (var ev in input)
    {
        if (!string.IsNullOrEmpty(ev.SystemProperties.PartitionKey))
        {
            var record = new
            {
                id = ev.SystemProperties.PartitionKey,
                data = JsonDocument.Parse(ev.Body),
                properties = ev.Properties
            };
            await output.AddAsync(record);
        }
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

- [Aplicaciones del replicador de eventos en Azure Functions][1]
- [Replicación de eventos entre Event Hubs][2]
- [Replicación de eventos en Azure Service Bus][3]

[1]: event-hubs-federation-replicator-functions.md
[2]: https://github.com/Azure-Samples/azure-messaging-replication-dotnet/tree/main/functions/config/EventHubCopy
[3]: https://github.com/Azure-Samples/azure-messaging-replication-dotnet/tree/main/functions/config/EventHubCopyToServiceBus
