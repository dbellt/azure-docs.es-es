---
title: 'Supervisión y registro en Live Video Analytics: Azure'
description: En este artículo se proporciona información general sobre la supervisión y el registro en Azure Live Video Analytics en IoT Edge.
ms.topic: reference
ms.date: 04/27/2020
ms.openlocfilehash: 4bc69192d0d36d01755d625ee46cd77cd9eec1b9
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110372147"
---
# <a name="monitoring-and-logging-in-live-video-analytics-on-iot-edge"></a>Supervisión y registro en Live Video Analytics en Azure IoT Edge

[!INCLUDE [redirect to Azure Video Analyzer](./includes/redirect-video-analyzer.md)]

En este artículo, aprenderá a recibir eventos para la supervisión remota desde el módulo Live Video Analytics on IoT Edge. 

También aprenderá a controlar los registros que genera el módulo.

## <a name="taxonomy-of-events"></a>Taxonomía de eventos

Live Video Analytics on IoT Edge emite eventos o datos de telemetría según la siguiente taxonomía:

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/telemetry-schema/taxonomy.png" alt-text="Diagrama que muestra la taxonomía de eventos.":::

* Operativo: eventos generados por las acciones de un usuario o durante la ejecución de un [grafo multimedia](media-graph-concept.md)
   
   * Volumen: se espera que sea bajo (algunas veces un minuto o incluso menos).
   * Ejemplos:

      - Grabación iniciada (se muestra en el ejemplo siguiente)
      - Grabación detenida.
      
      ```
      {
        "body": {
          "outputType": "assetName",
          "outputLocation": "sampleAssetFromEVR-LVAEdge-20200512T233309Z"
        },
        "applicationProperties": {
          "topic": "/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<my-resource-group>/providers/microsoft.media/mediaservices/<ams-account-name>",
          "subject": "/graphInstances/Sample-Graph-2/sinks/assetSink",
          "eventType": "Microsoft.Media.Graph.Operational.RecordingStarted",
          "eventTime": "2020-05-12T23:33:10.392Z",
          "dataVersion": "1.0"
        }
      }
      ```
