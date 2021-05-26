---
title: Preguntas frecuentes sobre Azure Live Video Analytics on IoT Edge
description: En este artículo se responden las preguntas más frecuentes sobre Azure Live Video Analytics on IoT Edge.
ms.topic: conceptual
ms.date: 12/01/2020
ms.openlocfilehash: ed27b120ff28704aeb3595815544262bab78e95d
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110371458"
---
# <a name="azure-live-video-analytics-on-iot-edge-faq"></a>Preguntas frecuentes sobre Azure Live Video Analytics on IoT Edge

[!INCLUDE [redirect to Azure Video Analyzer](./includes/redirect-video-analyzer.md)]

En este artículo se responden las preguntas más frecuentes sobre Live Video Analytics en Azure IoT Edge.

## <a name="general"></a>General

**¿Qué variables del sistema se pueden usar en la definición de la topología de grafos?**

| Variable   |  Descripción  | 
| --- | --- | 
| [System.DateTime](/dotnet/framework/data/adonet/sql/linq/system-datetime-methods) | Representa un instante en hora UTC, normalmente expresado como fecha y hora del día en el formato siguiente:<br>*yyyyMMddTHHmmssZ* | 
| System.PreciseDateTime | Representa una instancia de fecha y hora UTC (hora universal coordinada) en un formato compatible con el archivo ISO8601 con milisegundos en el formato siguiente:<br>*yyyyMMddTHHmmss.fffZ* | 
| System.GraphTopologyName   | Representa una topología de grafos multimedia y contiene el plano técnico de un grafo. | 
| System.GraphInstanceName |    Representa una instancia de grafo multimedia, contiene los valores de parámetro y hace referencia a la topología. | 

## <a name="configuration-and-deployment"></a>Configuración e implementación

**¿Se puede implementar el módulo multimedia de Edge en un dispositivo Windows 10?**

Sí. Para más información, consulte [Contenedores de Linux en Windows 10](/virtualization/windowscontainers/deploy-containers/linux-containers).

## <a name="capture-from-ip-camera-and-rtsp-settings"></a>Captura con una cámara IP y configuración de RTSP

**¿Es necesario usar un SDK especial en el dispositivo para enviarlo en una secuencia de vídeo?**

No, Live Video Analytics en IoT Edge permite capturar elementos multimedia mediante RTSP (protocolo de streaming en tiempo real) para el streaming de vídeo, que es compatible con la mayoría de las cámaras IP.

**¿Puedo enviar contenido multimedia a Live Video Analytics en IoT Edge mediante el protocolo de mensajería en tiempo real (RTMP) o el protocolo de Smooth Streaming (por ejemplo, un evento en directo de Media Services)?**

No, Live Video Analytics solo admite RTSP para capturar vídeo de cámaras IP. Cualquier cámara que admita el streaming con RTSP a través de TCP/HTTP debería funcionar. 

**¿Se puede restablecer o actualizar la dirección URL de origen RTSP en una instancia de grafo?**

Sí, cuando la instancia del grafo está en estado *inactivo*.  

**¿Hay algún simulador RTSP disponible para usarlo durante las pruebas y el desarrollo?**

Sí, hay un módulo perimetral del [simulador RTSP](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555) disponible para su uso en los tutoriales y artículos de inicio rápido para respaldar el proceso de aprendizaje. Este módulo se proporciona siempre que sea posible y puede que no siempre esté disponible. Se recomienda encarecidamente que *no* usar el simulador durante más de unas horas. Debe invertir en las pruebas con el origen RTSP real antes de planear una implementación en producción.

**¿Admite la detección de ONVIF de cámaras IP en el perímetro?**

No se admite la detección de Open Network Video Interface Forum (ONVIF) de dispositivos en el perímetro.

## <a name="streaming-and-playback"></a>Streaming y reproducción

**¿Se pueden reproducir recursos grabados en Azure Media Services desde el perímetro mediante tecnologías de streaming, como HLS o DASH?**

