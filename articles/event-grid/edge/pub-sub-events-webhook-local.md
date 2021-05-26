---
title: 'Publicación de eventos y suscripción a ellos de forma local: Azure Event Grid en IoT Edge | Microsoft Docs'
description: Publicación de eventos y suscripción a ellos de forma local mediante webhook con Event Grid en IoT Edge
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.subservice: iot-edge
ms.date: 05/10/2021
ms.topic: article
ms.openlocfilehash: 4080fc274f03cbc4e5ba48b3c3e2ac5699886c96
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110377215"
---
# <a name="tutorial-publish-subscribe-to-events-locally"></a>Tutorial: Publicación de eventos y suscripción a ellos de forma local

Este artículo le guía por todos los pasos necesarios para publicar eventos y suscribirse a ellos mediante Event Grid en IoT Edge.

> [!NOTE]
> Para información sobre los temas y las suscripciones de Azure Event Grid, consulte [Conceptos de Event Grid](concepts.md).

## <a name="prerequisites"></a>Prerrequisitos 
Para realizar este tutorial, necesitará lo siguiente:

* Una **suscripción a Azure**: cree una [cuenta gratuita](https://azure.microsoft.com/free) si aún no tiene una. 
* **Dispositivo Azure IoT Hub e IoT Edge**: siga los pasos del inicio rápido para dispositivos [Linux](../../iot-edge/quickstart-linux.md) o [Windows](../../iot-edge/quickstart.md) si aún no tiene uno.

## <a name="deploy-event-grid-iot-edge-module"></a>Implementación del módulo de Event Grid en IoT Edge

Hay varias maneras de implementar módulos en un dispositivo IoT Edge, y todas ellas funcionan con Azure Event Grid en IoT Edge. En este artículo se describen los pasos para implementar Event Grid en IoT Edge desde Azure Portal.

>[!NOTE]
> En este tutorial, implementará el módulo de Event Grid sin persistencia. Esto significa que los temas y las suscripciones que cree en este tutorial se eliminarán al volver a implementar el módulo. Para más información sobre cómo configurar la persistencia, consulte los artículos siguientes: [Persistencia del estado en Linux](persist-state-linux.md) o [Persistencia del estado en Windows](persist-state-windows.md). En el caso de las cargas de trabajo de producción, se recomienda instalar el módulo de Event Grid con persistencia.


### <a name="select-your-iot-edge-device"></a>Selección del dispositivo IoT Edge

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com)
1. Vaya a su instancia de IoT Hub.
1. Seleccione **IoT Edge** en el menú de la sección **Administración automática de dispositivos**. 
1. Haga clic en el identificador del dispositivo de destino en la lista de dispositivos.
1. Seleccione **Set modules** (Establecer módulos). Mantenga la página abierta. Continuará con los pasos de la siguiente sección.

### <a name="configure-a-deployment-manifest"></a>Configuración de un manifiesto de implementación

Un manifiesto de implementación es un documento JSON que describe qué módulos se van a implementar, cómo fluyen los datos entre los módulos y las propiedades deseadas de los módulos gemelos. Azure Portal tiene un asistente que le guía en la creación de un manifiesto de implementación, en lugar de crear el documento JSON de forma manual.  Consta de tres pasos: **Adición de módulos**, **Especificación de rutas** y **Revisión de la implementación**.

### <a name="add-modules"></a>Adición de módulos

