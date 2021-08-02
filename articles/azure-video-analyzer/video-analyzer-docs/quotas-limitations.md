---
title: 'Cuotas y limitaciones de Azure Video Analyzer: Azure'
description: En este artículo se describen las cuotas y limitaciones de Azure Video Analyzer.
ms.service: azure-video-analyzer
ms.topic: conceptual
ms.date: 05/26/2021
ms.openlocfilehash: d4bc488411319832b59540fb1392b5d3259cef69
ms.sourcegitcommit: 1b698fb8ceb46e75c2ef9ef8fece697852c0356c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110652193"
---
# <a name="video-analyzer-quotas-and-limitations"></a>Cuotas y limitaciones de Azure Video Analyzer

En este artículo se describen las cuotas y limitaciones de Azure Video Analyzer.

## <a name="quotas-and-limitations---edge-module"></a>Cuotas y limitaciones: módulo perimetral

En esta sección se enumeran las cuotas y limitaciones del módulo perimetral de Video Analyzer.

### <a name="maximum-period-of-disconnected-use"></a>Período máximo de uso sin conexión

El módulo perimetral puede admitir la pérdida temporal de conectividad a Internet. Si el módulo permanece desconectado durante más de 36 horas, desactivará las canalizaciones en directo que estuvieran en ejecución. Además, se bloquearán las demás llamadas de método directo.

Para restaurar el estado operativo del módulo perimetral tendrá que restablecer la conectividad a Internet, de modo que el módulo sea capaz de comunicarse correctamente con la cuenta de Azure Video Analyzer.

### <a name="maximum-number-of-live-pipelines"></a>Número máximo de canalizaciones en directo

Se admite un máximo de 1000 canalizaciones en directo por cada módulo (creadas mediante `livePipelineSet`).

### <a name="maximum-number-of-pipeline-topologies"></a>Número máximo de topologías de canalización

Se admite un máximo de 50 topologías de canalización por cada módulo (creadas mediante `pipelineTopologySet`).

### <a name="limitations-on-pipeline-topologies"></a>Limitaciones de las topologías de canalización

A continuación se indican las limitaciones sobre cómo se pueden conectar nodos diferentes en una topología de canalización.

* Origen RTSP
   * Solo se permite un origen RTSP por cada topología de canalización.
* Procesador de detección de movimiento
   * Debe estar inmediatamente después que el origen RTSP.
   * No se puede usar después de un procesador de extensión HTTP o gRPC.
* Procesador de la puerta de señales
   * Debe estar inmediatamente después que el origen RTSP.
   * No se puede usar en elementos de nivel superior desde un protocolo HTTP o un procesador de extensiones gRPC.
* Procesador de seguimiento de objetos
   * Debe estar inmediatamente después de un procesador de extensión HTTP o gRPC. El procesador de extensiones no debe aplicar un modelo de IA en cada fotograma del vídeo en directo.
* Procesador de cruce de líneas
   * Debe estar inmediatamente después de un procesador de seguimiento de objetos.
* Receptor de vídeo 
   * Debe estar inmediatamente después del origen RTSP o el procesador de la puerta de señales.
* Receptor de archivos
   * Debe estar inmediatamente después del procesador de la puerta de señales.
   * No puede estar inmediatamente después de un procesador de extensión HTTP o gRPC, ni de un procesador de detección de movimiento.
* Receptor de IoT Hub
   * No puede estar inmediatamente después de un origen de IoT Hub.

### <a name="supported-cameras"></a>Cámaras compatibles
Solo puede usar cámaras IP que admitan el protocolo RTSP. Puede encontrar cámaras IP compatibles con RTSP en la página de [productos compatibles con ONVIF](https://www.onvif.org/conformant-products). Busque dispositivos que cumplan con los perfiles G, S o T.

Además, debe configurar estas cámaras para usar vídeo H.264 y audio AAC. Actualmente no se admiten otros códecs.

### <a name="support-for-video-ai"></a>Compatibilidad con IA de vídeo
Los procesadores de extensión HTTP o gRPC solo admiten el envío de datos de fotogramas de imagen o vídeo con un módulo de IA externo. Por lo tanto, no se admite la ejecución de la inferencia en datos de audio. Como resultado, los nodos de procesador de las topologías de canalización que tienen un nodo de origen RTSP como uno de los elementos `inputs` también usan la propiedad `outputSelectors` para asegurarse de que solo se pasa vídeo al procesador. Consulte esta [topología](https://github.com/Azure/video-analyzer/blob/main/pipelines/live/topologies/evr-grpcExtension-video-sink/topology.json) como ejemplo.

## <a name="quotas-and-limitations---service"></a>Cuotas y limitaciones: servicio

En esta sección se enumeran las cuotas y limitaciones de la cuenta de Video Analyzer.

### <a name="limitations-on-service-operations-at-preview"></a>Limitaciones de las operaciones del servicio en versión preliminar

En el momento del lanzamiento de la versión preliminar, el servicio Video Analyzer no admite lo siguiente:

* La capacidad de migrar la cuenta de una suscripción a otra sin una interrupción.
* La capacidad de usar más de una cuenta de almacenamiento con la cuenta.
* La capacidad de usar una cuenta de almacenamiento de una región diferente a la de la cuenta de Video Analyzer.

### <a name="limits-on-video-resources"></a>Límites de los recursos de vídeo
En el momento del lanzamiento de la versión preliminar, cada cuenta de Video Analyzer puede tener como máximo 10 000 recursos de vídeo. Si tiene una necesidad empresarial de un límite mayor, abra una incidencia de soporte técnico en Azure Portal.

### <a name="limits-on-access-policies"></a>Límites de las directivas de acceso
En el momento del lanzamiento de la versión preliminar, cada cuenta de Video Analyzer puede tener como máximo 20 directivas de acceso.

### <a name="limits-on-registered-edge-modules"></a>Límites de módulos perimetrales registrados
En el momento del lanzamiento de la versión preliminar, cada cuenta de Video Analyzer puede tener como máximo 1,000 módulos perimetrales registrados en ella. Si tiene una necesidad empresarial de un límite mayor, abra una incidencia de soporte técnico en Azure Portal.

### <a name="limits-on-client-api-calls"></a>Límites en las llamadas a la API de cliente
Se aplican los límites siguientes a las API de cliente en la versión preliminar:

* Hasta 50 solicitudes en 10 segundos
* Hasta 600 solicitudes por hora

## <a name="limitations---video-analyzer-player-widgets"></a>Limitaciones: widgets del reproductor de Video Analyzer

En el momento del lanzamiento de la versión preliminar, no se admite la reproducción en dispositivos iOS.

## <a name="next-steps"></a>Pasos siguientes

[Información general](overview.md)
