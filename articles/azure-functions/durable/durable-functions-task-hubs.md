---
title: 'Centrales de tareas en Durable Functions: Azure'
description: Aprenda qué son las centrales de tareas en la extensión Durable Functions para Azure Functions. Aprenda a configurar centrales de tareas.
author: cgillum
ms.topic: conceptual
ms.date: 05/12/2021
ms.author: azfuncdf
ms.openlocfilehash: 9172075ca22937a85fd7fd5827ebb40a4b58bcfa
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110375764"
---
# <a name="task-hubs-in-durable-functions-azure-functions"></a>Centrales de tareas en Durable Functions (Azure Functions)

Una *central de tareas* en [Durable Functions](durable-functions-overview.md) es un contenedor lógico de recursos de almacenamiento duraderos que se usan en las orquestaciones y las entidades. Las funciones de orquestador, actividad y entidad solo pueden interactuar entre sí si pertenecen a la misma central de tareas.

> [!NOTE]
> En este documento se describen los detalles de las centrales de tareas de forma específica para el [proveedor de Azure Storage para Durable Functions](durable-functions-storage-providers.md#azure-storage) predeterminado. Si usa un proveedor de almacenamiento no predeterminado para la aplicación Durable Functions, puede encontrar documentación detallada de la central de tareas en la documentación específica del proveedor:
> 
> * [Información de la central de tareas para el proveedor de almacenamiento Netherite](https://microsoft.github.io/durabletask-netherite/#/storage)
> * [Información de la central de tareas para el proveedor de almacenamiento Microsoft SQL (MSSQL)](https://microsoft.github.io/durabletask-mssql/#/taskhubs)
> 
> Para más información sobre las diversas opciones del proveedor de almacenamiento y cómo se comparan, consulte la documentación sobre [proveedores de almacenamiento de Durable Functions](durable-functions-storage-providers.md).

Si varias aplicaciones de función comparten una cuenta de almacenamiento, se *debe* configurar cada una de ellas con un nombre de central de tareas independiente. Una cuenta de almacenamiento puede contener varias centrales de tareas. Esta restricción se suele aplicar también a otros proveedores de almacenamiento.

El siguiente diagrama muestra una central de tareas por cada aplicación de función en las cuentas de Azure Storage compartidas y dedicadas.

![Diagrama de almacenamiento que muestra las cuentas de almacenamiento compartidas y dedicadas.](./media/durable-functions-task-hubs/task-hubs-storage.png)

## <a name="azure-storage-resources"></a>Recursos de Azure Storage

Una central de tareas en Azure Storage consta de los siguientes recursos:

* Una o más colas de control.
* Una cola de elementos de trabajo.
* Una tabla de historial.
* Una tabla de instancias.
* Un contenedor de almacenamiento que contiene uno o varios blobs de concesión.
* Un contenedor de almacenamiento que contiene cargas de mensajes grandes, si procede.

Todos estos recursos se crean automáticamente en la cuenta de Azure Storage configurada cuando se ejecutan funciones de orquestador, entidad o actividad o cuando se programa su ejecución. El artículo [Rendimiento y escala](durable-functions-perf-and-scale.md) explica cómo se utilizan estos recursos.

## <a name="task-hub-names"></a>Nombres de las centrales de tareas

Las centrales de tareas de Azure Storage se identifican mediante un nombre que se ajusta a estas reglas:

* Solo contiene caracteres alfanuméricos.
* Comienza con una letra.
* Tiene una longitud mínima de 3 caracteres, con una longitud máxima de 45 caracteres.

El nombre de la central de tareas se declara en el archivo *host.json*, como se muestra en el ejemplo siguiente:

### <a name="hostjson-functions-20"></a>host.json (Functions 2.0)

```json
{
  "version": "2.0",
  "extensions": {
    "durableTask": {
      "hubName": "MyTaskHub"
    }
  }
}
```

### <a name="hostjson-functions-1x"></a>host.json (Functions 1.x)

```json
{
  "durableTask": {
    "hubName": "MyTaskHub"
  }
}
```

Las centrales de tareas también se pueden configurar mediante la configuración de la aplicación, tal como se muestra en el siguiente archivo de ejemplo `host.json`:

### <a name="hostjson-functions-10"></a>host.json (Functions 1.0)

```json
{
  "durableTask": {
    "hubName": "%MyTaskHub%"
  }
}
```

### <a name="hostjson-functions-20"></a>host.json (Functions 2.0)

```json
{
  "version": "2.0",
  "extensions": {
    "durableTask": {
      "hubName": "%MyTaskHub%"
    }
  }
}
```

El nombre de la central de tareas se establecerá en el valor de la configuración `MyTaskHub` de la aplicación. El siguiente valor de `local.settings.json` muestra cómo definir la configuración `MyTaskHub` como `samplehubname`:

```json
{
  "IsEncrypted": false,
  "Values": {
    "MyTaskHub" : "samplehubname"
  }
}
```

Además de **host.json**, los nombres de central de tareas también se pueden configurar en los metadatos de [enlace de cliente de orquestación](durable-functions-bindings.md#orchestration-client). Esta opción resulta útil si necesita acceder a orquestaciones o entidades que se encuentran en una aplicación de función aparte. En el siguiente código se muestra cómo escribir una función que usa el [enlace del cliente de orquestación](durable-functions-bindings.md#orchestration-client) para trabajar con una central de tareas definida como configuración de la aplicación:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("HttpStart")]
public static async Task<HttpResponseMessage> Run(
    [HttpTrigger(AuthorizationLevel.Function, methods: "post", Route = "orchestrators/{functionName}")] HttpRequestMessage req,
    [DurableClient(TaskHub = "%MyTaskHub%")] IDurableOrchestrationClient starter,
    string functionName,
    ILogger log)
{
    // Function input comes from the request content.
    object eventData = await req.Content.ReadAsAsync<object>();
    string instanceId = await starter.StartNewAsync(functionName, eventData);

    log.LogInformation($"Started orchestration with ID = '{instanceId}'.");

    return starter.CreateCheckStatusResponse(req, instanceId);
}
```

> [!NOTE]
> El ejemplo de C# anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar `DurableOrchestrationContext` en lugar de `IDurableOrchestrationContext`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

La propiedad de la central de tareas del archivo `function.json` se establece mediante la configuración de aplicación:

```json
{
    "name": "input",
    "taskHub": "%MyTaskHub%",
    "type": "orchestrationClient",
    "direction": "in"
}
```

# <a name="python"></a>[Python](#tab/python)

La propiedad de la central de tareas del archivo `function.json` se establece mediante la configuración de aplicación:

```json
{
    "name": "input",
    "taskHub": "%MyTaskHub%",
    "type": "orchestrationClient",
    "direction": "in"
}
```

---

> [!NOTE]
> La configuración de nombres de central de tareas en los metadatos de enlace de cliente solo es necesaria cuando se usa una aplicación de funciones para acceder a orquestaciones y entidades de otra aplicación de funciones. Si las funciones de cliente se definen en la misma aplicación de funciones que las orquestaciones y las entidades, debe evitar especificar nombres de central de tareas en los metadatos de enlace. De forma predeterminada, todos los enlaces de cliente obtienen sus metadatos de central de tareas de la configuración de **host.jsconfiguración**.

Los nombres de central de tareas de Azure Storage deben empezar por una letra y estar formados únicamente por letras y números. Si no se especifica, se usará un nombre de central de tareas predeterminado, tal y como se muestra en la tabla siguiente:

| Versión de la extensión de Durable | Nombre de central de tareas predeterminado |
| - | - |
| 2.x | Cuando se implementa en Azure, el nombre de la central de tareas se deriva del nombre de la _aplicación de funciones_. Cuando se ejecuta fuera de Azure, el nombre predeterminado de la central de tareas es `TestHubName`. |
| 1.x | El nombre predeterminado de la central de tareas para todos los entornos es `DurableFunctionsHub`. |

Para obtener más información sobre las diferencias entre versiones de extensión, consulte el artículo [Versiones de Durable Functions](durable-functions-versions.md).

> [!NOTE]
> El nombre es lo que diferencia una central de tareas de otra cuando hay varias de ellas en una cuenta de almacenamiento compartido. Si tiene varias aplicaciones de función que comparten una cuenta de almacenamiento, deberá configurar explícitamente nombres diferentes para cada central de tareas en el archivo *host.json*. En caso contrario, las diversas aplicaciones de funciones competirán entre sí por los mensajes, lo cual podría provocar un comportamiento indefinido, como, por ejemplo, orquestaciones que se "atascan" inesperadamente en el estado `Pending` o `Running`.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre cómo administrar el control de versiones de orquestación](durable-functions-versioning.md)
