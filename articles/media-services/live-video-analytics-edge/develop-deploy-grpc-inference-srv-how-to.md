---
title: 'Desarrollo e implementación de un servidor de inferencia de gRPC con Live Video Analytics: Azure'
description: En este artículo se proporciona una guía sobre cómo desarrollar e implementar un servidor de inferencia de gRPC con Live Video Analytics.
ms.topic: how-to
ms.date: 12/02/2020
ms.openlocfilehash: d7d636803ac49d78aae78dad256598d9de3e7448
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110371580"
---
# <a name="develop-and-deploy-a-grpc-inference-server-with-live-video-analytics"></a>Desarrollo e implementación de un servidor de inferencia de gRPC con Live Video Analytics

[!INCLUDE [redirect to Azure Video Analyzer](./includes/redirect-video-analyzer.md)]

## <a name="overview"></a>Información general

En este artículo se muestra cómo puede encapsular los modelos de IA que prefiera en un servidor de inferencia de gRPC, de modo que pueda integrarse con Live Video Analytics (LVA) a través de la extensión de grafos. 

## <a name="suggested-pre-reading"></a>Sugerencias antes de la lectura

* [Extensiones de grafos multimedia](media-graph-extension-concept.md)
* [Protocolo de la extensión gRPC](grpc-extension-protocol.md)
* [Esquema de metadatos de inferencia](inference-metadata-schema.md)
* [Introducción a gRPC](https://www.grpc.io/docs/what-is-grpc/introduction/)
* [Guía del lenguaje proto3](https://developers.google.com/protocol-buffers/docs/proto3)

## <a name="prerequisites"></a>Requisitos previos

* Un dispositivo x86-64 o ARM64 que ejecute uno de los [sistemas operativos Linux admitidos](../../iot-edge/support.md#operating-systems) o una máquina Windows.
* [Instalar Docker](https://docs.docker.com/desktop/#download-and-install) en la máquina.
* Instalar un [entorno de ejecución de Azure IoT Edge](../../iot-edge/how-to-install-iot-edge.md?tabs=linux).

## <a name="grpc-implementation-steps"></a>Pasos de implementación de gRPC

Para crear un servidor de inferencia de gRPC e implementarlo como una extensión con Live Video Analytics, se usarán los pasos siguientes:

### <a name="setup"></a>Configuración:

Siga los pasos necesarios para [implementar y ejecutar el módulo de Live Video Analytics en un dispositivo IoT Edge](deploy-iot-edge-device.md).

### <a name="high-level-implementation-steps"></a>Pasos de implementación de nivel alto:

1. Elija uno de los muchos lenguajes compatibles con gRPC: C#, C++, Dart, Go, Java, Node, Objective-C, PHP, Python, Ruby.
1. Implemente un servidor de gRPC que se comunicará con Live Video Analytics mediante [los archivos de proto3](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc).

    :::image type="content" source="./media/develop-deploy-grpc-inference-srv-how-to/inference-srv-container-process.png" alt-text="Servidor de gRPC que se comunicará con Live Video Analytics mediante los archivos de proto3":::

    Dentro de este servicio podrá hacer lo siguiente:
    1. Controlar el intercambio de los mensajes de descripción de la sesión entre el servidor y el cliente.
    1. Controlar los [mensajes MediaSample](https://github.com/Azure/live-video-analytics/blob/master/contracts/grpc/extension.proto) y obtener resultados.

        1. Invocar el motor de inferencia que usa un modelo entrenado para realizar inferencias en los mensajes entrantes.
        1. Reciba los resultados de la inferencia del motor, vuelva a empaquetarlos como MediaSample y envíelos de nuevo a Live Video Analytics mediante el archivo [inferencing.proto](https://github.com/Azure/live-video-analytics/blob/master/contracts/grpc/inferencing.proto).

            También puede invocar cualquier función de transformación de elementos multimedia en el archivo MediaSample.
1. Implementar la implementación del servidor de gRPC. Existen dos formas de hacerlo:

    1. Implementarlo como módulo de IoT colocalizado con el módulo de Live Video Analytics.
    1. Implementarlo como módulo de IoT en un nodo accesible desde la red (ya sea en el entorno local o en la nube) que puede intercambiar datos con el módulo de Live Video Analytics.
1. Configurar una topología de grafos de Live Video Analytics con el módulo de Live Video Analytics y apuntarla al servidor de gRPC.

### <a name="recommendation"></a>Recomendación:

Cuando se coloca en el mismo nodo, la memoria compartida se puede usar para obtener el mejor rendimiento. Esto requiere usar las funcionalidades de memoria compartida de Linux expuestas por el entorno o lenguaje de programación.

1. Abra el identificador de memoria compartida de Linux.
1. Tras la recepción de un fotograma, acceda al desplazamiento de dirección dentro de la memoria compartida.
1. Confirme que el procesamiento del fotograma se realizó correctamente para que Live Video Analytics pueda reclamar su memoria.

## <a name="create-a-grpc-inference-server"></a>Creación de un servidor de inferencia de gRPC

Ahora creará un módulo de IoT Edge (IA externa) que acepta fotogramas de vídeo de Live Video Analytics mediante mensajes [protobuf](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc) a través de la memoria compartida, clasificará los fotogramas como "oscuros" o "claros" y devolverá los resultados de la inferencia al receptor de mensajes de IoT Hub en Live Video Analytics mediante el [esquema de metadatos de inferencia](inference-metadata-schema.md).

:::image type="content" source="./media/develop-deploy-grpc-inference-srv-how-to/external-ai.png" alt-text="Creación de un módulo de IoT Edge (IA externa)":::

Este servidor de inferencia de gRPC es una aplicación de la consola de .NET Core que se basa en los mensajes [protobuf](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc) enviados entre Live Video Analytics y su IA personalizada. A continuación, se muestra el flujo de los mensajes entre Live Video Analytics y el servidor de inferencia de gRPC:

1. Live Video Analytics envía un descriptor de flujo multimedia (consulte [extension.proto](https://github.com/Azure/live-video-analytics/blob/master/contracts/grpc/extension.proto)) que define la información del flujo multimedia que se enviará seguida de fotogramas de vídeo al servidor como un mensaje [protobuf](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc) a través de la sesión de flujo de gRPC. 
1. El servidor valida y confirma el descriptor de flujo y configura el método de transferencia de datos deseado.
1. Luego, Live Video Analytics empieza a enviar los archivos MediaSample que contienen los fotogramas de vídeo.
1. El servidor analiza los fotogramas de vídeo apenas los recibe y comienza a procesarlos con un procesador de imágenes definido por el usuario.
1. A continuación, el servidor devuelve los resultados de la inferencia como mensajes [protobuf](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc) en cuanto están disponibles. 

    :::image type="content" source="./media/develop-deploy-grpc-inference-srv-how-to/grpc-external-srv.png" alt-text="Creación de un servidor de inferencia de gRPC":::

Los fotogramas de vídeo se pueden transferir a través de la [memoria compartida](https://en.wikipedia.org/wiki/Shared_memory#:~:text=In%20computer%20science%2C%20shared%20memory,of%20passing%20data%20between%20programs.) o se pueden insertar dentro del mensaje protobuf. El modo de transferencia de datos se puede configurar en la topología de grafos de LVA para determinar cómo se transferirán los fotogramas. Para ello, configure el elemento **dataTransfer** de la propiedad MediaGraphGrpcExtension tal como se muestra a continuación:

Insertado:

```
"dataTransfer": {
              "mode": "Embedded"
            }
```

Memoria compartida:

```
"dataTransfer": {
              "mode": "sharedMemory",
              "SharedMemorySizeMiB": "20"
            }
```

> [!NOTE]
> Al comunicarse a través de la memoria compartida, el valor de IpcMode se debe establecer en **shareable** (compartible) y, en el módulo del servidor de gRPC, establezca el valor de IpcMode en **container:{CONTAINER_NAME}** (contenedor:{NOMBRE_DE_CONTENEDOR}). Esta configuración se hará en el archivo de manifiesto de implementación que se usa para implementar los módulos en Azure IoT Hub. A continuación, se muestra un ejemplo de las opciones de contenedor que se van a usar al configurar los módulos de IoT Edge.

Módulo de Live Video Analytics:

```
{
    "HostConfig": {
        "LogConfig": {
            "Config": {
                "max-size": "10m",
                "max-file": "10"
            }
        },
        "IpcMode": "shareable"
    }
}
```

Módulo de extensión de gRPC:

```
{
    "HostConfig": {
        "LogConfig": {
            "Config": {
                "max-size": "10m",
                "max-file": "10"
            }
        },
        "IpcMode": "container:lvaEdge"
    }
}
```

> [!NOTE]
> Asegúrese de tener acceso al área de memoria compartida de "container:lvaEdge" dentro de grpcExtension.

## <a name="sample-grpc-server"></a>Servidor de gRPC de ejemplo

Para entender los detalles sobre cómo se desarrolla el servidor de gRPC, analicemos el ejemplo de código.

1. Clone el repositorio desde el vínculo de GitHub [https://github.com/Azure-Samples/live-video-analytics-iot-edge-csharp](https://github.com/Azure-Samples/live-video-analytics-iot-edge-csharp).
1. Inicie VSCode y vaya a la carpeta /src/edge/modules/grpcExtension.
1. Veamos un tutorial rápido de los archivos:

    1. **Program.cs**: el punto de entrada de la aplicación. Es responsable de inicializar y administrar el servidor de gRPC, que actuará como host. En el ejemplo, el puerto para escuchar los mensajes gRPC entrantes de un cliente de gRPC (como Live Video Analytics) lo especifica el elemento de configuración grpcBindings en AppConfig.json.
    
        ```json    
        {
          "grpcBinding": "tcp://0.0.0.0:5001"
        }
        ```
    1. **Services\MediaGraphExtensionService.cs**: esta clase es responsable de controlar los mensajes [protobuf](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc). Leerá el fotograma del mensaje, invocará a ImageProcessor y escribirá los resultados de la inferencia.
Ahora que configuramos e inicializamos las conexiones del puerto del servidor de gRPC, veamos cómo podemos procesar los mensajes gRPC entrantes.

        * Una vez que se establece una sesión de gRPC, el primer mensaje que el servidor de gRPC recibirá del cliente (Live Video Analytics) es un descriptor MediaStreamDescriptor, que se define en el archivo[extension.proto](https://github.com/Azure/live-video-analytics/blob/master/contracts/grpc/extension.proto). 

            ```
            message MediaStreamDescriptor {
              oneof stream_identifier {
                GraphIdentifier graph_identifier = 1;     // Media Stream graph identifier
                string extension_identifier = 2;          // Media Stream extension identifier
              }
              MediaDescriptor media_descriptor = 5;       // Session media information.
              // Additional data transfer properties. If none is set, it is assumed
              // that all content will be transferred through messages (embedded transfer).
              oneof data_transfer_properties {
                SharedMemoryBufferTransferProperties shared_memory_buffer_transfer_properties = 10;
              }
            }
            ```
        * En la implementación del servidor, el método `ProcessMediaStreamDescriptor` validará la propiedad MediaDescriptor del mensaje MediaStreamDescriptor, que corresponde a un archivo de vídeo y, luego, configurará el modo de transferencia de datos (que es usar el modo de transferencia de memoria compartida o de fotograma insertado) en función de lo que especifique en la topología y el archivo de plantilla de implementación que se usa. 
        * Tras recibir el mensaje y configurar correctamente el modo de transferencia de datos, el servidor de gRPC devuelve el mensaje MediaStreamDescriptor al cliente como una confirmación y, por tanto, establece una conexión entre el servidor y el cliente.    
        * Una vez que Live Video Analytics reciba la confirmación, empezará a transferir el flujo multimedia al servidor de gRPC. En la implementación del servidor, el método `ProcessMediaStream` procesará el mensaje MediaStreamMessage entrante. El mensaje MediaStreamMessage también se define en [extension.proto](https://github.com/Azure/live-video-analytics/blob/master/contracts/grpc/extension.proto).

            ```
            message MediaStreamMessage {
              uint64 sequence_number = 1;       // Monotonically increasing directional message identifier starting from 1 when the gRPC connection is created
              uint64 ack_sequence_number = 2;   // 0 if this message is not referencing any sent message.
            
            
              // Possible payloads are strongly defined by the contract below
              oneof payload {
                MediaStreamDescriptor media_stream_descriptor = 5;
                MediaSample media_sample = 6;
              }
            }
            ```
        * Según el valor de batchSize en Appconfig.json, nuestro servidor seguirá recibiendo el mensaje y almacenará los fotogramas de vídeo en una lista. Una vez que se alcanza el límite de batchSize, la función llamará a la función o al archivo que procesará la imagen. En nuestro caso, el método llama a un archivo denominado BatchImageProcessor.cs
    1. **Processors\BatchImageProcessor.cs**: esta clase es responsable de procesar la imagen. En este ejemplo, usamos un modelo de clasificación de imágenes. Para cada imagen que se va a procesar, el algoritmo utilizado es el siguiente:

        1. Convierta la imagen en una matriz de bytes para su procesamiento. Consulte el método: `GetBytes(Bitmap image)`.
        
            El procesador de ejemplo que usamos solo admite un fotograma de imagen con codificación JPG y None como formato de píxel. Si un procesador personalizado admite una codificación o formato distintos, actualice el método `IsMediaFormatSupported` de la clase del procesador.
        1. Use la [clase ColorMatrix](/dotnet/api/system.drawing.imaging.colormatrix?preserve-view=true&view=dotnet-plat-ext-3.1) y convierta la imagen a la escala de grises. Consulte el método: `ToGrayScale(Image source)`.
        1. Una vez que se obtenga la imagen en escala de grises, se calculará el promedio de los bytes de la escala de grises.
        1. Si el valor promedio es menor que 127, la imagen se clasifica como "oscura"; de lo contrario, se clasificará como "clara" con un valor de confianza de 1,0. Consulte el método: `ProcessImage(List<Image> images)`.

    Puede agregar su propia lógica de procesador ya sea mediante la modificación de esta clase o agregando una clase nueva e implementando el método:

    ```
    IEnumerable<Inference> ProcessImage(List<Image> images) 
    ```

    Una vez que agregue la clase nueva, deberá actualizar MediaGraphExtensionService.cs para que cree una instancia de la clase e invoque al método ProcessImage en ella para ejecutar la lógica de procesamiento. 

## <a name="connect-with-live-video-analytics-module"></a>Conexión con el módulo de Live Video Analytics

Ahora que creó el módulo de la extensión de gRPC, vamos a crear e implementar la topología de grafos multimedia.

1. Mediante Visual Studio Code, siga [estas instrucciones](../../iot-edge/tutorial-develop-for-linux.md#build-and-push-your-solution) para iniciar sesión en Docker.
1. En Visual Studio Code, vaya a src/edge. Puede ver el archivo .env y algunos archivos de plantilla de implementación.

    La plantilla de implementación hace referencia al manifiesto de implementación del dispositivo perimetral. Incluye algunos valores de marcador de posición. El archivo .env incluye los valores de esas variables.
    
    A continuación, seleccione Build and Push IoT Edge Solution (Compilar e insertar solución IoT Edge). En este paso, use src/edge/deployment.grpc.template.json.
        
    :::image type="content" source="./media/develop-deploy-grpc-inference-srv-how-to/build-push-iot-edge-solution.png" alt-text="Conexión con el módulo de Live Video Analytics":::
    
    Esta acción crea el módulo del servidor de gRPC e insertar la imagen en la instancia de Azure Container Registry.
    Compruebe que ha definido las variables de entorno CONTAINER_REGISTRY_USERNAME_myacr y CONTAINER_REGISTRY_PASSWORD_myacr en el archivo .env.
1. Vaya a la carpeta src/cloud-to-device-console-app. Aquí verá tanto su archivo appsettings.json como otros archivos:

    * c2d-console-app.csproj: el archivo de proyecto de Visual Studio Code.
    * operations.json: una lista de las operaciones que desea que ejecute el programa.
    * Program.cs: el código del programa de ejemplo. Este código:

        * Carga la configuración de la aplicación.
        * Invoca los métodos directos que expone el módulo Live Video Analytics en IoT Edge. Puede usar el módulo para analizar secuencias de vídeo en directo mediante la invocación de sus [métodos directos](direct-methods.md).
        * Se pone en pausa para que pueda examinar la salida del programa en la ventana TERMINAL y los eventos generados por el módulo en la ventana SALIDA.
        * Invoca los métodos directos para limpiar los recursos.
1. Edite el archivo operations.json:

    * Cambie el vínculo a la topología del grafo:

        * `"topologyUrl" : https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/grpcExtension/topology.json`
        * En GraphInstanceSet, edite el nombre de la topología del grafo para que coincida con el valor del vínculo anterior:<br/>`"topologyName": "InferencingWithGrpcExtension"`
        * En GraphTopologyDelete, edite el nombre:<br/>`"name": "InferencingWithGrpcExtension"`

            La topología (por ejemplo, `https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/grpcExtensionOpenVINO/2.0/topology.json`) debe definir una dirección para la extensión:
    * Parámetro de dirección de la extensión

        ```
        {
            "name": "grpcExtensionAddress",
            "type": "String",
            "description": "gRPC LVA Extension Address",
            "default": "https://<REPLACE-WITH-IP-OR-CONTAINER-NAME>/score"
        },
        ```
    * Configuración

        ```
        {
            "@apiVersion": "1.0",
            "name": "TopologyName",
            "properties": {
            "processors": [
              {
                "@type": "#Microsoft.Media.MediaGraphGrpcExtension",
                "name": "grpcExtension",
                "endpoint": {
                  "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
                  "url": "${grpcExtensionAddress}",
                  "credentials": {
                    "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
                    "username": "${grpcExtensionUserName}",
                    "password": "${grpcExtensionPassword}"
                  }
                },
                "dataTransfer": {
                        "mode": "sharedMemory",
                        "SharedMemorySizeMiB": "5"
                },
                "image": {
                  "scale": {
                    "mode": "${imageScaleMode}",
                    "width": "${frameWidth}",
                    "height": "${frameHeight}"
                  },
                  "format": {
                    "@type": "#Microsoft.Media.MediaGraphImageFormatEncoded",
                    "encoding": "${imageEncoding}",
                    "quality": "${imageQuality}"
                  }
                }
            ]
          }
        }
        ```
    
## <a name="generate-and-deploy-the-iot-edge-deployment-manifest"></a>Generación e implementación del manifiesto de implementación de IoT Edge

El manifiesto de implementación define los módulos que se implementan en un dispositivo IoT Edge y los valores de configuración de dichos módulos. Siga estos pasos para generar un manifiesto a partir del archivo de plantilla y, a continuación, impleméntelo en el dispositivo IoT Edge.
Este paso crea el manifiesto de implementación de IoT Edge en src/edge/config/deployment.grpc.amd64.json. Haga clic con el botón derecho en ese archivo y seleccione **Crear una implementación para un dispositivo individual**.

:::image type="content" source="./media/develop-deploy-grpc-inference-srv-how-to/create-deployment-single-device.png" alt-text="Generación e implementación del manifiesto de implementación de IoT Edge":::

A continuación, Visual Studio Code le pide que seleccione un dispositivo IoT Hub. Seleccione su dispositivo IoT Edge, que debería ser lva-sample-device.
En esta fase, se ha iniciado la implementación de los módulos perimetrales en el dispositivo IoT Edge. En unos 30 segundos, actualice Azure IoT Hub en la sección inferior izquierda de Visual Studio Code. Debería ver que se implementó un módulo nuevo, denominado lvaExtension.

:::image type="content" source="./media/develop-deploy-grpc-inference-srv-how-to/devices.png" alt-text="Se implementó un módulo nuevo, denominado lvaExtension":::

## <a name="next-steps"></a>Pasos siguientes

Siga los pasos de **preparación de la supervisión de eventos** mencionados en el inicio rápido [Análisis de vídeo en directo con su propio modelo](use-your-model-quickstart.md) para ejecutar el ejemplo e interpretar los resultados. Consulte también las topologías de gRPC de ejemplo: [gRPCExtension](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/grpcExtensionOpenVINO/2.0/topology.json), [CVRWithGrpcExtension](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/cvr-with-grpcExtension/topology.json), [EVRtoAssetsByGrpcExtension y [EVROnMotionPlusGrpcExtension](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/motion-with-grpcExtension/topology.json).
