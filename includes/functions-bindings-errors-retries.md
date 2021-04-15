---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 10/01/2020
ms.author: glenga
ms.openlocfilehash: 2ccff72be66a88b9bf0a5e9eb9c29ade8397804b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96356200"
---
Los errores que se producen en una instancia de Azure Functions pueden provenir de cualquiera de los orígenes siguientes:

- Uso de [desencadenadores y enlaces](..\articles\azure-functions\functions-triggers-bindings.md) de Azure Functions integrados
- Llamadas a las API de los servicios de Azure subyacentes
- Llamadas a puntos de conexión REST
- Llamadas a bibliotecas de cliente, paquetes o API de terceros

Seguir las prácticas recomendadas de control de errores es importante para evitar la pérdida de datos o mensajes. Entre las prácticas recomendadas de control de errores se incluyen las acciones siguientes:

- [Habilitar Application Insights](../articles/azure-functions/functions-monitoring.md)
- [Uso del control de errores estructurado](#use-structured-error-handling)
- [Diseño para idempotencia](../articles/azure-functions/functions-idempotent.md)
- [Implementar directivas de reintentos](#retry-policies-preview) (si procede)

### <a name="use-structured-error-handling"></a>Uso del control de errores estructurado

La captura y el registro de los errores son fundamentales para supervisar el estado de la aplicación. El nivel superior de cualquier código de función debe incluir un bloque try/catch. En el bloque catch, puede capturar y registrar errores.

## <a name="retry-policies-preview"></a>Directivas de reintentos (versión preliminar)

Se puede definir una directiva de reintentos en cualquier función para cualquier tipo de desencadenador en la aplicación de funciones.  La directiva de reintentos vuelve a ejecutar una función hasta que se ejecuta correctamente o hasta que se produce el número máximo de reintentos.  Las directivas de reintentos se pueden definir para todas las funciones de una aplicación o para funciones individuales.  De forma predeterminada, una aplicación de funciones no volverá a intentar los mensajes (aparte de los [desencadenadores específicos que tienen una directiva de reintentos en el origen del desencadenador](#using-retry-support-on-top-of-trigger-resilience)).  Se evalúa una directiva de reintentos cada vez que una ejecución produce una excepción no detectada.  Como procedimiento recomendado, debería detectar todas las excepciones del código y volver a generar los errores que debieran dar lugar a un reintento.  Los puntos de comprobación de Event Hubs y Azure Cosmos DB no se escribirán hasta que se haya completado la directiva de reintentos para la ejecución, lo que significa que el progreso en esa partición se pausará hasta que se complete el lote actual.

### <a name="retry-policy-options"></a>Opciones de la directiva de reintentos

Las opciones siguientes están disponibles para definir una directiva de reintentos.

El **número máximo de reintentos** es el número máximo de veces que se reintenta una ejecución antes de que se produzca un error eventual. Un valor de `-1` significa que se reintentará indefinidamente. El número de reintentos actual se almacena en la memoria de la instancia. Es posible que una instancia tenga un error entre reintentos.  Cuando se produce un error en una instancia durante una directiva de reintentos, se pierde el número de reintentos.  Cuando hay errores de instancia, los desencadenadores como Event Hubs, Azure Cosmos DB y Queue Storage pueden reanudar el procesamiento y reintentar el lote en una nueva instancia, con el número de reintentos restablecido en cero.  Otros desencadenadores, como HTTP y el temporizador, no se reanudan en una nueva instancia.  Esto significa que el número máximo de reintentos es un esfuerzo óptimo y, en algunos casos excepcionales, una ejecución puede volver a intentarse más que el máximo de veces, o bien, para los desencadenadores como HTTP y el temporizador reintentarse menos que el número máximo de veces.

La **estrategia de reintento** controla cómo se comportan los reintentos.  A continuación se indican dos opciones de reintento admitidas:

| Opción | Descripción|
|---|---|
|**`fixedDelay`**| Puede transcurrir un período de tiempo especificado entre cada reintento.|
| **`exponentialBackoff`**| El primer reintento espera el retraso mínimo. En los reintentos posteriores, se agrega el tiempo de manera exponencial a la duración inicial de cada reintento, hasta que se alcanza el retraso máximo.  El retroceso exponencial agrega una pequeña aleatorización a los retrasos para escalonar los reintentos en escenarios de alto rendimiento.|

#### <a name="app-level-configuration"></a>Configuración del nivel de la aplicación

Se puede definir una directiva de reintentos para todas las funciones de una aplicación [mediante el archivo `host.json`](../articles/azure-functions/functions-host-json.md#retry). 

#### <a name="function-level-configuration"></a>Configuración del nivel de la función

Se puede definir una directiva de reintentos para una función específica.  La configuración específica de la función tiene prioridad sobre la configuración de nivel de la aplicación.

#### <a name="fixed-delay-retry"></a>Retraso fijo entre reintentos

# <a name="c"></a>[C#](#tab/csharp)

Los reintentos requieren el paquete NuGet [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs) >= 3.0.23

```csharp
[FunctionName("EventHubTrigger")]
[FixedDelayRetry(5, "00:00:10")]
public static async Task Run([EventHubTrigger("myHub", Connection = "EventHubConnection")] EventData[] events, ILogger log)
{
// ...
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

Esta es la directiva de reintentos del archivo *function.json*:

```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "fixedDelay",
        "maxRetryCount": 4,
        "delayInterval": "00:00:10"
    }
}
```
# <a name="javascript"></a>[JavaScript](#tab/javascript)

Esta es la directiva de reintentos del archivo *function.json*:


```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "fixedDelay",
        "maxRetryCount": 4,
        "delayInterval": "00:00:10"
    }
}
```

# <a name="python"></a>[Python](#tab/python)

Esta es la directiva de reintentos del archivo *function.json*:

```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "fixedDelay",
        "maxRetryCount": 4,
        "delayInterval": "00:00:10"
    }
}
```

# <a name="java"></a>[Java](#tab/java)

Esta es la directiva de reintentos del archivo *function.json*:


```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "fixedDelay",
        "maxRetryCount": 4,
        "delayInterval": "00:00:10"
    }
}
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Esta es la directiva de reintentos del archivo *function.json*:


