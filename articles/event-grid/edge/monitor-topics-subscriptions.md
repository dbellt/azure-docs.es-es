---
title: Supervisión de temas y suscripciones de eventos - Azure Event Grid IoT Edge | Microsoft Docs
description: Supervisión de temas y suscripciones de eventos
ms.date: 05/10/2021
ms.topic: article
ms.subservice: iot-edge
ms.openlocfilehash: 02fa0daa54d7b5a079ee1d8dff5a104ca23cc56e
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110367803"
---
# <a name="monitor-topics-and-event-subscriptions"></a>Supervisión de temas y suscripciones de eventos

Event Grid en Microsoft Edge expone una serie de métricas para los temas y las suscripciones de eventos en el [formato de exposición Prometheus](https://prometheus.io/docs/instrumenting/exposition_formats/). En este artículo se describen las métricas disponibles y cómo habilitarlas.

## <a name="enable-metrics"></a>Habilitación de métricas

Para configurar el módulo y emitir métricas, establezca la variable de entorno `metrics__reporterType` en `prometheus` en las opciones de creación del contenedor:

 ```json
        {
          "Env": [
            "metrics__reporterType=prometheus"
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

Las métricas estarán disponibles en `5888/metrics` del módulo para HTTP y en `4438/metrics` para HTTPS. Por ejemplo, `http://<modulename>:5888/metrics?api-version=2019-01-01-preview` para HTTP. En este punto, un módulo de métricas puede sondear el punto de conexión para recopilar métricas, como se ilustra en la siguiente [arquitectura de ejemplo](https://github.com/veyalla/ehm).

## <a name="available-metrics"></a>Métricas disponibles

Tanto los temas como las suscripciones de eventos emiten métricas para proporcionarle información sobre la entrega de eventos y el rendimiento de los módulos.

### <a name="topic-metrics"></a>Métricas del tema

| Métrica | Descripción |
| ------ | ----------- |
| EventsReceived | Número de eventos publicados en el tema
| UnmatchedEvents | Número de eventos publicados en el tema que no coinciden con una suscripción de eventos y que se quitan
| SuccessRequests | Número de solicitudes de publicación de entrada que recibe el tema
| SystemErrorRequests | Número de solicitudes de publicación de entrada con errores debido a un error interno del sistema
| UserErrorRequests | Número de solicitudes de publicación de entrada con errores debido a un error de usuario como JSON con formato incorrecto
| SuccessRequestLatencyMs | Latencia de respuesta de la solicitud de publicación en milisegundos


### <a name="event-subscription-metrics"></a>Métricas de la suscripción de eventos

| Métrica | Descripción |
| ------ | ----------- |
| DeliverySuccessCounts | Número de eventos entregados correctamente al punto de conexión configurado
| DeliveryFailureCounts | Número de eventos que no se pudieron entregar al punto de conexión configurado
| DeliverySuccessLatencyMs | Latencia de eventos entregados correctamente en milisegundos
| DeliveryFailureLatencyMs | Latencia de errores de entrega de eventos en milisegundos
| SystemDelayForFirstAttemptMs | Retraso del sistema en eventos antes del primer intento de entrega en milisegundos
| DeliveryAttemptsCount | Número de intentos de entrega de eventos (correctos y erróneos)
| ExpiredCounts | Número de eventos que expiraron y no se entregaron al punto de conexión configurado