1. En la sección **Módulos de implementación**, seleccione **Agregar**.
1. En los tipos de módulos de la lista desplegable, seleccione **Módulo IoT Edge**
1. Proporcione el nombre, la imagen y las opciones de creación del contenedor:

   * **Nombre**: eventgridmodule
   * **URI de imagen**: `mcr.microsoft.com/azure-event-grid/iotedge:latest`
   * **Opciones de creación del contenedor**:

   [!INCLUDE [event-grid-edge-module-version-update](../../../includes/event-grid-edge-module-version-update.md)]

    ```json
        {
          "Env": [
            "inbound__clientAuth__clientCert__enabled=false"
          ],
          "HostConfig": {
            "PortBindings": {
              "4438/tcp": [
                {
                    "HostPort": "4438"
                }
              ]
            }
          }
        }
    ```    
 1. Haga clic en **Guardar**
 1. Continúe con la sección siguiente para agregar el módulo de suscriptor de Azure Event Grid antes de implementarlos juntos.

    >[!IMPORTANT]
    > En este tutorial, implementará el módulo de Event Grid con la autenticación de cliente deshabilitada. En el caso de las cargas de trabajo de producción, se recomienda habilitar la autenticación de cliente. Para más información sobre cómo configurar el módulo de Event Grid de forma segura, consulte [Seguridad y autenticación](security-authentication.md).
    > 
    > Si va a usar una máquina virtual de Azure como dispositivo perimetral, agregue una regla de puerto de entrada para permitir tráfico entrante en el puerto 4438. Para instrucciones sobre cómo agregar la regla, consulte [Apertura de puertos en una máquina virtual](../../virtual-machines/windows/nsg-quickstart-portal.md).
    

## <a name="deploy-event-grid-subscriber-iot-edge-module"></a>Implementación del módulo IoT Edge de suscriptor de Event Grid

En esta sección se muestra cómo implementar otro módulo de IoT que funcionará como controlador de eventos al que se pueden entregar eventos.

### <a name="add-modules"></a>Adición de módulos

1. En la sección **Módulos de implementación**, seleccione de nuevo **Agregar**. 
1. En los tipos de módulos de la lista desplegable, seleccione **Módulo IoT Edge**
1. Proporcione el nombre, la imagen y las opciones de creación del contenedor:

   * **Nombre**: suscriptor
   * **URI de imagen**: `mcr.microsoft.com/azure-event-grid/iotedge-samplesubscriber:latest`
   * **Opciones de creación del contenedor**: None
1. Haga clic en **Guardar**
1. Seleccione **Siguiente** para ir a la sección de rutas

 ### <a name="setup-routes"></a>Configuración de rutas

Mantenga las rutas predeterminadas y seleccione **Siguiente** para pasar a la sección de revisión.

### <a name="submit-the-deployment-request"></a>Envío de la solicitud de implementación

1. La sección de revisión le muestra el manifiesto de implementación en formato JSON que se ha creado en función de las selecciones de la sección anterior. Confirme que ve los módulos **eventgridmodule** y **subscriber** enumerados en el archivo JSON. 
1. Revise la información de implementación y seleccione **Enviar**. Después de enviar la implementación, regresará a la página del **dispositivo**.
1. En la sección **Módulos**, compruebe que aparecen los módulos **eventgrid** y **subscriber**. Además, compruebe que las columnas **Especificado en la implementación** y **Notificado por el dispositivo** están establecidas en **Sí**.

    Puede tardar unos minutos para que el módulo se inicie en el dispositivo y, a continuación, notifique a IoT Hub. Actualice la página para ver el estado actualizado.

## <a name="create-a-topic"></a>de un tema

Como publicador de un evento, debe crear un tema de Event Grid. En Azure Event Grid, un tema hace referencia a un punto de conexión al que los publicadores pueden enviar eventos.

1. Cree topic.json con el siguiente contenido. Para más información sobre la carga, consulte nuestra [documentación de API](api.md).

    ```json
        {
          "name": "sampleTopic1",
          "properties": {
            "inputschema": "eventGridSchema"
          }
        }
    ```

1. Ejecute el comando siguiente para crear un tema de Event Grid. Confirme que se muestra el código de estado HTTP `200 OK`.

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @topic.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic1?api-version=2019-01-01-preview
    ```

1. Use el siguiente comando para comprobar que se creó el tema correctamente. Se devolverá el código de estado HTTP "200 - Correcto".

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic1?api-version=2019-01-01-preview
    ```

   Salida del ejemplo:

   ```json
        [
          {
            "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic1",
            "name": "sampleTopic1",
            "type": "Microsoft.EventGrid/topics",
            "properties": {
              "endpoint": "https://<edge-vm-ip>:4438/topics/sampleTopic1/events?api-version=2019-01-01-preview",
              "inputSchema": "EventGridSchema"
            }
          }
        ]
   ```

## <a name="create-an-event-subscription"></a>Creación de una suscripción a evento

