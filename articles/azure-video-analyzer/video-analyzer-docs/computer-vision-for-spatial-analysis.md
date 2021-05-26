---
title: 'Análisis de vídeo en directo con Computer Vision para análisis espacial: Azure'
description: En este tutorial se muestra cómo usar Azure Video Analyzer junto con la característica de IA de Computer Vision para análisis espacial, parte de Azure Cognitive Services, para analizar una fuente de vídeo en directo desde una cámara IP (simulada).
author: Juliako
ms.author: juliako
ms.service: azure-video-analyzer
ms.topic: tutorial
ms.date: 04/01/2021
ms.openlocfilehash: d54983e25abc769a75923e59c483a4cf9495770f
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110384248"
---
# <a name="tutorial-live-video-with-computer-vision-for-spatial-analysis-preview"></a>Tutorial: Vídeo en directo con Computer Vision para análisis espacial (versión preliminar)

En este tutorial se muestra cómo usar Azure Video Analyzer junto con el [servicio de IA de Computer Vision para el análisis espacial, parte de Azure Cognitive Services,](../../cognitive-services/computer-vision/intro-to-spatial-analysis-public-preview.md) para analizar una fuente de vídeo en directo desde una cámara IP (simulada). Verá cómo este servidor de inferencia le permite analizar el vídeo en streaming para comprender las relaciones espaciales entre las personas y el movimiento en el espacio físico. Un subconjunto de los fotogramas de la fuente de vídeo se envía a este servidor de inferencia y los resultados se envían al centro de IoT Edge. Cuando se cumplen algunas condiciones, los clips de vídeo se graban y almacenan en la cuenta de Video Analyzer.

En este tutorial, aprenderá lo siguiente:

> [!div class="checklist"]
>
> - Configuración de recursos.
> - Examine el código.
> - Ejecución del código de ejemplo
> - Supervisión de eventos.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="suggested-pre-reading"></a>Sugerencias antes de la lectura

Consulte estos artículos antes de empezar:

- [Introducción a Video Analyzer](overview.md)
- [Terminología de Video Analyzer](terminology.md)
- [Conceptos de canalización](pipeline.md)
- [Grabación de vídeo basada en eventos](record-event-based-live-video.md)
- [Tutorial: Desarrollo de un módulo IoT Edge](../../iot-edge/tutorial-develop-for-linux.md)

## <a name="prerequisites"></a>Requisitos previos

Estos son los requisitos previos para conectar el módulo de análisis espacial a un módulo de Azure Video Analyzer.

