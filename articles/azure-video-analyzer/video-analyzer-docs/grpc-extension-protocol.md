---
title: 'Protocolo de la extensión gRPC: Azure'
description: Azure Video Analyzer permite mejorar sus funcionalidades de procesamiento mediante un nodo de extensión de canalización. El procesador de extensiones de gRPC permite escenarios de extensibilidad mediante el uso de un protocolo estructurado de alto rendimiento basado en gRPC.
ms.topic: reference
ms.date: 05/15/2021
ms.openlocfilehash: aaa1030f2c751514568c96142268d41c2eba4df6
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388728"
---
# <a name="use-the-grpc-extension-protocol"></a>Uso del protocolo de la extensión gRPC 

Azure Video Analyzer permite mejorar sus funcionalidades de procesamiento de canalizaciones mediante un [nodo de extensión de canalización](pipeline-extension.md). El procesador de la extensión gRPC permite escenarios de extensibilidad mediante el uso de un [protocolo estructurado de alto rendimiento basado en gRPC](pipeline-extension.md#grpc-extension-processor).

En este artículo, aprenderá a usar el protocolo de la extensión gRPC para enviar mensajes entre el módulo de Video Analyzer y el servidor gRPC que procesa esos mensajes y devuelve los resultados. gRPC es un marco de RPC moderno, de código abierto y de alto rendimiento que se ejecuta en cualquier entorno y admite la comunicación entre plataformas y entre lenguajes. El servicio de transporte gRPC utiliza streaming bidireccional HTTP/2 entre:

* el cliente gRPC (módulo de Video Analyzer) y
* el servidor de gRPC (su extensión personalizada).

Una sesión de gRPC es una conexión única desde el cliente de gRPC al servidor de gRPC a través del puerto TCP/TLS.
En una sola sesión: El cliente envía un descriptor de flujo multimedia seguido de fotogramas de vídeo al servidor como un mensaje [protobuf](https://github.com/Azure/video-analyzer/tree/main/contracts/grpc) en la sesión de flujo de gRPC. El servidor valida el descriptor de flujo, analiza el fotograma de vídeo y devuelve los resultados de la inferencia como un mensaje protobuf.

Se recomienda que las respuestas se devuelvan mediante documentos JSON válidos y que se siga el esquema preestablecido definido en el [modelo de objetos del esquema de metadatos de inferencia](inference-metadata-schema.md). Esto garantizará mejor la interoperabilidad con otros componentes y escenarios, como la grabación y reproducción de vídeo con metadatos de inferencia.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/grpc-extension-protocol/ava-module.png" alt-text="Módulo de Azure Video Analyzer" lightbox="./media/grpc-extension-protocol/ava-module.png":::

## <a name="implementing-grpc-protocol"></a>Implementación del protocolo gRPC

### <a name="creating-a-grpc-connection"></a>Creación de una conexión de gRPC

La extensión personalizada tiene que implementar el siguiente servicio gRPC:

```
service MediaGraphExtension
    {
        rpc ProcessMediaStream(stream MediaStreamMessage) returns (stream MediaStreamMessage);
    }
```

Cuando se le llama, se abrirá una secuencia bidireccional para que los mensajes fluyan entre la extensión gRPC y la canalización en directo de Video Analyzer. El primer mensaje enviado en esta secuencia por cada entidad contendrá un MediaStreamDescriptor, que define qué información se enviará en los siguientes MediaSamples.

Por ejemplo, la extensión puede enviar el mensaje (expresado aquí en formato JSON) para indicar que enviará los fotogramas codificados en rgb24 de 416x416 insertados en los mensajes gRPC a la extensión personalizada.

```
 {
    "sequence_number": 1,
    "ack_sequence_number": 0,
    "media_stream_descriptor": 
    {
        "graph_identifier": 
        {
            "media_services_arm_id": "/subscriptions/{subscriptionID}/resourceGroups/{resource-group-name}/providers/microsoft.media/videoanalyzers/{video-analyzer-account-name}",
            "graph_instance_name": "{live-pipeline-name}",
            "graph_node_name": "{grpc-extension-node-name}"
        },
        "media_descriptor": 
        {
            "timescale": 90000,
            "video_frame_sample_format": 
            {
                "encoding": "RAW",
                "pixel_format": "RGB24",
                "dimensions": 
                {
                    "width": 416,
                    "height": 416
                },
                "stride_bytes": 1248
            }
        }
    }
}
```

La extensión personalizada, en respuesta, enviará el mensaje siguiente para indicar que solo devuelve inferencias.

```
{
    "sequence_number": 1,
    "ack_sequence_number": 1,
    "media_stream_descriptor": 
    {
        "extension_identifier": "customExtensionName"    
    }
}
```

Ahora que ambos lados han intercambiado los descriptores multimedia, Video Analyzer comenzará a transmitir fotogramas al servidor gRPC.

> [!NOTE]
> El servidor gRPC se puede implementar con el lenguaje de programación que se desee.

### <a name="sequence-numbers"></a>Números de secuencia

Tanto el nodo de extensión gRPC como la extensión personalizada mantienen un conjunto independiente de números de secuencia, que se asignan a sus mensajes. Estos números de secuencia deben aumentar con una progresión continua a partir de 1. Se puede omitir ack_sequence_number si no se ha confirmado ningún mensaje, lo que puede ocurrir cuando se envía el primer mensaje.

Una solicitud se tiene que confirmarse una vez que se haya procesado completamente el mensaje correspondiente. En el caso de una transferencia de memoria compartida, esta confirmación indica que el remitente puede liberar la memoria compartida y que el receptor ya no tendrá acceso a ella. El remitente tiene que contener cualquier memoria compartida hasta que se confirme.

### <a name="reading-embedded-content"></a>Lectura del contenido insertado

El contenido insertado se puede leer directamente del mensaje MediaSample mediante el campo content_bytes. Los datos se codificarán de acuerdo con el elemento MediaDescriptor.

### <a name="reading-content-from-shared-memory"></a>Lectura del contenido de la memoria compartida

Al transferir contenido mediante la memoria compartida, el remitente incluirá el elemento SharedMemoryBufferTransferProperties en el elemento MediaStreamDescriptor cuando establezca una sesión por primera vez. Esto puede tener el siguiente aspecto (expresado en JSON):

```
{
  "handle_name": "inference_client_share_memory_2146989006636459346"
  "length_bytes": 20971520
}
```

A continuación, el receptor abre el archivo /dev/shm/inference_client_share_memory_2146989006636459346. El remitente enviará un mensaje MediaSample, que contiene un elemento ContentReference que hace referencia a una ubicación específica de este archivo.

```
{
    "timestamp": 143598615750000,
    "content_reference": 
    {
        "address_offset": 519168,
        "length_bytes": 173056
    }
}
```

A continuación, el receptor lee los datos en esta ubicación en el archivo.

Para que el módulo perimetral de Video Analyzer se comunique mediante la memoria compartida, el modo IPC del contenedor debe estar configurado correctamente. Esto se puede hacer de muchas maneras, pero estas son algunas de las configuraciones recomendadas.

* Al comunicarse con un motor de inferencia de gRPC que se ejecuta en el dispositivo host, el modo IPC debe establecerse en host.
* Al comunicarse con un servidor gRPC que se ejecuta en otro módulo IoT Edge, el modo IPC se debe establecer en `shareable` para el módulo de Video Analyzer y en `container:avaedge` para la extensión personalizada, donde `avaedge` es el nombre del módulo de Video Analyzer.

Este es el aspecto que podría tener esto en el dispositivo gemelo cuando se usa la primera de las opciones anteriores.

```
"avaedge": 
{
  "version": "1.0",
  "type": "docker",
  "status": "running",
  "restartPolicy": "always",
  "settings": 
  {
    "image": "mcr.microsoft.com/media/video-analyzer:1",
    "createOptions": 
      "HostConfig": 
      {
        "IpcMode": "shareable"
      }
  }
}
```

Para más información sobre los modos IPC, consulte [Configuración de IPC (--ipc)](https://docs.docker.com/engine/reference/run/#ipc-settings---ipc).

## <a name="video-analyzer-grpc-extension-contract-definitions"></a>Definiciones de contrato de la extensión gRPC de Video Analyzer

En esta sección define el contrato gRPC que define el flujo de datos.

### <a name="protocol-messages"></a>Mensajes de protocolo

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/grpc-extension-protocol/contract-definitions.png" alt-text="Mensajes de protocolo de Video Analyzer"  lightbox="./media/grpc-extension-protocol/contract-definitions.png":::

### <a name="client-authentication"></a>Autenticación de cliente

Los implementadores de extensiones personalizadas pueden validar la autenticidad de las conexiones gRPC entrantes para asegurarse de que provienen del nodo de extensión gRPC. El nodo proporcionará una entrada en los encabezados de solicitud que sirve para la validación.

Las credenciales de nombre de usuario y contraseña se pueden usar con este fin. Al crear un nodo de extensión gRPC, las credenciales se proporcionan como se indica a continuación:

```
{
  "@type": " #Microsoft.VideoAnalyzer.GrpcExtension ",
  "name": "{moduleIdentifier}",
  "endpoint": 
  {
    "@type": " #Microsoft.VideoAnalyzer.UnsecuredEndpoint ",
    "url": "tcp://customExtension:8081",
    "credentials": 
    {
      "@type": "#Microsoft. VideoAnalyzer.UsernamePasswordCredentials",
      "username": "username",
      "password": "password"
    }
  }
  // Other fields omitted
}
```

Cuando se envía la solicitud gRPC, el encabezado siguiente se incluirá en los metadatos de la solicitud, imitando la autenticación HTTP Basic.

```
x-ms-authentication: Basic (Base64 Encoded username:password)
```

## <a name="configuring-inference-server-for-each-live-pipeline-over-grpc-extension"></a>Configuración de un servidor de inferencia para cada canalización de directo sobre la extensión gRPC

Al configurar el servidor de inferencia, no es preciso exponer un nodo para cada modelo de IA que esté empaquetado en el servidor de inferencia. En su lugar, en el caso de una canalización en directo, puede usar la propiedad `extensionConfiguration` del nodo **GrpcExtension** y definir cómo seleccionar los modelos de IA. Durante la ejecución, Video Analyzer pasará esta cadena al servidor de inferencia, que puede usarla para invocar al modelo de IA deseado. `extensionConfiguration property` es una propiedad opcional y es específica de la implementación del servidor gRPC. La propiedad se puede utilizar como se muestra a continuación:

```
{
  "@type": "#Microsoft.VideoAnalyzer.GrpcExtension",
  "name": "{moduleIdentifier}",
  "endpoint": 
  {
    "@type": "#Microsoft.VideoAnalyzer.UnsecuredEndpoint",
    "url": "${grpcExtensionAddress}",
    "credentials": 
    {
      "@type": "#Microsoft.VideoAnalyzer.UsernamePasswordCredentials",
      "username": "${grpcExtensionUserName}",
      "password": "${grpcExtensionPassword}"
    }
  },
    // Optional server configuration string. This is server specific 
  "extensionConfiguration": "{Optional extension specific string}",
  "dataTransfer": 
  {
    "mode": "sharedMemory",
    "SharedMemorySizeMiB": "75"
  }
    //Other fields omitted
}
```

## <a name="using-grpc-over-tls"></a>Uso de gRPC a través de TLS

Una conexión gRPC utilizada para la inferencia se puede proteger a través de TLS. Esto resulta útil en situaciones en las que no se puede garantizar la seguridad de la red entre Video Analyzer y el motor de inferencia. TLS cifrará todo el contenido insertado en los mensajes de gRPC, lo que provocará una sobrecarga adicional de la CPU al transmitir fotogramas a alta velocidad.

Las opciones de comprobación `IgnoreHostname` e `IgnoreSignature` no son compatibles con gRPC, por lo que el certificado de servidor que presente el motor de inferencia debe contener un nombre común (CN) que coincida exactamente con la dirección IP o el nombre de host de la dirección URL del punto de conexión del nodo de extensión gRPC.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre el [esquema de metadatos de inferencia](inference-metadata-schema.md).
