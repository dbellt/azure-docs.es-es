---
title: 'Control de versiones en Durable Functions: Azure'
description: Aprenda a implementar el control de versiones en la extensión Durable Functions para Azure Functions.
author: cgillum
ms.topic: conceptual
ms.date: 05/12/2021
ms.author: azfuncdf
ms.openlocfilehash: c5d3ac097efd81b369db16d6f7b6a4bf59b7540a
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110377405"
---
# <a name="versioning-in-durable-functions-azure-functions"></a>Control de versiones en Durable Functions (Azure Functions)

Resulta inevitable agregar, eliminar y cambiar funciones durante el ciclo de vida de una aplicación. [Durable Functions](durable-functions-overview.md) permite el encadenamiento de funciones entre sí de maneras que no eran posibles anteriormente. Este encadenamiento afecta a la manera de controlar las versiones.

## <a name="how-to-handle-breaking-changes"></a>Cómo controlar los cambios importantes

Hay varios ejemplos de cambios importantes que se deben tener en cuenta. En este artículo se abordan los más habituales. Todos ellos tienen en común que tanto las orquestaciones de función nuevas como las ya existentes se ven afectadas por los cambios realizados en el código de función.

### <a name="changing-activity-or-entity-function-signatures"></a>Cambiar las firmas de función de actividad o entidad

Un cambio de firma hace referencia a un cambio en el nombre, la entrada o la salida de una función. Si este tipo de cambio se realiza en una función de actividad o entidad, podría interrumpir la función de orquestador que dependa de él. Si actualiza la función de orquestador para dar cabida a este cambio, se podrían interrumpir las instancias en curso existentes.

Por ejemplo, supongamos que tenemos la siguiente función de orquestador de C#.

```csharp
[FunctionName("FooBar")]
public static Task Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    bool result = await context.CallActivityAsync<bool>("Foo");
    await context.CallActivityAsync("Bar", result);
}
```

Esta sencilla función toma los resultados de **Foo** y los pasa a **Bar**. Supongamos que es necesario cambiar el valor devuelto de **Foo** de `bool` a `int` para admitir una mayor variedad de valores de resultado. El resultado tiene el aspecto siguiente:

```csharp
[FunctionName("FooBar")]
public static Task Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string result = await context.CallActivityAsync<string>("Foo");
    await context.CallActivityAsync("Bar", result);
}
```

Este cambio funciona bien en todas las nuevas instancias de la función de orquestador, pero interrumpe todas las instancias en curso. Por ejemplo, considere el caso en el que una instancia de orquestación llama a una función denominada `Foo`, obtiene un valor booleano y, a continuación, establece puntos de control. Si el cambio de firma se implementa en este momento, se producirá un error en la instancia con los puntos de control inmediatamente después de que se reanude y reproduzca la llamada a `Foo`. Este error se debe a que el resultado de la tabla de historial es `bool`, pero el nuevo código intenta deserializarlo como `string`, lo que provoca una excepción en tiempo de ejecución para los lenguajes con seguridad de tipos.

Este ejemplo es solo una de las muchas maneras en las que un cambio de firma de la función puede interrumpir las instancias existentes. En general, si un orquestador necesita cambiar la manera en que llama a una función, probablemente el cambio sea problemático.

### <a name="changing-orchestrator-logic"></a>Cambiar la lógica del orquestador

El otro tipo de problemas de versiones se produce al cambiar el código de la función del orquestador de forma que cambie la ruta de acceso de ejecución de las instancias en curso.

Considere la función de orquestador de C# siguiente:

```csharp
[FunctionName("FooBar")]
public static Task Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    bool result = await context.CallActivityAsync<bool>("Foo");
    await context.CallActivityAsync("Bar", result);
}
```

Ahora supongamos que quiere realizar un cambio para agregar una nueva llamada de función entre las dos llamadas de función existentes.

```csharp
[FunctionName("FooBar")]
public static Task Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    bool result = await context.CallActivityAsync<bool>("Foo");
    if (result)
    {
        await context.CallActivityAsync("SendNotification");
    }

    await context.CallActivityAsync("Bar", result);
}
```

Este cambio agrega una nueva llamada de función a *SendNotification* entre *Foo* y *Bar*. No hay ningún cambio de firma. El problema surge cuando se reanuda una instancia existente desde la llamada a *Bar*. Durante la reproducción, si la llamada original a *Foo* devolvió `true`, la reproducción del orquestador llamará a *SendNotification*, que no está en su historial de ejecución. Durable Task Framework detecta esta incoherencia y genera una excepción `NonDeterministicOrchestrationException` porque ha encontrado una llamada a *SendNotification* cuando esperaba una llamada a *Bar*. Se puede producir el mismo tipo de problema al agregar llamadas de API a otras operaciones duraderas, como la creación de temporizadores duraderos, la espera de eventos externos, la llamada a orquestaciones secundarias, etc.

