---
title: Procedimientos recomendados para mejorar el rendimiento mediante Azure Service Bus
description: Describe cómo usar Service Bus para optimizar el rendimiento al intercambiar mensajes asincrónicos.
ms.topic: article
ms.date: 03/09/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 19c365a233a2dafc98fb91e340717ba998616891
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107904701"
---
# <a name="best-practices-for-performance-improvements-using-service-bus-messaging"></a>Procedimientos recomendados para mejorar el rendimiento mediante la mensajería de Service Bus

En este artículo se describe cómo usar Azure Service Bus para optimizar el rendimiento al intercambiar mensajes asincrónicos. En la primera parte de este artículo se describen distintos mecanismos para aumentar el rendimiento. En la segunda parte se proporcionan instrucciones sobre cómo usar Service Bus de manera que pueda ofrecer el mejor rendimiento en un escenario determinado.

En todo este artículo, el término "cliente" hace referencia a cualquier entidad que acceda a Service Bus. Un cliente puede asumir el rol de un remitente o receptor. El término "remitente" se usa para referirse a un cliente de una cola o un tema de Service Bus que envía mensajes a una cola o un tema de Service Bus. El término "receptor" hace referencia a un cliente de una cola o una suscripción de Service Bus que recibe mensajes de una cola o una suscripción de Service Bus.

## <a name="resource-planning-and-considerations"></a>Planeamiento de recursos y consideraciones

Al igual que con cualquier recurso técnico, el planeamiento prudente es clave para garantizar que Azure Service Bus proporciona el rendimiento esperado por la aplicación. La configuración o topología correctas para los espacios de nombres de Service Bus depende de una serie de factores que involucran la arquitectura de la aplicación y cómo se utilizan cada una de las características de Service Bus.

### <a name="pricing-tier"></a>Plan de tarifa

Service Bus ofrece varios planes de tarifa. Se recomienda elegir el nivel adecuado para los requisitos de la aplicación.

   * **Nivel Estándar**: adecuado para entornos de desarrollo y pruebas o escenarios de bajo rendimiento en los que las aplicaciones **no son sensibles** a la limitación.

   * **Nivel Premium**: adecuado para entornos de producción con diversos requisitos de rendimiento en los que se requiere una latencia y un rendimiento predecibles. Además, los espacios de nombres prémium de Service Bus se pueden [escalar automáticamente](automate-update-messaging-units.md) para dar cabida a picos de rendimiento.

> [!NOTE]
> Si no se selecciona el nivel correcto, existe el riesgo de desbordar el espacio de nombres de Service Bus, lo que puede dar lugar a la [limitación](service-bus-throttling.md).
>
> La limitación no conduce a la pérdida de datos. Las aplicaciones que aprovechan el SDK de Service Bus pueden usar la directiva de reintentos predeterminada para asegurarse de que Service Bus finalmente acepta los datos.
>

### <a name="calculating-throughput-for-premium"></a>Cálculo del rendimiento para el nivel Premium

Los datos enviados Service Bus se serializan a binario y luego se deserializan cuando los recibe el receptor. Por lo tanto, mientras que las aplicaciones piensan en los **mensajes** como unidades atómicas de trabajo, Service Bus mide el rendimiento en términos de bytes (o megabytes).

Al calcular el requisito de rendimiento, tenga en cuenta los datos que se envían a Service Bus (entrada) y los datos que se reciben de Service Bus (salida).

Como se esperaba, el rendimiento es mayor para cargas útiles de mensajes más pequeñas que se pueden procesar por lotes conjuntamente.

#### <a name="benchmarks"></a>Pruebas comparativas

