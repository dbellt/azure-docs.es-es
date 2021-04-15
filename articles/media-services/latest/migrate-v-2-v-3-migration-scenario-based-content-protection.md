---
title: Guía de migración de protección de contenido
description: En este artículo se proporcionan instrucciones basadas en escenarios de protección de contenido que le ayudarán en la migración de Azure Media Services v2 a v3.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: conceptual
ms.workload: media
ms.date: 04/05/2021
ms.author: inhenkel
ms.openlocfilehash: a481759da3f1e7d67accdca7b4322db53abbcb0c
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106490954"
---
# <a name="content-protection-scenario-based-migration-guidance"></a>Guía de migración basada en escenarios de protección de contenido

![logotipo de la guía de migración](./media/migration-guide/azure-media-services-logo-migration-guide.svg)

<hr color="#5ea0ef" size="10">

![pasos de migración 2](./media/migration-guide/steps-4.svg)

En este artículo se proporcionan detalles e instrucciones sobre la migración de casos de uso de protección de contenido de la API v2 a la nueva API de Azure Media Services v3.

## <a name="protect-content-in-v3-api"></a>Protección del contenido en API v3

Use la compatibilidad con las características de [varias claves](architecture-design-multi-drm-system.md) en la nueva API v3.

Consulte los conceptos, tutoriales y guías de procedimientos de protección de contenido al final de este artículo para conocer los pasos específicos.

> [!NOTE]
> En el resto de este artículo se describe cómo migrar la protección de contenido de la versión v2 a la v3 con .NET.  Si necesita instrucciones o código de ejemplo para un lenguaje o método diferentes, cree un problema de GitHub para esta página.

## <a name="v3-visibility-of-v2-assets-streaminglocators-and-properties"></a>Visibilidad en la versión v3 de Assets, StreamingLocators y propiedades de la versión v2

En la API v2, `Assets`, `StreamingLocators` y `ContentKeys` se usaban para proteger el contenido de streaming. Al migrar a la API v3, los elementos `Assets` `StreamingLocators` y `ContentKeys` de la API v2 se exponen de forma automática en la API v3, y todos los datos de los mismos están disponibles para que pueda acceder a ellos.

Sin embargo, no puede *actualizar* las propiedades de las entidades de la versión v2 que se crearon en dicha versión a través de la API v3.

Si necesita actualizar, cambiar o modificar el contenido almacenado en entidades de la versión v2, actualícelo con la API v2 o cree nuevas entidades de API v3 para migrarlas.

## <a name="asset-identifier-differences"></a>Diferencias en los identificadores de Asset

Para realizar la migración, necesitará acceder a las propiedades o claves de contenido de los elementos Asset de la versión v2.  Es importante comprender que la API v2 usa el valor de `AssetId` como clave de identificación principal, pero la nueva API v3 usa el *nombre de administración de recursos de Azure* de la entidad como identificador principal.  (La propiedad `Asset.Name` de la versión v2 no se usa como identificador único). Con la API v3, el nombre del elemento Asset de la versión v2 aparece como `Asset.Description`.

Por ejemplo, si anteriormente tenía un elemento Asset de la versión v2 con el identificador `nb:cid:UUID:8cb39104-122c-496e-9ac5-7f9e2c2547b8`, el identificador se encuentra ahora al final del GUID `8cb39104-122c-496e-9ac5-7f9e2c2547b8`. Podrá ver esto cuando enumere los elementos Asset de la versión v2 a través de la API v3.

Los elementos Asset que se crearon y publicaron con la API v2 tendrán tanto una propiedad `ContentKeyPolicy` como una propiedad `ContentKey` en la API v3, en lugar de una directiva de clave de contenido predeterminada en `StreamingPolicy`.

