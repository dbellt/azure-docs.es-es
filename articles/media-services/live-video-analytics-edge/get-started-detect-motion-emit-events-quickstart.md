---
title: Introducción a Azure Live Video Analytics on IoT Edge
description: En este inicio rápido se muestra cómo empezar a usar Azure Live Video Analytics on IoT Edge. Aprenda a detectar movimiento en transmisiones de vídeo en directo.
ms.topic: quickstart
ms.date: 04/27/2020
ms.openlocfilehash: 06a84a53db4e981802ad3de930288b6aa467f563
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110371476"
---
# <a name="quickstart-get-started-with-live-video-analytics-on-iot-edge"></a>Inicio rápido: Introducción a Live Video Analytics en IoT Edge

[!INCLUDE [redirect to Azure Video Analyzer](./includes/redirect-video-analyzer.md)]

Este inicio rápido le guiará por los pasos necesarios para empezar a usar Live Video Analytics en IoT Edge. Utiliza una máquina virtual de Azure como dispositivo IoT Edge. También usa una secuencia de vídeo en directo simulada. 

Después de completar los pasos de la configuración, podrá ejecutar una secuencia de vídeo en directo simulada mediante un grafo multimedia que detecta e informa de todos los movimientos de esa secuencia. El siguiente diagrama representa gráficamente ese grafo multimedia.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/analyze-live-video/motion-detection.svg" alt-text="Live Video Analytics basado en la detección de movimiento":::

Puede ver el siguiente vídeo con pasos detallados sobre cómo empezar a usar Live Video Analytics en IoT Edge:

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4Hcax]

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure que tenga una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), en caso de que aún no lo haya hecho.

  > [!NOTE]
  > Necesitará una suscripción de Azure con permisos para crear entidades de servicio (el **rol de propietario** permite esto). Si no tiene los permisos adecuados, póngase en contacto con el administrador de la cuenta para que se los conceda.  

