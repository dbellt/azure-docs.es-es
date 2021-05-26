---
title: Preguntas frecuentes acerca de Azure Video Analyzer - Azure
description: En este artículo se responden las preguntas más frecuentes sobre Azure Video Analyzer.
ms.service: azure-video-analyzer
ms.topic: conceptual
ms.date: 03/26/2021
ms.openlocfilehash: 94f85cecdb8ee3d18ad7521d8157c01f1410c23c
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388503"
---
# <a name="faq-about-azure-video-analyzer"></a>Preguntas frecuentes acerca de Azure Video Analyzer

En este artículo se responden las preguntas más frecuentes sobre Azure Video Analyzer.

## <a name="general"></a>General

**¿Qué variables del sistema se pueden usar en la definición de la topología de canalización?**

| Variable   |  Descripción  | 
| --- | --- | 
| System.Runtime.DateTime | Representa un instante en hora UTC, normalmente expresado como fecha y hora del día en el formato siguiente:<br>*yyyyMMddTHHmmssZ* | 
| System.Runtime.PreciseDateTime | Representa una instancia de fecha y hora UTC (hora universal coordinada) en un formato compatible con el archivo ISO8601 con milisegundos en el formato siguiente:<br>*yyyyMMddTHHmmss.fffZ* | 
| System.TopologyName    | Representa el nombre de la topología de canalización. | 
| System.PipelineName | Representa el nombre de la topología de canalización en vivo. | 

> [!Note] 
> System.Runtime.DateTime y System.Runtime.PreciseDateTime no se pueden usar como parte del nombre de un recurso de vídeo de Azure Video Analyzer, en un nodo receptor de vídeo. Estas variables se pueden usar en un nodo FileSink para asignar un nombre al archivo.

**¿Qué es la directiva de privacidad de Video Analyzer?**

Video Analyzer está cubierto por la [declaración de privacidad de Microsoft](https://privacy.microsoft.com/privacystatement). En la declaración de privacidad se explican los datos personales que Microsoft procesa, cómo los procesa y los propósitos de este procesamiento. Para más información sobre la privacidad, visite [Microsoft Trust Center](https://www.microsoft.com/trustcenter).

## <a name="configuration-and-deployment"></a>Configuración e implementación

**¿Se puede implementar el módulo perimetral en un dispositivo Windows 10?**

Sí. Para más información, consulte [Contenedores de Linux en Windows 10](/virtualization/windowscontainers/deploy-containers/linux-containers).

## <a name="capture-from-ip-camera-and-rtsp-settings"></a>Captura con una cámara IP y configuración de RTSP

**¿Es necesario usar un SDK especial en el dispositivo para enviarlo en una secuencia de vídeo?**

No, Video Analyzer permite capturar elementos multimedia mediante RTSP (protocolo de streaming en tiempo real) para el streaming de vídeo, que es compatible con la mayoría de las cámaras IP.

**¿Se pueden insertar medios en el Video Analyzer mediante protocolos distintos de RTSP?**

No, Video Analyzer solo admite RTSP para capturar vídeo de cámaras IP. Cualquier cámara que admita el streaming con RTSP a través de TCP/HTTP debería funcionar. 

**¿Se puede restablecer o actualizar la dirección URL de origen RTSP en una canalización en vivo?**

Sí, cuando la canalización en vivo está en estado *inactivo*.  

**¿Hay algún simulador RTSP disponible para usarlo durante las pruebas y el desarrollo?**

Sí, hay un módulo perimetral<!--add-valid-link.md)--><!-- https://github.com/Azure/video-analyzer/tree/main/utilities/rtspsim-live555 --> del simulador RTSP disponible para su uso en los tutoriales y artículos de inicio rápido para respaldar el proceso de aprendizaje. Este módulo se proporciona siempre que sea posible y puede que no siempre esté disponible. Se recomienda encarecidamente que *no* usar el simulador durante más de unas horas. Debe invertir en las pruebas con el origen RTSP real antes de planear una implementación en producción.

## <a name="design-your-ai-model"></a>Diseño del modelo de IA 

**Tengo varios modelos de IA encapsulados en un contenedor de Docker. ¿Cómo deben usarse con Azure Video Analyzer?** 

