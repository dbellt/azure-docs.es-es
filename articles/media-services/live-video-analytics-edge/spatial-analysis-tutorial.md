---
title: 'Análisis de vídeo en directo con Computer Vision para análisis espacial con Live Video Analytics: Azure'
description: En este tutorial se muestra cómo usar Azure Live Video Analytics junto con la característica de IA de análisis espacial de Computer Vision, parte de Azure Cognitive Services, para analizar una fuente de vídeo en directo desde una cámara IP (simulada).
ms.topic: tutorial
ms.date: 09/08/2020
ms.openlocfilehash: 549c1405e7deb3531da65ff4425befa2d29aaeef
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110373570"
---
# <a name="analyze-live-video-with-computer-vision-for-spatial-analysis-and-live-video-analytics-preview"></a>Análisis de vídeo en directo con Computer Vision para análisis espacial y Live Video Analytics (versión preliminar)

[!INCLUDE [redirect to Azure Video Analyzer](./includes/redirect-video-analyzer.md)]

En este tutorial se muestra cómo usar Live Video Analytics junto con el [servicio de IA de Computer Vision para análisis espacial de Azure Cognitive Services](https://azure.microsoft.com/services/cognitive-services/computer-vision/) para analizar una fuente de vídeo en directo desde una cámara IP (simulada). Verá cómo este servidor de inferencia le permite analizar el vídeo en streaming para comprender las relaciones espaciales entre las personas y el movimiento en el espacio físico.  Un subconjunto de los fotogramas de la fuente de vídeo se envía a este servidor de inferencia y los resultados se envían al centro de IoT Edge. Cuando se cumplen algunas condiciones, los clips de vídeo se graban y almacenan como recursos de Azure Media Services.

En este tutorial, aprenderá lo siguiente:

> [!div class="checklist"]
> * Configuración de recursos.
> * Examine el código.
> * Ejecución del código de ejemplo
> * Supervisión de eventos.
 
[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]
  > [!NOTE]
  > Necesitará una suscripción de Azure con permisos para crear entidades de servicio (el **rol de propietario** permite esto). Si no tiene los permisos adecuados, póngase en contacto con el administrador de la cuenta para que se los conceda. 
## <a name="suggested-pre-reading"></a>Sugerencias antes de la lectura

Consulte estos artículos antes de empezar:

* [Introducción a Análisis de vídeos en vivo en IoT Edge](overview.md)
* [Terminología de Live Video Analytics en IoT Edge](terminology.md)
* [Concepto de grafo multimedia](media-graph-concept.md)
* [Grabación de vídeo basada en eventos](event-based-video-recording-concept.md)
* [Tutorial: Desarrollo de un módulo IoT Edge](../../iot-edge/tutorial-develop-for-linux.md)
* [Implementación de Live Video Analytics en Azure Stack Edge](deploy-azure-stack-edge-how-to.md) 

## <a name="prerequisites"></a>Requisitos previos

Estos son los requisitos previos para conectar el módulo de análisis espacial a un módulo de Live Video Analytics.

* [Visual Studio Code](https://code.visualstudio.com/) en la máquina de desarrollo. Asegúrese de tener la [extensión Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
    * Asegúrese de que la red a la que está conectada la máquina de desarrollo permita Advanced Message Queueing Protocol a través del puerto 5671. Esta configuración permite a Azure IoT Tools comunicarse con Azure IoT Hub.
* [Azure Stack Edge](https://azure.microsoft.com/products/azure-stack/edge/) con aceleración de GPU.  
    Se recomienda usar Azure Stack Edge con aceleración de GPU; sin embargo, el contenedor funciona en cualquier otro dispositivo con una [GPU NVIDIA Tesla T4](https://www.nvidia.com/en-us/data-center/tesla-t4/). 
* [Contenedor de Computer Vision de Azure Cognitive Services](https://azure.microsoft.com/services/cognitive-services/computer-vision/) para análisis espacial.  
    Para usar este contenedor, debe tener un recurso de Computer Vision para obtener la **clave de API** y el **URI de punto de conexión** asociados. La clave está disponible en las páginas Claves e Información general de Computer Vision en Azure Portal. La clave y el punto de conexión son necesarios para iniciar el contenedor.

## <a name="overview"></a>Información general

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/spatial-analysis-tutorial/overview.png" alt-text="Introducción al análisis espacial":::
 
En este diagrama se muestra el flujo de las señales en este tutorial. Un [módulo perimetral](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555) simula una cámara IP que hospeda un servidor de protocolo RTSP. Un nodo de [origen RTSP](media-graph-concept.md#rtsp-source) extrae la fuente de vídeo de este servidor y envía fotogramas de vídeo al nodo del procesador `MediaGraphCognitiveServicesVisionExtension`.

El nodo MediaGraphCognitiveServicesVisionExtension desempeña el rol de un proxy. Convierte los fotogramas de vídeo en el tipo de imagen especificado. Luego, utiliza la **memoria compartida** para retransmitir la imagen a otro módulo perimetral que ejecuta operaciones de IA detrás de un punto de conexión gRPC. En este ejemplo, ese módulo perimetral es el módulo de análisis espacial. El nodo de procesador MediaGraphCognitiveServicesVisionExtension realiza dos tareas:

* Recopila los resultados y publica eventos en el nodo de [receptor de IoT Hub](media-graph-concept.md#iot-hub-message-sink). que posteriormente los envía a [IoT Edge Hub](../../iot-fundamentals/iot-glossary.md#iot-edge-hub). 
* También captura un clip de vídeo de 30 segundos del origen RTSP mediante un [procesador de puerta de señal](media-graph-concept.md#signal-gate-processor) y lo almacena como un recurso de Media Services.

## <a name="create-the-computer-vision-resource"></a>Creación del recurso de Computer Vision

Es necesario crear un recurso de Azure de tipo Computer Vision en [Azure Portal](../../iot-edge/how-to-deploy-modules-portal.md) o mediante la CLI de Azure. El recurso se puede crear una vez que se haya aprobado la solicitud de acceso al contenedor y se haya registrado su identificador de suscripción de Azure. Vaya a https://aka.ms/csgate para enviar el caso de uso y el identificador de suscripción de Azure.  El recurso de Azure se debe crear con la misma suscripción de Azure que se ha proporcionado en el formulario de solicitud de acceso.

### <a name="gathering-required-parameters"></a>Recopilación de los parámetros obligatorios

Hay tres parámetros principales que son necesarios para todos los contenedores de Cognitive Services, incluido el contenedor de análisis espacial. El contrato de licencia para el usuario final (CLUF) se debe haber aceptado. Además, se necesitan un identificador URI de punto de conexión y una clave de API.

### <a name="keys-and-endpoint-uri"></a>Claves e identificador URI de punto de conexión

Se usa una clave para iniciar el contenedor de análisis espacial y dicha clave está disponible en la página `Keys and Endpoint` de Azure Portal del recurso de Cognitive Services correspondiente. Vaya a esa página y busque el identificador URI del punto de conexión.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/spatial-analysis-tutorial/keys-endpoint.png" alt-text="URI de punto de conexión":::

## <a name="set-up-azure-stack-edge"></a>Configuración de Azure Stack Edge

Siga [estos pasos](../../databox-online/azure-stack-edge-gpu-deploy-prep.md) para configurar Azure Stack Edge y continúe con los pasos que se indican a continuación para implementar Live Video Analytics en los módulos de análisis espacial.

## <a name="set-up-your-development-environment"></a>Configurado su entorno de desarrollo

1. Clone el repositorio desde esta ubicación: https://github.com/Azure-Samples/live-video-analytics-iot-edge-csharp.
1. En Visual Studio Code, abra la carpeta en que se ha descargado.
1. En Visual Studio Code, vaya a la carpeta src/cloud-to-device-console-app/operations.json. Allí, cree un archivo y asígnele el nombre appsettings.json. Este archivo contendrá la configuración necesaria para ejecutar el programa.
1. Para obtener IotHubConnectionString desde Azure Stack Edge, siga estos pasos:

    * Vaya a su centro de IoT en Azure Portal y haga clic en `Shared access policies` en el panel de navegación izquierdo.
    * Haga clic en `iothubowner` para obtener las claves de acceso compartido.
    * Copie el valor de `Connection String – primary key` y péguelo en el cuadro de entrada de VSCode.
    
        La cadena de conexión tendrá el siguiente aspecto: <br/>`HostName=xxx.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=xxx`
1. Copie el contenido siguiente en el archivo. Asegúrese de reemplazar las variables.
    
    ```json
    {
        "IoThubConnectionString" : "HostName=<IoTHubName>.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=<SharedAccessKey>",
        "deviceId" : "<your Azure Stack Edge name>",
        "moduleId" : "lvaEdge"
    } 
    ```
1. Vaya a la carpeta src/edge y cree un archivo llamado .env.
1. Copie el contenido del archivo /clouddrive/lva-sample/edge-deployment/.env. El texto debería ser similar al siguiente código.

    ```env
    SUBSCRIPTION_ID="<Subscription ID>"  
    RESOURCE_GROUP="<Resource Group>"  
    AMS_ACCOUNT="<AMS Account ID>"  
    IOTHUB_CONNECTION_STRING="HostName=xxx.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=xxx"  
    AAD_TENANT_ID="<AAD Tenant ID>"  
    AAD_SERVICE_PRINCIPAL_ID="<AAD SERVICE_PRINCIPAL ID>"  
    AAD_SERVICE_PRINCIPAL_SECRET="<AAD SERVICE_PRINCIPAL ID>"  
    VIDEO_INPUT_FOLDER_ON_DEVICE="/home/lvaedgeuser/samples/input"  
    VIDEO_OUTPUT_FOLDER_ON_DEVICE="/var/media"
    APPDATA_FOLDER_ON_DEVICE="/var/local/mediaservices"
    CONTAINER_REGISTRY_USERNAME_myacr="<your container registry username>"  
    CONTAINER_REGISTRY_PASSWORD_myacr="<your container registry password>"   
    ```
    
## <a name="set-up-deployment-template"></a>Configuración de la plantilla de implementación  

Busque el archivo de implementación en /src/edge/deployment.spatialAnalysis.template.json. En la plantilla, está el módulo lvaEdge, el módulo rtspsim y nuestro módulo spatial-analysis.

Hay algunas cosas a las que debe prestar atención en el archivo de la plantilla de implementación:

1. Establezca el enlace de puerto.
    
    ```
    "PortBindings": {
        "50051/tcp": [
            {
                "HostPort": "50051"
            }
        ]
    },
    ```
1. `IpcMode` en createOptions de los módulos lvaEdge y de análisis espacial debe ser igual y estar establecido en host.
1. Para que el simulador RTSP funcione, asegúrese de que ha configurado los límites del volumen. Para más información, consulte [Configuración de montajes de volúmenes de Docker](deploy-azure-stack-edge-how-to.md#optional-setup-docker-volume-mounts).

    1. [Conéctese al recurso compartido de SMB](../../databox-online/azure-stack-edge-deploy-add-shares.md#connect-to-an-smb-share) y copie el [archivo de vídeo con la excavadora de ejemplo](https://lvamedia.blob.core.windows.net/public/bulldozer.mkv) en el recurso compartido local.  
        > [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4Mesi]  
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
1. Al lado del panel AZURE IOT HUB, seleccione el icono Más acciones para establecer la cadena de conexión de IoT Hub. La cadena se puede copiar del archivo `src/cloud-to-device-console-app/appsettings.json`.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/spatial-analysis-tutorial/connection-string.png" alt-text="Análisis espacial: cadena de conexión":::
1. Haga clic con el botón derecho en `src/edge/deployment.spatialAnalysis.template.json` y seleccione Generar manifiesto de implementación de IoT Edge.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/spatial-analysis-tutorial/deployment-template-json.png" alt-text="Análisis espacial: deployment amd64 json":::
    
    Esta acción debe crear un archivo de manifiesto llamado deployment.amd64.json en la carpeta src/edge/config llamado.
1. Haga clic con el botón derecho en `src/edge/config/deployment.spatialAnalysis.amd64.json`, seleccione Create Deployment for Single Device (Crear una implementación para un dispositivo individual) y, a continuación, seleccione el nombre del dispositivo perimetral.
    
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/spatial-analysis-tutorial/deployment-amd64-json.png" alt-text="Análisis espacial: deployment template json":::   
1. Cuando se le pida que seleccione un dispositivo IoT Hub, elija el nombre de su instancia de Azure Stack Edge en el menú desplegable.
1. Tras aproximadamente 30 segundos, en la esquina inferior izquierda de la ventana, actualice Azure IoT Hub. El dispositivo perimetral ahora muestra los siguientes módulos implementados:
    
    * Live Video Analytics en IoT Edge (nombre de módulo lvaEdge).
    * Simulador del Protocolo de transmisión en tiempo real (RTSP) (nombre del módulo rtspsim).
    * Análisis espacial (nombre del módulo spatialAnalysis).
    
Si la implementación se realiza correctamente, aparecerá un mensaje en SALIDA similar al siguiente:

```
[Edge] Start deployment to device [<Azure Stack Edge name>]
[Edge] Deployment succeeded.
```

Después, puede encontrar los módulos `lvaEdge`, `rtspsim`, `spatialAnalysis` y `rtspsim` en Dispositivos/Módulos, y su estado debe ser "en ejecución".

## <a name="prepare-to-monitor-events"></a>Preparación para supervisar eventos

Para ver estos eventos, siga estos pasos:

1. En Visual Studio Code, abra la pestaña **Extensiones** (o presione Ctrl + Mayús + X) y busque Azure IoT Hub.
1. Haga clic con el botón derecho y seleccione la opción **Configuración de la extensión**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/extensions-tab.png" alt-text="Configuración de la extensión":::
1. Busque y habilite "Show Verbose Message" (Mostrar mensaje detallado).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/show-verbose-message.png" alt-text="Show Verbose Message"::: (Mostrar mensaje detallado)
1. Abra el panel del explorador y busque Azure IoT Hub en la esquina inferior izquierda.
1. Expanda el nodo Devices (Dispositivos).
1. Haga clic con el botón derecho en su instancia de Azure Stack Edge y seleccione Iniciar la supervisión del punto de conexión de eventos integrado.
    
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/spatial-analysis-tutorial/start-monitoring.png" alt-text="Análisis espacial: iniciar supervisión":::
     
## <a name="run-the-program"></a>Ejecución del programa

Un archivo program.cs invocará los métodos directos en src/cloud-to-device-console-app/operations.json. Es necesario configurar operations.json y proporcionar una topología para el uso de grafos de elementos multimedia.  

En operations.json:

* Establezca la topología de la siguiente manera:

```json
{
    "opName": "GraphTopologySet",
    "opParams": {
        "topologyUrl": "https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/lva-spatial-analysis/2.0/topology.json"
    }
},
```

* Cree una instancia de grafo como esta y establezca los parámetros de la topología aquí:

```json
{
    "opName": "GraphInstanceSet",
    "opParams": {
        "name": "Sample-Graph-1",
        "properties": {
            "topologyName": "InferencingWithCVExtension",
            "description": "Sample graph description",
            "parameters": [
                {
                    "name": "rtspUrl",
                    "value": " rtsp://rtspsim:554/media/bulldozer.mkv"
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

>[!Note]
Consulte el uso de MediaGraphRealTimeComputerVisionExtension para conectarse con el módulo de análisis espacial. Establezca ${grpcUrl} to **tcp://spatialAnalysis:<NúmeroDePuerto>** , por ejemplo, tcp://spatialAnalysis:50051

```json
{
    "@type": "#Microsoft.Media.MediaGraphCognitiveServicesVisionExtension",
    "name": "computerVisionExtension",
    "endpoint": {
        "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
        "url": "${grpcUrl}",
        "credentials": {
            "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
            "username": "${spatialanalysisusername}",
            "password": "${spatialanalysispassword}"
        }
    },
    "image": {
        "scale": {
            "mode": "pad",
            "width": "1408",
            "height": "786"
        },
        "format": {
            "@type": "#Microsoft.Media.MediaGraphImageFormatRaw",
            "pixelFormat": "bgr24"
        }
    },
    "samplingOptions": {
        "skipSamplesWithoutAnnotation": "false",
        "maximumSamplesPerSecond": "20"
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
    ]
}
```

Ejecute una sesión de depuración y siga las instrucciones de **TERMINAL**; se establecerá la topología, se establecerá la instancia del grafo, se activará la instancia del grafo y, por último, se eliminarán los recursos.

## <a name="interpret-results"></a>Interpretación de los resultados

Cuando se cree una instancia de un grafo de elementos multimedia, verá el evento "MediaSessionEstablished"; este es un [evento MediaSessionEstablished de ejemplo](detect-motion-emit-events-quickstart.md#mediasessionestablished-event).

El módulo de análisis espacial también enviará eventos de información de inteligencia artificial a Live Video Analytics y, luego, a IoTHub; también se mostrarán en **SALIDA**. ENTITY son los objetos de detección y EVENT es el evento spaceanalytics. Esta salida se pasará a Live Video Analytics.

Salida de ejemplo de personZoneEvent (de la operación cognitiveservices.vision.spatialanalysis-personcrossingpolygon.livevideoanalytics):

```
{
  "body": {
    "timestamp": 143810610210090,
    "inferences": [
      {
        "type": "entity",
        "subtype": "",
        "inferenceId": "a895c86824db41a898f776da1876d230",
        "relatedInferences": [],
        "entity": {
          "tag": {
            "value": "person",
            "confidence": 0.66026187
          },
          "attributes": [],
          "box": {
            "l": 0.26559368,
            "t": 0.17887735,
            "w": 0.49247605,
            "h": 0.76629657
          }
        },
        "extensions": {},
        "valueCase": "entity"
      },
      {
        "type": "event",
        "subtype": "",
        "inferenceId": "995fe4206847467f8dfde83a15187d76",
        "relatedInferences": [
          "a895c86824db41a898f776da1876d230"
        ],
        "event": {
          "name": "personZoneEvent",
          "properties": {
            "status": "Enter",
            "metadataVersion": "1.0",
            "zone": "polygon0",
            "trackingId": "a895c86824db41a898f776da1876d230"
          }
        },
        "extensions": {},
        "valueCase": "event"
      }
    ]
  },
  "applicationProperties": {
    "topic": "/subscriptions/81631a63-f0bd-4f35-8344-5bc541b36bfc/resourceGroups/lva-sample-resources/providers/microsoft.media/mediaservices/lvasamplea4bylcwoqqypi",
    "subject": "/graphInstances/Sample-Graph-1/processors/spatialanalysisExtension",
    "eventType": "Microsoft.Media.Graph.Analytics.Inference",
    "eventTime": "2020-08-20T03:54:29.001Z",
    "dataVersion": "1.0"
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

Pruebe las distintas operaciones que ofrece el módulo `spatialAnalysis`, como **personCount** y **personDistance** alternando la marca "habilitado" en el nodo del grafo del archivo de manifiesto de implementación.
>[!Tip]
> Use un [archivo de vídeo de ejemplo](https://lvamedia.blob.core.windows.net/public/2018-03-07.16-50-00.16-55-00.school.G421.mkv) que tenga más de una persona en el fotograma.

> [!NOTE]
> Solo puede ejecutar una operación cada vez. Por lo tanto, asegúrese de que solo una marca esté establecida en **true** y que las otras estén establecidas en **false**.
