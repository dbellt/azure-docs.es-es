---
title: Administración de puntos de conexión de streaming
description: En este artículo se muestra cómo administrar los puntos de conexión de streaming mediante Azure Media Services v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
writer: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 95873a4a4b815df53e42bf265a3187f73344f882
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108072592"
---
# <a name="manage-streaming-endpoints-with--media-services-v3"></a>Administración de puntos de conexión de streaming con Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Cuando se crea la cuenta de Media Services, se agrega un [punto de conexión de streaming](stream-streaming-endpoint-concept.md) **predeterminado** a la cuenta con estado **Stopped** (Detenido). Para iniciar la transmisión del contenido y aprovechar el [empaquetado dinámico](encode-dynamic-packaging-concept.md) y el [cifrado dinámico](drm-content-protection-concept.md), el punto de conexión de streaming desde el que va a transmitir el contenido tiene que estar en estado **Ejecutando**.

En este artículo se muestra cómo ejecutar el comando [start](/rest/api/media/streamingendpoints/start) en el punto de conexión de streaming mediante distintas tecnologías. 
 
> [!NOTE]
> Solo se le cobrará cuando StreamingEndpoint esté en estado en ejecución.
    
## <a name="prerequisites"></a>Prerrequisitos

Revisión: 

* [Conceptos de Azure Media Services](concepts-overview.md)
* [Concepto de punto de conexión de streaming](stream-streaming-endpoint-concept.md)
* [Empaquetado dinámico](encode-dynamic-packaging-concept.md)

## <a name="use-rest"></a>Uso de la API REST

```rest
POST https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/mediaresources/providers/Microsoft.Media/mediaservices/slitestmedia10/streamingEndpoints/myStreamingEndpoint1/start?api-version=2018-07-01
```

Para más información, consulte: 

* La documentación de referencia [Inicio de un punto de conexión de streaming](/rest/api/media/streamingendpoints/start).
* El inicio de un punto de conexión de streaming es una operación asincrónica. 

    Para información sobre cómo supervisar las operaciones de larga duración, consulte [Operaciones de larga duración](media-services-apis-overview.md).
* Esta [colección de Postman](https://github.com/Azure-Samples/media-services-v3-rest-postman/blob/master/Postman/Media%20Services%20v3.postman_collection.json) contiene ejemplos de varias operaciones de REST, entre ellas, cómo iniciar un punto de conexión de streaming.

## <a name="use-the-azure-portal"></a>Uso de Azure Portal 
 
1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Vaya a la cuenta de Azure Media Services.
1. En el panel izquierdo, seleccione **Streaming Endpoints** (Puntos de conexión de streaming).
1. Seleccione el punto de conexión de streaming que quiere iniciar y, luego, elija **Start** (Iniciar).

## <a name="use-the-azure-cli"></a>Uso de la CLI de Azure

```cli
az ams streaming-endpoint start [--account-name]
                                [--ids]
                                [--name]
                                [--no-wait]
                                [--resource-group]
                                [--subscription]
```

Para más información, consulte [az ams streaming-endpoint start](/cli/azure/ams/streaming-endpoint#az_ams_streaming_endpointstart).

## <a name="use-sdks"></a>Uso de SDK

### <a name="java"></a>Java
    
```java
if (streamingEndpoint != null) {
// Start The Streaming Endpoint if it is not running.
if (streamingEndpoint.resourceState() != StreamingEndpointResourceState.RUNNING) {
    manager.streamingEndpoints().startAsync(config.getResourceGroup(), config.getAccountName(), STREAMING_ENDPOINT_NAME).await();
}
```

Consulte el [ejemplo de código de Java](https://github.com/Azure-Samples/media-services-v3-java/blob/master/DynamicPackagingVODContent/StreamHLSAndDASH/src/main/java/sample/StreamHLSAndDASH.java#L128) completo.

### <a name="net"></a>.NET

```csharp
StreamingEndpoint streamingEndpoint = await client.StreamingEndpoints.GetAsync(config.ResourceGroup, config.AccountName, DefaultStreamingEndpointName);

if (streamingEndpoint != null)
{
    if (streamingEndpoint.ResourceState != StreamingEndpointResourceState.Running)
    {
        await client.StreamingEndpoints.StartAsync(config.ResourceGroup, config.AccountName, DefaultStreamingEndpointName);
    }
```

Consulte el [ejemplo de código de .NET](https://github.com/Azure-Samples/media-services-v3-dotnet/blob/main/Streaming/StreamHLSAndDASH/Program.cs#L112) completo.

---

## <a name="next-steps"></a>Pasos siguientes

* [Especificación de OpenAPI de Media Services v3 (Swagger)](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/mediaservices/resource-manager/Microsoft.Media/stable/2018-07-01)
* [Operaciones de punto de conexión de streaming](/rest/api/media/streamingendpoints)