* [Visual Studio Code](https://code.visualstudio.com/) en la máquina de desarrollo. Asegúrese de tener la [extensión Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* Asegúrese de que la red a la que está conectada su máquina de desarrollo permita usar el protocolo Advanced Message Queuing Protocol (AMQP) a través del puerto 5671 para el tráfico saliente. Esta configuración permite a Azure IoT Tools comunicarse con Azure IoT Hub.

> [!TIP]
> Al instalar la extensión de Azure IoT Tools, es posible que se le pida que instale Docker. Si lo desea, ignore esta petición.

## <a name="set-up-azure-resources"></a>Configuración de los recursos de Azure

Este tutorial requiere los siguientes recursos de Azure:

* IoT Hub
* Cuenta de almacenamiento
* Cuenta de Azure Media Services
* Una máquina virtual Linux en Azure, con el [runtime de IoT Edge](../../iot-edge/how-to-install-iot-edge.md) instalado

Para este inicio rápido, se recomienda usar el [script de configuración de recursos de Live Video Analytics](https://github.com/Azure/live-video-analytics/tree/master/edge/setup) para implementar los recursos necesarios en su suscripción de Azure. Para hacerlo, siga estos pasos:

1. Vaya a [Azure Portal](https://portal.azure.com) y seleccione el icono de Cloud Shell.
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/quickstarts/cloud-shell.png" alt-text="Cloud Shell":::
1. Si es la primera vez que usa Cloud Shell, se le pedirá que seleccione una suscripción para crear una cuenta de almacenamiento y un recurso compartido de Microsoft Azure Files. Seleccione **Create storage** (Crear almacenamiento) para crear una cuenta de almacenamiento para la información de la sesión de Cloud Shell. Esta cuenta de almacenamiento es independiente de la que creará el script para usarla con su cuenta de Azure Media Services.
1. En el menú desplegable del lado izquierdo de la ventana de Cloud Shell, seleccione el entorno **Bash**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/quickstarts/env-selector.png" alt-text="Selector de entorno":::
1. Ejecute el siguiente comando.

    ```
    bash -c "$(curl -sL https://aka.ms/lva-edge/setup-resources-for-samples)"
    ```
    
    Tras completar correctamente el script, debería ver todos los recursos necesarios en la suscripción. El script configurará 12 recursos en total:
    1. **Punto de conexión de streaming**: este recurso le ayudará a reproducir el activo AMS grabado.
    1. **Máquina virtual**: es una máquina virtual que funcionará como dispositivo perimetral.
    1. **Disco**: se trata de un disco de almacenamiento que se conecta a la máquina virtual para almacenar elementos multimedia y artefactos.
    1. **Grupo de seguridad de red**: se usa para filtrar el tráfico de red hacia y desde los recursos de Azure en una red virtual de Azure.
    1. **Interfaz de red**: permite que una máquina virtual de Azure se comunique con recursos de Internet o Azure, entre otros.
    1. **Conexión bastión**: le permite conectarse a la máquina virtual mediante el explorador y Azure Portal.
    1. **Dirección IP pública**: facilita la comunicación de los recursos de Azure con los servicios de Azure orientados al público y a Internet.
    1. **Red virtual**: hace posible que muchos tipos de recursos de Azure, como las máquinas virtuales, se comuniquen de forma segura entre sí, con Internet y con las redes locales. Obtenga más información sobre las [redes virtuales](../../virtual-network/virtual-networks-overview.md).
    1. **IoT Hub**: funciona como un centro de mensajes común para la comunicación bidireccional entre la aplicación de IoT, los módulos de IoT Edge y los dispositivos que administra.
    1. **Cuenta de Media Services**: ayuda a administrar y transmitir contenido multimedia en Azure.
    1. **Cuenta de almacenamiento**: debe tener una cuenta de almacenamiento principal y puede tener cualquier número de cuentas de almacenamiento secundarias asociadas a la cuenta de Media Services. Para más información, consulte [Cuentas de Azure Storage con cuentas de Azure Media Services](../latest/storage-account-concept.md).
    1. **Registro de contenedor**: ayuda a almacenar y administrar las imágenes privadas del contenedor de Docker y los artefactos relacionados.

En la salida del script, una tabla de recursos enumera el nombre del centro de IoT. Busque el tipo de recurso **`Microsoft.Devices/IotHubs`** y anote el nombre, ya que lo necesitará en el paso siguiente.  

> [!NOTE]
> El script también genera varios archivos de configuración en el directorio ***~/clouddrive/lva-sample/*** . Los necesitará más adelante en este mismo inicio rápido.

> [!TIP]
> Si tiene problemas con los recursos de Azure que se crean, consulte la **[guía de solución de problemas](troubleshoot-how-to.md#common-error-resolutions)** para resolver algunos de los problemas más comunes que pueden surgir.

## <a name="deploy-modules-on-your-edge-device"></a>Implementación de módulos en el dispositivo perimetral

En Cloud Shell, ejecute el comando siguiente.

```
az iot edge set-modules --hub-name <iot-hub-name> --device-id lva-sample-device --content ~/clouddrive/lva-sample/edge-deployment/deployment.amd64.json
```

Este comando implementa los siguientes módulos en el dispositivo perimetral, que en este caso es la máquina virtual Linux.

* Live Video Analytics on IoT Edge (nombre del módulo `lvaEdge`)
* Simulador del Protocolo de transmisión en tiempo real (RTSP) (nombre del módulo `rtspsim`)

El módulo del simulador de RTSP simula una secuencia de vídeo en directo mediante un archivo de vídeo que se copió en el dispositivo perimetral al ejecutar el [script de configuración de los recursos de Live Video Analytics](https://github.com/Azure/live-video-analytics/tree/master/edge/setup). 

Ahora se implementan los módulos, pero no hay grafos multimedia activos.

## <a name="configure-the-azure-iot-tools-extension"></a>Configuración de la extensión Azure IoT Tools

Siga estas instrucciones para conectarse a su centro de IoT mediante la extensión Azure IoT Tools.

1. En Visual Studio Code, abra la pestaña **Extensiones** (o presione Ctrl + Mayús + X) y busque Azure IoT Hub.
1. Haga clic con el botón derecho y seleccione la opción **Configuración de la extensión**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/extensions-tab.png" alt-text="Configuración de la extensión":::
1. Busque y habilite "Show Verbose Message" (Mostrar mensaje detallado).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/show-verbose-message.png" alt-text="Show Verbose Message"::: (Mostrar mensaje detallado)
1. Seleccione **Ver** > **Explorador**. O bien, seleccione Ctrl+Mayús+E.
1. En la esquina inferior izquierda de la pestaña **Explorador**, seleccione **Azure IoT Hub**.
1. Seleccione el icono **Más opciones** para ver el menú contextual. Luego, seleccione **Set IoT Hub Connection String** (Establecer cadena de conexión de IoT Hub).
1. Cuando aparezca un cuadro de entrada, escriba la cadena de conexión de IoT Hub. En Cloud Shell, puede obtener la cadena de conexión de *~/clouddrive/lva-sample/appsettings.json*.

> [!NOTE]
> Es posible que se le pida que proporcione información del punto de conexión integrado de IoT Hub. Para obtener esa información, en Azure Portal, vaya a su centro de IoT y busque la opción **Puntos de conexión integrados** en el panel de navegación izquierdo. Haga clic ahí y busque el **punto de conexión compatible con el centro de eventos** en la sección **Punto de conexión compatible con el centro de eventos**. Copie y use el texto del cuadro. El punto de conexión será similar a este:  
    ```
    Endpoint=sb://iothub-ns-xxx.servicebus.windows.net/;SharedAccessKeyName=iothubowner;SharedAccessKey=XXX;EntityPath=<IoT Hub name>
    ```

Si la conexión se realiza correctamente, aparece la lista de dispositivos perimetrales. Debería ver al menos un dispositivo denominado **lva-sample-device**. Ahora puede administrar los dispositivos IoT Edge e interactuar con Azure IoT Hub mediante el menú contextual. Para ver los módulos implementados en el dispositivo perimetral, en **lva-sample-device**, expanda el nodo **Módulos**.

![Nodo lva-sample-device](./media/quickstarts/lva-sample-device-node.png)

> [!TIP]
> Si ha [implementado manualmente Live Video Analytics en IoT Edge](deploy-iot-edge-device.md) en un dispositivo perimetral (por ejemplo, un dispositivo ARM64), verá que el módulo se muestra en ese dispositivo, bajo Azure IoT Hub. Puede seleccionar ese módulo y seguir el resto de los pasos siguientes.

## <a name="use-direct-method-calls"></a>Uso de llamadas de método directo

Puede usar el módulo para analizar secuencias de vídeo en directo mediante la invocación de métodos directos. Para más información, consulte [Métodos directos para Live Video Analytics on IoT Edge](direct-methods.md). 

### <a name="invoke-graphtopologylist"></a>Invocación de GraphTopologyList

Para enumerar todas las [topologías de grafos](media-graph-concept.md#media-graph-topologies-and-instances) del módulo:

1. En Visual Studio Code, haga clic con el botón derecho en el módulo **lvaEdge** y seleccione **Invoke Module Direct Method** (Invocar método directo de módulo).
1. En el cuadro que aparece, escriba *GraphTopologyList*.
1. Copie la siguiente carga de JSON y péguela en el cuadro. Después, seleccione la tecla Entrar.

    ```
    {
        "@apiVersion" : "2.0"
    }
    ```

    Tras unos segundos, la ventana **SALIDA** muestra la respuesta siguiente.

    ```
    [DirectMethod] Invoking Direct Method [GraphTopologyList] to [lva-sample-device/lvaEdge] ...
    [DirectMethod] Response from [lva-sample-device/lvaEdge]:
    {
      "status": 200,
      "payload": {
        "value": []
      }
    }
    ```
    
    Esta respuesta es la que se espera, ya que no se ha creado ninguna topología de grafo.
    

### <a name="invoke-graphtopologyset"></a>Invocación de GraphTopologySet

Tal como hicimos antes, ahora puede invocar `GraphTopologySet` para establecer una [topología de grafo](media-graph-concept.md#media-graph-topologies-and-instances). Use el siguiente archivo JSON como carga.

```
{
    "@apiVersion": "2.0",
    "name": "MotionDetection",
    "properties": {
        "description": "Analyzing live video to detect motion and emit events",
        "parameters": [
            {
                "name": "rtspUserName",
                "type": "String",
                "description": "rtsp source user name.",
                "default": "dummyUserName"
            },
            {
                "name": "rtspPassword",
                "type": "String",
                "description": "rtsp source password.",
                "default": "dummyPassword"
            },
            {
                "name": "rtspUrl",
                "type": "String",
                "description": "rtsp Url"
            }
        ],
        "sources": [
            {
                "@type": "#Microsoft.Media.MediaGraphRtspSource",
                "name": "rtspSource",
                "endpoint": {
                    "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
                    "url": "${rtspUrl}",
                    "credentials": {
                        "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
                        "username": "${rtspUserName}",
                        "password": "${rtspPassword}"
                    }
                }
            }
        ],
        "processors": [
            {
                "@type": "#Microsoft.Media.MediaGraphMotionDetectionProcessor",
                "name": "motionDetection",
                "sensitivity": "medium",
                "inputs": [
                    {
                        "nodeName": "rtspSource"
                    }
                ]
            }
        ],
        "sinks": [
            {
                "@type": "#Microsoft.Media.MediaGraphIoTHubMessageSink",
                "name": "hubSink",
                "hubOutputName": "inferenceOutput",
                "inputs": [
                    {
                        "nodeName": "motionDetection"
                    }
                ]
            }
        ]
    }
}

```

Esta carga JSON crea una topología de grafo que define tres parámetros. Dos de esos parámetros tienen valores predeterminados. La topología tiene un nodo de origen (origen RTSP), un nodo de procesador (procesador de detección de movimiento) y un nodo receptor (receptor de IoT Hub).

En unos segundos, se ve la siguiente respuesta en la ventana **SALIDA**.

```
[DirectMethod] Invoking Direct Method [GraphTopologySet] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 201,
  "payload": {
    "systemData": {
      "createdAt": "2020-05-19T07:41:34.507Z",
      "lastModifiedAt&quot;: &quot;2020-05-19T07:41:34.507Z"
    },
    "name": "MotionDetection",
    "properties": {
      "description": "Analyzing live video to detect motion and emit events",
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
        }
      ],
      "sources": [
        {
          "@type": "#Microsoft.Media.MediaGraphRtspSource",
          "name": "rtspSource",
          "transport": "Tcp",
          "endpoint": {
            "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
            "url": "${rtspUrl}",
            "credentials": {
              "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
              "username&quot;: &quot;${rtspUserName}"
            }
          }
        }
      ],
      "processors": [
        {
          "@type": "#Microsoft.Media.MediaGraphMotionDetectionProcessor",
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
          "@type": "#Microsoft.Media.MediaGraphIoTHubMessageSink",
          "hubOutputName": "inferenceOutput",
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

El estado que se devuelve es 201, que indica que se ha creado una topología. 

Pruebe los siguientes pasos:

1. Vuelva a invocar a `GraphTopologySet`. El código de estado que se devuelve es 200. Este código indica que se ha actualizado correctamente una topología ya existente.
1. Vuelva a invocar a `GraphTopologySet`, pero cambie la cadena de descripción. El código de estado devuelto es 200 y la descripción se ha actualizado al nuevo valor.
1. Invoque a `GraphTopologyList` como se describió en la sección anterior. Ahora puede ver la topología `MotionDetection` en la carga que se devuelve.

### <a name="invoke-graphtopologyget"></a>Invocación de GraphTopologyGet

Invoque a `GraphTopologyGet` mediante el uso de la siguiente carga.

```
{
    "@apiVersion" : "2.0",
    "name&quot; : &quot;MotionDetection"
}
```

En unos segundos, se ve la siguiente respuesta en la ventana **SALIDA**:

```
[DirectMethod] Invoking Direct Method [GraphTopologyGet] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 200,
  "payload": {
    "systemData": {
      "createdAt": "2020-05-19T07:41:34.507Z",
      "lastModifiedAt&quot;: &quot;2020-05-19T07:41:34.507Z"
    },
    "name": "MotionDetection",
    "properties": {
      "description": "Analyzing live video to detect motion and emit events",
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
        }
      ],
      "sources": [
        {
          "@type": "#Microsoft.Media.MediaGraphRtspSource",
          "name": "rtspSource",
          "transport": "Tcp",
          "endpoint": {
            "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
            "url": "${rtspUrl}",
            "credentials": {
              "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
              "username&quot;: &quot;${rtspUserName}"
            }
          }
        }
      ],
      "processors": [
        {
          "@type": "#Microsoft.Media.MediaGraphMotionDetectionProcessor",
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
          "@type": "#Microsoft.Media.MediaGraphIoTHubMessageSink",
          "hubOutputName": "inferenceOutput",
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

En la carga de respuesta, observe estos detalles:

* El código de estado es 200, lo que indica que se ha realizado correctamente.
* La carga incluye las marcas de tiempo `created` y `lastModified`.

### <a name="invoke-graphinstanceset"></a>Invocación de GraphInstanceSet

Cree una instancia de grafo que haga referencia a la topología de grafo anterior. Las instancias de grafos le permiten analizar secuencias de vídeo en directo desde muchas cámaras mediante la misma topología de grafos. Para más información, consulte el apartado [Topologías e instancias de los grafos multimedia](media-graph-concept.md#media-graph-topologies-and-instances).

Invoque el método directo `GraphInstanceSet` mediante la carga siguiente.

```
{
    "@apiVersion" : "2.0",
    "name" : "Sample-Graph-1",
    "properties" : {
        "topologyName" : "MotionDetection",
        "description" : "Sample graph description",
        "parameters" : [
            { "name" : "rtspUrl", "value" : "rtsp://rtspsim:554/media/camera-300s.mkv" }
        ]
    }
}
```

Tenga en cuenta que esta carga:

* Especifica el nombre de la topología (`MotionDetection`) para la que se debe crear la instancia.
* Contiene el valor de parámetro para `rtspUrl`, que no tenía un valor predeterminado en la carga de la topología de grafo. Este valor es un vínculo al vídeo de ejemplo siguiente:
    > [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4LTY4]
En unos segundos, se ve la siguiente respuesta en la ventana **SALIDA**:

```
[DirectMethod] Invoking Direct Method [GraphInstanceSet] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 201,
  "payload": {
    "name": "Sample-Graph-1",
    "properties": {
      "created": "2020-05-19T07:44:33.868Z",
      "lastModified": "2020-05-19T07:44:33.868Z",
      "state": "Inactive",
      "description": "Sample graph description",
      "topologyName": "MotionDetection",
      "parameters": [
        {
          "name": "rtspUrl",
          "value&quot;: &quot;rtsp://rtspsim:554/media/camera-300s.mkv"
        }
      ]
    }
  }
}
```

En la carga de respuesta, observe que:

* El código de estado es 201, lo que indica que se ha creado una instancia.
* El estado es `Inactive`, lo que indica que la instancia de grafo se ha creado, pero no se ha activado. Para más información, consulte los [estados de los grafos multimedia](media-graph-concept.md).

Pruebe los siguientes pasos:

1. Vuelva a invocar a `GraphInstanceSet`, y use para ello la misma carga. Observe que el código de estado devuelto es 200.
1. Vuelva a invocar a `GraphInstanceSet`, pero use otra descripción. Observe la descripción actualizada en la carga de respuesta, la cual indica que la instancia de grafo se actualizó correctamente.
1. Invoque a `GraphInstanceSet`, pero cambie el nombre a `Sample-Graph-2`. En la carga de respuesta, observe la instancia del grafo que se acaba de crear (es decir, el código de estado 201).

### <a name="invoke-graphinstanceactivate"></a>Invocación de GraphInstanceActivate

Ahora, active la instancia de grafo para iniciar el flujo de vídeo en directo a través del módulo. Invoque el método directo `GraphInstanceActivate` mediante la carga siguiente.

```
{
    "@apiVersion" : "2.0",
    "name" : "Sample-Graph-1"
}
```

En pocos segundos, verá la siguiente respuesta en la ventana **SALIDA**.

```
[DirectMethod] Invoking Direct Method [GraphInstanceActivate] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 200,
  "payload": null
}
```

El código de estado 200 indica que la instancia de grafo se ha activado correctamente.

### <a name="invoke-graphinstanceget"></a>Invocación de GraphInstanceGet

Ahora, invoque el método directo `GraphInstanceGet` mediante la carga siguiente.

```
 {
     "@apiVersion" : "2.0",
     "name&quot; : &quot;Sample-Graph-1"
 }
 ```

En pocos segundos, verá la siguiente respuesta en la ventana **SALIDA**.

```
[DirectMethod] Invoking Direct Method [GraphInstanceGet] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 200,
  "payload": {
    "name": "Sample-Graph-1",
    "properties": {
      "created": "2020-05-19T07:44:33.868Z",
      "lastModified": "2020-05-19T07:44:33.868Z",
      "state": "Active",
      "description": "graph description",
      "topologyName": "MotionDetection",
      "parameters": [
        {
          "name": "rtspUrl",
          "value&quot;: &quot;rtsp://rtspsim:554/media/camera-300s.mkv"
        }
      ]
    }
  }
}
```

En la carga de respuesta, observe los siguientes detalles:

* El código de estado es 200, lo que indica que se ha realizado correctamente.
* El estado es `Active`, lo que indica que la instancia del grafo tiene ahora ese estado.

## <a name="observe-results"></a>Observación de resultados

La instancia del grafo que hemos creado y activado usa el nodo de procesador de detección de movimiento para detectar el movimiento en la secuencia de vídeo en directo entrante. Envía eventos al nodo receptor de IoT Hub, y dichos eventos se retransmiten a IoT Edge Hub. 

Para observar los resultados, siga estos pasos.

1. En Visual Studio Code, abra el panel **Explorer** (Explorador). En la esquina inferior izquierda, busque **Azure IoT Hub**.
2. Expanda el nodo **Devices** (Dispositivos).
3. Haga clic con el botón derecho en **Iva-sample-device** y elija la opción **Iniciar la supervisión del punto de conexión de eventos integrado**.

    ![Iniciar supervisión de eventos de IoT Hub](./media/quickstarts/start-monitoring-iothub-events.png)

    > [!NOTE]
    > Es posible que se le pida que proporcione información del punto de conexión integrado del centro de IoT. Para obtener esa información, en Azure Portal, vaya a su centro de IoT y busque la opción **Puntos de conexión integrados** en el panel de navegación izquierdo. Haga clic ahí y busque el **punto de conexión compatible con el centro de eventos** en la sección **Punto de conexión compatible con el centro de eventos**. Copie y use el texto del cuadro. El punto de conexión será similar a este:  
        ```
        Endpoint=sb://iothub-ns-xxx.servicebus.windows.net/;SharedAccessKeyName=iothubowner;SharedAccessKey=XXX;EntityPath=<IoT Hub name>
        ```
    
La ventana **SALIDA** muestra el siguiente mensaje:

```
[IoTHubMonitor] [7:44:33 AM] Message received from [lva-sample-device/lvaEdge]:
{
    "body": {
    "timestamp": 143005362606360,
    "inferences": [
        {
        "type": "motion",
        "motion": {
            "box": {
            "l": 0.828452,
            "t": 0.455224,
            "w": 0.1,
            "h": 0.088889
            }
        }
        },
        {
        "type": "motion",
        "motion": {
            "box": {
            "l": 0.661088,
            "t": 0.597015,
            "w": 0.0625,
            "h": 0.051852
            }
        }
        }
    ]
    }
}
```

Observe esta información:

* El mensaje contiene una sección `body` y una sección `applicationProperties`. Para más información, consulte [Creación y lectura de mensajes de IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).
* En `applicationProperties`, `subject` hace referencia al nodo de `MediaGraph` desde el que se generó el mensaje. En este caso, el mensaje se origina en el procesador de detección del movimiento.
* En `applicationProperties`, `eventType` indica que este es un evento de análisis.
* El valor `eventTime` es la hora en que se produjo el evento.
* La sección `body` contiene datos sobre el evento de análisis. En este caso, el evento es un evento de inferencia, por lo que el cuerpo contiene los datos `timestamp` y `inferences`.
* La sección `inferences` indica que el valor de `type` es `motion`. Proporciona datos adicionales sobre el evento `motion`.

Si deja que el grafo multimedia se ejecute un tiempo, verá el siguiente mensaje en la ventana **SALIDA**.

```
[IoTHubMonitor] [7:47:45 AM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "sdp&quot;: &quot;SDP:\nv=0\r\no=- 1588948185746703 1 IN IP4 172.xx.xx.xx\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/camera-300s.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.04.12\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/camera-300s.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets={SPS}\r\na=control:track1\r\n"
  },
  "applicationProperties": {
    "dataVersion": "1.0",
    "topic": "/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<my-resource-group>/providers/microsoft.media/mediaservices/<ams-account-name>",
    "subject": "/graphInstances/Sample-Graph-1/sources/rtspSource",
    "eventType": "Microsoft.Media.Graph.Diagnostics.MediaSessionEstablished",
    "eventTime": "2020-05-19T07:47:45.747Z"
  }
}
```

En este mensaje, tenga en cuenta los siguientes detalles:

* En `applicationProperties`, `subject` indica que el mensaje se generó desde el nodo de origen RTSP del grafo multimedia.
* En `applicationProperties`, `eventType` indica que este evento es un diagnóstico.
* `body` contiene datos sobre el evento de diagnóstico. En este caso, el mensaje contiene el cuerpo, ya que el evento es `MediaSessionEstablished`.

## <a name="invoke-additional-direct-methods-to-clean-up"></a>Invocación de métodos directos adicionales para la limpieza

Invoque métodos directos para desactivar primero la instancia del grafo y, después, eliminarla.

### <a name="invoke-graphinstancedeactivate"></a>Invocación de GraphInstanceDeactivate

Invoque el método directo `GraphInstanceDeactivate` mediante la carga siguiente.

```
{
    "@apiVersion" : "2.0",
    "name" : "Sample-Graph-1"
}
```

En pocos segundos, verá la siguiente respuesta en la ventana **SALIDA**:

```
[DirectMethod] Invoking Direct Method [GraphInstanceDeactivate] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 200,
  "payload": null
}
```

El código de estado 200 indica que la instancia del grafo se ha desactivado correctamente.

A continuación, intente invocar `GraphInstanceGet` como se ha indicado en este mismo artículo. Observe el valor de `state`.

### <a name="invoke-graphinstancedelete"></a>Invocación de GraphInstanceDelete

Invoque el método directo `GraphInstanceDelete` mediante la carga siguiente.

```
{
    "@apiVersion" : "2.0",
    "name&quot; : &quot;Sample-Graph-1"
}
```

En pocos segundos, verá la siguiente respuesta en la ventana **SALIDA**:

```
[DirectMethod] Invoking Direct Method [GraphInstanceDelete] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 200,
  "payload": null
}
```

El código de estado 200 indica que la instancia del grafo se ha eliminado correctamente.

### <a name="invoke-graphtopologydelete"></a>Invocación de GraphTopologyDelete

Invoque el método directo `GraphTopologyDelete` mediante la siguiente carga.

```
{
    "@apiVersion" : "2.0",
    "name&quot; : &quot;MotionDetection"
}
```

En pocos segundos, verá la siguiente respuesta en la ventana **SALIDA**.

```
[DirectMethod] Invoking Direct Method [GraphTopologyDelete] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 200,
  "payload": null
}
```

El código de estado 200 indica que la topología del grafo se ha eliminado correctamente.

Pruebe los siguientes pasos:

1. Invoque a `GraphTopologyList` y observe que el módulo no contiene topologías de grafo.
1. Invoque a `GraphInstanceList` y use la misma carga que `GraphTopologyList`. Observe que no se enumeran instancias de grafo.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, elimine los recursos que ha creado en este inicio rápido.

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [grabar vídeo mediante Live Video Analytics on IoT Edge](continuous-video-recording-tutorial.md).
* Más información sobre los [mensajes de diagnóstico](monitoring-logging.md).
