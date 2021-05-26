---
title: 'Introducción a Azure Video Analyzer: Azure'
description: Este inicio rápido le guiará por los pasos necesarios para empezar a usar Azure Video Analyzer. Usa una máquina virtual de Azure como un dispositivo IoT Edge y una secuencia de vídeo en directo simulada.
ms.service: azure-video-analyzer
ms.topic: quickstart
ms.date: 04/21/2021
ms.openlocfilehash: 3606442101c8e20173ed3cd18c583fce02a45da1
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388453"
---
# <a name="quickstart-get-started-with-azure-video-analyzer"></a>Inicio rápido: Introducción a Azure Video Analyzer

Este inicio rápido le guiará por los pasos necesarios para empezar a usar Azure Video Analyzer. Usa una máquina virtual de Azure como un dispositivo IoT Edge y una secuencia de vídeo en directo simulada.

Después de completar los pasos de la configuración, podrá ejecutar la secuencia de vídeo en directo simulada mediante una canalización que detecta e informa de todos los movimientos de esa secuencia. El siguiente diagrama representa gráficamente esa canalización.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/get-started-detect-motion-emit-events/motion-detection.svg" alt-text="Detección de movimiento":::

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure que tenga una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), en caso de que aún no lo haya hecho.

    > [!NOTE]    
    > Necesitará una suscripción de Azure en la que tenga acceso a los roles [Colaborador](../../role-based-access-control/built-in-roles.md#contributor) y [Administrador de acceso de usuario](../../role-based-access-control/built-in-roles.md#user-access-administrator). Si no tiene los permisos adecuados, póngase en contacto con el administrador de la cuenta para que se los conceda.  
* [Visual Studio Code](https://code.visualstudio.com/) con las siguientes extensiones:

    * [Herramientas de Azure IoT](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)

> [!TIP] 
> Al instalar la extensión de Azure IoT Tools, es posible que se le pida que instale Docker. Si lo desea, ignore esta petición.

## <a name="set-up-azure-resources"></a>Configuración de los recursos de Azure

[![Implementación en Azure](https://aka.ms/deploytoazurebutton)](https://aka.ms/ava-click-to-deploy)

El proceso de implementación tardará unos **20 minutos**. Al finalizar, tendrá determinados recursos de Azure implementados en la suscripción de Azure, entre los que se incluyen:
1. **Cuenta de Video Analyzer**: este [servicio en la nube](overview.md) se usa para registrar el módulo perimetral Video Analyzer y para reproducir vídeo grabado y análisis de vídeo.
1. **Cuenta de almacenamiento**: para almacenar vídeo grabado y análisis de vídeo.
1. **Identidad administrada**: esta es la [identidad administrada] asignada por el usuario../../active-directory/managed-identities-azure-resources/overview.md) que se usa para administrar el acceso a la cuenta de almacenamiento anterior.
1. **Máquina virtual**: esta es una máquina virtual que actuará como su dispositivo perimetral simulado.
1. **IoT Hub**: funciona como un centro de mensajes común para la comunicación bidireccional entre la aplicación de IoT, los módulos de IoT Edge y los dispositivos que administra.

<!-- TODO: provide a link to the readme.md in github.com/azure-video-analyzer/setup/readme.md where we can list out all resources like virtual network etc. -->

## <a name="set-up-your-development-environment"></a>Configurado su entorno de desarrollo

### <a name="obtain-your-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

1. En Azure Portal, vaya a la instancia de IoT Hub que creó como parte del paso de configuración anterior.
1. Busque la opción **Directivas de acceso compartido** en el panel de navegación izquierdo y haga clic allí.
1. Haga clic en la directiva llamada **iothubowner**
1. Copie la **Cadena de conexión principal**: su aspecto será similar a `HostName=xxx.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=XXX`

### <a name="connect-to-the-iot-hub"></a>Conexión a IoT Hub

1. Abra Visual Studio Code, seleccione **Ver** > **Explorador**. O bien, seleccione Ctrl+Mayús+E.
1. En la esquina inferior izquierda de la pestaña **Explorador**, seleccione **Azure IoT Hub**.
1. Seleccione el icono **Más opciones** para ver el menú contextual. Luego, seleccione **Set IoT Hub Connection String** (Establecer cadena de conexión de IoT Hub).
1. Cuando aparezca un cuadro de entrada, escriba la cadena de conexión de IoT Hub.
1. En unos 30 segundos, actualice Azure IoT Hub en la sección inferior izquierda. Debería ver el dispositivo perimetral `avasample-iot-edge-device`, que debe tener los siguientes módulos implementados:
    * Módulo perimetral de Video Analyzer (nombre del módulo **avaedge**)
    * Simulador RTSP (nombre de módulo **rtspsim**)


> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/get-started-detect-motion-emit-events/modules-node.png" alt-text="Expansión del nodo Módulos":::

> [!TIP]
> Si ha [implementado manualmente Video Analyzer](deploy-iot-edge-device.md) en un dispositivo perimetral (por ejemplo, un dispositivo ARM64), verá que el módulo se muestra en ese dispositivo, bajo Azure IoT Hub. Puede seleccionar ese módulo y seguir el resto de los pasos siguientes.

### <a name="prepare-to-monitor-the-modules"></a>Preparación para supervisar los módulos 

Cuando use o ejecute este inicio rápido, los eventos se enviarán a IoT Hub. Para ver estos eventos, siga estos pasos:

1. En Visual Studio Code, abra la pestaña **Extensiones** (o presione Ctrl + Mayús + X) y busque **Azure IoT Hub**.
1. Haga clic con el botón derecho y seleccione la opción **Configuración de la extensión**.
 
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/get-started-detect-motion-emit-events/extension-settings.png" alt-text="Selección de la configuración de extensiones":::
1. Busque y habilite "Show Verbose Message" (Mostrar mensaje detallado).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/get-started-detect-motion-emit-events/verbose-message.png" alt-text="Show Verbose Message"::: (Mostrar mensaje detallado)
1. Abra el panel del explorador en Visual Studio Code y busque **Azure IoT Hub** en la esquina inferior izquierda.
1. Expanda el nodo **Devices** (Dispositivos).
1. Haga clic con el botón derecho en `avasample-iot-edge-device` y seleccione **Iniciar la supervisión del punto de conexión de eventos integrado**.

    > [!NOTE]
    > Es posible que se le pida que proporcione información del punto de conexión integrado del centro de IoT. Para obtener esa información, en Azure Portal, vaya a su centro de IoT y busque la opción **Puntos de conexión integrados** en el panel de navegación izquierdo. Haga clic ahí y busque el **punto de conexión compatible con el centro de eventos** en la sección **Punto de conexión compatible con el centro de eventos**. Copie y use el texto del cuadro. El punto de conexión será similar a este:  
        ```
        Endpoint=sb://iothub-ns-xxx.servicebus.windows.net/;SharedAccessKeyName=iothubowner;SharedAccessKey=XXX;EntityPath=<IoT Hub name>
        ```

## <a name="use-direct-method-calls"></a>Uso de llamadas de método directo

Ahora puede analizar secuencias de vídeo en directo mediante la invocación de métodos directos expuestos por el módulo perimetral Video Analyzer. Lea [Métodos directos de Video Analyzer](direct-methods.md) para examinar todos los métodos directos proporcionados por el módulo. 

### <a name="enumerate-pipeline-topologies"></a>Enumeración de topologías de canalización

En este paso se enumeran todas las [topologías de canalización](pipeline.md) del módulo.

1. Haga clic con el botón derecho en el módulo "avaedge" y seleccione **Invoke Module Direct Method** (Invocar método directo del módulo) en el menú contextual.
1. Verá un cuadro de edición emergente en la parte superior central de la ventana de Visual Studio Code. Escriba "pipelineTopologyList" en el cuadro de edición y presione Entrar.
1. A continuación, copie y pegue la carga de JSON siguiente en el cuadro de edición y presione ENTRAR.
   
```json
{
    "@apiVersion" : "1.0"
}
```

En unos segundos, verá la siguiente respuesta en la ventana de salida:
    
```
[DirectMethod] Invoking Direct Method [pipelineTopologyList] to [avasample-iot-edge-device/avaedge] ...
[DirectMethod] Response from [avasample-iot-edge-device/avaedge]:
{
  "status": 200,
  "payload": {
    "value": []
  }
}
```

Se espera la respuesta anterior ya que no se ha creado ninguna topología de canalización.

### <a name="set-a-pipeline-topology"></a>Establecimiento de una topología de canalización

Con los mismos pasos de antes, puede invocar `pipelineTopologySet` para establecer una topología de canalización mediante el siguiente JSON como carga. Se ocupará de la creación de una topología de canalización llamada "MotionDetection".


```json
{
    "@apiVersion": "1.0",
    "name": "MotionDetection",
    "properties": {
        "description": "Analyzing live video to detect motion and emit events",
        "parameters": [
            {
                "name": "rtspUrl",
                "type": "string",
                "description&quot;: &quot;rtspUrl"
            },
            {
                "name": "rtspUserName",
                "type": "string",
                "description": "rtspUserName",
                "default&quot;: &quot;dummyUserName"
            },
            {
                "name": "rtspPassword",
                "type": "string",
                "description": "rtspPassword",
                "default&quot;: &quot;dummypw"
            }
        ],
        "sources": [
            {
                "@type": "#Microsoft.VideoAnalyzer.RtspSource",
                "name": "rtspSource",
                "transport": "tcp",
                "endpoint": {
                    "@type": "#Microsoft.VideoAnalyzer.UnsecuredEndpoint",
                    "credentials": {
                        "@type": "#Microsoft.VideoAnalyzer.UsernamePasswordCredentials",
                        "username": "${rtspUserName}",
                        "password&quot;: &quot;${rtspPassword}"
                    },
                    "url&quot;: &quot;${rtspUrl}"
                }
            }
        ],
        "processors": [
            {
                "@type": "#Microsoft.VideoAnalyzer.MotionDetectionProcessor",
                "sensitivity": "medium",
                "name": "motionDetection",
                "inputs": [
                    {
                        "nodeName": "rtspSource",
                        "outputSelectors": []
                    }
                ]
            }
        ],
        "sinks": [
            {
                "hubOutputName": "inferenceOutput",
                "@type": "#Microsoft.VideoAnalyzer.IotHubMessageSink",
                "name": "iotHubSink",
                "inputs": [
                    {
                        "nodeName&quot;: &quot;motionDetection"
                    }
                ]
            }
        ]
    }
}
```

Esta carga de JSON crea una topología que define tres parámetros, donde dos de ellos tienen valores predeterminados. La topología tiene un nodo de origen ([origen RTSP](pipeline.md#rtsp-source)), un nodo de procesador ([procesador de detección de movimiento](pipeline.md#motion-detection-processor) y un nodo receptor ([receptor de mensajes de IoT Hub](pipeline.md#iot-hub-message-sink)). La representación visual de la topología se muestra más arriba.

En unos segundos, se ve la siguiente respuesta en la ventana **SALIDA**.

```json
{
  "status": 201,
  "payload": {
    "systemData": {
      "createdAt": "2021-03-21T18:16:46.491Z",
      "lastModifiedAt": "2021-03-21T18:16:46.491Z"
    },
    "name": "MotionDetection",
    "properties": {
      "description": "Analyzing live video to detect motion and emit events",
      "parameters": [
        {
          "name": "rtspPassword",
          "type": "string",
          "description": "rtspPassword",
          "default": "dummypw"
        },
        {
          "name": "rtspUrl",
          "type": "string",
          "description": "rtspUrl"
        },
        {
          "name": "rtspUserName",
          "type": "string",
          "description": "rtspUserName",
          "default": "dummyUserName"
        }
      ],
      "sources": [
        {
          "@type": "#Microsoft.VideoAnalyzer.RtspSource",
          "name": "rtspSource",
          "transport": "tcp",
          "endpoint": {
            "@type": "#Microsoft.VideoAnalyzer.UnsecuredEndpoint",
            "url": "${rtspUrl}",
            "credentials": {
              "@type": "#Microsoft.VideoAnalyzer.UsernamePasswordCredentials",
              "username": "${rtspUserName}",
              "password": "${rtspPassword}"
            }
          }
        }
      ],
      "processors": [
        {
          "@type": "#Microsoft.VideoAnalyzer.MotionDetectionProcessor",
          "sensitivity": "medium",
          "eventAggregationWindow": "PT1S",
          "name": "motionDetection",
          "inputs": [
            {
              "nodeName": "rtspSource",
              "outputSelectors": []
            }
          ]
        }
      ],
      "sinks": [
        {
          "@type": "#Microsoft.VideoAnalyzer.IotHubMessageSink",
          "hubOutputName": "inferenceOutput",
          "name": "iotHubSink",
          "inputs": [
            {
              "nodeName": "motionDetection",
              "outputSelectors": []
            }
          ]
        }
      ]
    }
  }
}
```

El estado que se devuelve es 201, que indica que se ha creado una topología.

Pruebe los siguientes pasos:

1. Vuelva a invocar a `pipelineTopologySet`. El código de estado que se devuelve es 200. Este código indica que se ha actualizado correctamente una topología ya existente.
1. Vuelva a invocar a `pipelineTopologySet`, pero cambie la cadena de descripción. El código de estado devuelto es 200 y la descripción se ha actualizado al nuevo valor.
1. Invoque a `pipelineTopologyList` como se describió en la sección anterior. Ahora puede ver la topología "MotionDetection" en la carga devuelta.

### <a name="read-the-pipeline-topology"></a>Lectura de la topología de canalización

Invoque a `pipelineTopologyGet` mediante el uso de la siguiente carga.

```json
{
    "@apiVersion" : "1.0",
    "name" : "MotionDetection"
}
```

En unos segundos, se ve la siguiente respuesta en la ventana **SALIDA**:

```json
{
  "status": 200,
  "payload": {
    "systemData": {
      "createdAt": "2021-03-21T18:16:46.491Z",
      "lastModifiedAt": "2021-03-21T18:16:46.491Z"
    },
    "name": "MotionDetection",
    "properties": {
      "description": "Analyzing live video to detect motion and emit events",
      "parameters": [
        {
          "name": "rtspPassword",
          "type": "string",
          "description": "rtspPassword",
          "default": "dummypw"
        },
        {
          "name": "rtspUrl",
          "type": "string",
          "description": "rtspUrl"
        },
        {
          "name": "rtspUserName",
          "type": "string",
          "description": "rtspUserName",
          "default": "dummyUserName"
        }
      ],
      "sources": [
        {
          "@type": "#Microsoft.VideoAnalyzer.RtspSource",
          "name": "rtspSource",
          "transport": "tcp",
          "endpoint": {
            "@type": "#Microsoft.VideoAnalyzer.UnsecuredEndpoint",
            "url": "${rtspUrl}",
            "credentials": {
              "@type": "#Microsoft.VideoAnalyzer.UsernamePasswordCredentials",
              "username": "${rtspUserName}",
              "password": "${rtspPassword}"
            }
          }
        }
      ],
      "processors": [
        {
          "@type": "#Microsoft.VideoAnalyzer.MotionDetectionProcessor",
          "sensitivity": "medium",
          "eventAggregationWindow": "PT1S",
          "name": "motionDetection",
          "inputs": [
            {
              "nodeName": "rtspSource",
              "outputSelectors": []
            }
          ]
        }
      ],
      "sinks": [
        {
          "@type": "#Microsoft.VideoAnalyzer.IotHubMessageSink",
          "hubOutputName": "inferenceOutput",
          "name": "iotHubSink",
          "inputs": [
            {
              "nodeName": "motionDetection",
              "outputSelectors": []
            }
          ]
        }
      ]
    }
  }
}
```

En la carga de respuesta, observe estos detalles:

* El código de estado es 200, lo que indica que se ha realizado correctamente.
* La carga incluye las marcas de tiempo `createdAt` y `lastModifiedAt`.

### <a name="create-a-live-pipeline-using-the-topology"></a>Creación de una canalización en vivo mediante la topología

A continuación, cree una canalización en vivo que haga referencia a la topología de canalización anterior. Invoque el método directo `livePipelineSet` mediante la siguiente carga:

```json
{
    "@apiVersion" : "1.0",
    "name": "mdpipeline1",
    "properties": {
        "topologyName": "MotionDetection",
        "description": "Sample pipeline description",
        "parameters": [
            {
                "name": "rtspUrl",
                "value": "rtsp://rtspsim:554/media/camera-300s.mkv"
            },
            {
                "name": "rtspUserName",
                "value": "testuser"
            },
            {
                "name": "rtspPassword",
                "value": "testpassword"
            }
        ]
    }
}
```

Tenga en cuenta que esta carga:

* La carga anterior especifica la topología ("MotionDetection") que usará la canalización en vivo.
* La carga contiene el valor de parámetro para `rtspUrl`, el cual no tenía un valor predeterminado en la carga de topología. Este valor es un vínculo al vídeo de ejemplo siguiente:

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4LTY4]


En unos segundos, se ve la siguiente respuesta en la ventana **SALIDA**:

```json
{
  "status": 201,
  "payload": {
    "systemData": {
      "createdAt": "2021-03-21T18:27:41.639Z",
      "lastModifiedAt": "2021-03-21T18:27:41.639Z"
    },
    "name": "mdpipeline1",
    "properties": {
      "state": "Inactive",
      "description": "Sample pipeline description",
      "topologyName": "MotionDetection",
      "parameters": [
        {
          "name": "rtspPassword",
          "value": "testpassword"
        },
        {
          "name": "rtspUrl",
          "value": "rtsp://rtspsim:554/media/camera-300s.mkv"
        },
        {
          "name": "rtspUserName",
          "value": "testuser"
        }
      ]
    }
  }
}
```

En la carga de respuesta, observe que:

* El código de estado es 201, lo que indica que se ha creado una nueva canalización en vivo.
* El estado es "Inactivo", lo que indica que la canalización en vivo se creó pero no se activó. Para más información, consulte [Estados de canalización](pipeline.md#pipeline-states).

Intente los siguientes métodos directos como se indica a continuación:

* Vuelva a invocar a `livePipelineSet` con la misma carga y observe que el código de estado devuelto es ahora 200.
* Vuelva a invocar a `livePipelineSet`, pero con una descripción diferente y observe la descripción actualizada en la carga de respuesta, la cual indica que la canalización en vivo se actualizó correctamente.
* Invoque `livePipelineSet`, pero cambie el nombre a "mdpipeline2" y `rtspUrl` a "rtsp://rtspsim:554/media/lots_015.mkv". En la carga de respuesta, observe la canalización en vivo que se acaba de crear (es decir, el código de estado 201).
    > [!NOTE]
    > Como se explica en [Topologías de canalización](pipeline.md#pipeline-topologies), puede crear varias canalizaciones en vivo para analizar secuencias de vídeo en directo de muchas cámaras con la misma topología de canalización. Si crea canalizaciones en vivo adicionales, procure eliminarlas durante el paso de limpieza.

### <a name="activate-the-live-pipeline"></a>Activación de la canalización en vivo

A continuación, puede activar la canalización en vivo, que inicia el flujo de vídeo en directo (simulado) a través de la canalización. Invoque el método directo `livePipelineActivate` mediante la siguiente carga:

```json
{
    "@apiVersion" : "1.0",
    "name" : "mdpipeline1"
}
```

En pocos segundos, verá la siguiente respuesta en la ventana SALIDA.

```json
{
  "status": 200,
  "payload": null
}
```

El código de estado 200 indica que la canalización en vivo se ha activado correctamente.

### <a name="check-the-state-of-the-live-pipeline"></a>Comprobación del estado de la canalización en vivo

Ahora, invoque el método directo `livePipelineGet` mediante la carga siguiente:

```json
{
    "@apiVersion" : "1.0",
    "name" : "mdpipeline1"
}
```

En pocos segundos, verá la siguiente respuesta en la ventana SALIDA.

```json
{
  "status": 200,
  "payload": {
    "systemData": {
      "createdAt": "2021-03-21T18:27:41.639Z",
      "lastModifiedAt": "2021-03-21T18:27:41.639Z"
    },
    "name": "mdpipeline1",
    "properties": {
      "state": "Active",
      "description": "Sample pipeline description",
      "topologyName": "MotionDetection",
      "parameters": [
        {
          "name": "rtspPassword",
          "value": "testpassword"
        },
        {
          "name": "rtspUrl",
          "value": "rtsp://rtspsim:554/media/camera-300s.mkv"
        },
        {
          "name": "rtspUserName",
          "value": "testuser"
        }
      ]
    }
  }
}
```

En la carga de respuesta, observe los siguientes detalles:

* El código de estado es 200, lo que indica que se ha realizado correctamente.
* El estado es "Activo", lo que indica que la canalización en vivo se encuentra activa en este momento.

## <a name="observe-results"></a>Observación de resultados

La canalización en vivo que ha creado y activado anteriormente usa el nodo de procesador de detección de movimiento para detectar el movimiento en la secuencia de vídeo en directo entrante y envía eventos al receptor de IoT Hub. A continuación, estos eventos se retransmiten a IoT Hub como mensajes, como se puede observar ahora. Verá mensajes en la ventana de salida que tienen el siguiente "cuerpo":


```json
{
  "timestamp": 145471641211899,
  "inferences": [
    {
      "type": "motion",
      "motion": {
        "box": {
          "l": 0.514644,
          "t": 0.574627,
          "w": 0.3375,
          "h": 0.096296
        }
      }
    }
  ]
}
```

Observe este detalle:

*   En la sección Inferencias se indica que el tipo es movimiento. Proporciona datos adicionales sobre el evento de movimiento y proporciona un rectángulo de selección para la región del fotograma de vídeo (en la marca de tiempo especificada) donde se detectó el movimiento.

    
## <a name="invoke-additional-direct-method-calls-to-clean-up"></a>Invocación de llamadas de método directo adicionales para la limpieza

A continuación, puede invocar métodos directos para desactivar y eliminar la canalización en vivo (en ese orden).

### <a name="deactivate-the-live-pipeline"></a>Desactivación de la canalización en vivo

Invoque el método directo `livePipelineDeactivate` mediante la siguiente carga:

```json
{
    "@apiVersion" : "1.0",
    "name" : "mdpipeline1"
}
```

En pocos segundos, verá la siguiente respuesta en la ventana **SALIDA**:

```json
{
  "status": 200,
  "payload": null
}
```

El código de estado 200 indica que la canalización en vivo se ha desactivado correctamente. 

A continuación, intente invocar `livePipelineGet` como se ha indicado en este mismo artículo. Observe el valor de estado.

### <a name="delete-the-live-pipeline"></a>Eliminación de la canalización en vivo

Invoque el método directo `livePipelineDelete` mediante la siguiente carga

```json
{
    "@apiVersion" : "1.0",
    "name" : "mdpipeline1"
}
```

En pocos segundos, verá la siguiente respuesta en la ventana **SALIDA**:

```json
{
  "status": 200,
  "payload": null
}
```
El código de estado 200 indica que la canalización en vivo se ha eliminado correctamente.

Si también creó la canalización denominada "mdpipeline2", no podrá eliminar la topología de canalización sin eliminar también esta canalización adicional. Vuelva a invocar el método directo `livePipelineDelete` mediante la siguiente carga:

```
{
    "@apiVersion" : "1.0",
    "name" : "mdpipeline2"
}
```

En pocos segundos, verá la siguiente respuesta en la ventana de salida:

```json
{
  "status": 200,
  "payload": null
}
```

El código de estado 200 indica que la canalización en vivo se ha eliminado correctamente.

Puede invocar a `livePipelineList` con la misma carga que `pipelineTopologyList`. Observe que no se enumera ninguna canalización en vivo.

### <a name="delete-the-pipeline-topology"></a>Eliminación de la topología de canalización

Una vez eliminadas todas las canalizaciones en vivo, puede invocar el método directo `pipelineTopologyDelete` con la siguiente carga:

```json
{
    "@apiVersion" : "1.0",
    "name" : "MotionDetection"
}
```

En pocos segundos, verá la siguiente respuesta en la ventana **SALIDA**.

```json
{
  "status": 200,
  "payload": null
}
```

El código de estado 200 indica que la topología se ha eliminado correctamente.

Puede intentar invocar a `pipelineTopologyList` y observar que el módulo no contiene topologías.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [prerequisites](./includes/common-includes/clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

* Pruebe el [inicio rápido para grabar vídeos en la nube al detectarse movimiento](detect-motion-record-video-clips-cloud.md)
* Pruebe el [inicio rápido para analizar vídeo en directo](analyze-live-video-use-your-model-http.md)
* Más información sobre los [mensajes de diagnóstico](monitor-log-edge.md) 
