---
title: 'Análisis del vídeo en directo sin grabación: Azure'
description: Se puede usar una tipología de canalización para extraer análisis a partir de una secuencia de vídeo en directo sin tener que grabarlo en el borde ni en la nube. En este artículo se describe este concepto.
ms.topic: conceptual
ms.date: 03/27/2021
ms.openlocfilehash: af3c1438814a77a4687c2fdd58e3a6e6f9589b32
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386464"
---
# <a name="analyzing-live-videos-without-recording"></a>Análisis de vídeos en directo sin grabación

## <a name="suggested-pre-reading"></a>Sugerencias antes de la lectura 

* [Concepto de canalización](pipeline.md)
* [Concepto de la extensión de canalización](pipeline-extension.md)
* [Concepto de grabación de vídeo basada en eventos](event-based-video-recording-concept.md)

## <a name="overview"></a>Información general  

Puede usar una topología de canalización para analizar vídeo en directo, sin grabar partes del vídeo en un archivo o recurso. Las topologías de canalización que se muestran a continuación son similares a las del artículo sobre [grabación de vídeo basada en eventos](event-based-video-recording-concept.md), pero sin un nodo receptor de vídeos o de archivos.

### <a name="motion-detection"></a>Detección de movimiento

La topología de canalización que se muestra a continuación consta de un nodo [origen RTSP](pipeline.md#rtsp-source), un nodo [procesador de detección de movimiento](pipeline.md#motion-detection-processor) y un nodo [receptor de mensajes de IoT Hub](pipeline.md#iot-hub-message-sink); puede ver la configuración utilizada en su [representación JSON](https://github.com/Azure/video-analyzer/blob/main/pipelines/live/topologies/motion-detection/topology.json). Esta topología permite detectar movimiento en la secuencia de vídeo entrante en directo y retransmitir los eventos de movimiento a otras aplicaciones y servicios mediante el nodo receptor de mensajes de IoT Hub. Las aplicaciones o servicios externos pueden desencadenar una alerta o enviar una notificación al personal adecuado.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/analyze-live-video-without-recording/motion-detection.svg" alt-text="Detección de movimiento en un vídeo en directo":::

### <a name="analyzing-video-using-a-custom-vision-model"></a>Analizar vídeo con un modelo de visión personalizado 

La topología de canalización que se muestra a continuación le permite analizar una secuencia de vídeo en directo mediante un modelo de Custom Vision empaquetado en un módulo independiente. Puede ver la configuración que se usa en su [representación JSON](https://github.com/Azure/video-analyzer/blob/main/pipelines/live/topologies/httpExtension/topology.json). Hay disponibles otros [ejemplos](https://github.com/Azure/video-analyzer/tree/main/edge-modules/extensions) para encapsular modelos en módulos IoT Edge que se ejecutan como servicios de inferencia.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/analyze-live-video-without-recording/motion-detected-frames.svg" alt-text="Análisis de un vídeo en directo con un modelo de Custom Vision":::

En esta topología de canalización, la entrada de vídeo del origen RTSP se envía a un nodo [procesador de extensiones HTTP](pipeline.md#http-extension-processor), que envía fotogramas de imagen (en formato JPEG, BMP o PNG) a un servicio de inferencia externo a través de REST. Los resultados del servicio de inferencia externa se recuperan mediante el nodo de extensión HTTP y se retransmiten al centro de IoT Edge mediante el nodo receptor de mensajes de IoT Hub. Este tipo de topología de canalización se puede usar para crear soluciones para una variedad de escenarios, como comprender la distribución de una serie temporal de vehículos en una intersección, comprender el patrón de tráfico de consumidores en una tienda, etc.

>[!TIP]
> Puede administrar la velocidad de fotogramas dentro del nodo del procesador de la extensión HTTP mediante el campo `samplingOptions` antes de enviarlo hacia abajo.

Una mejora de este ejemplo es usar un procesador de detección de movimiento antes del nodo del procesador de la extensión HTTP. Esto reducirá la carga en el servicio de inferencia, ya que solo se usa cuando hay actividad de movimiento en el vídeo.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/analyze-live-video-without-recording/custom-model.svg" alt-text="Análisis de vídeo en directo mediante un módulo de Custom Vision para fotogramas con movimiento":::

## <a name="next-steps"></a>Pasos siguientes

[Inicio rápido: Análisis de vídeo en directo con su propio modelo - HTTP](analyze-live-video-use-your-model-http.md)
