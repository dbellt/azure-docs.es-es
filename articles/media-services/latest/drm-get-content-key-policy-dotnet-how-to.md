---
title: 'Obtención de una clave de firma de una directiva: .NET'
description: En este tema se muestra cómo obtener una clave de firma de la directiva existente mediante el SDK de .NET de Media Services v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
ms.date: 05/25/2021
ms.author: inhenkel
ms.custom: seodec18
ms.openlocfilehash: 373a97f26ee2fd2d448d142c89528576d3495510
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110791798"
---
# <a name="get-a-signing-key-from-the-existing-policy"></a>Obtención de una clave de firma de la directiva existente

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Uno de los principales principios de diseño de la versión v3 de la API es hacerla más segura. Las API de la versión v3 no devuelven secretos o credenciales en las operaciones **Get** o **List**. Consulte la explicación detallada aquí: Para más información, consulte [Control de acceso basado en roles (RBAC) para cuentas de Media Services](security-rbac-concept.md).

En el ejemplo de este artículo se muestra cómo usar .NET para obtener una clave de firma a partir de la directiva existente. 
 
## <a name="download"></a>Descargar 

Clone un repositorio de GitHub que contenga el ejemplo de .NET completo en la máquina mediante el siguiente comando:  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```
 
El ejemplo de ContentKeyPolicy con secretos se encuentra en la carpeta [EncryptWithDRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/tree/main/AMSV3Tutorials/EncryptWithDRM).

## <a name="get-contentkeypolicy-with-secrets"></a>Obtención de ContentKeyPolicy con secretos 

Para obtener la clave, use **GetPolicyPropertiesWithSecretsAsync**, tal y como se muestra en el ejemplo siguiente.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#GetOrCreateContentKeyPolicy)]

## <a name="next-steps"></a>Pasos siguientes

[Diseño del sistema de protección de contenido con DRM múltiple con control de acceso](architecture-design-multi-drm-system.md) 
