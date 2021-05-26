---
title: 'Temporizadores en Durable Functions: Azure'
description: Aprenda a implementar temporizadores durables en la extensión Durable Functions para Azure Functions.
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: azfuncdf
ms.openlocfilehash: d96afbad061071bfc80a69764b577032fdcb95c0
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110375747"
---
# <a name="timers-in-durable-functions-azure-functions"></a>Temporizadores en Durable Functions (Azure Functions)

[Durable Functions](durable-functions-overview.md) proporciona *temporizadores durables* para usarlos en funciones de orquestador para implementar retrasos o configurar tiempos de expiración en acciones asincrónicas. Los temporizadores durables deben usarse en funciones de orquestador en lugar de `Thread.Sleep` y `Task.Delay` (C#), `setTimeout()` o `setInterval()` (JavaScript) y `time.sleep()` (Python).

Puede crear un temporizador durable si llama al método `CreateTimer` (.NET) o al método `createTimer` (JavaScript) del [enlace del desencadenador de orquestación](durable-functions-bindings.md#orchestration-trigger). El método devuelve una tarea que se completa en una fecha y hora especificadas.

## <a name="timer-limitations"></a>Limitaciones de los temporizadores

Cuando se crea un temporizador que expira a las 16:30, la instancia de Durable Task Framework subyacente pone en cola un mensaje que se vuelve visible solo a las 16:30. Al ejecutarse en el plan Consumo de Azure Functions, el mensaje del temporizador ahora visible garantiza que la aplicación de función se active en una máquina virtual adecuada.

> [!NOTE]
> * A partir de la [versión 2.3.0](https://github.com/Azure/azure-functions-durable-extension/releases/tag/v2.3.0) de la extensión duradera, los temporizadores duraderos son ilimitados. En versiones anteriores de la extensión, los temporizadores duraderos están limitados a siete días. Si usa una versión anterior y necesita retrasar el periodo más de siete días, use las API de temporizador en un bucle `while` para simular un retraso.
> * Use siempre `CurrentUtcDateTime` en lugar de `DateTime.UtcNow` en .NET, o bien `currentUtcDateTime` en lugar de `Date.now` o `Date.UTC` en JavaScript al calcular el tiempo de activación de los temporizadores durables. Para más información, consulte el artículo sobre las [restricciones de código de las funciones de orquestador](durable-functions-code-constraints.md).

## <a name="usage-for-delay"></a>Uso para retraso

En el ejemplo siguiente se muestra cómo utilizar temporizadores durables para retrasar la ejecución. En el ejemplo se emite una notificación de facturación cada día durante 10 días.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("BillingIssuer")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    for (int i = 0; i < 10; i++)
    {
        DateTime deadline = context.CurrentUtcDateTime.Add(TimeSpan.FromDays(1));
        await context.CreateTimer(deadline, CancellationToken.None);
        await context.CallActivityAsync("SendBillingEvent");
    }
}
```

> [!NOTE]
> El ejemplo de C# anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar `DurableOrchestrationContext` en lugar de `IDurableOrchestrationContext`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```js
const df = require("durable-functions");
const moment = require("moment");

module.exports = df.orchestrator(function*(context) {
    for (let i = 0; i < 10; i++) {
        const deadline = moment.utc(context.df.currentUtcDateTime).add(1, 'd');
        yield context.df.createTimer(deadline.toDate());
        yield context.df.callActivity("SendBillingEvent");
    }
});
```
# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df
from datetime import datetime, timedelta

def orchestrator_function(context: df.DurableOrchestrationContext):
    for i in range(0, 9):
        deadline = context.current_utc_datetime + timedelta(days=1)
        yield context.create_timer(deadline)
        yield context.call_activity("SendBillingEvent")

main = df.Orchestrator.create(orchestrator_function)
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
param($Context)

for ($num = 0 ; $num -le 9 ; $num++){    
    $expiryTime =  New-TimeSpan -Days 1
    $timerTask = Start-DurableTimer -Duration $expiryTime
    Invoke-DurableActivity -FunctionName 'SendBillingEvent'
}
```
---

> [!WARNING]
> Evite bucles infinitos en funciones de orquestador. Para más información acerca de cómo implementar de forma segura y eficaz escenarios de bucle infinito, consulte [Orquestaciones infinitas](durable-functions-eternal-orchestrations.md).

## <a name="usage-for-timeout"></a>Uso para tiempo de expiración

En este ejemplo se muestra cómo utilizar temporizadores durables para implementar tiempos de expiración.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("TryGetQuote")]
public static async Task<bool> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    TimeSpan timeout = TimeSpan.FromSeconds(30);
    DateTime deadline = context.CurrentUtcDateTime.Add(timeout);

    using (var cts = new CancellationTokenSource())
    {
        Task activityTask = context.CallActivityAsync("GetQuote");
        Task timeoutTask = context.CreateTimer(deadline, cts.Token);

        Task winner = await Task.WhenAny(activityTask, timeoutTask);
        if (winner == activityTask)
        {
            // success case
            cts.Cancel();
            return true;
        }
        else
        {
            // timeout case
            return false;
        }
    }
}
```

> [!NOTE]
> El ejemplo de C# anterior corresponde a Durable Functions 2.x. En el caso de Durable Functions 1.x, debe usar `DurableOrchestrationContext` en lugar de `IDurableOrchestrationContext`. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```js
const df = require("durable-functions");
const moment = require("moment");

