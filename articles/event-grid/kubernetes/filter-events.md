---
title: 'Azure Event Grid en Kubernetes: Filtrar eventos'
description: En este artículo se describe cómo filtrar eventos al crear una suscripción de Azure Event Grid.
author: jfggdl
ms.subservice: kubernetes
ms.author: jafernan
ms.date: 05/25/2021
ms.topic: conceptual
ms.openlocfilehash: 814c703a2db23088fc959501d3f0266ff19b6f49
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388734"
---
# <a name="event-grid-on-kubernetes---event-filtering-for-subscriptions"></a>Event Grid en Kubernetes: Filtrado de eventos para suscripciones
Event Grid en Kubernetes permite especificar filtros por cualquier propiedad de la carga JSON. Estos filtros se modelan como un conjunto de condiciones AND, donde cada condición externa tiene condiciones OR internas opcionales. Para cada condición AND, se especifican los siguientes valores:

- Tipo de operador: el tipo de comparación.
- Clave: la ruta de acceso JSON a la propiedad en la que se va a aplicar el filtro.
- Valor: el valor de referencia con el que se ejecuta el filtro; (o) Valores: el conjunto de valores de referencia con el que se ejecuta el filtro.

[!INCLUDE [event-grid-preview-feature-note.md](../../../includes/event-grid-preview-feature-note.md)]



## <a name="filter-by-event-type"></a>Filtrar por tipo de evento
De manera predeterminada, todos los [tipos de evento](event-schemas.md) (atributo `type`) del origen de eventos se envían al punto de conexión. Puede decidir enviar solo determinados tipos de eventos al punto de conexión. La sintaxis JSON para el filtrado por tipo de evento es:

```json
"filter": {
  "includedEventTypes": [
    "orderCreated",
    "orderUpdated"
  ]
}
```

En el ejemplo anterior, los únicos eventos de tipo `orderCreated` y los eventos `orderUpdated` se envían al punto de conexión del suscriptor. 

Este es un evento de muestra:

```json
[{
      "specVersion": "1.0",
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

## <a name="filter-by-subject"></a>Filtrar por asunto
Para filtrar solo por asunto, especifique un valor inicial o final para el asunto. La sintaxis JSON para el filtrado por asunto es:

```json
"filter": {
  "subjectBeginsWith": "/account/acct-123224/"
}
``` 

Por ejemplo, el filtro configurado anteriormente enviará todos los pedidos asociados con la cuenta `acct-123224` al punto de conexión del suscriptor. 

Al publicar eventos en temas, cree asuntos para los eventos de manera que faciliten que los suscriptores sepan si les interesa el evento. Los suscriptores usan la propiedad del asunto para filtrar y redirigir eventos. Considere la posibilidad de agregar la ruta de acceso de donde se produjo el evento, para que los suscriptores puedan filtrar por los segmentos de esa ruta de acceso. La ruta de acceso permite que los suscriptores filtren eventos de una manera más amplia o más restringida. Si proporciona una ruta de acceso de tres segmentos, como /A/B/C en el asunto, los suscriptores pueden filtrar por el primer segmento /A para obtener un amplio conjunto de eventos. Esos suscriptores obtienen eventos con asuntos como /A/B/C o /A/D/E. Otros suscriptores pueden filtrar por /A/B para obtener un conjunto de eventos más reducido.

## <a name="filter-by-values-in-event-data"></a>Filtrado por valores en datos de eventos
Consulte la [sección Filtrado avanzado del artículo Event Grid en Azure](../event-filtering.md) para obtener información detallada sobre el filtrado avanzado. Las siguientes características y operadores no son compatibles con Event Grid en Kubernetes. 

- Filtrado de datos de matriz en claves de eventos entrantes
- Permita el filtrado en [atributos de contexto de extensiones de CloudEvents](https://github.com/cloudevents/spec/blob/v1.0/documented-extensions.md).
- Operadores siguientes
    - StringNotContains
    - StringNotBeginsWith
    - StringNotEndsWith
    - NumberInRange
    - NumberNotInRange
    - IsNullOrUndefined
    - IsNotNull
    

## <a name="next-steps"></a>Pasos siguientes
Para obtener información sobre los destinos y los controladores admitidos por Event Grid en Azure Arc para Kubernetes, consulte el documento sobre [Event Grid en Kubernetes - controladores de eventos](event-handlers.md).