---
title: Terminología de Azure Video Analyzer
description: En este artículo se proporciona información general sobre la terminología de Azure Video Analyzer.
ms.service: azure-video-analyzer
ms.topic: conceptual
ms.date: 05/10/2021
ms.openlocfilehash: b29e2c788654a5445c0e14c00ad64690c405b003
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388636"
---
# <a name="azure-video-analyzer-terminology"></a>Terminología de Azure Video Analyzer

En este artículo se proporciona información general sobre la terminología relacionada con [Azure Video Analyzer](overview.md).

## <a name="pipeline-topology"></a>Topología de canalización

Una [topología de canalización](pipeline.md) le permite definir dónde se debe capturar el elemento multimedia, cómo se debe procesar y dónde se deben entregar los resultados. Permite definir una canalización que consta de orígenes, procesadores y nodos receptores con la que puede crear aplicaciones de análisis de vídeo en vivo. 

## <a name="rtsp"></a>RTSP

[RTSP](https://tools.ietf.org/html/rfc2326) hace referencia al Protocolo de streaming en tiempo real. Este es un protocolo de nivel de aplicación para controlar la entrega de datos con propiedades en tiempo real. RTSP proporciona un marco extensible para habilitar la entrega controlada a petición de datos en tiempo real, como audio y vídeo. Video Analyzer [admite](pipeline.md#rtsp-source) la captura, el análisis y la grabación de vídeo desde cámaras IP que admitan RTSP.


## <a name="recording"></a>Grabando

En el contexto de un sistema de administración de vídeo para cámaras de seguridad, la grabación de vídeo hace referencia al proceso de capturar vídeo desde las cámaras y almacenarlo para verlo posteriormente mediante aplicaciones móviles y de explorador. La grabación de vídeo se puede clasificar en [grabación continua de vídeo](continuous-video-recording.md) y [grabación de vídeo basada en eventos](event-based-video-recording-concept.md). Ambas categorías se explican con más detalle en la página de conceptos sobre la [grabación de vídeo](video-recording.md).

## <a name="video"></a>Vídeo

Al crear una cuenta de Video Analyzer, tiene que asociar una cuenta de Azure Storage a ella. Puede usar Video Analyzer para grabar el vídeo en directo desde una cámara y conservar los datos en el almacenamiento en disco o en la nube. En el último caso, los datos se almacenan como blobs en un contenedor de la cuenta de almacenamiento. Los vídeos son recursos de la cuenta de Video Analyzer que se asignan a estos contenedores de blobs. Todo el contenido asociado a estos recursos de vídeo se almacena como blobs en los contenedores correspondientes, mientras que Video Analyzer contiene los metadatos (por ejemplo, nombre, descripción u hora de creación).

Puede usar Video Analyzer para crear recursos de vídeo y agregar datos a los vídeos existentes. Esto posibilita escenarios de grabación y reproducción de vídeo continua y basada en eventos (con la captura de vídeo en el dispositivo perimetral, la grabación en Video Analyzer y la reproducción mediante las funcionalidades de streaming de Video Analyzer).

## <a name="grpc"></a>gRPC

[gRPC](https://grpc.io/docs/guides/) es un marco de llamada a procedimiento remoto (RPC) de alto rendimiento e independiente del idioma. Usa esquemas estructurados basados en sesiones a través de [búferes de protocolo 3](https://developers.google.com/protocol-buffers/docs/proto3) como su formato de intercambio de mensajes subyacente para la comunicación.

## <a name="streaming"></a>Streaming

Puede usar Video Analyzer para transmitir grabaciones de vídeo a los clientes mediante protocolos de streaming multimedia estándar del sector basados en HTTP, como [HTTP Live Streaming (HLS)](https://developer.apple.com/streaming/) y [MPEG-DASH](https://dashif.org/about/). Puede usar los [widgets del reproductor de Azure Video Analyzer](https://github.com/Azure/video-analyzer/blob/main/widgets/readme.md) (componentes web) para reproducir recursos de vídeo. Además, HLS es compatible con reproductores web como [JW Player](https://www.jwplayer.com/), [hls.js](https://github.com/video-dev/hls.js/), [VideoJS](https://videojs.com/) y [Shaka Player, de Google](https://github.com/google/shaka-player), o bien se puede representar de forma nativa en aplicaciones móviles con [Exoplayer](https://github.com/google/ExoPlayer) para Android y [AV Foundation](https://developer.apple.com/av-foundation/) para iOS. De forma similar, MPEG-DASH es compatible con [la lista de clientes de esta página](https://dashif.org/clients/).

## <a name="vms"></a>VMS

[VMS](https://en.wikipedia.org/wiki/Video_management_system) hace referencia a un sistema de administración de vídeo. Estos sistemas se usan para configurar y controlar cámaras de vigilancia, así como para capturar y grabar vídeos de ellas. Estos sistemas también proporcionan aplicaciones cliente para reproducir el vídeo grabado.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre [canalizaciones](pipeline.md).
