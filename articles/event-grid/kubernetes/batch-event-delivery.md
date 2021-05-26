---
title: 'Azure Event Grid en Kubernetes: entrega de eventos por lotes'
description: En este artículo se describe cómo entregar lotes de eventos al destino.
author: jfggdl
ms.subservice: kubernetes
ms.author: jafernan
ms.date: 05/25/2021
ms.topic: conceptual
ms.openlocfilehash: a246ee493b89051ad35436cd9213462a09a5d1b0
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388431"
---
# <a name="event-grid-on-kubernetes---batch-event-delivery"></a>Event Grid en Kubernetes: entrega de eventos por lotes 
Event Grid en Kubernetes con Azure Arc admite la entrega de más de un evento en una única solicitud de entrega. Esta característica permite aumentar el rendimiento general de la entrega sin tener las sobrecargas de HTTP por solicitud. La entrega de eventos por lotes está desactivada de forma predeterminada y se puede activar mediante la configuración de la suscripción de eventos. 

[!INCLUDE [event-grid-preview-feature-note.md](../../../includes/event-grid-preview-feature-note.md)]

> [!WARNING]
> La duración máxima permitida para procesar cada solicitud de entrega no cambia, aunque el código del controlador de eventos pueda tener potencialmente más trabajo por cada solicitud por lotes. El tiempo de espera de entrega predeterminado es de 60 segundos.

## <a name="batch-event-delivery-policy"></a>Directiva de entrega de eventos por lotes
El comportamiento de entrega de eventos por lotes de Event Grid en Kubernetes puede personalizarse por suscripción de eventos, mediante la modificación de los dos valores siguientes:

- **Número máximo de eventos por lote**
    
    Esta opción establece un límite superior en el número de eventos que se pueden agregar a una solicitud de entrega por lotes.
- **Tamaño de lote preferido en kilobytes**
    
    Este elemento de configuración se usa para controlar aún más el número máximo de kilobytes que se pueden enviar por solicitud de entrega.

## <a name="batch-event-delivery-behavior"></a>Comportamiento de entrega de eventos por lotes   

- **Todos o ninguno**

    Event Grid en Kubernetes funciona con la semántica de todos o ninguno. No admite el procesamiento parcial de una entrega de eventos por lotes. Los controladores de eventos deben tener cuidado de solicitar solo los eventos por lote que puedan controlar de forma razonable en 60 segundos.
- **Procesamiento por lotes optimista**

    La configuración de la directiva de procesamiento por lotes no tiene límites estrictos acerca del comportamiento del procesamiento por lotes y se respeta en función del mejor esfuerzo. En las tasas de eventos bajas, a menudo observará que el tamaño del lote es menor que el número máximo de eventos solicitado por lote.
- **La entrega por lotes está establecida como DESACTIVADA de forma predeterminada**

    De forma predeterminada, Event Grid en Kubernetes solo agrega un evento a cada solicitud de entrega. La manera de activar la entrega de eventos por lotes es establecer una de las configuraciones mencionadas anteriormente en el artículo en la carga de la suscripción de eventos.
- **Valores predeterminados**

    No es necesario especificar ambas configuraciones (eventos máximos por lote y tamaño de lote aproximado en kilobytes) al crear una suscripción de eventos. Si solo se establece un valor, Event Grid en Kubernetes usa los valores predeterminados (configurables). 

## <a name="example"></a>Ejemplo
En el ejemplo siguiente se muestra cómo establecer `maxEventsPerBatch` y `preferredBatchSizeInKilobytes` en las propiedades del punto de conexión para habilitar el procesamiento por lotes. 

```json
{
    "properties":
    {
        "destination":
        {
            "endpointType": "WebHook",
            "properties":
             {
                "endpointUrl": "<your_webhook_url>",
                "maxEventsPerBatch": 10,
                "preferredBatchSizeInKilobytes": 64
             }
        },
    }
}
```

## <a name="next-steps"></a>Pasos siguientes
Para obtener información sobre los destinos y los controladores admitidos por Event Grid en Azure Arc para Kubernetes, consulte el documento sobre [Event Grid en Kubernetes - controladores de eventos](event-handlers.md).