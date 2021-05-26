---
title: Detección de movimiento y grabación de vídeo con Azure Video Analyzer
description: En esta guía de inicio rápido se muestra cómo usar el módulo perimetral de Azure Video Analyzer para detectar movimiento en una secuencia de vídeo en directo y grabar vídeo en la cuenta de Video Analyzer.
ms.topic: quickstart
ms.date: 04/03/2021
ms.openlocfilehash: 851310ff3b03730a1fc9adb4eaa7c0c3f9cf84f2
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388527"
---
# <a name="quickstart-detect-motion-record-video-to-video-analyzer"></a>Inicio rápido: Detección de movimiento y grabación de vídeo en Video Analyzer

Este artículo le guía por los pasos necesarios para usar el módulo perimetral de Azure Video Analyzer para la [grabación basada en eventos](event-based-video-recording-concept.md). Usa una máquina virtual Linux en Azure como dispositivo IoT Edge y una secuencia de vídeo en directo simulada. Esta secuencia de vídeo se analiza para detectar la presencia de objetos en movimiento. Cuando se detecta movimiento, los eventos se envían a Azure IoT Hub y la parte correspondiente de la secuencia de vídeo se graba como un [recurso de vídeo](terminology.md#video) en la cuenta de Video Analyzer.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure que incluya una suscripción activa. [Cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) gratuita, en caso de que aún no lo haya hecho.

    > [!NOTE]    
    > Necesitará una suscripción a Azure en la que tenga acceso a los roles [Colaborador](../../role-based-access-control/built-in-roles.md#contributor) y [Administrador de acceso de usuario](../../role-based-access-control/built-in-roles.md#user-access-administrator). Si no tiene los permisos adecuados, póngase en contacto con el administrador de la cuenta para que se los conceda.
* [Visual Studio Code](https://code.visualstudio.com/) con las siguientes extensiones:
    * [Herramientas de Azure IoT](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)

### <a name="set-up-azure-resources"></a>Configuración de los recursos de Azure

[![Implementación en Azure](https://aka.ms/deploytoazurebutton)](https://aka.ms/ava-click-to-deploy)

El proceso de implementación tardará unos **20 minutos**. Al finalizar, tendrá determinados recursos de Azure implementados en la suscripción de Azure, entre los que se incluyen:
1. **Cuenta de Video Analyzer**: este [servicio en la nube](overview.md) se usa para registrar el módulo perimetral Video Analyzer y para reproducir vídeo grabado y análisis de vídeo.
1. **Cuenta de almacenamiento**: para almacenar vídeo grabado y análisis de vídeo.
1. **Identidad administrada**: esta es la [identidad administrada] asignada por el usuario../../active-directory/managed-identities-azure-resources/overview.md) que se usa para administrar el acceso a la cuenta de almacenamiento anterior.
1. **Máquina virtual**: esta es una máquina virtual que actuará como su dispositivo perimetral simulado.
1. **IoT Hub**: funciona como un centro de mensajes común para la comunicación bidireccional entre la aplicación de IoT, los módulos de IoT Edge y los dispositivos que administra.

Puede obtener más detalles [aquí](https://github.com/Azure/video-analyzer/tree/main/setup).

## <a name="review-the-sample-video"></a>Revisión del vídeo de ejemplo

En la máquina virtual creada por la implementación anterior hay varios archivos MKV.  Uno de estos archivos se denomina `lots_015.mkv`. En los pasos siguientes, usaremos este archivo de vídeo para simular una secuencia en vivo para este tutorial.

Puede usar una aplicación como [VLC Player](https://www.videolan.org/vlc/), iníciela, presione `Ctrl+N` y pegue el vínculo al [ejemplo de vídeo sobre el aparcamiento](https://lvamedia.blob.core.windows.net/public/lots_015.mkv) para iniciar la reproducción. Hacia la marca de 5 segundos, un coche blanco se desplaza por el aparcamiento.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4LUbN]

Cuando realice los pasos que se indican a continuación, habrá utilizado el módulo perimetral de Video Analyzer para detectar ese movimiento del coche y grabar un clip de vídeo que empiece en torno a esa marca de los 5 segundos. 

El diagrama siguiente es la representación visual de todo el flujo.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/detect-motion-record-video-clips-cloud/topology.png" alt-text="Grabación de vídeo basada en eventos en un recurso de vídeo que se basa en eventos de movimiento":::

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

## <a name="use-direct-method-calls-to-analyze-live-video"></a>Uso de llamadas a métodos directos para analizar vídeo en directo

Ahora puede analizar secuencias de vídeo en directo mediante la invocación de métodos directos expuestos por el módulo perimetral de Video Analyzer. Lea [Métodos directos de Video Analyzer](direct-methods.md) para examinar todos los métodos directos proporcionados por el módulo. 

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

Con los mismos pasos de antes, puede invocar `pipelineTopologySet` para establecer una topología de canalización con el siguiente JSON como carga. Se creará una topología de canalización llamada "EVRtoVideoSinkOnMotionDetection".

> [!NOTE]
> En la carga siguiente, la propiedad `videoName` se establecerá en "sample-motion-video-camera001", que será el nombre del recurso de vídeo que se creará en la cuenta de Video Analyzer. Este nombre de recurso debe ser único para cada origen de vídeo en directo que grabe. Debe editar la propiedad `videoName` a continuación según sea necesario para garantizar que no se repite.

```
{
  "@apiVersion": "1.0",
  "name": "EVRtoVideoSinkOnMotionDetection",
  "properties": {
    "description": "Event-based video recording to Video Sink based on motion events",
    "parameters": [
      {
        "name": "rtspUserName",
        "type": "String",
        "description": "rtsp source user name.",
        "default&quot;: &quot;dummyUserName"
      },
      {
        "name": "rtspPassword",
        "type": "String",
        "description": "rtsp source password.",
        "default&quot;: &quot;dummyPassword"
      },
      {
        "name": "rtspUrl",
        "type": "String",
        "description&quot;: &quot;rtsp Url"
      },
      {
        "name": "motionSensitivity",
        "type": "String",
        "description": "motion detection sensitivity",
        "default&quot;: &quot;medium"
      },
      {
        "name": "hubSinkOutputName",
        "type": "String",
        "description": "hub sink output name",
        "default&quot;: &quot;inferenceOutput"
      }
    ],
    "sources": [
      {
        "@type": "#Microsoft.VideoAnalyzer.RtspSource",
        "name": "rtspSource",
        "endpoint": {
          "@type": "#Microsoft.VideoAnalyzer.UnsecuredEndpoint",
          "url": "${rtspUrl}",
          "credentials": {
            "@type": "#Microsoft.VideoAnalyzer.UsernamePasswordCredentials",
            "username": "${rtspUserName}",
            "password&quot;: &quot;${rtspPassword}"
          }
        }
      }
    ],
    "processors": [
      {
        "@type": "#Microsoft.VideoAnalyzer.MotionDetectionProcessor",
        "name": "motionDetection",
        "sensitivity": "${motionSensitivity}",
        "inputs": [
          {
            "nodeName": "rtspSource",
            "outputSelectors": [
              {
                "property": "mediaType",
                "operator": "is",
                "value&quot;: &quot;video"
              }
            ]
          }
        ]
      },
      {
        "@type": "#Microsoft.VideoAnalyzer.SignalGateProcessor",
        "name": "signalGateProcessor",
        "inputs": [
          {
            "nodeName&quot;: &quot;motionDetection"
          },
          {
            "nodeName": "rtspSource",
            "outputSelectors": [
              {
                "property": "mediaType",
                "operator": "is",
                "value&quot;: &quot;video"
              }
            ]
          }
        ],
        "activationEvaluationWindow": "PT1S",
        "activationSignalOffset": "PT0S",
        "minimumActivationTime": "PT30S",
        "maximumActivationTime&quot;: &quot;PT30S"
      }
    ],
    "sinks": [
      {
        "@type": "#Microsoft.VideoAnalyzer.VideoSink",
        "name": "videoSink",
        "videoName": "sample-motion-video-camera001",
        "inputs": [
          {
            "nodeName": "signalGateProcessor",
            "outputSelectors": [
              {
                "property": "mediaType",
                "operator": "is",
                "value&quot;: &quot;video"
              }
            ]
          }
        ],
        "videoCreationProperties": {
          "title": "sample-motion-video-camera001",
          "description": "Motion-detection based recording of clips to a video resource",
          "segmentLength&quot;: &quot;PT30S"
        },
        "localMediaCachePath": "/var/lib/videoanalyzer/tmp/",
        "localMediaCacheMaximumSizeMiB&quot;: &quot;2048"
      },
      {
        "@type": "#Microsoft.VideoAnalyzer.IoTHubMessageSink",
        "name": "hubSink",
        "hubOutputName": "${hubSinkOutputName}",
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

La carga de JSON anterior da como resultado la creación de una topología de canalización que define cinco parámetros (cuatro de los cuales tienen valores predeterminados). La topología tiene un nodo de origen ([origen RTSP](pipeline.md#rtsp-source)), dos nodos de procesador ([procesador de detección de movimiento](pipeline.md#motion-detection-processor) y [procesador de la puerta de señal](pipeline.md#signal-gate-processor)) y dos nodos receptores (receptor de IoT Hub y [receptor de vídeo](pipeline.md#video-sink)). La representación visual de la topología se muestra más arriba.

En unos segundos, verá la siguiente respuesta en la ventana **SALIDA**.

```
[DirectMethod] Invoking Direct Method [pipelineTopologySet] to [avasample-iot-edge-device/avaedge] ...
[DirectMethod] Response from [avasample-iot-edge-device/avaedge]:
{
  "status": 201,
  "payload": {
    "systemData": {
      "createdAt": "2021-05-03T15:17:53.483Z",
      "lastModifiedAt&quot;: &quot;2021-05-03T15:17:53.483Z"
    },
    "name": "EVRtoVideoSinkOnMotionDetection",
    "properties": {
      "description": "Event-based video recording to Video Sink based on motion events",
      "parameters": [
        {
          "name": "hubSinkOutputName",
          "type": "string",
          "description": "hub sink output name",
          "default&quot;: &quot;inferenceOutput"
        },
        {
          "name": "motionSensitivity",
          "type": "string",
          "description": "motion detection sensitivity",
          "default&quot;: &quot;medium"
        },
        {
          "name": "rtspPassword",
          "type": "string",
          "description": "rtsp source password.",
          "default&quot;: &quot;dummyPassword"
        },
        {
          "name": "rtspUrl",
          "type": "string",
          "description&quot;: &quot;rtsp Url"
        },
        {
          "name": "rtspUserName",
          "type": "string",
          "description": "rtsp source user name.",
          "default&quot;: &quot;dummyUserName"
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
              "password&quot;: &quot;${rtspPassword}"
            }
          }
        }
      ],
      "processors": [
        {
          "@type": "#Microsoft.VideoAnalyzer.MotionDetectionProcessor",
          "sensitivity": "${motionSensitivity}",
          "eventAggregationWindow": "PT1S",
          "name": "motionDetection",
          "inputs": [
            {
              "nodeName": "rtspSource",
              "outputSelectors": [
                {
                  "property": "mediaType",
                  "operator": "is",
                  "value&quot;: &quot;video"
                }
              ]
            }
          ]
        },
        {
          "@type": "#Microsoft.VideoAnalyzer.SignalGateProcessor",
          "activationEvaluationWindow": "PT1S",
          "activationSignalOffset": "PT0S",
          "minimumActivationTime": "PT30S",
          "maximumActivationTime": "PT30S",
          "name": "signalGateProcessor",
          "inputs": [
            {
              "nodeName": "motionDetection",
              "outputSelectors": []
            },
            {
              "nodeName": "rtspSource",
              "outputSelectors": [
                {
                  "property": "mediaType",
                  "operator": "is",
                  "value&quot;: &quot;video"
                }
              ]
            }
          ]
        }
      ],
      "sinks": [
        {
          "@type": "#Microsoft.VideoAnalyzer.VideoSink",
          "localMediaCachePath": "/var/lib/videoanalyzer/tmp/",
          "localMediaCacheMaximumSizeMiB": "2048",
          "videoName": "sample-motion-video-camera001",
          "videoCreationProperties": {
            "title": "sample-motion-video-camera001",
            "description": "Motion-detection based recording of clips to a video resource",
            "segmentLength&quot;: &quot;PT30S"
          },
          "name": "videoSink",
          "inputs": [
            {
              "nodeName": "signalGateProcessor",
              "outputSelectors": [
                {
                  "property": "mediaType",
                  "operator": "is",
                  "value&quot;: &quot;video"
                }
              ]
            }
          ]
        },
        {
          "@type": "#Microsoft.VideoAnalyzer.IotHubMessageSink",
          "hubOutputName": "${hubSinkOutputName}",
          "name": "hubSink",
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



El estado devuelto es 201, lo que indica que se ha creado una nueva topología de canalización. Intente los siguientes métodos directos como se indica a continuación:

* Vuelva a invocar a `pipelineTopologySet` y compruebe que el código de estado devuelto es 200. Un código de estado 200 indica que se ha actualizado correctamente una topología de canalización existente.
* Vuelva a invocar a `pipelineTopologySet`, pero cambie la cadena de descripción. Compruebe que el código de estado de la respuesta es 200 y que la descripción se ha actualizado al nuevo valor.
* Invoque `pipelineTopologyList` como se describe en el paso anterior y compruebe que ahora puede ver la topología "EVRtoVideoSinkOnMotionDetection" mencionada en la carga devuelta.

### <a name="read-the-pipeline-topology"></a>Lectura de la topología de canalización

Ahora, invoque a `pipelineTopologyGet` con la siguiente carga.
```

{
    "@apiVersion" : "1.0",
    "name&quot; : &quot;EVRtoVideoSinkOnMotionDetection"
}
```

En unos segundos, verá la siguiente respuesta en la ventana de salida.

```
[DirectMethod] Invoking Direct Method [pipelineTopologyGet] to [avasample-iot-edge-device/avaedge] ...
[DirectMethod] Response from [avasample-iot-edge-device/avaedge]:
{
  "status": 200,
  "payload": {
    "systemData": {
      "createdAt": "2021-05-03T15:17:53.483Z",
      "lastModifiedAt&quot;: &quot;2021-05-03T15:17:53.483Z"
    },
    "name": "EVRtoVideoSinkOnMotionDetection",
    "properties": {
      "description": "Event-based video recording to Video Sink based on motion events",
      "parameters": [
        {
          "name": "hubSinkOutputName",
          "type": "string",
          "description": "hub sink output name",
          "default&quot;: &quot;inferenceOutput"
        },
        {
          "name": "motionSensitivity",
          "type": "string",
          "description": "motion detection sensitivity",
          "default&quot;: &quot;medium"
        },
        {
          "name": "rtspPassword",
          "type": "string",
          "description": "rtsp source password.",
          "default&quot;: &quot;dummyPassword"
        },
        {
          "name": "rtspUrl",
          "type": "string",
          "description&quot;: &quot;rtsp Url"
        },
        {
          "name": "rtspUserName",
          "type": "string",
          "description": "rtsp source user name.",
          "default&quot;: &quot;dummyUserName"
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
              "password&quot;: &quot;${rtspPassword}"
            }
          }
        }
      ],
      "processors": [
        {
          "@type": "#Microsoft.VideoAnalyzer.MotionDetectionProcessor",
          "sensitivity": "${motionSensitivity}",
          "eventAggregationWindow": "PT1S",
          "name": "motionDetection",
          "inputs": [
            {
              "nodeName": "rtspSource",
              "outputSelectors": [
                {
                  "property": "mediaType",
                  "operator": "is",
                  "value&quot;: &quot;video"
                }
              ]
            }
          ]
        },
        {
          "@type": "#Microsoft.VideoAnalyzer.SignalGateProcessor",
          "activationEvaluationWindow": "PT1S",
          "activationSignalOffset": "PT0S",
          "minimumActivationTime": "PT30S",
          "maximumActivationTime": "PT30S",
          "name": "signalGateProcessor",
          "inputs": [
            {
              "nodeName": "motionDetection",
              "outputSelectors": []
            },
            {
              "nodeName": "rtspSource",
              "outputSelectors": [
                {
                  "property": "mediaType",
                  "operator": "is",
                  "value&quot;: &quot;video"
                }
              ]
            }
          ]
        }
      ],
      "sinks": [
        {
          "@type": "#Microsoft.VideoAnalyzer.IotHubMessageSink",
          "hubOutputName": "${hubSinkOutputName}",
          "name": "hubSink",
          "inputs": [
            {
              "nodeName": "motionDetection",
              "outputSelectors": []
            }
          ]
        },
        {
          "@type": "#Microsoft.VideoAnalyzer.VideoSink",
          "localMediaCachePath": "/var/lib/videoanalyzer/tmp/",
          "localMediaCacheMaximumSizeMiB": "2048",
          "videoName": "sample-motion-video-camera001",
          "videoCreationProperties": {
            "title": "sample-motion-video-camera001",
            "description": "Motion-detection based recording of clips to a video resource",
            "segmentLength&quot;: &quot;PT30S"
          },
          "name": "videoSink",
          "inputs": [
            {
              "nodeName": "signalGateProcessor",
              "outputSelectors": [
                {
                  "property": "mediaType",
                  "operator": "is",
                  "value&quot;: &quot;video"
                }
              ]
            }
          ]
        }
      ]
    }
  }
}
```

Observe las siguientes propiedades en la carga de respuesta:

* El código de estado es 200, lo que indica que se ha realizado correctamente.
* La carga incluye las marcas de tiempo `createdAt` y `lastModifiedAt`.

### <a name="create-a-live-pipeline-using-the-topology"></a>Creación de una canalización en vivo mediante la topología

A continuación, cree una canalización en vivo que haga referencia a la topología de canalización anterior. Invoque el método directo `livePipelineSet` con la siguiente carga:

```
{
    "@apiVersion" : "1.0",
    "name" : "Sample-Pipeline-1",
    "properties" : {
        "topologyName" : "EVRtoVideoSinkOnMotionDetection",
        "description" : "Sample pipeline description",
        "parameters" : [
            { "name" : "rtspUrl", "value" : "rtsp://rtspsim:554/media/lots_015.mkv" }
        ]
    }
}
```

Tenga en cuenta lo siguiente:

* La carga anterior especifica la topología ("EVRtoVideoSinkOnMotionDetection") que usará la canalización en vivo.
* La carga contiene el valor de parámetro para `rtspUrl`, el cual no tenía un valor predeterminado en la carga de topología.

En unos segundos, verá la siguiente respuesta en la ventana de salida:

```
[DirectMethod] Invoking Direct Method [livePipelineSet] to [avasample-iot-edge-device/avaedge] ...
[DirectMethod] Response from [avasample-iot-edge-device/avaedge]:
{
  "status": 201,
  "payload": {
    "systemData": {
      "createdAt": "2021-05-03T15:20:29.023Z",
      "lastModifiedAt&quot;: &quot;2021-05-03T15:20:29.023Z"
    },
    "name": "Sample-Pipeline-1",
    "properties": {
      "state": "Inactive",
      "description": "Sample pipeline description",
      "topologyName": "EVRtoVideoSinkOnMotionDetection",
      "parameters": [
        {
          "name": "rtspUrl",
          "value&quot;: &quot;rtsp://rtspsim:554/media/lots_015.mkv"
        }
      ]
    }
  }
}
```

Observe las siguientes propiedades en la carga de respuesta:

* El código de estado es 201, lo que indica que se ha creado una nueva canalización en vivo.
* El estado es "Inactivo", lo que indica que la canalización en vivo se creó pero no se activó. Para más información, consulte [Estados de canalización](pipeline.md#pipeline-states).

Intente los siguientes métodos directos como se indica a continuación:

* Vuelva a invocar a `livePipelineSet` con la misma carga y observe que el código de estado devuelto es ahora 200.
* Vuelva a invocar a `livePipelineSet`, pero con una descripción diferente y observe la descripción actualizada en la carga de respuesta, lo que indica que la canalización en vivo se actualizó correctamente.

    > [!NOTE]
    > Como se explica en [Topologías de canalización](pipeline.md#pipeline-topologies), puede crear varias canalizaciones en vivo para analizar secuencias de vídeo en directo de muchas cámaras con la misma topología de canalización. Sin embargo, esta topología concreta incluye en el código el valor de `videoName`. Ya que solo se debe grabar un origen de vídeo en directo en un recurso de vídeo de Video Analyzer, no debe crear canalizaciones en vivo adicionales con esta topología concreta.

### <a name="activate-the-live-pipeline"></a>Activación de la canalización en vivo

A continuación, puede activar la canalización en vivo, que inicia el flujo de vídeo en directo (simulado) a través de la canalización. Invoque el método directo `livePipelineActivate` con la siguiente carga:

```
{
    "@apiVersion" : "1.0",
    "name" : "Sample-Pipeline-1"
}
```

En unos segundos, verá la siguiente respuesta en la ventana de salida.

```
[DirectMethod] Invoking Direct Method [livePipelineActivate] to [avasample-iot-edge-device/avaedge] ...
[DirectMethod] Response from [avasample-iot-edge-device/avaedge]:
{
  "status": 200,
  "payload": null
}
```

El código de estado 200 en la carga de respuesta indica que la canalización en vivo se activó correctamente.

### <a name="check-the-state-of-the-live-pipeline"></a>Comprobación del estado de la canalización en vivo

Ahora, invoque el método directo `livePipelineGet` con la carga siguiente:

```
{
    "@apiVersion" : "1.0",
    "name&quot; : &quot;Sample-Pipeline-1"
}
```

En unos segundos, verá la siguiente respuesta en la ventana de salida.

```
[DirectMethod] Invoking Direct Method [livePipelineGet] to [avasample-iot-edge-device/avaedge] ...
[DirectMethod] Response from [avasample-iot-edge-device/avaedge]:
{
  "status": 200,
  "payload": {
    "systemData": {
      "createdAt": "2021-05-03T14:21:21.750Z",
      "lastModifiedAt&quot;: &quot;2021-05-03T14:21:21.750Z"
    },
    "name": "Sample-Pipeline-1",
    "properties": {
      "state": "Active",
      "description": "Sample pipeline description",
      "topologyName": "EVRtoVideoSinkOnMotionDetection",
      "parameters": [
        {
          "name": "rtspUrl",
          "value&quot;: &quot;rtsp://rtspsim:554/media/lots_015.mkv"
        }
      ]
    }
  }
}
```

Observe las siguientes propiedades en la carga de respuesta:

* El código de estado es 200, lo que indica que se ha realizado correctamente.
* El estado es "Activo", lo que indica que la canalización en vivo se encuentra en el estado "Activo" en este momento.

## <a name="observe-results"></a>Observación de resultados

La canalización en vivo que ha creado y activado anteriormente usa el nodo de procesador de detección de movimiento para detectar el movimiento en la secuencia de vídeo en directo entrante y envía eventos al receptor de IoT Hub. A continuación, estos eventos se retransmiten a IoT Hub como mensajes, como se puede observar ahora. Verá los siguientes mensajes en la ventana de salida

```
[IoTHubMonitor] [1:22:53 PM] Message received from [avasample-iot-edge-device/avaedge]:
{
  "body": {
    "sdp&quot;: &quot;SDP:\nv=0\r\no=- 1620066173760872 1 IN IP4 172.18.0.4\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/lots_015.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.08.19\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-73.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/lots_015.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=640028;sprop-parameter-sets=Z2QAKKzZQHgCJoQAAAMABAAAAwDwPGDGWA==,aOvhEsiw\r\na=control:track1\r\n"
  },
  "properties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/videoAnalyzers/{ava-account-name}",
    "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/sources/rtspSource",
    "eventType": "Microsoft.VideoAnalyzer.Diagnostics.MediaSessionEstablished",
    "eventTime": "2021-05-03T18:22:53.761Z",
    "dataVersion": "1.0"
  },
  "systemProperties": {
    "iothub-connection-device-id": "avasample-iot-edge-device",
    "iothub-connection-module-id": "avaedge",
    "iothub-connection-auth-method": "{\"scope\":\"module\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
    "iothub-connection-auth-generation-id": "637556611958535962",
    "iothub-enqueuedtime": 1620066173816,
    "iothub-message-source": "Telemetry",
    "messageId": "c2de6a40-1e0a-45ef-9449-599fc5680d05",
    "contentType": "application/json",
    "contentEncoding": "utf-8"
  }
}
[IoTHubMonitor] [1:22:59 PM] Message received from [avasample-iot-edge-device/avaedge]:
{
  "body": {
    "timestamp": 145805956115743,
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
  "properties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/videoAnalyzers/{ava-account-name}",
    "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/processors/motionDetection",
    "eventType": "Microsoft.VideoAnalyzer.Analytics.Inference",
    "eventTime": "2021-05-03T18:22:59.063Z",
    "dataVersion&quot;: &quot;1.0"
  },
  "systemProperties": {
    "iothub-connection-device-id": "avasample-iot-edge-device",
    "iothub-connection-module-id": "avaedge",
    "iothub-connection-auth-method": "{\"scope\":\"module\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
    "iothub-connection-auth-generation-id": "637556611958535962",
    "iothub-enqueuedtime": 1620066179359,
    "iothub-message-source": "Telemetry",
    "messageId": "9ccfab80-2993-42c7-9452-92e21df96413",
    "contentType": "application/json",
    "contentEncoding&quot;: &quot;utf-8"
  }
}

```

Observe las siguientes propiedades de los mensajes anteriores:

* Cada mensaje contiene una sección `body` y una sección `properties`. Para comprender lo que representan estas secciones, consulte [Creación y lectura de mensajes de IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).
* El primer mensaje es **MediaSessionEstablished**, que indica que el nodo de origen RTSP (sujeto) ha podido establecer la conexión con el simulador RTSP y ha comenzado a recibir una fuente en directo (simulada).
* `subject` hace referencia al nodo de la canalización en vivo desde el que se generó el mensaje. En este caso, el mensaje se origina en el nodo de origen RTSP.
* `eventType` indica que este es un evento de diagnóstico.
* `eventTime` indica la hora a la que se produjo el evento.
* `body` contiene datos sobre el evento: es el mensaje [SDP](https://en.wikipedia.org/wiki/Session_Description_Protocol).
* El segundo mensaje es un evento de **inferencia**. Puede comprobar que se envió aproximadamente cinco segundos después del mensaje **MediaSessionEstablished**, lo cual se corresponde con el retraso entre el inicio del vídeo y el momento en que el automóvil circula por el aparcamiento.
* `subject` hace referencia al nodo del procesador de detección de movimiento de la canalización que generó este mensaje.
* Este es un evento de análisis y, por lo tanto, la sección `body` contiene datos de `timestamp` y `inferences`.
* La sección `inferences` indica que el `type` es "motion" y contiene datos adicionales sobre el evento "motion".

Observe que la sección `messageId` de body es "9ccfab80-2993-42c7-9452-92e21df96413". Se muestra en el siguiente evento operativo:

```
[IoTHubMonitor] [1:23:31 PM] Message received from [avasample-iot-edge-device/avaedge]:
{
  "body": {
    "outputType": "video",
    "outputLocation&quot;: &quot;sample-motion-video-camera001"
  },
  "properties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/videoAnalyzers/{ava-account-name}",
    "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/sinks/videoSink",
    "eventType": "Microsoft.VideoAnalyzer.Operational.RecordingStarted",
    "eventTime": "2021-05-03T18:23:31.319Z",
    "dataVersion&quot;: &quot;1.0"
  },
  "systemProperties": {
    "iothub-connection-device-id": "avasample-iot-edge-device",
    "iothub-connection-module-id": "avaedge",
    "iothub-connection-auth-method": "{\"scope\":\"module\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
    "iothub-connection-auth-generation-id": "637556611958535962",
    "iothub-enqueuedtime": 1620066211373,
    "iothub-message-source": "Telemetry",
    "messageId": "c7cbb363-7cc7-4169-936f-55de5fae111c",
    "correlationId": "9ccfab80-2993-42c7-9452-92e21df96413",
    "contentType": "application/json",
    "contentEncoding&quot;: &quot;utf-8"
  }
}
```
Este mensaje **RecordingStarted** indica que se inició la grabación de vídeo. Observe que el valor de `correlationId` de "9ccfab80-2993-42c7-9452-92e21df96413" coincide con el `messageId` del mensaje de **inferencia**, que permite realizar un seguimiento del evento que desencadenó la grabación. El siguiente es el próximo evento operativo:

```
[IoTHubMonitor] [1:24:00 PM] Message received from [avasample-iot-edge-device/avaedge]:
{
  "body": {
    "outputType": "video",
    "outputLocation&quot;: &quot;sample-motion"
  },
  "properties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/videoAnalyzers/{ava-account-name}",
    "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/sinks/videoSink",
    "eventType": "Microsoft.VideoAnalyzer.Operational.RecordingAvailable",
    "eventTime": "2021-05-03T18:24:00.686Z",
    "dataVersion&quot;: &quot;1.0"
  },
  "systemProperties": {
    "iothub-connection-device-id": "avasample-iot-edge-device",
    "iothub-connection-module-id": "avaedge",
    "iothub-connection-auth-method": "{\"scope\":\"module\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
    "iothub-connection-auth-generation-id": "637556611958535962",
    "iothub-enqueuedtime": 1620066240741,
    "iothub-message-source": "Telemetry",
    "messageId": "5b26aa88-e037-4834-af34-a6a4df3c42c2",
    "correlationId": "9ccfab80-2993-42c7-9452-92e21df96413",
    "contentType": "application/json",
    "contentEncoding&quot;: &quot;utf-8"
  }
}
[IoTHubMonitor] [1:24:00 PM] Message received from [avasample-iot-edge-device/avaedge]:
{
```

Este evento **RecordingAvailable** indica que los datos multimedia se han grabado ahora en el recurso de vídeo. Observe que es el valor de `correlationId` es el mismo: "9ccfab80-2993-42c7-9452-92e21df96413". El último evento operativo para esta cadena de mensajes (con el mismo valor de `correlationId`) es el siguiente:

```
[IoTHubMonitor] [1:24:00 PM] Message received from [avasample-iot-edge-device/avaedge]:
{
  "body": {
    "outputType": "video",
    "outputLocation&quot;: &quot;sample-motion"
  },
  "properties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/videoAnalyzers/{ava-account-name}",
    "subject": "/edgeModules/avaedge/livePipelines/Sample-Pipeline-1/sinks/videoSink",
    "eventType": "Microsoft.VideoAnalyzer.Operational.RecordingStopped",
    "eventTime": "2021-05-03T18:24:00.710Z",
    "dataVersion&quot;: &quot;1.0"
  },
  "systemProperties": {
    "iothub-connection-device-id": "avasample-iot-edge-device",
    "iothub-connection-module-id": "avaedge",
    "iothub-connection-auth-method": "{\"scope\":\"module\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
    "iothub-connection-auth-generation-id": "637556611958535962",
    "iothub-enqueuedtime": 1620066240766,
    "iothub-message-source": "Telemetry",
    "messageId": "f3dbd5d5-3176-4d5b-80d8-c67de85bc619",
    "correlationId": "9ccfab80-2993-42c7-9452-92e21df96413",
    "contentType": "application/json",
    "contentEncoding&quot;: &quot;utf-8"
  }
}
```
Este evento **RecordingStopped** indica que la puerta de señales se ha cerrado y se ha grabado la parte pertinente del vídeo en directo entrante. Observe que es el valor de `correlationId` es el mismo: "9ccfab80-2993-42c7-9452-92e21df96413".

En la topología, el nodo del procesador de la puerta de señales se ha configurado con tiempos de activación de 30 segundos, lo que significa que la topología de la canalización grabará aproximadamente 30 segundos de vídeo.  Mientras se graba el vídeo, el nodo del procesador de detección de movimiento seguirá generando eventos de **inferencia**, los cuales se mostrarán en la ventana de salida entre los eventos **RecordingAvailable** y **RecordingStopped**.

Si permite que la canalización en vivo continúe ejecutándose, el simulador RTSP alcanzará el final del archivo de vídeo y se detendrá o desconectará. El nodo de origen RTSP se volverá entonces a conectar al simulador y el proceso se repetirá.
    
## <a name="invoke-additional-direct-method-calls-to-clean-up"></a>Invocación de llamadas de método directo adicionales para la limpieza

A continuación, puede invocar métodos directos para desactivar y eliminar la canalización en vivo (en ese orden).

### <a name="deactivate-the-live-pipeline"></a>Desactivación de la canalización en vivo

Invoque el método directo `livePipelineDeactivate` con la siguiente carga:

```
{
    "@apiVersion" : "1.0",
    "name" : "Sample-Pipeline-1"
}
```

En unos segundos, verá la siguiente respuesta en la ventana de salida.

```
[DirectMethod] Invoking Direct Method [livePipelineDeactivate] to [avasample-iot-edge-device/avaedge] ...
[DirectMethod] Response from [avasample-iot-edge-device/avaedge]:
{
  "status": 200,
  "payload": null
}
```

El código de estado 200 indica que la canalización en vivo se ha desactivado correctamente.


### <a name="delete-the-live-pipeline"></a>Eliminación de la canalización en vivo

Invoque el método directo `livePipelineDelete` con la siguiente carga

```
{
    "@apiVersion" : "1.0",
    "name&quot; : &quot;Sample-Pipeline-1"
}
```

En unos segundos, verá la siguiente respuesta en la ventana de salida:

```
[DirectMethod] Invoking Direct Method [livePipelineDelete] to [avasample-iot-edge-device/avaedge] ...
[DirectMethod] Response from [avasample-iot-edge-device/avaedge]:
{
  "status": 200,
  "payload": null
}
```

El código de estado 200 de la respuesta indica que la canalización en vivo se eliminó correctamente.


### <a name="delete-the-pipeline-topology"></a>Eliminación de la topología de canalización

Invoque el método directo `pipelineTopologyDelete` con la siguiente carga:

```
{
    "@apiVersion" : "1.0",
    "name&quot; : &quot;EVRtoVideoSinkOnMotionDetection"
}
```

En unos segundos, verá la siguiente respuesta en la ventana de salida.

```
[DirectMethod] Invoking Direct Method [pipelineTopologyDelete] to [avasample-iot-edge-device/avaedge] ...
[DirectMethod] Response from [avasample-iot-edge-device/avaedge]:
{
  "status": 200,
  "payload": null
}
```

El código de estado 200 indica que la topología de canalización se ha eliminado correctamente.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [prerequisites](./includes/common-includes/clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [reproducir las grabaciones de vídeo](playback-recordings-how-to.md)
* Pruebe el [inicio rápido para analizar vídeo en directo](analyze-live-video-use-your-model-http.md)
