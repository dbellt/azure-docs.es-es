---
title: 'API REST: Azure Event Grid IoT Edge | Microsoft Docs'
description: API REST de Event Grid en IoT Edge.
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.subservice: iot-edge
ms.date: 05/10/2021
ms.topic: article
ms.openlocfilehash: d3f353693aeafb3093a2a8edce017cc24c0aca49
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110370691"
---
# <a name="rest-api"></a>API DE REST
En este artículo se describen las API REST de Azure Event Grid en IoT Edge

## <a name="common-api-behavior"></a>Comportamiento común de la API

### <a name="base-url"></a>URL base
Event Grid en IoT Edge tiene las siguientes API expuestas a través de HTTP (puerto 5888) y HTTPS (puerto 4438).

* URL base para HTTP: http://eventgridmodule:5888
* URL base para HTTPS: https://eventgridmodule:4438

### <a name="request-query-string"></a>Cadena de consulta de la solicitud
Todas las solicitudes de API requieren el parámetro de cadena de consulta siguiente:

```?api-version=2019-01-01-preview```

### <a name="request-content-type"></a>Tipo de contenido de la solicitud
Todas las solicitudes de API deben tener un elemento **Content-Type**.

En el caso de **EventGridSchema** o **CustomSchema**, el valor de Content-Type puede ser uno de los siguientes valores:

```Content-Type: application/json```

```Content-Type: application/json; charset=utf-8```

En el caso de **CloudEventSchemaV1_0**, el valor de Content-Type puede ser uno de los siguientes valores:

```Content-Type: application/cloudevents+json```
    
```Content-Type: application/cloudevents+json; charset=utf-8```
    
```Content-Type: application/cloudevents-batch+json```
    
```Content-Type: application/cloudevents-batch+json; charset=utf-8```

