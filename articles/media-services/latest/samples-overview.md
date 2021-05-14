---
title: Ejemplos de Media Services v3
description: Este artículo contiene una lista de todos los ejemplos disponibles para Media Services v3 organizados por método y SDK.  Entre los ejemplos se incluyen .NET, Node.JS, Python y Java, así como REST con Postman.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: overview
ms.date: 03/24/2021
ms.author: inhenkel
ms.openlocfilehash: cd82da805538abcddf2f76a20c28cb0a74b5c89b
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109685610"
---
# <a name="media-services-v3-samples"></a>Ejemplos de Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Este artículo contiene una lista de todos los ejemplos disponibles para Media Services organizados por método y SDK.  Entre los ejemplos se incluyen .NET, Node.JS, Python y Java, así como REST con Postman.

## <a name="rest-postman-collection"></a>Colección de Postman para REST

Los ejemplos de [Postman para REST](https://github.com/Azure-Samples/media-services-v3-rest-postman) incluyen un entorno y una colección de Postman para que los importe en el cliente de Postman.

## <a name="samples-by-sdk"></a>Ejemplos por SDK

Encontrará la descripción y los vínculos a los ejemplos que puede encontrar en cada una de las pestañas.

## <a name="net"></a>[.NET](#tab/net/)

| Carpeta | Descripción |
|-------------|-------------|
| [VideoEncoding/EncodingWithMESPredefinedPreset](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoEncoding/Encoding_PredefinedPreset)|Cómo enviar un trabajo mediante un valor preestablecido integrado y una entrada de dirección URL HTTP, publicar un recurso de salida para la transmisión y descargar los resultados para su comprobación.|
| [VideoEncoding/EncodingWithMESCustomPreset_H264](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoEncoding/Encoding_H264)|Cómo enviar un trabajo mediante un valor preestablecido de codificación H.264 personalizado y una entrada de dirección URL HTTP, publicar un recurso de salida para la transmisión y descargar los resultados para su comprobación.|
| [VideoEncoding/EncodingWithMESCustomPreset_HEVC](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoEncoding/Encoding_HEVC)|Cómo enviar un trabajo mediante un valor preestablecido de codificación HEVC y una entrada de dirección URL HTTP, publicar un recurso de salida para la transmisión y descargar los resultados para su comprobación.|
| [VideoEncoding/EncodingWithMESCustomStitchTwoAssets](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoEncoding/Encoding_StitchTwoAssets)|Cómo enviar un trabajo mediante JobInputSequence para unir 2 o más recursos que se puedan recortar por hora de inicio o de finalización. El archivo codificado resultante es un vídeo único con todos los recursos unidos.  El ejemplo también publicará el recurso de salida para la transmisión y descargará los resultados para su comprobación.|
| [VideoEncoding/EncodingWithMESCustomPresetAndSprite](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoEncoding/Encoding_SpriteThumbnail)|Cómo enviar un trabajo mediante un valor preestablecido personalizado con un sprite de miniaturas y una entrada de dirección URL HTTP, publicar un recurso de salida para la transmisión y descargar los resultados para su comprobación.|
| [Live/LiveEventWithDVR](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/Live/LiveEventWithDVR)|Cómo crear un evento en directo con un archivo completo de hasta 25 horas y un filtro en el recurso con una ventana de 5 minutos de DVR. Cómo usar un filtro para crear un localizador para la transmisión.|
| [VideoAnalytics/VideoAnalyzer](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoAnalytics/VideoAnalyzer)|Cómo crear una transformación del analizador de vídeo, cargar un archivo de vídeo en un recurso de entrada, enviar un trabajo con la transformación y descargar los resultados para su comprobación.|
| [AudioAnalytics/AudioAnalyzer](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/AudioAnalytics/AudioAnalyzer)|Cómo crear una transformación del analizador de audio, cargar un archivo multimedia en un recurso de entrada, enviar un trabajo con la transformación y descargar los resultados para su comprobación.|
| [ContentProtection/BasicAESClearKey](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/ContentProtection/BasicAESClearKey)|Cómo crear una transformación con un valor preestablecido de AdaptiveStreaming integrado, enviar un trabajo, crear un elemento ContentKeyPolicy con una clave secreta, asociar el elemento ContentKeyPolicy con un objeto StreamingLocator, obtener un token e imprimir una dirección URL para la reproducción en Azure Media Player. Cuando un reproductor solicita una secuencia, Media Services usa la clave especificada para cifrar de forma dinámica el contenido con AES-128 y Azure Media Player utiliza el token para descifrarlo.|
| [ContentProtection/BasicWidevine](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/ContentProtection/BasicWidevine)|Cómo crear una transformación con un valor preestablecido de AdaptiveStreaming integrado, enviar un trabajo, crear un elemento ContentKeyPolicy con la configuración de Widevine con una clave secreta, asociar el elemento ContentKeyPolicy con un objeto StreamingLocator, obtener un token e imprimir una dirección URL para la reproducción en Widevine Player. Cuando un usuario solicita contenido protegido por Widevine, la aplicación del reproductor solicita una licencia al servicio de licencias de Media Services. Si la aplicación del reproductor está autorizada, el servicio de licencias de Media Services otorga una licencia al reproductor. Una licencia de Widevine contiene la clave de descifrado que puede usar el reproductor cliente para descifrar y transmitir el contenido.|
| [ContentProtection/BasicPlayReady](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/ContentProtection/BasicPlayReady)|Cómo crear una transformación con un valor preestablecido de AdaptiveStreaming integrado, enviar un trabajo, crear un elemento ContentKeyPolicy con la configuración de PlayReady con una clave secreta, asociar el elemento ContentKeyPolicy con un objeto StreamingLocator, obtener un token e imprimir una dirección URL para la reproducción en Azure Media Player. Cuando un usuario solicita contenido protegido por PlayReady, la aplicación del reproductor solicita una licencia al servicio de licencias de Media Services. Si la aplicación del reproductor está autorizada, el servicio de licencias de Media Services otorga una licencia al reproductor. Una licencia de PlayReady contiene la clave de descifrado que se puede usar por el reproductor del cliente para descifrar y hacer streaming del contenido.|
| [ContentProtection/OfflinePlayReadyAndWidevine](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/ContentProtection/OfflinePlayReadyAndWidevine)|Cómo cifrar dinámicamente el contenido con DRM de PlayReady y Widevine y reproducir el contenido sin solicitar una licencia al servicio de licencias. Cómo crear una transformación con un valor preestablecido de AdaptiveStreaming integrado, enviar un trabajo, crear un elemento ContentKeyPolicy con restricciones abiertas y la configuración persistente de PlayReady y Widevine, asociar el elemento ContentKeyPolicy con un objeto StreamingLocator e imprimir una dirección URL para la reproducción.|
| [Streaming/AssetFilters](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/Streaming/AssetFilters)|Cómo crear una transformación con valores preestablecidos de AdaptiveStreaming integrados, enviar un trabajo, crear un filtro de recursos y un filtro de cuentas, asociar los filtros a los localizadores de streaming e imprimir direcciones URL para la reproducción.|
| [Streaming/StreamHLSAndDASH](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/Streaming/StreamHLSAndDASH)|Cómo crear una transformación con un valor preestablecido de AdaptiveStreaming integrado, enviar un trabajo y publicar un recurso de salida para la transmisión HLS y DASH.|
| [HighAvailabilityEncodingStreaming](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/HighAvailabilityEncodingStreaming/) | Instrucciones y procedimientos recomendados para un sistema de producción con codificación o análisis a petición. Los lectores deben comenzar por el artículo complementario [Alta disponibilidad con Media Services y Vídeo bajo demanda (VOD)](./architecture-high-availability-encoding-concept.md). Se proporciona un archivo de solución independiente para el ejemplo [HighAvailabilityEncodingStreaming](https://github.com/Azure-Samples/media-services-v3-dotnet/blob/main/HighAvailabilityEncodingStreaming/README.md). |

## <a name="nodejs"></a>[Node.JS](#tab/node/)

|Carpeta|Descripción|
|---|---|
|[HelloWorld-ListAssets](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/main/AMSv3Samples/HelloWorld-ListAssets) |Cómo conectar y enumerar los recursos |
|[Directo](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/main/AMSv3Samples/Live)| Cómo hacer streaming en vivo básico. **ADVERTENCIA**: asegúrese de comprobar que todos los recursos se han limpiado y ya no se facturan en el portal al usar en producción.|
|[StreamFilesSample](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/main/AMSv3Samples/StreamFilesSample)| Cómo cargar un archivo local o una codificación desde una dirección URL de origen, cómo usar el SDK de Storage para descargar contenido y cómo transmitir a un reproductor. |
|[StreamFilesWithDRMSample](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/main/AMSv3Samples/StreamFilesWithDRMSample)| Cómo codificar y transmitir con DRM de Widevine y PlayReady |
|[VideoIndexerSample](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/main/AMSv3Samples/VideoIndexerSample)| Cómo usar los valores preestablecidos del analizador de vídeo y audio para generar metadatos e información a partir de un archivo de vídeo o audio. |

## <a name="python"></a>[Python](#tab/python)

Actualmente, hay un ejemplo de Python, [Codificación básica con Python](https://github.com/Azure-Samples/media-services-v3-python).

## <a name="java"></a>[Java](#tab/java)

|Carpeta|Descripción|
|---|---|
|[AudioAnalytics/AudioAnalyzer/](https://github.com/Azure-Samples/media-services-v3-java/tree/master/AudioAnalytics/AudioAnalyzer)|Cómo analizar el audio en un archivo multimedia. |
|Content Protection|
|ContentProtection||
|[BasicAESClearKey](https://github.com/Azure-Samples/media-services-v3-java/tree/master/ContentProtection/BasicAESClearKey)|Cómo cifrar dinámicamente el contenido con AES-128.|
|[BasicPlayReady](https://github.com/Azure-Samples/media-services-v3-java/tree/master/ContentProtection/BasicPlayReady)|Cómo cifrar dinámicamente el contenido con DRM de PlayReady.|
|[BasicWidevine](https://github.com/Azure-Samples/media-services-v3-java/tree/master/ContentProtection/BasicWidevine)|Cómo cifrar dinámicamente el contenido con DRM de Widevine.|
|[OfflineFairPlay](https://github.com/Azure-Samples/media-services-v3-java/tree/master/ContentProtection/OfflineFairPlay)|Cómo cifrar dinámicamente el contenido con DRM de FairPlay y reproducir el contenido sin solicitar una licencia al servicio de licencias.|
|[OfflinePlayReadyAndWidevine](https://github.com/Azure-Samples/media-services-v3-java/tree/master/ContentProtection/OfflinePlayReadyAndWidevine)|Cómo cifrar dinámicamente el contenido con DRM de PlayReady y Widevine y reproducir el contenido sin solicitar una licencia al servicio de licencias.|
|DynamicPackagingVODContent||
|[AssetFilters](https://github.com/Azure-Samples/media-services-v3-java/tree/master/DynamicPackagingVODContent/AssetFilters)|Cómo filtrar el contenido mediante los filtros de cuentas y recursos.|
|[StreamHLSAndDASH](https://github.com/Azure-Samples/media-services-v3-java/tree/master/DynamicPackagingVODContent/StreamHLSAndDASH)|Cómo empaquetar dinámicamente el contenido de VOD en HLS y DASH para la transmisión.|
|[LiveIngest/LiveEventWithDVR](https://github.com/Azure-Samples/media-services-v3-java/tree/master/LiveIngest/LiveEventWithDVR)|Cómo crear un evento en directo con un archivo completo de hasta 25 horas y un filtro en el recurso con una ventana de 5 minutos de DVR y cómo crear un localizador para la transmisión y usar el filtro.|
|[VideoAnalytics/VideoAnalyzer](https://github.com/Azure-Samples/media-services-v3-java/tree/master/VideoAnalytics/VideoAnalyzer)|Cómo crear un evento en directo con un archivo completo de hasta 25 horas y un filtro en el recurso con una ventana de 5 minutos de DVR y cómo crear un localizador para la transmisión y usar el filtro.|
|VideoEncoding||
|[EncodingWithMESCustomPreset](https://github.com/Azure-Samples/media-services-v3-java/tree/master/VideoEncoding/EncodingWithMESCustomPreset)|Cómo crear una transformación de codificación personalizada mediante la configuración de StandardEncoderPreset.|
|[EncodingWithMESPredefinedPreset](https://github.com/Azure-Samples/media-services-v3-java/tree/master/VideoEncoding/EncodingWithMESPredefinedPreset)|Cómo enviar un trabajo mediante un valor preestablecido integrado y una entrada de dirección URL HTTP, publicar un recurso de salida para la transmisión y descargar los resultados para su comprobación.|

---
