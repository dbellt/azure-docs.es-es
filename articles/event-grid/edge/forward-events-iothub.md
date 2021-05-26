---
title: 'Reenviar eventos de Event Grid al IoTHub: Azure Event Grid IoT Edge | Microsoft Docs'
description: Reenviar eventos de Event Grid a IoTHub
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.subservice: iot-edge
ms.date: 05/10/2021
ms.topic: article
ms.openlocfilehash: 9dd47981ba3067fefefb7a2360894a1f57ff7a64
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110368047"
---
# <a name="tutorial-forward-events-to-iothub"></a>Tutorial: reenviar eventos a IoTHub

Este artículo le guía a través de todos los pasos necesarios para reenviar eventos de Event Grid a otros módulos de IoT Edge, IoTHub con rutas. Puede querer hacerlo por los siguientes motivos:

* Siga usando las inversiones existentes ya implementadas con el enrutamiento de edgeHub
* Elija enrutar todos los eventos desde un dispositivo solo a través de IoT Hub

Para completar este tutorial, debe conocer los siguientes requisitos previos:

- [Conceptos sobre la cuadrícula de eventos](concepts.md)
- [Centro de IoT Edge](../../iot-edge/module-composition.md) 

## <a name="prerequisites"></a>Requisitos previos 
Para realizar este tutorial, necesitará lo siguiente:

* Una **suscripción a Azure**: cree una [cuenta gratuita](https://azure.microsoft.com/free) si aún no tiene una. 
* **Dispositivo Azure IoT Hub e IoT Edge**: siga los pasos del inicio rápido para dispositivos [Linux](../../iot-edge/quickstart-linux.md) o [Windows](../../iot-edge/quickstart.md) si aún no tiene uno.

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-deploy-iot-edge.md)]

## <a name="create-topic"></a>Crear tema

Como publicador de un evento, debe crear un tema de Event Grid. El tema hace referencia a un punto de conexión adonde los publicadores pueden enviar eventos.

1. Cree topic4.json con el siguiente contenido. Para más información sobre la carga, consulte nuestra [documentación de API](api.md).

   ```json
    {
          "name": "sampleTopic4",
          "properties": {
            "inputschema": "eventGridSchema"
          }
    }
    ```
1. Para crear el tema, ejecute el siguiente comando. Se devolverá el código de estado HTTP "200 - Correcto".

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @topic4.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic4?api-version=2019-01-01-preview
    ```

1. Use el siguiente comando para comprobar que se creó el tema correctamente. Se devolverá el código de estado HTTP "200 - Correcto".

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic4?api-version=2019-01-01-preview
    ```

   Salida del ejemplo:

   ```json
        [
          {
            "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic4",
            "name": "sampleTopic4",
            "type": "Microsoft.EventGrid/topics",
            "properties": {
              "endpoint": "https://<edge-vm-ip>:4438/topics/sampleTopic4/events?api-version=2019-01-01-preview",
              "inputSchema": "EventGridSchema"
            }
          }
        ]
   ```

## <a name="create-event-subscription"></a>Creación de la suscripción de eventos

Los suscriptores pueden registrarse en eventos publicados en un tema. Para recibir cualquier evento, deberán crear una suscripción de Event Grid a un tema de interés.

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-edge-persist-event-subscriptions.md)]

1. Cree subscription4.json con el contenido siguiente. Para más información sobre la carga, consulte nuestra [documentación de API](api.md).

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

   >[!NOTE]
   > El `endpointType` especifica que el suscriptor es `edgeHub`. En el `outputName` se especifica la salida en la que el módulo de Event Grid enrutará los eventos que coinciden con esta suscripción a edgeHub. Por ejemplo, los eventos que coinciden con la suscripción anterior se escribirán en `/messages/modules/eventgridmodule/outputs/sampleSub4`.
