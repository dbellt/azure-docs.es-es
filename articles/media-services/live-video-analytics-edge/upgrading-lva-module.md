---
title: Actualización de Live Video Analytics en IoT Edge de la versión 1.0 a la 2.0
description: En este artículo se describen las diferencias y las distintas cosas que hay que tener en cuenta al actualizar el módulo de Live Video Analytics (LVA) en Azure IoT Edge.
author: naiteeks
ms.topic: how-to
ms.author: naiteeks
ms.date: 12/14/2020
ms.openlocfilehash: 2ade7eca6f9e4e41641de7470445fc9ffd43e0a2
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110368498"
---
# <a name="upgrading-live-video-analytics-on-iot-edge-from-10-to-20"></a>Actualización de Live Video Analytics en IoT Edge de la versión 1.0 a la 2.0

[!INCLUDE [redirect to Azure Video Analyzer](./includes/redirect-video-analyzer.md)]

En este artículo se describen las diferencias y las distintas cosas que hay que tener en cuenta al actualizar el módulo de Live Video Analytics (LVA) en Azure IoT Edge.

## <a name="change-list"></a>Lista de cambios

> [!div class="mx-tdCol4BreakAll"]
> |Title|Live Video Analytics 1.0|Live Video Analytics 2.0|Descripción|
> |-------------|----------|---------|---------|
> |Imagen de contenedor|mcr.microsoft.com/media/live-video-analytics:1|mcr.microsoft.com/media/live-video-analytics:2|Microsoft publicó imágenes de Docker para Live Video Analytics en Azure IoT Edge|
> |**Nodos de MediaGraph** |    |   |   |
> |Orígenes|:::image type="icon" source="./././media/upgrading-lva/check.png"::: Origen RTSP </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Origen de mensajes de IoT Hub |:::image type="icon" source="./././media/upgrading-lva/check.png"::: Origen RTSP </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Origen de mensajes de IoT Hub | Nodos de MediaGraph que actúan como orígenes para la ingesta de elementos multimedia y los mensajes.|
> |Procesadores|:::image type="icon" source="./././media/upgrading-lva/check.png"::: Procesador de detección de movimiento </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Procesador de filtros de velocidad de fotogramas </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Procesador de extensiones HTTP </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Procesador de extensiones gRPC </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Procesador de la puerta de señales |:::image type="icon" source="./././media/upgrading-lva/check.png"::: Procesador de detección de movimiento </br>:::image type="icon" source="./././media/upgrading-lva/remove.png"::: **Procesador de filtros de velocidad de fotogramas**</br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Procesador de extensiones HTTP </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Procesador de extensiones gRPC </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Procesador de la puerta de señales | Nodos de MediaGraph que permiten formatear los elementos multimedia antes de enviarlos a los servidores de inferencia de IA.|
> |Receptores|:::image type="icon" source="./././media/upgrading-lva/check.png"::: Receptor de recursos </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Receptor de archivos </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Receptor de mensajes de IoT Hub|:::image type="icon" source="./././media/upgrading-lva/check.png"::: Receptor de recursos </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Receptor de archivos </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Receptor de mensajes de IoT Hub| Nodos de MediaGraph que permiten almacenar los elementos multimedia procesados.|
> |**Nodos de MediaGraph admitidos** |    |   |   |
> |Topologías|:::image type="icon" source="./././media/upgrading-lva/check.png"::: Grabación continua de vídeo </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Análisis de movimiento y grabación continua de vídeo </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Análisis externo y grabación continua de vídeo </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Grabación basada en eventos en movimiento </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Grabación basada en eventos con IA</br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Grabación basada en eventos externos </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Análisis de movimiento </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Análisis de movimiento seguido de inferencia de IA |:::image type="icon" source="./././media/upgrading-lva/check.png"::: Grabación continua de vídeo </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Análisis de movimiento y grabación continua de vídeo </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Análisis externo y grabación continua de vídeo </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Grabación basada en eventos en movimiento </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Grabación basada en eventos con IA</br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Grabación basada en eventos externos </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Análisis de movimiento </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Análisis de movimiento seguido de inferencia de IA </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: **Composición de IA** </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: **Grabación de audio y vídeo** </br>  | Topologías de MediaGraph que permiten definir el plano técnico de un grafo, con parámetros como marcadores de posición para los valores.|

