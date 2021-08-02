---
title: Referencia de la API de MQTT 5 de Azure IoT Hub (versión preliminar)
description: Más información sobre la referencia de la API de MQTT 5 de IoT Hub
services: iot-hub
author: jlian
ms.service: iot-fundamentals
ms.topic: reference
ms.date: 11/19/2020
ms.author: jlian
ms.openlocfilehash: b4102171a14abc6eeb037f8b7425b7923f7b0651
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110069839"
---
# <a name="iot-hub-data-plane-mqtt-5-api-reference"></a>Referencia de la API de MQTT 5 del plano de datos de IoT Hub

En este documento se definen las operaciones disponibles en la versión 2.0 (api-version: `2020-10-01-preview`) de la API del plano de datos de IoT Hub.

## <a name="operations"></a>Operations

### <a name="get-twin"></a>Get Twin

Estado de Get Twin

#### <a name="request"></a>Solicitud

**Nombre del tema:** `$iothub/twin/get`

**Propiedades**: ninguna

**Carga útil**: vacía

#### <a name="success-response"></a>Respuesta de acción correcta

**Propiedades**: ninguna

**Carga útil**: Gemelo

#### <a name="alternative-responses"></a>Respuestas alternativas

| Estado | Nombre | Descripción |
| :----- | :--- | :---------- |
| 0100 |  Bad Request | El mensaje de la operación tiene un formato incorrecto y no se puede procesar. |
| 0101 |  No autorizado | El cliente no tiene autorización para realizar la operación. |
| 0102 |  No permitida | No se permite la operación. |
| 0501 |  Limitado | La tasa de solicitudes es demasiado alta por SKU. |
| 0502 |  Cuota superada | Se superó la cuota diaria por SKU actual. |
| 0601 |  Error de servidor | Error interno del servidor. |
| 0602 |  Tiempo de espera | Se agotó el tiempo de espera de la operación antes de que pudiera completarse. |
| 0603 |  Servidor ocupado | Servidor ocupado. |

#### <a name="pseudo-code-sample"></a>Ejemplo de pseudocódigo

```

-> PUBLISH
    QoS: 0
    Topic: $iothub/twin/get
<- PUBLISH
    QoS: 0
    Topic: $iothub/responses

```

### <a name="patch-twin-reported"></a>Operación Patch Twin notificada

Estado notificado de Patch Twin

#### <a name="request"></a>Solicitud

**Nombre del tema:** `$iothub/twin/patch/reported`

**Propiedades**:

| Nombre | Tipo | Obligatorio | Descripción |
| :--- | :--- | :------- | :---------- |
| if-version | u64 | no |  |

**Carga útil**: TwinState

#### <a name="success-response"></a>Respuesta de acción correcta

**Propiedades**:

| Nombre | Tipo | Obligatorio | Descripción |
| :--- | :--- | :------- | :---------- |
| version | u64 | sí | Versión de estado notificado después de aplicar la revisión. |

**Carga útil**: vacía

#### <a name="alternative-responses"></a>Respuestas alternativas

| Estado | Nombre | Descripción |
| :----- | :--- | :---------- |
| 0104 |  Error en la condición previa | No se cumplió la condición previa, lo que provocó la cancelación de la solicitud. |
| 0100 |  Bad Request | El mensaje de la operación tiene un formato incorrecto y no se puede procesar. |
| 0101 |  No autorizado | El cliente no tiene autorización para realizar la operación. |
| 0102 |  No permitida | No se permite la operación. |
| 0501 |  Limitado | La tasa de solicitudes es demasiado alta por SKU. |
| 0502 |  Cuota superada | Se superó la cuota diaria por SKU actual. |
| 0601 |  Error de servidor | Error interno del servidor. |
| 0602 |  Tiempo de espera | Se agotó el tiempo de espera de la operación antes de que pudiera completarse. |
| 0603 |  Servidor ocupado | Servidor ocupado. |

#### <a name="pseudo-code-sample"></a>Ejemplo de pseudocódigo

```

-> PUBLISH
    QoS: 0
    Topic: $iothub/twin/patch/reported
    [if-version: <u64>]
<- PUBLISH
    QoS: 0
    Topic: $iothub/responses

```

### <a name="receive-commands"></a>Recepción de comandos

Recepción y control de comandos

#### <a name="message"></a>Message

**Nombre del tema:** `$iothub/commands`

