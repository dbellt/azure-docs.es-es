---
title: 'Notas de la versión de Azure Live Video Analytics en IoT Edge: Azure'
description: En este tema se proporcionan las notas de la versión de Azure Live Video Analytics on IoT Edge, que incluyen las mejoras, las correcciones de errores, las versiones y los problemas conocidos de este servicio.
ms.topic: conceptual
ms.date: 08/19/2020
ms.openlocfilehash: 67b30cfbcb374dbe8773023f67b77ef43b35f3ba
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110372052"
---
# <a name="azure-live-video-analytics-on-iot-edge-release-notes"></a>Notas de la versión de Azure Live Video Analytics on IoT Edge

[!INCLUDE [redirect to Azure Video Analyzer](./includes/redirect-video-analyzer.md)]

>Reciba notificaciones para volver a visitar esta página y obtener actualizaciones; para ello, copie y pegue esta URL (`https://docs.microsoft.com/api/search/rss?search=%22Live+Video+Analytics+on+IoT+Edge+release+notes%22&locale=en-us`) en el lector de fuentes RSS.

En este artículo se proporciona información acerca de lo siguiente:

* Versiones más recientes
* Problemas conocidos
* Corrección de errores
* Funciones obsoletas

<hr width=100%>

## <a name="january-12-2021"></a>12 de enero de 2021

La etiqueta de esta versión de la actualización de enero de 2021 del módulo es:

```
mcr.microsoft.com/media/live-video-analytics:2.0.1
```

> [!NOTE]
> En los inicios rápido y tutoriales, los manifiestos de implementación usan una etiqueta 2 (live-video-analytics:2). Por lo tanto, con solo repetir la implementación de tales manifiestos se actualizará el módulo en los dispositivos perimetrales.
### <a name="bug-fixes"></a>Corrección de errores 

* Los campos `ActivationSignalOffset`, `MinimumActivationTime` y `MaximumActivationTime` de los procesadores de Signal Gate se establecieron incorrectamente como propiedades obligatorias. Ahora son propiedades **opcionales**.
* Se corrigió un error de uso que hacía que el módulo Live Video Analytics en IoT Edge se bloqueara cuando se implementaba en determinadas regiones.

<hr width=100%>

## <a name="december-14-2020"></a>14 de diciembre de 2020
Esta versión es la versión preliminar pública actualizada de Live Video Analytics on IoT Edge. La etiqueta de versión es la siguiente:

```
     mcr.microsoft.com/media/live-video-analytics:2.0.0
```
### <a name="module-updates"></a>Actualizaciones del módulo
* Se ha agregado compatibilidad con el uso de más de un procesador de extensión HTTP y un procesador de extensión gRPC por topología de grafos.
* Se ha agregado compatibilidad para la [administración del espacio en disco para los nodos receptores](upgrading-lva-module.md#disk-space-management-with-sink-nodes).
* El nodo `MediaGraphGrpcExtension` ahora admite la propiedad [extensionConfiguration](grpc-extension-protocol.md) para usar varios modelos de IA en un único servidor gRPC.
* Se ha agregado compatibilidad con la recopilación de métricas del módulo Live Video Analytics en el [formato Prometheus](https://prometheus.io/docs/practices/naming/). Obtenga más información sobre cómo [recopilar métricas y visualizarlas en Azure Monitor.](monitoring-logging.md#azure-monitor-collection-via-telegraf) 
* Se ha agregado la posibilidad de filtrar la selección de salida. Puede pasar **solo audio**, **solo vídeo** o **audio y vídeo** con la ayuda de `outputSelectors` a cualquier nodo de grafo. 
* El procesador de filtros de velocidad de fotogramas está **en desuso**.  
    * La administración de la velocidad de fotogramas ahora está disponible en los nodos mismos del procesador de extensión de grafos.

### <a name="visual-studio-code-extension"></a>Extensión de Visual Studio Code
* Se publicó [Live Video Analytics en IoT Edge](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.live-video-analytics-edge), una extensión de Visual Studio Code para ayudarle a administrar grafos multimedia de LVA. Esta extensión funciona con el **módulo LVA 2.0** y ofrece la edición y administración de grafos multimedia con una interfaz gráfica elegante y fácil de usar.
## <a name="september-22-2020"></a>22 de septiembre de 2020

La etiqueta de esta versión para la actualización de septiembre de 2020 del módulo es:

```
mcr.microsoft.com/media/live-video-analytics:1.0.4
```

> [!NOTE]
> En los inicios rápido y tutoriales, los manifiestos de implementación usan una etiqueta de 1 (live-video-analytics:1). Por lo tanto, con solo repetir la implementación de tales manifiestos se actualizará el módulo en los dispositivos perimetrales.

### <a name="module-updates"></a>Actualizaciones del módulo

* Hay un nuevo nodo de extensión de grafos, [MediaGraphCognitiveServicesVisionExtension](spatial-analysis-tutorial.md), disponible para integrarse con el módulo [Spatial Analysis](/legal/cognitive-services/computer-vision/intro-to-spatial-analysis-public-preview) (versión preliminar) de Cognitive Services.
* Se ha agregado compatibilidad con dispositivos ARM64 de Linux: use [pasos manuales](deploy-iot-edge-device.md) para implementar en esos dispositivos.

### <a name="documentation-updates"></a>Actualizaciones de la documentación

* Hay [instrucciones](deploy-azure-stack-edge-how-to.md) disponibles para usar Live Video Analytics en IoT Edge en dispositivos Azure Stack Edge.
* Nuevo tutorial sobre el desarrollo de modelos de visión de proceso específicos de cada escenario mediante el [servicio Custom Vision](https://azure.microsoft.com/services/cognitive-services/custom-vision-service/) y su uso para [analizar vídeo en vivo](custom-vision-tutorial.md) en tiempo real.

<hr width=100%>

## <a name="august-19-2020"></a>19 de agosto de 2020

La etiqueta de esta versión de la actualización de agosto de 2020 del módulo es:

```
mcr.microsoft.com/media/live-video-analytics:1.0.3
```

> [!NOTE]
> En los inicios rápido y tutoriales, los manifiestos de implementación usan una etiqueta de 1 (live-video-analytics:1). Por lo tanto, con solo repetir la implementación de tales manifiestos se actualizará el módulo en los dispositivos perimetrales.

### <a name="module-updates"></a>Actualizaciones del módulo

* Ahora puede obtener un alto rendimiento de la transferencia de contenido de datos entre Live Video Analytics en IoT Edge y su extensión personalizada mediante el marco de gRPC. Consulte el [inicio rápido](analyze-live-video-use-your-grpc-model-quickstart.md) para comenzar.
* Implementación regional más amplia de Live Video Analytics y solo se ha actualizado el servicio en la nube.  
* Live Video Analytics ahora está disponible en 25 regiones más de todo el mundo. A continuación, se muestra la [lista](https://azure.microsoft.com/global-infrastructure/services/?products=media-services) de todas las regiones disponibles.  
* La [configuración](https://aka.ms/lva-edge/setup-resources-for-samples) de los inicios rápidos se actualizó también con compatibilidad con regiones nuevas.
    * No hay ninguna llamada a la acción para ningún usuario que ya haya configurado los recursos.

### <a name="bug-fixes"></a>Corrección de errores 

* Eliminación del uso de una extensión de Azure en desuso en el script de configuración.

<hr width=100%>

## <a name="july-13-2020"></a>13 de julio de 2020

La etiqueta de esta versión de la actualización de julio de 2020 del módulo es:

```
mcr.microsoft.com/media/live-video-analytics:1.0.2
```

> [!NOTE]
> En los inicios rápido y tutoriales, los manifiestos de implementación usan una etiqueta de 1 (live-video-analytics:1). Por lo tanto, con solo repetir la implementación de tales manifiestos se actualizará el módulo en los dispositivos perimetrales.

### <a name="module-updates"></a>Actualizaciones del módulo

* Ahora puede crear topologías de grafos que tengan un nodo receptor de recursos y un nodo receptor de archivos de nivel inferior de un nodo procesador de puerta de señal. Consulte [la topología](https://github.com/Azure/live-video-analytics/tree/master/MediaGraph/topologies/evr-motion-assets-files) para obtener un ejemplo.

### <a name="bug-fixes"></a>Corrección de errores

* Mejoras en la validación de las propiedades deseadas

<hr width=100%>

## <a name="june-1-2020"></a>1 de junio de 2020

Esta versión es la primera versión preliminar pública de Live Video Analytics on IoT Edge. La etiqueta de versión es la siguiente:

```
     mcr.microsoft.com/media/live-video-analytics:1.0.0
```

### <a name="supported-functionalities"></a>Funcionalidades admitidas

* Analice secuencias de vídeo en directo con los módulos de inteligencia artificial que prefiera y, de manera opcional, grabe vídeos en su dispositivo perimetral o en la nube.
* Use este módulo en sistemas operativos Linux AMD64 [compatibles](../../iot-edge/support.md) con IoT Edge.
* Implemente y configure el módulo a través de IoT Hub mediante Azure Portal o Visual Studio Code.
* Administre [topologías e instancias de grafos](media-graph-concept.md#media-graph-topologies-and-instances) de forma remota o local mediante la llamada a los siguientes métodos directos:

    *   GraphTopologyGet
    *   GraphTopologySet
    *   GraphTopologyDelete
    *   GraphTopologyList
    *   GraphInstanceGet
    *   GraphInstanceSet
    *   GraphInstanceDelete
    *   GraphInstanceList

## <a name="next-steps"></a>Pasos siguientes

[Información general](overview.md)
