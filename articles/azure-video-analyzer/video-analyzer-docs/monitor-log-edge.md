---
title: 'Supervisión y registro: Azure'
description: En este artículo se proporciona información general sobre la supervisión y el registro de Azure Video Analyzer.
ms.topic: how-to
ms.date: 04/27/2020
ms.openlocfilehash: d7f048aecd89d75ad7bff728bc8a4ddebc8f515a
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388668"
---
# <a name="monitor-and-log-on-iot-edge"></a>Supervisión y registro en IoT Edge

En este artículo, aprenderá a recibir eventos para la supervisión remota desde el módulo de Azure Video Analyzer IoT Edge. 

También aprenderá a controlar los registros que genera el módulo.

## <a name="taxonomy-of-events"></a>Taxonomía de eventos

Video Analyzer basado en IoT Edge emite eventos o datos de telemetría según la siguiente taxonomía:

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/telemetry-schema/taxonomy.png" alt-text="Diagrama que muestra la taxonomía de eventos.":::

* Operativo: eventos generados por las acciones de un usuario o durante la ejecución de una [canalización](pipeline.md).
  
   * Volumen: se espera que sea bajo (algunas veces un minuto o incluso menos).
   * Ejemplos:

      - Canalización en vivo activada
      
      - Canalización en vivo desactivada
      
      *Evento operativo de ejemplo*
      
      ```json
      {
         "body": {
             "outputType": "filePath",
             "outputLocation": "/var/media/Sample-1-fileSink/sampleFilesFromEVR-motion-fileSinkOutput-20210426T181911Z.mp4"
           },
           "properties": {
             "topic": "/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<my-resource-group>/providers/Microsoft.Media/videoAnalyzers/<account-name>",
             "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/sinks/fileSink",
             "eventType": "Microsoft.VideoAnalyzer.Operational.RecordingStarted",
             "eventTime": "2021-04-26T18:19:13.298Z",
             "dataVersion": "1.0"
          },
      }
      ```
