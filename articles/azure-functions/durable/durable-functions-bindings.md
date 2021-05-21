---
title: 'Enlaces para Durable Functions: Azure'
description: Aprenda a utilizar desencadenadores y enlaces en la extensión Durable Functions para Azure Functions.
ms.topic: conceptual
ms.date: 05/07/2021
ms.author: azfuncdf
ms.openlocfilehash: a07748f996788825b21b5c23a117954085dadcbf
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109656945"
---
# <a name="bindings-for-durable-functions-azure-functions"></a>Enlaces para Durable Functions (Azure Functions)

La extensión [Durable Functions](durable-functions-overview.md) presenta tres enlaces de desencadenador que controlan la ejecución de las funciones de orquestador, entidad y actividad. También presenta a un enlace de salida que actúa como un cliente para el tiempo de ejecución de Durable Functions.

## <a name="orchestration-trigger"></a>Desencadenador de orquestación

El desencadenador de orquestación permite crear [funciones de orquestador durables](durable-functions-types-features-overview.md#orchestrator-functions). Este desencadenador se ejecuta cuando se programa una nueva instancia de orquestación o cuando una ya existente recibe un evento. Entre los ejemplos de eventos que pueden desencadenar funciones de orquestador se incluyen expiraciones de temporizadores de larga duración, respuestas de funciones de actividades y eventos desencadenados por clientes externos.

Al crear funciones en .NET, el desencadenador de orquestación se configura mediante el atributo [OrchestrationTriggerAttribute](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.orchestrationtriggerattribute) de .NET.

Al escribir funciones de orquestador en lenguajes de scripting como JavaScript, Python o PowerShell, el desencadenador de orquestación se define mediante el objeto JSON siguiente de la matriz `bindings` del archivo *function.json*:

```json
{
    "name": "<Name of input parameter in function signature>",
    "orchestration": "<Optional - name of the orchestration>",
    "type": "orchestrationTrigger",
    "direction": "in"
}
```

* `orchestration` es el nombre de la orquestación que los clientes tienen que usar cuando quieren iniciar nuevas instancias de esta función de orquestador. Esta propiedad es opcional. Si no se especifica, se utiliza el nombre de la función.

Internamente, este enlace de desencadenador sondea el almacén de larga duración configurado en busca de nuevos eventos de orquestación, como eventos de inicio de orquestación, eventos de expiración de temporizadores de larga duración, eventos de respuestas de funciones de actividades y eventos externos desencadenados por otras funciones.

### <a name="trigger-behavior"></a>Comportamiento de un desencadenador

Estas son algunas notas acerca del desencadenador de orquestación:

* **Subprocesamiento único**: un único subproceso de distribución se usa para toda la ejecución de función del orquestador en una instancia de host. Por esta razón, es importante asegurarse de que el código de función del orquestador es eficaz y no realiza ninguna E/S. También es importante asegurarse de que este subproceso no realizar ningún trabajo asincrónico excepto cuando se espera en tipos de tareas específicos de Durable Functions.
* **Control de mensajes dudosos**: no hay compatibilidad con mensajes dudosos en los desencadenadores de orquestación.
* **Visibilidad de los mensajes**: los mensajes de desencadenador de orquestación se quitan de la cola y se mantienen invisibles durante un tiempo configurable. La visibilidad de estos mensajes se renueva automáticamente siempre que la aplicación de la función se esté ejecutando y se mantenga correcta.
* **Valores devueltos**: los valores devueltos se serializan en JSON y se conservan en la tabla de historial de orquestación en Azure Table Storage. Estos valores devueltos pueden ser consultados por el enlace de cliente de orquestación que se describe más adelante.

> [!WARNING]
> Las funciones del orquestador nunca deben usar ningún enlace de entrada o salida que no sea el enlace de desencadenador de orquestación. Si lo hacen, existe la posibilidad de que se produzcan problemas con la extensión Durable Task porque esos enlaces pueden no seguir las reglas de E/S y subprocesamiento único. Si desea usar otros enlaces, agréguelos a una función de actividad llamada desde la función de orquestador. Para más información sobre las restricciones de código de las funciones de orquestador, consulte [Restricciones de código de las funciones de orquestador](durable-functions-code-constraints.md).

> [!WARNING]
> Las funciones de orquestador de JavaScript y Python nunca se deberían declarar `async`.

### <a name="trigger-usage"></a>Uso del desencadenador

El enlace de desencadenador de orquestación admite entradas y salidas. Estas son algunas cosas que hay que saber acerca del control de entradas y salidas:

* **entradas**: los desencadenadores de orquestación se pueden invocar con entradas a las que se accede a través del objeto de entrada de contexto. Todas las entradas tienen que ser serializables con JSON.
* **salidas**: los desencadenadores de orquestación admiten valores de salida, así como de entrada. El valor devuelto de la función se utiliza para asignar el valor de salida y tiene que ser serializable con JSON.

### <a name="trigger-sample"></a>Ejemplo de desencadenador

El código de ejemplo siguiente muestra el aspecto que podría tener la función del orquestador más simple "Hola mundo":

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("HelloWorld")]
public static string Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string name = context.GetInput<string>();
    // ... do some work ...
    return $"Hello {name}!";
}
```

> [!NOTE]
> El código anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar `DurableOrchestrationContext` en lugar de `IDurableOrchestrationContext`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const name = context.df.getInput();
    // ... do some work ...
    return `Hello ${name}!`;
});
```

