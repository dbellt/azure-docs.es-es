---
title: Azure Event Grid en Kubernetes - Esquemas de eventos
description: En este artículo se describen los esquemas de eventos compatibles con Event Grid en Azure Arc para Kubernetes
author: jfggdl
ms.subservice: kubernetes
ms.author: jafernan
ms.date: 05/25/2021
ms.topic: conceptual
ms.openlocfilehash: 78576362ff3258344b6a340762799dfa1a52f623
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388740"
---
# <a name="event-schemas-in-event-grid-on-kubernetes"></a>Esquemas de eventos en Event Grid en Kubernetes
Event Grid en Kubernetes acepta y entrega eventos en formato JSON. Admite la [especificación del esquema Cloud Events 1.0](https://github.com/cloudevents/spec/blob/v1.0/spec.md) y ese es el esquema que se debe usar al publicar eventos en Event Grid. 

[!INCLUDE [event-grid-preview-feature-note.md](../../../includes/event-grid-preview-feature-note.md)]



## <a name="cloudevent-schema"></a>Esquema CloudEvent
[CloudEvents](https://cloudevents.io/) es una especificación abierta para la descripción de datos de eventos. Simplifica la interoperabilidad al proporcionar un esquema de eventos común para la publicación y el consumo de eventos. Consulte [Especificación de CloudEvents](https://github.com/cloudevents/spec/blob/master/json-format.md#3-envelope) para obtener información sobre los atributos de contexto obligatorios.

## <a name="example--event-using-cloudevents-schema"></a>Ejemplo - Evento con esquema de CloudEvents

```json
[{
      "specversion": "1.0",
      "type" : "orderCreated",
      "source": "myCompanyName/us/webCommerceChannel/myOnlineCommerceSiteBrandName",
      "id" : "eventId-n",
      "time" : "2020-12-25T20:54:07+00:00",
      "subject" : "account/acct-123224/order/o-123456",
      "dataSchema" : "1.0",
      "data" : {
         "orderId" : "123",
         "orderType" : "PO",
         "reference" : "https://www.myCompanyName.com/orders/123"
      }
}]
```

## <a name="next-steps"></a>Pasos siguientes
Para obtener información sobre los destinos y los controladores admitidos por Event Grid en Azure Arc para Kubernetes, consulte el documento sobre [Event Grid en Kubernetes - controladores de eventos](event-handlers.md).