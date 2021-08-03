---
title: Enlace de salida de Azure Event Grid para Azure Functions
description: Aprenda a enviar un evento de Event Grid en Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/14/2020
ms.author: cshoe
ms.custom: devx-track-csharp, fasttrack-edit, devx-track-python
ms.openlocfilehash: 9ebf502fc2ae83651e8f69472b3b2042cef723d8
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110537250"
---
# <a name="azure-event-grid-output-binding-for-azure-functions"></a>Enlace de salida de Azure Event Grid para Azure Functions

Use el enlace de salida de Event Hubs para escribir eventos en un tema personalizado. Debe tener una [clave de acceso válida para el tema personalizado](../event-grid/security-authenticate-publishing-clients.md).

Para obtener información sobre los detalles de instalación y configuración, consulte la [información general](./functions-bindings-event-grid.md).

> [!NOTE]
> El enlace de salida de Event Grid no es compatible con las firmas de acceso compartido (tokens de SAS). Debe usar la clave de acceso del tema.

> [!IMPORTANT]
> El enlace de salida de Event Grid solo está disponible para Functions 2.x y versiones posteriores.

## <a name="example"></a>Ejemplo

# <a name="c"></a>[C#](#tab/csharp)

### <a name="c-2x-and-higher"></a>C# (2.x y versiones posteriores)

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que escribe un mensaje en un tema personalizado de Event Grid usando el valor devuelto del método como resultado:

```csharp
[FunctionName("EventGridOutput")]
[return: EventGrid(TopicEndpointUri = "MyEventGridTopicUriSetting", TopicKeySetting = "MyEventGridTopicKeySetting")]
public static EventGridEvent Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, ILogger log)
{
    return new EventGridEvent("message-id", "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0");
}
```

En el siguiente ejemplo se muestra cómo usar la interfaz `IAsyncCollector` para enviar un lote de mensajes.

```csharp
[FunctionName("EventGridAsyncOutput")]
public static async Task Run(
    [TimerTrigger("0 */5 * * * *")] TimerInfo myTimer,
    [EventGrid(TopicEndpointUri = "MyEventGridTopicUriSetting", TopicKeySetting = "MyEventGridTopicKeySetting")]IAsyncCollector<EventGridEvent> outputEvents,
    ILogger log)
{
    for (var i = 0; i < 3; i++)
    {
        var myEvent = new EventGridEvent("message-id-" + i, "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0");
        await outputEvents.AddAsync(myEvent);
    }
}
```

### <a name="version-3x-preview"></a>Versión 3.x (versión preliminar)

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) de Functions 3.x que se enlaza a `CloudEvent`:

```cs
using System.Threading.Tasks;
using Azure.Messaging;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;
using Microsoft.Azure.WebJobs.Extensions.Http;

namespace Azure.Extensions.WebJobs.Sample
{
    public static class CloudEventBindingFunction
    {
        [FunctionName("CloudEventBindingFunction")]
        public static async Task<IActionResult> RunAsync(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
            [EventGrid(TopicEndpointUri = "EventGridEndpoint", TopicKeySetting = "EventGridKey")] IAsyncCollector<CloudEvent> eventCollector)
        {
            CloudEvent e = new CloudEvent("IncomingRequest", "IncomingRequest", await req.ReadAsStringAsync());
            await eventCollector.AddAsync(e);
            return new OkResult();
        }
    }
}
```

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) de Functions 3.x que se enlaza a `EventGridEvent`:

```cs
using System.Threading.Tasks;
using Azure.Messaging.EventGrid;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Http;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;

namespace Azure.Extensions.WebJobs.Sample
{
    public static class EventGridEventBindingFunction
    {
        [FunctionName("EventGridEventBindingFunction")]
        public static async Task<IActionResult> RunAsync(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
            [EventGrid(TopicEndpointUri = "EventGridEndpoint", TopicKeySetting = "EventGridKey")] IAsyncCollector<EventGridEvent> eventCollector)
        {
            EventGridEvent e = new EventGridEvent(await req.ReadAsStringAsync(), "IncomingRequest", "IncomingRequest", "1.0.0");
            await eventCollector.AddAsync(e);
            return new OkResult();
        }
    }
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

En el ejemplo siguiente se muestran los datos del enlace de salida de Event Grid en el archivo *function.json*.

```json
{
    "type": "eventGrid",
    "name": "outputEvent",
    "topicEndpointUri": "MyEventGridTopicUriSetting",
    "topicKeySetting": "MyEventGridTopicKeySetting",
    "direction": "out"
}
```

Este es el código de script de C# que crea un evento:

```cs
#r "Microsoft.Azure.EventGrid"
using System;
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Extensions.Logging;

