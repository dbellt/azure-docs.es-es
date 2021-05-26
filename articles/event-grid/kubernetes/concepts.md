---
title: 'Azure Event Grid en Kubernetes: conceptos'
description: En este artículo se explican los conceptos básicos de Azure Event Grid en Kubernetes con Azure Arc (versión preliminar)
author: jfggdl
ms.subservice: kubernetes
ms.author: jafernan
ms.date: 05/25/2021
ms.topic: conceptual
ms.openlocfilehash: e9d7fff98a6d6c062f529528f9f1d3071a038049
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388487"
---
# <a name="event-grid-on-kubernetes---concepts"></a>Event Grid en Kubernetes: conceptos
En este artículo se describen los conceptos principales de Event Grid en Kubernetes con Azure Arc (versión preliminar).

[!INCLUDE [event-grid-preview-feature-note.md](../../../includes/event-grid-preview-feature-note.md)]

## <a name="events"></a>Eventos
Un evento es un registro de datos que anuncia un hecho sobre el funcionamiento de un sistema de software. Normalmente, un evento anuncia un cambio de estado debido a una señal que el sistema genera u observa. Los eventos contienen dos tipos de información: 

- [Datos de evento](https://github.com/cloudevents/spec/blob/master/spec.md#event-data), que representan la aparición de un cambio de estado. 
- [Atributos de contexto](https://github.com/cloudevents/spec/blob/master/spec.md#context-attributes), que proporcionan información contextual sobre la aparición del evento.     

    Tanto los datos de eventos como los atributos de contexto pueden usarse para filtrar eventos. 

Event Grid en Kubernetes admite la especificación de esquema [CloudEvents](https://github.com/cloudevents/spec/tree/master). Este es un ejemplo de un evento que usa el esquema CloudEvents. Event Grid admite un evento con un tamaño máximo de 1 MB.

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

## <a name="source"></a>Source
El atributo de origen (source) describe el contexto en el que se produjo el evento. El origen puede ser el originador de los eventos. Sin embargo, en algunos casos, hay productores que crean y publican eventos. Además, estos productores son distintos del origen. Para simplificar, en este artículo se supone que el origen es el productor de los eventos. 

Cada origen de evento genera eventos de uno o varios tipos. Como origen de los eventos, una aplicación define un conjunto de eventos relacionados para anunciar sus cambios de estado. Todos los eventos tienen información común, como el origen del evento, la hora en que el evento ha tenido lugar y un identificador único. Cada evento tiene además información específica que solo es relevante para el tipo de evento concreto. La compatibilidad de un evento con un tamaño de hasta 1 MB se encuentra actualmente en versión preliminar.

Para obtener las propiedades que se incluyen en un evento, consulte el documento sobre el [esquema CloudEvents](event-schemas.md#cloudevent-schema).

## <a name="publishers"></a>Publicadores
Los publicadores de eventos son aplicaciones o sistemas que envían eventos a Event Grid para que se entreguen a los suscriptores de eventos.

## <a name="topics"></a>Temas
Un tema es una forma de canal de entrada que expone un punto de conexión en el que los publicadores envían eventos a Event Grid.

Un tema se puede usar para una colección de eventos relacionados. Puede crear un tema para cada categoría de eventos relacionados. En algunos casos, el origen se puede usar para organizar los eventos en categorías, ya que los orígenes suelen estar asociados a un conjunto de tipos de eventos estrechamente relacionados ("MyApp.OrderCreated”, “MyApp.OderDeleted”, “MyApp.OrderRejected” y así sucesivamente). 

Considere una aplicación que envía eventos relacionados con la administración de las cuentas de usuario y el procesamiento de pedidos. Es poco probable que un suscriptor de eventos esté interesado en consumir ambas categorías de eventos. Cree dos temas personalizados y deje que los controladores de eventos se suscriban a uno que les interese. Para soluciones pequeñas, puede que prefiera enviar todos los eventos a un solo tema. 

## <a name="event-subscribers"></a>Suscriptores de eventos
Los suscriptores de eventos son sistemas de software, como los microservicios, que exponen puntos de conexión a los que Event Grid entrega eventos. 

## <a name="event-subscriptions"></a>Suscripciones a eventos
Una suscripción a eventos indica a Event Grid los eventos de un tema que le interesa recibir (filtrado de eventos) y dónde enviarlos (enrutamiento de eventos). Al crear una suscripción de eventos, proporciona un punto de conexión para controlar el evento. Puede seleccionar los eventos que quiere que se entreguen al punto de conexión mediante la configuración de cláusulas de filtro en la suscripción de eventos. 

## <a name="event-handlers"></a>Controladores de eventos
Un controlador de eventos es un sistema de software que expone un punto de conexión al que se envían los eventos. El controlador recibe el evento y realiza acciones para procesar el evento. Event Grid admite varios tipos de controladores. Como controlador, puede usar un servicio compatible de Azure hospedado en Kubernetes o en Azure, o bien usar su propia solución que expone un webhook (punto de conexión) dondequiera que se hospede. Según el tipo de controlador, Event Grid sigue distintos procedimientos para garantizar la entrega del evento. Si el controlador de eventos de destino es un webhook HTTP, el evento se reintenta hasta que el controlador devuelve un código de estado 200 - Correcto. Para obtener más información, vea [Controladores de eventos](event-handlers.md).

## <a name="sas-authentication"></a>Autenticación de SAS
Event Grid en Kubernetes proporciona autenticación basada en claves SAS para publicar eventos en los temas.

## <a name="event-delivery"></a>Entrega de eventos
Event Grid en Kubernetes proporciona un mecanismo de entrega y reintento confiable. Si Event Grid no puede confirmar que un evento se haya recibido en el punto de conexión del controlador de eventos, vuelve a entregarlo. Para más información, vea [Entrega y reintento de entrega de mensajes de Event Grid](delivery-retry.md).

## <a name="batch-event-publishing"></a>Publicación de eventos por lotes
Cuando se usa un tema, los eventos siempre deben publicarse en una matriz. En escenarios de bajo rendimiento, la matriz solo tendrá un evento. Para los casos de uso de gran volumen, se recomienda procesar varios eventos juntos por publicación para lograr una mayor eficacia. Los lotes pueden tener hasta 1 MB. Cada uno de los eventos no debe superar el valor de 1 MB. Para obtener más información, consulte [Entrega de eventos por lotes](batch-event-delivery.md).

## <a name="event-grid-on-kubernetes-components"></a>Componentes de Event Grid en Kubernetes

- El **operador de Event Grid** implementa el [patrón de operador](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/). Busca los cambios de estado de los recursos de Event Grid como resultado de las solicitudes del plano de control realizadas al servidor de API de Kubernetes. Cuando haya una solicitud que afecte al estado de cualquiera de los recursos de Event Grid, el operador de Event Grid sincroniza ese estado con el agente de Event Grid.
- El **agente de Event Grid** actúa en las operaciones de plano de control y de plano de datos.

   Como servicio de plano de control, es el responsable de llevar Event Grid al estado deseado que comunique el operador de Event Grid. Por ejemplo, cuando se realiza una solicitud para crear un tema, completa esa solicitud y se actualizan los metadatos del servicio.

   Como servicio de plano de datos, atiende a todas las solicitudes de publicación de eventos y entrega los eventos a sus destinos configurados en las suscripciones de eventos.

## <a name="next-steps"></a>Pasos siguientes
Para empezar, consulte [Creación de temas y suscripciones](create-topic-subscription.md).