Las soluciones varían en función del protocolo de comunicación que use el servidor de inferencia para comunicarse con Azure Video Analyzer. Los pasos descritos en las secciones siguientes explican cómo funciona cada protocolo.

*Uso del protocolo HTTP*:

* Contenedor único (módulo denominado *avaextension*):  

   En el servidor de inferencia, puede usar un solo puerto, pero distintos puntos de conexión para distintos modelos de IA. Por ejemplo, para una muestra de Python, puede usar diferentes `routes` por modelo, como se muestra a continuación: 

   ```
   @app.route('/score/face_detection', methods=['POST']) 
   … 
   Your code specific to face detection model

   @app.route('/score/vehicle_detection', methods=['POST']) 
   … 
   Your code specific to vehicle detection model 
   … 
   ```

   Después, en la implementación de Video Analyzer, al activar canalizaciones en vivo, establezca la dirección URL del servidor de inferencia para cada una como se muestra a continuación: 

   Primera canalización en vivo: dirección URL del servidor de inferencia=`http://avaextension:44000/score/face_detection`<br/>
   Segunda canalización en vivo: dirección URL del servidor de inferencia=`http://avaextension:44000/score/vehicle_detection`  
   
    > [!NOTE]
    > Como alternativa, también puede exponer los modelos de IA en diferentes puertos y llamarlos al crear las canalizaciones en vivo.  

* Varios contenedores: 

   Cada contenedor se implementa con un nombre distinto. En los inicios rápidos y tutoriales, hemos mostrado cómo implementar una extensión denominada *avaextension*. Ahora puede desarrollar dos contenedores diferentes, cada uno con la misma interfaz HTTP, lo que significa que tienen el mismo punto de conexión `/score`. Implemente estos dos contenedores con nombres diferentes y asegúrese de que ambos escuchen en *puertos distintos*. 

   Por ejemplo, un contenedor denominado `avaextension1` escucha el puerto `44000` y un segundo contenedor denominado `avaextension2` escucha el puerto `44001`. 

   En la topología de Video Analyzer, se crean dos canalizaciones en vivo con diferentes direcciones URL de inferencia, como se muestra a continuación: 

   Primera canalización en vivo: dirección URL del servidor de inferencia = `http://avaextension1:44001/score`    
   Segunda canalización en vivo: dirección URL del servidor de inferencia = `http://avaextension2:44001/score`
   
*Uso del protocolo gRPC*: 

* El nodo de extensión gRPC tiene una propiedad `extensionConfiguration`, una cadena opcional que se puede usar como parte del contrato gRPC. Cuando hay varios modelos de IA empaquetados en un único servidor de inferencia, no es necesario exponer un nodo para cada modelo de IA. En su lugar, para una canalización en vivo, el usuario, como proveedor de extensiones, puede definir cómo seleccionar los diferentes modelos de AI mediante la propiedad `extensionConfiguration`. Durante la ejecución, Video Analyzer pasa esta cadena al servidor de inferencia, que puede usarla para invocar al modelo de IA deseado. 

**Quiero crear un servidor de gRPC en torno a un modelo de IA y poder usarlo en varias cámaras o canalizaciones en vivo. ¿Cómo se debe crear el servidor?** 

 En primer lugar, asegúrese de que el servidor puede administrar más de una solicitud a la vez o trabajar en subprocesos paralelos. 

Por ejemplo, en la siguiente [muestra de gRPC de Video Analyzer de Azure](), se ha establecido un número predeterminado de canales paralelos<!--add-valid-link.md)--><!-- https://github.com/Azure/video-analyzer/tree/main/utilities/video-analysis/notebooks/Yolo/yolov3/yolov3-grpc-icpu-onnx/avaextension/server/server.py -->: 

```
server = grpc.server(futures.ThreadPoolExecutor(max_workers=3)) 
```

En la creación de instancias del servidor de gRPC anterior, el servidor solo puede abrir tres canales a la vez por cámara o por canalización en vivo. No intente conectar más de tres instancias al servidor. Si intenta abrir más de tres canales, las solicitudes permanecerán pendientes hasta que se anule un canal existente.  

