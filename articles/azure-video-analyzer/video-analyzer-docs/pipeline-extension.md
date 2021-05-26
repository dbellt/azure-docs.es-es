---
title: 'Extensión de canalización: Azure Video Analyzer'
description: Azure Video Analyzer permite extender las funcionalidades de procesamiento de canalizaciones mediante un nodo de extensión de canalización. En este artículo se describe el nodo de extensión de canalización.
ms.service: azure-video-analyzer
ms.topic: conceptual
ms.date: 03/30/2021
ms.openlocfilehash: 59476f465d65db6e716ca5e5b43da995d8e737de
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388661"
---
# <a name="pipeline-extension"></a>Extensión de canalización

Azure Video Analyzer permite extender las funcionalidades de procesamiento de canalizaciones mediante un nodo de extensión de canalización. El complemento de extensión de análisis puede usar técnicas tradicionales de procesamiento de imágenes o modelos de inteligencia artificial de Computer Vision. Las extensiones de canalización se habilitan al incluir un nodo de procesador de extensiones en un flujo de canalización. El nodo de procesador de extensiones retransmite fotogramas de vídeo al punto de conexión configurado y actúa como la interfaz de la extensión. La conexión se puede realizar con un punto de conexión local o remoto y se puede proteger mediante autenticación y cifrado TLS, si fuera necesario. Además, el nodo procesador de extensiones de canalización permite el escalado y la codificación opcionales de los fotogramas de vídeo antes de enviarlos a la extensión personalizada.

Video Analyzer admite estos procesadores de extensión de canalización:

* [Procesador de extensiones HTTP](pipeline.md#http-extension-processor) 
* [Procesador de extensiones de gRPC](pipeline.md#grpc-extension-processor)
* [Procesador de extensiones de Cognitive Services](pipeline.md#cognitive-services-extension-processor) 
    
El nodo de extensión de canalización espera que el complemento de extensión de análisis devuelva los resultados en formato JSON. Idealmente, los resultados deben seguir el [modelo de objetos del esquema de metadatos de inferencia](inference-metadata-schema.md).

## <a name="http-extension-processor"></a>Procesador de extensiones HTTP

El procesador de extensiones HTTP permite escenarios de extensibilidad mediante el [protocolo HTTP](http-extension-protocol.md), donde el rendimiento o el uso óptimo de los recursos no son las principales preocupaciones. Puede exponer su propia IA a una canalización a través de un punto de conexión de REST HTTP.

Use el nodo de procesador de extensiones HTTP cuando:

* Quiera una mejor interoperabilidad con los sistemas de inferencia de HTTP existentes.
* La transferencia de datos de bajo rendimiento es aceptable.
* Quiera usar una interfaz de solicitud-respuesta simple para Video Analyzer.

## <a name="grpc-extension-processor"></a>Procesador de extensiones de gRPC

El procesador de extensiones de gRPC permite escenarios de extensibilidad mediante el uso de un [protocolo estructurado](grpc-extension-protocol.md) de alto rendimiento basado en gRPC. Es ideal para escenarios en los que el rendimiento o la utilización óptima de los recursos es una prioridad. El procesador de extensiones de gRPC permite sacar el máximo partido de las definiciones de datos estructurados. gRPC ofrece rendimiento alto de transferencia de contenido mediante:

* la [memoria compartida integrada](https://en.wikipedia.org/wiki/Shared_memory) o
* la inserción directa del contenido en el cuerpo de los mensajes gRPC.

El procesador de extensiones de gRPC se puede usar para enviar propiedades junto con el intercambio de mensajes de inferencia. Por lo tanto, use un nodo de procesador de extensiones de gRPC cuando:

* Quiera usar un contrato estructurado (por ejemplo, mensajes estructurados para solicitudes y respuestas).
* Quiera usar [búferes de protocolo (protobuf)](https://developers.google.com/protocol-buffers) como su formato de intercambio de mensajes subyacentes para la comunicación.
* Quiera comunicarse con un servidor de gRPC en una sesión de un solo flujo, en lugar del modelo de solicitud-respuesta tradicional que necesita un controlador de solicitud personalizado para analizar las solicitudes entrantes y llamar a las funciones de implementación correctas.
* Quiera una comunicación de baja latencia y alto rendimiento entre Video Analyzer y el módulo.

## <a name="cognitive-services-extension-processor"></a>Procesador de extensiones de Cognitive Services

El procesador de extensiones de Cognitive Services es un procesador de extensiones personalizado que permite que Video Analyzer funcione correctamente con las funcionalidades de [análisis espacial de Computer Vision]../../cognitive-services/computer-vision/) mediante el [protocolo estructurado](grpc-extension-protocol.md) de alto rendimiento basado en gRPC. 

Use el nodo de procesador de extensiones de Cognitive Services cuando:

* Quiera una mejor interoperabilidad con las [operaciones de análisis espacial](../../cognitive-services/computer-vision/intro-to-spatial-analysis-public-preview.md) existentes.
* Quiera usar todas las ventajas del protocolo gRPC, la precisión y el rendimiento de la inteligencia artificial integrada y compatible de Microsoft.
* Analice varias fuentes de cámara con baja latencia y alto rendimiento.

## <a name="use-your-inferencing-model"></a>Uso del modelo de inferencia

Las extensiones de canalizaciones le permiten ejecutar el modelo de inferencia que prefiera en cualquier runtime de inferencia disponible, como ONNX, TensorFlow o PyTorch, u otros en su propio contenedor de Docker. La extensión personalizada de inferencia se debe implementar junto con el módulo perimetral de Video Analyzer para obtener el mejor rendimiento y se invocará a través del procesador de extensiones HTTP, del procesador de extensiones de gRPC o del procesador de extensiones de Cognitive Services que se incluye en la topología de canalización. Además, la frecuencia de las llamadas a la extensión personalizada se puede limitar si se agrega un [procesador de detección de movimiento](pipeline.md#motion-detection-processor) ascendente al procesador de extensiones de canalización.

En el diagrama siguiente se muestra el flujo de datos general:

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/pipeline-extension/ai-inference-results.svg" alt-text="Modelo de inferencia":::
 
## <a name="samples"></a>Ejemplos

Puede comenzar con uno de los inicios rápidos que ilustran Video Analyzer con el servicio de extensión precompilado a velocidades de fotogramas bajas con el [procesador de extensiones HTTP](pipeline.md#http-extension-processor) o a velocidades de fotogramas altas con el [procesador de extensiones de gRPC](pipeline.md#grpc-extension-processor).


## <a name="next-steps"></a>Pasos siguientes 

Concepto: [Grabación de vídeo basada en eventos](event-based-video-recording-concept.md)