2. Ejecute el siguiente comando para crear la suscripción. Se devolverá el código de estado HTTP "200 - Correcto".

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @subscription4.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic4/eventSubscriptions/sampleSubscription4?api-version=2019-01-01-preview
    ```
3. Ejecute el siguiente comando para comprobar que la suscripción se ha creado correctamente. Se devolverá el código de estado HTTP "200 - Correcto".

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic4/eventSubscriptions/sampleSubscription4?api-version=2019-01-01-preview
    ```

    Salida del ejemplo:

   ```json
        {
          "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic4/eventSubscriptions/sampleSubscription4",
          "type": "Microsoft.EventGrid/eventSubscriptions",
          "name": "sampleSubscription4",
          "properties": {
            "Topic": "sampleTopic4",
            "destination": {
                      "endpointType": "edgeHub",
                      "properties": {
                            "outputName": "sampleSub4"
                      }
                }
          }
        }
    ```

## <a name="set-up-an-edge-hub-route"></a>Configuración de una ruta de centro de Edge

Actualice la ruta del centro de Edge para reenviar los eventos de la suscripción de eventos que se reenviarán a IoTHub como se indica a continuación:

1. Inicie sesión en el [Portal de Azure](https://ms.portal.azure.com)
1. Vaya a su instancia de **IoT Hub**.
1. En el menú, seleccione **IoT Edge**
1. Haga clic en el identificador del dispositivo de destino en la lista de dispositivos.
1. Seleccione **Set modules** (Establecer módulos).
1. Seleccione **Siguiente** para ir a la sección de rutas.
1. En las rutas, agregue una nueva ruta

  ```sh
  "fromEventGridToIoTHub":"FROM /messages/modules/eventgridmodule/outputs/sampleSub4 INTO $upstream"
  ```

  Por ejemplo,

  ```json
  {
      "routes": {
        "fromEventGridToIoTHub": "FROM /messages/modules/eventgridmodule/outputs/sampleSub4 INTO $upstream"
      }
  }
  ```

   >[!NOTE]
   > La ruta anterior reenviará los eventos coincidentes para que esta suscripción se reenvíe al centro de IoT. Puede usar las características de [enrutamiento del centro de Edge](../../iot-edge/module-composition.md) para filtrar aún más y enrutar los eventos de Event Grid a otros módulos de IoT Edge.

## <a name="setup-iot-hub-route"></a>Configuración de una ruta IoT Hub

Consulte el [Tutorial de enrutamiento de IoT Hub](../../iot-hub/tutorial-routing.md) para configurar una ruta desde el centro de IoT para que pueda ver los eventos reenviados desde el módulo de Event Grid. Use `true` para que la consulta siga simplificando el tutorial.  



## <a name="publish-an-event"></a>Publicación de un evento

1. Cree el archivo event4.json con el siguiente contenido. Para más información sobre la carga, consulte nuestra [documentación de API](api.md).

    ```json
        [
          {
            "id": "eventId-iothub-1",
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

1. Para publicar el evento, ejecute el siguiente comando:

    ```sh
    curl -k -H "Content-Type: application/json" -X POST -g -d @event4.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic4/events?api-version=2019-01-01-preview
    ```

## <a name="verify-event-delivery"></a>Comprobación de la entrega del evento

Consulte el [Tutorial de enrutamiento](../../iot-hub/tutorial-routing.md) de IoT Hub para conocer los pasos para ver los eventos.

## <a name="cleanup-resources"></a>Limpieza de recursos

* Ejecute el siguiente comando para eliminar el tema y todas sus suscripciones en el borde:

    ```sh
    curl -k -H "Content-Type: application/json" -X DELETE https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic4?api-version=2019-01-01-preview
    ```
* Elimine también los recursos creados al configurar el enrutamiento de IoTHub en la nube.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un tema de Event Grid y una suscripción de centro de Edge, además de haber publicado eventos. Ahora que conoce los pasos básicos para desviar a un centro de Edge, consulte los siguientes artículos:

* Para solucionar problemas relacionados con el uso de Azure Event Grid en IoT Edge, consulte la [Guía de solución de problemas](troubleshoot.md).
* Use los filtros de enrutamiento de [centro de Edge](../../iot-edge/module-composition.md) para particionar eventos
* Configure la persistencia del módulo de Event Grid en [Linux](persist-state-linux.md) o [Windows](persist-state-windows.md)
* Siga la [documentación](configure-client-auth.md) para configurar la autenticación de cliente
* Reenvíe eventos a Azure Event Grid en la nube siguiendo este [tutorial](forward-events-event-grid-cloud.md)
* [Supervisión de temas y suscripciones en el perímetro](monitor-topics-subscriptions.md)