```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "fixedDelay",
        "maxRetryCount": 4,
        "delayInterval": "00:00:10"
    }
}
```
---

#### <a name="exponential-backoff-retry"></a>Reintento de retroceso exponencial

# <a name="c"></a>[C#](#tab/csharp)

Los reintentos requieren el paquete NuGet [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs) >= 3.0.23

```csharp
[FunctionName("EventHubTrigger")]
[ExponentialBackoffRetry(5, "00:00:04", "00:15:00")]
public static async Task Run([EventHubTrigger("myHub", Connection = "EventHubConnection")] EventData[] events, ILogger log)
{
// ...
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

Esta es la directiva de reintentos del archivo *function.json*:

```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "exponentialBackoff",
        "maxRetryCount": 5,
        "minimumInterval": "00:00:10",
        "maximumInterval": "00:15:00"
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Esta es la directiva de reintentos del archivo *function.json*:

```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "exponentialBackoff",
        "maxRetryCount": 5,
        "minimumInterval": "00:00:10",
        "maximumInterval": "00:15:00"
    }
}
```

# <a name="python"></a>[Python](#tab/python)

Esta es la directiva de reintentos del archivo *function.json*:

```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "exponentialBackoff",
        "maxRetryCount": 5,
        "minimumInterval": "00:00:10",
        "maximumInterval": "00:15:00"
    }
}
```

# <a name="java"></a>[Java](#tab/java)

Esta es la directiva de reintentos del archivo *function.json*:

```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "exponentialBackoff",
        "maxRetryCount": 5,
        "minimumInterval": "00:00:10",
        "maximumInterval": "00:15:00"
    }
}
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Esta es la directiva de reintentos del archivo *function.json*:

```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "exponentialBackoff",
        "maxRetryCount": 5,
        "minimumInterval": "00:00:10",
        "maximumInterval": "00:15:00"
    }
}
```
---

|Propiedad de function.json  |Propiedad de atributo | Descripción |
|---------|---------|---------| 
|strategy|n/a|Necesario. Estrategia de reintentos que se usará. Los valores válidos son `fixedDelay` y `exponentialBackoff`.|
|maxRetryCount|n/a|Necesario. Número máximo de reintentos permitidos por ejecución de función. `-1` significa que se reintentará indefinidamente.|
|delayInterval|n/a|Retraso que se utilizará entre reintentos al usar la estrategia `fixedDelay`.|
|minimumInterval|n/a|Retraso entre reintentos mínimo al usar la estrategia `exponentialBackoff`.|
|maximumInterval|n/a|Retraso entre reintentos máximo al usar la estrategia `exponentialBackoff`.| 

### <a name="retry-limitations-during-preview"></a>Limitaciones de reintentos durante la versión preliminar

- En los proyectos de .NET, es posible que tenga que incorporar una versión de [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs) >= 3.0.23.
- En el plan de consumo, la aplicación se puede reducir verticalmente hasta cero mientras se reintentan los mensajes finales de una cola.
- En el plan de consumo, la aplicación se puede reducir verticalmente mientras se realizan los reintentos.  Para obtener unos resultados óptimos, elija un intervalo de reintentos <= 00:01:00 y <= 5 reintentos.

## <a name="using-retry-support-on-top-of-trigger-resilience"></a>Uso de la compatibilidad con los reintentos sobre la resistencia del desencadenador

La directiva de reintentos de la aplicación de funciones es independiente de los reintentos o la resistencia que proporciona el desencadenador.  La directiva de reintentos de función solo se coloca sobre un reintento resistente del desencadenador.  Por ejemplo, si usa Azure Service Bus, de manera predeterminada, las colas tienen un número de entregas de mensajes de 10.  El número de entregas predeterminado significa que después de 10 intentos de entrega de un mensaje en cola, Service Bus coloca el mensaje en la cola de mensajes fallidos.  Puede definir una directiva de reintentos para una función que tenga un desencadenador de Service Bus, pero los reintentos se van a colocar sobre los intentos de entrega de Service Bus.  

Por ejemplo, si ha usado el número de entregas predeterminado de Service Bus de 10 y ha definido una directiva de reintentos de función de 5.  Primero, el mensaje se quita de la cola, lo que incrementa la cuenta de entrega de Service Bus en 1.  Si se ha producido un error en cada ejecución después de cinco intentos de desencadenar el mismo mensaje, ese mensaje se marca como abandonado.  Service Bus vuelve a poner en cola el mensaje inmediatamente, desencadena la función e incrementa el número de entregas en 2.  Por último, después de 50 intentos posibles (10 entregas de Service Bus * 5 reintentos de función por entrega), el mensaje se abandona y se desencadena la colocación en la cola de mensajes fallidos en Service Bus.

> [!WARNING]
> No se recomienda establecer el número de entregas de un desencadenador como las colas de Service Bus en 1, lo que significa que el mensaje se colocaría inmediatamente en la cola de mensajes fallidos después de un único ciclo de reintentos de función.  Esto se debe a que los desencadenadores proporcionan resistencia con los reintentos, mientras que la directiva de reintentos de función es el mejor esfuerzo y puede dar lugar a un número total de reintentos menor que el deseado.

### <a name="triggers-with-additional-resiliency-or-retries"></a>Desencadenadores con resistencia adicional o reintentos

Los siguientes desencadenadores admiten reintentos en el origen del desencadenador:

* [Almacenamiento de blobs de Azure](../articles/azure-functions/functions-bindings-storage-blob.md)
* [Azure Queue Storage](../articles/azure-functions/functions-bindings-storage-queue.md)
* [Azure Service Bus (cola/tema)](../articles/azure-functions/functions-bindings-service-bus.md)

De manera predeterminada, la mayoría de los desencadenadores reintenta las solicitudes hasta cinco veces. Después del quinto reintento, Azure Queue Storage escribe un mensaje en una [cola de mensajes dudosos](../articles/azure-functions/functions-bindings-storage-queue-trigger.md#poison-messages).  La directiva predeterminada de colas y temas de Service Bus escribe un mensaje en una [cola de mensajes fallidos](../articles/service-bus-messaging/service-bus-dead-letter-queues.md) después de 10 intentos.
