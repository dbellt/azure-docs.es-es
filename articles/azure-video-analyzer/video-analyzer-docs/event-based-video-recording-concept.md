---
title: 'Grabación de vídeo basada en eventos de Azure Video Analyzer: Azure'
description: La grabación de vídeo basada en eventos (EVR) de Azure Video Analyzer hace referencia al proceso de grabación de vídeo desencadenado por un evento. El evento en cuestión podría originarse debido al procesamiento de la propia señal de vídeo (por ejemplo, cuando se detecta movimiento) o podría proceder de un origen independiente (por ejemplo, un sensor de puerta indica que se ha abierto la puerta). En este artículo se describen algunos casos de uso relacionados con la grabación de vídeo basada en eventos.
ms.topic: conceptual
ms.date: 05/13/2021
ms.openlocfilehash: de6581b60d0cee4c1ac34ffbd60ee22509807e8b
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388508"
---
# <a name="event-based-video-recording"></a>Grabación de vídeo basada en eventos  

La grabación de vídeo basada en eventos (EVR) hace referencia al proceso de grabación de vídeo desencadenado por un evento. El evento en cuestión podría originarse debido al procesamiento de la propia señal de vídeo (por ejemplo, cuando se detecta movimiento) o podría proceder de un origen independiente (por ejemplo, un sensor de puerta indica que se ha abierto la puerta). En este artículo se describen algunos casos de uso relacionados con la grabación de vídeo basada en eventos.

## <a name="suggested-pre-reading"></a>Sugerencias previas a la lectura  

* [Grabación de vídeo continua](continuous-video-recording.md)
* [Reproducción de contenido grabado](playback-recordings-how-to.md)
* [Concepto de canalización](pipeline.md)

## <a name="overview"></a>Información general 

Puede usar Video Analyzer para la grabación de vídeo basada en eventos de dos maneras:
* Grabar la entrada de una determinada cámara IP con capacidad RTSP en un determinado recurso de vídeo en la nube, donde cada nuevo evento se anexaría a la grabación disponible en ese recurso de vídeo.
* Grabar en archivos MP4 independientes en el almacenamiento local del dispositivo IoT Edge: cada evento daría lugar a un nuevo archivo MP4.

En este artículo se describen algunos casos de uso relacionados con la grabación de vídeo basada en eventos.

### <a name="video-recording-triggered-by-motion-detection"></a>Grabación de vídeo desencadenada por la detección de movimiento  

En este caso de uso, puede grabar clips de vídeo solo cuando se detecten movimientos y recibir una alerta cada vez que se genere un clip de vídeo. Esto puede ser relevante en escenarios de seguridad comercial que impliquen la protección de infraestructura crítica. La EVR le ayudará a reducir los costos de almacenamiento.

En el diagrama siguiente se muestra una representación gráfica de una canalización que aborda este caso de uso. La representación JSON de la topología de canalización de este tipo de canalización se puede encontrar [aquí](https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/evr-motion-video-sink/topology.json).

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording/motion-detection.png" alt-text="Grabación basada en eventos de vídeo en directo cuando se detecta movimiento.":::

