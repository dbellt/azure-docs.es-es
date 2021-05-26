---
title: 'API de HTTP en Durable Functions: Azure Functions'
description: Aprenda a implementar API de HTTP en la extensión Durable Functions para Azure Functions.
author: cgillum
ms.topic: conceptual
ms.date: 05/11/2021
ms.author: azfuncdf
ms.openlocfilehash: eff6a44734600a6399f76fc7be331835ae395593
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110377456"
---
# <a name="http-api-reference"></a>Referencia a las API de HTTP

La extensión Durable Functions expone un conjunto de API de HTTP integradas que se pueden usar para realizar tareas de administración en [orquestaciones](durable-functions-types-features-overview.md#orchestrator-functions), [entidades](durable-functions-types-features-overview.md#entity-functions) y [centrales de tareas](durable-functions-task-hubs.md). Estas API de HTTP son webhooks de extensibilidad que están autorizados por el host de Azure Functions pero que se administran directamente mediante la extensión Durable Functions.

Todas las API de HTTP que implementa la extensión requieren los siguientes parámetros. El tipo de datos de todos los parámetros es `string`.

| Parámetro        | Tipo de parámetro  | Descripción |
|------------------|-----------------|-------------|
| **`taskHub`**    | Cadena de consulta    | Nombre de la [central de tareas](durable-functions-task-hubs.md). Si no se especifica, se toma el nombre de la central de tareas de la aplicación de función actual. |
| **`connection`** | Cadena de consulta    | **Nombre** de la configuración de aplicación de conexión para el proveedor de almacenamiento de back-end. Si no se especifica, se toma el de la configuración de conexión predeterminada de la aplicación de funciones. |
| **`systemKey`**  | Cadena de consulta    | Clave de autorización necesaria para invocar la API. |

`systemKey` es una clave de autorización que el host de Azure Functions genera automáticamente. En concreto, concede acceso a las API de la extensión Durable Tasks y se administra igual que las [demás claves de acceso de Azure Functions](../security-concepts.md#function-access-keys). Puede generar direcciones URL que contengan los valores de cadena de consulta `taskHub`, `connection` y `systemKey` correctos mediante API de [enlace de clientes de orquestación](durable-functions-bindings.md#orchestration-client), como las API `CreateCheckStatusResponse` y `CreateHttpManagementPayload` en .NET o las API `createCheckStatusResponse` y `createHttpManagementPayload` en JavaScript, etc.

Las siguientes secciones tratan las API de HTTP específicas que admite la extensión y ofrecen ejemplos de uso.

## <a name="start-orchestration"></a>Inicio de la orquestación

Inicia la ejecución de una nueva instancia de la función de orquestador especificada.

### <a name="request"></a>Solicitud

Para la versión 1.x del entorno de ejecución de Functions, la solicitud tiene el formato siguiente (se muestran varias líneas para mayor claridad):

```http
POST /admin/extensions/DurableTaskExtension/orchestrators/{functionName}/{instanceId?}
     ?taskHub={taskHub}
     &connection={connectionName}
     &code={systemKey}
```

En la versión 2.x del entorno de ejecución de Functions, el formato de dirección URL tiene los mismos parámetros, pero un prefijo ligeramente diferente:

```http
POST /runtime/webhooks/durabletask/orchestrators/{functionName}/{instanceId?}
     ?taskHub={taskHub}
     &connection={connectionName}
     &code={systemKey}
```

Los parámetros de solicitud de esta API incluyen el conjunto predeterminado mencionado anteriormente, así como los siguientes parámetros únicos:

| Campo              | Tipo de parámetro  | Descripción |
|--------------------|-----------------|-------------|
| **`functionName`** | URL             | El nombre de la función de orquestador que iniciar. |
| **`instanceId`**   | URL             | Parámetro opcional. Identificador de la instancia de orquestación. Si no se especifica, la función de orquestador se iniciará con un identificador de instancia aleatorio. |
| **`{content}`**    | Contenido de la solicitud | Opcional. La entrada de la función de orquestador en formato JSON. |

### <a name="response"></a>Response

Se pueden devolver varios valores de código de estado.

* **HTTP 202 (aceptado)** : la función de orquestador especificada estaba programada para empezar a ejecutarse. El encabezado de respuesta `Location` contiene una dirección URL para sondear el estado de la orquestación.
* **HTTP 400 (solicitud incorrecta)** : la función de orquestador especificada no existe, el identificador de instancia especificado no era válido o el contenido de la solicitud no era JSON válido.

A continuación, se muestra una solicitud de ejemplo que inicia una función de orquestador `RestartVMs` e incluye la carga del objeto JSON:

```http
POST /runtime/webhooks/durabletask/orchestrators/RestartVMs?code=XXX
Content-Type: application/json
Content-Length: 83

{
    "resourceGroup": "myRG",
    "subscriptionId": "111deb5d-09df-4604-992e-a968345530a9"
}
```

La carga de respuesta para los casos **HTTP 202** es un objeto JSON con los siguientes campos:

| Campo                       | Descripción                          |
|-----------------------------|--------------------------------------|
| **`id`**                    |Identificador de la instancia de orquestación. |
| **`statusQueryGetUri`**     |Dirección URL del estado de la instancia de orquestación. |
| **`sendEventPostUri`**      |Dirección URL de generación del evento de la instancia de orquestación. |
| **`terminatePostUri`**      |Dirección URL de finalización de la instancia de orquestación. |
| **`purgeHistoryDeleteUri`** |La dirección URL del "historial de purga" de la instancia de orquestación. |
| **`rewindPostUri`**         |(versión preliminar) La dirección URL de "rebobinado" de la instancia de orquestación. |

El tipo de datos de todos los campos es `string`.

A continuación, se muestra una carga de respuesta de ejemplo para una instancia de orquestación con `abc123` como identificador (con formato para mejorar la legibilidad):

```http
{
    "id": "abc123",
    "purgeHistoryDeleteUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123?code=XXX",
    "sendEventPostUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123/raiseEvent/{eventName}?code=XXX",
    "statusQueryGetUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123?code=XXX",
    "terminatePostUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123/terminate?reason={text}&code=XXX"
}
```

La respuesta HTTP está diseñada para ser compatible con el *patrón de consumidor de sondeo*. También incluye los siguientes encabezados de respuesta significativos:

* **Ubicación**: dirección URL del punto de conexión de estado. Esta dirección URL contiene el mismo valor que el campo `statusQueryGetUri`.
* **Retry-After**: número de segundos que se va a esperar entre las operaciones de sondeo. El valor predeterminado es `10`.

Para más información sobre el patrón de sondeo HTTP asincrónico, consulte la documentación de [seguimiento de operaciones HTTP asincrónicas](durable-functions-http-features.md#async-operation-tracking).

## <a name="get-instance-status"></a>Obtención del estado de la instancia

Para obtener el estado de una instancia de orquestación específica:

### <a name="request"></a>Solicitud

Para la versión 1.x del entorno de ejecución de Functions, la solicitud tiene el formato siguiente (se muestran varias líneas para mayor claridad):

```http
GET /admin/extensions/DurableTaskExtension/instances/{instanceId}
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &showHistory=[true|false]
    &showHistoryOutput=[true|false]
    &showInput=[true|false]
    &returnInternalServerErrorOnFailure=[true|false]
```

En la versión 2.x del entorno de ejecución de Functions, el formato de dirección URL tiene los mismos parámetros, pero un prefijo ligeramente diferente:

```http
GET /runtime/webhooks/durabletask/instances/{instanceId}
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &showHistory=[true|false]
    &showHistoryOutput=[true|false]
    &showInput=[true|false]
    &returnInternalServerErrorOnFailure=[true|false]
```

Los parámetros de solicitud de esta API incluyen el conjunto predeterminado mencionado anteriormente, así como los siguientes parámetros únicos:

| Campo                   | Tipo de parámetro  | Descripción |
|-------------------------|-----------------|-------------|
| **`instanceId`**        | URL             | Identificador de la instancia de orquestación. |
| **`showInput`**         | Cadena de consulta    | Parámetro opcional. Si se establece en `false`, la entrada de la función no se incluye en la carga de respuesta.|
| **`showHistory`**       | Cadena de consulta    | Parámetro opcional. Si se establece en `true`, el historial de ejecución de orquestación se incluirá en la carga de respuesta.|
| **`showHistoryOutput`** | Cadena de consulta    | Parámetro opcional. Si se establece en `true`, las salidas de la función se incluyen en el historial de ejecución de la orquestación.|
| **`createdTimeFrom`**   | Cadena de consulta    | Parámetro opcional. Cuando se especifica, se filtra la lista de instancias devueltas que se crearon durante la marca de tiempo ISO8601 especificada o después de esta.|
| **`createdTimeTo`**     | Cadena de consulta    | Parámetro opcional. Cuando se especifica, se filtra la lista de instancias devueltas que se crearon durante la marca de tiempo ISO8601 especificada o antes de esta.|
| **`runtimeStatus`**     | Cadena de consulta    | Parámetro opcional. Cuando se especifica, se filtra la lista de instancias devueltas según su estado en tiempo de ejecución. Para ver la lista de valores posibles del estado en tiempo de ejecución, consulte el artículo [Consulta de instancias](durable-functions-instance-management.md). |
| **`returnInternalServerErrorOnFailure`**  | Cadena de consulta    | Parámetro opcional. Si se establece en `true`, esta API devolverá una respuesta HTTP 500, en lugar de HTTP 200, si la instancia esta en estado de error. Este parámetro está pensado para escenarios automatizados de sondeo de estado. |

### <a name="response"></a>Response

Se pueden devolver varios valores de código de estado.

* **HTTP 200 (correcto)** : la instancia especificada está en estado completado o de error.
* **HTTP 202 (aceptado)** : la instancia especificada está en curso.
* **HTTP 400 (solicitud incorrecta)** : se produjo un error en la instancia especificada o esta se ha finalizado.
* **HTTP 404 (no se encuentra)** : la instancia especificada no existe o no ha empezado a ejecutarse.
* **HTTP 500 (error interno del servidor)** : solo se devuelve cuando `returnInternalServerErrorOnFailure` está establecido en `true` y la instancia especificada ha generado una excepción no controlada.

La carga de respuesta para los casos **HTTP 200** y **HTTP 202** es un objeto JSON con los siguientes campos:

| Campo                 | Tipo de datos | Descripción |
|-----------------------|-----------|-------------|
| **`runtimeStatus`**   | string    | Estado en tiempo de ejecución de la instancia. Los valores son *En ejecución*, *Pendiente*, *Erróneo*, *Cancelado*, *Finalizado*, *Completado*. |
| **`input`**           | JSON      | Datos JSON usados para inicializar la instancia. Este campo es `null` si el parámetro de cadena de consulta `showInput` esté establecido en `false`.|
| **`customStatus`**    | JSON      | Datos JSON que se usan en el estado de orquestación personalizada. Este campo es `null` si no se establece. |
| **`output`**          | JSON      | Salida JSON de la instancia. Este campo es `null` si el estado de la instancia no es Completado. |
| **`createdTime`**     | string    | Hora a la que se creó la instancia. Usa la notación ampliada de ISO 8601. |
| **`lastUpdatedTime`** | string    | Hora a la que se almacenó la instancia por última vez. Usa la notación ampliada de ISO 8601. |
| **`historyEvents`**   | JSON      | Una matriz JSON que contiene el historial de ejecución de orquestación. Este campo es `null` a menos que el parámetro de cadena de consulta `showHistory` esté establecido en `true`. |

Este es un ejemplo de una carga de respuesta que incluye las salidas de historial de ejecución de orquestación y de actividad (a la que se ha aplicado formato para mejorar la legibilidad):

```json
{
  "createdTime": "2018-02-28T05:18:49Z",
  "historyEvents": [
      {
          "EventType": "ExecutionStarted",
          "FunctionName": "E1_HelloSequence",
          "Timestamp": "2018-02-28T05:18:49.3452372Z"
      },
      {
          "EventType": "TaskCompleted",
          "FunctionName": "E1_SayHello",
          "Result": "Hello Tokyo!",
          "ScheduledTime": "2018-02-28T05:18:51.3939873Z",
          "Timestamp": "2018-02-28T05:18:52.2895622Z"
      },
      {
          "EventType": "TaskCompleted",
          "FunctionName": "E1_SayHello",
          "Result": "Hello Seattle!",
          "ScheduledTime": "2018-02-28T05:18:52.8755705Z",
          "Timestamp": "2018-02-28T05:18:53.1765771Z"
      },
      {
          "EventType": "TaskCompleted",
          "FunctionName": "E1_SayHello",
          "Result": "Hello London!",
          "ScheduledTime": "2018-02-28T05:18:53.5170791Z",
          "Timestamp": "2018-02-28T05:18:53.891081Z"
      },
      {
          "EventType": "ExecutionCompleted",
          "OrchestrationStatus": "Completed",
          "Result": [
              "Hello Tokyo!",
              "Hello Seattle!",
              "Hello London!"
          ],
          "Timestamp": "2018-02-28T05:18:54.3660895Z"
      }
  ],
  "input": null,
  "customStatus": { "nextActions": ["A", "B", "C"], "foo": 2 },
  "lastUpdatedTime": "2018-02-28T05:18:54Z",
  "output": [
      "Hello Tokyo!",
      "Hello Seattle!",
      "Hello London!"
  ],
  "runtimeStatus": "Completed"
}
```

La respuesta **HTTP 202** también incluye un encabezado de respuesta **Location** que hace referencia a la misma dirección URL que el campo `statusQueryGetUri` mencionado anteriormente.

## <a name="get-all-instances-status"></a>Obtención de todos los estados de instancias

También puede consultar el estado de todas las instancias mediante la eliminación de `instanceId` de la solicitud "Obtener el estado de la instancia". En este caso, los parámetros básicos son los mismos que en "Obtener el estado de la instancia". También se admiten parámetros de cadena de consulta para filtrar.

### <a name="request"></a>Solicitud

Para la versión 1.x del entorno de ejecución de Functions, la solicitud tiene el formato siguiente (se muestran varias líneas para mayor claridad):

```http
GET /admin/extensions/DurableTaskExtension/instances
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &createdTimeFrom={timestamp}
    &createdTimeTo={timestamp}
    &runtimeStatus={runtimeStatus1,runtimeStatus2,...}
    &showInput=[true|false]
    &top={integer}
```

En la versión 2.x del entorno de ejecución de Functions, el formato de dirección URL tiene los mismos parámetros, pero un prefijo ligeramente diferente:

```http
GET /runtime/webhooks/durableTask/instances?
    taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &createdTimeFrom={timestamp}
    &createdTimeTo={timestamp}
    &runtimeStatus={runtimeStatus1,runtimeStatus2,...}
    &showInput=[true|false]
    &top={integer}
```

Los parámetros de solicitud de esta API incluyen el conjunto predeterminado mencionado anteriormente, así como los siguientes parámetros únicos:

| Campo                   | Tipo de parámetro  | Descripción |
|-------------------------|-----------------|-------------|
| **`instanceId`**        | URL             | Identificador de la instancia de orquestación. |
| **`showInput`**         | Cadena de consulta    | Parámetro opcional. Si se establece en `false`, la entrada de la función no se incluye en la carga de respuesta.|
| **`showHistory`**       | Cadena de consulta    | Parámetro opcional. Si se establece en `true`, el historial de ejecución de orquestación se incluirá en la carga de respuesta.|
| **`showHistoryOutput`** | Cadena de consulta    | Parámetro opcional. Si se establece en `true`, las salidas de la función se incluyen en el historial de ejecución de la orquestación.|
| **`createdTimeFrom`**   | Cadena de consulta    | Parámetro opcional. Cuando se especifica, se filtra la lista de instancias devueltas que se crearon durante la marca de tiempo ISO8601 especificada o después de esta.|
| **`createdTimeTo`**     | Cadena de consulta    | Parámetro opcional. Cuando se especifica, se filtra la lista de instancias devueltas que se crearon durante la marca de tiempo ISO8601 especificada o antes de esta.|
| **`runtimeStatus`**     | Cadena de consulta    | Parámetro opcional. Cuando se especifica, se filtra la lista de instancias devueltas según su estado en tiempo de ejecución. Para ver la lista de valores posibles del estado en tiempo de ejecución, consulte el artículo [Consulta de instancias](durable-functions-instance-management.md). |
| **`top`**               | Cadena de consulta    | Parámetro opcional. Cuando se especifica, limita el número de instancias que devuelve la consulta. |

### <a name="response"></a>Response

Este es un ejemplo de cargas de respuesta, incluido el estado de orquestación (con formato para mejorar la legibilidad):

```json
[
    {
        "instanceId": "7af46ff000564c65aafbfe99d07c32a5",
        "runtimeStatus": "Completed",
        "input": null,
        "customStatus": null,
        "output": [
            "Hello Tokyo!",
            "Hello Seattle!",
            "Hello London!"
        ],
        "createdTime": "2018-06-04T10:46:39Z",
        "lastUpdatedTime": "2018-06-04T10:46:47Z"
    },
    {
        "instanceId": "80eb7dd5c22f4eeba9f42b062794321e",
        "runtimeStatus": "Running",
        "input": null,
        "customStatus": null,
        "output": null,
        "createdTime": "2018-06-04T15:18:28Z",
        "lastUpdatedTime": "2018-06-04T15:18:38Z"
    },
    {
        "instanceId": "9124518926db408ab8dfe84822aba2b1",
        "runtimeStatus": "Completed",
        "input": null,
        "customStatus": null,
        "output": [
            "Hello Tokyo!",
            "Hello Seattle!",
            "Hello London!"
        ],
        "createdTime": "2018-06-04T10:46:54Z",
        "lastUpdatedTime": "2018-06-04T10:47:03Z"
    },
    {
        "instanceId": "d100b90b903c4009ba1a90868331b11b",
        "runtimeStatus": "Pending",
        "input": null,
        "customStatus": null,
        "output": null,
        "createdTime": "2018-06-04T15:18:39Z",
        "lastUpdatedTime": "2018-06-04T15:18:39Z"
    }
]
```

> [!NOTE]
> Esta operación puede ser muy costosa en términos de E/S de Azure Storage si usa el [proveedor predeterminado de Azure Storage](durable-functions-storage-providers.md#azure-storage) y si hay muchas filas en la tabla Instancias. Puede encontrar más detalles sobre la tabla Instancias en la documentación [Rendimiento y escalado horizontal en Durable Functions (Azure Functions)](durable-functions-perf-and-scale.md#instances-table).

Si existen más resultados, se devuelve un token de continuación en el encabezado de respuesta.  El nombre del encabezado es `x-ms-continuation-token`.

Si establece el valor del token de continuación en el siguiente encabezado de solicitud, puede obtener la página de resultados siguiente. El nombre del encabezado de solicitud también es `x-ms-continuation-token`.

## <a name="purge-single-instance-history"></a>Purgado del historial de una instancia

Elimina el historial y los artefactos relacionados para una instancia de orquestación específica.

### <a name="request"></a>Solicitud

Para la versión 1.x del entorno de ejecución de Functions, la solicitud tiene el formato siguiente (se muestran varias líneas para mayor claridad):

```http
DELETE /admin/extensions/DurableTaskExtension/instances/{instanceId}
    ?taskHub={taskHub}
    &connection={connection}
    &code={systemKey}
```

En la versión 2.x del entorno de ejecución de Functions, el formato de dirección URL tiene los mismos parámetros, pero un prefijo ligeramente diferente:

```http
DELETE /runtime/webhooks/durabletask/instances/{instanceId}
    ?taskHub={taskHub}
    &connection={connection}
    &code={systemKey}
```

Los parámetros de solicitud de esta API incluyen el conjunto predeterminado mencionado anteriormente, así como los siguientes parámetros únicos:

| Campo             | Tipo de parámetro  | Descripción |
|-------------------|-----------------|-------------|
| **`instanceId`**  | URL             | Identificador de la instancia de orquestación. |

### <a name="response"></a>Response

Se pueden devolver los siguientes valores de código de estado HTTP.

* **HTTP 200 (correcto)** : El historial de la instancia se ha purgado correctamente.
* **HTTP 404 (no se encuentra)** : La instancia especificada no existe.

La carga de respuesta para el caso **HTTP 200** es un objeto JSON con el siguiente campo:

| Campo                  | Tipo de datos | Descripción |
|------------------------|-----------|-------------|
| **`instancesDeleted`** | integer   | Número de instancias eliminadas. Para el caso de instancia única, este valor debe ser siempre `1`. |

Esta es una carga de respuesta de ejemplo (con formato para mejorar la legibilidad):

```json
{
    "instancesDeleted": 1
}
```

## <a name="purge-multiple-instance-histories"></a>Purgado de historiales de varias instancias

También puede eliminar el historial y los artefactos relacionados de varias instancias de una central de tareas mediante la eliminación de `{instanceId}` de la solicitud "Purge single instance history" (Purgar el historial de una instancia). Para purgar selectivamente el historial de instancia, use los mismos filtros que se describe en la solicitud "Obtener el estado de todas las instancias".

### <a name="request"></a>Solicitud

Para la versión 1.x del entorno de ejecución de Functions, la solicitud tiene el formato siguiente (se muestran varias líneas para mayor claridad):

```http
DELETE /admin/extensions/DurableTaskExtension/instances
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &createdTimeFrom={timestamp}
    &createdTimeTo={timestamp}
    &runtimeStatus={runtimeStatus1,runtimeStatus2,...}
```

En la versión 2.x del entorno de ejecución de Functions, el formato de dirección URL tiene los mismos parámetros, pero un prefijo ligeramente diferente:

```http
DELETE /runtime/webhooks/durabletask/instances
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &createdTimeFrom={timestamp}
    &createdTimeTo={timestamp}
    &runtimeStatus={runtimeStatus1,runtimeStatus2,...}
```

Los parámetros de solicitud de esta API incluyen el conjunto predeterminado mencionado anteriormente, así como los siguientes parámetros únicos:

| Campo                 | Tipo de parámetro  | Descripción |
|-----------------------|-----------------|-------------|
| **`createdTimeFrom`** | Cadena de consulta    | Se filtra la lista de instancias purgadas que se crearon durante la marca de tiempo ISO8601 especificada o después de esta.|
| **`createdTimeTo`**   | Cadena de consulta    | Parámetro opcional. Cuando se especifica, se filtra la lista de instancias purgadas que se crearon durante la marca de tiempo ISO8601 especificada o antes de esta.|
| **`runtimeStatus`**   | Cadena de consulta    | Parámetro opcional. Cuando se especifica, se filtra la lista de instancias purgadas según su estado en tiempo de ejecución. Para ver la lista de valores posibles del estado en tiempo de ejecución, consulte el artículo [Consulta de instancias](durable-functions-instance-management.md). |

> [!NOTE]
> Esta operación puede ser muy costosa en términos de E/S de Azure Storage si usa el [proveedor predeterminado de Azure Storage](durable-functions-storage-providers.md#azure-storage) y si hay muchas filas en las tablas Instancias o Historial. Puede encontrar más detalles sobre estas tablas en la documentación [Performance and scale in Durable Functions (Azure Functions)](durable-functions-perf-and-scale.md#instances-table) (Rendimiento y escalado horizontal en Durable Functions [Azure Functions]).

### <a name="response"></a>Response

Se pueden devolver los siguientes valores de código de estado HTTP.

* **HTTP 200 (correcto)** : El historial de la instancia se ha purgado correctamente.
* **HTTP 404 (no se encuentra)** : No se encontraron instancias que coincidan con la expresión de filtro.

La carga de respuesta para el caso **HTTP 200** es un objeto JSON con el siguiente campo:

| Campo                   | Tipo de datos | Descripción |
|-------------------------|-----------|-------------|
| **`instancesDeleted`**  | integer   | Número de instancias eliminadas. |

Esta es una carga de respuesta de ejemplo (con formato para mejorar la legibilidad):

```json
{
    "instancesDeleted": 250
}
```

## <a name="raise-event"></a>Generación de eventos

Para enviar un mensaje de notificación de eventos a una instancia de orquestación en ejecución:

### <a name="request"></a>Solicitud

Para la versión 1.x del entorno de ejecución de Functions, la solicitud tiene el formato siguiente (se muestran varias líneas para mayor claridad):

```http
POST /admin/extensions/DurableTaskExtension/instances/{instanceId}/raiseEvent/{eventName}
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
```

En la versión 2.x del entorno de ejecución de Functions, el formato de dirección URL tiene los mismos parámetros, pero un prefijo ligeramente diferente:

```http
POST /runtime/webhooks/durabletask/instances/{instanceId}/raiseEvent/{eventName}
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
```

Los parámetros de solicitud de esta API incluyen el conjunto predeterminado mencionado anteriormente, así como los siguientes parámetros únicos:

| Campo             | Tipo de parámetro  | Descripción |
|-------------------|-----------------|-------------|
| **`instanceId`**  | URL             | Identificador de la instancia de orquestación. |
| **`eventName`**   | URL             | Nombre del evento al que espera la instancia de orquestación de destino. |
| **`{content}`**   | Contenido de la solicitud | Carga del evento con formato JSON. |

### <a name="response"></a>Response

Se pueden devolver varios valores de código de estado.

* **HTTP 202 (aceptado)** : se aceptó el evento generado para su procesamiento.
* **HTTP 400 (solicitud incorrecta)** : el contenido de la solicitud no era del tipo `application/json` o no tenía un valor JSON válido.
* **HTTP 404 (no se encuentra)** : no se encontró la instancia especificada.
* **HTTP 410 (no existe)** : la instancia especificada ha finalizado o se ha producido un error y no puede procesar los eventos generados.

Esta es una solicitud de ejemplo que envía la cadena JSON `"incr"` a una instancia que espera un evento denominado **operation**:

```http
POST /admin/extensions/DurableTaskExtension/instances/bcf6fb5067b046fbb021b52ba7deae5a/raiseEvent/operation?taskHub=DurableFunctionsHub&connection=Storage&code=XXX
Content-Type: application/json
Content-Length: 6

"incr"
```

Las respuestas para esta API no tienen contenido.

## <a name="terminate-instance"></a>Finalización de instancias

Para terminar una instancia de orquestación en ejecución:

### <a name="request"></a>Solicitud

Para la versión 1.x del entorno de ejecución de Functions, la solicitud tiene el formato siguiente (se muestran varias líneas para mayor claridad):

```http
POST /admin/extensions/DurableTaskExtension/instances/{instanceId}/terminate
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &reason={text}
```

En la versión 2.x del entorno de ejecución de Functions, el formato de dirección URL tiene los mismos parámetros, pero un prefijo ligeramente diferente:

```http
POST /runtime/webhooks/durabletask/instances/{instanceId}/terminate
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &reason={text}
```

Los parámetros de solicitud de esta API incluyen el conjunto predeterminado mencionado anteriormente, así como el siguiente parámetro único.

| Campo             | Tipo de parámetro  | Descripción |
|-------------------|-----------------|-------------|
| **`instanceId`**  | URL             | Identificador de la instancia de orquestación. |
| **`reason`**      | Cadena de consulta    | Opcional. Motivo de finalización de la instancia de orquestación. |

### <a name="response"></a>Response

Se pueden devolver varios valores de código de estado.

* **HTTP 202 (aceptado)** : se aceptó la solicitud de finalización para su procesamiento.
* **HTTP 404 (no se encuentra)** : no se encontró la instancia especificada.
* **HTTP 410 (no existe)** : la instancia especificada se ha completado o ha producido un error.

Esta es una solicitud de ejemplo que finaliza una instancia en ejecución y especifica un motivo **buggy**:

```
POST /admin/extensions/DurableTaskExtension/instances/bcf6fb5067b046fbb021b52ba7deae5a/terminate?reason=buggy&taskHub=DurableFunctionsHub&connection=Storage&code=XXX
```

Las respuestas para esta API no tienen contenido.

## <a name="rewind-instance-preview"></a>Rebobinado de instancias (versión preliminar)

Restaura una instancia de orquestación errónea a un estado de ejecución mediante la reproducción de las operaciones erróneas más recientes.

### <a name="request"></a>Solicitud

Para la versión 1.x del entorno de ejecución de Functions, la solicitud tiene el formato siguiente (se muestran varias líneas para mayor claridad):

```http
POST /admin/extensions/DurableTaskExtension/instances/{instanceId}/rewind
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &reason={text}
```

En la versión 2.x del entorno de ejecución de Functions, el formato de dirección URL tiene los mismos parámetros, pero un prefijo ligeramente diferente:

```http
POST /runtime/webhooks/durabletask/instances/{instanceId}/rewind
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &reason={text}
```

Los parámetros de solicitud de esta API incluyen el conjunto predeterminado mencionado anteriormente, así como el siguiente parámetro único.

| Campo             | Tipo de parámetro  | Descripción |
|-------------------|-----------------|-------------|
| **`instanceId`**  | URL             | Identificador de la instancia de orquestación. |
| **`reason`**      | Cadena de consulta    | Opcional. Motivo para rebobinar la instancia de orquestación. |

### <a name="response"></a>Response

Se pueden devolver varios valores de código de estado.

* **HTTP 202 (aceptado)** : se aceptó la solicitud de rebobinado para su procesamiento.
* **HTTP 404 (no se encuentra)** : no se encontró la instancia especificada.
* **HTTP 410 (no existe)** : la instancia especificada se completó o se terminó.

Esta es una solicitud de ejemplo que rebobina una instancia errónea y especifica un motivo de **fixed**:

```http
POST /admin/extensions/DurableTaskExtension/instances/bcf6fb5067b046fbb021b52ba7deae5a/rewind?reason=fixed&taskHub=DurableFunctionsHub&connection=Storage&code=XXX
```

Las respuestas para esta API no tienen contenido.

## <a name="signal-entity"></a>Entidad de señal

Envía un mensaje de operación unidireccional a una [entidad duradera](durable-functions-types-features-overview.md#entity-functions). Si la entidad no existe, se creará automáticamente.

> [!NOTE]
> Las entidades durables están disponibles a partir de Durable Functions 2.0.

### <a name="request"></a>Solicitud

La solicitud HTTP tiene el formato siguiente (se muestran varias líneas para mayor claridad):

```http
POST /runtime/webhooks/durabletask/entities/{entityName}/{entityKey}
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &op={operationName}
```

Los parámetros de solicitud de esta API incluyen el conjunto predeterminado mencionado anteriormente, así como los siguientes parámetros únicos:

| Campo             | Tipo de parámetro  | Descripción |
|-------------------|-----------------|-------------|
| **`entityName`**  | URL             | El nombre (tipo) de la entidad. |
| **`entityKey`**   | URL             | La clave (identificador único) de la entidad. |
| **`op`**          | Cadena de consulta    | Opcional. El nombre de la operación definida por el usuario que se va a invocar. |
| **`{content}`**   | Contenido de la solicitud | Carga del evento con formato JSON. |

A continuación se muestra una solicitud de ejemplo que envía un mensaje "Add" definido por el usuario a una entidad `Counter`denominada `steps`. El contenido del mensaje es el valor `5`. Si la entidad todavía no existe, se creará mediante esta solicitud:

```http
POST /runtime/webhooks/durabletask/entities/Counter/steps?op=Add
Content-Type: application/json

5
```

> [!NOTE]
> De forma predeterminada en las [entidades basadas en clases en .NET](durable-functions-dotnet-entities.md#defining-entity-classes), al especificar el valor `op` de `delete` se eliminará el estado de una entidad. Sin embargo, si la entidad define una operación denominada `delete`, se invocará en su lugar la operación definida por el usuario.

### <a name="response"></a>Response

Esta operación tiene varias respuestas posibles:

* **HTTP 202 (aceptado)** : la operación de la señal se aceptó para el procesamiento asincrónico.
* **HTTP 400 (solicitud incorrecta)** : el contenido de la solicitud no era del tipo `application/json`, no tenía un valor JSON válido o no tenía un valor `entityKey` válido.
* **HTTP 404 (no se encuentra)** : no se encontró el `entityName` especificado.

Una solicitud HTTP correcta no contiene nada en la respuesta. Una solicitud HTTP con error puede contener información de error con formato JSON en el contenido de la respuesta.

## <a name="get-entity"></a>Obtención de la entidad

Obtiene el estado de la entidad especificada.

### <a name="request"></a>Solicitud

La solicitud HTTP tiene el formato siguiente (se muestran varias líneas para mayor claridad):

```http
GET /runtime/webhooks/durabletask/entities/{entityName}/{entityKey}
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
```

### <a name="response"></a>Response

Esta operación tiene dos respuestas posibles:

* **HTTP 200 (correcto)** : la entidad especificada existe.
* **HTTP 404 (no se encuentra)** : no se encontró la entidad especificada.

Una respuesta correcta contiene el estado serializado de JSON de la entidad como su contenido.

### <a name="example"></a>Ejemplo
El siguiente ejemplo de solicitud HTTP obtiene el estado de una entidad `Counter` existente denominada `steps`:

```http
GET /runtime/webhooks/durabletask/entities/Counter/steps
```

Si la entidad `Counter` simplemente contenía una serie de pasos guardados en un campo `currentValue`, el contenido de la respuesta podría ser similar al siguiente (con formato para facilitar la legibilidad):

```json
{
    "currentValue": 5
}
```

## <a name="list-entities"></a>Entidades de lista

Puede consultar varias entidades por el nombre de la entidad o por la fecha de la última operación.

### <a name="request"></a>Solicitud

La solicitud HTTP tiene el formato siguiente (se muestran varias líneas para mayor claridad):

```http
GET /runtime/webhooks/durabletask/entities/{entityName}
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &lastOperationTimeFrom={timestamp}
    &lastOperationTimeTo={timestamp}
    &fetchState=[true|false]
    &top={integer}
```

Los parámetros de solicitud de esta API incluyen el conjunto predeterminado mencionado anteriormente, así como los siguientes parámetros únicos:

| Campo                       | Tipo de parámetro  | Descripción |
|-----------------------------|-----------------|-------------|
| **`entityName`**            | URL             | Opcional. Cuando se especifica, filtra la lista de entidades devueltas por su nombre de entidad (no distingue mayúsculas y minúsculas). |
| **`fetchState`**            | Cadena de consulta    | Parámetro opcional. Si se establece en `true`, el estado de la entidad se incluirá en la carga de respuesta. |
| **`lastOperationTimeFrom`** | Cadena de consulta    | Parámetro opcional. Cuando se especifica, filtra la lista de entidades devueltas que procesaron operaciones después de la marca de tiempo ISO8601 especificada. |
| **`lastOperationTimeTo`**   | Cadena de consulta    | Parámetro opcional. Cuando se especifica, filtra la lista de entidades devueltas que procesaron operaciones antes de la marca de tiempo ISO8601 especificada. |
| **`top`**                   | Cadena de consulta    | Parámetro opcional. Cuando se especifica, limita el número de entidades que devuelve la consulta. |


### <a name="response"></a>Response

Una respuesta HTTP 200 correcta contiene una matriz de entidades serializada con JSON y, opcionalmente, el estado de cada entidad.

De forma predeterminada, la operación devuelve las 100 primeras entidades que coinciden con los criterios de consulta. El autor de la llamada puede especificar un valor de parámetro de cadena de consulta para que `top` devuelva un número máximo de resultados diferente. Si existen otros resultados más allá de lo que se devuelve, también se devuelve un token de continuación en el encabezado de respuesta. El nombre del encabezado es `x-ms-continuation-token`.

Si establece el valor del token de continuación en el siguiente encabezado de solicitud, puede obtener la página de resultados siguiente. El nombre del encabezado de solicitud también es `x-ms-continuation-token`.

### <a name="example---list-all-entities"></a>Ejemplo: lista de todas las entidades

En la siguiente solicitud HTTP de ejemplo se enumeran todas las entidades de la central de tareas:

```http
GET /runtime/webhooks/durabletask/entities
```

El JSON de respuesta puede tener un aspecto similar al siguiente (se ha dado formato para mejorar la legibilidad):

```json
[
    {
        "entityId": { "key": "cats", "name": "counter" },
        "lastOperationTime": "2019-12-18T21:45:44.6326361Z",
    },
    {
        "entityId": { "key": "dogs", "name": "counter" },
        "lastOperationTime": "2019-12-18T21:46:01.9477382Z"
    },
    {
        "entityId": { "key": "mice", "name": "counter" },
        "lastOperationTime": "2019-12-18T21:46:15.4626159Z"
    },
    {
        "entityId": { "key": "radio", "name": "device" },
        "lastOperationTime": "2019-12-18T21:46:18.2616154Z"
    },
]
```

### <a name="example---filtering-the-list-of-entities"></a>Ejemplo: filtrado de la lista de entidades

En la siguiente solicitud HTTP de ejemplo solo se enumeran las dos primeras entidades de tipo `counter` y también se captura su estado:

```http
GET /runtime/webhooks/durabletask/entities/counter?top=2&fetchState=true
```

El JSON de respuesta puede tener un aspecto similar al siguiente (se ha dado formato para mejorar la legibilidad):

```json
[
    {
        "entityId": { "key": "cats", "name": "counter" },
        "lastOperationTime": "2019-12-18T21:45:44.6326361Z",
        "state": { "value": 9 }
    },
    {
        "entityId": { "key": "dogs", "name": "counter" },
        "lastOperationTime": "2019-12-18T21:46:01.9477382Z",
        "state": { "value": 10 }
    }
]
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Aprenda a usar Application Insights para supervisar sus instancias de Durable Functions](durable-functions-diagnostics.md)