Para obtener más información, consulte la documentación de la [Directiva de clave de contenido](https://docs.microsoft.com/azure/media-services/latest/drm-content-key-policy-concept) y la documentación de la [Directiva de streaming](https://docs.microsoft.com/azure/media-services/latest/stream-streaming-policy-concept).

## <a name="use-azure-media-services-explorer-amse-v2-and-amse-v3-tools-side-by-side"></a>Uso de las herramientas de Azure Media Services Explorer v2 (AMSE) y AMSE v3 en paralelo

Use la [herramienta Azure Media Services Explorer v2](https://github.com/Azure/Azure-Media-Services-Explorer/releases/tag/v4.3.15.0) junto con la [herramienta Azure Media Services Explorer v3](https://github.com/Azure/Azure-Media-Services-Explorer) para comparar en paralelo los datos de un elemento Asset creado y publicado a través de las API v2. Todas las propiedades deben ser visibles, aunque en ubicaciones diferentes.

## <a name="use-the-net-content-protection-migration-sample"></a>Uso del ejemplo de migración de protección de contenido de .NET

Puede consultar un ejemplo de código para comparar las diferencias en los identificadores de elementos Asset en [v2tov3MigrationSample](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/ContentProtection/v2tov3Migration), en ContentProtection de los ejemplos de código de Media Services.

## <a name="list-the-streaming-locators"></a>Lista de localizadores de streaming

Puede consultar los `StreamingLocators` asociados a los elementos Asset creados en la API v2 mediante el nuevo método [ListStreamingLocators](https://docs.microsoft.com/rest/api/media/assets/liststreaminglocators)de la versión v3 en la entidad Asset.  También debe hacer referencia a la versión del SDK de cliente de .NET de [ListStreamingLocatorsAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.media.assetsoperationsextensions.liststreaminglocatorsasync?view=azure-dotnet&preserve-view=true)

Los resultados del método `ListStreamingLocators` le proporcionarán las propiedades `Name` y `StreamingLocatorId` del localizador, junto con `StreamingPolicyName`.

## <a name="find-the-content-keys"></a>Búsqueda de las claves de contenido

Para encontrar la propiedad `ContentKeys` usada con `StreamingLocators`, puede llamar al método [StreamingLocator.ListContentKeysAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.media.streaminglocatorsoperationsextensions.listcontentkeysasync?view=azure-dotnet&preserve-view=true).  

Para obtener más información sobre la protección de contenido en la API v3, consulte el artículo [Protección del contenido mediante el cifrado dinámico de Media Services](https://docs.microsoft.com/azure/media-services/latest/drm-content-protection-concept).

## <a name="change-the-v2-contentkeypolicy-keeping-the-same-contentkey"></a>Cambio de la ContentKeyPolicy v2 mientras se conserva la misma ContentKey

Primero debe anular la publicación (quitar todos los localizadores de streaming) en el elemento Asset a través del SDK v2. A continuación, se indica cómo puede hacerlo.

1. Elimine el localizador.
1. Desvincule `ContentKeyAuthorizationPolicy`.
1. Desvincule `AssetDeliveryPolicy`.
1. Desvincule `ContentKey`.
1. Elimine `ContentKey`.
1. Cree un nuevo `StreamingLocator` en la versión v3 con una propiedad `StreamingPolicy` y `ContentKeyPolicy` e la versión v3. Para ello, especifique los valores necesarios de identificador de clave de contenido y valor de clave.

> [!NOTE]
> Es posible eliminar el localizador v2 mediante la API v3, pero esto no quitará la clave de contenido ni la directiva de clave de contenido si se crearon en la API v2.

## <a name="content-protection-concepts-tutorials-and-how-to-guides"></a>Conceptos, tutoriales y guías de procedimientos de protección de contenido

### <a name="concepts"></a>Conceptos

- [Protección del contenido mediante el cifrado dinámico de Media Services](drm-content-protection-concept.md)
- [Diseño del sistema de protección de contenido con DRM múltiple con control de acceso](architecture-design-multi-drm-system.md)
- [Media Services v3 con plantilla de licencia de PlayReady](drm-playready-license-template-concept.md)
- [Introducción a Media Services v3 con plantilla de licencia de Widevine](drm-widevine-license-template-concept.md)
- [Configuración y requisitos de licencia de FairPlay de Apple](drm-fairplay-license-overview.md)
- [Directivas de streaming](stream-streaming-policy-concept.md)
- [Directivas de claves de contenido](drm-content-key-policy-concept.md)

### <a name="tutorials"></a>Tutoriales

[Inicio rápido: Uso del portal para cifrar contenido](drm-encrypt-content-how-to.md)

### <a name="how-to-guides"></a>Guías de procedimientos

- [Obtención de una clave de firma de la directiva existente](drm-get-content-key-policy-dotnet-how-to.md)
- [Streaming de FairPlay sin conexión para iOS con Media Services v3](drm-offline-fairplay-for-ios-concept.md)
- [Streaming de Widevine sin conexión para Android con Media Services v3](drm-offline-widevine-for-android.md)
- [Streaming de PlayReady sin conexión para Windows 10 con Media Services v3](drm-offline-playready-streaming-for-windows-10.md)

## <a name="samples"></a>Ejemplos

- [v2tov3MigrationSample](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/ContentProtection/v2tov3Migration)
- También puede [comparar el código de la versión v2 y v3 en los ejemplos de código](migrate-v-2-v-3-migration-samples.md).

## <a name="tools"></a>Herramientas

- [Herramienta Azure Media Services Explorer v3](https://github.com/Azure/Azure-Media-Services-Explorer)
- [Herramienta Azure Media Services Explorer v2](https://github.com/Azure/Azure-Media-Services-Explorer/releases/tag/v4.3.15.0)
