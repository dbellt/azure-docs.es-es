---
title: 'Entrega y reintento: Azure Event Grid en IoT Edge | Microsoft Docs'
description: Entrega y reintento en Event Grid en IoT Edge
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.subservice: iot-edge
ms.date: 05/10/2021
ms.topic: article
ms.openlocfilehash: 541f85181546dcb9998509841e463fdb0687a5d6
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110370435"
---
# <a name="delivery-and-retry"></a>Entrega y nuevo intento

Event Grid ofrece entrega duradera. Intenta entregar cada mensaje al menos una vez inmediatamente después de encontrar una suscripción coincidente. Si el punto de conexión de un suscriptor no reconoce la recepción de un evento o se produce un error, Event Grid reintenta la entrega según una **programación de reintentos** fija y una **directiva de reintentos**.  De forma predeterminada, el módulo de Event Grid entrega un evento cada vez al suscriptor. No obstante, la carga es una matriz con un solo evento. Puede hacer que el módulo entregue más de un evento a la vez si habilita la característica de procesamiento por lotes de salida. Para más información sobre esta característica, consulte [Procesamiento por lotes de salida](delivery-output-batching.md).  

> [!IMPORTANT]
>No se admite la persistencia de los datos de eventos. Esto significa que al volver a implementar o reiniciar el módulo de Event Grid, se perderán todos los eventos que no se hayan entregado aún.

## <a name="retry-schedule"></a>Programación de reintentos

Event Grid espera 60 segundos para obtener una respuesta después de entregar un mensaje. Si el punto de conexión del suscriptor no confirma la respuesta, el mensaje se pondrá en una de nuestras colas de reserva en los sucesivos reintentos.

Hay dos colas de reserva preconfiguradas que determinan la programación según la que se realizará un reintento. Son los siguientes:

| Programación | Descripción |
| ---------| ------------ |
| 1 minuto | Los mensajes que terminan aquí se intentan cada minuto.
| 10 minutos | Los mensajes que terminan aquí se intentan cada 10 minutos.

### <a name="how-it-works"></a>Funcionamiento

1. El mensaje llega al módulo de Event Grid. Se intenta entregar inmediatamente.
1. Si se produce un error en la entrega, el mensaje se pone en una cola de 1 minuto y se vuelve a intentar transcurrido ese tiempo.
1. Si se sigue produciendo un error en la entrega, el mensaje se pone en una cola de 10 minutos y se vuelve a intentar transcurrido ese tiempo.
1. Las entregas se intentan hasta que se realizan o se alcanzan los límites de la directiva de reintentos.

## <a name="retry-policy-limits"></a>Límites de la directiva de reintentos

Hay dos configuraciones que determinan la directiva de reintentos. Son los siguientes:

* Número máximo de intentos
* Período de vida del evento (TTL)

Si se alcanza alguno de los límites de la directiva de reintentos, se descartará un evento. La programación de reintentos se describió en la sección Programación de reintentos. La configuración de estos límites se puede realizar para todos los suscriptores o por suscripción. En la siguiente sección se describe cada uno de ellos con más detalle.

## <a name="configuring-defaults-for-all-subscribers"></a>Configuración de los valores predeterminados para todos los suscriptores

Hay dos propiedades, `brokers__defaultMaxDeliveryAttempts` y `broker__defaultEventTimeToLiveInSeconds`, que se pueden configurar como parte de la implementación de Event Grid, que controlan los valores predeterminados de la directiva de reintentos de todos los suscriptores.

| Nombre de la propiedad | Descripción |
| ---------------- | ------------ |
| `broker__defaultMaxDeliveryAttempts` | Número máximo de intentos de entrega de un evento. Valor predeterminado: 30.
| `broker__defaultEventTimeToLiveInSeconds` | TTL del evento en segundos transcurrido el cual se descartará un evento si no se entrega. Valor predeterminado: **7200** segundos.

## <a name="configuring-defaults-per-subscriber"></a>Configuración de valores predeterminados por suscriptor

También puede especificar los límites de la directiva de reintentos por suscripción.
Consulte la [documentación de la API](api.md) para información sobre cómo configurar los valores predeterminados por suscriptor. Los valores predeterminados de nivel de suscripción invalidan las configuraciones de nivel de módulo.

## <a name="examples"></a>Ejemplos

En el ejemplo siguiente se configura la directiva de reintentos en el módulo de Event Grid con maxNumberOfAttempts = 3 y TTL de eventos de 30 segundos.

```json
{
  "Env": [
    "broker__defaultMaxDeliveryAttempts=3",
    "broker__defaultEventTimeToLiveInSeconds=1800"
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

En el ejemplo siguiente se configura una suscripción de webhook con maxNumberOfAttempts = 3 y TTL de eventos de 30 minutos.

```json
{
 "properties": {
  "destination": {
   "endpointType": "WebHook",
   "properties": {
    "endpointUrl": "<your_webhook_url>",
    "eventDeliverySchema": "eventgridschema"
   }
  },
  "retryPolicy": {
   "eventExpiryInMinutes": 30,
   "maxDeliveryAttempts": 3
  }
 }
}
```
