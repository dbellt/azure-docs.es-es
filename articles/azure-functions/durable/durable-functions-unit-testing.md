---
title: Prueba unitaria de Azure Durable Functions
description: Obtenga información sobre cómo ejecutar una prueba unitaria de Durable Functions.
ms.topic: conceptual
ms.date: 11/03/2019
ms.openlocfilehash: 3d3b89337d4643b59d37ebe0aaeaf6f207bf053e
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108165450"
---
# <a name="durable-functions-unit-testing"></a>Prueba unitaria de Durable Functions

La prueba unitaria es parte importante de las prácticas modernas de desarrollo de software. Las pruebas unitarias comprueban el comportamiento de la lógica de negocios e impiden los cambios importantes desapercibidos en el futuro. Durable Functions puede aumentar fácilmente su complejidad, por lo que realizar pruebas unitarias ayudará a evitar los cambios importantes. En las secciones siguientes se explica cómo ejecutar una prueba unitaria de los tres tipos de función: funciones de cliente de orquestación, de orquestador y de actividad.

> [!NOTE]
> En este artículo se ofrecen instrucciones sobre las pruebas unitarias para aplicaciones de Durable Functions que tienen como destino Durable Functions 2.x. Para obtener más información sobre las diferencias entre versiones, vea el artículo [Versiones de Durable Functions](durable-functions-versions.md).

## <a name="prerequisites"></a>Requisitos previos

Los ejemplos que aparecen en este artículo requieren conocer los siguientes conceptos y marcos:

* Pruebas unitarias

* Funciones duraderas

* [xUnit](https://github.com/xunit/xunit): marco de pruebas

* [moq](https://github.com/moq/moq4): marco de simulación

## <a name="base-classes-for-mocking"></a>Clases base para simulación

La simulación se admite a través de la interfaz siguiente:

* [IDurableOrchestrationClient](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.idurableorchestrationclient), [IDurableEntityClient](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.idurableentityclient) e [IDurableClient](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.idurableclient)

* [IDurableOrchestrationContext](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.idurableorchestrationcontext)

* [IDurableActivityContext](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.idurableactivitycontext)

* [IDurableEntityContext](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.idurableentitycontext)

Estas interfaces se pueden usar con los distintos desencadenadores y enlaces que admite Durable Functions. Al ejecutar Azure Functions, el entorno de ejecución de Functions ejecutará el código de función con una implementación concreta de estas interfaces. En las pruebas unitarias, puede pasar una versión ficticia de estas interfaces para probar la lógica de negocios.

## <a name="unit-testing-trigger-functions"></a>Funciones de desencadenador de prueba unitaria

En esta sección, la prueba unitaria validará la lógica de la función de desencadenador HTTP siguiente para iniciar nuevas orquestaciones.

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/HttpStart.cs)]

La tarea de prueba unitaria será comprobar el valor del encabezado `Retry-After` proporcionado en la carga útil de la respuesta. Por lo tanto, la prueba unitaria simulará algunos de los métodos de `IDurableClient` para garantizar un comportamiento predecible.