## <a name="upgrading-the-live-video-analytics-on-iot-edge-module-from-10-to-20"></a>Actualización del módulo de Live Video Analytics en IoT Edge de la versión 1.0 a la 2.0
Al actualizar el módulo de Live Video Analytics en Azure IoT Edge, asegúrese de actualizar la información siguiente.
### <a name="container-image"></a>Imagen de contenedor
En la plantilla de implementación, busque su módulo de Live Video Analytics en IoT Edge en el nodo `modules` y actualice el campo `image` de la siguiente forma:
```
"image": "mcr.microsoft.com/media/live-video-analytics:2"
```
> [!TIP]
> Si no ha modificado el nombre del módulo de Live Video Analytics en IoT Edge, busque `lvaEdge` en el nodo del módulo.

### <a name="topology-file-changes"></a>Cambios en los archivos de topología
En los archivos de topología, asegúrese de que **`apiVersion`** está establecido en 2.0.

### <a name="mediagraph-node-changes"></a>Cambios en los nodos de MediaGraph


* Ahora se puede pasar el audio desde el origen de la cámara a niveles inferiores junto con el vídeo. Puede pasar **solo audio** o **solo vídeo**, o **audio y vídeo** con la ayuda de **`outputSelectors`** a cualquier nodo de grafo. Por ejemplo, si desea seleccionar solo vídeo desde el origen RTSP y pasarlo a niveles inferiores, agregue lo siguiente al nodo de origen RTSP:
    ```
    "outputSelectors": [
      {
        "property": "mediaType",
        "operator": "is",
        "value": "video"
      }
    ```
>[!NOTE]
>**`outputSelectors`** es una propiedad opcional. Si no se usa, el grafo de elementos multimedia pasará el audio (si está habilitado) y el vídeo desde la cámara RTSP a niveles inferiores. 

* En los procesadores `MediaGraphHttpExtension` y `MediaGraphGrpcExtension`, tenga en cuenta los siguientes cambios:  
    #### <a name="image-properties"></a>Propiedades de la imagen
    * `MediaGraphImageFormatEncoded` ya no se admite. 
      * En su lugar, use **`MediaGraphImageFormatBmp`** o **`MediaGraphImageFormatJpeg`** o **`MediaGraphImageFormatPng`** . Por ejemplo,
        ```
        "image": {
                "scale": 
                {
                    "mode": "preserveAspectRatio",
                    "width": "416",
                    "height": "416"
                },
                "format": 
                {
                    "@type": "#Microsoft.Media.MediaGraphImageFormatJpeg"
                }
            }
        ```
        * Si desea usar imágenes sin procesar, use **`MediaGraphImageFormatRaw`** . Para usar esta opción, actualice el nodo de imagen de la siguiente forma:
        ```
        "image": {
                "scale": 
                {
                    "mode": "preserveAspectRatio",
                    "width": "416",
                    "height": "416"
                },
                "format": 
                {
                    "@type": "#Microsoft.Media.MediaGraphImageFormatRaw",
                    "pixelFormat": "rgba"
                }
            }
        ```
        >[!NOTE]
        > Los valores posibles de pixelFormat son, entre otros: `yuv420p`,`rgb565be`, `rgb565le`, `rgb555be`, `rgb555le`, `rgb24`, `bgr24`, `argb`, `rgba` `abgr` y `bgra`.  

    #### <a name="extensionconfiguration-for-grpc-extension-processor"></a>extensionConfiguration para el procesador de extensión de gRPC  
    * En el procesador `MediaGraphGrpcExtension`, hay una nueva propiedad denominada **`extensionConfiguration`** disponible, que es una cadena opcional que se puede usar como parte del contrato de gRPC. Este campo se puede utilizar para pasar los datos al servidor de inferencia, y definir la forma en que el servidor de inferencia usa esos datos.  
    Un caso de uso de esta propiedad es cuando se tienen varios modelos de IA empaquetados en un único servidor de inferencia. Con esta propiedad, no es necesario exponer un nodo para cada modelo de IA. En lugar de ello, en el caso de una instancia de grafo, el proveedor de la extensión (usted), puede definir cómo seleccionar los diferentes modelos de IA mediante la propiedad **`extensionConfiguration`** y, durante la ejecución, LVA pasará esta cadena al servidor de inferencia, que puede usarla para invocar el modelo de IA deseado.  

    #### <a name="ai-composition"></a>Composición de IA
    * Live Video Analytics 2.0 admite ahora el uso de más de un procesador de extensión de grafos de elementos multimedia dentro de una topología. Puede pasar los fotogramas multimedia de la cámara RTSP a diferentes modelos de IA secuencialmente, en paralelo o en una combinación de ambos. Vea una topología de ejemplo que muestra dos modelos de IA que se usan de forma secuencial.