Sí. Los recursos grabados se pueden transmitir como cualquier otro recurso de Azure Media Services. Para transmitir el contenido, debe existir un punto de conexión de streaming en ejecución. El uso del proceso de creación del localizador de streaming estándar dará acceso a un manifiesto de HTTP Live Streaming de Apple (HLS) o streaming adaptable dinámico mediante HTTP (DASH, también conocido como MPEG-DASH) para el streaming a cualquier marco de trabajo para reproductores compatible. Para obtener más información sobre la creación y publicación de manifiestos de HLS o DASH, vea [Empaquetado dinámico](../latest/encode-dynamic-packaging-concept.md).

**¿Se pueden usar las características de DRM y protección de contenido estándar de Media Services en un recurso archivado?**

Sí. Todas las características de protección de contenido de cifrado dinámico y administración de derechos digitales (DRM) están disponibles para usarlas en los recursos grabados desde un grafo multimedia.

**¿Qué reproductores se pueden usar para ver el contenido de los recursos grabados?**

Se admiten todos los reproductores estándar compatibles con la versión 3 o versión 4 de HLS. Además, también se admite cualquier reproductor que sea compatible con la reproducción de MPEG-DASH.

Los reproductores recomendados para las pruebas incluyen:

* [Azure Media Player](../latest/player-use-azure-media-player-how-to.md)
* [HLS.js](https://hls-js.netlify.app/demo/)
* [Video.js](https://videojs.com/)
* [Dash.js](https://github.com/Dash-Industry-Forum/dash.js/wiki)
* [Shaka Player](https://github.com/google/shaka-player)
* [ExoPlayer](https://github.com/google/ExoPlayer)
* [HTTP Live Streaming nativo de Apple](https://developer.apple.com/streaming/)
* Reproductor de vídeo HTML5 integrado de Edge, Chrome o Safari
* Reproductores comerciales compatibles con la reproducción HLS o DASH

**¿Cuáles son los límites del streaming de un recurso de grafo multimedia?**

El streaming de un recurso grabado o en directo desde un grafo multimedia usa la misma infraestructura de gran escala y el punto de conexión de streaming que Media Services admite para el streaming a petición y en directo para clientes de medios y entretenimiento, Over the Top (OTT) y difusión. Esto significa que se puede habilitar rápida y fácilmente Azure Content Delivery Network, Verizon o Akamai para ofrecer el contenido a una audiencia tan pequeña como algunos espectadores, o hasta millones, en función de su escenario.

Puede proporcionar contenido mediante HLS de Apple o MPEG-DASH.

## <a name="design-your-ai-model"></a>Diseño del modelo de IA 

**Tengo varios modelos de IA encapsulados en un contenedor de Docker. ¿Cómo debo usarlos con Live Video Analytics?** 

Las soluciones varían en función del protocolo de comunicación que use el servidor de inferencia para comunicarse con Live Video Analytics. Los pasos descritos en las secciones siguientes explican cómo funciona cada protocolo.

*Uso del protocolo HTTP*:

* Contenedor único (lvaExtension único):  

   En el servidor de inferencia, puede usar un solo puerto, pero distintos puntos de conexión para distintos modelos de IA. Por ejemplo, para un ejemplo de Python, puede usar diferentes `route` por modelo, como se muestra a continuación: 

   ```
   @app.route('/score/face_detection', methods=['POST']) 
   … 
   Your code specific to face detection model… 

   @app.route('/score/vehicle_detection', methods=['POST']) 
   … 
   Your code specific to vehicle detection model 
   … 
   ```

   Y, después, en la implementación de Live Video Analytics, al crear una instancia de los grafos, establezca la dirección URL del servidor de inferencia para cada instancia como se muestra a continuación: 

   1\.ª instancia: URL del servidor de inferencia = `http://lvaExtension:44000/score/face_detection`<br/>
   2\.ª instancia: URL del servidor de inferencia = `http://lvaExtension:44000/score/vehicle_detection`  
   
    > [!NOTE]
    > Como alternativa, también puede exponer los modelos de IA en diferentes puertos y llamarlos al crear las instancias de los grafos.  

* Varios contenedores: 

   Cada contenedor se implementa con un nombre distinto. Anteriormente, en el conjunto de documentación de Live Video Analytics, le mostramos cómo implementar una extensión denominada *lvaExtension*. Ahora puede desarrollar dos contenedores diferentes, cada uno con la misma interfaz HTTP, lo que significa que tienen el mismo punto de conexión `/score`. Implemente estos dos contenedores con nombres diferentes y asegúrese de que ambos escuchen en *puertos distintos*. 

   Por ejemplo, un contenedor denominado `lvaExtension1` escucha el puerto `44000` y un segundo contenedor denominado `lvaExtension2` escucha el puerto `44001`. 

   En la topología de Live Video Analytics, se crean instancias de dos grafos con diferentes direcciones URL de inferencia, como se muestra a continuación: 

   Primera instancia: URL del servidor de inferencia = `http://lvaExtension1:44001/score`    
   Segunda instancia: URL del servidor de inferencia = `http://lvaExtension2:44001/score`
   
*Uso del protocolo gRPC*: 

* Con el módulo Live Video Analytics 1.0, cuando se usa un protocolo de llamada a procedimiento remoto (gRPC) de uso general, la única manera es si el servidor de gRPC expusiera distintos modelos de IA a través de puertos distintos. En [este ejemplo de código](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/grpcExtensionOpenVINO/2.0/topology.json), un único puerto (44000) expone todos los modelos de YOLO. En teoría, el servidor gRPC de YOLO se podría volver a escribir para exponer algunos modelos en el puerto 44000 y otros en el puerto 45000. 

* Con el módulo Live Video Analytics 2.0, se agrega una nueva propiedad al nodo de extensión gRPC. Esta propiedad, **extensionConfiguration**, es una cadena opcional que se puede usar como parte del contrato gRPC. Cuando hay varios modelos de IA empaquetados en un único servidor de inferencia, no es necesario exponer un nodo para cada modelo de IA. En su lugar, para una instancia de grafo, el usuario, como proveedor de extensiones, puede definir cómo seleccionar los diferentes modelos de AI mediante la propiedad **extensionConfiguration**. Durante la ejecución, Live Video Analytics pasa esta cadena al servidor de inferencia, que puede usarla para invocar al modelo de IA deseado. 

**Quiero crear un servidor de gRPC en torno a un modelo de IA y poder usarlo en varias cámaras o instancias de grafos. ¿Cómo se debe crear el servidor?** 

 En primer lugar, asegúrese de que el servidor puede administrar más de una solicitud a la vez o trabajar en subprocesos paralelos. 

Por ejemplo, en el siguiente [ejemplo de gRPC de Live Video Analytics](https://github.com/Azure/live-video-analytics/blob/master/utilities/video-analysis/notebooks/Yolo/yolov3/yolov3-grpc-icpu-onnx/lvaextension/server/server.py), se ha establecido un número predeterminado de canales paralelos: 

```
server = grpc.server(futures.ThreadPoolExecutor(max_workers=3)) 
```

En la creación de instancias del servidor de gRPC anterior, el servidor solo puede abrir tres canales a la vez por cámara o por instancia de topología de grafos. No intente conectar más de tres instancias al servidor. Si intenta abrir más de tres canales, las solicitudes estarán pendientes hasta que se anule un canal existente.  

La implementación del servidor de gRPC anterior se usa en nuestros ejemplos de Python. Como desarrollador, puede implementar su propio servidor o usar la implementación predeterminada anterior para aumentar el número de trabajos, que se establece en el número de cámaras que se va a usar para las fuentes de vídeo. 

Para configurar y usar varias cámaras, puede crear varias instancias de la topología de grafos, donde cada instancia apunte al mismo servidor de inferencia o a otro distinto (por ejemplo, el servidor mencionado en el párrafo anterior). 

**Quiero poder recibir varios fotogramas del nivel superior antes de tomar una decisión de inferencia. ¿Cómo se puede habilitar?** 

Nuestros [ejemplos predeterminados](https://github.com/Azure/live-video-analytics/tree/master/utilities/video-analysis) actuales funcionan en modo *sin estado*. No conservan el estado de las llamadas anteriores ni quién llamó. Esto significa que varias instancias de topología podrían llamar al mismo servidor de inferencia, pero el servidor no puede distinguir quién está llamando ni el estado por el autor de llamada. 

*Uso del protocolo HTTP*:

Para mantener el estado, cada autor de llamada o instancia de topología de grafos llama al servidor de inferencia con el parámetro de consulta HTTP único para el autor de la llamada. Por ejemplo, las direcciones URL del servidor de inferencia para cada instancia se muestran a continuación:  

1\.ª instancia de topología = `http://lvaExtension:44000/score?id=1`<br/>
2\.ª instancia de topología = `http://lvaExtension:44000/score?id=2`

… 

En el lado del servidor, la ruta de puntuación sabe quién llama. Si id=1, puede mantener el estado por separado para ese autor de llamada o instancia de topología de grafos. Después, puede mantener los fotogramas de vídeo recibidos en un búfer. Por ejemplo, use una matriz, o un diccionario con una clave datetime, y el valor es el marco. Después, puede definir el servidor de procesamiento (inferencia) después de que se reciba un número *x* de fotogramas. 

*Uso del protocolo gRPC*: 

Con una extensión gRPC, cada sesión es para una única fuente de cámara, por lo que no es necesario proporcionar un identificador. Ahora, con la propiedad extensionConfiguration, puede almacenar los fotogramas de vídeo en un búfer y definir el servidor de procesamiento (inferencia) después de que se reciba un número *x* de fotogramas. 

**¿Ejecutan todos los ProcessMediaStreams de un contenedor determinado el mismo modelo de IA?** 

No. Las llamadas de inicio o detención del usuario final en una instancia de grafos constituyen una sesión o quizás haya una desconexión o reconexión de la cámara. El objetivo es conservar una sesión si la cámara está transmitiendo vídeo. 

* Dos cámaras que envían vídeo para su procesamiento crean dos sesiones. 
* Una cámara que va a un grafo que tiene dos nodos de extensión gRPC crea dos sesiones. 

Cada sesión es una conexión dúplex completa entre Live Video Analytics y el servidor gRPC, y cada sesión puede tener un modelo o una canalización diferentes. 

> [!NOTE]
> En el caso de una desconexión o reconexión de la cámara (donde la cámara se desconecta durante un período que supera los límites de tolerancia), Live Video Analytics abrirá una nueva sesión con el servidor gRPC. No es necesario que el servidor haga el seguimiento del estado en estas sesiones. 

Live Video Analytics también agrega compatibilidad con varias extensiones gRPC para una sola cámara en una instancia de grafos. Podrá usar estas extensiones gRPC para llevar a cabo el procesamiento de IA secuencialmente o en paralelo, o como una combinación de ambos. 

> [!NOTE]
> El hecho de que varias extensiones se ejecuten en paralelo afectará a los recursos de hardware. Tenga esto en cuenta al elegir el hardware que mejor se adapte a sus necesidades computacionales. 

**¿Cuál es el número máximo de ProcessMediaStreams simultáneos?** 

Live Video Analytics no aplica límites a este número.  

**¿Cómo se puede decidir si el servidor de inferencia debe usar la CPU o GPU o cualquier otro acelerador de hardware?** 

La decisión depende de la complejidad del modelo de AI desarrollado y de cómo desea usar los aceleradores de hardware y CPU. A medida que desarrolla el modelo de AI, puede especificar qué recursos debe usar el modelo y qué acciones debe realizar. 

**¿Cómo se almacenan imágenes con cuadros de límite después del procesamiento?** 

En la actualidad, las coordenadas del cuadro de límite solo se proporcionan como mensajes de inferencia. Puede crear un secuenciador MJPEG personalizado que pueda usar estos mensajes y superponer los cuadros de límite en los fotogramas de vídeo.  

## <a name="grpc-compatibility"></a>Compatibilidad con gRPC 

**¿Cómo sabré cuáles son los campos obligatorios para el descriptor de flujo multimedia?** 

A todos los campos a los que no se proporciona un valor se les asigna un [valor predeterminado, según se especifica en gRPC](https://developers.google.com/protocol-buffers/docs/proto3#default).  

Live Video Analytics usa la versión *proto3* del lenguaje de búfer de protocolo. Todos los datos del búfer de protocolo que usan los contratos de Live Video Analytics están disponibles en los [archivos de búfer de protocolo](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc). 

**¿Cómo puedo asegurarme de que estoy usando los archivos de búfer de protocolo más recientes?** 

Puede obtener los archivos de búfer de protocolo más recientes en el [sitio de archivos de contrato](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc). Cada vez que se actualicen los archivos de contrato, estarán en esta ubicación. No hay ningún plan inmediato para actualizar los archivos de protocolo, por lo que debe buscar el nombre del paquete en la parte superior de los archivos para conocer la versión. Debería decir esto: 

```
microsoft.azure.media.live_video_analytics.extensibility.grpc.v1 
```

Todas las actualizaciones de estos archivos harán aumentar la porción "v-valor" al final del nombre. 

> [!NOTE]
> Dado que Live Video Analytics usa la versión proto3 del lenguaje, los campos son opcionales, y la versión es compatible con versiones anteriores y posteriores. 

**¿Qué características de gRPC se pueden usar con Live Video Analytics? ¿Qué características son obligatorias y cuáles son opcionales?** 

Se pueden usar todas las características de gRPC del lado servidor siempre que se cumpla el contrato Protocol Buffers (Protobuf). 

## <a name="monitoring-and-metrics"></a>Supervisión y métricas

**¿Se puede supervisar el grafo multimedia en el perímetro mediante Azure Event Grid?**

Sí. Puede consumir las métricas de Prometheus y publicarlas en la cuadrícula de eventos. 

**¿Se puede usar Azure Monitor para ver el estado, las métricas y el rendimiento de los grafos multimedia en la nube o en el perímetro?**

Sí, se admite este enfoque. Para obtener más información, consulte [Información general sobre las métricas de Azure Monitor](../../azure-monitor/essentials/data-platform-metrics.md).

**¿Hay alguna herramienta que facilite la supervisión del módulo de IoT Edge de Media Services?**

Visual Studio Code admite la extensión Azure IoT Tools, que permite supervisar fácilmente los puntos de conexión del módulo LVAEdge. Puede usar esta herramienta para empezar a supervisar rápidamente el punto de conexión integrado de IoT Hub para "eventos" y ver los mensajes de inferencia que se enrutan desde el dispositivo perimetral hasta la nube. 

Además, puede usar esta extensión para editar el módulo gemelo para el módulo LVAEdge con el fin de modificar la configuración del grafo multimedia.

Para más información, vea el artículo [Supervisión y registro](monitoring-logging.md).

## <a name="billing-and-availability"></a>Facturación y disponibilidad

**¿Cómo se factura Live Video Analytics on IoT Edge?**

Para obtener información detallada sobre la facturación, consulte [Precios de Media Services](https://azure.microsoft.com/pricing/details/media-services/).

## <a name="next-steps"></a>Pasos siguientes

[Inicio rápido: Introducción a Live Video Analytics en IoT Edge](get-started-detect-motion-emit-events-quickstart.md)