En primer lugar, usamos un marco ficticio (en este caso, [moq](https://github.com/moq/moq4)) para simular `IDurableClient`:

```csharp
// Mock IDurableClient
var durableClientMock = new Mock<IDurableClient>();
```

> [!NOTE]
> Aunque puede simular interfaces si las implementa directamente como una clase, los marcos ficticios simplifican el proceso de varias maneras. Por ejemplo, si se agrega un nuevo método a la interfaz en versiones secundarias, moq no requerirá ningún cambio de código, a diferencia de las implementaciones concretas.

Luego, se simula el método `StartNewAsync` para devolver un identificador de instancia conocido.

```csharp
// Mock StartNewAsync method
durableClientMock.
    Setup(x => x.StartNewAsync(functionName, It.IsAny<object>())).
    ReturnsAsync(instanceId);
```

A continuación, se simula `CreateCheckStatusResponse` para devolver siempre una respuesta HTTP 200 vacía.

```csharp
// Mock CreateCheckStatusResponse method
durableClientMock
    // Notice that even though the HttpStart function does not call IDurableClient.CreateCheckStatusResponse() 
    // with the optional parameter returnInternalServerErrorOnFailure, moq requires the method to be set up
    // with each of the optional parameters provided. Simply use It.IsAny<> for each optional parameter
    .Setup(x => x.CreateCheckStatusResponse(It.IsAny<HttpRequestMessage>(), instanceId, returnInternalServerErrorOnFailure: It.IsAny<bool>())
    .Returns(new HttpResponseMessage
    {
        StatusCode = HttpStatusCode.OK,
        Content = new StringContent(string.Empty),
        Headers =
        {
            RetryAfter = new RetryConditionHeaderValue(TimeSpan.FromSeconds(10))
        }
    });
```

También se simula `ILogger`:

```csharp
// Mock ILogger
var loggerMock = new Mock<ILogger>();
```

Ahora, se llama al método `Run` desde la prueba unitaria:

```csharp
// Call Orchestration trigger function
var result = await HttpStart.Run(
    new HttpRequestMessage()
    {
        Content = new StringContent("{}", Encoding.UTF8, "application/json"),
        RequestUri = new Uri("http://localhost:7071/orchestrators/E1_HelloSequence"),
    },
    durableClientMock.Object,
    functionName,
    loggerMock.Object);
```

 El último paso es comparar la salida con el valor esperado:

```csharp
// Validate that output is not null
Assert.NotNull(result.Headers.RetryAfter);

// Validate output's Retry-After header value
Assert.Equal(TimeSpan.FromSeconds(10), result.Headers.RetryAfter.Delta);
```

Después de combinar todos los pasos, la prueba unitaria tendrá el código siguiente:

[!code-csharp[Main](~/samples-durable-functions/samples/VSSample.Tests/HttpStartTests.cs)]

## <a name="unit-testing-orchestrator-functions"></a>Funciones de Orchestrator de prueba unitaria

Las funciones de Orchestrator son incluso más interesantes para la prueba unitaria porque habitualmente tienen mucha más lógica de negocios.

En esta sección, las pruebas unitarias validarán la salida de la función `E1_HelloSequence` de Orchestrator:

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/HelloSequence.cs)]

El código de la prueba unitaria comenzará con la creación de una simulación:

```csharp
var durableOrchestrationContextMock = new Mock<IDurableOrchestrationContext>();
```

Luego, se simularán las llamadas al método de la actividad:

```csharp
durableOrchestrationContextMock.Setup(x => x.CallActivityAsync<string>("E1_SayHello", "Tokyo")).ReturnsAsync("Hello Tokyo!");
durableOrchestrationContextMock.Setup(x => x.CallActivityAsync<string>("E1_SayHello", "Seattle")).ReturnsAsync("Hello Seattle!");
durableOrchestrationContextMock.Setup(x => x.CallActivityAsync<string>("E1_SayHello", "London")).ReturnsAsync("Hello London!");
```

A continuación, la prueba unitaria llamará al método `HelloSequence.Run`:

```csharp
var result = await HelloSequence.Run(durableOrchestrationContextMock.Object);
```

Finalmente, se validará la salida:

```csharp
Assert.Equal(3, result.Count);
Assert.Equal("Hello Tokyo!", result[0]);
Assert.Equal("Hello Seattle!", result[1]);
Assert.Equal("Hello London!", result[2]);
```

Después de combinar todos los pasos, la prueba unitaria tendrá el código siguiente:

[!code-csharp[Main](~/samples-durable-functions/samples/VSSample.Tests/HelloSequenceOrchestratorTests.cs)]

## <a name="unit-testing-activity-functions"></a>Funciones de la actividad de prueba unitaria

Es posible realizar una prueba unitaria de las funciones de actividad del mismo modo que las funciones no duraderas.

En esta sección, la prueba unitaria validará el comportamiento de la actividad de la función `E1_SayHello`:

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/HelloSequence.cs)]

Y las pruebas unitarias comprobarán el formato de la salida. Las pruebas unitarias pueden utilizar los tipos de parámetro directamente o simular la clase `IDurableActivityContext`:

[!code-csharp[Main](~/samples-durable-functions/samples/VSSample.Tests/HelloSequenceActivityTests.cs)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre xUnit](https://xunit.net/docs/getting-started/netcore/cmdline)
>
> [Más información sobre moq](https://github.com/Moq/moq4/wiki/Quickstart)