- [Visual Studio Code](https://code.visualstudio.com/) en la máquina de desarrollo. Asegúrese de tener la [extensión Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
  - Asegúrese de que la red a la que está conectada la máquina de desarrollo permita Advanced Message Queueing Protocol a través del puerto 5671. Esta configuración permite a Azure IoT Tools comunicarse con Azure IoT Hub.
- [Contenedor de Computer Vision de Azure Cognitive Services](../../cognitive-services/computer-vision/intro-to-spatial-analysis-public-preview.md) para análisis espacial.
  Para usar este contenedor, debe tener un recurso de Computer Vision para obtener la **clave de API** y el **URI de punto de conexión** asociados. La clave está disponible en las páginas Claves e Información general de Computer Vision en Azure Portal. La clave y el punto de conexión son necesarios para iniciar el contenedor.

## <a name="set-up-azure-resources"></a>Configuración de los recursos de Azure

1. Para ejecutar el contenedor de análisis espacial se necesita un dispositivo de proceso con una [GPU NVIDIA Tesla T4](https://www.nvidia.com/data-center/tesla-t4/). Se recomienda usar [Azure Stack Edge](https://azure.microsoft.com/products/azure-stack/edge/) con aceleración de GPU; sin embargo, el contenedor se ejecuta en cualquier otra máquina de escritorio que tenga [Ubuntu Desktop 18.04 LTS](http://releases.ubuntu.com/18.04/) instalado en el equipo host.

   #### <a name="azure-stack-edge-device"></a>[Dispositivo Azure Stack Edge](#tab/azure-stack-edge)

   Azure Stack Edge es una solución de hardware como servicio y un dispositivo informático perimetral habilitado para inteligencia artificial que cuenta con funcionalidades de transferencia de datos de red. Para instrucciones detalladas sobre la preparación y configuración, consulte la [documentación de Azure Stack Edge](../../databox-online/azure-stack-edge-deploy-prep.md).

   #### <a name="desktop-machine"></a>[Máquina de escritorio](#tab/desktop-machine)

   #### <a name="minimum-hardware-requirements"></a>Requisitos mínimos de hardware

   - 4 GB de RAM del sistema
   - 4 GB de RAM de GPU
   - CPU de 8 núcleos
   - 1 GPU NVIDIA Tesla T4
   - 20 GB de espacio en HDD

   #### <a name="recommended-hardware"></a>Hardware recomendado

   - 32 GB de RAM del sistema
   - 16 GB de RAM de GPU
   - CPU de 8 núcleos
   - 2 GPU NVIDIA Tesla T4
   - 50 GB de espacio en SSD

   En este artículo, descargará e instalará los paquetes de software siguientes. El equipo host debe poder ejecutar lo siguiente (consulte las instrucciones a continuación):

   - [Controladores de gráficos de NVIDIA](https://docs.nvidia.com/datacenter/tesla/tesla-installation-notes/index.html) y [NVIDIA CUDA Toolkit](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)
   - Configuraciones para [NVIDIA MPS](https://docs.nvidia.com/deploy/pdf/CUDA_Multi_Process_Service_Overview.pdf) (servicio multiproceso).
   - [Docker CE](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-engine---community-1) y [NVIDIA-Docker2](https://github.com/NVIDIA/nvidia-docker)
   - Entorno de ejecución de [Azure IoT Edge](../../iot-edge/how-to-install-iot-edge.md).

   #### <a name="azure-vm-with-gpu"></a>[Azure VM con GPU](#tab/virtual-machine)

   Puede usar una [máquina virtual de la serie NC](../../virtual-machines/nc-series.md?bc=%2fazure%2fvirtual-machines%2flinux%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) con una GPU K80.

   1. Conéctese a la máquina virtual y, en el tipo de terminal, en el siguiente comando:

        `bash -c "$(curl -sL https://aka.ms/ava-edge/prep_device)"`
    
        El módulo de Live Video Analytics se ejecuta en el dispositivo perimetral con cuentas de usuario locales sin privilegios. Además, necesita ciertas carpetas locales para almacenar los datos de configuración de la aplicación. Por último, para esta guía paso a paso, se emplea un [simulador RTSP](https://github.com/Azure/video-analyzer/tree/main/edge-modules/sources/rtspsim-live555) que retransmite una fuente de vídeo en tiempo real a un módulo de AVA para su análisis. Este simulador toma como entrada archivos de vídeo grabados previamente desde un directorio de entrada. 
    
        El script prep-device usado anteriormente automatiza estas tareas, por lo que puede ejecutar un comando y hacer que todas las carpetas de entrada y configuración pertinentes, archivos de entrada de vídeo y cuentas de usuario con privilegios se creen sin problemas. Una vez que el comando finalice correctamente, debería ver las siguientes carpetas creadas en el dispositivo perimetral. 
    
        * `/home/localedgeuser/samples`
        * `/home/localedgeuser/samples/input`
        * `/var/lib/videoanalyzer`
        * `/var/media`
    
        Observe los archivos de vídeo (*.mkv) de la carpeta /home/lvaedgeuser/samples/input, ya que le servirán como los archivos de entrada que se van a analizar. 

1. [Configuración del dispositivo perimetral](../../cognitive-services/computer-vision/spatial-analysis-container.md#set-up-the-host-computer)

1. A continuación, implemente los demás recursos de Azure.

   [![Implementación en Azure](https://aka.ms/deploytoazurebutton)](https://aka.ms/ava-click-to-deploy)

   > [!NOTE]
   > El botón anterior crea y usa la máquina virtual predeterminada que NO tiene la GPU NVIDIA. Use la opción "Use existing edge device" (Usar el dispositivo perimetral existente) cuando se le pida en la plantilla de Azure Resource Manager (ARM) y use IoT Hub y la información del dispositivo del paso anterior.
   > :::image type="content" source="./media/spatial-analysis/use-existing-device.png" alt-text="Uso del dispositivo existente":::

   [!INCLUDE [resources](./includes/common-includes/azure-resources.md)]

## <a name="overview"></a>Información general

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/spatial-analysis/overview.png" alt-text="Introducción al análisis espacial":::

En este diagrama se muestra el flujo de las señales en este tutorial. Un [módulo perimetral](https://github.com/Azure/video-analyzer/tree/main/edge-modules/sources/rtspsim-live555) simula una cámara IP que hospeda un servidor de protocolo RTSP. Un nodo de [origen RTSP](pipeline.md#rtsp-source) extrae la fuente de vídeo de este servidor y envía fotogramas de vídeo al nodo del `CognitiveServicesVisionProcessor`.

El nodo de `CognitiveServicesVisionProcessor` desempeña el rol de un servidor proxy. Convierte los fotogramas de vídeo en el tipo de imagen especificado. Luego, utiliza la **memoria compartida** para retransmitir la imagen a otro módulo perimetral que ejecuta operaciones de IA detrás de un punto de conexión gRPC. En este ejemplo, ese módulo perimetral es el módulo de análisis espacial. El nodo `CognitiveServicesVisionProcessor` realiza dos acciones:

- Recopila los resultados y publica eventos en el nodo de [receptor de IoT Hub](pipeline.md#iot-hub-message-sink). que posteriormente los envía a [IoT Edge Hub](../../iot-fundamentals/iot-glossary.md#iot-edge-hub).
- También captura un clip de vídeo de 30 segundos del origen RTSP mediante un [procesador de puerta de señal](pipeline.md#signal-gate-processor) y lo almacena como un recurso de vídeo.

## <a name="create-the-computer-vision-resource"></a>Creación del recurso de Computer Vision

Es necesario crear un recurso de Azure de tipo Computer Vision en [Azure Portal](../../iot-edge/how-to-deploy-modules-portal.md) o mediante la CLI de Azure. El recurso se puede crear una vez que se haya aprobado la solicitud de acceso al contenedor y se haya registrado su identificador de suscripción de Azure. Vaya a https://aka.ms/csgate para enviar el caso de uso y el identificador de suscripción de Azure. El recurso de Azure se debe crear con la misma suscripción de Azure que se ha proporcionado en el formulario de solicitud de acceso.

### <a name="gathering-required-parameters"></a>Recopilación de los parámetros obligatorios

Hay tres parámetros principales que son necesarios para todos los contenedores de Cognitive Services, incluido el contenedor de análisis espacial. El contrato de licencia para el usuario final (CLUF) se debe haber aceptado. Además, se necesitan un identificador URI de punto de conexión y una clave de API.

### <a name="keys-and-endpoint-uri"></a>Claves e identificador URI de punto de conexión

Se usa una clave para iniciar el contenedor de análisis espacial y dicha clave está disponible en la página `Keys and Endpoint` de Azure Portal del recurso de Cognitive Services correspondiente. Vaya a esa página y busque el identificador URI del punto de conexión.  

Necesitará esta clave y el URI del punto de conexión en los archivos de manifiesto de implementación para implementar el contenedor spatialanalysis.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/spatial-analysis/keys-endpoint.png" alt-text="URI de punto de conexión":::

## <a name="set-up-your-development-environment"></a>Configurado su entorno de desarrollo

1. Clone el repositorio desde esta ubicación: [https://github.com/Azure-Samples/azure-video-analyzer-iot-edge-csharp](https://github.com/Azure-Samples/azure-video-analyzer-iot-edge-csharp).
1. En Visual Studio Code, abra la carpeta en que se ha descargado.
1. En Visual Studio Code, vaya a la carpeta src/cloud-to-device-console-app/operations.json. Allí, cree un archivo y asígnele el nombre *appsettings.json*. Este archivo contendrá la configuración necesaria para ejecutar el programa.
1. Obtenga `IotHubConnectionString` desde el dispositivo perimetral siguiendo estos pasos:

   - Vaya a su centro de IoT en Azure Portal y haga clic en `Shared access policies` en el panel de navegación izquierdo.
   - Haga clic en `iothubowner` para obtener las claves de acceso compartido.
   - Copie el valor de `Connection String – primary key` y péguelo en el cuadro de entrada de VSCode.

     La cadena de conexión tendrá el siguiente aspecto: <br/>`HostName=xxx.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=xxx`

1. Copie el contenido siguiente en el archivo. Asegúrese de reemplazar las variables.

   ```json
   {
     "IoThubConnectionString": "HostName=<IoTHubName>.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=<SharedAccessKey>",
     "deviceId": "<your Edge Device name>",
     "moduleId": "avaedge"
   }
   ```

1. Vaya a la carpeta src/edge y cree un archivo llamado .env.
1. Copie el contenido del archivo env de Azure Portal. El texto debería ser similar al siguiente código.

   ```env
   SUBSCRIPTION_ID="<Subscription ID>"
   RESOURCE_GROUP="<Resource Group>"
   AVA_PROVISIONING_TOKEN="<The provisioning token>"
   VIDEO_INPUT_FOLDER_ON_DEVICE="/home/localedgeuser/samples/input"
   VIDEO_OUTPUT_FOLDER_ON_DEVICE="/var/media"
   APPDATA_FOLDER_ON_DEVICE="/var/lib/videoanalyzer"
   CONTAINER_REGISTRY_USERNAME_myacr="<your container registry username>"  
   CONTAINER_REGISTRY_PASSWORD_myacr="<your container registry password>"
   ```

## <a name="set-up-deployment-template"></a>Configuración de la plantilla de implementación

Busque el archivo de implementación en /src/edge/deployment.spatialAnalysis.template.json. En la plantilla, está el módulo avaedge, el módulo rtspsim y nuestro módulo spatialanalysis.

Hay algunas cosas a las que debe prestar atención en el archivo de la plantilla de implementación:

1. Establezca el enlace de puerto en el módulo `spatialanalysis`.

   ```
   "PortBindings": {
       "50051/tcp": [
           {
               "HostPort": "50051"
           }
       ]
   }
   ```

1. `IpcMode` en createOptions de los módulos `avaedge` y `spatialanalysis` debe ser igual y estar establecido en **host**.
1. Para que el simulador RTSP funcione, asegúrese de que ha configurado los límites del volumen al usar un dispositivo de Azure Stack Edge.

   1. [Conéctese al recurso compartido de SMB](../../databox-online/azure-stack-edge-deploy-add-shares.md#connect-to-an-smb-share) y copie el [archivo de vídeo con el hueco de la escalera de ejemplo](https://lvamedia.blob.core.windows.net/public/2018-03-05.10-27-03.10-30-01.admin.G329.mp4) en el recurso compartido local.

      > [!VIDEO https://www.microsoft.com/videoplayer/embed/RWDRJd]

   1. Verá que el módulo rtspsim tiene la siguiente configuración:
      ```
      "createOptions": {
                          "HostConfig": {
                            "Mounts": [
                              {
                                "Target": "/live/mediaServer/media",
                                "Source": "<your Local Docker Volume Mount name>",
                                "Type": "volume"
                              }
                            ],
                            "PortBindings": {
                              "554/tcp": [
                                {
                                  "HostPort": "554"
                                }
                              ]
                            }
                          }
                        }
      ```
## <a name="generate-and-deploy-the-deployment-manifest"></a>Generación e implementación del manifiesto de implementación

El manifiesto de implementación define los módulos que se implementan en un dispositivo perimetral. También define los valores de configuración de los módulos.

Siga estos pasos para generar el manifiesto a partir del archivo de plantilla y, después, impleméntelo en el dispositivo perimetral.

1. Abra Visual Studio Code.
1. Al lado del panel `AZURE IOT HUB`, seleccione el icono Más acciones para establecer la cadena de conexión de IoT Hub. La cadena se puede copiar del archivo `src/cloud-to-device-console-app/appsettings.json`.

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/vscode-common-screenshots/set-connection-string.png" alt-text="Análisis espacial: cadena de conexión":::

1. Haga clic con el botón derecho en `src/edge/deployment.spatialAnalysis.template.json` y seleccione Generar manifiesto de implementación de IoT Edge.

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/spatial-analysis/generate-deployment-manifest.png" alt-text="Análisis espacial: deployment amd64 json":::

   Esta acción debe crear un archivo de manifiesto llamado `deployment.spatialAnalysis.amd64.json` en la carpeta src/edge/config.

1. Haga clic con el botón derecho en `src/edge/config/deployment.spatialAnalysis.amd64.json`, seleccione **Create Deployment for Single Device** (Crear una implementación para un dispositivo individual) y, a continuación, seleccione el nombre del dispositivo perimetral.

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/spatial-analysis/deployment-single-device.png" alt-text="Análisis espacial: implementación en un único dispositivo":::

1. En la parte superior de la página, se le pedirá que seleccione un dispositivo IoT Hub, elija el nombre del dispositivo perimetral en el menú desplegable.
1. Tras aproximadamente 30 segundos, en la esquina inferior izquierda de la ventana, actualice el panel **AZURE IOT HUB**. El dispositivo perimetral ahora muestra los siguientes módulos implementados:

   - Azure Video Analyzer (nombre del módulo **avaedge**).
   - Simulador del Protocolo de transmisión en tiempo real (RTSP) (nombre del módulo **rtspsim**).
   - Análisis espacial (nombre del módulo **spatialanalysis**).

Tras una implementación correcta, habrá un mensaje en la ventana SALIDA como el siguiente:

```
[Edge] Start deployment to device [<edge device name>]
[Edge] Deployment succeeded.
```

Después, puede encontrar los módulos `avaedge`, `spatialanalysis`y `rtspsim` en Dispositivos/Módulos, y su estado debe ser "**en ejecución**".

## <a name="prepare-to-monitor-events"></a>Preparación para supervisar eventos

Para ver estos eventos, siga estos pasos:

1. En Visual Studio Code, abra la pestaña **Extensiones** (o presione Ctrl + Mayús + X) y busque Azure IoT Hub.
1. Haga clic con el botón derecho y seleccione la opción **Configuración de la extensión**.

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/vscode-common-screenshots/extension-settings.png" alt-text="Configuración de la extensión":::

1. Busque y habilite "Show Verbose Message" (Mostrar mensaje detallado).

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/vscode-common-screenshots/verbose-message.png" alt-text="Show Verbose Message"::: (Mostrar mensaje detallado)

1. Abra el panel Explorador y busque **AZURE IOT HUB** en la esquina inferior izquierda, haga clic con el botón derecho y seleccione Iniciar la supervisión del punto de conexión de eventos integrado.

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/vscode-common-screenshots/start-monitoring.png" alt-text="Análisis espacial: iniciar supervisión":::

## <a name="run-the-program"></a>Ejecución del programa

Un archivo program.cs invocará los métodos directos en `src/cloud-to-device-console-app/operations.json`. Es necesario configurar operations.json y proporcionar un objeto pipelineTopology para que lo use la canalización.

En operations.json:

- Establezca el objeto del modo siguiente:

  ```json
  {
      "opName": "pipelineTopologySet",
      "opParams": {
          "topologyUrl": "https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/spatial-analysis/person-count-operation-topology.json"
      }
  },
  ```

- Cree un objeto livePipeline como este, establezca los parámetros de pipelineTopology aquí:

  ```json
  {
      "opName": "livePipelineSet",
      "opParams": {
          "name": "Sample-Pipeline-1",
          "properties": {
              "topologyName": "InferencingWithCVExtension",
              "description": "Sample pipeline description",
              "parameters": [
                  {
                      "name": "rtspUrl",
                      "value": " rtsp://rtspsim:554/media/stairwell.mkv"
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
  },
  ```

  > [!Note]
  > Consulte el uso de `CognitiveServicesVisionExtension` para conectarse con el módulo spatialanalysis. Establezca ${grpcUrl} en **tcp://spatialAnalysis:<NÚMERO_PUERTO>** , por ejemplo, tcp://spatialAnalysis:50051

  ```json
  {
          "@type": "#Microsoft.VideoAnalyzer.CognitiveServicesVisionProcessor",
          "name": "computerVisionExtension",
          "endpoint": {
            "@type": "#Microsoft.VideoAnalyzer.UnsecuredEndpoint",
            "url": "${grpcUrl}",
            "credentials": {
              "@type": "#Microsoft.VideoAnalyzer.UsernamePasswordCredentials",
              "username": "${spatialanalysisusername}",
              "password": "${spatialanalysispassword}"
            }
          },
          "inputs": [
            {
              "nodeName": "rtspSource",
              "outputSelectors": [
                {
                  "property": "mediaType",
                  "operator": "is",
                  "value": "video"
                }
              ]
            }
          ],
          "operation": {
            "@type": "#Microsoft.VideoAnalyzer.SpatialAnalysisPersonCountOperation",
            "zones": [
              {
                "zone": {
                  "@type": "#Microsoft.VideoAnalyzer.NamedPolygonString",
                  "polygon": "[[0.37,0.43],[0.48,0.42],[0.53,0.56],[0.34,0.57],[0.34,0.46]]",
                  "name": "stairlanding"
                },
                "events": [
                  {
                    "trigger": "event",
                    "outputFrequency": "1",
                    "threshold": "16",
                    "focus": "bottomCenter"
                  }
                ]
              }
            ]
          }
        }
      ],
  ```

Ejecute una sesión de depuración y siga las instrucciones de **TERMINAL**; se establecerá la topología, se establecerá pipelineTopology y livePipeline, se activará livePipeline y, por último, se eliminarán los recursos.

## <a name="interpret-results"></a>Interpretación de los resultados

Cuando se cree una instancia de pipelineTopology, verá el evento "MediaSessionEstablished"; este es un [evento MediaSessionEstablished de ejemplo](detect-motion-emit-events-quickstart.md#mediasessionestablished-event).

El módulo spatialanalysis también enviará eventos de información de inteligencia artificial a Azure Video Analyzer y, luego, a IoTHub; también se mostrarán en **SALIDA**. ENTITY son los objetos de detección y EVENT es el evento spaceanalytics. Esta salida se pasará a Azure Video Analyzer.

Salida de ejemplo para personZoneEvent (desde la operación `SpatialAnalysisPersonZoneCrossingOperation`):

```
{
  "timestamp": 145666725597833,
  "inferences": [
    {
      "type": "entity",
      "inferenceId": "8724dff43d3c4716936aa6c9a808ee2e",
      "entity": {
        "tag": {
          "value": "person",
          "confidence": 0.9980469
        },
        "box": {
          "l": 0.3848941,
          "t": 0.28569314,
          "w": 0.092775516,
          "h": 0.3819766
        }
      },
      "extensions": {
        "footprintX": "inf",
        "centerGroundPointY": "0.0",
        "trackingId": "80eeb5bbb6f542b89b5b16a1e37c53c4",
        "footprintY": "inf",
        "centerGroundPointX": "0.0"
      }
    },
    {
      "type": "event",
      "inferenceId": "0b0de3d228b640488fc0624950a6c9e8",
      "relatedInferences": [
        "8724dff43d3c4716936aa6c9a808ee2e"
      ],
      "event": {
        "name": "personZoneEnterExitEvent",
        "properties": {
          "trackingId": "80eeb5bbb6f542b89b5b16a1e37c53c4",
          "status": "Enter",
          "zone": "door"
        }
      }
    }
  ]
}
```

## <a name="operations"></a>Operaciones:

### <a name="person-zone-crossing"></a>Cruce de zona de personas

#### <a name="parameters"></a>Parámetros:

| Nombre                      | Tipo    | Descripción                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ------------------------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| zones                     | list    | Lista de zonas.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| name                      | string  | Nombre descriptivo de esta zona.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| polygon                   | string  | Cada par de valores representa la x e y de los vértices del polígono. El polígono representa las áreas en las que se realiza el seguimiento o el recuento de personas. Los valores flotantes representan la posición del vértice relativa a la esquina superior izquierda. Para calcular los valores x, y absolutos, debe multiplicar estos valores por el tamaño del fotograma. Los eventos flotantes del umbral se generan cuando la persona tiene un tamaño mayor que este número de píxeles dentro de la zona. El valor predeterminado es 48 cuando el tipo es zonecrossing, y 16 cuando la hora es DwellTime. Estos son los valores recomendados para lograr la máxima precisión. |
| eventType                 | string  | Para cognitiveservices.vision.spatialanalysis-personcrossingpolygon, debe ser zonecrossing o zonedwelltime.                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| desencadenador                   | string  | Tipo de desencadenador para enviar un evento. Valores admitidos: "event": se activa cuando alguna persona entra o sale de la zona.                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| foco                     | string  | Ubicación del punto en el rectángulo delimitador de la persona usado para calcular los eventos. El valor del foco puede ser footprint (la superficie de la persona), bottom_center (la parte inferior central del rectángulo delimitador de la persona) o center (el centro del cuadro delimitador de la persona). El valor predeterminado es footprint.                                                                                                                                                                                                                                                                                               |
| enableFaceMaskClassifier  | boolean | true para habilitar la detección de personas que lleven máscaras faciales en la secuencia de vídeo; false para deshabilitarla. De manera predeterminada, esta opción está deshabilitada. La detección de máscaras faciales requiere que el parámetro de ancho de vídeo de entrada sea 1920 "INPUT_VIDEO_WIDTH": 1920. No se devolverá el atributo de máscaras faciales.                                                                                                                                                                                                                                                                                                        |
| detectorNodeConfiguration | string  | Los parámetros DETECTOR_NODE_CONFIG para todas las operaciones de análisis espacial.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |

#### <a name="output"></a>Salida:

```json
{
  "timestamp": 145666725597833,
  "inferences": [
    {
      "type": "entity",
      "inferenceId": "8724dff43d3c4716936aa6c9a808ee2e",
      "entity": {
        "tag": {
          "value": "person",
          "confidence": 0.9980469
        },
        "box": {
          "l": 0.3848941,
          "t": 0.28569314,
          "w": 0.092775516,
          "h": 0.3819766
        }
      },
      "extensions": {
        "footprintX": "inf",
        "centerGroundPointY": "0.0",
        "trackingId": "80eeb5bbb6f542b89b5b16a1e37c53c4",
        "footprintY": "inf",
        "centerGroundPointX": "0.0"
      }
    },
    {
      "type": "event",
      "inferenceId": "0b0de3d228b640488fc0624950a6c9e8",
      "relatedInferences": ["8724dff43d3c4716936aa6c9a808ee2e"],
      "event": {
        "name": "personZoneEnterExitEvent",
        "properties": {
          "trackingId": "80eeb5bbb6f542b89b5b16a1e37c53c4",
          "status": "Enter",
          "zone": "door"
        }
      }
    }
  ]
}
```

### <a name="more-operations"></a>Más operaciones:

<details>
  <summary>Hacer clic para expandir</summary>

### <a name="person-line-crossing"></a>Cruce de línea de personas

#### <a name="parameters"></a>Parámetros:

| Nombre                      | Tipo    | Descripción                                                                                                                                                                                                                                                                   |
| ------------------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| lines                     | list    | Lista de líneas.                                                                                                                                                                                                                                                                |
| name                      | string  | Nombre descriptivo de esta línea.                                                                                                                                                                                                                                                  |
| line                      | string  | Cada par de valores representa el punto inicial y final de la línea. Los valores flotantes representan la posición del vértice relativa a la esquina superior izquierda. Para calcular los valores x, y absolutos, debe multiplicar estos valores por el tamaño del fotograma.                            |
| outputFrequency           | int     | Velocidad a la que se generan los eventos. Cuando output_frequency = X, se genera cada evento X, por ejemplo. output_frequency = 2 significa que se genera cualquier otro evento. La frecuencia output_frequency se aplica tanto a eventos como a intervalos.                                                       |
| foco                     | string  | Ubicación del punto en el rectángulo delimitador de la persona usado para calcular los eventos. El valor del foco puede ser footprint (la superficie de la persona), bottom_center (la parte inferior central del rectángulo delimitador de la persona) o center (el centro del cuadro delimitador de la persona). El valor predeterminado es footprint. |
| threshold                 | FLOAT   | Los eventos se generan cuando la persona tiene un tamaño mayor que este número de píxeles dentro de la zona. El valor predeterminado es 16. Este es el valor recomendado para lograr la máxima precisión.                                                                                                |
| enableFaceMaskClassifier  | boolean | true para habilitar la detección de personas que lleven máscaras faciales en la secuencia de vídeo; false para deshabilitarla. De manera predeterminada, esta opción está deshabilitada. La detección de máscaras faciales requiere que el parámetro de ancho de vídeo de entrada sea 1920 "INPUT_VIDEO_WIDTH": 1920. No se devolverá el atributo de máscaras faciales.          |
| detectorNodeConfiguration | string  | Los parámetros DETECTOR_NODE_CONFIG para todas las operaciones de análisis espacial.                                                                                                                                                                                                      |

#### <a name="output"></a>Salida:

```json
{
  "timestamp": 145666620394490,
  "inferences": [
    {
      "type": "entity",
      "inferenceId": "2d3c7c7d6c0f4af7916eb50944523bdf",
      "entity": {
        "tag": {
          "value": "person",
          "confidence": 0.38330078
        },
        "box": {
          "l": 0.5316645,
          "t": 0.28169397,
          "w": 0.045862257,
          "h": 0.1594377
        }
      },
      "extensions": {
        "centerGroundPointX": "0.0",
        "centerGroundPointY": "0.0",
        "footprintX": "inf",
        "trackingId": "ac4a79a29a67402ba447b7da95907453",
        "footprintY": "inf"
      }
    },
    {
      "type": "event",
      "inferenceId": "2206088c80eb4990801f62c7050d142f",
      "relatedInferences": ["2d3c7c7d6c0f4af7916eb50944523bdf"],
      "event": {
        "name": "personLineEvent",
        "properties": {
          "trackingId": "ac4a79a29a67402ba447b7da95907453",
          "status": "CrossLeft",
          "zone": "door"
        }
      }
    }
  ]
}
```

### <a name="person-distance"></a>Distancia de la persona

#### <a name="parameters"></a>Parámetros:

| Nombre                      | Tipo    | Descripción                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ------------------------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| zones                     | list    | Lista de zonas.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| name                      | string  | Nombre descriptivo de esta zona.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| polygon                   | string  | Cada par de valores representa la x e y de los vértices del polígono. El polígono representa las áreas en las que se realiza el seguimiento o el recuento de personas. Los valores flotantes representan la posición del vértice relativa a la esquina superior izquierda. Para calcular los valores x, y absolutos, debe multiplicar estos valores por el tamaño del fotograma. Los eventos flotantes del umbral se generan cuando la persona tiene un tamaño mayor que este número de píxeles dentro de la zona. El valor predeterminado es 48 cuando el tipo es zonecrossing, y 16 cuando la hora es DwellTime. Estos son los valores recomendados para lograr la máxima precisión. |
| outputFrequency           | int     | Velocidad a la que se generan los eventos. Cuando output_frequency = X, se genera cada evento X, por ejemplo. output_frequency = 2 significa que se genera cualquier otro evento. La frecuencia output_frequency se aplica tanto a eventos como a intervalos.                                                                                                                                                                                                                                                                                                                                                     |
| foco                     | string  | Ubicación del punto en el rectángulo delimitador de la persona usado para calcular los eventos. El valor del foco puede ser footprint (la superficie de la persona), bottom_center (la parte inferior central del rectángulo delimitador de la persona) o center (el centro del cuadro delimitador de la persona). El valor predeterminado es footprint.                                                                                                                                                                                                                                                                                               |
| threshold                 | FLOAT   | Los eventos se generan cuando la persona tiene un tamaño mayor que este número de píxeles dentro de la zona.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| outputFrequency           | int     | Velocidad a la que se generan los eventos. Cuando output_frequency = X, se genera cada evento X, por ejemplo. output_frequency = 2 significa que se genera cualquier otro evento. La frecuencia output_frequency se aplica tanto a eventos como a intervalos.                                                                                                                                                                                                                                                                                                                                                     |
| minimumDistanceThreshold  | FLOAT   | Distancia en pies que desencadenará un evento "TooClose" cuando las personas estén más cerca de esa distancia entre sí.                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| maximumDistanceThreshold  | FLOAT   | Distancia en pies que desencadenará un evento "TooFar" cuando las personas estén más lejos de esa distancia entre sí.                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| aggregationMethod         | string  | Método para agregar el resultado de persondistance. AggregationMethod es aplicable tanto al modo como al promedio.                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| enableFaceMaskClassifier  | boolean | true para habilitar la detección de personas que lleven máscaras faciales en la secuencia de vídeo; false para deshabilitarla. De manera predeterminada, esta opción está deshabilitada. La detección de máscaras faciales requiere que el parámetro de ancho de vídeo de entrada sea 1920 "INPUT_VIDEO_WIDTH": 1920. No se devolverá el atributo de máscaras faciales.                                                                                                                                                                                                                                                                                                        |
| detectorNodeConfiguration | string  | Los parámetros DETECTOR_NODE_CONFIG para todas las operaciones de análisis espacial.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |

#### <a name="output"></a>Salida:

```json
{
  "timestamp": 145666613610297,
  "inferences": [
    {
      "type": "event",
      "inferenceId": "85a5fc4936294a3bac90b9c43876741a",
      "event": {
        "name": "personDistanceEvent",
        "properties": {
          "maximumDistanceThreshold": "14.5",
          "personCount": "0.0",
          "eventName": "Unknown",
          "zone": "door",
          "averageDistance": "0.0",
          "minimumDistanceThreshold": "1.5",
          "distanceViolationPersonCount": "0.0"
        }
      }
    }
  ]
}
```

### <a name="person-count"></a>Recuento de personas

#### <a name="parameters"></a>Parámetros:

| Nombre                      | Tipo    | Descripción                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ------------------------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| zones                     | list    | Lista de zonas.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| name                      | string  | Nombre descriptivo de esta zona.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| polygon                   | string  | Cada par de valores representa la x e y de los vértices del polígono. El polígono representa las áreas en las que se realiza el seguimiento o el recuento de personas. Los valores flotantes representan la posición del vértice relativa a la esquina superior izquierda. Para calcular los valores x, y absolutos, debe multiplicar estos valores por el tamaño del fotograma. Los eventos flotantes del umbral se generan cuando la persona tiene un tamaño mayor que este número de píxeles dentro de la zona. El valor predeterminado es 48 cuando el tipo es zonecrossing, y 16 cuando la hora es DwellTime. Estos son los valores recomendados para lograr la máxima precisión. |
| outputFrequency           | int     | Velocidad a la que se generan los eventos. Cuando output_frequency = X, se genera cada evento X, por ejemplo. output_frequency = 2 significa que se genera cualquier otro evento. La frecuencia output_frequency se aplica tanto a eventos como a intervalos.                                                                                                                                                                                                                                                                                                                                                     |
| desencadenador                   | string  | Tipo de desencadenador para enviar un evento. Los valores admitidos son event para enviar eventos cuando el recuento cambia o interval para enviar eventos de manera periódica, independientemente de si el recuento ha cambiado o no.                                                                                                                                                                                                                                                                                                                                                           |
| foco                     | string  | Ubicación del punto en el rectángulo delimitador de la persona usado para calcular los eventos. El valor del foco puede ser footprint (la superficie de la persona), bottom_center (la parte inferior central del rectángulo delimitador de la persona) o center (el centro del cuadro delimitador de la persona). El valor predeterminado es footprint.                                                                                                                                                                                                                                                                                               |
| threshold                 | FLOAT   | Los eventos se generan cuando la persona tiene un tamaño mayor que este número de píxeles dentro de la zona.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| enableFaceMaskClassifier  | boolean | true para habilitar la detección de personas que lleven máscaras faciales en la secuencia de vídeo; false para deshabilitarla. De manera predeterminada, esta opción está deshabilitada. La detección de máscaras faciales requiere que el parámetro de ancho de vídeo de entrada sea 1920 "INPUT_VIDEO_WIDTH": 1920. No se devolverá el atributo de máscaras faciales.                                                                                                                                                                                                                                                                                                        |
| detectorNodeConfiguration | string  | Los parámetros DETECTOR_NODE_CONFIG para todas las operaciones de análisis espacial.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |

#### <a name="output"></a>Salida:

```json
{
  "timestamp": 145666599533564,
  "inferences": [
    {
      "type": "entity",
      "inferenceId": "5b8076753b8c47bba8c72a7e0f7c5cc0",
      "entity": {
        "tag": {
          "value": "person",
          "confidence": 0.9458008
        },
        "box": {
          "l": 0.474487,
          "t": 0.26522297,
          "w": 0.066929355,
          "h": 0.2828749
        }
      },
      "extensions": {
        "centerGroundPointX": "0.0",
        "centerGroundPointY": "0.0",
        "footprintX": "inf",
        "footprintY": "inf"
      }
    },
    {
      "type": "event",
      "inferenceId": "fb309c9285f94f268378540b5fbbf5ad",
      "relatedInferences": ["5b8076753b8c47bba8c72a7e0f7c5cc0"],
      "event": {
        "name": "personCountEvent",
        "properties": {
          "personCount": "1.0",
          "zone": "demo"
        }
      }
    }
  ]
}
```

### <a name="custom"></a>Personalizado

#### <a name="parameters"></a>Parámetros:

| Nombre                   | Tipo   | Descripción                           |
| ---------------------- | ------ | ------------------------------------- |
| extensionConfiguration | string | Representación JSON de la operación. |

#### <a name="output"></a>Salida:

```json
{
  "timestamp": 145666599533564,
  "inferences": [
    {
      "type": "entity",
      "inferenceId": "5b8076753b8c47bba8c72a7e0f7c5cc0",
      "entity": {
        "tag": {
          "value": "person",
          "confidence": 0.9458008
        },
        "box": {
          "l": 0.474487,
          "t": 0.26522297,
          "w": 0.066929355,
          "h": 0.2828749
        }
      },
      "extensions": {
        "centerGroundPointX": "0.0",
        "centerGroundPointY": "0.0",
        "footprintX": "inf",
        "footprintY": "inf"
      }
    },
    {
      "type": "event",
      "inferenceId": "fb309c9285f94f268378540b5fbbf5ad",
      "relatedInferences": ["5b8076753b8c47bba8c72a7e0f7c5cc0"],
      "event": {
        "name": "personCountEvent",
        "properties": {
          "personCount": "1.0",
          "zone": "demo"
        }
      }
    }
  ]
}
```

</details>

## <a name="video-player"></a>Reproductor de vídeo

Puede usar un reproductor de vídeo para ver el vídeo generado, incluidas las inferencias (rectángulos delimitadores), como se muestra a continuación:

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/spatial-analysis/inference.png" alt-text="Rectángulos delimitadores":::

## <a name="troubleshooting"></a>Solución de problemas

Spatialanalysis es un contenedor grande y su tiempo de inicio puede ser de hasta 30 segundos. Una vez que el contenedor spatialanalysis esté en funcionamiento, comenzará a enviar los eventos de inferencias.

```JSON
[IoTHubMonitor] [3:37:28 PM] Message received from [ase03-edge/avaedge]:
{
  "sdp&quot;: &quot;SDP:\nv=0\r\no=- 1620671848135494 1 IN IP4 172.27.86.122\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/cafeteria.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.08.19\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.066\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/cafeteria.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=640028;sprop-parameter-sets=Z2QAKKzZQHgCHoQAAAMABAAAAwDwPGDGWA==,aOvssiw=\r\na=control:track1\r\n"
}
[IoTHubMonitor] [3:37:30 PM] Message received from [ase03-edge/avaedge]:
{
  "type": "video",
  "location": "/videos/customoperation-05102021",
  "startTime&quot;: &quot;2021-05-10T18:37:27.931Z"
}
[IoTHubMonitor] [3:37:40 PM] Message received from [ase03-edge/avaedge]:
{
  "state&quot;: &quot;initializing"
}
[IoTHubMonitor] [3:37:50 PM] Message received from [ase03-edge/avaedge]:
{
  "state&quot;: &quot;initializing"
}
[IoTHubMonitor] [3:38:18 PM] Message received from [ase03-edge/avaedge]:
{
  "type": "video",
  "location": "/videos/customoperation-05102021",
  "startTime&quot;: &quot;2021-05-10T18:37:27.931Z"
}
[IoTHubMonitor] [3:38:42 PM] Message received from [ase03-edge/avaedge]:
{
  "timestamp": 145860472980260,
  "inferences": [
    {
      "type": "entity",
      "inferenceId": "647aacf9d8bc47078a1ed31d1c459c24",
      "entity": {
        "tag": {
          "value": "person",
          "confidence": 0.7583008
        },
        "box": {
          "l": 0.48213565,
          "t": 0.21217245,
          "w": 0.056364775,
          "h": 0.29961595
        }
      },
      "extensions": {
        "centerGroundPointY": "0.0",
        "centerGroundPointX": "0.0",
        "footprintX": "0.5087100982666015",
        "footprintY&quot;: &quot;0.49634415356080924"
      }
    },
    {
      "type": "event",
      "inferenceId": "dae6c2b742634196b615c128654845dc",
      "relatedInferences": [
        "647aacf9d8bc47078a1ed31d1c459c24"
      ],
      "event": {
        "name": "personCountEvent",
        "properties": {
          "personCount": "1.0",
          "zone&quot;: &quot;stairlanding"
        }
      }
    }
  ]
}

```

> [!NOTE]
> Es posible que vea mensajes de **"inicializando"** . Estos mensajes se muestran mientras se inicia el módulo spatialAnalysis y pueden tardar hasta 60 segundos en llegar a un estado en ejecución. Tenga paciencia y debería ver el flujo de eventos de inferencia.

## <a name="next-steps"></a>Pasos siguientes

Pruebe las distintas operaciones `spatialAnalysis` que ofrece el módulo; consulte las siguientes topologías pipelineTopologies:

- [personCount](https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/spatial-analysis/person-count-operation-topology.json)
- [personDistance](https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/spatial-analysis/person-distance-pperation-topology.json)
- [personCrossingLine](https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/spatial-analysis/person-line-crossing-operation-topology.json)
- [personZoneCrossing](https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/spatial-analysis/person-zone-crossing-operation-topology.json)
- [customOperation](https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/spatial-analysis/custom-operation-topology.json)

> [!Tip]
> Use un **[archivo de vídeo de ejemplo](https://lvamedia.blob.core.windows.net/public/2018-03-07.16-50-00.16-55-00.school.G421.mkv)** que tenga más de una persona en el fotograma.
