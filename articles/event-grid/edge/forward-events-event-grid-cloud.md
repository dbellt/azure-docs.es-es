---
title: 'Reenviar eventos de Event a la nube de Event Grid: Azure Event Grid IoT Edge | Microsoft Docs'
description: Reenviar eventos perimetrales a la nube de Event Grid
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.subservice: iot-edge
ms.date: 05/10/2021
ms.topic: article
ms.openlocfilehash: 5a35cfb293ca7cda0a9a1d24f4faf88ef0ed733d
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110378324"
---
# <a name="tutorial-forward-events-to-event-grid-cloud"></a>Tutorial: Reenvío de eventos a la nube de Event Grid

Este artículo le guía a través de todos los pasos necesarios para reenviar eventos perimetrales a Event Grid en la nube de Azure. Puede querer hacerlo por los siguientes motivos:

* Reaccione a los eventos perimetrales en la nube.
* Reenvíe eventos a Event Grid en la nube y use las colas de Event Hubs o Azure Storage de Azure para almacenar en búfer los eventos antes de procesarlos en la nube.

 Para completar este tutorial, necesita comprender los conceptos de Event Grid sobre [perimetral](concepts.md) y de [Azure](../concepts.md). Para obtener más tipos de destino, consulte [Controladores de eventos](event-handlers.md). 

## <a name="prerequisites"></a>Requisitos previos 
Para realizar este tutorial, necesitará lo siguiente:

* Una **suscripción a Azure**: cree una [cuenta gratuita](https://azure.microsoft.com/free) si aún no tiene una. 
* **Dispositivo Azure IoT Hub e IoT Edge**: siga los pasos del inicio rápido para dispositivos [Linux](../../iot-edge/quickstart-linux.md) o [Windows](../../iot-edge/quickstart.md) si aún no tiene uno.

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-deploy-iot-edge.md)] 
## <a name="create-event-grid-topic-and-subscription-in-cloud"></a>Creación de un tema y una suscripción de Event Grid en la nube

Cree un tema y una suscripción de Event Grid en la nube siguiendo [este tutorial](../custom-event-quickstart-portal.md). Anote `topicURL`, `sasKey`y `topicName` del tema recién creado que usará más adelante en el tutorial.

Por ejemplo, si creó un tema denominado `testegcloudtopic` en el oeste de EE. UU., los valores tendrían un aspecto similar al siguiente:

* **TopicUrl**: `https://testegcloudtopic.westus2-1.eventgrid.azure.net/api/events`
* **TopicName**: `testegcloudtopic`
* **SasKey**: disponible en **AccessKey** del tema. Use **Key1**.

## <a name="create-event-grid-topic-at-the-edge"></a>Creación de un tema de Event Grid en el perímetro

1. Cree topic3.json con el siguiente contenido. Para más información sobre la carga, consulte nuestra [documentación de API](api.md).

    ```json
        {
          "name": "sampleTopic3",
          "properties": {
            "inputschema": "eventGridSchema"
          }
        }
    ```
1. Para crear el tema, ejecute el siguiente comando. Se devolverá el código de estado HTTP "200 - Correcto".

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @topic3.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic3?api-version=2019-01-01-preview
    ```
1. Use el siguiente comando para comprobar que se creó el tema correctamente. Se devolverá el código de estado HTTP "200 - Correcto".

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic3?api-version=2019-01-01-preview
    ```

   Salida del ejemplo:

   ```json
        [
          {
            "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic3",
            "name": "sampleTopic3",
            "type": "Microsoft.EventGrid/topics",
            "properties": {
              "endpoint": "https://<edge-vm-ip>:4438/topics/sampleTopic3/events?api-version=2019-01-01-preview",
              "inputSchema": "EventGridSchema"
            }
          }
        ]
   ```
  
## <a name="create-event-grid-subscription-at-the-edge"></a>Creación de una suscripción de Event Grid en el perímetro

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-edge-persist-event-subscriptions.md)]

