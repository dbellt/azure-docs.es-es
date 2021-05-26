---
title: 'Controladores de eventos y destinos: Azure Event Grid IoT Edge | Microsoft Docs'
description: Controladores de eventos y destinos en Event Grid en Microsoft Edge
ms.subservice: iot-edge
ms.date: 05/10/2021
ms.topic: article
ms.openlocfilehash: b449368a94da95727729fe6100915e55ce106ecb
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110370452"
---
# <a name="event-handlers-and-destinations-in-event-grid-on-edge"></a>Controladores de eventos y destinos en Event Grid en Microsoft Edge

Un controlador de eventos es el lugar donde se guarda el evento para realizar acciones posteriores o para procesarse. Con el módulo de Event Grid en Microsoft Edge, el controlador de eventos puede estar en el mismo dispositivo Edge, en otro dispositivo o en la nube. Puede usar cualquier webhook para controlar eventos o para enviarlos a uno de los controladores nativos como Azure Event Grid.

En este artículo se proporciona información sobre cómo configurar cada caso.

## <a name="webhook"></a>WebHook

Para publicar en un punto de conexión de webhook, establezca `endpointType` en `WebHook` y proporcione:

* endpointUrl: La dirección URL del punto de conexión del webhook

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "<your-webhook-endpoint>"
              }
            }
          }
        }
    ```

## <a name="azure-event-grid"></a>Azure Event Grid

Para publicar en un punto de conexión de nube de Azure Event Grid, establezca `endpointType` en `eventGrid` y proporcione:

* endpointUrl: dirección URL del tema de Event Grid en la nube
* sasKey: clave de SAS del tema de Event Grid
* topicName: nombre para marcar todos los eventos salientes en Event Grid. El nombre del tema es útil cuando se publica en un tema de dominio de Event Grid.

   ```json
        {
          "properties": {
            "destination": {
              "endpointType": "eventGrid",
              "properties": {
                 "endpointUrl": "<your-event-grid-cloud-topic-endpoint-url>?api-version=2018-01-01",
                 "sasKey": "<your-event-grid-topic-saskey>",
                 "topicName": null
              }
            }
          }
    }
   ```

## <a name="iot-edge-hub"></a>Centro de IoT Edge

Para realizar la publicación en un centro de IoT Edge, configure `endpointType` como `edgeHub` y proporcione:

* outputName: la salida en la que el módulo de Event Grid enrutará los eventos que coinciden con esta suscripción a edgeHub. Por ejemplo, los eventos que coinciden con la suscripción siguiente se escribirán en /messages/modules/eventgridmodule/outputs/sampleSub4.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "edgeHub",
              "properties": {
                "outputName": "sampleSub4"
              }
            }
          }
        }
    ```

## <a name="event-hubs"></a>Event Hubs

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

## <a name="service-bus-queues"></a>Colas de Service Bus

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

## <a name="service-bus-topics"></a>Temas de Service Bus

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

## <a name="storage-queues"></a>Colas de almacenamiento

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