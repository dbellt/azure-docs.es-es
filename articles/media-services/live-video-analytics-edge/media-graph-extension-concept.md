---
title: 'Definición de la extensión de grafos de elementos multimedia de Live Video Analytics: Azure'
description: Azure Live Video Analytics en IoT Edge permite ampliar las funcionalidades de procesamiento de grafos multimedia mediante un nodo de extensión de grafos.
ms.topic: overview
ms.date: 09/14/2020
ms.openlocfilehash: 8f3165d948e636bda49128f080af00df7f30c977
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110372419"
---
# <a name="live-video-analytics-media-graph-extension"></a>Extensión de grafos de elementos multimedia de Live Video Analytics

[!INCLUDE [redirect to Azure Video Analyzer](./includes/redirect-video-analyzer.md)]

Live Video Analytics en IoT Edge permite ampliar las funcionalidades de procesamiento de grafos multimedia mediante un nodo de extensión de grafo. El complemento de extensión de análisis puede usar técnicas tradicionales de procesamiento de imágenes o modelos de inteligencia artificial de Computer Vision. Las extensiones de grafos se habilitan al incluir un nodo de procesador de extensiones en un grafo multimedia. El nodo de procesador de extensiones retransmite fotogramas de vídeo al punto de conexión configurado y actúa como la interfaz de la extensión. La conexión se puede realizar con un punto de conexión local o remoto y se puede proteger mediante autenticación y cifrado TLS, si fuera necesario. Además, el nodo procesador de extensiones de grafos permite el escalado y la codificación opcionales de los fotogramas de vídeo antes de enviarlos a la extensión personalizada. 

Live Video Analytics admite dos tipos de procesadores de extensión de grafos multimedia:

* [Procesador de extensiones HTTP](media-graph-concept.md#http-extension-processor)
* [Procesador de extensiones de gRPC](media-graph-concept.md#grpc-extension-processor)

El nodo de extensión de grafos espera que el complemento de extensión de análisis devuelva los resultados en formato JSON. Idealmente, los resultados deben seguir el [modelo de objetos del esquema de metadatos de inferencia ](./inference-metadata-schema.md).

## <a name="http-extension-processor"></a>Procesador de extensiones HTTP

El procesador de extensiones HTTP permite escenarios de extensibilidad mediante el [protocolo HTTP](./http-extension-protocol.md), donde el rendimiento o el uso óptimo de los recursos no son las principales preocupaciones. Puede exponer su propia IA a un grafo multimedia a través de un punto de conexión de REST HTTP. 

Use el nodo de procesador de extensiones HTTP cuando:

* Quiera una mejor interoperabilidad con los sistemas de inferencia de HTTP existentes.
* La transferencia de datos de bajo rendimiento es aceptable.
* Quiera usar una interfaz de solicitud-respuesta simple para Live Video Analytics.

## <a name="grpc-extension-processor"></a>Procesador de extensiones de gRPC

El procesador de extensiones de gRPC permite escenarios de extensibilidad mediante el uso de un [protocolo estructurado](./grpc-extension-protocol.md) de alto rendimiento basado en gRPC. Es ideal para escenarios en los que el rendimiento o la utilización óptima de los recursos es una prioridad. El procesador de extensiones de gRPC permite sacar el máximo partido de las definiciones de datos estructurados. gRPC ofrece rendimiento alto de transferencia de contenido mediante:

* la [memoria compartida integrada](https://en.wikipedia.org/wiki/Shared_memory) o 
* la inserción directa del contenido en el cuerpo de los mensajes gRPC. 

El procesador de extensiones de gRPC se puede usar para enviar propiedades multimedia junto con el intercambio de mensajes de inferencia.
Por lo tanto, use un nodo de procesador de extensiones de gRPC cuando:

* Quiera usar un contrato estructurado (por ejemplo, mensajes estructurados para solicitudes y respuestas).
* Quiera usar búferes de protocolo ([protobuf](https://developers.google.com/protocol-buffers)) como su formato de intercambio de mensajes subyacentes para la comunicación.
* Quiera comunicarse con un servidor de gRPC en una sesión de un solo flujo, en lugar del modelo de solicitud-respuesta tradicional que necesita un controlador de solicitud personalizado para analizar las solicitudes entrantes y llamar a las funciones de implementación correctas. 
* Quiera una comunicación de baja latencia y alto rendimiento entre Live Video Analytics y el módulo.

## <a name="use-your-inferencing-model-with-live-video-analytics"></a>Uso del modelo de inferencia con Live Video Analytics

Las extensiones de grafos multimedia le permiten ejecutar el modelo de inferencia que prefiera en cualquier runtime de inferencia disponible, como ONNX, TensorFlow o PyTorch, u otros en su propio contenedor de Docker. La extensión personalizada de inferencia debe implementarse junto con el módulo perimetral de Live Video Analytics para obtener el mejor rendimiento, y se invoca a través del procesador de extensiones HTTP o del procesador de extensiones de gRPC incluido en la topología de grafos. Además, la frecuencia de las llamadas a la extensión personalizada se puede limitar si se agrega un [procesador de detección de movimiento](media-graph-concept.md#motion-detection-processor) ascendentes al procesador de extensiones multimedia.

En el diagrama siguiente se muestra el flujo de datos general:

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/media-graph-extension/analyze-live-video-with-AI-inference-service.svg" alt-text="Servicio de inferencia de IA":::

## <a name="samples"></a>Ejemplos

Puede comenzar con uno de los inicios rápidos que ilustran Live Video Analytics con el servicio de extensión precompilado a velocidades de fotogramas bajas con el [procesador de extensiones HTTP](./use-your-model-quickstart.md?pivots=programming-language-csharp) o a velocidades de fotogramas altas con el [procesador de extensiones gRPC](./analyze-live-video-use-your-grpc-model-quickstart.md?pivots=programming-language-csharp)

En el caso de usuarios avanzados, puede consultar algunos de los ejemplos de [Jupyter Notebook](https://github.com/Azure/live-video-analytics/blob/master/utilities/video-analysis/notebooks/readme.md) para Live Video Analytics. Estos cuadernos le proporcionarán instrucciones detalladas para las **extensiones del grafo multimedia** sobre:

* Cómo crear una imagen de contenedor de Docker de un servicio de extensión.
* Cómo implementar el servicio de extensión como un contenedor junto con el contenedor de Live Video Analytics.
* Cómo usar un grafo multimedia de Live Video Analytics con un cliente de extensión y dirigirlo al punto de conexión de la extensión (HTTP o gRPC).
