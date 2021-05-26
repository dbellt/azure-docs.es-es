---
title: 'Módulo gemelo: Azure Event Grid IoT Edge | Microsoft Docs'
description: Configuración a través de módulo gemelo.
author: HiteshMadan
manager: rajarv
ms.author: himad
ms.reviewer: spelluru
ms.subservice: iot-edge
ms.date: 05/10/2021
ms.topic: article
ms.openlocfilehash: 27e5194b86ea5f76a346412c3239dac111c07d80
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110379789"
---
# <a name="module-twin-json-schema-azure-event-grid"></a>Esquema JSON de módulo gemelo (Azure Event Grid)

Event Grid en IoT Edge se integra con el ecosistema de IoT Edge y admite la creación de temas y suscripciones a través del módulo gemelo. También informa del estado actual de todos los temas y suscripciones de eventos a las propiedades notificadas en el módulo gemelo.

> [!WARNING]
> Debido a las limitaciones del ecosistema IoT Edge, todos los elementos de la matriz del siguiente ejemplo de JSON se han codificado como cadenas JSON. Consulte las claves `EventSubscription.Filter.EventTypes` y `EventSubscription.Filter.AdvancedFilters` en el ejemplo siguiente.

## <a name="desired-properties-json"></a>Propiedades JSON deseadas

* El valor de cada par clave-valor de la sección de temas tiene exactamente el mismo esquema JSON que se usó para `Topic.Properties` en la API al crear temas.
* El valor de cada par clave-valor de la sección **EventSubscriptions** tiene exactamente el mismo esquema JSON que se usó para `EventSubscription.Properties` en la API al crear temas.
* Para eliminar un tema, establezca su valor en `null` en las propiedades deseadas.
* No se admite la eliminación de suscripciones de eventos a través de las propiedades deseadas.

```json
{
    "topics": {
        "twinTopic1": {},
        "twinTopic2": {
            "inputSchema": "customEventSchema"
        }
    },
    "eventSubscriptions": {
        "twinTopic1WebhookSub": {
            "topic": "twinTopic1",
            "retryPolicy": {
                "eventExpiryInMinutes": 120,
                "maxDeliveryAttempts": 30
            },
            "destination": {
                "endpointType": "WebHook",
                "properties": {
                    "endpointUrl": "https://localhost:4438"
                }
            },
            "filter": {
                "subjectBeginsWith": "^",
                "subjectEndsWith": "$",
                "isSubjectCaseSensitive": false,
                "includedEventTypes": "[\"et1\",\"et2\"]",
                "advancedFilters": "[{\"value\":true,\"operatorType\":\"BoolEquals\",\"key\":\"data.b\"},{\"values\":[\"\\\"\",\"c\"],    \"operatorType\":\"StringContains\",\"key\":\"data.s\"}]"
            }
        },
        "twinTopic2EdgeHubSub": {
            "topic": "twinTopic2",
            "deliveryPolicy": {
                "approxBatchSizeInBytes": 200000,
                "maxEventsPerBatch": 25
            },
            "destination": {
                "endpointType": "EdgeHub",
                "properties": {
                    "outputName": "twinTopic2EdgeHubSub"
                }
            },
            "filter": {
                "advancedFilters": "[{\"value\":true,\"operatorType\":\"BoolEquals\",\"key\":\"dAt\\\"A.a\"},{\"values\":[\"\\\"\",    \"c\"],\"operatorType\":\"StringContains\",\"key\":\"dAt\\\"A.a\"}]"
            }
        }
    }
}
```

## <a name="reported-properties-json"></a>JSON de propiedades notificadas

En la sección de propiedades notificadas del módulo gemelo se incluye la siguiente información:

* Conjunto de temas y suscripciones que existen en el almacén del módulo
* Errores encontrados al crear suscripciones a eventos o temas deseados
* Errores de arranque (como errores en el análisis en el JSON de las propiedades deseadas)

```json
{
    "topics": {
        "twinTopic1": {},
        "twinTopic2": {
            "inputSchema": "customEventSchema"
        }
    },
    "eventSubscriptions": {
        "twinTopic1WebhookSub": {
            "topic": "twinTopic1",
            "retryPolicy": {
                "eventExpiryInMinutes": 120,
                "maxDeliveryAttempts": 30
            },
            "destination": {
                "endpointType": "WebHook",
                "properties": {
                    "endpointUrl": "https://localhost:4438"
                }
            },
            "filter": {
                "subjectBeginsWith": "^",
                "subjectEndsWith": "$",
                "isSubjectCaseSensitive": false,
                "includedEventTypes": "[\"et1\",\"et2\"]",
                "advancedFilters": "[{\"value\":true,\"operatorType\":\"BoolEquals\",\"key\":\"data.b\"},{\"values\":[\"\\\"\",\"c\"],    \"operatorType\":\"StringContains\",\"key\":\"data.s\"}]"
            }
        },
        "twinTopic2EdgeHubSub": {
            "topic": "twinTopic2",
            "deliveryPolicy": {
                "approxBatchSizeInBytes": 200000,
                "maxEventsPerBatch": 25
            },
            "destination": {
                "endpointType": "EdgeHub",
                "properties": {
                    "outputName": "twinTopic2EdgeHubSub"
                }
            },
            "filter": {
                "advancedFilters": "[{\"value\":true,\"operatorType\":\"BoolEquals\",\"key\":\"dAt\\\"A.a\"},{\"values\":[\"\\\"\",    \"c\"],\"operatorType\":\"StringContains\",\"key\":\"dAt\\\"A.a\"}]"
            }
        }
    },
    "errors": {
        "bootupMessage": "",
        "bootupException": "",
        "topics": {},
        "eventSubscriptions": {
            "twinTopic1EventGridUserTopicSub": "HttpStatusCode='BadRequest' ErrorCode='InvalidDestination' Message='EventSubscription.Properties.Destination failed validation. Reason: EndpointUrl must target the /api/events API of Azure Event Grid in the cloud..'"
        }
    }
}
```
