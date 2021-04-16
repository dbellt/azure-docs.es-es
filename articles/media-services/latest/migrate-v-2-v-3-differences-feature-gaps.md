---
title: Diferencias entre características de Azure Media Services v2 y v3
description: En este artículo se describen las diferencias de características entre Azure Media Services v2 y v3.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.devlang: multiple
ms.topic: conceptual
ms.tgt_pltfrm: multiple
ms.workload: media
ms.date: 03/25/2021
ms.author: inhenkel
ms.openlocfilehash: 564f3127fc6901695890daa520152a7aa1a2337f
ms.sourcegitcommit: edc7dc50c4f5550d9776a4c42167a872032a4151
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105962975"
---
# <a name="feature-gaps-between-azure-media-services-v2-and-v3"></a>Diferencias entre características de Azure Media Services v2 y v3

![logotipo de la guía de migración](./media/migration-guide/azure-media-services-logo-migration-guide.svg)

<hr color="#5ea0ef" size="10">

![pasos de migración 2](./media/migration-guide/steps-2.svg)

En esta parte de la guía de migración se ofrece información detallada sobre las diferencias entre las API v2 y v3.

## <a name="feature-gaps-between-v2-and-v3-apis"></a>Diferencias entre características de las API v2 y v3

La API v3 tiene las siguientes diferencias entre características con respecto a la API v2. Algunas de las características avanzadas de Media Encoder Standard de las API v2 actualmente no están disponibles en v3, como:

- Inserción de una pista de audio silenciosa cuando la entrada no tiene audio, ya que esto ya no es necesario con Azure Media Player.

- Inserción de una pista de vídeo cuando la entrada no tiene vídeo.

- Los eventos en directo con transcodificación actualmente no admiten la inserción de caretas durante la transmisión y la inserción de anuncios mediante una llamada API.

- Azure Media Encoder Premium ya no se admitirá en v2. Si está usando la codificación de HEVC de 8 bits, use la nueva compatibilidad con HEVC en el codificador Standard. 
    - Hemos agregado compatibilidad con la asignación de canales de audio al codificador Standard.  Consulte la sección sobre [audio en la documentación de codificación en Media Services de Swagger](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/mediaservices/resource-manager/Microsoft.Media/stable/2020-05-01/Encoding.json).
    - Si usaba características o formatos de salida avanzadas de un producto con licencia de terceros, como MXF o ProRes, use la solución de asociado de Azure de Telestream, que será transaccional en el momento en que se retire la versión v2. También puede usar Imagine Communications o [Bitmovin](http://bitmovin.com).

- Ya no se admite la propiedad "availability set" en el punto de conexión de streaming de v2. Consulte el proyecto de ejemplo y la guía para lograr la entrega de [VoD de alta disponibilidad](./architecture-high-availability-encoding-concept.md) en la API v3.

- En Media Services v3, no se puede especificar el vector de inicialización de FairPlay. Aunque no afecta a los clientes que usan Media Services tanto para el empaquetado como para la entrega de licencias, puede ser un problema al usar un sistema DRM de terceros para entregar las licencias de FairPlay (modo híbrido).

- El cifrado de almacenamiento del lado cliente para la protección de recursos en reposo se eliminó de la API v3 y se reemplazó por el cifrado del servicio Storage para datos en reposo. Las API v3 siguen funcionando con los recursos cifrados de almacenamiento ya existentes, pero no permitirán la creación de otros.

## <a name="terminology-and-entity-changes"></a>Cambios de terminología y de entidad

Consulte los cambios de [terminología y entidades](migrate-v-2-v-3-differences-terminology.md) para obtener más cambios en la API.