module.exports = df.orchestrator(function*(context) {
    const deadline = moment.utc(context.df.currentUtcDateTime).add(30, "s");

    const activityTask = context.df.callActivity("GetQuote");
    const timeoutTask = context.df.createTimer(deadline.toDate());

    const winner = yield context.df.Task.any([activityTask, timeoutTask]);
    if (winner === activityTask) {
        // success case
        timeoutTask.cancel();
        return true;
    }
    else
    {
        // timeout case
        return false;
    }
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df
from datetime import datetime, timedelta

def orchestrator_function(context: df.DurableOrchestrationContext):
    deadline = context.current_utc_datetime + timedelta(seconds=30)
    activity_task = context.call_activity("GetQuote")
    timeout_task = context.create_timer(deadline)

    winner = yield context.task_any([activity_task, timeout_task])
    if winner == activity_task:
        timeout_task.cancel()
        return True
    elif winner == timeout_task:
        return False

main = df.Orchestrator.create(orchestrator_function)
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)
```powershell
param($Context)

$expiryTime =  New-TimeSpan -Seconds 30

$activityTask = Invoke-DurableActivity -FunctionName 'GetQuote'-NoWait
$timerTask = Start-DurableTimer -Duration $expiryTime -NoWait

$winner = Wait-DurableTask -Task @($activityTask, $timerTask) -Any

if ($winner -eq $activityTask) {
    Stop-DurableTaskTimer -Task $timerTask
    return $True
}
else {
    return $False
}
```

---

> [!WARNING]
> Use `CancellationTokenSource` (.NET) o llame a `cancel()` en el elemento `TimerTask` devuelto (JavaScript) para cancelar un temporizador durable si el código no va a esperar a que se complete. Durable Task Framework no cambiará el estado de una orquestación a "Completed" (Completado) hasta que todas las tareas pendientes se hayan completado o cancelado.

Este mecanismo de cancelación no finaliza las ejecuciones de suborquestación o función de actividad en curso. En su lugar, simplemente permite que la función de orquestador pase por alto el resultado y continúe. Si la aplicación de función usa el plan Consumo, se le sigue facturando por el tiempo y la memoria consumidos por la función de actividad abandonada. De manera predeterminada, las funciones en ejecución en el plan de consumo tienen un tiempo de expiración de cinco minutos. Si se supera este límite, el host de Azure Functions se recicla para detener toda la ejecución y evitar una situación de facturación descontrolada. El [tiempo de expiración de la función se puede configurar](../functions-host-json.md#functiontimeout).

Para obtener un ejemplo más detallado de cómo implementar tiempos de espera en funciones de orquestador, vea el artículo [Las interacciones humanas en Durable Functions: comprobación telefónica de ejemplo](durable-functions-phone-verification.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información acerca de cómo generar y controlar eventos externos](durable-functions-external-events.md)
