---
title: 'Administración de varios inquilinos con Azure Video Analyzer for Media (anteriormente, Video Indexer): Azure'
description: En este artículo se sugieren distintas opciones de integración para administrar varios inquilinos con Azure Video Analyzer for Media (anteriormente, Video Indexer).
services: media-services
documentationcenter: ''
author: ika-microsoft
manager: femila
editor: ''
ms.workload: ''
ms.topic: article
ms.custom: ''
ms.date: 05/15/2019
ms.author: ikbarmen
ms.openlocfilehash: 706dee5ad4b675085b481e84dd53aca4f1ba3481
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388480"
---
# <a name="manage-multiple-tenants"></a>Administración de varios inquilinos

En este artículo se abordan distintas opciones para administrar varios inquilinos con Azure Video Analyzer for Media (anteriormente, Video Indexer). Elija el método que sea más adecuado para su escenario:

* Cuenta de Video Analyzer for Media por inquilino
* Cuenta única de Video Analyzer for Media para todos los inquilinos
* Una suscripción de Azure por cada inquilino

## <a name="video-analyzer-for-media-account-per-tenant"></a>Cuenta de Video Analyzer for Media por inquilino

Cuando se usa esta arquitectura, se crea una cuenta de Video Analyzer for Media para cada inquilino. Los inquilinos tienen un aislamiento completo en el nivel persistente y en el de proceso.  

![Cuenta de Video Analyzer for Media por inquilino](./media/manage-multiple-tenants/video-indexer-account-per-tenant.png)

### <a name="considerations"></a>Consideraciones

* Los clientes no comparten cuentas de almacenamiento (a menos que el cliente las configure manualmente).
* Los clientes no comparten la capacidad de proceso (unidades reservadas) y los tiempos de procesamiento de los trabajos de otros usuarios no se ven afectados.
* Puede quitar fácilmente un inquilino del sistema mediante la eliminación de la cuenta de Video Analyzer for Media.
* No hay ninguna posibilidad de compartir modelos personalizados entre los inquilinos.

    Asegúrese de que no hay ningún requisito empresarial para compartir modelos personalizados.
* Más difícil de administrar debido a las múltiples cuentas de Video Analyzer for Media (y las instancias de Media Services asociadas) por cada inquilino.

> [!TIP]
> Cree un usuario administrador para el sistema en el [Portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/) y use la API de autorización para proporcionar a los inquilinos el correspondiente [token de acceso de la cuenta](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Get-Account-Access-Token).

## <a name="single-video-analyzer-for-media-account-for-all-users"></a>Cuenta única de Video Analyzer for Media para todos los usuarios

Cuando se usa esta arquitectura, el cliente es responsable del aislamiento de los inquilinos. Todos los inquilinos tienen que usar una única cuenta de Video Analyzer for Media con una única cuenta de Azure Media Services. Al cargar, buscar o eliminar contenido, el cliente deberá filtrar los resultados adecuados para ese inquilino.

![Cuenta única de Video Analyzer for Media para todos los usuarios](./media/manage-multiple-tenants/single-video-indexer-account-for-all-users.png)

Con esta opción, se pueden compartir los modelos de personalización (persona, idioma y marcas) o se pueden aislar entre inquilinos filtrando los modelos por inquilino.

Al [cargar vídeos](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Upload-Video), puede especificar un atributo de partición diferente por cada inquilino. Esto permitirá el aislamiento en la [API de búsqueda](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Search-Videos). Si especifica el atributo de partición en la API de búsqueda, solo obtendrá los resultados de la partición especificada. 

### <a name="considerations"></a>Consideraciones

* Posibilidad de compartir los modelos de contenido y personalización entre los inquilinos.
* Un inquilino afecta al rendimiento de otros inquilinos.
* El cliente necesita crear un nivel de administración complejo por encima de Video Analyzer for Media.

> [!TIP]
> Puede usar el atributo [priority](upload-index-videos.md) para clasificar los trabajos de los inquilinos.

## <a name="azure-subscription-per-tenant"></a>Una suscripción de Azure por cada inquilino 

Si se usa esta arquitectura, cada inquilino tiene su propia suscripción de Azure. Para cada usuario, creará una nueva cuenta de Video Analyzer for Media en la suscripción del inquilino.

![Una suscripción de Azure por cada inquilino](./media/manage-multiple-tenants/azure-subscription-per-tenant.png)

### <a name="considerations"></a>Consideraciones

* Esta es la única opción que permite la separación de la facturación.
* Esta integración tiene más sobrecarga de administración que la del escenario con la cuenta de Video Analyzer for Media por cada inquilino. Si la facturación no es un requisito, se recomienda utilizar una de las otras opciones que se describen en este artículo.

## <a name="next-steps"></a>Pasos siguientes

[Información general](video-indexer-overview.md)