* Diagnóstico: eventos que ayudan a diagnosticar problemas de rendimiento.

   * Volumen: puede ser alto (varias veces por minuto).
   * Ejemplos:
   
      - Información de [SDP](https://en.wikipedia.org/wiki/Session_Description_Protocol) de RTSP (se muestra en el ejemplo siguiente)       
      - Errores cuando se produce un error en una conexión a algún elemento, como una cámara o una extensión de IA
      
    *Evento de diagnóstico de ejemplo*
  
    ```json
    {
      "body": {
        "sdp": "SDP:\nv=0\r\no=- 1589326384077235 1 IN IP4 XXX.XX.XX.XXX\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/lots_015.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.04.12\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-73.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/lots_015.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=640028;sprop-parameter-sets=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\r\na=control:track1\r\n"
      },
      "applicationProperties": {
        "topic": "/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<my-resource-group>/providers/Microsoft.Media/videoAnalyzers/<account-name>",
        "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/sources/rtspSource",
        "eventType": "Microsoft.VideoAnalyzer.Diagnostics.MediaSessionEstablished",
        "eventTime": "2021-04-26T18:15:13.298Z",
        "dataVersion": "1.0"
      }
    }
    ```
* Análisis: eventos generados como resultado del análisis de vídeo

   * Volumen: puede ser alto (varias veces por minuto o más).
   * Ejemplos:
     
      - Movimiento detectado (se muestra en el ejemplo siguiente) 
      
      - Resultado de la inferencia.
     
      *Evento de análisis de ejemplo*
        
      ```json
      {
        "body": {
          "timestamp": 143039375044290,
          "inferences": [
            {
              "type": "motion",
              "motion": {
                "box": {
                  "l": 0.48954,
                  "t": 0.140741,
                  "w": 0.075,
                  "h": 0.058824
                }
              }
            }
          ]
        },
        "applicationProperties": {
          "topic": "/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<my-resource-group>/providers/Microsoft.Media/videoAnalyzers/<account-name>",
          "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/processors/md",
          "eventType": "Microsoft.VideoAnalyzer.Analytics.Inference",
          "eventTime": "2021-04-26T18:15:13.298Z",
          "dataVersion": "1.0"
        }
      }
      ```
        
Los eventos emitidos por el módulo se envían al [centro de IoT Edge](../../iot-edge/iot-edge-runtime.md#iot-edge-hub). Desde este punto, se pueden enrutar a otros destinos. 

### <a name="events-and-video-playback"></a>Eventos y reproducción de vídeo

Como se indicó anteriormente, los eventos generados como parte del análisis de vídeo tienen un valor de `eventTime` asociado. Si [grabó el vídeo en directo](video-recording.md) como parte de la topología de canalización, este valor le ayuda a encontrar en qué punto del vídeo grabado se produjo ese evento determinado. Puede cargar la grabación de vídeo en el [widget del reproductor de Video Analyzer](player-widget.md) y usar sus controles para buscar la fecha y hora de su interés. Si la canalización implicaba el uso de un modelo de IA para generar resultados de inferencia, debería grabar los datos de inferencia junto con el vídeo. Esto le permitirá reproducir los metadatos de inferencia junto con el vídeo, como se muestra en este [tutorial](record-stream-inference-data-with-video.md).


## <a name="routing-events"></a>Enrutamiento de eventos

Puede usar las siguientes propiedades del módulo gemelo para enrutar los eventos operativos y de diagnóstico publicados por el módulo de Video Analyzer. Estas propiedades se documentan en el [esquema JSON del módulo gemelo](module-twin-configuration-schema.md).

- `diagnosticsEventsOutputName`: para obtener eventos de diagnóstico del módulo, incluya esta propiedad y proporcione un valor para ella. Para que el módulo deje de publicar eventos de diagnóstico, omita esta información o déjela en blanco.
  
- `operationalEventsOutputName`: para obtener eventos operativos del módulo, incluya esta propiedad y proporcione un valor para ella. Para que el módulo deje de publicar eventos operativos, omita esta información o déjela en blanco.

Los nodos como el procesador de detección de movimiento o el procesador de extensiones emiten automáticamente los eventos de diagnóstico y operativos. Los eventos de análisis se deben enrutar dentro de una canalización a un receptor de mensajes de IoT Hub para que se envíen al centro de IoT Edge. 

Puede controlar el [enrutamiento de todos los eventos anteriores](../../iot-edge/module-composition.md#declare-routes) mediante la propiedad `desired` del módulo gemelo `$edgeHub` en el manifiesto de implementación:

```json
 "$edgeHub": {
   "properties.desired": {
     "schemaVersion": "1.0",
     "routes": {
       "moduleToHub": "FROM /messages/modules/avaedge/outputs/* INTO $upstream"
     },
     "storeAndForwardConfiguration": {
       "timeToLiveSecs": 7200
     }
   }
 }
```

En el código JSON anterior, `avaedge` es el nombre del módulo de Video Analyzer. La regla de enrutamiento sigue el esquema definido en [Declaración de rutas](../../iot-edge/module-composition.md#declare-routes).

> [!NOTE]
> Para garantizar que los eventos de análisis llegan al centro de IoT Edge, debe haber un nodo receptor de mensajes de IoT Hub en el nivel inferior a cualquier nodo procesador de detección de movimiento o nodo procesador de extensiones.

## <a name="event-schema"></a>Esquema del evento

Los eventos que se originan en el dispositivo perimetral se pueden consumir tanto en el borde como en la nube. Los eventos generados por Video Analyzer se ajustan al [patrón de mensajería de streaming](../../iot-hub/iot-hub-devguide-messages-construct.md) establecido por Azure IoT Hub. El patrón consta de propiedades del sistema, propiedades de la aplicación y un cuerpo.

### <a name="summary"></a>Resumen

Cada evento, cuando se observa mediante IoT Hub, tiene un conjunto de propiedades comunes:

| Propiedad      | Tipo de propiedad       | Tipo de datos | Descripción                                                  |
| ------------- | ------------------- | --------- | ------------------------------------------------------------ |
| `message-id`  | sistema              | guid      | Identificador de evento único                                             |
| `topic`       | applicationProperty | string    | Ruta de acceso de Azure Resource Manager para la cuenta de Azure Video Analyzer. |
| `subject`     | applicationProperty | string    | Subruta de la entidad que emite el evento.                    |
| `eventTime`   | applicationProperty | string    | Hora a la que se generó el evento                                |
| `eventType`   | applicationProperty | string    | Identificador del tipo de evento. (Consulte la sección siguiente).          |
| `body`        | body                | object    | Datos de evento concretos                                       |
| `dataVersion` | applicationProperty | string    | {Major}.{Minor}                                              |

### <a name="properties"></a>Propiedades

#### <a name="message-id"></a>message-id

Identificador único global (GUID) del evento.

#### <a name="topic"></a>topic

Representa el módulo perimetral de Video Analyzer que generó el evento. Ejemplo:

`/subscriptions/{subId}/resourceGroups/{rgName}/providers/Microsoft.Media/videoAnalyzers/{accountName}`

#### <a name="subject"></a>subject

Entidad que emite el evento. Ejemplos:

`/edgeModules/avaedge/livePipelines/{livePipelineName}`<br/>
`/edgeModules/avaedge/livePipelines/{livePipelineName}/sources/{nodeName}`<br/>
`/edgeModules/avaedge/livePipelines/{livePipelineName}/processors/{nodeName}`<br/>
`/edgeModules/avaedge/livePipelines/{livePipelineName}/sinks/{nodeName}`

La propiedad `subject` permite asignar eventos genéricos al nodo de la canalización en vivo que emitió el evento. Por ejemplo, para un nombre de usuario o contraseña de RTSP no válidos, el evento generado sería `Microsoft.VideoAnalyzer.Diagnostics.ProtocolError` en el nodo `/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/sources/rtspSource`.

#### <a name="eventtype"></a>eventType

Los tipos de eventos se asignan a un espacio de nombres de acuerdo con este esquema:

`Microsoft.VideoAnalyzer.{EventClass}.{EventType}`

La clase de eventos puede ser una de las siguientes:

| Nombre de la clase  | Descripción                                                  |
| ----------- | ------------------------------------------------------------ |
| Análisis   | Eventos generados como parte del análisis de contenido                |
| Diagnóstico | Eventos que ayudan a diagnosticar problemas y rendimiento. |
| Operativos | Eventos generados como parte de las operaciones de recursos              |

Ejemplos:

* `Microsoft.VideoAnalyzer.Analytics.Inference`
* `Microsoft.VideoAnalyzer.Diagnostics.AuthorizationError`
* `Microsoft.VideoAnalyzer.Operational.RecordingStarted`

#### <a name="eventtime"></a>eventTime

La hora del evento tiene el formato de una cadena ISO 8601. Representa la hora en que se produjo el evento.

Ejemplo:

`2021-04-26T18:15:13.298Z`

## <a name="metrics"></a>Métricas

Estas métricas se notifican desde el módulo de Video Analyzer a través de un [punto de conexión de Prometheus](https://prometheus.io/docs/instrumenting/exposition_formats/#text-based-format) que se ejecuta en el puerto 9600 del módulo.

| Nombre de métrica                           | Tipo    | Etiquetas                                                                              | Descripción                                                  |
| ------------------------------------- | ------- | ----------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| va_active_live_pipelines              | Medidor   | iothub, edge_device, module_name, pipeline_topology                                 | Número total de canalizaciones en vivo activas por topología.          |
| va_cumulative_latency_seconds_average | Medidor   | iothub, edge_device, module_name, pipeline_topology, live_pipeline, node            | Latencia media desde el momento en que un origen recibió un fotograma de vídeo hasta el momento en que dicho fotograma salió del nodo durante el último minuto. Esto no se notifica si no hubo ningún fotograma. |
| va_cumulative_latency_seconds_max     | Medidor   | iothub, edge_device, module_name, pipeline_topology, live_pipeline, node            | Latencia máxima desde el momento en que un origen recibió un fotograma de vídeo hasta el momento en que dicho fotograma salió del nodo durante el último minuto. Esto no se notifica si no hubo ningún fotograma. |
| va_data_dropped_total                 | Contador | iothub, edge_device, module_name, pipeline_topology, live_pipeline, node, data_kind | Contador de los datos descartados (eventos, medios, etc.)      |
| va_received_bytes_total               | Contador | iothub, edge_device, module_name, pipeline_topology, live_pipeline, node            | Número total de bytes recibidos por un nodo. Solo se admite para orígenes RTSP. |


> [!NOTE]
> Si asigna el nombre `avaedge` al módulo de Video Analyzer, se puede enviar una solicitud GET a `http://avaedge:9600/metrics` para acceder a las métricas. Según la implementación, es posible que deba asignar este puerto al dispositivo para que sea accesible.

### <a name="azure-monitor-collection-via-telegraf"></a>Colección de Azure Monitor a través de Telegraf 

Siga estos pasos para habilitar la recopilación de métricas del módulo de Video Analyzer. [Telegraf](https://github.com/influxdata/telegraf) recopilará las métricas y, a continuación, se cargarán en [Azure Monitor](../../azure-monitor/overview.md) como una [métrica personalizada](../../azure-monitor/essentials/metrics-custom-overview.md). En Telegraf, el complemento [Prometheus](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/prometheus) extraerá los registros y el complemento [azure_monitor](https://github.com/influxdata/telegraf/tree/master/plugins/outputs/azure_monitor) los cargará.

A continuación, el agente se podrá usar para recopilar métricas de cualquier módulo que exponga un punto de conexión de Prometheus.

1. Cree una carpeta en el equipo de desarrollo y vaya a esa carpeta.
1. En la carpeta, cree un archivo `telegraf.toml` que contenga las siguientes configuraciones:
    ```toml
    [agent]
        interval = "30s"
        omit_hostname = true

    [[inputs.prometheus]]
      metric_version = 2
      urls = ["http://edgeHub:9600/metrics", "http://edgeAgent:9600/metrics", "http://{AVA_EDGE_MODULE_NAME}:9600/metrics"]

    [[outputs.azure_monitor]]
      namespace_prefix = "{AVA_EDGE_MODULE_NAME}"
      region = "westus2"
      resource_id = "/subscriptions/{SUBSCRIPTON_ID}/resourceGroups/{RESOURCE_GROUP}/providers/Microsoft.Devices/IotHubs/{IOT_HUB_NAME}"
    ```
    > [!IMPORTANT]
    > Asegúrese de reemplazar las variables en el archivo .toml. Las variables se representan mediante llaves (`{}`). Además, actualice el valor de `region`.
1. En la misma carpeta, cree un Dockerfile que contenga los siguientes comandos:
    ```docker
    FROM telegraf:1.15.3-alpine
    COPY telegraf.toml /etc/telegraf/telegraf.conf
    ```
1. Use los comandos de la CLI de Docker para crear el archivo de Docker y publique la imagen en Azure Container Registry.
   
   Para más información sobre el uso de la CLI de Docker para realizar inserciones en un registro de contenedor, consulte [Inserción y extracción de imágenes de Docker](../../container-registry/container-registry-get-started-docker-cli.md). Para obtener información sobre Azure Container Registry, consulte la [documentación](../../container-registry/index.yml).

   ```bash
   # Insert your own container image URL
   docker build . -t myregistry.azurecr.io/telegraf_agent
   docker push myregistry.azurecr.io/telegraf_agent
   ```

1. Tras la inserción en Azure Container Registry, agregue el siguiente nodo al archivo de manifiesto de implementación:
    ```json
    "telegraf": 
    {
      "settings": 
        {
            "image": "{AZURE_CONTAINER_REGISTRY_URL_OF_YOUR_TELEGRAF_IMAGE}"
        },
        "type": "docker",
        "version": "1.0",
        "status": "running",
        "restartPolicy": "always",
        "env": 
        {
            "AZURE_TENANT_ID": { "value": "{YOUR_TENANT_ID}" },
            "AZURE_CLIENT_ID": { "value": "{YOUR CLIENT_ID}" },
            "AZURE_CLIENT_SECRET": { "value": "{YOUR_CLIENT_SECRET}" }
        }
    }
    ```
    > [!IMPORTANT]
    > Asegúrese de reemplazar las variables en el archivo de manifiesto. Las variables se representan mediante llaves (`{}`).


   Azure Monitor se puede [autenticar mediante la entidad de servicio](https://github.com/influxdata/telegraf/blob/master/plugins/outputs/azure_monitor/README.md#azure-authentication).
        
   El complemento Telegraf de Azure Monitor expone [varios métodos de autenticación](https://github.com/influxdata/telegraf/blob/master/plugins/outputs/azure_monitor/README.md#azure-authentication). 
  1. Para usar la autenticación de la entidad de servicio, establezca estas variables de entorno:  
     `AZURE_TENANT_ID`: especifica el inquilino en el que se va a autenticar.  
     `AZURE_CLIENT_ID`: especifica el identificador de cliente de la aplicación que se va a usar.  
     `AZURE_CLIENT_SECRET`: especifica el secreto de la aplicación que se va a usar.  
     
     >[!TIP]
     >A la entidad de servicio se le puede otorgar el rol **Publicador de métricas de supervisión**. Siga los pasos descritos en **[Creación de una entidad de servicio](../../azure-arc/data/upload-metrics-and-logs-to-azure-monitor.md?pivots=client-operating-system-macos-and-linux#create-service-principal)** para crear la entidad de servicio y asignar el rol.
1. Una vez implementados los módulos, las métricas aparecerán en Azure Monitor en un espacio de nombres único. Los nombres de métricas coincidirán con los emitidos por Prometheus. 

   En este caso, en Azure Portal, vaya al centro de IoT y seleccione **Métricas** en el panel izquierdo. Allí debería ver las métricas. 

### <a name="log-analytics-metrics-collection"></a>Recopilación de métricas de Log Analytics

Si usa un [punto de conexión de Prometheus](https://prometheus.io/docs/practices/naming/) junto con [Log Analytics](../../azure-monitor/logs/log-analytics-tutorial.md), puede generar y [supervisar métricas](../../azure-monitor/essentials/metrics-supported.md), como el porcentaje de CPU usado, el porcentaje usado de memoria, etc.   

> [!NOTE]
> La configuración siguiente no recopila registros, **solo métricas**. Es factible ampliar el módulo recopilador para que también recopile y cargue registros.

[![Diagrama que muestra la recopilación de métricas mediante Log Analytics.](./media/telemetry-schema/log-analytics.svg)](./media/telemetry-schema/log-analytics.svg#lightbox)

1. Aprenda a [recopilar métricas](https://github.com/Azure/iotedge/tree/master/edge-modules/MetricsCollector).
1. Use los comandos de la CLI de Docker para crear el [archivo de Docker](https://github.com/Azure/iotedge/tree/master/edge-modules/MetricsCollector/docker/linux) y publique la imagen en Azure Container Registry.
   
   Para más información sobre el uso de la CLI de Docker para realizar inserciones en un registro de contenedor, consulte [Inserción y extracción de imágenes de Docker](../../container-registry/container-registry-get-started-docker-cli.md). Para más información sobre Azure Container Registry, consulte la [documentación](../../container-registry/index.yml).
1. Tras la inserción en Azure Container Registry, se inserta lo siguiente en el manifiesto de implementación:
    ```json
    "azmAgent": {
      "settings": {
        "image": "{AZURE_CONTAINER_REGISTRY_URL_OF_YOUR_METRICS_COLLECTOR}"
      },
      "type": "docker",
      "version": "1.0",
      "status": "running",
      "restartPolicy": "always",
      "env": {
        "LogAnalyticsWorkspaceId": { "value": "{YOUR_LOG_ANALYTICS_WORKSPACE_ID}" },
        "LogAnalyticsSharedKey": { "value": "{YOUR_LOG_ANALYTICS_WORKSPACE_SECRET}" },
        "LogAnalyticsLogType": { "value": "IoTEdgeMetrics" },
        "MetricsEndpointsCSV": { "value": "http://edgeHub:9600/metrics,http://edgeAgent:9600/metrics,http://avaedge:9600/metrics" },
        "ScrapeFrequencyInSecs": { "value": "30 " },
        "UploadTarget": { "value": "AzureLogAnalytics" }
      }
    }
    ```
    > [!NOTE]
    > Los módulos `edgeHub`, `edgeAgent` y `avaedge` son los nombres de los módulos definidos en el archivo de manifiesto de implementación. Asegúrese de que los nombres de los módulos coinciden.   

    Puede obtener los valores `LogAnalyticsWorkspaceId` y `LogAnalyticsSharedKey` mediante estos pasos:
    1. Vaya a Azure Portal.
    1. Busque sus áreas de trabajo de Log Analytics.
    1. Una vez que encuentre su área de trabajo de Log Analytics, vaya a la opción `Agents management` en el panel de navegación izquierdo.
    1. Encontrará el identificador del área de trabajo y las claves secretas que puede usar.

1. A continuación, cree un libro haciendo clic en la pestaña `Workbooks` del panel de navegación izquierdo.
1. Con el lenguaje de consulta Kusto, puede escribir consultas como se indica a continuación y obtener el porcentaje de CPU que usan los módulos de IoT Edge.
    ```kusto
    let cpu_metrics = IoTEdgeMetrics_CL
    | where Name_s == "edgeAgent_used_cpu_percent"
    | extend dimensions = parse_json(Tags_s)
    | extend module_name = tostring(dimensions.module_name)
    | where module_name in ("avaedge","yolov3","tinyyolov3")
    | summarize cpu_percent = avg(Value_d) by bin(TimeGenerated, 5s), module_name;
    cpu_metrics
    | summarize cpu_percent = sum(cpu_percent) by TimeGenerated
    | extend module_name = "Total"
    | union cpu_metrics
    ```

    [ ![Diagrama que muestra las métricas mediante la consulta de Kusto.](./media/telemetry-schema/metrics.png)](./media/telemetry-schema/metrics.png#lightbox)
## <a name="logging"></a>Registro

Al igual que sucede con otros módulos de IoT Edge, también se pueden [examinar los registros de contenedor](../../iot-edge/troubleshoot.md#check-container-logs-for-issues) en el dispositivo perimetral. Puede configurar la información que se escribe en los registros mediante las propiedades del [siguiente módulo gemelo](module-twin-configuration-schema.md):

* `logLevel`

   * Los valores permitidos son `Verbose`, `Information`, `Warning`, `Error` y `None`.
   * El valor predeterminado es `Information`. Los registros contendrán mensajes de error, de advertencia y de información.
   * Si el valor se establece en `Warning`, los registros contendrán mensajes de error y de advertencia.
   * Si el valor se establece en `Error`, los registros solo contendrán mensajes de error.
   * Si el valor se establece en `None`, no se generará ningún registro. Esto no es recomendable.
   * Use `Verbose` solo si necesita compartir registros con el soporte técnico de Azure para diagnosticar un problema.

* `logCategories`

   * Lista separada por comas de uno o varios de estos valores: `Application`, `Events`, `MediaPipeline`.
   * El valor predeterminado es `Application, Events`.
   * `Application`: información de alto nivel del módulo, como mensajes de inicio del módulo, errores del entorno y llamadas a métodos directos.
   * `Events`: todos los eventos descritos anteriormente en este artículo.
   * `MediaPipeline`: registros de bajo nivel que pueden ofrecer información detallada cuando está solucionando problemas, como dificultades para establecer una conexión con una cámara compatible con RTSP.
   
### <a name="generating-debug-logs"></a>Generación de registros de depuración

En algunos casos, para ayudar al soporte técnico de Azure a resolver un problema, puede que tenga que generar registros más detallados que los descritos anteriormente. Para generar estos registros:

1. [Vincule el almacenamiento del módulo con el almacenamiento del dispositivo](../../iot-edge/how-to-access-host-storage-from-module.md#link-module-storage-to-device-storage) mediante `createOptions`. Si examina una [plantilla de manifiesto de implementación](https://github.com/Azure-Samples/azure-video-analyzer-iot-edge-csharp/blob/master/src/edge/deployment.template.json) de los inicios rápidos, verá este código:

   ```json
   "createOptions": {
     …
     "Binds": [
       "/var/local/videoAnalyzer/:/var/lib/videoAnalyzer/"
     ]
    }
   ```

   Este código permite que el módulo de Edge escriba registros en la ruta de acceso de almacenamiento del dispositivo `/var/local/videoAnalyzer/`. 

 1. Agregue la siguiente propiedad `desired` al módulo:

    `"debugLogsDirectory": "/var/lib/videoAnalyzer/debuglogs/"`

El módulo escribirá ahora los registros de depuración en un formato binario en la ruta de acceso de almacenamiento del dispositivo `/var/local/videoAnalyzer/debuglogs/`. Puede compartir estos registros con el soporte técnico de Azure.

## <a name="faq"></a>Preguntas más frecuentes

Si tiene alguna pregunta, consulte las [preguntas frecuentes sobre supervisión y métricas](faq-edge.md#monitoring-and-metrics).

## <a name="next-steps"></a>Pasos siguientes

[Solución de problemas de Azure Video Analyzer](troubleshoot.md)
