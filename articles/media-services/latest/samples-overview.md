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
ms.openlocfilehash: abdaafbaf03e76c9de466fa8fb264c3ab5cc00d3
ms.sourcegitcommit: 6bd31ec35ac44d79debfe98a3ef32fb3522e3934
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/02/2021
ms.locfileid: "113216322"
---
# <a name="media-services-v3-samples"></a>Ejemplos de Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Este artículo contiene una lista de todos los ejemplos disponibles para Media Services organizados por método y SDK.  Entre los ejemplos se incluyen .NET, Node.JS (Typescript), Python y Java, así como REST con Postman.

## <a name="rest-postman-collection"></a>Colección de Postman para REST

Los ejemplos de [Postman para REST](https://github.com/Azure-Samples/media-services-v3-rest-postman) incluyen un entorno y una colección de Postman para que los importe en el cliente de Postman. Los ejemplos de la colección de Postman se recomiendan para familiarizarse con la estructura de la API y cómo funciona con Azure Resource Manager (ARM), así como la estructura de las llamadas desde los SDK de cliente. 

En el caso de las cargas de trabajo de producción, debe usar los SDK de cliente que encapsulan esta API REST, ya que admiten directivas de reintento definidas por la puerta de enlace de Azure Resource Manager. Si decide implementar llamadas a la API REST directamente, debe tener en cuenta que hay casos en los que se requieren reintentos para lograr mayores SLA.

## <a name="samples-by-sdk"></a>Ejemplos por SDK

Encontrará la descripción y los vínculos a los ejemplos que puede encontrar en cada una de las pestañas.

## <a name="net"></a>[.NET](#tab/net/)

| Muestra | Descripción |
|-------------|-------------|
| [Account/CreateAccount](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/Account/CreateAccount)|En el ejemplo, se muestra cómo crear una cuenta de Media Services y establecer la cuenta de almacenamiento principal, además de opciones de configuración avanzadas, como la lista de direcciones IP permitidas para la entrega de claves, la identidad administrada, la autenticación del almacenamiento y la incorporación de su propia clave de cifrado.|
| [VideoEncoding/Encoding_PredefinedPreset](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoEncoding/Encoding_PredefinedPreset)|En el ejemplo, se muestra cómo enviar un trabajo mediante un valor preestablecido integrado y una entrada de dirección URL HTTP, publicar un recurso de salida para el streaming y descargar los resultados para su comprobación.|
| [VideoEncoding/Encoding_H264](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoEncoding/Encoding_H264)|En el ejemplo, se muestra cómo enviar un trabajo mediante un valor preestablecido de codificación H.264 personalizado y una entrada de dirección URL HTTP, publicar un recurso de salida para la transmisión y descargar los resultados para su comprobación.|
| [VideoEncoding/Encoding_HEVC](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoEncoding/Encoding_HEVC)|En el ejemplo, se muestra cómo enviar un trabajo mediante un valor preestablecido de codificación HEVC personalizado y una entrada de dirección URL HTTP, publicar un recurso de salida para la transmisión y descargar los resultados para su comprobación.|
| [VideoEncoding/Encoding_StitchTwoAssets](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoEncoding/Encoding_StitchTwoAssets)|En el ejemplo, se muestra cómo enviar un trabajo mediante JobInputSequence para unir 2 o más recursos que se puedan recortar por hora de inicio o de finalización. El archivo codificado resultante es un vídeo único con todos los recursos unidos.  El ejemplo también publicará el recurso de salida para la transmisión y descargará los resultados para su comprobación.|
| [VideoEncoding/Encoding_SpriteThumbnail](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoEncoding/Encoding_SpriteThumbnail)|En el ejemplo, se muestra cómo enviar un trabajo mediante un valor preestablecido personalizado con un sprite de miniaturas y una entrada de dirección URL HTTP, publicar un recurso de salida para la transmisión y descargar los resultados para su comprobación.|
| [Live/LiveEventWithDVR](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/Live/LiveEventWithDVR)|En primer lugar, en este ejemplo se muestra cómo crear un objeto LiveEvent con un archivo completo de hasta 25 horas y un filtro en el recurso con una ventana de DVR de 5 minutos y, a continuación, se muestra cómo usar el filtro para crear un localizador para streaming.|
| [VideoAnalytics/VideoAnalyzer](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoAnalytics/VideoAnalyzer)|En este ejemplo, se muestra cómo crear una transformación del analizador de vídeo, cargar un archivo de vídeo en un recurso de entrada, enviar un trabajo con la transformación y descargar los resultados para su comprobación.|
| [AudioAnalytics/AudioAnalyzer](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/AudioAnalytics/AudioAnalyzer)|En este ejemplo, se muestra cómo crear una transformación del analizador de audio, cargar un archivo multimedia en un recurso de entrada, enviar un trabajo con la transformación y descargar los resultados para su comprobación.|
| [ContentProtection/BasicAESClearKey](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/ContentProtection/BasicAESClearKey)|En este ejemplo, se muestra cómo crear una transformación con un valor preestablecido de AdaptiveStreaming integrado, enviar un trabajo, crear un elemento ContentKeyPolicy con una clave secreta, asociar el elemento ContentKeyPolicy con un objeto StreamingLocator, obtener un token e imprimir una dirección URL para la reproducción en Azure Media Player. Cuando un reproductor solicita una secuencia, Media Services usa la clave especificada para cifrar de forma dinámica el contenido con AES-128 y Azure Media Player utiliza el token para descifrarlo.|
| [ContentProtection/BasicWidevine](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/ContentProtection/BasicWidevine)|En este ejemplo, se muestra cómo crear una transformación con un valor preestablecido de AdaptiveStreaming integrado, enviar un trabajo, crear un elemento ContentKeyPolicy con la configuración de Widevine con una clave secreta, asociar el elemento ContentKeyPolicy con un objeto StreamingLocator, obtener un token e imprimir una dirección URL para la reproducción en Widevine Player. Cuando un usuario solicita contenido protegido por Widevine, la aplicación del reproductor solicita una licencia al servicio de licencias de Media Services. Si la aplicación del reproductor está autorizada, el servicio de licencias de Media Services otorga una licencia al reproductor. Una licencia de Widevine contiene la clave de descifrado que puede usar el reproductor cliente para descifrar y transmitir el contenido.|
| [ContentProtection/BasicPlayReady](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/ContentProtection/BasicPlayReady)|En este ejemplo, se muestra cómo crear una transformación con un valor preestablecido de AdaptiveStreaming integrado, enviar un trabajo, crear un elemento ContentKeyPolicy con la configuración de PlayReady con una clave secreta, asociar el elemento ContentKeyPolicy con un objeto StreamingLocator, obtener un token e imprimir una dirección URL para la reproducción en Azure Media Player. Cuando un usuario solicita contenido protegido por PlayReady, la aplicación del reproductor solicita una licencia al servicio de licencias de Media Services. Si la aplicación del reproductor está autorizada, el servicio de licencias de Media Services otorga una licencia al reproductor. Una licencia de PlayReady contiene la clave de descifrado que se puede usar por el reproductor del cliente para descifrar y hacer streaming del contenido.|
| [ContentProtection/OfflinePlayReadyAndWidevine](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/ContentProtection/OfflinePlayReadyAndWidevine)|En este ejemplo, se muestra cómo cifrar dinámicamente el contenido con DRM de PlayReady y Widevine y reproducir el contenido sin solicitar una licencia al servicio de licencias. Cómo crear una transformación con un valor preestablecido de AdaptiveStreaming integrado, enviar un trabajo, crear un elemento ContentKeyPolicy con restricciones abiertas y la configuración persistente de PlayReady y Widevine, asociar el elemento ContentKeyPolicy con un objeto StreamingLocator e imprimir una dirección URL para la reproducción.|
| [Streaming/AssetFilters](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/Streaming/AssetFilters)|En este ejemplo, se muestra cómo crear una transformación con valores preestablecidos de AdaptiveStreaming integrados, enviar un trabajo, crear un filtro de recursos y un filtro de cuentas, asociar los filtros a los localizadores de streaming e imprimir direcciones URL para la reproducción.|
| [Streaming/StreamHLSAndDASH](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/Streaming/StreamHLSAndDASH)|En este ejemplo, se muestra cómo crear una transformación con un valor preestablecido de AdaptiveStreaming integrado, enviar un trabajo y publicar un recurso de salida para la transmisión HLS y DASH.|
| [HighAvailabilityEncodingStreaming](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/HighAvailabilityEncodingStreaming/) | En este ejemplo, se proporcionan instrucciones y procedimientos recomendados para un sistema de producción con codificación o análisis a petición. Los lectores deben comenzar por el artículo complementario [Alta disponibilidad con Media Services y Vídeo bajo demanda (VOD)](media-services-high-availability-encoding.md). Se proporciona un archivo de solución independiente para el ejemplo [HighAvailabilityEncodingStreaming](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/HighAvailabilityEncodingStreaming/Readme.md). |

## <a name="nodejs"></a>[Node.JS](#tab/node/)

|Muestra|Descripción|
|---|---|
|[Hola mundo: enumeración de recursos](https://github.com/Azure-Samples/media-services-v3-node-tutorials/blob/main/AMSv3Samples/HelloWorld-ListAssets/list-assets.ts)|Ejemplo básico sobre cómo conectarse y enumerar los recursos |
|[Streaming en directo](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/main/AMSv3Samples/Live/index.ts)| Ejemplo básico de streaming en vivo. **ADVERTENCIA**: asegúrese de comprobar que todos los recursos se han limpiado y ya no se facturan en el portal al usar en producción.|
|[Carga y transmitir con HLS y DASH](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/main/AMSv3Samples/StreamFilesSample/index.ts)| Ejemplo básico para cargar un archivo local o codificación de una dirección URL de origen. En el ejemplo se muestra cómo usar el SDK de Storage para descargar contenido y se muestra cómo transmitir a un reproductor. |
|[Carga y transmisión mediante HLS y DASH con DRM de Playready y Widevine](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/main/AMSv3Samples/StreamFilesWithDRMSample/index.ts)| Muestra cómo codificar y transmitir mediante DRM de Widevine y PlayReady. |
|[Carga y uso de IA para indexar vídeos y audio](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/main/AMSv3Samples/VideoIndexerSample/index.ts)| Ejemplo de uso de los valores preestablecidos del analizador de vídeo y audio para generar metadatos e información a partir de un archivo de vídeo o audio. |

## <a name="python"></a>[Python](#tab/python)

|Muestra|Descripción|
|---|---|
|[Codificación básica con Python](https://github.com/Azure-Samples/media-services-v3-python)| Ejemplo básico para cargar un archivo local o codificación de una dirección URL de origen. En el ejemplo se muestra cómo usar el SDK de Storage para descargar contenido y se muestra cómo transmitir a un reproductor. |
|[Difuminado de caras mediante eventos y funciones](https://github.com/Azure-Samples/media-services-v3-python/tree/main/VideoAnalytics/FaceRedactorEventBased)| Este es un ejemplo de un enfoque basado en eventos que desencadena un trabajo del difuminador de caras de Azure Media Services en un vídeo en cuanto llega a una cuenta de Azure Storage. Aprovecha las ventajas de Azure Media Services, Azure Functions, Event Grid y Azure Storage para la solución. Para obtener la descripción completa de la solución, consulte el archivo [README.md](https://github.com/Azure-Samples/media-services-v3-python/blob/main/VideoAnalytics/FaceRedactorEventBased/README.md).|


## <a name="java"></a>[Java](#tab/java)

|Muestra|Descripción|
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
