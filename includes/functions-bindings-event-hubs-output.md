---
author: craigshoemaker
ms.service: azure-functions
ms.topic: include
ms.date: 02/21/2020
ms.author: cshoe
ms.openlocfilehash: bc2bec364f8d752b7416ecccf0b00d0fbec4c8e8
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105729901"
---
Use el enlace de salida de Event Hubs para escribir eventos en una secuencia. Debe tener permiso de envío a un centro de eventos para escribir eventos en él.

Asegúrese de que las referencias de paquete necesarias están implementadas antes de tratar de implementar un enlace de salida.

<a id="example" name="example"></a>

# <a name="c"></a>[C#](#tab/csharp)

En el ejemplo siguiente se muestra una [función de C#](../articles/azure-functions/functions-dotnet-class-library.md) que escribe un mensaje en un centro de eventos usando el valor devuelto del método como resultado:

```csharp
[FunctionName("EventHubOutput")]
[return: EventHub("outputEventHubMessage", Connection = "EventHubConnectionAppSetting")]
public static string Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, ILogger log)
{
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
    return $"{DateTime.Now}";
}
```

En el siguiente ejemplo se muestra cómo usar la interfaz `IAsyncCollector` para enviar un lote de mensajes. Este escenario es habitual cuando se procesan mensajes procedentes de un centro de eventos y el resultado se envía a otro centro de eventos.

```csharp
[FunctionName("EH2EH")]
public static async Task Run(
    [EventHubTrigger("source", Connection = "EventHubConnectionAppSetting")] EventData[] events,
    [EventHub("dest", Connection = "EventHubConnectionAppSetting")]IAsyncCollector<string> outputEvents,
    ILogger log)
{
    foreach (EventData eventData in events)
    {
        // do some processing:
        var myProcessedEvent = DoSomething(eventData);

        // then send the message
        await outputEvents.AddAsync(JsonConvert.SerializeObject(myProcessedEvent));
    }
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

En el ejemplo siguiente se muestra un enlace de desencadenador de centro de eventos en un archivo *function.json* y una [función de script de C#](../articles/azure-functions/functions-reference-csharp.md) que usa el enlace. La función escribe un mensaje a un centro de eventos.

Los ejemplos siguientes muestran datos de enlace de Event Hubs en el archivo *function.json*. El primer ejemplo es para Functions 2.x y posteriores, y el segundo para Functions 1.x. 

```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "eventHubName": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "path": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

Este es el código de script de C# que crea un mensaje:

```cs
using System;
using Microsoft.Extensions.Logging;

public static void Run(TimerInfo myTimer, out string outputEventHubMessage, ILogger log)
{
    String msg = $"TimerTriggerCSharp1 executed at: {DateTime.Now}";
    log.LogInformation(msg);   
    outputEventHubMessage = msg;
}
```

Este es el código de script de C# que crea varios mensajes:

```cs
public static void Run(TimerInfo myTimer, ICollector<string> outputEventHubMessage, ILogger log)
{
    string message = $"Message created at: {DateTime.Now}";
    log.LogInformation(message);
    outputEventHubMessage.Add("1 " + message);
    outputEventHubMessage.Add("2 " + message);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

En el ejemplo siguiente se muestra un enlace de desencadenador de centro de eventos en un archivo *function.json* y una [función de JavaScript](../articles/azure-functions/functions-reference-node.md) que usa el enlace. La función escribe un mensaje a un centro de eventos.

Los ejemplos siguientes muestran datos de enlace de Event Hubs en el archivo *function.json*. El primer ejemplo es para Functions 2.x y posteriores, y el segundo para Functions 1.x. 

```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "eventHubName": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "path": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

Este es el código JavaScript que envía un mensaje:

```javascript
module.exports = function (context, myTimer) {
    var timeStamp = new Date().toISOString();
    context.log('Message created at: ', timeStamp);   
    context.bindings.outputEventHubMessage = "Message created at: " + timeStamp;
    context.done();
};
```

Este es el código JavaScript que envía varios mensajes:

```javascript
module.exports = function(context) {
    var timeStamp = new Date().toISOString();
    var message = 'Message created at: ' + timeStamp;

    context.bindings.outputEventHubMessage = [];

    context.bindings.outputEventHubMessage.push("1 " + message);
    context.bindings.outputEventHubMessage.push("2 " + message);
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

En el ejemplo siguiente se muestra un enlace de desencadenador de centro de eventos en un archivo *function.json* y una [función de Python](../articles/azure-functions/functions-reference-python.md) que usa el enlace. La función escribe un mensaje a un centro de eventos.

Los ejemplos siguientes muestran datos de enlace de Event Hubs en el archivo *function.json*.

```json
{
    "type": "eventHub",
    "name": "$return",
    "eventHubName": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

Este es el código de Python que envía un solo mensaje:

```python
import datetime
import logging
import azure.functions as func


def main(timer: func.TimerRequest) -> str:
    timestamp = datetime.datetime.utcnow()
    logging.info('Message created at: %s', timestamp)
    return 'Message created at: {}'.format(timestamp)
```

# <a name="java"></a>[Java](#tab/java)

El ejemplo siguiente muestra una función de Java que escribe un mensaje que contiene la hora actual en un centro de eventos.

```java
@FunctionName("sendTime")
@EventHubOutput(name = "event", eventHubName = "samples-workitems", connection = "AzureEventHubConnection")
public String sendTime(
   @TimerTrigger(name = "sendTimeTrigger", schedule = "0 */5 * * * *") String timerInfo)  {
     return LocalDateTime.now().toString();
 }
```

En la [biblioteca en tiempo de ejecución de funciones de Java](/java/api/overview/azure/functions/runtime), utilice la anotación `@EventHubOutput` en los parámetros cuyo valor se publicaría en el centro de eventos.  El parámetro debe ser del tipo `OutputBinding<T>`, donde T es un tipo POJO o cualquier tipo nativo de Java.

---

## <a name="attributes-and-annotations"></a>Atributos y anotaciones

# <a name="c"></a>[C#](#tab/csharp)

En las [bibliotecas de clases de C#](../articles/azure-functions/functions-dotnet-class-library.md), use el atributo [EventHubAttribute](https://github.com/Azure/azure-functions-eventhubs-extension/blob/master/src/Microsoft.Azure.WebJobs.Extensions.EventHubs/EventHubAttribute.cs).

El constructor del atributo toma el nombre del centro de eventos y el nombre de una configuración de aplicación que contenga la cadena de conexión. Para obtener más información sobre estas configuraciones, vea [Salida: configuración](#configuration). Este es un ejemplo de atributo `EventHub`:

```csharp
[FunctionName("EventHubOutput")]
[return: EventHub("outputEventHubMessage", Connection = "EventHubConnectionAppSetting")]
public static string Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, ILogger log)
{
    ...
}
```

Para obtener un ejemplo completo, consulte [Salida: ejemplo de C#](#example).

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

El script de C# no admite atributos.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

JavaScript no admite atributos.

# <a name="python"></a>[Python](#tab/python)

Python no admite atributos.

# <a name="java"></a>[Java](#tab/java)

En la [biblioteca en tiempo de ejecución de funciones de Java](/java/api/overview/azure/functions/runtime), utilice la anotación [EventHubOutput](/java/api/com.microsoft.azure.functions.annotation.eventhuboutput) en los parámetros cuyo valor se publicaría en el centro de eventos. El parámetro debe ser de tipo `OutputBinding<T>`, donde `T` es un tipo POJO o cualquier tipo nativo de Java.

---

## <a name="configuration"></a>Configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `EventHub`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
|**type** | N/D | Debe establecerse en "eventHub". |
|**direction** | N/D | Debe establecerse en "out". Este parámetro se establece automáticamente cuando se crea el enlace en Azure Portal. |
|**name** | N/D | Nombre de la variable que se usa en el código de la función que representa el evento. |
|**path** |**EventHubName** | Solo Functions 1.x. El nombre del centro de eventos. Cuando el nombre del centro de eventos también está presente en la cadena de conexión, ese valor reemplaza esta propiedad en tiempo de ejecución. |
|**eventHubName** |**EventHubName** | Functions 2.x y versiones posteriores. El nombre del centro de eventos. Cuando el nombre del centro de eventos también está presente en la cadena de conexión, ese valor reemplaza esta propiedad en tiempo de ejecución. |
|**connection** |**Connection** | El nombre de una configuración de aplicación que contenga la cadena de conexión para el espacio de nombres del centro de eventos. Copie esta cadena de conexión haciendo clic en el botón **Información de conexión** del [espacio de nombres](../articles/event-hubs/event-hubs-create.md#create-an-event-hubs-namespace), no del propio centro de eventos. Esta cadena de conexión debe tener permisos de envío para enviar el mensaje a la secuencia de eventos. <br><br>Si usa [la versión 5.x o superior de la extensión](../articles/azure-functions/functions-bindings-event-hubs.md#event-hubs-extension-5x-and-higher), en lugar de una cadena de conexión puede proporcionar una referencia a una sección de configuración que defina la conexión. Consulte [Conexiones](../articles/azure-functions/functions-reference.md#connections).|

[!INCLUDE [app settings to local.settings.json](../articles/azure-functions/../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Uso

# <a name="c"></a>[C#](#tab/csharp)

### <a name="default"></a>Valor predeterminado

Puede usar los siguientes tipos de parámetro para el enlace de salida del centro de eventos:

* `string`
* `byte[]`
* `POCO`
* `EventData` - Las propiedades predeterminadas de EventData se proporcionan para el [espacio de nombres Microsoft.Azure.EventHubs](/dotnet/api/microsoft.azure.eventhubs.eventdata).

Envíe mensajes mediante un parámetro de método, como `out string paramName`. En script de C#, `paramName` es el valor especificado en la propiedad `name` de *function.json*. Para escribir varios mensajes, puede usar `ICollector<string>` o `IAsyncCollector<string>` en lugar de `out string`.

### <a name="additional-types"></a>Tipos adicionales 
Las aplicaciones que usan la versión 5.0.0 o posterior de la extensión del centro de eventos utilizan el tipo `EventData` en [Azure.Messaging.EventHubs](/dotnet/api/azure.messaging.eventhubs.eventdata) en lugar del que está en el [espacio de nombres Microsoft.Azure.EventHubs](/dotnet/api/microsoft.azure.eventhubs.eventdata). En esta versión se elimina la compatibilidad con el tipo `Body` heredado en favor de los siguientes tipos:

- [EventBody](/dotnet/api/azure.messaging.eventhubs.eventdata.eventbody)

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

### <a name="default"></a>Valor predeterminado

Puede usar los siguientes tipos de parámetro para el enlace de salida del centro de eventos:

* `string`
* `byte[]`
* `POCO`
* `EventData` - Las propiedades predeterminadas de EventData se proporcionan para el [espacio de nombres Microsoft.Azure.EventHubs](/dotnet/api/microsoft.azure.eventhubs.eventdata).

Envíe mensajes mediante un parámetro de método, como `out string paramName`. En script de C#, `paramName` es el valor especificado en la propiedad `name` de *function.json*. Para escribir varios mensajes, puede usar `ICollector<string>` o `IAsyncCollector<string>` en lugar de `out string`.

### <a name="additional-types"></a>Tipos adicionales 
Las aplicaciones que usan la versión 5.0.0 o posterior de la extensión del centro de eventos utilizan el tipo `EventData` en [Azure.Messaging.EventHubs](/dotnet/api/azure.messaging.eventhubs.eventdata) en lugar del que está en el [espacio de nombres Microsoft.Azure.EventHubs](/dotnet/api/microsoft.azure.eventhubs.eventdata). En esta versión se elimina la compatibilidad con el tipo `Body` heredado en favor de los siguientes tipos:

- [EventBody](/dotnet/api/azure.messaging.eventhubs.eventdata.eventbody)

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Acceda al evento de salida mediante `context.bindings.<name>`, donde `<name>` es el valor especificado en la propiedad `name` de *function.json*.

# <a name="python"></a>[Python](#tab/python)

Hay dos opciones para la generación de un mensaje del centro de eventos desde una función:

- **Valor devuelto**: Establezca la propiedad `name` de *function.json* en `$return`. Con esta configuración, el valor devuelto de la función se conserva como mensaje del centro de eventos.

- **Imperativa**: Pase un valor al método [set](/python/api/azure-functions/azure.functions.out#set-val--t-----none) del parámetro declarado como tipo [Out](/python/api/azure-functions/azure.functions.out). El valor pasado a `set` se conserva como mensaje del centro de eventos.

# <a name="java"></a>[Java](#tab/java)

Hay dos opciones para la generación de un mensaje del centro de eventos desde una función mediante la anotación [EventHubOutput](/java/api/com.microsoft.azure.functions.annotation.eventhuboutput):

- **Valor devuelto**: Al aplicar la anotación a la propia función, el valor devuelto de la función se conserva como un mensaje del centro de eventos.

- **Imperativa**: Para establecer explícitamente el valor del mensaje, aplique la anotación a un parámetro específico del tipo [`OutputBinding<T>`](/java/api/com.microsoft.azure.functions.OutputBinding), donde `T` es un POJO o cualquier tipo de Java nativo. Con esta configuración, pasar un valor al método `setValue` conserva el valor como un mensaje del centro de eventos.

---

## <a name="exceptions-and-return-codes"></a>Excepciones y códigos de retorno

| Enlace | Referencia |
|---|---|
| Centro de eventos | [Guía de operaciones](/rest/api/eventhub/publisher-policy-operations) |