La implementación del servidor de gRPC anterior se usa en nuestros ejemplos de Python. Como desarrollador, puede implementar su propio servidor o usar la implementación predeterminada anterior para aumentar el número de trabajos, que se establece en el número de cámaras que se va a usar para las fuentes de vídeo. 

Para configurar y usar varias cámaras, puede crear varias canalizaciones en vivo, donde cada instancia apunte al mismo servidor de inferencia o a otro distinto (por ejemplo, el servidor mencionado en el párrafo anterior). 

**Quiero poder recibir varios fotogramas antes de tomar una decisión de inferencia. ¿Cómo se puede habilitar?** 

Nuestras [muestras predeterminadas]() actuales<!--add-valid-link.md)--><!--https://github.com/Azure/video-analyzer/tree/main/utilities/video-analysis--> funcionan en un modo *sin estado*. No conservan el estado de las llamadas anteriores ni el identificador de quien llamó. Esto significa que varias canalizaciones en vivo podrían llamar al mismo servidor de inferencia, pero el servidor no puede distinguir quién está llamando ni el estado por autor de llamada. 

*Uso del protocolo HTTP*:

Para mantener el estado, cada autor de llamada o canalización en vivo llama al servidor de inferencia con el parámetro de consulta HTTP único para el autor de la llamada. Por ejemplo, las direcciones URL del servidor de inferencia para cada canalización en vivo se muestran a continuación:  

Primera canalización en vivo: `http://avaextension:44000/score?id=1`<br/>
Segunda canalización en vivo: `http://avaextension:44000/score?id=2`

… 

En el lado del servidor, el `id` ayuda a identificar al autor de la llamada. Si `id`=1, el servidor puede mantener el estado por separado para esa canalización en directo. Después, puede mantener los fotogramas de vídeo recibidos en un búfer. Por ejemplo, use una matriz, o un diccionario con una clave datetime, y el valor es el marco. Después, puede definir el servidor de procesamiento (inferencia) después de que se reciba un número *x* de fotogramas. 

*Uso del protocolo gRPC*: 

Con una extensión gRPC, cada sesión es para una única fuente de cámara, por lo que no es necesario proporcionar un identificador. Con la propiedad extensionConfiguration, puede almacenar los fotogramas de vídeo en un búfer y definir el servidor de procesamiento (inferencia) después de que se reciba un número *x* de fotogramas. 

**¿Ejecutan todos los ProcessMediaStreams de un contenedor determinado el mismo modelo de IA?** 

No. Las llamadas de inicio o detención del usuario final en una canalización en vivo constituyen una sesión o quizás haya una desconexión o reconexión de la cámara. El objetivo es conservar una sesión si la cámara está transmitiendo vídeo. 

* Dos cámaras que envían vídeo para su procesamiento (a dos canalizaciones en vivo independientes) crean dos sesiones. 
* Una cámara que va a una canalización que tiene dos nodos de extensión gRPC crea dos sesiones. 

Cada sesión es una conexión dúplex completa entre Video Analyzer y el servidor gRPC, y cada sesión puede tener un modelo diferente. 

> [!NOTE]
> En el caso de una desconexión o reconexión de la cámara (donde la cámara se desconecta durante un período que supera los límites de tolerancia), Video Analyzer abrirá una nueva sesión con el servidor gRPC. No es necesario que el servidor haga el seguimiento del estado en estas sesiones. 

Video Analyzer también agrega compatibilidad con varias extensiones gRPC para una sola cámara en una canalización en vivo. Podrá usar estas extensiones gRPC para llevar a cabo el procesamiento de IA secuencialmente o en paralelo, o como una combinación de ambos. 

> [!NOTE]
> El hecho de que varias extensiones se ejecuten en paralelo afectará a los recursos de hardware. Tenga esto en cuenta al elegir el hardware que mejor se adapte a sus necesidades computacionales. 

**¿Cuál es el número máximo de ProcessMediaStreams simultáneos?** 

Video Analyzer no aplica límites a este número.  

**¿Cómo se puede decidir si el servidor de inferencia debe usar la CPU o GPU o cualquier otro acelerador de hardware?** 

La decisión depende de la complejidad del modelo de AI desarrollado y de cómo desea usar los aceleradores de hardware y CPU. A medida que desarrolla el modelo de AI, puede especificar qué recursos debe usar el modelo y qué acciones debe realizar. 

