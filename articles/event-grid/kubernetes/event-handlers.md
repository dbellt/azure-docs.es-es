---
title: 'Azure Event Grid en Kubernetes: controladores de eventos y destinos'
description: En este artículo se describen los distintos tipos de controladores de eventos y destinos admitidos por Event Grid en Kubernetes.
author: jfggdl
ms.author: jafernan
ms.subservice: kubernetes
ms.date: 05/25/2021
ms.topic: conceptual
ms.openlocfilehash: 838084204ed2c1979f618bb2bfe644d1f88cd51e
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110536505"
---
# <a name="event-handlers-destinations-in-event-grid-on-kubernetes"></a>Destinos de controladores de eventos en Event Grid en Kubernetes
Un controlador de eventos es cualquier sistema que expone un punto de conexión y es el destino de los eventos enviados por Event Grid. Un controlador de eventos que recibe un evento actúa sobre él y usa la carga del evento para ejecutar alguna lógica, lo que podría derivar en la aparición de nuevos eventos.

La manera de configurar Event Grid para enviar eventos a un destino es mediante la creación de una suscripción de eventos. Se puede realizar con la [CLI de Azure](/cli/azure/eventgrid/event-subscription#az_eventgrid_event_subscription_create), el [SDK de administración](../sdk-overview.md#management-sdks) o las llamadas HTTPS directas mediante la versión [2020-10-15-preview API](/rest/api/eventgrid/version2020-10-15-preview/eventsubscriptions/createorupdate).

En general, Event Grid en Kubernetes puede enviar eventos a cualquier destino mediante **webhooks**. Los webhooks son puntos de conexión HTTP(S) expuestos por un servicio o una carga de trabajo a los que Event Grid tiene acceso. El webhook puede ser una carga de trabajo hospedada en el mismo clúster, en el mismo espacio de red, en la nube, localmente o en cualquier lugar al que Event Grid puede acceder. 

[!INCLUDE [event-grid-preview-feature-note.md](../../../includes/event-grid-preview-feature-note.md)]

Con webhooks, Event Grid admite los siguientes destinos **hospedados en un clúster de Kubernetes**:

* Azure App Service en Kubernetes con Azure Arc 
* Azure Functions en Kubernetes con Azure Arc 
* Azure Logic Apps en Kubernetes con Azure Arc

Además de los webhooks, Event Grid en Kubernetes puede enviar eventos a los siguientes destinos **hospedados en Azure**:

- Azure Event Grid **con webhooks**
- Azure Functions **con webhooks solamente**
- Azure Event Hubs con su identificador de recursos de Azure Resource Manager
- Temas o colas de Azure Service Bus con su identificador de recursos de Azure Resource Manager
- Cola de Azure Storage con su identificador de recursos de Azure Resource Manager



## <a name="feature-parity"></a>Paridad de características
Event Grid en Kubernetes ofrece un buen nivel de paridad de características mediante la compatibilidad de Azure Event Grid con las suscripciones de eventos. En la lista siguiente se enumeran las principales diferencias en la funcionalidad de suscripción de eventos. Además de tales diferencias, puede usar la [versión de la API de REST 2020-10-15-preview](/rest/api/eventgrid/version2020-10-15-preview/eventsubscriptions) de Azure Event Grid como referencia al administrar las suscripciones de eventos en Event Grid en Kubernetes.

1. Use la [versión de la API de REST 2020-10-15-preview](/rest/api/eventgrid/version2020-10-15-preview/eventsubscriptions).
2. El [desencadenador de Azure Event Grid para Azure Functions](../../azure-functions/functions-bindings-event-grid-trigger.md?tabs=csharp%2Cconsole) no es compatible. Puede usar un tipo de destino de webhook para entregar eventos a Azure Functions.
3. No hay compatibilidad con la [ubicación de mensajes fallidos](../manage-event-delivery.md#set-dead-letter-location). Eso significa que no puede usar ``properties.deadLetterDestination`` en la carga de la suscripción de eventos.
4. Las conexiones híbridas de Azure Relay como destino aún no son compatibles.
5. Solo se admite el esquema CloudEvents. El valor de esquema admitido es "[CloudEventSchemaV1_0](/rest/api/eventgrid/version2020-10-15-preview/eventsubscriptions/createorupdate#eventdeliveryschema)". El esquema de eventos en la nube es extensible y se basa en estándares abiertos.  
6. Las etiquetas ([properties.labels](/rest/api/eventgrid/version2020-10-15-preview/eventsubscriptions/createorupdate#request-body)) no son aplicables a Event Grid en Kubernetes. Por lo tanto, no están disponibles.
7. No se admite la [entrega con un identidad de recurso](/rest/api/eventgrid/version2020-10-15-preview/eventsubscriptions/createorupdate#deliverywithresourceidentity). Por lo tanto, no se admiten todas las propiedades de la [identidad de la suscripción de eventos](/rest/api/eventgrid/version2020-10-15-preview/eventsubscriptions/createorupdate#eventsubscriptionidentity).
8. Todavía no se admite la [validación del punto de conexión de destino](../webhook-event-delivery.md#endpoint-validation-with-event-grid-events).

## <a name="event-filtering-in-event-subscriptions"></a>Filtrado de eventos en suscripciones de eventos
El otro aspecto importante de la configuración de una suscripción de eventos es seleccionar los eventos que están diseñados para entregarse a un destino. Para más información, vea [Filtrado de eventos](filter-events.md).

## <a name="sample-destination-configurations"></a>Configuraciones de destino de ejemplo

A continuación, se encuentran algunas configuraciones de ejemplo básicas en función del destino previsto.

## <a name="webhook"></a>WebHook
Para publicar en un punto de conexión de webhook, establezca `endpointType` en `WebHook` y proporcione:

* **endpointUrl**: la dirección URL del punto de conexión del webhook.

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

Para publicar en un punto de conexión de nube de Azure Event Grid, establezca `endpointType` en `WebHook` y proporcione:

* **endpointUrl**: dirección URL del tema de Azure Event Grid en la nube con el parámetro de versión de la API establecido en **2018-01-01** y `aeg-sas-key` establecido en la clave SAS codificada en la URL. 

   ```json
    {
        "properties": {
            "destination": {
                "endpointType": "WebHook",
                "properties": {
                    "endpointUrl": "<your-event-grid-cloud-topic-endpoint-url>?api-version=2018-01-01&aeg-sas-key=urlencoded(sas-key-value)"
                }
            }
        }
    }
   ```

## <a name="event-hubs"></a>Event Hubs

Para realizar la publicación en un centro de eventos, configure `endpointType` como `eventHub` y proporcione:

* **resourceId**: identificador de recurso del centro de eventos específico.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "eventHub",
              "properties": {
                "resourceId": "<Azure Resource ID of your event hub>"
              }
            }
          }
        }
    ```

## <a name="service-bus-queues"></a>Colas de Service Bus

Para realizar la publicación en una cola de Service Bus, configure `endpointType` como `serviceBusQueue` y proporcione:

* **resourceId**: identificador de recurso de la cola de Service Bus específica.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "serviceBusQueue",
              "properties": {
                "resourceId": "<Azure Resource ID of your Service Bus queue>"
              }
            }
          }
        }
    ```

## <a name="service-bus-topics"></a>Temas de Service Bus

Para realizar la publicación en un tema de Service Bus, configure `endpointType` como `serviceBusTopic` y proporcione:

* **resourceId**: identificador de recurso del tema de Service Bus específico.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "serviceBusTopic",
              "properties": {
                "resourceId": "<Azure Resource ID of your Service Bus topic>"
              }
            }
          }
        }
    ```

## <a name="storage-queues"></a>Colas de almacenamiento

Para realizar la publicación en una cola de almacenamiento, configure `endpointType` como `storageQueue` y proporcione:

* **queueName**: nombre de la cola de Azure Storage en la que va a realizar la publicación.
* **resourceID**: identificador de recurso de Azure de la cuenta de almacenamiento que contiene la cola.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "storageQueue",
              "properties": {
                "queueName": "<your-storage-queue-name>",
                "resourceId": "<Azure Resource ID of your Storage account>"
              }
            }
          }
        }
    ```

## <a name="next-steps"></a>Pasos siguientes
* Agregue la [configuración de filtro](filter-events.md) a la suscripción de eventos para seleccionar los eventos que se van a entregar. 
* Para obtener información sobre los esquemas admitidos por Event Grid en Azure Arc para Kubernetes, vea [Event Grid en Kubernetes - Esquemas de eventos](event-schemas.md).