**Propiedades**:

| Nombre | Tipo | Obligatorio | Descripción |
| :--- | :--- | :------- | :---------- |
| sequence-no | u64 | sí | Número de secuencia del mensaje. |
| enqueued-time | time | sí | Marca de tiempo de la entrada del mensaje en el sistema. |
| delivery-count | u32 | sí | Número de veces que se intentó la entrega del mensaje. |
| creation-time | time | no | Marca de tiempo de creación del mensaje (proporcionada por el remitente). |
| message-id | string | no | Identidad del mensaje (proporcionada por el remitente). |
| user-id | string | no | Identidad de usuario (proporcionada por el remitente). |
| correlation-id | string | no | Identidad de correlación (proporcionada por el remitente). |
| Tipo de contenido | string | no | Determina el tipo de contenido de la carga útil. |
| content-encoding | string | no | Determina la codificación de contenido de la carga útil. |

**Carga útil**: cualquier secuencia de bytes

#### <a name="success-acknowledgment"></a>Reconocimiento de acción correcta

Indica que el cliente aceptó el comando para controlarlo.

**Propiedades**: ninguna

**Carga útil**: vacía

#### <a name="alternative-acknowledgments"></a>Reconocimientos alternativos

| Código de motivo | Estado | Nombre | Descripción |
| :---------- | :----- | :--- | :---------- |
| 131 | 0603 | Abandon | Indica que el comando no se procesará en este momento y que debe volver a entregarse en el futuro. |
| 131 | 0100 | Reject | Indica que el cliente ha rechazado el comando y no se debe volver a intentar. |

#### <a name="pseudo-code-sample"></a>Ejemplo de pseudocódigo

```

-> SUBSCRIBE
    - Topic: $iothub/commands
      QoS: 1
<- PUBLISH
    QoS: 1
    Topic: $iothub/commands
    sequence-no: <u64>enqueued-time: <time>delivery-count: <u32>[creation-time: <time>][message-id: <string>][user-id: <string>][correlation-id: <string>][Content Type: <string>][content-encoding: <string>]
    Payload: ...

-> PUBACK

```

### <a name="receive-direct-methods"></a>Recepción de métodos directos

Recepción y control de llamadas a métodos directos

#### <a name="request"></a>Solicitud

**Nombre del tema:** `$iothub/methods/{name}`

**Propiedades**: ninguna

**Carga útil**: cualquier secuencia de bytes

#### <a name="success-response"></a>Respuesta de acción correcta

**Propiedades**:

| Nombre | Tipo | Obligatorio | Descripción |
| :--- | :--- | :------- | :---------- |
| response-code | u32 | sí |  |

**Carga útil**: cualquier secuencia de bytes

#### <a name="alternative-responses"></a>Respuestas alternativas

| Estado | Nombre | Descripción |
| :----- | :--- | :---------- |
| 06A0 |  No disponible | Indica que el cliente no es accesible a través de esta conexión. |

#### <a name="pseudo-code-sample"></a>Ejemplo de pseudocódigo

```

-> SUBSCRIBE
    - Topic: methods/{name}
      QoS: 0
<- SUBACK
<- PUBLISH
    QoS: 0
    Topic: $iothub/methods/{name}
-> PUBLISH
    QoS: 0
    Topic: $iothub/responses

```

### <a name="receive-twin-desired-state-changes"></a>Recepción de cambios de estado deseados del gemelo

Recibir actualizaciones del estado deseado del gemelo

#### <a name="message"></a>Message

**Nombre del tema:** `$iothub/twin/patch/desired`

**Propiedades**:

| Nombre | Tipo | Obligatorio | Descripción |
| :--- | :--- | :------- | :---------- |
| version | u64 | sí | Versión del estado deseado coincidente con esta actualización. |

**Carga útil**: TwinState

#### <a name="pseudo-code-sample"></a>Ejemplo de pseudocódigo

```

-> SUBSCRIBE
    - Topic: $iothub/twin/patch/desired
      QoS: 0
<- PUBLISH
    QoS: 0
    Topic: $iothub/twin/patch/desired
    version: <u64>
    Payload: ...

```

### <a name="send-telemetry"></a>Envío de telemetría

Publicación del mensaje en el canal de telemetría: EventHubs de forma predeterminada u otro punto de conexión a través de la configuración de enrutamiento.

#### <a name="message"></a>Message

