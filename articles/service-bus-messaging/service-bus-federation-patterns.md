---
title: 'Patrones de tareas de replicación de mensajes: Azure Service Bus | Microsoft Docs'
description: En este artículo se proporcionan instrucciones detalladas para implementar patrones de tareas de replicación de mensajes específicos.
ms.topic: article
ms.date: 12/12/2020
ms.openlocfilehash: 5e6161d39281dc48284737cf3ee1f83853db17ef
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110462324"
---
# <a name="message-replication-tasks-patterns"></a>Patrones de tareas de replicación de mensajes

En las páginas de [introducción a la federación](service-bus-federation-overview.md) e [introducción a las funciones del replicador](service-bus-federation-replicator-functions.md), se explica la lógica y los elementos básicos de las tareas de replicación. Se recomienda familiarizarse con ellos antes de continuar con este artículo.

En este artículo, se detallan las instrucciones de implementación de algunos de los patrones resaltados en la sección de información general. 

## <a name="replication"></a>Replicación 

El patrón de replicación copia mensajes de una cola o de un tema en el siguiente, o en otro destino, como Event Hubs. Los mensajes se reenvían sin realizar ninguna modificación en su carga. 

La implementación de este patrón se trata en el ejemplo de [replicación de mensajes hacia y desde Azure Service Bus](https://github.com/Azure-Samples/azure-messaging-replication-dotnet/tree/main/functions/config/ServiceBusCopy).

### <a name="sequences-and-order-preservation"></a>Secuencias y conservación del orden

El objetivo del modelo de replicación no es conservar el orden absoluto de los mensajes de una cola o tema de origen en una cola o tema de destino, sino conservar, siempre que sea necesario, el orden relativo de los mensajes cuando la aplicación lo requiera. Para habilitar dicho modelo, la aplicación habilita la compatibilidad con sesiones de la entidad de origen y agrupa los mensajes relacionados con la misma [clave de sesión](message-sessions.md).

Las funciones de replicación pregeneradas que tienen en cuenta la sesión garantizan que las secuencias de mensajes con el mismo identificador de sesión recuperado de una entidad de origen se envían a la cola o tema de destino como un lote en la secuencia original y con el mismo identificador de sesión. 

### <a name="service-assigned-metadata"></a>Metadatos asignados por el servicio 

Los metadatos asignados por el servicio de un mensaje obtenido de la cola o tema de origen, el tiempo de puesta en cola original y el número de secuencia se sustituyen por nuevos valores asignados por el servicio en la cola o tema de destino, pero con las tareas de replicación predeterminadas que se proporcionan en nuestros ejemplos, los valores originales se conservan en las propiedades del usuario: `repl-enqueue-time` (cadena ISO8601) y `repl-sequence`.

Dichas propiedades son de tipo cadena y contienen el valor en cadena de las propiedades originales respectivas.  Si el mensaje se reenvía varias veces, los metadatos asignados por el servicio del origen inmediato se anexan a las propiedades existentes, y los valores se separan con punto y coma. 

### <a name="failover"></a>Conmutación por error

Si usa la replicación para la recuperación ante desastres, para protegerse de los mensajes de disponibilidad regional en el servicio de Service Bus, o de interrupciones de la red, cualquier escenario de error requiere la realización de una conmutación por error de una cola o tema al siguiente, lo que indica a los productores o consumidores que usen el punto de conexión secundario.

En todos los escenarios de conmutación por error, se supone que los elementos necesarios de los espacios de nombres son estructuralmente idénticos, lo que significa que tanto las colas como los temas tienen un nombre idéntico y que las reglas de firma de acceso compartido o las reglas de control de acceso basado en roles se configuran de la misma manera. Para crear (y actualizar) un espacio de nombres secundario, siga la [guía para mover espacios de nombres](move-across-regions.md) y omita el paso de limpieza.

Para obligar a los productores y consumidores a realizar el cambio, debe proporcionar la información sobre qué espacios de nombres usar para realizar la búsqueda en una ubicación con la que se pueda establecer contacto fácilmente y que sea fácil de actualizar. Si los productores o los consumidores encuentran errores frecuentes o persistentes, deben consultar esa ubicación y ajustar su configuración. Hay muchas maneras de compartir esa configuración, pero se indican dos a continuación: DNS y recursos compartidos.

#### <a name="dns-based-failover-configuration"></a>Configuración de conmutación por error basada en DNS

Un posible enfoque consiste en mantener la información de los registros SRV de DNS en un DNS que controla y apunta a los puntos de conexión respectivos de la cola o del tema. Tenga en cuenta que el centro de mensajes no permite que sus puntos de conexión tengan alias directamente con registros CNAME, lo que significa que usará DNS como mecanismo de búsqueda resistente para las direcciones de punto de conexión y no para resolver directamente la información de dirección IP. 

Supongamos que posee el dominio `example.com` y, para su aplicación, una zona `test.example.com`. En el caso de dos instancias de Service Bus alternativas, ahora creará dos zonas anidadas adicionales y un registro SRV en cada una de ellas. 

A los registros SRV, siguiendo la convención común, se les agrega el prefijo `_azure_servicebus._amqp` y contienen dos registros de punto de conexión: uno para AMQP mediante TLS en el puerto 5671 y otro para AMQP mediante WebSockets en el puerto 443, y ambos apuntan al punto de conexión de Service Bus del espacio de nombres correspondiente a la zona.

| Zona                 | Registro SRV
|----------------------|-------------------------------------------------------------
| `sb1.test.example.com` | **`_azure_servicebus._amqp.sb1.test.example.com`**<br>`1 1 5671 sb1-test-example-com.servicebus.windows.net`<br>`2 2 443 sb1-test-example-com.servicebus.windows.net`
| `sb2.test.example.com` | **`_azure_servicebus._amqp.sb1.test.example.com`**<br>`1 1 5671 sb2-test-example-com.servicebus.windows.net`<br>`2 2 443 sb2-test-example-com.servicebus.windows.net`

En la zona de la aplicación, creará una entrada CNAME que apunte a la zona subordinada correspondiente a la cola o tema principales:

| Registro CNAME                 | Alias
|------------------------------|-------------------------------------------------------------
| `servicebus.test.example.com`  | `sb1.test.example.com`

El uso de un cliente DNS que permite consultar los registros CNAME y SRV explícitamente (los clientes integrados de Java y .NET solo permiten la resolución simple de nombres en direcciones IP), puede resolver el punto de conexión deseado. Con [DnsClient.NET](https://dnsclient.michaco.net/), por ejemplo, la función de búsqueda es:

``` C#
static string GetServiceBusName(string aliasName)
{
    const string SrvRecordPrefix = "_azure_servicebus._amqp.";
    LookupClient lookup = new LookupClient();

    return (from CNameRecord alias in (lookup.Query(aliasName, QueryType.CNAME).Answers)
            from SrvRecord srv in lookup.Query(SrvRecordPrefix + alias.CanonicalName, QueryType.SRV).Answers
            where srv.Port == 5671
            select srv.Target).FirstOrDefault()?.Value.TrimEnd('.');
}
```

La función devuelve el nombre de host de destino registrado para el puerto 5671 de la zona que tiene un alias actualmente con el CNAME como se mostró anteriormente. 

Para realizar una conmutación por error es necesario editar el registro CNAME y apuntarlo a la zona alternativa. 

La ventaja de usar DNS y, en concreto [Azure DNS](../dns/dns-overview.md), es que la información de Azure DNS se replica globalmente y, por consiguiente, es resistente a las interrupciones de una sola región.

Este procedimiento es similar al funcionamiento de la [recuperación ante desastres geográfica de Service Bus](service-bus-geo-dr.md), pero totalmente bajo su propio control y también funciona con escenarios activos/activos.

#### <a name="file-share-based-failover-configuration"></a>Configuración de conmutación por error basada en recursos compartidos de archivos

La alternativa más sencilla a usar DNS para compartir la información del punto de conexión es colocar el nombre del punto de conexión principal en un archivo de texto sin formato y servir el archivo desde una infraestructura que sea sólida frente a las interrupciones y, además, permitir actualizaciones. 

Si ya ha ejecutado una infraestructura de sitio web de alta disponibilidad con replicación de contenido y disponibilidad global, agréguele el archivo y vuelva a publicar el archivo si se necesita un modificador.

## <a name="merge"></a>Merge

El patrón de combinación tiene una o varias tareas de replicación que apuntan a un destino, posiblemente al mismo tiempo que productores normales que también envían mensajes al mismo destino. 

Estas son algunas variaciones de este patrón:
- Dos o más funciones de replicación que adquieren al mismo tiempo mensajes de orígenes independientes y los envían al mismo destino.
- Una función de replicación más que adquiere mensajes de un origen mientras que el destino también lo usan directamente los productores. 
- El patrón anterior, salvo los mensajes reflejados entre dos o más temas, lo que genera los temas que contienen los mismos mensajes, independientemente de dónde se produzcan los mensajes.

Las dos primeras variaciones de patrón son triviales y no difieren de las tareas de replicación sin formato.

El último escenario requiere la exclusión de la replicación de los mensajes que ya se han replicado. La técnica se muestra y se explica en el ejemplo activo/activo.

## <a name="editor"></a>Editor

El patrón de editor se basa en el patrón de [replicación](#replication), pero los mensajes se modifican antes de reenviarse. Estos son algunos ejemplos de esas modificaciones:

- ***Transcodificación***: si el contenido del mensaje (también llamado "cuerpo" o "carga útil") llega codificado de origen con formato *Apache Avro*, o algún otro formato de serialización propietario, pero el sistema al que pertenece el destino espera que la codificación del contenido sea *JSON*, una tarea de replicación de transcodificación primero deserializará la carga útil de *Apache Avro* en un gráfico de objetos en memoria y, después, serializará ese gráfico con el formato *JSON* para el mensaje que se reenvíe. La transcodificación también incluye las tareas de descompresión y **compresión del contenido**.
- ***Transformación***: es posible que los mensajes con datos estructurados necesiten remodelar esos datos para facilitar el consumo por parte de los consumidores finales. lo que puede obligar a realizar trabajos como el acoplamiento de estructuras anidadas, la eliminación de elementos de datos extraños o la remodelación de la carga para ajustarse exactamente a un esquema determinado.
- ***Procesamiento por lotes***: los mensajes se pueden recibir en lotes (varios mensajes en una única transferencia) desde un origen, pero se deben reenviar uno a uno a su destino, o viceversa. Por tanto, un tarea puede reenviar varios mensajes según una única transferencia de mensajes de entrada, o bien agregar un conjunto de mensajes que, posteriormente, se transfieren juntos. 
- ***Validación***: a menudo, hay que comprobar si los datos de los mensajes de orígenes externos cumplen un conjunto de reglas antes de reenviarlos. Las reglas se pueden expresar mediante esquemas o un código. Los mensajes que se detecte que no las cumplen se pueden eliminar, e indicar el problema en los registros, o se pueden reenviar a un destino especial para tratarlos más.   
- ***Enrique cimiento***: es posible que los datos de mensajes que provienen de algunos orígenes deban enriquecerse con mayor contexto, con el fin de que se puedan usar en los sistemas de destino, lo que puede implicar la búsqueda de datos de referencia y la inserción de datos con el mensaje, o la incorporación de información sobre el origen que se conoce en la tarea de replicación, pero que no se incluye en los mensajes. 
- ***Filtro***: es posible que algunos mensajes que lleguen de un origen tengan que retenerse en el destino debido a alguna regla. Un filtro prueba el mensaje con una regla y lo elimina si no la cumple. El filtrado de mensajes duplicados mediante la observación de determinados criterios y la eliminación de los mensajes posteriores con los mismos valores es una forma de filtrado.
- ***Enrutamiento y creación de particiones***: es posible que algunas tareas de replicación permitan dos o más destinos alternativos y definan las reglas para las que se elige el destino de replicación en cualquier mensaje determinado en función de los metadatos o del contenido del mensaje. Una forma especial de enrutamiento es la creación de particiones, donde la tarea asigna explícitamente las particiones a un destino de replicación en función de las reglas.
- ***Criptografía***: puede que una tarea de replicación tenga que descifrar el contenido que llega del origen o cifrar el contenido que se reenvía a un destino, o puede que tenga que comprobar la integridad del contenido y los metadatos, en relación con una firma que incluye en el mensaje, o bien adjuntar dicha firma. 
- ***Atestación***: una tarea de replicación puede adjuntar metadatos, potencialmente protegidos por una firma digital, a un mensaje que atestigua que el mensaje se ha recibido a través de un canal concreto o en un momento determinado.     
- ***Encadenamiento***: una tarea de replicación puede aplicar firmas a secuencias de mensajes, de tal manera que la integridad de la secuencia esté protegida y se puedan detectar mensajes que falten.

Todos esos patrones se pueden implementar mediante Azure Functions, y usar el [desencadenador de los centros de mensajes](../azure-functions/functions-bindings-service-bus-trigger.md) para adquirir mensajes y el [enlace de salida de la cola o del tema](../azure-functions/functions-bindings-service-bus-output.md) para entregarlos. 

## <a name="routing"></a>Enrutamiento

El patrón de enrutamiento se basa en el patrón de [replicación](#replication), pero en lugar de tener un origen y un destino, la tarea de replicación tiene varios destinos, que se muestran aquí en C#:

``` csharp
[FunctionName("SBRouter")]
public static async Task Run(
    [ServiceBusTrigger("source", Connection = "serviceBusConnectionAppSetting")] Message[] messages,
    [ServiceBus("dest1", Connection = "serviceBusConnectionAppSetting")] QueueClient output1,
    [ServiceBus("dest2", Connection = "serviceBusConnectionAppSetting")] QueueClient output2,
    ILogger log)
{
    foreach (Message messageData in messages)
    {
        // send to output1 or output2 based on criteria 
    }
}
```

La función de enrutamiento considerará los metadatos del mensaje o la carga del mensaje y, después, elegirá uno de los destinos disponibles a los que realizar el envío.

## <a name="next-steps"></a>Pasos siguientes

- [Aplicaciones del replicador de mensajes en Azure Functions][1]
- [Réplica de mensajes entre Service Bus][2]
- [Réplica de mensajes en Azure Event Hubs][3]

[1]: service-bus-federation-replicator-functions.md
[2]: https://github.com/Azure-Samples/azure-messaging-replication-dotnet/tree/main/functions/config/ServiceBusCopy
[3]: https://github.com/Azure-Samples/azure-messaging-replication-dotnet/tree/main/functions/config/ServiceBusCopyToEventHub