public static void Run(TimerInfo myTimer, out EventGridEvent outputEvent, ILogger log)
{
    outputEvent = new EventGridEvent("message-id", "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0");
}
```

Este es el código de script de C# que crea varios eventos:

```cs
#r "Microsoft.Azure.EventGrid"
using System;
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Extensions.Logging;

public static void Run(TimerInfo myTimer, ICollector<EventGridEvent> outputEvent, ILogger log)
{
    outputEvent.Add(new EventGridEvent("message-id-1", "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0"));
    outputEvent.Add(new EventGridEvent("message-id-2", "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0"));
}
```

# <a name="java"></a>[Java](#tab/java)

El enlace de salida Event Grid no está disponible para Java.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

En el ejemplo siguiente se muestran los datos del enlace de salida de Event Grid en el archivo *function.json*.

```json
{
    "type": "eventGrid",
    "name": "outputEvent",
    "topicEndpointUri": "MyEventGridTopicUriSetting",
    "topicKeySetting": "MyEventGridTopicKeySetting",
    "direction": "out"
}
```

Este es el código de JavaScript que crea un único evento:

```javascript
module.exports = async function (context, myTimer) {
    var timeStamp = new Date().toISOString();

    context.bindings.outputEvent = {
        id: 'message-id',
        subject: 'subject-name',
        dataVersion: '1.0',
        eventType: 'event-type',
        data: "event-data",
        eventTime: timeStamp
    };
    context.done();
};
```

Este es el código de JavaScript que crea varios eventos:

```javascript
module.exports = function(context) {
    var timeStamp = new Date().toISOString();

    context.bindings.outputEvent = [];

    context.bindings.outputEvent.push({
        id: 'message-id-1',
        subject: 'subject-name',
        dataVersion: '1.0',
        eventType: 'event-type',
        data: "event-data",
        eventTime: timeStamp
    });
    context.bindings.outputEvent.push({
        id: 'message-id-2',
        subject: 'subject-name',
        dataVersion: '1.0',
        eventType: 'event-type',
        data: "event-data",
        eventTime: timeStamp
    });
    context.done();
};
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

En el ejemplo siguiente se muestra cómo configurar una función para generar un mensaje de evento de Event Grid. La sección donde `type` está establecida en `eventGrid` permite configurar los valores necesarios para establecer un enlace de salida de Event Grid.

```powershell
{
  "bindings": [
    {
      "type": "eventGrid",
      "name": "outputEvent",
      "topicEndpointUri": "MyEventGridTopicUriSetting",
      "topicKeySetting": "MyEventGridTopicKeySetting",
      "direction": "out"
    },
    {
      "authLevel": "anonymous",
      "type": "httpTrigger",
      "direction": "in",
      "name": "Request",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "Response"
    }
  ]
}
```

En la función, use `Push-OutputBinding` para enviar un evento a un tema personalizado a través del enlace de salida de Event Grid.

```powershell
using namespace System.Net

# Input bindings are passed in via param block.
param($Request, $TriggerMetadata)

# Write to the Azure Functions log stream.
Write-Host "PowerShell HTTP trigger function processed a request."

# Interact with query parameters or the body of the request.
$message = $Request.Query.Message

Push-OutputBinding -Name outputEvent -Value  @{
    id = "1"
    EventType = "testEvent"
    Subject = "testapp/testPublish"
    EventTime = "2020-08-27T21:03:07+00:00"
    Data = @{
        Message = $message
    }
    DataVersion = "1.0"
}

Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = 200
    Body = "OK"
})
```

# <a name="python"></a>[Python](#tab/python)

En el ejemplo siguiente se muestra un enlace de desencadenador en un archivo *function.json* y una [función de Python](functions-reference-python.md) que usa el enlace. Después, envía un evento al tema personalizado, tal y como se especifica en `topicEndpointUri`.