**Nombre del tema:** `$iothub/telemetry`

**Propiedades**:

| Nombre | Tipo | Obligatorio | Descripción |
| :--- | :--- | :------- | :---------- |
| Tipo de contenido | string | no | Se convierte en la propiedad del sistema `content-type` en el mensaje publicado. |
| content-encoding | string | no | Se convierte en la propiedad del sistema `content-encoding` en el mensaje publicado. |
| message-id | string | no | Se convierte en la propiedad del sistema `message-id` en el mensaje publicado. |
| user-id | string | no | Se convierte en la propiedad del sistema `user-id` en el mensaje publicado. |
| correlation-id | string | no | Se convierte en la propiedad del sistema `correlation-id` en el mensaje publicado. |
| creation-time | time | no | Se convierte en la propiedad `iothub-creation-time-utc` en el mensaje publicado. |

> [!TIP]
> El formato de `creation-time` debe ser UTC sin información de zona horaria. Por ejemplo, `2021-04-21T11:30:16Z` es válido y `2021-04-21T11:30:16-07:00` no lo es.

**Carga útil**: cualquier secuencia de bytes

#### <a name="success-acknowledgment"></a>Reconocimiento de acción correcta

El mensaje se publicó correctamente en el canal de telemetría.

**Propiedades**: ninguna

**Carga útil**: vacía

#### <a name="alternative-acknowledgments"></a>Reconocimientos alternativos

| Código de motivo | Estado | Nombre | Descripción |
| :---------- | :----- | :--- | :---------- |
| 131 | 0100 | Bad Request | El mensaje de la operación tiene un formato incorrecto y no se puede procesar. |
| 135 | 0101 | No autorizado | El cliente no tiene autorización para realizar la operación. |
| 131 | 0102 | No permitida | No se permite la operación. |
| 131 | 0601 | Error de servidor | Error interno del servidor. |
| 151 | 0501 | Limitado | La tasa de solicitudes es demasiado alta por SKU. |
| 151 | 0502 | Cuota superada | Se superó la cuota diaria por SKU actual. |
| 131 | 0602 | Tiempo de espera | Se agotó el tiempo de espera de la operación antes de que pudiera completarse. |
| 131 | 0603 | Servidor ocupado | Servidor ocupado. |

#### <a name="pseudo-code-sample"></a>Ejemplo de pseudocódigo

```
-> PUBLISH
    QoS: 1
    Topic: $iothub/telemetry
    [Content Type: <string>]
    [content-encoding: <string>]
    [message-id: <string>]
    [user-id: <string>]
    [correlation-id: <string>]
    [creation-time: <time>]

<- PUBACK

```

## <a name="responses"></a>Respuestas

### <a name="bad-request"></a>Bad Request

El mensaje de la operación tiene un formato incorrecto y no se puede procesar.

**Código del motivo:** `131`

**Estado:** `0100`

**Propiedades**:

| Nombre | Tipo | Obligatorio | Descripción |
| :--- | :--- | :------- | :---------- |
| reason | string | no | Contiene información sobre los elementos específicamente no válidos en relación con el mensaje. |

**Carga útil**: vacía

### <a name="conflict"></a>Conflicto

La operación está en conflicto con otra operación en curso.

**Código del motivo:** `131`

**Estado:** `0103`

**Propiedades**:

| Nombre | Tipo | Obligatorio | Descripción |
| :--- | :--- | :------- | :---------- |
| trace-id | string | no | Identificador de seguimiento de correlación con diagnósticos adicionales del error. |
| reason | string | no | Contiene información sobre los elementos específicamente no válidos en relación con el mensaje. |

**Carga útil**: vacía

### <a name="not-allowed"></a>No permitida

No se permite la operación.

**Código del motivo:** `131`

**Estado:** `0102`

**Propiedades**:

| Nombre | Tipo | Obligatorio | Descripción |
| :--- | :--- | :------- | :---------- |
| reason | string | no | Contiene información sobre los elementos específicamente no válidos en relación con el mensaje. |

**Carga útil**: vacía

### <a name="not-authorized"></a>No autorizado

El cliente no tiene autorización para realizar la operación.

**Código del motivo:** `135`

**Estado:** `0101`

**Propiedades**:

| Nombre | Tipo | Obligatorio | Descripción |
| :--- | :--- | :------- | :---------- |
| trace-id | string | no | Identificador de seguimiento de correlación con diagnósticos adicionales del error. |

