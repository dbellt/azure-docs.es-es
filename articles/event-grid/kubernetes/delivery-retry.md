---
title: 'Azure Event Grid en Kubernetes: entrega de eventos y reintento'
description: En este artículo se describe cómo entrega los eventos Event Grid en Kubernetes con Azure Arc y cómo administra los mensajes no entregados.
author: jfggdl
ms.subservice: kubernetes
ms.author: jafernan
ms.date: 05/25/2021
ms.topic: conceptual
ms.openlocfilehash: c62ffc4374a461036d2e766fd57d96ec1906face
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386978"
---
# <a name="event-grid-on-kubernetes---event-delivery-and-retry"></a>Event Grid en Kubernetes: entrega de eventos y reintento
Event Grid en Kubernetes con Azure Arc intenta entregar cada uno de los mensajes al menos una vez para cada suscripción coincidente de forma inmediata. Si no recibe una respuesta HTTP 200 de operación correcta del suscriptor o si se produce algún error, Event Grid en Kubernetes reintenta la entrega de acuerdo con una programación de reintentos fija y la directiva de reintentos. 

De forma predeterminada, Event Grid en Kubernetes entrega un evento cada vez al suscriptor. Sin embargo, la carga de la solicitud de entrega es una matriz con un único evento. Puede entregar más de un evento a la vez si habilita la característica de procesamiento por lotes de salida. Para obtener más información sobre esta característica, consulte [Entrega de eventos por lotes](batch-event-delivery.md).

[!INCLUDE [event-grid-preview-feature-note.md](../../../includes/event-grid-preview-feature-note.md)]

> [!NOTE]
> Durante la versión preliminar, las características de Event Grid en Kubernetes se admiten a través de la versión de API [2020-10-15-Preview](/rest/api/eventgrid/version2020-10-15-preview/eventsubscriptions/createorupdate). 


## <a name="retry-schedule"></a>Programación de reintentos
Event Grid en Kubernetes espera 60 segundos para obtener una respuesta después de entregar un evento. Si el punto de conexión del suscriptor no envía una respuesta de acción correcta (HTTP 200 o similar), reintenta enviar el evento. Aquí le mostramos cómo funciona: 

1. El mensaje llega a la instancia de Event Grid en Kubernetes. Se intenta entregar inmediatamente.
1. Si se produce un error en la entrega, el mensaje se pone en una cola de un minuto y se reintenta transcurrido ese tiempo.
1. Si se sigue produciendo un error en la entrega, el mensaje se pone en una cola de 10 minutos y se reintenta transcurrido ese tiempo.
1. Las entregas se intentan hasta que se realizan o se alcanzan los límites de la directiva de reintentos.
 
## <a name="retry-policy"></a>Directiva de reintentos
Hay dos configuraciones que determinan la directiva de reintentos. Son los siguientes:

- Número máximo de intentos
- Período de vida del evento (TTL)

Si se alcanza alguno de los límites de la directiva de reintentos, el evento se descarta. La configuración de estos límites se realiza por cada suscripción. En la siguiente sección se describe cada uno de ellos con más detalle.

### <a name="configuring-defaults-per-subscriber"></a>Configuración de valores predeterminados por suscriptor
También puede especificar los límites de la directiva de reintentos por suscripción. Consulte la [documentación de la API](/rest/api/eventgrid/version2020-10-15-preview/eventsubscriptions/createorupdate) para obtener información sobre cómo configurar los valores predeterminados por suscriptor. Los valores predeterminados del nivel de suscripción reemplazan al módulo de Event Grid en las configuraciones de nivel de Kubernetes.

En el ejemplo siguiente se configura una suscripción de webhook con `maxNumberOfAttempts` establecido en 3 y `eventTimeToLiveInMinutes` definido como 30 minutos.

```json
{
 "properties": {
  "destination": {
   "endpointType": "WebHook",
   "properties": {
    "endpointUrl": "<your_webhook_url>",
    "eventDeliverySchema": " CloudEventSchemaV1_0"
   }
  },
  "retryPolicy": {
   "eventTimeToLiveInMinutes": 30,
   "maxDeliveryAttempts": 3
  }
 }
```

## <a name="next-steps"></a>Pasos siguientes
Para obtener información sobre los destinos y los controladores admitidos por Event Grid en Azure Arc para Kubernetes, consulte el documento sobre [Event Grid en Kubernetes - controladores de eventos](event-handlers.md).