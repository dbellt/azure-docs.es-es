---
title: Cuotas y limitaciones de Azure Live Video Analytics on IoT Edge - Azure
description: En este artículo se describen las cuotas y limitaciones de Live Video Analytics en IoT Edge.
ms.topic: conceptual
ms.date: 05/22/2020
ms.openlocfilehash: 02347b5ae963008b3f8dd256afe2d8c264245650
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110378427"
---
# <a name="quotas-and-limitations-in-live-video-analytics"></a>Cuotas y limitaciones en Live Video Analytics

[!INCLUDE [redirect to Azure Video Analyzer](./includes/redirect-video-analyzer.md)]

En este artículo se enumeran las cuotas y limitaciones del módulo Live Video Analytics en IoT Edge.

## <a name="maximum-period-of-disconnected-use"></a>Período máximo de uso sin conexión

El módulo perimetral puede admitir la pérdida temporal de conectividad a Internet. Si el módulo permanece desconectado durante más de 36 horas, desactivará las instancias de grafos que estaban en ejecución. Además, se bloquearán las demás llamadas de método directo.

Para reanudar el estado operativo del módulo perimetral, tendrá que restaurar la conectividad a Internet, de modo que el módulo sea capaz de comunicarse correctamente con la cuenta de Azure Media Services.

## <a name="maximum-number-of-graph-instances"></a>Número máximo de instancias de grafos

Se admite un máximo de 1000 instancias de grafo por módulo (creadas a través de GraphInstanceSet).

## <a name="maximum-number-of-graph-topologies"></a>Número máximo de topologías de grafo

Se admite un máximo de 50 topologías de grafo por módulo (creadas a través de GraphTopologySet).

## <a name="limitations-on-graph-topologies-at-preview"></a>Limitaciones de las topologías de grafo en versión preliminar

Con la versión preliminar, hay limitaciones sobre los diferentes nodos que se pueden conectar entre sí en una topología de grafo multimedia.

* Origen RTSP
   * Solo se permite un origen RTSP por topología de grafo.
* Procesador de detección de movimiento
   * Debe estar inmediatamente después que el origen RTSP.
   * No se puede usar antes que un procesador de extensión HTTP o gRPC.
* Procesador de la puerta de señales
   * Debe estar inmediatamente después que el origen RTSP.
* Receptor de recursos 
   * Debe estar inmediatamente después del origen RTSP o el procesador de la puerta de señales.
* Receptor de archivos
   * Debe estar inmediatamente después del procesador de la puerta de señales.
   * No puede estar inmediatamente después de un procesador de extensión HTTP o gRPC, ni de un procesador de detección de movimiento.
* Receptor de IoT Hub
   * No puede estar inmediatamente después de un origen de IoT Hub.

## <a name="limitations-on-media-service-operations-at-preview"></a>Limitaciones de las operaciones de Media Services en versión preliminar

En la versión preliminar, Live Video Analytics en IoT Edge no admite lo siguiente:

* La capacidad de migrar la cuenta de Media Services de una suscripción a otra sin interrupción.
* La capacidad de usar más de una cuenta de almacenamiento con la cuenta de Media Services.
* La capacidad de cambiar de forma dinámica la información de la entidad de servicio en las propiedades deseadas del módulo, sin necesidad de reiniciar.

Solo puede usar cámaras IP que admitan el protocolo RTSP. Puede encontrar cámaras IP compatibles con RTSP en la página de [productos compatibles con ONVIF](https://www.onvif.org/conformant-products). Busque dispositivos que cumplan con los perfiles G, S o T.

Además, debe configurar estas cámaras para usar vídeo H.264 y audio AAC. Actualmente no se admiten otros códecs. 

## <a name="next-steps"></a>Pasos siguientes

[Información general](overview.md)