**Carga útil**: vacía

### <a name="not-found"></a>No encontrado

El recurso de la solicitud no existe.

**Código del motivo:** `131`

**Estado:** `0504`

**Propiedades**:

| Nombre | Tipo | Obligatorio | Descripción |
| :--- | :--- | :------- | :---------- |
| reason | string | no | Contiene información sobre los elementos específicamente no válidos en relación con el mensaje. |

**Carga útil**: vacía

### <a name="not-modified"></a>No modificado

El recurso no se modificó según la condición previa proporcionada.

**Código del motivo:** `0`

**Estado:** `0001`

**Propiedades**: ninguna

**Carga útil**: vacía

### <a name="precondition-failed"></a>Error en la condición previa

No se cumplió la condición previa, lo que provocó la cancelación de la solicitud

**Código del motivo:** `131`

**Estado:** `0104`

**Propiedades**: ninguna

**Carga útil**: vacía

### <a name="quota-exceeded"></a>Cuota superada

Se superó la cuota diaria por SKU actual.

**Código del motivo:** `151`

**Estado:** `0502`

**Propiedades**: ninguna

**Carga útil**: vacía

### <a name="resource-exhausted"></a>Recurso agotado

El recurso no tiene capacidad para completar la operación.

**Código del motivo:** `131`

**Estado:** `0503`

**Propiedades**:

| Nombre | Tipo | Obligatorio | Descripción |
| :--- | :--- | :------- | :---------- |
| reason | string | no | Contiene información sobre los elementos específicamente no válidos en relación con el mensaje. |

**Carga útil**: vacía

### <a name="server-busy"></a>Servidor ocupado

Servidor ocupado.

**Código del motivo:** `131`

**Estado:** `0603`

**Propiedades**:

| Nombre | Tipo | Obligatorio | Descripción |
| :--- | :--- | :------- | :---------- |
| trace-id | string | no | Identificador de seguimiento de correlación con diagnósticos adicionales del error. |

**Carga útil**: vacía

### <a name="server-error"></a>Error de servidor

Error interno del servidor.

**Código del motivo:** `131`

**Estado:** `0601`

**Propiedades**:

| Nombre | Tipo | Obligatorio | Descripción |
| :--- | :--- | :------- | :---------- |
| trace-id | string | no | Identificador de seguimiento de correlación con diagnósticos adicionales del error. |

**Carga útil**: vacía

### <a name="target-failed"></a>Error de destino

El destino respondió pero la respuesta no era válida o tenía un formato incorrecto.

**Código del motivo:** `131`

**Estado:** `06A2`

**Propiedades**:

| Nombre | Tipo | Obligatorio | Descripción |
| :--- | :--- | :------- | :---------- |
| reason | string | no | Contiene información sobre los elementos específicamente no válidos en relación con el mensaje. |

**Carga útil**: vacía

### <a name="target-timeout"></a>Tiempo de espera agotado del destino

Se agotó el tiempo de espera para que el destino completara la solicitud.

**Código del motivo:** `131`

**Estado:** `06A1`

**Propiedades**:

| Nombre | Tipo | Obligatorio | Descripción |
| :--- | :--- | :------- | :---------- |
| trace-id | string | no | Identificador de seguimiento de correlación con diagnósticos adicionales del error. |
| reason | string | no | Contiene información sobre los elementos específicamente no válidos en relación con el mensaje. |

**Carga útil**: vacía

### <a name="target-unavailable"></a>Destino no disponible

No se puede acceder al destino para completar la solicitud.

**Código del motivo:** `131`

**Estado:** `06A0`

**Propiedades**: ninguna

**Carga útil**: vacía

### <a name="throttled"></a>Limitado

La tasa de solicitudes es demasiado alta por SKU.

**Código del motivo:** `151`

**Estado:** `0501`

**Propiedades**: ninguna

**Carga útil**: vacía

### <a name="timeout"></a>Tiempo de espera

Se agotó el tiempo de espera de la operación antes de que pudiera completarse.

**Código del motivo:** `131`

**Estado:** `0602`

**Propiedades**:

| Nombre | Tipo | Obligatorio | Descripción |
| :--- | :--- | :------- | :---------- |
| trace-id | string | no | Identificador de seguimiento de correlación con diagnósticos adicionales del error. |

**Carga útil**: vacía