Este es un [ejemplo de GitHub](https://github.com/Azure-Samples/service-bus-dotnet-messaging-performance) que puede ejecutar para ver el rendimiento esperado que recibirá para el espacio de nombres de Service Bus. En nuestras [pruebas comparativas](https://techcommunity.microsoft.com/t5/Service-Bus-blog/Premium-Messaging-How-fast-is-it/ba-p/370722) observamos aproximadamente 4 MB/segundo por unidad de mensajería (MU) de entrada y salida.

El ejemplo de pruebas comparativas no usa características avanzadas, por lo que el rendimiento que observan las aplicaciones será diferente en función de los escenarios.

#### <a name="compute-considerations"></a>Consideraciones de proceso

El uso de ciertas características de Service Bus puede requerir un uso de proceso que puede reducir el rendimiento esperado. Algunas de estas características:

1. Son sesiones.
2. Se extienden a varias suscripciones sobre un solo tema.
3. Ejecutan muchos filtros en una sola suscripción.
4. Son mensajes programados.
5. Son mensajes aplazados.
6. Transacciones.
7. Son la desduplicación y ventana de tiempo de retrospectiva.
8. Se desvían (desvío de una entidad a otra).

Si la aplicación aprovecha cualquiera de las características anteriores y no recibe el rendimiento esperado, puede revisar las métricas de **uso de CPU** y considerar la posibilidad de escalar verticalmente el espacio de nombres Premium de Service Bus.

También puede usar Azure Monitor para [escalar automáticamente el espacio de nombres Service Bus](automate-update-messaging-units.md).

### <a name="sharding-across-namespaces"></a>Particionamiento entre espacios de nombres

Aunque el escalado vertical de proceso (unidades de mensajería) asignado al espacio de nombres es una solución más sencilla, **es posible que no** proporcione un aumento lineal en el rendimiento. Esto se debe a los elementos internos de Service Bus (almacenamiento, red, etc.), que pueden estar limitando el rendimiento.

En este caso, la solución más limpia es particionar las entidades (colas y temas) entre diferentes espacios de nombres Premium de Service Bus. También puede considerar el particionamiento entre distintos espacios de nombres en diferentes regiones de Azure.

## <a name="protocols"></a>Protocolos
Service Bus permite a los clientes enviar y recibir mensajes a través de uno de estos tres protocolos:

1. Advanced Message Queuing Protocol (AMQP)
2. Protocolo de mensajería de Service Bus (SBMP)
3. Protocolo de transferencia de hipertexto (HTTP)

AMQP es el más eficaz, ya que mantiene la conexión a Service Bus. También implementa el [procesamiento por lotes](#batching-store-access) y la [captura previa](#prefetching). A menos que se mencione explícitamente, en todo el contenido de este artículo se supone que se usa AMQP o SBMP.

> [!IMPORTANT]
> SBMP solo está disponible para .NET Framework. AMQP es el valor predeterminado de .NET Standard.

## <a name="choosing-the-appropriate-service-bus-net-sdk"></a>Elección del SDK de .NET para Service Bus adecuado
Hay tres SDK de .NET para Azure Service Bus admitidos. Sus API son muy similares, y puede resultar confuso cuál elegir. Consulte la siguiente tabla para que le ayude a tomar su decisión. El SDK de Azure.Messaging.ServiceBus es el más reciente y se recomienda usarlo antes que otros SDK. Los SDK de Azure.Messaging.ServiceBus and Microsoft.Azure.ServiceBus son modernos, eficaces y compatibles con varias plataformas. Además, admiten el protocolo AMQP sobre WebSockets y forman parte de la colección de SDK de .NET para Azure de proyectos de código abierto.

| Paquete NuGet | Espacios de nombres principales | Plataformas mínimas | Protocolos |
|---------------|----------------------|---------------------|-------------|
| [Azure.Messaging.ServiceBus](https://www.nuget.org/packages/Azure.Messaging.ServiceBus) | `Azure.Messaging.ServiceBus`<br>`Azure.Messaging.ServiceBus.Administration` | .NET Core 2.0<br>.NET Framework 4.6.1<br>Mono 5.4<br>Xamarin.iOS 10.14<br>Xamarin.Mac 3.8<br>Xamarin.Android 8.0<br>Plataforma universal de Windows 10.0.16299 | AMQP<br>HTTP |
| [Microsoft.Azure.ServiceBus](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus) | `Microsoft.Azure.ServiceBus`<br>`Microsoft.Azure.ServiceBus.Management` | .NET Core 2.0<br>.NET Framework 4.6.1<br>Mono 5.4<br>Xamarin.iOS 10.14<br>Xamarin.Mac 3.8<br>Xamarin.Android 8.0<br>Plataforma universal de Windows 10.0.16299 | AMQP<br>HTTP |
| [WindowsAzure.ServiceBus](https://www.nuget.org/packages/WindowsAzure.ServiceBus) | `Microsoft.ServiceBus`<br>`Microsoft.ServiceBus.Messaging` | .NET Framework 4.6.1 | AMQP<br>SBMP<br>HTTP |

Para más información sobre la compatibilidad mínima con la plataforma .NET Standard, consulte [Compatibilidad con la implementación de .NET](/dotnet/standard/net-standard#net-implementation-support).

## <a name="reusing-factories-and-clients"></a>Reutilización de factorías y clientes
# <a name="azuremessagingservicebus-sdk"></a>[SDK de Azure.Messaging.ServiceBus](#tab/net-standard-sdk-2)
Los objetos de Service Bus que interactúan con el servicio, como [ServiceBusClient](/dotnet/api/azure.messaging.servicebus.servicebusclient), [ServiceBusSender](/dotnet/api/azure.messaging.servicebus.servicebussender), [ServiceBusReceiver](/dotnet/api/azure.messaging.servicebus.servicebusreceiver) y [ServiceBusProcessor](/dotnet/api/azure.messaging.servicebus.servicebusprocessor), deben registrarse para la inserción de dependencias como singleton (o bien crear instancias una vez y compartirlas). ServiceBusClient se puede registrar para la inserción de dependencias con [ServiceBusClientBuilderExtensions](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/servicebus/Azure.Messaging.ServiceBus/src/Compatibility/ServiceBusClientBuilderExtensions.cs). 

Se recomienda no cerrar ni desechar estos objetos después de enviar o recibir cada mensaje. Si se cierran o desechan los objetos específicos de la entidad (ServiceBusSender/Receiver/Processor), se rompe el vínculo al servicio Service Bus. Si se desecha ServiceBusClient, se rompe la conexión al servicio Service Bus. 

# <a name="microsoftazureservicebus-sdk"></a>[SDK de Microsoft.Azure.ServiceBus](#tab/net-standard-sdk)

Objetos de cliente Service Bus, como las implementaciones de [`IQueueClient`][QueueClient] o [`IMessageSender`][MessageSender], se deben registrar para la inserción de dependencias como singletons (o crear una instancia de una vez y compartirse). Se recomienda no cerrar las factorías de mensajería ni los clientes de cola, tema o suscripción después de enviar un mensaje y luego volver a crearlos al enviar el mensaje siguiente. Al cerrar una factoría de mensajería, se elimina la conexión al servicio Service Bus. Al volver a crear la factoría, se establece una nueva conexión. 

# <a name="windowsazureservicebus-sdk"></a>[SDK de WindowsAzure.ServiceBus](#tab/net-framework-sdk)

Los objetos de cliente de Service Bus, como `QueueClient` o `MessageSender`, se crean mediante un objeto [MessagingFactory][MessagingFactory], que también proporciona administración interna de las conexiones. Se recomienda no cerrar las factorías de mensajería ni los clientes de cola, tema o suscripción después de enviar un mensaje y luego volver a crearlos al enviar el mensaje siguiente. Al cerrar una factoría de mensajería, se elimina la conexión con el servicio Service Bus y, al volver a crearla, se establece una nueva conexión. 

---

La siguiente nota se aplica a todos los SDK:

> [!NOTE]
> Establecer una conexión es una operación costosa que puede evitar si vuelve a usar la misma factoría y los mismos objetos de cliente para varias operaciones. Puede utilizar estos objetos de cliente para operaciones asincrónicas simultáneas y desde varios subprocesos.

## <a name="concurrent-operations"></a>Operaciones simultáneas
Las operaciones como enviar, recibir, eliminar, etc., tardan algún tiempo. Este tiempo incluye el que tarda el servicio Service Bus en procesar la operación y la latencia de la solicitud y la respuesta. Para aumentar el número de operaciones por tiempo, las operaciones deberán ejecutarse simultáneamente.

El cliente programa las operaciones simultáneas mediante la realización de operaciones **asincrónicas**. La siguiente solicitud se inicia antes de que se complete la anterior. En el siguiente fragmento de código se proporciona un ejemplo de operación asincrónica de envío:

# <a name="azuremessagingservicebus-sdk"></a>[SDK de Azure.Messaging.ServiceBus](#tab/net-standard-sdk-2)
```csharp
var messageOne = new ServiceBusMessage(body);
var messageTwo = new ServiceBusMessage(body);

var sendFirstMessageTask =
    sender.SendMessageAsync(messageOne).ContinueWith(_ =>
    {
        Console.WriteLine("Sent message #1");
    });
var sendSecondMessageTask =
    sender.SendMessageAsync(messageTwo).ContinueWith(_ =>
    {
        Console.WriteLine("Sent message #2");
    });

await Task.WhenAll(sendFirstMessageTask, sendSecondMessageTask);
Console.WriteLine("All messages sent");

```

# <a name="microsoftazureservicebus-sdk"></a>[SDK de Microsoft.Azure.ServiceBus](#tab/net-standard-sdk)

```csharp
var messageOne = new Message(body);
var messageTwo = new Message(body);

var sendFirstMessageTask =
    queueClient.SendAsync(messageOne).ContinueWith(_ =>
    {
        Console.WriteLine("Sent message #1");
    });
var sendSecondMessageTask =
    queueClient.SendAsync(messageTwo).ContinueWith(_ =>
    {
        Console.WriteLine("Sent message #2");
    });

await Task.WhenAll(sendFirstMessageTask, sendSecondMessageTask);
Console.WriteLine("All messages sent");
```

# <a name="windowsazureservicebus-sdk"></a>[SDK de WindowsAzure.ServiceBus](#tab/net-framework-sdk)

```csharp
var messageOne = new BrokeredMessage(body);
var messageTwo = new BrokeredMessage(body);

var sendFirstMessageTask =
    queueClient.SendAsync(messageOne).ContinueWith(_ =>
    {
        Console.WriteLine("Sent message #1");
    });
var sendSecondMessageTask =
    queueClient.SendAsync(messageTwo).ContinueWith(_ =>
    {
        Console.WriteLine("Sent message #2");
    });

await Task.WhenAll(sendFirstMessageTask, sendSecondMessageTask);
Console.WriteLine("All messages sent");
```

---

El siguiente código es un ejemplo de operación asincrónica de recepción.

# <a name="azuremessagingservicebus-sdk"></a>[SDK de Azure.Messaging.ServiceBus](#tab/net-standard-sdk-2)

```csharp
var client = new ServiceBusClient(connectionString);
var options = new ServiceBusProcessorOptions 
{

      AutoCompleteMessages = false,
      MaxConcurrentCalls = 20
};
await using ServiceBusProcessor processor = client.CreateProcessor(queueName,options);
processor.ProcessMessageAsync += MessageHandler;
processor.ProcessErrorAsync += ErrorHandler;

static Task ErrorHandler(ProcessErrorEventArgs args)
{
    Console.WriteLine(args.Exception);
    return Task.CompletedTask;
};

static async Task MessageHandler(ProcessMessageEventArgs args)
{
    Console.WriteLine("Handle message");
    await args.CompleteMessageAsync(args.Message);
}

await processor.StartProcessingAsync();
```

# <a name="microsoftazureservicebus-sdk"></a>[SDK de Microsoft.Azure.ServiceBus](#tab/net-standard-sdk)

En el repositorio de GitHub encontrará <a href="https://github.com/Azure/azure-service-bus/blob/master/samples/DotNet/Microsoft.Azure.ServiceBus/SendersReceiversWithQueues" target="_blank">ejemplos de código fuente<span class="docon docon-navigate-external x-hidden-focus"></span></a> completos:

```csharp
var receiver = new MessageReceiver(connectionString, queueName, ReceiveMode.PeekLock);

static Task LogErrorAsync(Exception exception)
{
    Console.WriteLine(exception);
    return Task.CompletedTask;
};

receiver.RegisterMessageHandler(
    async (message, cancellationToken) =>
    {
        Console.WriteLine("Handle message");
        await receiver.CompleteAsync(message.SystemProperties.LockToken);
    },
    new MessageHandlerOptions(e => LogErrorAsync(e.Exception))
    {
        AutoComplete = false,
        MaxConcurrentCalls = 20
    });
```

Se crea una instancia del objeto `MessageReceiver` con la cadena de conexión, el nombre de la cola y un modo de recepción peek-look. Luego, se utiliza la instancia de `receiver` para registrar el controlador de mensajes.

# <a name="windowsazureservicebus-sdk"></a>[SDK de WindowsAzure.ServiceBus](#tab/net-framework-sdk)

En el repositorio de GitHub encontrará <a href="https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/SendersReceiversWithQueues" target="_blank">ejemplos de código fuente<span class="docon docon-navigate-external x-hidden-focus"></span></a> completos:

```csharp
var factory = MessagingFactory.CreateFromConnectionString(connectionString);
var receiver = await factory.CreateMessageReceiverAsync(queueName, ReceiveMode.PeekLock);

// Register the handler to receive messages asynchronously
receiver.OnMessageAsync(
    async message =>
    {
        Console.WriteLine("Handle message");
        await message.CompleteAsync();
    },
    new OnMessageOptions
    {
        AutoComplete = false,
        MaxConcurrentCalls = 20
    });
```

`MessagingFactory` crea un objeto `factory` a partir de la cadena de conexión. Con la instancia de `factory`, se crea una instancia de un `MessageReceiver`. Luego, se utiliza la instancia de `receiver` para registrar el controlador de mensajes.

---

## <a name="receive-mode"></a>Modo de recepción

Al crear un cliente de cola o suscripción, puede especificar un modo de recepción: *Peek-lock* (Inspección y bloqueo) o *Receive And Delete* (Recepción y eliminación). El modo de recepción predeterminado es `PeekLock`. Cuando se trabaja en el modo predeterminado, el cliente envía una solicitud para recibir un mensaje de Service Bus. Después de que el cliente haya recibido el mensaje, envía una solicitud para completarlo.

Al establecer el modo de recepción en `ReceiveAndDelete`, ambos pasos se combinan en una sola solicitud. Estos pasos reducen el número total de operaciones y pueden mejorar el rendimiento general de los mensajes. Esta mejora del rendimiento conlleva el riesgo de la pérdida de mensajes.

Service Bus no admite transacciones para las operaciones de recepción y eliminación. Además, se necesita la semántica de bloqueo de información en aquellos casos en los que el cliente quiera aplazar un mensaje o [procesarlo como correo devuelto](service-bus-dead-letter-queues.md).

## <a name="client-side-batching"></a>Procesamiento por lotes del lado cliente

El procesamiento por lotes del lado cliente permite que un cliente de cola o tema retrase el envío de un mensaje durante un período determinado. Si el cliente envía más mensajes durante este período, los transmite en un único lote. El procesamiento por lotes del lado cliente también hace que un cliente de cola o suscripción procese varias solicitudes **Complete** por lotes en una única solicitud. El procesamiento por lotes solo está disponible para las operaciones **Send** y **Complete** asincrónicas. Las operaciones sincrónicas se envían de inmediato al servicio Service Bus. El procesamiento por lotes no tiene lugar con las operaciones de inspección o recepción, así como tampoco entre clientes.

# <a name="azuremessagingservicebus-sdk"></a>[SDK de Azure.Messaging.ServiceBus](#tab/net-standard-sdk-2)
La funcionalidad de procesamiento por lotes del SDK de .NET Standard aún no expone ninguna propiedad que se pueda manipular.

# <a name="microsoftazureservicebus-sdk"></a>[SDK de Microsoft.Azure.ServiceBus](#tab/net-standard-sdk)

La funcionalidad de procesamiento por lotes del SDK de .NET Standard aún no expone ninguna propiedad que se pueda manipular.

# <a name="windowsazureservicebus-sdk"></a>[SDK de WindowsAzure.ServiceBus](#tab/net-framework-sdk)

De forma predeterminada, un cliente usa un intervalo entre lotes de 20 ms. Puede cambiar el intervalo entre lotes si configura la propiedad [BatchFlushInterval][BatchFlushInterval] antes de crear la factoría de mensajería. Esta configuración afecta a todos los clientes que se creen en esta factoría.

Para deshabilitar el procesamiento por lotes, establezca la propiedad [BatchFlushInterval][BatchFlushInterval] en **TimeSpan.Zero**. Por ejemplo:

```csharp
var settings = new MessagingFactorySettings
{
    NetMessagingTransportSettings =
    {
        BatchFlushInterval = TimeSpan.Zero
    }
};
var factory = MessagingFactory.Create(namespaceUri, settings);
```

El procesamiento por lotes no afecta al número de operaciones de mensajería facturables y solo está disponible para el protocolo de cliente de Service Bus mediante la biblioteca [Microsoft.ServiceBus.Messaging](https://www.nuget.org/packages/WindowsAzure.ServiceBus/). El protocolo HTTP no admite el procesamiento por lotes.

> [!NOTE]
> El valor `BatchFlushInterval` garantiza que el procesamiento por lotes es implícito desde la perspectiva de la aplicación. Es decir, la aplicación realiza llamadas `SendAsync` y `CompleteAsync`, y no realiza llamadas de Batch específicas.
>
> El procesamiento por lotes del lado cliente explícito se puede implementar mediante el uso de la siguiente llamada al método:
> ```csharp
> Task SendBatchAsync(IEnumerable<BrokeredMessage> messages);
> ```
> Aquí el tamaño combinado de los mensajes debe ser menor que el tamaño máximo admitido por el plan de tarifa.

---

## <a name="batching-store-access"></a>Acceso al almacén de procesamiento por lotes

Para aumentar el rendimiento de una cola, un tema o una suscripción, Service Bus procesa varios mensajes por lotes al escribir en su almacén interno. 

- Al habilitar el procesamiento por lotes en una cola, la escritura de mensajes en el almacén y la eliminación de mensajes del almacén se procesarán por lotes. 
- Al habilitar el procesamiento por lotes en un tema, la escritura de mensajes en el almacén se procesa por lotes. 
- Cuando se habilita el procesamiento por lotes en una suscripción, la eliminación de mensajes del almacén se procesa por lotes. 
- Si el acceso al almacén por lotes está habilitado para una entidad, Service Bus retrasa una operación de escritura en almacén para ella unos 20 ms.

> [!NOTE]
> No hay ningún riesgo de pérdida de mensajes con procesamiento por lotes, aunque se produzca un error de Service Bus al final de un intervalo de procesamiento por lotes de 20 ms.

Las demás operaciones de almacenamiento que se producen durante este intervalo se agregan al lote. El acceso al almacén por lotes solo afecta a las operaciones **Enviar** y **Completar**, pero no a las de recepción. El acceso al almacén de procesamiento por lotes es una propiedad de una entidad. El procesamiento por lotes se produce en todas las entidades que tengan habilitado el acceso al almacén de procesamiento por lotes.

Cuando se crea una cola, un tema o una suscripción nuevos, el acceso al almacén de procesamiento por lotes está habilitado de manera predeterminada.


# <a name="azuremessagingservicebus-sdk"></a>[SDK de Azure.Messaging.ServiceBus](#tab/net-standard-sdk-2)
Para deshabilitar el acceso al almacén por lotes, necesitará una instancia de `ServiceBusAdministrationClient`. Cree `CreateQueueOptions` a partir de una descripción de la cola que establezca la propiedad `EnableBatchedOperations` en `false`.

```csharp
var options = new CreateQueueOptions(path)
{
    EnableBatchedOperations = false
};
var queue = await administrationClient.CreateQueueAsync(options);
```


# <a name="microsoftazureservicebus-sdk"></a>[SDK de Microsoft.Azure.ServiceBus](#tab/net-standard-sdk)

Para deshabilitar el acceso al almacén por lotes, necesitará una instancia de `ManagementClient`. Cree una cola a partir de una descripción de la cola que establezca la propiedad `EnableBatchedOperations` en `false`.

```csharp
var queueDescription = new QueueDescription(path)
{
    EnableBatchedOperations = false
};
var queue = await managementClient.CreateQueueAsync(queueDescription);
```

Para más información, consulte los siguientes artículos.
* <a href="https://docs.microsoft.com/dotnet/api/microsoft.azure.servicebus.management.queuedescription.enablebatchedoperations" target="_blank">`Microsoft.Azure.ServiceBus.Management.QueueDescription.EnableBatchedOperations` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.
* <a href="https://docs.microsoft.com/dotnet/api/microsoft.azure.servicebus.management.subscriptiondescription.enablebatchedoperations" target="_blank">`Microsoft.Azure.ServiceBus.Management.SubscriptionDescription.EnableBatchedOperations` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.
* <a href="https://docs.microsoft.com/dotnet/api/microsoft.azure.servicebus.management.topicdescription.enablebatchedoperations" target="_blank">`Microsoft.Azure.ServiceBus.Management.TopicDescription.EnableBatchedOperations` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

# <a name="windowsazureservicebus-sdk"></a>[SDK de WindowsAzure.ServiceBus](#tab/net-framework-sdk)

Para deshabilitar el acceso al almacén por lotes, necesitará una instancia de `NamespaceManager`. Cree una cola a partir de una descripción de la cola que establezca la propiedad `EnableBatchedOperations` en `false`.

```csharp
var queueDescription = new QueueDescription(path)
{
    EnableBatchedOperations = false
};
var queue = namespaceManager.CreateQueue(queueDescription);
```

Para más información, consulte los siguientes artículos.
* <a href="https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queuedescription.enablebatchedoperations" target="_blank">`Microsoft.ServiceBus.Messaging.QueueDescription.EnableBatchedOperations` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.
* <a href="https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription.enablebatchedoperations" target="_blank">`Microsoft.ServiceBus.Messaging.SubscriptionDescription.EnableBatchedOperations` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.
* <a href="https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.topicdescription.enablebatchedoperations" target="_blank">`Microsoft.ServiceBus.Messaging.TopicDescription.EnableBatchedOperations` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

---

El acceso al almacén por lotes no afecta al número de operaciones de mensajería facturables. Es una propiedad de una cola, un tema o una suscripción. Es independiente del modo de recepción y del protocolo que se usa entre un cliente y el servicio Service Bus.

## <a name="prefetching"></a>Captura previa

La [captura previa](service-bus-prefetch.md) permite que, al recibir mensajes, el cliente de la cola o la suscripción cargue otros adicionales desde el servicio. El cliente almacena estos mensajes en una memoria caché local. El tamaño de la caché está determinado por las propiedades `QueueClient.PrefetchCount` o `SubscriptionClient.PrefetchCount`. Cada cliente con la captura previa habilitada mantiene su propia memoria caché. Una caché no se comparte entre clientes. Si el cliente inicia una operación de recepción y su caché está vacía, el servicio transmite un lote de mensajes. El tamaño del lote equivale al tamaño de la memoria caché o a 256 KB, con preferencia por el menor valor. Si el cliente inicia una operación de recepción y la caché contiene un mensaje, este se recupera de la caché.

Cuando se lleva a cabo la captura previa de un mensaje, el servicio bloquea dicho mensaje. Con el bloqueo, se impide que otro receptor reciba el mensaje previamente capturado. Si el receptor no puede completar el mensaje antes de que expire el bloqueo, pasa a estar disponible para otros receptores. La copia de captura previa del mensaje permanece en la memoria caché. El receptor que consume la copia en caché expirada recibirá una excepción cuando intente completar dicho mensaje. De forma predeterminada, el bloqueo del mensaje expira tras 60 segundos. Este valor puede ampliarse a 5 minutos. Para evitar el consumo de mensajes expirados, establezca el tamaño de la caché en un valor inferior al número de mensajes que un cliente puede consumir en el intervalo de tiempo de expiración de bloqueo.

Cuando se usa la expiración de bloqueo predeterminada de 60 segundos, un buen valor de `PrefetchCount` es 20 veces la velocidad de procesamiento máxima de todos los destinatarios de la fábrica. Por ejemplo, una factoría crea tres receptores y cada receptor puede procesar hasta 10 mensajes por segundo. El número de capturas previas no debe superar 20 x 3 x 10 = 600. De forma predeterminada, `PrefetchCount` se establece en 0, lo que significa que no se realiza la captura previa de ningún mensaje adicional desde el servicio.

La captura previa de los mensajes aumenta el rendimiento general de una cola o una suscripción porque reduce el número total de operaciones de mensajes o recorridos de ida y vuelta. Sin embargo, capturar el primer mensaje tardará más tiempo (debido al aumento del tamaño del mensaje). Recibir mensajes previamente capturados de la caché será más rápido porque el cliente ya ha descargado estos mensajes.

El servidor comprueba la propiedad de período de vida (TTL) de un mensaje en el momento en que envía el mensaje al cliente. El cliente no comprueba la propiedad TTL del mensaje cuando lo recibe. En su lugar, se puede recibir el mensaje, aunque se haya superado el TTL del mensaje mientras el cliente lo almacenaba en caché.

La captura previa no afecta al número de operaciones de mensajería facturables y solo está disponible para el protocolo de cliente de Service Bus. El protocolo HTTP no admite la captura previa. La captura previa está disponible para las operaciones de recepción sincrónicas y asincrónicas.

# <a name="azuremessagingservicebus-sdk"></a>[SDK de Azure.Messaging.ServiceBus](#tab/net-standard-sdk-2)
Para más información, vea las propiedades `PrefetchCount` siguientes:

- [ServiceBusReceiver.PrefetchCount](/dotnet/api/azure.messaging.servicebus.servicebusreceiver.prefetchcount)
- [ServiceBusProcessor.PrefetchCount](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.prefetchcount)

Puede establecer los valores de estas propiedades en [ServiceBusReceiverOptions](/dotnet/api/azure.messaging.servicebus.servicebusreceiveroptions) o [ServiceBusProcessorOptions](/dotnet/api/azure.messaging.servicebus.servicebusprocessoroptions).

# <a name="microsoftazureservicebus-sdk"></a>[SDK de Microsoft.Azure.ServiceBus](#tab/net-standard-sdk)

Para más información, vea las propiedades `PrefetchCount` siguientes:

* <a href="https://docs.microsoft.com/dotnet/api/microsoft.azure.servicebus.queueclient.prefetchcount" target="_blank">`Microsoft.Azure.ServiceBus.QueueClient.PrefetchCount` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.
* <a href="https://docs.microsoft.com/dotnet/api/microsoft.azure.servicebus.subscriptionclient.prefetchcount" target="_blank">`Microsoft.Azure.ServiceBus.SubscriptionClient.PrefetchCount` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

# <a name="windowsazureservicebus-sdk"></a>[SDK de WindowsAzure.ServiceBus](#tab/net-framework-sdk)

Para más información, vea las propiedades `PrefetchCount` siguientes:

* <a href="https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient.prefetchcount" target="_blank">`Microsoft.ServiceBus.Messaging.QueueClient.PrefetchCount` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.
* <a href="https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.subscriptionclient.prefetchcount" target="_blank">`Microsoft.ServiceBus.Messaging.SubscriptionClient.PrefetchCount` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

---

## <a name="prefetching-and-receivebatch"></a>Captura previa y ReceiveBatch
Aunque los conceptos de realizar una captura previa de varios mensajes juntos tienen una semántica similar la de procesar los mensajes en un lote (`ReceiveBatch`), hay algunas diferencias menores que deben tenerse en cuenta al usar estos dos conceptos conjuntamente.

La captura previa es una configuración (o modo) en el cliente (`QueueClient` y `SubscriptionClient`) y `ReceiveBatch` es una operación (que tiene una semántica de solicitud-respuesta).

Cuando se usen estos enfoques conjuntamente, tenga en cuenta los siguientes casos:

* La captura previa debe ser mayor o igual que el número de mensajes que se espera recibir de `ReceiveBatch`.
* La captura previa puede alcanzar n/3 veces el número de mensajes procesados por segundo, donde n es la duración del bloqueo predeterminada.

El hecho de tener un enfoque ambicioso, es decir, mantener alto el número de capturas previas, presenta algunas dificultades, porque implica que el mensaje se bloquea para un receptor en particular. La recomendación es intentar capturar previamente los valores entre los umbrales mencionados anteriormente e identificar empíricamente lo que se ajuste.

## <a name="multiple-queues"></a>Varias colas

Si una única cola o tema no puede controlar el número esperado, use varias entidades de mensajería. Cuando use varias entidades, cree un cliente dedicado para cada una, en lugar de usar el mismo cliente para todas.

## <a name="development-and-testing-features"></a>Características de desarrollo y pruebas

> [!NOTE]
> Esta sección solo se aplica al SDK de WindowsAzure.ServiceBus, ya que Microsoft.Azure.ServiceBus y Azure.Messaging.ServiceBus no muestran esta funcionalidad.

Service Bus tiene una característica que se utiliza específicamente para desarrollo, que **nunca debe utilizarse en configuraciones de producción**: [`TopicDescription.EnableFilteringMessagesBeforePublishing`][TopicDescription.EnableFiltering].

Cuando se agregan nuevas reglas o filtros al tema, se puede utilizar [`TopicDescription.EnableFilteringMessagesBeforePublishing`][TopicDescription.EnableFiltering] para comprobar que la nueva expresión de filtro funciona según lo esperado.

## <a name="scenarios"></a>Escenarios

En las secciones siguientes, se describen escenarios habituales de mensajería y se explica la configuración preferida de Service Bus. Las tasas de rendimiento se clasifican como pequeñas (menos de 1 mensaje/segundo), moderadas (1 mensaje/segundo o más, pero menos de 100 mensajes/segundo) y altas (100 mensajes/segundo o más). El número de clientes se clasifica como pequeño (5 o menos), moderado (más de 5 pero un máximo de 20) y grande (más de 20).

### <a name="high-throughput-queue"></a>Cola de alto rendimiento

Objetivo: Maximizar el rendimiento de una sola cola. El número de remitentes y receptores es pequeño.

* Para aumentar la tasa general de envío a la cola, use varias factorías de mensajes para crear remitentes. Para cada remitente, use operaciones asincrónicas o varios subprocesos.
* Para aumentar la tasa general de recepción de la cola, use varias factorías de mensajes para crear receptores.
* Use operaciones asincrónicas para aprovechar el procesamiento por lotes del lado cliente.
* Establezca el intervalo de procesamiento por lotes en 50 ms para reducir el número de transmisiones del protocolo de cliente de Service Bus. Si se usan varios remitentes, aumente el intervalo de procesamiento por lotes a 100 ms.
* Deje habilitado el acceso al almacén de procesamiento por lotes. Este acceso aumenta la tasa general con que se pueden escribir mensajes en la cola.
* Establezca el número de capturas previas en el valor resultante de multiplicar por 20 las tasas máximas de procesamiento de todos los receptores de una factoría. Este número reduce la cantidad de transmisiones del protocolo de cliente de Service Bus.

### <a name="multiple-high-throughput-queues"></a>Varias colas de alto rendimiento

Objetivo: Maximizar el rendimiento general de varias colas. El rendimiento de una cola individual es moderado o alto.

Para obtener el máximo rendimiento en varias colas, use la configuración descrita para maximizar el rendimiento de una sola cola. Además, use distintas factorías para crear clientes que envíen o reciban desde distintas colas.

### <a name="low-latency-queue"></a>Cola de baja latencia

Objetivo: Minimizar la latencia de una cola o un tema. El número de remitentes y receptores es pequeño. El rendimiento de la cola es pequeño o moderado.

* Deshabilite el procesamiento por lotes del lado cliente. El cliente envía inmediatamente un mensaje.
* Deshabilite el acceso al almacén de procesamiento por lotes. El servicio escribe inmediatamente el mensaje en el almacén.
* Si usa un solo cliente, establezca el número de capturas previas en el valor resultante de multiplicar por 20 la tasa de procesamiento del receptor. Si llegan varios mensajes a la cola al mismo tiempo, el protocolo de cliente de Service Bus los transmite todos a la vez. Cuando el cliente recibe el siguiente mensaje, ese mensaje ya se encuentra en la memoria caché local. La memoria caché debe ser pequeña.
* Si se usan varios clientes, establezca el número de capturas previas en 0. Al establecer este número, el segundo cliente puede recibir el segundo mensaje mientras el primer cliente todavía está procesando el primero.

### <a name="queue-with-a-large-number-of-senders"></a>Cola con un gran número de remitentes

Objetivo: Maximizar el rendimiento de una cola o un tema con un gran número de remitentes. Cada remitente envía mensajes a una tasa moderada. El número de receptores es pequeño.

Service Bus permite hasta 1000 conexiones simultáneas a una entidad de mensajería. Este límite se aplica en el nivel de espacio de nombres, y las colas, los temas y las suscripciones quedan restringidos por el límite de conexiones simultáneas por espacio de nombres. Para las colas, este número se comparte entre remitentes y receptores. Si se requieren las 1000 conexiones para los remitentes, reemplace la cola por un tema y una única suscripción. Un tema acepta hasta 1000 conexiones simultáneas de los remitentes. La suscripción acepta 1000 conexiones simultáneas adicionales de los destinatarios. Si se requieren más de mil remitentes simultáneos, los remitentes deberían enviar mensajes al protocolo de Service Bus a través de HTTP.

Para maximizar el rendimiento, siga estos pasos:

* Si cada remitente se encuentra en un proceso diferente, use solo una única factoría para cada proceso.
* Use operaciones asincrónicas para aprovechar el procesamiento por lotes del lado cliente.
* Use el intervalo predeterminado de procesamiento por lotes de 20 ms para reducir el número de transmisiones del protocolo de cliente de Service Bus.
* Deje habilitado el acceso al almacén de procesamiento por lotes. Este acceso aumenta la velocidad global con que se pueden escribir mensajes en la cola o el tema.
* Establezca el número de capturas previas en el valor resultante de multiplicar por 20 las tasas máximas de procesamiento de todos los receptores de una factoría. Este número reduce la cantidad de transmisiones del protocolo de cliente de Service Bus.

### <a name="queue-with-a-large-number-of-receivers"></a>Cola con un gran número de receptores

Objetivo: Maximizar la velocidad de recepción de una cola o suscripción con un gran número de receptores. Cada receptor recibe mensajes a una tasa moderada. El número de remitentes es pequeño.

Service Bus permite hasta 1000 conexiones simultáneas a una entidad. Si una cola requiere más de 1000 receptores, reemplace la cola por un tema y varias suscripciones. Cada suscripción admite hasta mil conexiones simultáneas. Como alternativa, los receptores pueden acceder a la cola mediante el protocolo HTTP.

Para maximizar el rendimiento, siga estas instrucciones:

* Si cada receptor se encuentra en un proceso diferente, use solo una única factoría por proceso.
* Los receptores pueden usar operaciones sincrónicas o asincrónicas. Dada la tasa de recepción moderada de un receptor individual, el procesamiento por lotes del lado cliente de una solicitud Completar no afecta al rendimiento del receptor.
* Deje habilitado el acceso al almacén de procesamiento por lotes. Este acceso reduce la carga general de la entidad. También reduce la velocidad global con que se pueden escribir mensajes en la cola o el tema.
* Establezca el número de capturas previas en un valor pequeño (por ejemplo, PrefetchCount = 10). Este número evita que haya receptores inactivos mientras otros tienen un gran número de mensajes almacenados en caché.

### <a name="topic-with-a-few-subscriptions"></a>Tema con pocas suscripciones

Objetivo: maximizar el rendimiento de un tema con pocas suscripciones. Muchas suscripciones reciben un mensaje, lo que significa que la velocidad de recepción combinada de todas las suscripciones es mayor que la velocidad de envío. El número de remitentes es pequeño. El número de receptores por suscripción es pequeño.

Para maximizar el rendimiento, siga estas instrucciones:

* Para aumentar la velocidad de envío general al tema, use varios generadores de mensajes para crear remitentes. Para cada remitente, use operaciones asincrónicas o varios subprocesos.
* Para aumentar la velocidad de recepción general desde una suscripción, use varios generadores de mensajes para crear receptores. Para cada receptor, use operaciones asincrónicas o varios subprocesos.
* Use operaciones asincrónicas para aprovechar el procesamiento por lotes del lado cliente.
* Use el intervalo predeterminado de procesamiento por lotes de 20 ms para reducir el número de transmisiones del protocolo de cliente de Service Bus.
* Deje habilitado el acceso al almacén de procesamiento por lotes. Este acceso aumenta la velocidad general con que se pueden escribir mensajes en el tema.
* Establezca el número de capturas previas en el valor resultante de multiplicar por 20 las tasas máximas de procesamiento de todos los receptores de una factoría. Este número reduce la cantidad de transmisiones del protocolo de cliente de Service Bus.

### <a name="topic-with-a-large-number-of-subscriptions"></a>Tema con un gran número de suscripciones

Objetivo: Maximizar el rendimiento de un tema con un gran número de suscripciones. Muchas suscripciones reciben un mensaje, lo que significa que la velocidad de recepción combinada de todas las suscripciones es mucho mayor que la velocidad de envío. El número de remitentes es pequeño. El número de receptores por suscripción es pequeño.

Los temas con un gran número de suscripciones suelen ofrecer un rendimiento general bajo si todos los mensajes se enrutan a todas las suscripciones. El motivo es que cada mensaje se recibe muchas veces y todos los mensajes de un tema y todas sus suscripciones se almacenan en el mismo almacén. Aquí se supone que el número de remitentes y el número de receptores por suscripción son pequeños. Service Bus admite hasta 2000 suscripciones por tema.

Para maximizar el rendimiento, intente los siguientes pasos:

* Use operaciones asincrónicas para aprovechar el procesamiento por lotes del lado cliente.
* Use el intervalo predeterminado de procesamiento por lotes de 20 ms para reducir el número de transmisiones del protocolo de cliente de Service Bus.
* Deje habilitado el acceso al almacén de procesamiento por lotes. Este acceso aumenta la velocidad general con que se pueden escribir mensajes en el tema.
* Establezca el número de capturas previas en el valor resultante de multiplicar por 20 la tasa de recepción esperada en segundos. Este número reduce la cantidad de transmisiones del protocolo de cliente de Service Bus.

<!-- .NET Standard SDK, Microsoft.Azure.ServiceBus -->
[QueueClient]: /dotnet/api/microsoft.azure.servicebus.queueclient
[MessageSender]: /dotnet/api/microsoft.azure.servicebus.core.messagesender

<!-- .NET Framework SDK, Microsoft.Azure.ServiceBus -->
[MessagingFactory]: /dotnet/api/microsoft.servicebus.messaging.messagingfactory
[BatchFlushInterval]: /dotnet/api/microsoft.servicebus.messaging.messagesender.batchflushinterval
[ForcePersistence]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage.forcepersistence
[EnablePartitioning]: /dotnet/api/microsoft.servicebus.messaging.queuedescription.enablepartitioning
[TopicDescription.EnableFiltering]: /dotnet/api/microsoft.servicebus.messaging.topicdescription.enablefilteringmessagesbeforepublishing

<!-- Local links -->
[Partitioned messaging entities]: service-bus-partitioning.md
