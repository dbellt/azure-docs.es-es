---
title: Configuración de una puerta de señal para la grabación de vídeo basada en eventos con Live Video Analytics
description: En este artículo se proporciona una guía sobre cómo configurar una puerta de señal en un grafo de elementos multimedia con Live Video Analytics.
ms.topic: how-to
ms.date: 11/3/2020
ms.openlocfilehash: 6fc36757c44ad69d61466b373756bdd9428dbab9
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110367699"
---
# <a name="configure-a-signal-gate-for-event-based-video-recording-with-live-video-analytics"></a>Configuración de una puerta de señal para la grabación de vídeo basada en eventos con Live Video Analytics

[!INCLUDE [redirect to Azure Video Analyzer](./includes/redirect-video-analyzer.md)]

Dentro de un grafo de elementos multimedia, un [nodo de procesador de puerta de señal](media-graph-concept.md#signal-gate-processor) le permite reenviar el elemento multimedia de un nodo a otro cuando un evento desencadena la puerta. Cuando se desencadena, la puerta se abre y permite el flujo de elementos multimedia durante una duración especificada. En ausencia de eventos para desencadenar la puerta, esta se cierra y el elemento multimedia deja de fluir. Puede usar el procesador de puerta de señal para la grabación de vídeo basada en eventos.

En este artículo aprenderá a configurar un procesador de puerta de señal.

## <a name="suggested-prereading"></a>Lecturas previas sugeridas
-   [Grafo de elementos multimedia](media-graph-concept.md)
-   [Grabación de vídeo basada en eventos](event-based-video-recording-concept.md)


## <a name="problem"></a>Problema
Es posible que un usuario desee iniciar la grabación en un momento determinado antes o después de que un evento desencadene la puerta. El usuario conoce la latencia aceptable en su sistema. Por tanto, desea especificar la latencia del procesador de la puerta de señal. También está interesado en especificar la duración mínima y máxima de la grabación, independientemente del número de eventos nuevos que se reciban.
 
### <a name="use-case-scenario"></a>Escenarios de casos de uso
Supongamos que desea grabar vídeos cada vez que se abre la puerta principal de su edificio. Desea que la grabación: 

- Incluya *X* segundos antes de que se abra la puerta. 
- Dure al menos *Y* segundos si la puerta no se vuelve a abrir. 
- Dure como máximo *Z* segundos si la puerta se abre repetidamente. 
 
Sabe que el sensor de la puerta tiene una latencia de *K* segundos. Para reducir la posibilidad de que los eventos se descarten como llegadas tardías, desea dar un margen de al menos *K* segundos para que los eventos lleguen.


## <a name="solution"></a>Solución

Para solucionar el problema, modifique los parámetros del procesador de la puerta de señal.

Para configurar un procesador de puerta de señal, utilice estos cuatro parámetros:
- Ventana de evaluación de activación
- Desfase de la señal de activación
- Ventana de activación mínima
- Ventana de activación máxima

Cuando el procesador de puerta de señal se desencadena, permanece abierto durante el tiempo de activación mínimo. El evento de activación comienza en la marca de tiempo del evento más antiguo, más el desfase de la señal de activación. 

Si el procesador de la puerta de señal se vuelve a desencadenar mientras está abierto, el temporizador se restablece y la puerta permanece abierta durante al menos el tiempo de activación mínimo. El procesador de la puerta de señal nunca permanece abierto más tiempo que el tiempo de activación máximo. 

Un evento (evento 1) que se produce antes que otro evento (evento 2), basado en marcas de tiempo de elemento multimedia, podría no tenerse en cuenta si el sistema se retrasa y el evento 1 llega al procesador de la puerta de señal después del evento 2. Si el evento 1 no llega entre el momento en el que lo hace el evento 2 y la ventana de evaluación de activación, el evento 1 se descarta. No se pasa al procesador de puerta de señal. 

Los identificadores de correlación se establecen para cada evento. Estos identificadores se establecen a partir del evento inicial. Son secuenciales para cada evento siguiente.

> [!IMPORTANT]
> La hora del elemento multimedia se basa en la marca de tiempo de dicho elemento cuando se produce un evento en él. La secuencia de eventos que llegan a la puerta de señal podría no reflejar la secuencia de eventos que llegan en el momento del elemento multimedia.


### <a name="parameters-based-on-the-physical-time-that-events-arrive-at-the-signal-gate"></a>Parámetros basados en la hora física a la que llegan los eventos a la puerta de señal

* **minimumActivationTime (duración más corta posible de una grabación)** : el número mínimo de segundos que el procesador de la puerta de señal permanece abierto después de que se desencadene para recibir nuevos eventos, a menos que maximumActivationTime lo interrumpa.
* **maximumActivationTime (duración más larga posible de una grabación)** : el número máximo de segundos desde el evento inicial que el procesador de la puerta de señal permanece abierto después de haberse desencadenado para recibir nuevos eventos, independientemente de los eventos que se reciban.
* **activationSignalOffset**: el número de segundos entre la activación del procesador de la puerta de señal y el inicio de la grabación de vídeo. Normalmente, este valor es negativo porque inicia la grabación antes del evento de desencadenamiento.
* **activationEvaluationWindow**: a partir del evento de desencadenamiento inicial, el número de segundos en los que un evento que se produjo antes del evento inicial, en el tiempo del elemento multimedia, debe llegar al procesador de la puerta de señal antes de que se descarte y se considere una llegada tardía.

> [!NOTE]
> Una *llegada tardía* es cualquier evento que llegue después de que la ventana de evaluación de la activación haya pasado, pero que llegue antes del evento inicial en el tiempo del elemento multimedia.

### <a name="limits-of-parameters"></a>Límites de parámetros

* **activationEvaluationWindow**: de 0 a 10 segundos
* **activationSignalOffset**: de -1 minuto a 1 minuto
* **minimumActivationTime**: de 1 segundo a 1 hora
* **maximumActivationTime**: de 1 segundo a 1 hora


En el caso de uso, los parámetros se establecerían de la siguiente manera:

* **activationEvaluationWindow**: *K* segundos
* **activationSignalOffset**: *-X* segundos
* **minimumActivationWindow**: *Y* segundos
* **maximumActivationWindow**: *Z* segundos


A continuación se muestra un ejemplo del aspecto de la sección del nodo **Procesador de la puerta de señal** en una topología de grafos de elementos multimedia para los siguientes valores de parámetro:
* **activationEvaluationWindow**: 1 segundo
* **activationSignalOffset**: -5 segundos
* **minimumActivationTime**: 20 segundos
* **maximumActivationTime**: 40 segundos

> [!IMPORTANT]
> Se espera el [formato de duración de ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Durations
) para cada valor de parámetro. Por ejemplo, PT1S = 1 segundo.


```
"processors":              
[
          {
            "@type": "#Microsoft.Media.MediaGraphSignalGateProcessor",
            "name": "signalGateProcessor",
            "inputs": [
              {
                "nodeName": "iotMessageSource"
              },
              {
                "nodeName": "rtspSource"
              }
            ],
            "activationEvaluationWindow": "PT1S",
            "activationSignalOffset": "-PT5S",
            "minimumActivationTime": "PT20S",
            "maximumActivationTime": "PT40S"
          }
]
```


Ahora, considere cómo se comportará esta configuración del procesador de la puerta de señal en diferentes escenarios de grabación.

### <a name="recording-scenarios"></a>Escenarios de grabación

**Un evento de un origen (*activación normal*)**

Un procesador de puerta de señal que recibe un evento da lugar a una grabación que comienza 5 segundos (señal de activación = 5 segundos) antes de que el evento llegue a la puerta. El resto de la grabación es de 20 segundos (tiempo de activación mínimo = 20 segundos) porque ningún otro evento llega antes del final del tiempo de activación mínimo para volver a desencadenar la puerta.

Diagrama de ejemplo:
> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/configure-signal-gate-how-to/normal-activation.svg" alt-text="Diagrama que muestra la activación normal de un evento de un origen.":::

* Duración de la grabación = -desfase + minimumActivationTime = [E1+desfase, E1+minimumActivationTime]


**Dos eventos de un origen (*activación redesencadenada*)**

Un procesador de puerta de señal que recibe dos eventos da lugar a una grabación que comienza 5 segundos (desfase de la señal de activación = 5 segundos) antes de que el evento llegue a la puerta. Además, el evento 2 llega 5 segundos después del evento 1. Dado que el evento 2 llega antes del final del tiempo mínimo de activación del evento 1 (20 segundos), la puerta se vuelve a desencadenar. El resto de la grabación es de 20 segundos (tiempo de activación mínimo = 20 segundos) porque ningún otro evento llega antes del final del tiempo de activación mínimo del evento 2 para volver a desencadenar la puerta.

Diagrama de ejemplo:
> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/configure-signal-gate-how-to/retriggering-activation.svg" alt-text="Diagrama que muestra la activación redesencadenada de dos eventos de un origen.":::

* Duración de la grabación =-desfase + (llegada del evento 2 - llegada del evento 1) + minimumActivationTime


***N* eventos de un origen (*activación máxima*)**

Un procesador de puerta de señal que recibe *N* eventos da lugar a una grabación que comienza 5 segundos (desfase de la señal de activación = 5 segundos) antes de que el primer evento llegue a la puerta. Como cada evento llega antes del final del tiempo de activación mínimo de 20 segundos del evento anterior, la puerta se vuelve a desencadenar una y otra vez. Permanecerá abierta hasta el tiempo máximo de activación de 40 segundos después del primer evento. A continuación, la puerta se cierra y ya no aceptará ningún evento nuevo.

Diagrama de ejemplo:
> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/configure-signal-gate-how-to/maximum-activation.svg" alt-text="Diagrama que muestra la activación máxima de N eventos de un origen.":::
 
* Duración de la grabación =-offset + maximumActivationTime

> [!IMPORTANT]
> En los diagramas anteriores se supone que todos los eventos llegan en el mismo instante en tiempo físico y tiempo del elemento multimedia. Es decir, suponen que no hay ninguna llegada tardía.

## <a name="next-steps"></a>Pasos siguientes

Pruebe el [tutorial de grabación de vídeo basada en eventos](event-based-video-recording-tutorial.md). Empiece editando el archivo [topology.json](https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/evr-hubMessage-assets/topology.json). Modifique los parámetros para el nodo signalgateProcessor y luego siga el resto del tutorial. Revise las grabaciones de vídeo para analizar el efecto de los parámetros.



