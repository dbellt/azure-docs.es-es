---
title: Procesamiento por lotes de salida en Azure Event Grid IoT Edge | Microsoft Docs
description: Procesamiento por lotes de salida en Event Grid en IoT Edge.
author: HiteshMadan
manager: rajarv
ms.author: himad
ms.reviewer: spelluru
ms.subservice: iot-edge
ms.date: 05/10/2021
ms.topic: article
ms.openlocfilehash: a647b83a3a64de72cca81ef2ade411135410ab45
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110370469"
---
# <a name="output-batching"></a>Procesamiento por lotes de salida

Event Grid admite la entrega de más de un evento en una única solicitud de entrega. Esta característica permite aumentar el rendimiento general de la entrega sin pagar las sobrecargas de HTTP por solicitud. El procesamiento por lotes está desactivado de forma predeterminada y se puede activar por suscripción.

> [!WARNING]
> La duración máxima permitida para procesar cada solicitud de entrega no cambia, aunque el código del suscriptor pueda tener potencialmente más trabajo por cada solicitud por lotes. El tiempo de espera de entrega predeterminado es de 60 segundos.

## <a name="batching-policy"></a>Directiva de procesamiento por lotes

El comportamiento de procesamiento por lotes de Event Grid se puede personalizar por suscriptor mediante la modificación de los dos valores siguientes:

* Número máximo de eventos por lote

  Esta opción establece un límite superior en el número de eventos que se pueden agregar a una solicitud de entrega por lotes.

* Tamaño de lote preferido en kilobytes

  Este control se utiliza para controlar aún más el número máximo de kilobytes que se pueden enviar por solicitud de entrega.

## <a name="batching-behavior"></a>Comportamiento de procesamiento por lotes

* Todos o ninguno

  Event Grid funciona con la semántica de todos o ninguno. No admite el procesamiento parcial de una entrega por lotes. Los suscriptores deben tener cuidado de solicitar solo los eventos por lote que puedan controlar de forma razonable en 60 segundos.

* Procesamiento por lotes optimista

  La configuración de la directiva de procesamiento por lotes no tiene límites estrictos acerca del comportamiento del procesamiento por lotes, y se respeta en función del mejor esfuerzo. En las tasas de eventos bajas, a menudo observará que el tamaño del lote es menor que el número máximo de eventos solicitado por lote.

* De forma predeterminada, este valor está desactivado.

  De forma predeterminada, Event Grid solo agrega un evento a cada solicitud de entrega. La manera de activar el procesamiento por lotes es establecer una de las opciones mencionadas anteriormente en el artículo en el JSON de la suscripción de eventos.

* Valores predeterminados

  No es necesario especificar ambas configuraciones (eventos máximos por lote y tamaño de lote aproximado en kilobytes) al crear una suscripción de eventos. Si solo se establece un valor, Event Grid usa valores predeterminados (configurables). Vea las siguientes secciones para ver los valores predeterminados y aprender a reemplazarlos.

## <a name="turn-on-output-batching"></a>Activación del procesamiento por lotes de salida

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

## <a name="configuring-maximum-allowed-values"></a>Configuración de los valores máximos permitidos

La siguiente configuración de tiempo de implementación controla el valor máximo permitido al crear una suscripción de eventos.

| Nombre de la propiedad | Descripción |
| ------------- | ----------- | 
| `api__deliveryPolicyLimits__maxpreferredBatchSizeInKilobytes` | Valor máximo permitido para el control `PreferredBatchSizeInKilobytes`. El valor predeterminado es `1033`.
| `api__deliveryPolicyLimits__maxEventsPerBatch` | Valor máximo permitido para el control `MaxEventsPerBatch`. El valor predeterminado es `50`.

## <a name="configuring-runtime-default-values"></a>Configuración de valores predeterminados en tiempo de ejecución

La siguiente configuración de tiempo de implementación controla el valor predeterminado en tiempo de ejecución de cada control cuando no se especifica en la suscripción de eventos. Para volver a iterar, debe establecerse al menos un control en la suscripción de eventos para activar el comportamiento del procesamiento por lotes.

| Nombre de la propiedad | Descripción |
| ------------- | ----------- |
| `broker__defaultMaxBatchSizeInBytes` | Tamaño máximo de la solicitud de entrega cuando solo se especifica `MaxEventsPerBatch`. El valor predeterminado es `1_058_576`.
| `broker__defaultMaxEventsPerBatch` | Número máximo de eventos para agregar a un lote cuando solo se especifica `MaxBatchSizeInBytes`. El valor predeterminado es `10`.
