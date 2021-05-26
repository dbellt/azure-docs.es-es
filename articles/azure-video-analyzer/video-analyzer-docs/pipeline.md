---
title: Canalización de Azure Video Analyzer
description: Una canalización de Azure Video Analyzer le permite definir dónde se deben capturar los datos de entrada, cómo se deben procesar y dónde se deben entregar los resultados. Una canalización consta de nodos que están conectados para lograr el flujo de datos deseado.
ms.topic: conceptual
ms.date: 05/13/2021
ms.openlocfilehash: 9e3945624a52b64612c7edca2f3380b072f60bec
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388655"
---
# <a name="pipeline"></a>Canalización

Una canalización de Azure Video Analyzer le permite definir dónde se deben capturar los datos de entrada, cómo se deben procesar y dónde se deben entregar los resultados. Una canalización consta de nodos que están conectados para lograr el flujo de datos deseado. En el diagrama siguiente se muestra una representación gráfica de una canalización.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/pipeline/pipeline-representation.svg" alt-text="Representación de una canalización":::

Una canalización admite distintos tipos de nodos

* Los **nodos de origen** permiten capturar los datos en la canalización. Los datos se refieren a audio, vídeo o metadatos.
* Los **nodos de procesador** permiten el procesamiento de elementos multimedia dentro de la canalización.
* Los **nodos receptores** permiten entregar los resultados a servicios y aplicaciones fuera de la canalización.

## <a name="suggested-pre-reading"></a>Sugerencias antes de la lectura

* [Información general](overview.md)
* [Terminología](terminology.md)

## <a name="pipeline-topologies"></a>Topologías de canalización

Una topología de canalización le permite describir cómo se debe procesar y analizar el vídeo en directo según sus requisitos personalizados a través de un conjunto de nodos interconectados. Puede crear topologías diferentes para distintos escenarios al seleccionar qué nodos forman parte de la topología y cómo están conectados, a la vez que puede usar parámetros como marcadores de posición para los valores. Una canalización es una instancia independiente de una topología de canalización específica. Los elementos multimedia en realidad se procesan en una canalización. Las canalizaciones se pueden asociar a cámaras individuales (así como a otros aspectos) gracias a parámetros definidos por el usuario que se declaran en la topología de canalización.

Por ejemplo, si quiere grabar vídeos de varias cámaras IP, puede definir una topología de canalización que conste de un nodo de origen RTSP y un nodo receptor de vídeo. El nodo de origen RTSP puede tener como parámetros una dirección URL de RTSP, un nombre de usuario y una contraseña. El nodo receptor de vídeo puede tener el nombre del vídeo como parámetro. Los valores para estos parámetros se pueden especificar al crear varias canalizaciones a partir de la misma topología: una canalización por cámara.

## <a name="pipeline-states"></a>Estados de canalización

En el diagrama siguiente está representado el ciclo de vida de una canalización.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/pipeline/pipeline-activation.svg" alt-text="Ciclo de vida de una canalización":::

Empiece con la creación de la topología de canalización. Una vez definida la topología, puede especificar valores para los parámetros a fin de crear canalizaciones. Una vez que haya creado correctamente una canalización, esta se encontrará en estado "Inactivo". Tras activarse, una canalización entrará en el estado "Activando" y, a continuación, en "Activo". 

Los datos (vídeo en directo) comienzan a fluir a través de la canalización cuando esta se encuentra en el estado "Activo". Tras desactivarse, una canalización activa entra en el estado "Desactivando" y, a continuación, en "Inactivo". Solo se pueden eliminar las canalizaciones inactivas.

Se pueden crear varias canalizaciones a partir de una sola topología si se suministran valores diferentes para los parámetros de la topología. Una topología se puede eliminar cuando se han eliminado todas las canalizaciones.

> [!NOTE]
> Una canalización puede estar activa incluso si no fluyen datos a través de ella (por ejemplo, el origen del vídeo de entrada se queda sin conexión). Se facturará a la suscripción a Azure cuando la canalización esté en estado activo.

## <a name="sources-processors-and-sinks"></a>Orígenes, procesadores y receptores

El módulo perimetral de Video Analyzer admite los siguientes tipos de nodos dentro de una canalización:

### <a name="sources"></a>Orígenes

#### <a name="rtsp-source"></a>Origen RTSP