> [!NOTE]
> La biblioteca `durable-functions` se encarga de llamar al método `context.done` cuando se cierra la función del generador.

# <a name="python"></a>[Python](#tab/python)

```python
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    input_ = context.get_input()
    # Do some work
    return f"Hello {name}!"

main = df.Orchestrator.create(orchestrator_function)
```

---

La mayoría de las funciones del orquestador llaman a funciones de actividad, por lo que aquí tenemos un ejemplo de "Hola mundo" que muestra cómo llamar a una función de actividad:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("HelloWorld")]
public static async Task<string> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string name = context.GetInput<string>();
    string result = await context.CallActivityAsync<string>("SayHello", name);
    return result;
}
```

> [!NOTE]
> El código anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar `DurableOrchestrationContext` en lugar de `IDurableOrchestrationContext`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const name = context.df.getInput();
    const result = yield context.df.callActivity("SayHello", name);
    return result;
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    input_ = context.get_input()
    result = yield context.call_activity('SayHello', name)
    return result

main = df.Orchestrator.create(orchestrator_function)
```

---

## <a name="activity-trigger"></a>Desencadenador de actividad

El desencadenador de actividad le permite crear funciones que las funciones del orquestador llaman, conocidas como [funciones de actividad](durable-functions-types-features-overview.md#activity-functions).

Si va a crear funciones en .NET, el desencadenador de actividad se configura mediante el atributo [ActvityTriggerAttribute](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.activitytriggerattribute) de .NET.

Si utiliza JavaScript, Python o PowerShell, el desencadenador de actividad se definirá mediante el objeto JSON siguiente en la matriz `bindings` de *function.json*:

```json
{
    "name": "<Name of input parameter in function signature>",
    "activity": "<Optional - name of the activity>",
    "type": "activityTrigger",
    "direction": "in"
}
```

* `activity`es el nombre de la actividad. Este valor es el nombre que utilizan las funciones del orquestador para invocar esta función de actividad. Esta propiedad es opcional. Si no se especifica, se utiliza el nombre de la función.

Internamente, este enlace de desencadenador sondea el almacén de larga duración configurado para los nuevos eventos de ejecución de actividad.

### <a name="trigger-behavior"></a>Comportamiento de un desencadenador

Estas son algunas notas acerca del desencadenador de actividad:

* **Subprocesamiento**: a diferencia del desencadenador de orquestación, los desencadenadores de actividad no tienen ninguna restricción en relación a los subprocesos o E/S. Se puede tratar como funciones normales.
* **Control de mensajes dudosos**: no hay compatibilidad con mensajes dudosos en los desencadenadores de actividad.
* **Visibilidad de los mensajes**: los mensajes de desencadenador de actividad se quitan de la cola y se mantienen invisibles durante un tiempo configurable. La visibilidad de estos mensajes se renueva automáticamente siempre que la aplicación de la función se esté ejecutando y se mantenga correcta.
* **Valores devueltos**: los valores devueltos se serializan en JSON y se conservan en el almacenamiento de larga duración configurado.

### <a name="trigger-usage"></a>Uso del desencadenador

El enlace de desencadenador de actividad admite entradas y salidas como el desencadenador de orquestación. Estas son algunas cosas que hay que saber acerca del control de entradas y salidas:

* **entradas:** los desencadenadores de actividad se pueden invocar con entradas desde una función del orquestador. Todas las entradas tienen que ser serializables con JSON.
* **salidas**: las funciones de actividad admiten valores de salida, así como de entrada. El valor devuelto de la función se utiliza para asignar el valor de salida y tiene que ser serializable con JSON.
* **metadatos**: las funciones de actividad de .NET pueden enlazar a un parámetro `string instanceId` para obtener el identificador de la instancia de la orquestación que llama.

### <a name="trigger-sample"></a>Ejemplo de desencadenador

El ejemplo de código siguiente muestra el aspecto que podría tener una función de actividad simple `SayHello`:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("SayHello")]
public static string SayHello([ActivityTrigger] IDurableActivityContext helloContext)
{
    string name = helloContext.GetInput<string>();
    return $"Hello {name}!";
}
```

El tipo de parámetro predeterminado para el enlace `ActivityTriggerAttribute` de .NET es [IDurableActivityContext](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.idurableactivitycontext) (o [DurableActivityContext](/dotnet/api/microsoft.azure.webjobs.durableactivitycontext?view=azure-dotnet-legacy&preserve-view=true) para Durable Functions v1). Sin embargo, los desencadenadores de actividad de .NET también admiten enlazar directamente con tipos serializables con JSON (incluidos los tipos primitivos), por lo que la misma función podría simplificarse como sigue:

```csharp
[FunctionName("SayHello")]
public static string SayHello([ActivityTrigger] string name)
{
    return $"Hello {name}!";
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
module.exports = async function(context) {
    return `Hello ${context.bindings.name}!`;
};
```

Los enlaces de JavaScript también se pueden pasar como parámetros adicionales, por lo que la misma función podría simplificarse como sigue:

```javascript
module.exports = async function(context, name) {
    return `Hello ${name}!`;
};
```

# <a name="python"></a>[Python](#tab/python)

```python
def main(name: str) -> str:
    return f"Hello {name}!"
```

---

### <a name="using-input-and-output-bindings"></a>Uso de enlaces de entrada y de salida

Puede usar los enlaces de entrada y salida normales además del enlace de desencadenador de actividad. Por ejemplo, puede tomar la entrada del enlace de actividad y enviar un mensaje a un objeto EventHub mediante el enlace de salida de EventHub:

```json
{
  "bindings": [
    {
      "name": "message",
      "type": "activityTrigger",
      "direction": "in"
    },
    {
      "type": "eventHub",
      "name": "outputEventHubMessage",
      "connection": "EventhubConnectionSetting",
      "eventHubName": "eh_messages",
      "direction": "out"
  }
  ]
}
```

```javascript
module.exports = async function (context) {
    context.bindings.outputEventHubMessage = context.bindings.message;
};
```

## <a name="orchestration-client"></a>Cliente de orquestación

El enlace del cliente de orquestación le permite escribir funciones que interactúan con las funciones del orquestador. Estas funciones se conocen a menudo como [funciones de cliente](durable-functions-types-features-overview.md#client-functions). Por ejemplo, puede actuar en las instancias de orquestación de las siguientes formas:

* Iniciarlas.
* Consultar su estado.
* Finalizarlas.
* Enviarles eventos mientras se están ejecutando.
* Purgar del historial de instancias.

Si usa .NET, puede enlazar con el cliente de orquestación mediante el atributo [DurableClientAttribute](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.durableclientattribute) ([OrchestrationClientAttribute](/dotnet/api/microsoft.azure.webjobs.orchestrationclientattribute?view=azure-dotnet-legacy&preserve-view=true) en Durable Functions v1.x).

Si utiliza lenguajes de scripting como JavaScript, Python o PowerShell, el desencadenador del cliente de larga duración se definirá mediante el objeto JSON siguiente en la matriz `bindings` de *function.json*:

```json
{
    "name": "<Name of input parameter in function signature>",
    "taskHub": "<Optional - name of the task hub>",
    "connectionName": "<Optional - name of the connection string app setting>",
    "type": "orchestrationClient",
    "direction": "in"
}
```

* `taskHub`: se usa en escenarios donde varias aplicaciones de función comparten la misma cuenta de almacenamiento, pero tienen que estar aisladas entre sí. Si no se especifica, se usa el valor predeterminado `host.json`. Este valor tiene que coincidir con el valor usado por las funciones de orquestador de destino.
* `connectionName`: nombre de una configuración de aplicación que contiene una cadena de conexión de cuenta de almacenamiento. La cuenta de almacenamiento representada por esta cadena de conexión tiene que ser la misma que utilizan las funciones del orquestador de destino. Si no se especifica, se utiliza la cadena de conexión de cuenta de almacenamiento predeterminada de la aplicación de función.

> [!NOTE]
> En la mayoría de los casos, se recomienda omitir estas propiedades y confiar en el comportamiento predeterminado.

### <a name="client-usage"></a>Uso del cliente

En las funciones de .NET, habitualmente se enlaza con [IDurableClient](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.idurableclient) ([DurableOrchestrationClient](/dotnet/api/microsoft.azure.webjobs.durableorchestrationclient?view=azure-dotnet-legacy&preserve-view=true) en Durable Functions v1.x) lo que proporciona acceso completo a todas las API de cliente de orquestación compatibles con Durable Functions. En otros lenguajes, debe usar el SDK específico del lenguaje para obtener acceso a un objeto de cliente.

Aquí tenemos un ejemplo de una función desencadenada por la cola que inicia una orquestación "HelloWorld".

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("QueueStart")]
public static Task Run(
    [QueueTrigger("durable-function-trigger")] string input,
    [DurableClient] IDurableOrchestrationClient starter)
{
    // Orchestration input comes from the queue message content.
    return starter.StartNewAsync("HelloWorld", input);
}
```

> [!NOTE]
> El código de C# anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar el atributo `OrchestrationClient` en lugar del atributo `DurableClient`, además de usar el tipo de parámetro `DurableOrchestrationClient` en lugar de `IDurableOrchestrationClient`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

**function.json**
```json
{
  "bindings": [
    {
      "name": "input",
      "type": "queueTrigger",
      "queueName": "durable-function-trigger",
      "direction": "in"
    },
    {
      "name": "starter",
      "type": "durableClient",
      "direction": "in"
    }
  ]
}
```

**index.js**
```javascript
const df = require("durable-functions");

module.exports = async function (context) {
    const client = df.getClient(context);
    return instanceId = await client.startNew("HelloWorld", undefined, context.bindings.input);
};
```

# <a name="python"></a>[Python](#tab/python)

**function.json**
```json
{
  "bindings": [
    {
      "name": "input",
      "type": "queueTrigger",
      "queueName": "durable-function-trigger",
      "direction": "in"
    },
    {
      "name": "starter",
      "type": "durableClient",
      "direction": "in"
    }
  ]
}
```

**__init__.py**
```python
import json
import azure.functions as func
import azure.durable_functions as df

async def main(msg: func.QueueMessage, starter: str) -> None:
    client = df.DurableOrchestrationClient(starter)
    payload = msg.get_body().decode('utf-8')
    instance_id = await client.start_new("HelloWorld", client_input=payload)
```

---

Pueden encontrar más información acerca de cómo iniciar instancias en [Administración de instancias](durable-functions-instance-management.md).

## <a name="entity-trigger"></a>Desencadenador de entidad

Los desencadenadores de entidad permiten crear [funciones de entidad](durable-functions-entities.md). Este desencadenador admite el procesamiento de eventos para una instancia de entidad específica.

> [!NOTE]
> Los desencadenadores de entidad están disponibles a partir de Durable Functions 2.x.

Internamente, este enlace de desencadenador sondea el almacén de larga duración configurado para las nuevas operaciones de entidad que deben ejecutarse.

### <a name="trigger-behavior"></a>Comportamiento de un desencadenador

Estas son algunas notas sobre el desencadenador de entidad:

* **De un único subproceso**: un único subproceso de distribuidor se utiliza para procesar las operaciones de una entidad determinada. Si se envían varios mensajes a una sola entidad de manera simultánea, las operaciones se procesarán una a una.
* **Control de mensajes dudosos**: no hay compatibilidad con mensajes dudosos en los desencadenadores de entidad.
* **Visibilidad de los mensajes**: los mensajes de desencadenador de entidad se quitan de la cola y se mantienen invisibles durante un tiempo configurable. La visibilidad de estos mensajes se renueva automáticamente siempre que la aplicación de la función se esté ejecutando y se mantenga correcta.
* **Valores devueltos**: las funciones de entidad no admiten valores devueltos. Hay API específicas que se pueden usar para guardar el estado o pasar valores devuelta a las orquestaciones.

Los cambios de estado realizados en una entidad durante su ejecución se conservarán automáticamente una vez completada la ejecución.

Para más información y ejemplos sobre cómo definir e interactuar con desencadenadores de entidad, consulte la documentación sobre [entidades duraderas](durable-functions-entities.md).

## <a name="entity-client"></a>Cliente de entidad

El enlace de cliente de entidad permite desencadenar de manera asincrónica las [funciones de entidad](#entity-trigger). Estas funciones se conocen a veces como [funciones de cliente](durable-functions-types-features-overview.md#client-functions).

Si va a usar funciones precompiladas de .NET, puede enlazar con el cliente de entidad mediante el atributo [DurableClientAttribute](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.durableclientattribute) de .NET.

> [!NOTE]
> `[DurableClientAttribute]` también se puede usar para enlazar con el [cliente de orquestación](#orchestration-client).

Si utiliza lenguajes de scripting (como el scripting de C#, JavaScript o Python) para desarrollar, el desencadenador de entidad se define mediante el objeto JSON siguiente de la matriz `bindings` de *function.json*:

```json
{
    "name": "<Name of input parameter in function signature>",
    "taskHub": "<Optional - name of the task hub>",
    "connectionName": "<Optional - name of the connection string app setting>",
    "type": "durableClient",
    "direction": "in"
}
```

* `taskHub`: se usa en escenarios donde varias aplicaciones de función comparten la misma cuenta de almacenamiento, pero tienen que estar aisladas entre sí. Si no se especifica, se usa el valor predeterminado `host.json`. Este valor tiene que coincidir con el valor usado por las funciones de entidad de destino.
* `connectionName`: nombre de una configuración de aplicación que contiene una cadena de conexión de cuenta de almacenamiento. La cuenta de almacenamiento representada por esta cadena de conexión tiene que ser la misma que utilizan las funciones de entidad de destino. Si no se especifica, se utiliza la cadena de conexión de cuenta de almacenamiento predeterminada de la aplicación de función.

> [!NOTE]
> En la mayoría de los casos, se recomienda omitir las propiedades opcionales y confiar en el comportamiento predeterminado.

Para más información y ejemplos sobre la interacción con entidades como cliente, consulte la documentación sobre [entidades duraderas](durable-functions-entities.md#access-entities).

<a name="host-json"></a>
## <a name="hostjson-settings"></a>configuración de host.json

[!INCLUDE [durabletask](../../../includes/functions-host-json-durabletask.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Referencia de la API HTTP integrada para la administración de instancias](durable-functions-http-api.md)