## <a name="mitigation-strategies"></a>Estrategias de mitigación

Estas son algunas de las estrategias para tratar los desafíos que plantea el control de versiones:

* No hacer nada (no se recomienda)
* Detener todas las instancias en curso
* Implementaciones en paralelo

### <a name="do-nothing"></a>No hacer nada

El enfoque más simple del control de versiones es no hacer nada y dejar que las instancias de orquestación en curso no se realicen correctamente. En función del tipo de cambio, se pueden producir los siguientes tipos de errores.

* Las orquestaciones pueden producir un error `NonDeterministicOrchestrationException`.
* Las orquestaciones pueden quedarse bloqueadas indefinidamente, informando de un estado `Running`.
* Si se quita una función, cualquier función que intente llamarla puede producir un error.
* Si se quita una función después de que se programe para ejecutarse, la aplicación puede experimentar errores de bajo nivel en tiempo de ejecución en el motor de Durable Task Framework, lo que podría provocar una degradación grave del rendimiento.

Debido a estos posibles errores, no se recomienda la estrategia "no hacer nada".

### <a name="stop-all-in-flight-instances"></a>Detener todas las instancias en curso

Otra opción es detener todas las instancias en curso. Si usa el [proveedor predeterminado de Azure Storage para Durable Functions](durable-functions-storage-providers.md#azure-storage), la detención de todas las instancias se puede realizar borrando el contenido de las colas internas **control-queue** y **workitem-queue**. También puede detener la aplicación de funciones, eliminar estas colas y reiniciar la aplicación de nuevo. Las colas se volverán a crear automáticamente una vez que se reinicie la aplicación. Las instancias de orquestación anteriores pueden permanecer en el estado "En ejecución" indefinidamente, pero no abarrotarán los registros con mensajes de error ni provocarán daños en la aplicación. Este enfoque es el idóneo para un desarrollo rápido de prototipos, incluido el desarrollo local.

> [!NOTE]
> Este enfoque requiere acceso directo a los recursos de almacenamiento subyacentes y puede no ser adecuado para todos los proveedores de almacenamiento admitidos por Durable Functions.

### <a name="side-by-side-deployments"></a>Implementaciones en paralelo

El método menos propenso a errores para garantizar que los cambios importantes se implementen con seguridad consiste en implementarlos en paralelo con sus versiones anteriores. Esto puede realizarse mediante cualquiera de las técnicas siguientes:

* Implemente todas las implementaciones como funciones completamente nuevas, dejando las funciones existentes tal cual. Por lo general, esto no se recomienda debido a la complejidad implicada en la actualización recursiva de los autores de llamadas de las nuevas versiones de la función.
* Implemente todas las actualizaciones como una nueva aplicación de función con una cuenta de almacenamiento diferente.
* Implemente una nueva copia de la aplicación de funciones con la misma cuenta de almacenamiento, pero con un nombre de [central de tareas](durable-functions-task-hubs.md) actualizado. Esto da como resultado la creación de nuevos artefactos de almacenamiento que se pueden usar en la nueva versión de la aplicación. La versión anterior de la aplicación seguirá ejecutándose con el conjunto anterior de artefactos de almacenamiento.

La implementación en paralelo es la técnica recomendada para implementar nuevas versiones de las aplicaciones de funciones.

> [!NOTE]
> Esta guía para la estrategia de implementación en paralelo usa términos específicos de Azure Storage, pero se aplica generalmente a todos los [proveedores de almacenamiento de Durable Functions](durable-functions-storage-providers.md) admitidos.

### <a name="deployment-slots"></a>Ranuras de implementación

Al realizar implementaciones en paralelo en Azure Functions o Azure App Service, se recomienda implementar la nueva versión de la aplicación de funciones en una nueva [ranura de implementación](../functions-deployment-slots.md). Las ranuras de implementación permiten ejecutar varias copias de la aplicación de función en paralelo con solo una de ellas como ranura de *producción* activa. Cuando desee exponer la nueva lógica de orquestación en su infraestructura existente, será tan sencillo como intercambiar la nueva versión en la ranura de producción.

> [!NOTE]
> Esta estrategia funciona mejor cuando se usan desencadenadores HTTP y webhook para las funciones de orquestador. Para desencadenadores que no son HTTP, como las colas o Event Hubs, la definición del desencadenador debe [derivar de una configuración de aplicación](../functions-bindings-expressions-patterns.md#binding-expressions---app-settings) que se actualiza como parte de la operación de intercambio.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información acerca de cómo controlar el rendimiento y escalar problemas](durable-functions-perf-and-scale.md)