Un nodo de origen RTSP permite capturar elementos multimedia desde un servidor RTSP. El protocolo RTSP define el establecimiento y control de las sesiones multimedia entre un servidor (por ejemplo, una cámara IP) y un cliente. El nodo de origen [RTSP](https://tools.ietf.org/html/rfc2326) en una canalización actúa como un cliente y puede establecer una sesión con un servidor RTSP. Muchos dispositivos, como la mayoría de las [cámaras IP](https://en.wikipedia.org/wiki/IP_camera) tienen un servidor RTSP integrado. [ONVIF](https://www.onvif.org/) exigirá que se admita RTSP en la definición de los dispositivos compatibles con los [perfiles G, S y T](https://www.onvif.org/wp-content/uploads/2019/12/ONVIF_Profile_Feature_overview_v2-3.pdf). El nodo de origen RTSP requiere que especifique una dirección URL de RTSP, junto con las credenciales para habilitar una conexión autenticada.

#### <a name="iot-hub-message-source"></a>Origen de mensajes de IoT Hub

Al igual que otros [módulos de IoT Edge](../../iot-fundamentals/iot-glossary.md?view=iotedge-2020-11&preserve-view=true#iot-edge-device), el módulo Azure Video Analyzer puede recibir mensajes a través del [centro de IoT Edge](../../iot-fundamentals/iot-glossary.md?view=iotedge-2020-11&preserve-view=true#iot-edge-hub). Se pueden enviar mensajes desde otros módulos o aplicaciones que se ejecutan en el dispositivo perimetral o desde la nube. Estos mensajes se pueden entregar (enrutar) a una [entrada con nombre](../../iot-edge/module-composition.md?view=iotedge-2020-11&preserve-view=true#sink) en el módulo de Video Analyzer. Un nodo de origen de mensajes de IoT Hub permite ingerir estos mensajes en una canalización. A continuación, los mensajes se pueden usar en una canalización para activar una puerta de señales (consulte la sección de [puertas de señales](#signal-gate-processor) a continuación).

Por ejemplo, podría tener un módulo de IoT Edge que genere un mensaje cuando se abre una puerta. El mensaje de ese módulo se puede enrutar al centro de IoT Edge, desde donde se puede enrutar al origen de los mensajes del centro de IoT de una canalización. Dentro de la canalización, el mensaje se puede pasar del origen de los mensajes del centro de IoT a un procesador de la puerta de señales, que puede activar la grabación del vídeo desde un origen RTSP en un archivo.

### <a name="processors"></a>Procesadores

#### <a name="motion-detection-processor"></a>Procesador de detección de movimiento

El nodo del procesador de detección de movimiento permite detectar movimiento en vídeos en directo. Examina los fotogramas de vídeo entrantes y determina si hay movimiento en el vídeo. Si se detecta movimiento, pasa el fotograma de vídeo al siguiente nodo en la canalización y emite un evento. El nodo del procesador de detección de movimiento (junto con otros nodos) se puede usar para desencadenar la grabación del vídeo entrante cuando se detecta movimiento.

#### <a name="http-extension-processor"></a>Procesador de extensiones HTTP

El nodo del procesador de extensiones HTTP permite ampliar la canalización al módulo de IoT Edge. Este nodo recibe fotogramas de vídeo descodificados como entrada y los retransmite a un punto de conexión REST HTTP expuesto por el módulo, donde puede analizar el fotograma con un modelo de inteligencia artificial y devolver los resultados de la inferencia. Obtenga más información sobre el [procesador aquí](pipeline-extension.md#http-extension-processor). Además, el nodo tiene un formateador de imagen integrado para escalar y codificar los fotogramas de vídeo antes de retransmitirlos al punto de conexión HTTP. El escalador tiene opciones para conservar, rellenar o ajustar la relación de aspecto de la imagen. El codificador de imágenes admite los formatos JPEG, PNG, BMP y RAW. Obtenga más información sobre el [procesador aquí](/pipeline-extension.md#grpc-extension-processor).

#### <a name="grpc-extension-processor"></a>Procesador de extensiones gRPC

Este nodo procesador de extensiones gRPC toma los fotogramas de vídeo descodificados como entrada y los retransmite a un punto de conexión [gRPC](pipeline-extension.md#grpc-extension-processor) expuesto por el módulo. El nodo admite la transferencia de datos mediante [memoria compartida](https://en.wikipedia.org/wiki/Shared_memory) o la incrustación directa del fotograma en el cuerpo de los mensajes gRPC. Al igual que el proceso de la extensión HTTP, este nodo también tiene un formateador de imagen integrado para escalar y codificar los fotogramas de vídeo antes de retransmitirlos al punto de conexión gRPC. Obtenga más información sobre el [procesador aquí](/pipeline-extension.md#grpc-extension-processor).

#### <a name="cognitive-services-extension-processor"></a>Procesador de extensiones de Cognitive Services

El procesador de extensiones de Cognitive Services le permite ampliar la canalización al módulo de [análisis espacial](https://azure.microsoft.com/services/cognitive-services/computer-vision/) de IoT Edge. Este nodo recibe fotogramas de vídeo descodificados como entrada y los retransmite a un punto de conexión [gRPC](pipeline-extension.md#grpc-extension-processor) expuesto, donde puede analizar el fotograma con las aptitudes de análisis espacial y devolver los resultados de la inferencia. Obtenga más información sobre el [procesador aquí](pipeline-extension.md#cognitive-services-extension-processor).

#### <a name="signal-gate-processor"></a>Procesador de la puerta de señales

El nodo del procesador de la puerta de señales permite reenviar de manera condicional elementos multimedia de un nodo a otro. Un ejemplo de caso de uso consiste en insertar un nodo del procesador de la puerta de señales entre el nodo de origen RTSP y el nodo receptor del vídeo y, después, usar la salida de un nodo del procesador de detección de movimiento para desencadenar la puerta. Con este tipo de canalización, solo se graba el vídeo cuando se detecta movimiento. También puede usar la salida del nodo de extensiones HTTP o gRPC para desencadenar la puerta, en lugar del nodo del procesador de detección de movimiento, lo que permite la grabación de vídeo cuando se detecta algo interesante.

#### <a name="object-tracker-processor"></a>Procesador de seguimiento de objetos

El nodo del procesador de seguimiento de objetos permite realizar un seguimiento de los objetos detectados en un nodo ascendente del procesador de extensiones HTTP o gRPC. Este nodo resulta útil cuando necesita detectar objetos en cada fotograma, pero el dispositivo perimetral no tiene la capacidad de proceso necesaria para poder aplicar el modelo de inteligencia artificial en todos los fotogramas. Si solo puede ejecutar el modelo de Computer Vision cada diez fotogramas, el seguimiento de objetos puede tomar los resultados de uno de estos fotogramas y, a continuación, usar técnicas de [flujo óptico](https://en.wikipedia.org/wiki/Optical_flow) para generar resultados para el segundo, tercer y hasta noveno fotograma, hasta que el modelo se aplique de nuevo en el fotograma siguiente. Hay un equilibrio entre la capacidad de proceso y la precisión al usar este nodo. Cuanto más cercanos sean los fotogramas en los que se aplica el modelo de inteligencia artificial, mayor será la precisión. Sin embargo, esto significa aplicar el modelo de inteligencia artificial con más frecuencia, lo que se traduce en una mayor capacidad de proceso. Un uso común del nodo del procesador de seguimiento de objetos es detectar cuándo un objeto cruza una línea.

#### <a name="line-crossing-processor"></a>Procesador de cruce de líneas

El nodo del procesador de cruce de líneas permite detectar en qué momento un objeto cruza una línea definida por el usuario. Además, mantiene también un recuento del número de objetos que cruzan la línea (desde el momento en que se activa una canalización). Este nodo debe usarse en el nivel inferior de un nodo del procesador de seguimiento de objetos.

### <a name="sinks"></a>Receptores

#### <a name="video-sink"></a>Receptor de vídeo

Un nodo receptor de vídeo permite guardar el vídeo y los metadatos asociados en el recurso en la nube de Video Analyzer. El vídeo se puede grabar de forma continua o no continua (según los eventos). El nodo receptor de vídeo puede almacenar el vídeo en la caché del dispositivo perimetral si se pierde la conectividad a la nube y reanudar la carga cuando se restaure dicha conectividad. Puede consultar el artículo sobre la [grabación continua de vídeo](continuous-video-recording.md) para obtener más información sobre cómo se pueden configurar las propiedades de este nodo.

#### <a name="file-sink"></a>Receptor de archivos

El nodo receptor de archivos permite escribir vídeos en una ubicación del sistema de archivos local del dispositivo perimetral. Solo puede haber un nodo receptor de archivos en una canalización y debe encontrarse en un nivel inferior a un nodo del procesador de la puerta de señales. Esto limita la duración de los archivos de salida a los valores especificados en las propiedades del nodo del procesador de la puerta de señales. Para asegurarse de que el dispositivo perimetral no se queda sin espacio en disco, también puede establecer el tamaño máximo que el módulo perimetral de Video Analyzer puede usar para almacenar datos en caché.

> [!NOTE]
> Si la caché se llena, el módulo perimetral de Video Analyzer comenzará a eliminar los datos más antiguos y los reemplazará por los nuevos.

#### <a name="iot-hub-message-sink"></a>Receptor de mensajes de IoT Hub

Un nodo receptor de mensajes de IoT Hub permite publicar eventos en el centro de IoT Edge. El centro de IoT Edge puede configurarse para enrutar los datos a otros módulos o aplicaciones del dispositivo perimetral o a IoT Hub en la nube (según las rutas especificadas en el manifiesto de implementación). El nodo receptor de mensajes de IoT Hub puede aceptar eventos de nodos de procesadores de nivel superior, como un nodo del procesador de detección de movimiento o de un servicio de inferencia externo, a través de un nodo de procesador de extensiones HTTP.

## <a name="rules-on-the-use-of-nodes"></a>Reglas sobre el uso de los nodos

Consulte las [limitaciones de las canalizaciones](quotas-limitations.md#limitations-on-pipeline-topologies) para obtener más reglas sobre cómo se pueden usar diferentes nodos dentro de una canalización.

## <a name="scenarios"></a>Escenarios

Al combinar los orígenes, procesadores y receptores definidos anteriormente, puede crear canalizaciones para diversos escenarios que implican el análisis de vídeos en directo. Los escenarios de ejemplo son:

* [Grabación continua de vídeo](continuous-video-recording.md) 
* [Grabación de vídeo basada en eventos](event-based-video-recording-concept.md) 
* [Análisis de vídeos sin grabación](analyze-live-video-without-recording.md) 

## <a name="next-steps"></a>Pasos siguientes

Para ver cómo puede ejecutar la detección de movimiento en una fuente de vídeo en directo, consulte [Inicio rápido: Introducción a Azure Video Analyzer](get-started-detect-motion-emit-events.md).