Estos son los datos de enlace del archivo *function.json*:

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "type": "eventGridTrigger",
      "name": "eventGridEvent",
      "direction": "in"
    },
    {
      "type": "eventGrid",
      "name": "outputEvent",
      "topicEndpointUri": "MyEventGridTopicUriSetting",
      "topicKeySetting": "MyEventGridTopicKeySetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

Este es el ejemplo de Python para enviar un evento a un tema personalizado estableciendo `EventGridOutputEvent`:

```python
import logging
import azure.functions as func
import datetime

def main(eventGridEvent: func.EventGridEvent, 
         outputEvent: func.Out[func.EventGridOutputEvent]) -> None:

    logging.log("eventGridEvent: ", eventGridEvent)

    outputEvent.set(
        func.EventGridOutputEvent(
            id="test-id",
            data={"tag1": "value1", "tag2": "value2"},
            subject="test-subject",
            event_type="test-event-1",
            event_time=datetime.datetime.utcnow(),
            data_version="1.0"))
```

---

## <a name="attributes-and-annotations"></a>Atributos y anotaciones

# <a name="c"></a>[C#](#tab/csharp)

En las [bibliotecas de clases de C#](functions-dotnet-class-library.md), use el atributo [EventGridAttribute](https://github.com/Azure/azure-functions-eventgrid-extension/blob/dev/src/EventGridExtension/OutputBinding/EventGridAttribute.cs).

El constructor del atributo toma el nombre de un valor de la aplicación que contiene el nombre del tema personalizado y el nombre de un valor de la aplicación que contiene la clave del tema. Para obtener más información sobre estas configuraciones, vea [Salida: configuración](#configuration). Este es un ejemplo de atributo `EventGrid`:

```csharp
[FunctionName("EventGridOutput")]
[return: EventGrid(TopicEndpointUri = "MyEventGridTopicUriSetting", TopicKeySetting = "MyEventGridTopicKeySetting")]
public static string Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, ILogger log)
{
    ...
}
```

Para ver un ejemplo completo, consulte el [ejemplo](#example).

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

El script de C# no admite atributos.

# <a name="java"></a>[Java](#tab/java)

El enlace de salida Event Grid no está disponible para Java.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

JavaScript no admite atributos.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

PowerShell no admite atributos.

# <a name="python"></a>[Python](#tab/python)

El enlace de salida de Event Grid no está disponible para Python.

---

## <a name="configuration"></a>Configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se definen en el archivo *function.json* y el atributo `EventGrid`.

|Propiedad de function.json | Propiedad de atributo |Descripción|
|---------|---------|----------------------|
|**type** | N/D | Debe establecerse en "eventGrid". |
|**direction** | N/D | Debe establecerse en "out". Este parámetro se establece automáticamente cuando se crea el enlace en Azure Portal. |
|**name** | N/D | Nombre de la variable que se usa en el código de la función que representa el evento. |
|**topicEndpointUri** |**TopicEndpointUri** | Nombre de un valor de aplicación que contiene el URI para el tema personalizado, como `MyTopicEndpointUri`. |
|**topicKeySetting** |**TopicKeySetting** | Nombre de un valor de aplicación que contiene una clave de acceso para el tema personalizado. |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

> [!IMPORTANT]
> Asegúrese de establecer el valor de la propiedad de configuración `TopicEndpointUri` en el nombre de un valor de la aplicación que contiene el URI del tema personalizado. No especifique el URI del tema personalizado directamente en esta propiedad.

## <a name="usage"></a>Uso

# <a name="c"></a>[C#](#tab/csharp)

Envíe mensajes mediante un parámetro de método, como `out EventGridEvent paramName`. Para escribir varios mensajes, puede usar `ICollector<EventGridEvent>` o `IAsyncCollector<EventGridEvent>` en lugar de `out EventGridEvent`.

### <a name="additional-types"></a>Tipos adicionales 
Las aplicaciones que usan la versión 3.0.0 o posterior de la extensión de Event Grid usan el tipo `EventGridEvent` del espacio de nombres [Azure.Messaging.EventGrid](/dotnet/api/azure.messaging.eventgrid.eventgridevent). Además, puede enlazar al tipo `CloudEvent` desde el espacio de nombres [Azure.Messaging](/dotnet/api/azure.messaging.cloudevent).

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

Envíe mensajes mediante un parámetro de método, como `out EventGridEvent paramName`. En script de C#, `paramName` es el valor especificado en la propiedad `name` de *function.json*. Para escribir varios mensajes, puede usar `ICollector<EventGridEvent>` o `IAsyncCollector<EventGridEvent>` en lugar de `out EventGridEvent`.

### <a name="additional-types"></a>Tipos adicionales 
Las aplicaciones que usan la versión 3.0.0 o posterior de la extensión de Event Grid usan el tipo `EventGridEvent` del espacio de nombres [Azure.Messaging.EventGrid](/dotnet/api/azure.messaging.eventgrid.eventgridevent). Además, puede enlazar al tipo `CloudEvent` desde el espacio de nombres [Azure.Messaging](/dotnet/api/azure.messaging.cloudevent).

# <a name="java"></a>[Java](#tab/java)

El enlace de salida Event Grid no está disponible para Java.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Acceda al evento de salida mediante `context.bindings.<name>`, donde `<name>` es el valor especificado en la propiedad `name` de *function.json*.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Acceda al evento de salida mediante el commandlet `Push-OutputBinding` para enviar un evento al enlace de salida de Event Grid.

# <a name="python"></a>[Python](#tab/python)

El enlace de salida de Event Grid no está disponible para Python.

---

## <a name="next-steps"></a>Pasos siguientes

* [Envío de un evento de Event Grid](./functions-bindings-event-grid-trigger.md)
