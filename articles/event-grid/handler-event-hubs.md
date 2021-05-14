---
title: Event Hub como controlador de eventos para eventos de Azure Event Grid
description: Aquí se describe cómo puede usar los centros de eventos como controladores de eventos para eventos de Event Grid.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: b69e06335b4461ad9ee72f32d38e7e7f7056febc
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109734817"
---
# <a name="event-hub-as-an-event-handler-for-azure-event-grid-events"></a>Centro de eventos como controlador de eventos para eventos de Azure Event Grid
Un controlador de eventos es el lugar al que se envía el evento. El controlador realiza una acción para procesar el evento. Varios servicios de Azure se configuran automáticamente para controlar los eventos y **Azure Event Hubs** es uno de ellos. 

Use **Event Hubs** cuando la solución obtenga eventos de Event Grid más rápido de lo que puede procesarlos. Una vez que los eventos se encuentren en un centro de eventos, la aplicación puede procesar los eventos del centro de eventos en su propia programación. Puede escalar el procesamiento de eventos para controlar los eventos entrantes.

## <a name="tutorials"></a>Tutoriales
Consulte los siguientes ejemplos: 

|Título  |Descripción  |
|---------|---------|
| [Inicio rápido: enrutamiento de eventos personalizados a Azure Event Hubs con la CLI de Azure](custom-event-to-eventhub.md) | Envía un evento personalizado a un centro de eventos para que lo procese una aplicación. |
| [Plantilla de Resource Manager: creación de un tema personalizado Event Grid y envío de eventos a un centro de eventos](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.eventgrid/event-grid-event-hubs-handler)| Plantilla de Resource Manager que crea una suscripción a un tema personalizado. Envía eventos a una instancia de Azure Event Hubs. |

[!INCLUDE [event-grid-message-headers](../../includes/event-grid-message-headers.md)]


## <a name="rest-examples-for-put"></a>Ejemplos de REST (para PUT)


### <a name="event-hub"></a>Centro de eventos

```json
{
  "properties": 
  {
    "destination": 
    {
      "endpointType": "EventHub",
      "properties": 
      {
        "resourceId": "/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventHub/namespaces/<EVENT HUBS NAMESPACE NAME>/eventhubs/<EVENT HUB NAME>"
      }
    },
    "eventDeliverySchema": "EventGridSchema"
  }
}
```

### <a name="event-hub---delivery-with-managed-identity"></a>Centro de eventos: con identidad administrada

```json
{
  "properties": {
    "deliveryWithResourceIdentity": 
    {
      "identity": 
      {
        "type": "SystemAssigned"
      },
      "destination": 
      {
        "endpointType": "EventHub",
        "properties": 
        {
          "resourceId": "/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventHub/namespaces/<EVENT HUBS NAMESPACE NAME>/eventhubs/<EVENT HUB NAME>"
        }
      }
    },
    "eventDeliverySchema": "EventGridSchema"
  }
}
```

## <a name="next-steps"></a>Pasos siguientes
Consulte el artículo [Controladores de eventos](event-handlers.md) para obtener una lista de controladores de eventos compatibles. 