1. Cree subscription3.json con el siguiente contenido. Para más información sobre la carga, consulte nuestra [documentación de API](api.md).

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

   >[!NOTE]
   > El **endpointUrl** especifica la dirección URL del tema Event Grid en la nube. La **sasKey** hace referencia a la clave del tema de la nube de Event Grid. El valor de **topicName** se usará para marcar todos los eventos salientes en Event Grid. Esto puede ser útil al publicar en un tema de dominio de Event Grid. Para obtener más información acerca del tema del dominio de Event Grid, consulte [Dominios de eventos](../event-domains.md)

    Por ejemplo,
  
    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "eventGrid",
              "properties": {
                "endpointUrl": "https://testegcloudtopic.westus2-1.eventgrid.azure.net/api/events?api-version=2018-01-01",
                 "sasKey": "<your-event-grid-topic-saskey>",
                 "topicName": null
              }
            }
          }
        }
    ```

2. Ejecute el siguiente comando para crear la suscripción. Se devolverá el código de estado HTTP "200 - Correcto".

     ```sh
     curl -k -H "Content-Type: application/json" -X PUT -g -d @subscription3.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic3/eventSubscriptions/sampleSubscription3?api-version=2019-01-01-preview
     ```

3. Ejecute el siguiente comando para comprobar que la suscripción se ha creado correctamente. Se devolverá el código de estado HTTP "200 - Correcto".

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic3/eventSubscriptions/sampleSubscription3?api-version=2019-01-01-preview
    ```

    Salida del ejemplo:

    ```json
         {
          "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic3/eventSubscriptions/sampleSubscription3",
          "type": "Microsoft.EventGrid/eventSubscriptions",
          "name": "sampleSubscription3",
          "properties": {
            "Topic": "sampleTopic3",
            "destination": {
              "endpointType": "eventGrid",
              "properties": {
                "endpointUrl": "https://testegcloudtopic.westus2-1.eventgrid.azure.net/api/events?api-version=2018-01-01",
                "sasKey": "<your-event-grid-topic-saskey>",
                "topicName": null
              }
            }
          }
        }
    ```

## <a name="publish-an-event-at-the-edge"></a>Publicación de un evento en el perímetro

1. Cree el archivo event3.json con el siguiente contenido. Para más información sobre la carga, consulte la [documentación de API](api.md).

    ```json
        [
          {
            "id": "eventId-egcloud-0",
            "eventType": "recordInserted",
            "subject": "myapp/vehicles/motorcycles",
            "eventTime": "2019-07-28T21:03:07+00:00",
            "dataVersion": "1.0",
            "data": {
              "make": "Ducati",
              "model": "Monster"
            }
          }
        ]
    ```

1. Ejecute el siguiente comando:

    ```sh
    curl -k -H "Content-Type: application/json" -X POST -g -d @event3.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic3/events?api-version=2019-01-01-preview
    ```

## <a name="verify-edge-event-in-cloud"></a>Comprobación del evento perimetral en la nube

Para obtener información sobre cómo ver los eventos que se proporcionan en el tema de la nube, vea el [tutorial](../custom-event-quickstart-portal.md).

## <a name="cleanup-resources"></a>Limpieza de recursos

* Ejecute el siguiente comando para eliminar el tema y todas sus suscripciones

    ```sh
    curl -k -H "Content-Type: application/json" -X DELETE https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic3?api-version=2019-01-01-preview
    ```

* Elimine también el tema y las suscripciones creadas en la nube (Azure Event Grid).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha publicado un evento en el perímetro y lo ha reenviado a Event Grid en la nube de Azure. Ahora que conoce los pasos básicos para reenviar a Event Grid en la nube:

* Para solucionar problemas relacionados con el uso de Azure Event Grid en IoT Edge, consulte la [Guía de solución de problemas](troubleshoot.md).
* Reenvíe eventos a IoTHub siguiendo este [tutorial](forward-events-iothub.md)
* Reenvíe eventos a Webhook en la nube siguiendo este [tutorial](pub-sub-events-webhook-cloud.md)
* [Supervisión de temas y suscripciones en el perímetro](monitor-topics-subscriptions.md)