Los suscriptores pueden registrarse en eventos publicados en un tema. Para recibir cualquier evento, deberá crear una suscripción de Event Grid para un tema de interés.

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-edge-persist-event-subscriptions.md)]

1. Cree el archivo subscription.json con el siguiente contenido. Para más información sobre la carga, consulte la [Documentación de API](api.md)

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "https://subscriber:4430"
              }
            }
          }
        }
    ```

    >[!NOTE]
    > La propiedad **endpointType** especifica que el suscriptor es un **webhook**.  La propiedad **endpointUrl** especifica la dirección URL en la que el suscriptor escucha los eventos. Esta dirección URL corresponde al ejemplo de suscriptor de Azure que implementó anteriormente.
2. Ejecute el siguiente comando para crear una suscripción para el tema. Confirme que se muestra el código de estado HTTP `200 OK`.

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @subscription.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic1/eventSubscriptions/sampleSubscription1?api-version=2019-01-01-preview
    ```
3. Ejecute el siguiente comando para comprobar que la suscripción se ha creado correctamente. Se devolverá el código de estado HTTP "200 - Correcto".

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic1/eventSubscriptions/sampleSubscription1?api-version=2019-01-01-preview
    ```

    Salida del ejemplo:

   ```json
        {
          "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic1/eventSubscriptions/sampleSubscription1",
          "type": "Microsoft.EventGrid/eventSubscriptions",
          "name": "sampleSubscription1",
          "properties": {
            "Topic": "sampleTopic1",
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "https://subscriber:4430"
              }
            }
          }
        }
    ```

## <a name="publish-an-event"></a>Publicación de un evento

1. Cree el archivo event.json con el siguiente contenido. Para más información sobre la carga, consulte nuestra [documentación de API](api.md).

    ```json
        [
          {
            "id": "eventId-func-0",
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
1. Ejecute el siguiente comando para publicar un evento.

    ```sh
    curl -k -H "Content-Type: application/json" -X POST -g -d @event.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic1/events?api-version=2019-01-01-preview
    ```

## <a name="verify-event-delivery"></a>Comprobación de la entrega del evento

1. Use el protocolo SSH o RDP en la máquina virtual IoT Edge.
1. Compruebe los registros del suscriptor:

    En Windows, ejecute el siguiente comando:

    ```sh
    docker -H npipe:////./pipe/iotedge_moby_engine container logs subscriber
    ```

   En Linux, ejecute el siguiente comando:

    ```sh
    sudo docker logs subscriber
    ```

    Salida del ejemplo:

    ```sh
        Received Event:
            {
              "id": "eventId-func-0",
              "topic": "sampleTopic1",
              "subject": "myapp/vehicles/motorcycles",
              "eventType": "recordInserted",
              "eventTime": "2019-07-28T21:03:07+00:00",
              "dataVersion": "1.0",
              "metadataVersion": "1",
              "data": {
                "make": "Ducati",
                "model": "Monster"
              }
            }
    ```

## <a name="cleanup-resources"></a>Limpieza de recursos

* Ejecute el siguiente comando para eliminar el tema y todas sus suscripciones.

    ```sh
    curl -k -H "Content-Type: application/json" -X DELETE https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic1?api-version=2019-01-01-preview
    ```
* Elimine el módulo de suscriptor del dispositivo IoT Edge.


## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha creado un tema y una suscripción de Event Grid y ha publicado eventos. Ahora que conoce los pasos básicos, consulte los siguientes artículos: 

- Para solucionar problemas relacionados con el uso de Azure Event Grid en IoT Edge, consulte la [Guía de solución de problemas](troubleshoot.md).
- Cree o actualice una suscripción con [filtros](advanced-filtering.md).
- Habilite la persistencia del módulo de Event Grid en [Linux](persist-state-linux.md) o [Windows](persist-state-windows.md).
- Siga la [documentación](configure-client-auth.md) para configurar la autenticación de cliente
- Reenvíe eventos a Azure Functions en la nube siguiendo este [tutorial](pub-sub-events-webhook-cloud.md).
- [Reacción a eventos de Blob Storage en IoT Edge](react-blob-storage-events-locally.md)
- [Supervisión de temas y suscripciones en el perímetro](monitor-topics-subscriptions.md)