* Diagnóstico: eventos que ayudan a diagnosticar problemas de rendimiento.

   * Volumen: puede ser alto (varias veces por minuto).
   * Ejemplos:
   
      - Información de [SDP](https://en.wikipedia.org/wiki/Session_Description_Protocol) de RTSP (se muestra en el ejemplo siguiente) 
      - Intervalos en la fuente de vídeo entrante.

      ```
      {
        "body": {
          "sdp": "SDP:\nv=0\r\no=- 1589326384077235 1 IN IP4 XXX.XX.XX.XXX\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/lots_015.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.04.12\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-73.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/lots_015.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=640028;sprop-parameter-sets=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\r\na=control:track1\r\n"
        },
        "applicationProperties": {
          "topic": "/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<my-resource-group>/providers/microsoft.media/mediaservices/<ams-account-name>",
          "subject": "/graphInstances/Sample-Graph-2/sources/rtspSource",
          "eventType": "Microsoft.Media.Graph.Diagnostics.MediaSessionEstablished",
          "eventTime": "2020-05-12T23:33:04.077Z",
          "dataVersion": "1.0"
        }
      }
      ```
* Análisis: eventos generados como parte del análisis de vídeo.

   * Volumen: puede ser alto (varias veces por minuto o más).
   * Ejemplos:
      
      - Movimiento detectado (se muestra en el ejemplo siguiente) 
      - Resultado de la inferencia.

   ```      
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
       "topic": "/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<my-resource-group>/providers/microsoft.media/mediaservices/<ams-account-name>",
       "subject": "/graphInstances/Sample-Graph-2/processors/md",
       "eventType": "Microsoft.Media.Graph.Analytics.Inference",
       "eventTime": "2020-05-12T23:33:09.381Z",
       "dataVersion": "1.0"
     }
   }
   ```

Los eventos emitidos por el módulo se envían al [centro de IoT Edge](../../iot-edge/iot-edge-runtime.md#iot-edge-hub). Desde allí, se pueden enrutar a otros destinos. 

### <a name="timestamps-in-analytic-events"></a>Marcas de tiempo de los eventos de análisis

Como se indicó anteriormente, los eventos generados como parte del análisis de vídeo tienen marcas de tiempo asociadas. Si [grabó el vídeo en directo](video-recording-concept.md) como parte de la topología de grafos, esta marca de tiempo le ayuda a encontrar dónde se produjo en el vídeo grabado ese evento determinado. A continuación, se muestran instrucciones para asignar la marca de tiempo de un evento de análisis a la escala de tiempo del vídeo grabado en un [recurso de Azure Media Services](terminology.md#asset).

En primer lugar, extraiga el valor de `eventTime`. Use este valor en un [filtro de intervalo de tiempo](playback-recordings-how-to.md#time-range-filters) para recuperar una parte adecuada de la grabación. Por ejemplo, podría querer recuperar un vídeo que se inicie 30 segundos antes de `eventTime` y que finalice 30 segundos después. En el ejemplo anterior, donde `eventTime` es 2020-05-12T23:33:09.381 Z, una solicitud de un manifiesto HLS de los 30 segundos anteriores y posteriores a `eventTime` se parecería a esta:

```
https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=m3u8-aapl,startTime=2020-05-12T23:32:39Z,endTime=2020-05-12T23:33:39Z).m3u8
```

La dirección URL anterior devolvería una [lista de reproducción maestra](https://developer.apple.com/documentation/http_live_streaming/example_playlists_for_http_live_streaming) que contiene direcciones URL de las listas de reproducción multimedia. La lista de reproducción multimedia contendría entradas como esta:

```
...
#EXTINF:3.103011,no-desc
Fragments(video=143039375031270,format=m3u8-aapl)
...
```
La entrada anterior informa de que hay disponible un fragmento de vídeo que comienza en un valor de `timestamp` de `143039375031270`. El valor de `timestamp` del evento de análisis usa la misma escala de tiempo que la lista de reproducción multimedia. Se puede usar para identificar el fragmento de vídeo pertinente y buscar el fotograma correcto.

Para más información, consulte estos [artículos sobre la búsqueda con precisión de fotograma](https://www.bing.com/search?q=frame+accurate+seeking+in+HLS) en HLS.

## <a name="controlling-events"></a>Control de eventos

Puede usar las siguientes propiedades del módulo gemelo para controlar los eventos operativos y de diagnóstico publicados por el módulo Live Video Analytics on IoT Edge. Estas propiedades se documentan en el [esquema JSON del módulo gemelo](module-twin-configuration-schema.md).

- `diagnosticsEventsOutputName`: para obtener eventos de diagnóstico del módulo, incluya esta propiedad y proporcione un valor para ella. Para que el módulo deje de publicar eventos de diagnóstico, omita esta información o déjela en blanco.
   
- `operationalEventsOutputName`: para obtener eventos operativos del módulo, incluya esta propiedad y proporcione un valor para ella. Para que el módulo deje de publicar eventos operativos, omita esta información o déjela en blanco.
   
Los nodos, como el procesador de detección de movimiento o el procesador de extensiones HTTP, generan los eventos de análisis. El receptor de IoT Hub se usa para enviarlos al centro de IoT Edge. 

Puede controlar el [enrutamiento de todos los eventos anteriores](../../iot-edge/module-composition.md#declare-routes) mediante la propiedad `desired` del módulo gemelo `$edgeHub` en el manifiesto de implementación:

```
 "$edgeHub": {
   "properties.desired": {
     "schemaVersion": "1.0",
     "routes": {
       "moduleToHub": "FROM /messages/modules/lvaEdge/outputs/* INTO $upstream"
     },
     "storeAndForwardConfiguration": {
       "timeToLiveSecs": 7200
     }
   }
 }
```

En el código JSON anterior, `lvaEdge` es el nombre del módulo Live Video Analytics on IoT Edge. La regla de enrutamiento sigue el esquema definido en [Declaración de rutas](../../iot-edge/module-composition.md#declare-routes).

> [!NOTE]
> Para garantizar que los eventos de análisis llegan al centro de IoT Edge, debe haber un nodo receptor de IoT Hub en sentido descendente con respecto a cualquier nodo procesador de detección de movimiento o nodo procesador de extensiones HTTP.

## <a name="event-schema"></a>Esquema del evento

Los eventos se originan en el dispositivo perimetral y se pueden consumir tanto en el perímetro como en la nube. Los eventos generados por Live Video Analytics on IoT Edge se ajustan al [patrón de mensajería de streaming](../../iot-hub/iot-hub-devguide-messages-construct.md) establecido por Azure IoT Hub. El patrón consta de propiedades del sistema, propiedades de la aplicación y un cuerpo.

### <a name="summary"></a>Resumen

Cada evento, cuando se observa mediante IoT Hub, tiene un conjunto de propiedades comunes:

|Propiedad   |Tipo de propiedad| Tipo de datos   |Descripción|
|---|---|---|---|
|`message-id`   |sistema |guid|  Identificador de evento único|
|`topic`|   applicationProperty |string|    Ruta de acceso de Azure Resource Manager de la cuenta de Azure Media Services.|
|`subject`| applicationProperty |string|    Subruta de la entidad que emite el evento.|
|`eventTime`|   applicationProperty|    string| Hora a la que se generó el evento|
|`eventType`|   applicationProperty |string|    Identificador del tipo de evento. (Consulte la sección siguiente).|
|`body`|body    |object|    Datos de evento concretos|
|`dataVersion`  |applicationProperty|   string  |{Major}.{Minor}|

### <a name="properties"></a>Propiedades

#### <a name="message-id"></a>message-id

Identificador único global (GUID) del evento.

#### <a name="topic"></a>topic

Representa la cuenta de Azure Media Services asociada al grafo.

`/subscriptions/{subId}/resourceGroups/{rgName}/providers/Microsoft.Media/mediaServices/{accountName}`

#### <a name="subject"></a>subject

La entidad que emite el evento:

`/graphInstances/{graphInstanceName}`<br/>
`/graphInstances/{graphInstanceName}/sources/{sourceName}`<br/>
`/graphInstances/{graphInstanceName}/processors/{processorName}`<br/>
`/graphInstances/{graphInstanceName}/sinks/{sinkName}`

La propiedad `subject` permite asignar eventos genéricos al módulo generador. Por ejemplo, para un nombre de usuario o contraseña RTSP no válidos, el evento generado sería `Microsoft.Media.Graph.Diagnostics.ProtocolError` en el nodo `/graphInstances/myGraph/sources/myRtspSource`.

#### <a name="event-types"></a>Tipos de eventos

Los tipos de eventos se asignan a un espacio de nombres de acuerdo con este esquema:

`Microsoft.Media.Graph.{EventClass}.{EventType}`

#### <a name="event-classes"></a>Clases de eventos

|Nombre de la clase|Descripción|
|---|---|
|Análisis  |Eventos generados como parte del análisis de contenido|
|Diagnóstico    |Eventos que ayudan a diagnosticar problemas y rendimiento.|
|Operativos    |Eventos generados como parte de las operaciones de recursos|

Los tipos de evento son específicos de cada clase de evento.

Ejemplos:

* `Microsoft.Media.Graph.Analytics.Inference`
* `Microsoft.Media.Graph.Diagnostics.AuthorizationError`
* `Microsoft.Media.Graph.Operational.GraphInstanceStarted`

### <a name="event-time"></a>Hora del evento

La hora del evento tiene el formato de una cadena ISO 8601. Representa la hora en que se produjo el evento.

### <a name="azure-monitor-collection-via-telegraf"></a>Colección de Azure Monitor a través de Telegraf

Estas métricas se notificarán desde el módulo Live Video Analytics on IoT Edge:  

|Nombre de métrica|Tipo|Etiqueta|Descripción|
|-----------|----|-----|-----------|
|lva_active_graph_instances|Indicador|iothub, edge_device, module_name, graph_topology|Número total de grafos activos por topología.|
|lva_received_bytes_total|Contador|iothub, edge_device, module_name, graph_topology, graph_instance, graph_node|Número total de bytes recibidos por un nodo. Solo se admite para orígenes RTSP.|
|lva_data_dropped_total|Contador|iothub, edge_device, module_name, graph_topology, graph_instance, graph_node, data_kind|Contador de los datos descartados (eventos, medios, etc.)|

> [!NOTE]
> Un [punto de conexión de Prometheus](https://prometheus.io/docs/practices/naming/) se expone en el puerto 9600 del contenedor. Si asigna al módulo Live Video Analytics on IoT Edge el nombre "IvaEdge", se podrá acceder a las métricas enviando una solicitud GET a http://lvaEdge:9600/metrics.   

Siga estos pasos para habilitar la recopilación de métricas de Live Video Analytics en el módulo de IoT Edge:

1. Cree una carpeta en el equipo de desarrollo y vaya a esa carpeta.

1. En la carpeta, cree un archivo `telegraf.toml` que contenga las siguientes configuraciones:
    ```
    [agent]
        interval = "30s"
        omit_hostname = true

    [[inputs.prometheus]]
      metric_version = 2
      urls = ["http://edgeHub:9600/metrics", "http://edgeAgent:9600/metrics", "http://{LVA_EDGE_MODULE_NAME}:9600/metrics"]

    [[outputs.azure_monitor]]
      namespace_prefix = "lvaEdge"
      region = "westus"
      resource_id = "/subscriptions/{SUBSCRIPTON_ID}/resourceGroups/{RESOURCE_GROUP}/providers/Microsoft.Devices/IotHubs/{IOT_HUB_NAME}"
    ```
    > [!IMPORTANT]
    > Asegúrese de reemplazar las variables en el archivo .toml. Las variables se representan mediante llaves (`{}`).

1. En la misma carpeta, cree un Dockerfile que contenga los siguientes comandos:
    ```
        FROM telegraf:1.15.3-alpine
        COPY telegraf.toml /etc/telegraf/telegraf.conf
    ```

1. Use los comandos de la CLI de Docker para crear el archivo de Docker y publique la imagen en Azure Container Registry.
    
   Para más información sobre el uso de la CLI de Docker para realizar inserciones en un registro de contenedor, consulte [Inserción y extracción de imágenes de Docker](../../container-registry/container-registry-get-started-docker-cli.md). Para más información sobre Azure Container Registry, consulte la [documentación](../../container-registry/index.yml).


1. Tras la inserción en Azure Container Registry, agregue el siguiente nodo al archivo de manifiesto de implementación:
    ```
    "telegraf": 
    {
      "settings": 
        {
            "image": "{AZURE_CONTAINER_REGISTRY_LINK_TO_YOUR_TELEGRAF_IMAGE}"
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
     > A la entidad de servicio se le puede otorgar el rol **Publicador de métricas de supervisión**. Siga los pasos descritos en **[Creación de una entidad de servicio](../../azure-arc/data/upload-metrics-and-logs-to-azure-monitor.md?pivots=client-operating-system-macos-and-linux#create-service-principal)** para crear la entidad de servicio y asignar el rol.

1. Una vez implementados los módulos, las métricas aparecerán en Azure Monitor en un espacio de nombres único. Los nombres de métricas coincidirán con los emitidos por Prometheus. 

   En este caso, en Azure Portal, vaya al centro de IoT y seleccione **Métricas** en el panel izquierdo. Allí debería ver las métricas.

### <a name="log-analytics-metrics-collection"></a>Recopilación de métricas de Log Analytics
Si usa un [punto de conexión de Prometheus](https://prometheus.io/docs/practices/naming/) junto con [Log Analytics](../../azure-monitor/logs/log-analytics-tutorial.md), puede generar y [supervisar métricas](../../azure-monitor/essentials/metrics-supported.md), como el porcentaje de CPU usado, el porcentaje usado de memoria, etc.   

> [!NOTE]
> La configuración siguiente no recopila registros, **solo métricas**. Es factible ampliar el módulo recopilador para que también recopile y cargue registros.

[ ![Diagrama que muestra la recopilación de métricas mediante Log Analytics.](./media/telemetry-schema/log-analytics.png)](./media/telemetry-schema/log-analytics.png#lightbox)

1. Aprenda a [recopilar métricas](https://github.com/Azure/iotedge/tree/master/edge-modules/MetricsCollector).
1. Use los comandos de la CLI de Docker para crear el [archivo de Docker](https://github.com/Azure/iotedge/tree/master/edge-modules/MetricsCollector/docker/linux) y publique la imagen en Azure Container Registry.
    
   Para más información sobre el uso de la CLI de Docker para realizar inserciones en un registro de contenedor, consulte [Inserción y extracción de imágenes de Docker](../../container-registry/container-registry-get-started-docker-cli.md). Para más información sobre Azure Container Registry, consulte la [documentación](../../container-registry/index.yml).

1. Tras la inserción en Azure Container Registry, se inserta lo siguiente en el manifiesto de implementación:
    ```json
    "azmAgent": {
      "settings": {
        "image": "{AZURE_CONTAINER_REGISTRY_LINK_TO_YOUR_METRICS_COLLECTOR}"
      },
      "type": "docker",
      "version": "1.0",
      "status": "running",
      "restartPolicy": "always",
      "env": {
        "LogAnalyticsWorkspaceId": { "value": "{YOUR_LOG_ANALYTICS_WORKSPACE_ID}" },
        "LogAnalyticsSharedKey": { "value": "{YOUR_LOG_ANALYTICS_WORKSPACE_SECRET}" },
        "LogAnalyticsLogType": { "value": "IoTEdgeMetrics" },
        "MetricsEndpointsCSV": { "value": "http://edgeHub:9600/metrics,http://edgeAgent:9600/metrics,http://lvaEdge:9600/metrics" },
        "ScrapeFrequencyInSecs": { "value": "30 " },
        "UploadTarget": { "value": "AzureLogAnalytics" }
      }
    }
    ```
    > [!NOTE]
    > Los módulos `edgeHub`, `edgeAgent` y `lvaEdge` son los nombres de los módulos definidos en el archivo de manifiesto de implementación. Asegúrese de que los nombres de los módulos coinciden.   

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
    | where module_name in ("lvaEdge","yolov3","tinyyolov3")
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
   * Si el valor se establece en `None`, no se generará ningún registro. (No se recomienda esta configuración).
   * Use `Verbose` solo si necesita compartir registros con el soporte técnico de Azure para diagnosticar un problema.

* `logCategories`

   * Lista separada por comas de uno o varios de estos valores: `Application`, `Events`, `MediaPipeline`.
   * El valor predeterminado es `Application, Events`.
   * `Application`: información de alto nivel del módulo, como mensajes de inicio del módulo, errores del entorno y llamadas a métodos directos.
   * `Events`: todos los eventos descritos anteriormente en este artículo.
   * `MediaPipeline`: registros de bajo nivel que pueden ofrecer información detallada cuando está solucionando problemas, como dificultades para establecer una conexión con una cámara compatible con RTSP.
   
### <a name="generating-debug-logs"></a>Generación de registros de depuración

En algunos casos, para ayudar al soporte técnico de Azure a resolver un problema, puede que tenga que generar registros más detallados que los descritos anteriormente. Para generar estos registros:

1. [Vincule el almacenamiento del módulo con el almacenamiento del dispositivo](../../iot-edge/how-to-access-host-storage-from-module.md#link-module-storage-to-device-storage) mediante `createOptions`. Si examina una [plantilla de manifiesto de implementación](https://github.com/Azure-Samples/live-video-analytics-iot-edge-csharp/blob/master/src/edge/deployment.template.json) de los inicios rápidos, verá este código:

   ```
   "createOptions": {
     …
     "Binds": [
       "/var/local/mediaservices/:/var/lib/azuremediaservices/"
     ]
    }
   ```

   Este código permite que el módulo de Edge escriba registros en la ruta de acceso de almacenamiento del dispositivo `/var/local/mediaservices/`. 

 1. Agregue la siguiente propiedad `desired` al módulo:

    `"debugLogsDirectory": "/var/lib/azuremediaservices/debuglogs/",`

El módulo escribirá ahora los registros de depuración en un formato binario en la ruta de acceso de almacenamiento del dispositivo `/var/local/mediaservices/debuglogs/`. Puede compartir estos registros con el soporte técnico de Azure.

## <a name="faq"></a>Preguntas más frecuentes

Si tiene alguna pregunta, consulte las [preguntas frecuentes sobre supervisión y métricas](faq.md#monitoring-and-metrics).

## <a name="next-steps"></a>Pasos siguientes

[Grabación de vídeo continua](continuous-video-recording-tutorial.md)