En el caso de **CloudEventSchemaV1_0** en modo binario, consulte la [documentación](https://github.com/cloudevents/spec/blob/master/http-protocol-binding.md) para obtener más detalles.

### <a name="error-response"></a>Respuesta de error
Todas las API devuelven un error con la carga útil siguiente:

```json
{
    "error":
    {
        "code": "<HTTP STATUS CODE>",
        "details": 
        {
            "code": "<Detailed Error Code>",
            "message": "..."
        }
    }
}
```

## <a name="manage-topics"></a>Administración de temas

### <a name="put-topic-create--update"></a>Colocación de tema (crear/actualizar)

**Solicitud**: ``` PUT /topics/<topic_name>?api-version=2019-01-01-preview ```

**Carga útil**:

```json
    {
        "name": "<topic_name>", // optional, inferred from URL. If specified must match URL topic_name
        "properties":
        {
            "inputSchema": "EventGridSchema | CustomEventSchema | CloudEventSchemaV1_0" // optional
        }
    }
```

**Respuesta**: HTTP 200

**Carga útil**:

```json
{
    "id": "/iotHubs/<iot_hub_name>/devices/<iot_edge_device_id>/modules/<eventgrid_module_name>/topics/<topic_name>",
    "name": "<topic_name>",
    "type": "Microsoft.EventGrid/topics",
    "properties":
    {
        "endpoint": "<get_request_base_url>/topics/<topic_name>/events?api-version=2019-01-01-preview",
        "inputSchema": "EventGridSchema | CustomEventSchema | CloudEventSchemaV1_0" // populated with EventGridSchema if not explicitly specified in PUT request
    }
}
```

### <a name="get-topic"></a>Obtención de tema

**Solicitud**: ``` GET /topics/<topic_name>?api-version=2019-01-01-preview ```

**Respuesta**: HTTP 200

**Carga útil**:
```json
{
    "id": "/iotHubs/<iot_hub_name>/devices/<iot_edge_device_id>/modules/<eventgrid_module_name>/topics/<topic_name>",
    "name": "<topic_name>",
    "type": "Microsoft.EventGrid/topics",
    "properties":
    {
        "endpoint": "<request_base_url>/topics/<topic_name>/events?api-version=2019-01-01-preview",
        "inputSchema": "EventGridSchema | CustomEventSchema | CloudEventSchemaV1_0"
    }
}
```

### <a name="get-all-topics"></a>Obtención de todos los temas

**Solicitud**: ``` GET /topics?api-version=2019-01-01-preview ```

**Respuesta**: HTTP 200

**Carga útil**:
```json
[
    {
        "id": "/iotHubs/<iot_hub_name>/devices/<iot_edge_device_id>/modules/<eventgrid_module_name>/topics/<topic_name>",
        "name": "<topic_name>",
        "type": "Microsoft.EventGrid/topics",
        "properties":
        {
            "endpoint": "<request_base_url>/topics/<topic_name>/events?api-version=2019-01-01-preview",
            "inputSchema": "EventGridSchema | CustomEventSchema | CloudEventSchemaV1_0"
        }
    },
    {
        "id": "/iotHubs/<iot_hub_name>/devices/<iot_edge_device_id>/modules/<eventgrid_module_name>/topics/<topic_name>",
        "name": "<topic_name>",
        "type": "Microsoft.EventGrid/topics",
        "properties":
        {
            "endpoint": "<request_base_url>/topics/<topic_name>/events?api-version=2019-01-01-preview",
            "inputSchema": "EventGridSchema | CustomEventSchema | CloudEventSchemaV1_0"
        }
    }
]
```

### <a name="delete-topic"></a>Eliminación de tema

**Solicitud**: ``` DELETE /topics/<topic_name>?api-version=2019-01-01-preview ```

**Respuesta**: HTTP 200, carga vacía

## <a name="manage-event-subscriptions"></a>Administración de suscripciones a eventos
En los ejemplos de esta sección se usa `EndpointType=Webhook;`. Los ejemplos de JSON para `EndpointType=EdgeHub / EndpointType=EventGrid` se encuentran en la sección siguiente. 

### <a name="put-event-subscription-create--update"></a>Colocación de suscripción a evento (crear/actualizar)

**Solicitud**: ``` PUT /topics/<topic_name>/eventSubscriptions/<subscription_name>?api-version=2019-01-01-preview ```

**Carga útil**:
```json
{
    "name": "<subscription_name>", // optional, inferred from URL. If specified must match URL subscription_name
    "properties":
    {
        "topicName": "<topic_name>", // optional, inferred from URL. If specified must match URL topic_name
        "eventDeliverySchema": "EventGridSchema | CustomEventSchema | CloudEventSchemaV1_0", // optional
        "retryPolicy":  //optional
        {
            "eventExpiryInMinutes": 120,
            "maxDeliveryAttempts": 50
        },
        "persistencePolicy": "true",
        "destination":
        {
            "endpointType": "WebHook",
            "properties":
            {
                "endpointUrl": "<webhook_url>",
                "maxEventsPerBatch": 10, // optional
                "preferredBatchSizeInKilobytes": 1033 // optional
            }
        },
        "filter": // optional
        {
            "subjectBeginsWith": "...",
            "subjectEndsWith": "...",
            "isSubjectCaseSensitive": true|false,
            "includedEventTypes": ["...", "..."],
            "advancedFilters":
            [
                {
                    "OperatorType": "BoolEquals",
                    "Key": "...",
                    "Value": "..."
                },
                {
                    "OperatorType": "NumberLessThan",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberGreaterThan",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberLessThanOrEquals",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberGreaterThanOrEquals",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberIn",
                    "Key": "...",
                    "Values": [<number>, <number>, <number>]
                },
                {
                    "OperatorType": "NumberNotIn",
                    "Key": "...",
                    "Values": [<number>, <number>, <number>]
                },
                {
                    "OperatorType": "StringIn",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringNotIn",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringBeginsWith",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringEndsWith",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringContains",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                }
            ]
        }
    }
}
```

**Respuesta**: HTTP 200

**Carga útil**:

```json
{
    "id": "/iotHubs/<iot_hub_name>/devices/<iot_edge_device_id>/modules/<eventgrid_module_name>/topics/<topic_name>/eventSubscriptions/<subscription_name>",
    "name": "<subscription_name>",
    "type": "Microsoft.EventGrid/eventSubscriptions",
    "properties":
    {
        "topicName": "<topic_name>",
        "eventDeliverySchema": "EventGridSchema | CustomEventSchema  | CloudEventSchemaV1_0", // populated with EventGridSchema if not explicitly specified in PUT request
        "retryPolicy":  // only populated if specified in the PUT request
        {
            "eventExpiryInMinutes": 120,
            "maxDeliveryAttempts": 50
        },
        "destination":
        {
            "endpointType": "WebHook",
            "properties":
            {
                "endpointUrl": "<webhook_url>",
                "maxEventsPerBatch": 10, // optional
                "preferredBatchSizeInKilobytes": 1033 // optional
            }
        },
        "filter": // only populated if specified in the PUT request 
        {
            "subjectBeginsWith": "...",
            "subjectEndsWith": "...",
            "isSubjectCaseSensitive": true|false,
            "includedEventTypes": ["...", "..."],
            "advancedFilters":
            [
                {
                    "OperatorType": "BoolEquals",
                    "Key": "...",
                    "Value": "..."
                },
                {
                    "OperatorType": "NumberLessThan",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberGreaterThan",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberLessThanOrEquals",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberGreaterThanOrEquals",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberIn",
                    "Key": "...",
                    "Values": [<number>, <number>, <number>]
                },
                {
                    "OperatorType": "NumberNotIn",
                    "Key": "...",
                    "Values": [<number>, <number>, <number>]
                },
                {
                    "OperatorType": "StringIn",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringNotIn",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringBeginsWith",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringEndsWith",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringContains",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                }
            ]
        }
    }
}
```


### <a name="get-event-subscription"></a>Obtención de una suscripción a evento

**Solicitud**: ``` GET /topics/<topic_name>/eventSubscriptions/<subscription_name>?api-version=2019-01-01-preview ```

**Respuesta**: HTTP 200

**Carga útil**:
```json
{
    "id": "/iotHubs/<iot_hub_name>/devices/<iot_edge_device_id>/modules/<eventgrid_module_name>/topics/<topic_name>/eventSubscriptions/<subscription_name>",
    "name": "<subscription_name>",
    "type": "Microsoft.EventGrid/eventSubscriptions",
    "properties":
    {
        "topicName": "<topic_name>",
        "eventDeliverySchema": "EventGridSchema | CustomEventSchema  | CloudEventSchemaV1_0", // populated with EventGridSchema if not explicitly specified in PUT request
        "retryPolicy":  // only populated if specified in the PUT request
        {
            "eventExpiryInMinutes": 120,
            "maxDeliveryAttempts": 50
        },
        "destination":
        {
            "endpointType": "WebHook",
            "properties":
            {
                "endpointUrl": "<webhook_url>",
                "maxEventsPerBatch": 10, // optional
                "preferredBatchSizeInKilobytes": 1033 // optional
            }
        },
        "filter": // only populated if specified in the PUT request 
        {
            "subjectBeginsWith": "...",
            "subjectEndsWith": "...",
            "isSubjectCaseSensitive": true|false,
            "includedEventTypes": ["...", "..."],
            "advancedFilters":
            [
                {
                    "OperatorType": "BoolEquals",
                    "Key": "...",
                    "Value": "..."
                },
                {
                    "OperatorType": "NumberLessThan",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberGreaterThan",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberLessThanOrEquals",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberGreaterThanOrEquals",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberIn",
                    "Key": "...",
                    "Values": [<number>, <number>, <number>]
                },
                {
                    "OperatorType": "NumberNotIn",
                    "Key": "...",
                    "Values": [<number>, <number>, <number>]
                },
                {
                    "OperatorType": "StringIn",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringNotIn",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringBeginsWith",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringEndsWith",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringContains",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                }
            ]
        }
    }
}
```

### <a name="get-event-subscriptions"></a>Obtención de suscripciones a eventos

**Solicitud**: ``` GET /topics/<topic_name>/eventSubscriptions?api-version=2019-01-01-preview ```

**Respuesta**: HTTP 200

**Carga útil**:
```json
[
    {
        // same event-subscription json as that returned from Get-EventSubscription above
    },
    {
    },
    ...
]
```

### <a name="delete-event-subscription"></a>Eliminación de una suscripción a evento

**Solicitud**: ``` DELETE /topics/<topic_name>/eventSubscriptions/<subscription_name>?api-version=2019-01-01-preview ```

**Respuesta**: HTTP 200, sin carga útil


## <a name="publish-events-api"></a>API de publicación de eventos

### <a name="send-batch-of-events-in-event-grid-schema"></a>Envío del lote de eventos (en el esquema de Event Grid)

**Solicitud**: ``` POST /topics/<topic_name>/events?api-version=2019-01-01-preview ```

```json
[
    {
        "id": "<user-defined-event-id>",
        "topic": "<topic_name>",
        "subject": "",
        "eventType": "",
        "eventTime": ""
        "dataVersion": "",
        "metadataVersion": "1",
        "data": 
            ...
    }
]
```

**Respuesta**: HTTP 200, carga vacía


**Descripciones de los campos de carga útil**
- ```Id``` es obligatorio. Puede ser cualquier valor de cadena rellenado por el autor de la llamada. Event Grid NO realiza ninguna detección de duplicados ni aplica ninguna semántica en este campo.
- ```Topic``` es opcional, pero si se especifica debe coincidir con el valor de topic_name de la dirección URL de la solicitud.
- ```Subject``` es obligatorio, puede ser cualquier valor de cadena.
- ```EventType``` es obligatorio, puede ser cualquier valor de cadena.
- ```EventTime``` es obligatorio. No se valida, pero debe ser una fecha y hora adecuadas.
- ```DataVersion``` es obligatorio.
- ```MetadataVersion``` es opcional, pero si se especifica debe ser una cadena con el valor ```"1"```.
- ```Data``` es opcional y puede ser cualquier token JSON (número, cadena, booleano, matriz, objeto).

### <a name="send-batch-of-events-in-custom-schema"></a>Envío del lote de eventos (en el esquema personalizado)

**Solicitud**: ``` POST /topics/<topic_name>/events?api-version=2019-01-01-preview ```

```json
[
    {
        ...
    }
]
```

**Respuesta**: HTTP 200, carga vacía


**Restricciones de la carga útil**
- DEBE ser una matriz de eventos.
- Cada entrada de la matriz DEBE ser un objeto JSON.
- Ninguna otra restricción (excepto el tamaño de la carga).

## <a name="examples"></a>Ejemplos

### <a name="set-up-topic-with-eventgrid-schema"></a>Configuración del tema con el esquema de EventGrid
Configura un tema para solicitar que se publiquen los eventos en **eventgridschema**.

```json
    {
        "name": "myeventgridtopic",
        "properties":
        {
            "inputSchema": "EventGridSchema"
        }
    }
```

### <a name="set-up-topic-with-custom-schema"></a>Configuración del tema con el esquema personalizado
Configura un tema para solicitar que se publiquen los eventos en `customschema`.

```json
    {
        "name": "mycustomschematopic",
        "properties":
        {
            "inputSchema": "CustomSchema"
        }
    }
```

### <a name="set-up-topic-with-cloud-event-schema"></a>Configuración del tema con el esquema de evento en la nube
Configura un tema para solicitar que se publiquen los eventos en `cloudeventschema`.

```json
    {
        "name": "mycloudeventschematopic",
        "properties":
        {
            "inputSchema": "CloudEventSchemaV1_0"
        }
    }
```

### <a name="set-up-webhook-as-destination-events-to-be-delivered-in-eventgridschema"></a>Configuración de webhook como destino, eventos que se van a entregar en eventgridschema
Use este tipo de destino para enviar eventos a cualquier otro módulo (que hospede un punto de conexión HTTP) o a cualquier punto de conexión HTTP disponible en la red o en Internet.

```json
{
    "properties":
    {
        "destination":
        {
            "endpointType": "WebHook",
            "properties":
            {
                "endpointUrl": "<webhook_url>",
                "eventDeliverySchema": "eventgridschema",
            }
        }
    }
}
```

Restricciones en el atributo `endpointUrl`:
- No puede ser NULL.
- Debe ser una dirección URL absoluta.
- Si outbound__webhook__httpsOnly está establecido en true en la configuración de EventGridModule, solo puede ser HTTPS.
- Si outbound__webhook__httpsOnly está establecido en false, puede ser HTTP o HTTPS.

Restricciones en la propiedad `eventDeliverySchema`:
- Debe coincidir con el esquema de entrada del tema de suscripción.
- Puede ser NULL. El valor predeterminado es el esquema de entrada del tema.

### <a name="set-up-iot-edge-as-destination"></a>Configuración de IoT Edge como destino

Use este destino para enviar eventos al centro de IoT Edge y aplicarles el subsistema de enrutamiento, filtrado y reenvío del centro de Edge.

```json
{
    "properties":
    {
        "destination":
        {
            "endpointType": "EdgeHub",
            "properties":
            {
                "outputName": "<eventgridmodule_output_port_name>"
            }
        }
    }
}
```

### <a name="set-up-event-grid-cloud-as-destination"></a>Configuración de la nube de Event Grid como destino

Use este destino para enviar eventos a Event Grid en la nube (Azure). Primero deberá configurar un tema de usuario en la nube al que se deben enviar eventos, antes de crear una suscripción de eventos en el perímetro.

```json
{
    "properties":
    {
        "destination":
        {
            "endpointType": "EventGrid",
            "properties":
            {
                "endpointUrl": "<eventgrid_user_topic_url>",
                "sasKey": "<user_topic_sas_key>",
                "topicName": "<new value to populate in forwarded EventGridEvent.Topic>" // if not specified, the Topic field on every event gets nulled out before being sent to Azure Event Grid
            }
        }
    }
}
```

EndpointUrl:
- No puede ser NULL.
- Debe ser una dirección URL absoluta.
- La ruta de acceso `/api/events` debe definirse en la ruta de la dirección URL de la solicitud.
- Debe contener `api-version=2018-01-01` en la cadena de consulta.
- Si outbound__eventgrid__httpsOnly está establecido en true en la configuración de EventGridModule (el valor predeterminado es true), solo puede ser HTTPS.
- Si outbound__eventgrid__httpsOnly está establecido en false, puede ser HTTP o HTTPS.
- Si outbound__eventgrid__allowInvalidHostnames está establecido en false (el valor predeterminado es false), debe tener como destino uno de los puntos de conexión siguientes:
   - `eventgrid.azure.net`
   - `eventgrid.azure.us`
   - `eventgrid.azure.cn`

SasKey:
- No puede ser NULL.

TopicName:
- Si Subscription.EventDeliverySchema está establecido en EventGridSchema, el valor de este campo se coloca en el campo de tema de cada evento antes de reenviarse a Event Grid en la nube.
- Si Subscription.EventDeliverySchema está establecido en CustomEventSchema, esta propiedad se omite y la carga del evento personalizado se reenvía exactamente como se recibió.

## <a name="set-up-event-hubs-as-a-destination"></a>Configuración de Event Hubs como destino

Para realizar la publicación en un centro de eventos, configure `endpointType` como `eventHub` y proporcione:

* connectionString: cadena de conexión para el centro de eventos específico que se va a usar como destino generado mediante una directiva de acceso compartido.

    >[!NOTE]
    > La cadena de conexión debe ser específica de la entidad. El uso de una cadena de conexión de espacio de nombres no funcionará. Para generar una cadena de conexión específica de la entidad, en Azure Portal, vaya al centro de eventos en el que desea realizar la publicación y haga clic en **Directivas de acceso compartido** para generar una cadena de conexión específica de la entidad.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "eventHub",
              "properties": {
                "connectionString": "<your-event-hub-connection-string>"
              }
            }
          }
        }
    ```

## <a name="set-up-service-bus-queues-as-a-destination"></a>Configuración de colas de Service Bus como destino

Para realizar la publicación en una cola de Service Bus, configure `endpointType` como `serviceBusQueue` y proporcione:

* connectionString: cadena de conexión para la cola de Service Bus específica que se va a usar como destino generada mediante una directiva de acceso compartido.

    >[!NOTE]
    > La cadena de conexión debe ser específica de la entidad. El uso de una cadena de conexión de espacio de nombres no funcionará. Para generar una cadena de conexión específica de la entidad, en Azure Portal, vaya a la cola de Service Bus en el que desea realizar la publicación y haga clic en **Directivas de acceso compartido** para generar una cadena de conexión específica de la entidad.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "serviceBusQueue",
              "properties": {
                "connectionString": "<your-service-bus-queue-connection-string>"
              }
            }
          }
        }
    ```