**Cómo se pueden ver los cuadros de límite generados por mi servidor de inferencia?** 

El usuario puede registrar los resultados de la inferencia junto con los elementos multimedia del recurso de vídeo. El usuario puede usar un [widget]()<!--add-valid-link.md)--><!-- pointer to widget md --> para reproducir el vídeo con una superposición de los datos de inferencia.

## <a name="grpc-compatibility"></a>Compatibilidad con gRPC 

**¿Cómo sabré cuáles son los campos obligatorios para el descriptor de flujo multimedia?** 

A todos los campos a los que no se proporciona un valor se les asigna un [valor predeterminado, según se especifica en gRPC](https://developers.google.com/protocol-buffers/docs/proto3#default).  

Video Analyzer usa la versión *proto3* del lenguaje de búfer de protocolo. Todos los datos del búfer de protocolo que usan los contratos de Video Analyzer están disponibles en los [archivos de búfer de protocolo]()<!--add-valid-link.md)--><!--https://github.com/Azure/azree-video-analyzer/tree/master/contracts/grpc-->. 

**¿Cómo puedo asegurarme de que estoy usando los archivos de búfer de protocolo más recientes?** 

Puede obtener los archivos de búfer de protocolo más recientes en el [sitio de archivos de contrato]()<!--add-valid-link.md)--><!--https://github.com/Azure/azree-video-analyzer/tree/master/contracts/grpc-->. Cada vez que se actualicen los archivos de contrato, estarán en esta ubicación. No hay ningún plan inmediato para actualizar los archivos de protocolo, por lo que debe buscar el nombre del paquete en la parte superior de los archivos para conocer la versión. Debería decir esto: 

```
microsoft.azure.media.live_video_analytics.extensibility.grpc.v1
```

Todas las actualizaciones de estos archivos harán aumentar la porción "v-valor" al final del nombre. 

> [!NOTE]
> Dado que Video Analyzer usa la versión proto3 del lenguaje, los campos son opcionales, y la versión es compatible con versiones anteriores y posteriores. 

**¿Qué características de gRPC se pueden usar con Video Analyzer? ¿Qué características son obligatorias y cuáles son opcionales?** 

Se pueden usar todas las características de gRPC del lado servidor siempre que se cumpla el contrato Protocol Buffers (Protobuf). 

## <a name="monitoring-and-metrics"></a>Supervisión y métricas

**¿Se puede supervisar la canalización multimedia en el perímetro mediante Azure Event Grid?**

Sí. Puede consumir las [métricas de Prometheus](monitor-log-edge.md#azure-monitor-collection-via-telegraf) y publicarlas en la cuadrícula de eventos. 

**¿Se puede usar Azure Monitor para ver el estado, las métricas y el rendimiento de las canalizaciones en la nube o en el perímetro?**

Sí, se admite este enfoque. Para obtener más información, consulte [Información general sobre las métricas de Azure Monitor](../../azure-monitor/essentials/data-platform-metrics.md).

**¿Hay alguna herramienta que facilite la supervisión del módulo de IoT Edge de Azure Video Analyzer?**

Visual Studio Code admite la extensión Azure IoT Tools, que permite supervisar fácilmente los puntos de conexión del módulo perimetral de Video Analyzer. Puede usar esta herramienta para empezar a supervisar rápidamente el punto de conexión integrado de IoT Hub para "eventos" y ver los mensajes de inferencia que se enrutan desde el dispositivo perimetral hasta la nube. 

Además, puede usar esta extensión para editar el módulo gemelo para el módulo perimetral de Video Analyzer con el fin de modificar la configuración de la canalización.

Para más información, vea el artículo [Supervisión y registro](monitor-log-edge.md).

## <a name="billing-and-availability"></a>Facturación y disponibilidad

**¿Cómo se factura Azure Video Analyzer?**

Para obtener información sobre facturación, consulte los [Precios de Video Analyzer]()<!--add-valid-link.md)--><!--https://azure.microsoft.com/pricing/details/media-services/-->.

## <a name="next-steps"></a>Pasos siguientes

[Inicio rápido: Introducción a Azure Video Analyzer](get-started-detect-motion-emit-events.md)