### <a name="disk-space-management-with-sink-nodes"></a>Administración del espacio en disco con nodos receptores
* En el nodo **receptor de archivos**, ahora puede especificar el espacio en disco que puede usar el módulo de Live Video Analytics en IoT Edge para almacenar las imágenes procesadas. Para ello, agregue el campo **`maximumSizeMiB`** al nodo receptor de archivos. Este es un nodo receptor de archivos de ejemplo:
    ```
    "sinks": [
      {
        "@type": "#Microsoft.Media.MediaGraphFileSink",
        "name": "fileSink",
        "inputs": [
          {
            "nodeName": "signalGateProcessor",
            "outputSelectors": [
              {
                "property": "mediaType",
                "operator": "is",
                "value": "video"
              }
            ]
          }
        ],
        "fileNamePattern": "sampleFiles-${System.DateTime}",
        "maximumSizeMiB":"512",
        "baseDirectoryPath":"/var/media"
      }
    ]
    ```
* En el nodo **receptor de recursos**, puede especificar el espacio en disco que puede usar el módulo de Live Video Analytics en IoT Edge para almacenar las imágenes procesadas. Para ello, agregue el campo **`localMediaCacheMaximumSizeMiB`** al nodo receptor de recursos. Este es un nodo receptor de recursos de ejemplo:
    ```
    "sinks": [
      {
        "@type": "#Microsoft.Media.MediaGraphAssetSink",
        "name": "AssetSink",
        "inputs": [
          {
            "nodeName": "signalGateProcessor",
            "outputSelectors": [
              {
                "property": "mediaType",
                "operator": "is",
                "value": "video"
              }
            ]
          }
        ],
        "assetNamePattern": "sampleAsset-${System.GraphInstanceName}",
        "segmentLength": "PT30S",
        "localMediaCacheMaximumSizeMiB":"200",
        "localMediaCachePath":"/var/lib/azuremediaservices/tmp/"
      }
    ]
    ```
    >[!NOTE]
    >  La ruta de acceso del **receptor de archivos** se divide en el patrón de nombre de archivo y la ruta de acceso del directorio base, mientras que la ruta de acceso del **receptor de recursos** incluye la ruta de acceso del directorio base.  

### <a name="frame-rate-management"></a>Administración de la velocidad de fotogramas
* **`MediaGraphFrameRateFilterProcessor`** está en desuso en el **módulo de Live Video Analytics en IoT Edge 2.0**.
    * Para muestrear el vídeo entrante para su procesamiento, agregue la propiedad **`samplingOptions`** a los procesadores de extensión de MediaGraph (`MediaGraphHttpExtension` o `MediaGraphGrpcExtension`).  
     ```
          "samplingOptions": 
          {
            "skipSamplesWithoutAnnotation": "false",  // If true, limits the samples submitted to the extension to only samples which have associated inference(s) 
            "maximumSamplesPerSecond": "20"   // Maximum rate of samples submitted to the extension
          },
    ```
### <a name="module-metrics-in-the-prometheus-format-using-telegraf"></a>Métricas de módulo en el formato Prometheus con Telegraf
Con esta versión, se puede usar Telegraf para enviar métricas a Azure Monitor. Desde esa plataforma, las métricas pueden dirigirse a Log Analytics o a un centro de eventos.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/telemetry-schema/telegraf.png" alt-text="Taxonomía de eventos":::

Puede generar una imagen de Telegraf con una configuración personalizada fácilmente mediante Docker. Obtenga más información en la página [Supervisión y registro](monitoring-logging.md#azure-monitor-collection-via-telegraf).

## <a name="next-steps"></a>Pasos siguientes

[Introducción a Live Video Analytics en IoT Edge](get-started-detect-motion-emit-events-quickstart.md)