## <a name="set-up-service-bus-topics-as-a-destination"></a>Configuración de temas de Service Bus como destino

Para realizar la publicación en un tema de Service Bus, configure `endpointType` como `serviceBusTopic` y proporcione:

* connectionString: cadena de conexión para el tema de Service Bus específico que se va a usar como destino generado mediante una directiva de acceso compartido.

    >[!NOTE]
    > La cadena de conexión debe ser específica de la entidad. El uso de una cadena de conexión de espacio de nombres no funcionará. Para generar una cadena de conexión específica de la entidad, en Azure Portal, vaya al tema de Service Bus en el que desea realizar la publicación y haga clic en **Directivas de acceso compartido** para generar una cadena de conexión específica de la entidad.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "serviceBusTopic",
              "properties": {
                "connectionString": "<your-service-bus-topic-connection-string>"
              }
            }
          }
        }
    ```

## <a name="set-up-storage-queues-as-a-destination"></a>Configuración de colas de almacenamiento como destino

Para realizar la publicación en una cola de almacenamiento, configure `endpointType` como `storageQueue` y proporcione:

* queueName: nombre de la cola de almacenamiento en la que va a realizar la publicación.
* connectionString: cadena de conexión de la cuenta de almacenamiento en la que se encuentra la cola de almacenamiento.

    >[!NOTE]
    > Event Hubs en línea, colas de Service Bus y temas de Service Bus: la cadena de conexión usada para las colas de almacenamiento no es específica de la entidad. En su lugar, debe ser la cadena de conexión de la cuenta de almacenamiento.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "storageQueue",
              "properties": {
                "queueName": "<your-storage-queue-name>",
                "connectionString": "<your-storage-account-connection-string>"
              }
            }
          }
        }
    ```