En el diagrama, el nodo de origen RTSP captura la fuente de vídeo en directo de la cámara y la entrega a un nodo del [procesador de detección de movimiento](pipeline.md#motion-detection-processor). Después de detectar el movimiento en el vídeo en directo, el nodo del procesador de detección de movimiento genera eventos, que van al nodo del [procesador de la puerta de señales](pipeline.md#signal-gate-processor), así como al nodo receptor de mensajes de IoT Hub. El último nodo envía los eventos al centro de IoT Edge, desde donde se pueden enrutar a otros destinos para desencadenar alertas. 

Los eventos del nodo detector de movimiento también desencadenan el nodo del procesador de la puerta de señales, la abren y permiten que la fuente de vídeo en directo del nodo de origen RTSP pase a través del [nodo receptor de vídeo](pipeline.md#video-sink). En ausencia de estos eventos posteriores de detección de movimiento, la puerta se cerrará después de un período de tiempo configurado. Esto determina la duración del vídeo grabado que se anexa al recurso de vídeo.

### <a name="video-recording-based-on-events-from-other-sources"></a>Grabación de vídeo basada en eventos desde otros orígenes  

En este caso de uso, se pueden usar las señales de otro sensor de IoT para desencadenar la grabación de vídeos. En el diagrama siguiente se muestra una representación gráfica de una canalización que aborda este caso de uso. La representación JSON de la topología de canalización de este tipo de canalización se puede encontrar [aquí](https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/evr-hubMessage-files/topology.json).

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording/other-sources.png" alt-text="Grabación basada en eventos de vídeo en directo cuando lo señala un origen externo.":::

En el diagrama, el sensor externo envía eventos al centro de IoT Edge. Después, los eventos se enrutan al nodo del procesador de la puerta de señales a través del nodo de [origen de mensajes de IoT Hub](pipeline.md#iot-hub-message-source). El comportamiento del nodo del procesador de la puerta de señales es el mismo que el del caso de uso anterior; cuando lo desencadene un evento, se abrirá y permitirá que la fuente de vídeo en directo pase del nodo de origen RTSP al nodo receptor de archivos. Cada vez que se abre la puerta, se escribe un nuevo archivo MP4 en el almacenamiento local del dispositivo IoT Edge.

### <a name="video-recording-based-on-an-external-inferencing-module"></a>Grabación de vídeo basada en un módulo de inferencia externo 

En este caso de uso, puede grabar vídeo basado en una señal de un sistema lógico externo. Un ejemplo de este tipo de caso de uso podría ser la grabación de vídeo en la nube solo cuando se detecta un camión en la fuente de vídeo del tráfico en una autopista. En el diagrama siguiente se muestra una representación gráfica de una canalización que aborda este caso de uso. La representación JSON de la topología de canalización de este tipo de canalización [se puede encontrar aquí](https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/evr-hubMessage-video-sink/topology.json).

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording/external-inferencing-module.png" alt-text="Grabación basada en eventos de vídeo en directo cuando lo señala un módulo de inferencia externo.":::

En el diagrama, el nodo de origen RTSP captura la fuente de vídeo en directo de la cámara y la entrega a dos ramas: una tiene un nodo del [procesador de la puerta de señales](pipeline.md#signal-gate-processor) y la otra usa un nodo de [extensión HTTP](pipeline.md#http-extension-processor) para enviar datos a un módulo lógico externo. El nodo de extensión HTTP permite que la canalización envíe fotogramas de imagen (en formato JPEG, BMP o PNG) a un servicio de inferencia externo a través de REST. Normalmente, esta ruta de señal solo admite velocidades de fotogramas bajas (<5 fps). Puede usar el nodo del procesador de la extensión HTTP para reducir la velocidad de fotogramas del vídeo que pasa al módulo de inferencias externo.

Los resultados del servicio de inferencia externa se recuperan mediante el nodo de extensión HTTP y se retransmiten a centro de IoT Edge a través del nodo receptor de mensajes de IoT Hub, donde el módulo lógico externo puede procesarlos más. Por ejemplo, si el servicio de inferencia es capaz de detectar vehículos, el módulo lógico podría buscar un vehículo específico, como un camión. Cuando el módulo lógico detecta el objeto de interés, puede desencadenar el nodo del procesador de la puerta de señales mediante el envío de un evento a través del centro de IoT Edge al nodo de origen de mensajes de IoT Hub en la canalización. La salida de la puerta de señales se muestra para ir a un nodo receptor de vídeo. Cada vez que se detecta un camión, el vídeo se graba en la nube (se anexa al recurso de vídeo).

Una mejora de este ejemplo es usar un procesador de detección de movimiento antes del nodo del procesador de la extensión HTTP. Esto reducirá la carga en el servicio de inferencia, como por la noche, cuando hay períodos de tiempo prolongados en los que no pasan vehículos por la autopista. 

## <a name="next-steps"></a>Pasos siguientes

[Tutorial: grabación de vídeo basada en eventos](record-event-based-live-video.md)
