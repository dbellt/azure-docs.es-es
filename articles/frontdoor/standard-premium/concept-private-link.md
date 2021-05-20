---
title: Protección del origen con Private Link en Azure Front Door Estándar/Prémium (versión preliminar)
description: En esta página se proporciona información sobre cómo proteger la conectividad a su origen mediante Private Link.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.topic: conceptual
ms.date: 02/18/2021
ms.author: duau
ms.custom: references_regions
ms.openlocfilehash: 38bd02f478ebe34f3c5708e1df9cc7da1e885058
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109715221"
---
# <a name="secure-your-origin-with-private-link-in-azure-front-door-standardpremium-preview"></a>Protección del origen con Private Link en Azure Front Door Estándar/Prémium (versión preliminar)

> [!Note]
> Esta documentación trata sobre Azure Front Door Estándar/Prémium (versión preliminar). ¿Busca información sobre Azure Front Door? Vea la [documentación sobre Azure Front Door](../front-door-overview.md).

## <a name="overview"></a>Información general

[Azure Private Link](../../private-link/private-link-overview.md) le permite acceder a los servicios PaaS de Azure y a los servicios hospedados en Azure a través de un punto de conexión privado de la red virtual. El tráfico entre la red virtual y el servicio atraviesa la red troncal de Microsoft, eliminando la exposición a la red pública de Internet.

> [!IMPORTANT]
> Azure Front Door Estándar/Prémium (versión preliminar) está disponible actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

La SKU de Azure Front Door prémium puede conectarse a su origen mediante el servicio de vínculo privado. Las aplicaciones se pueden hospedar en la red virtual privada o detrás de un servicio PaaS (como una aplicación web y una cuenta de almacenamiento), lo que hace que no sea necesario que el origen sea de acceso público.

:::image type="content" source="../media/concept-private-link/front-door-private-endpoint-architecture.png" alt-text="Arquitectura de puntos de conexión privados de Front Door":::

Al habilitar Private Link para su origen en la configuración de Azure Front Door Prémium, Front Door crea un punto de conexión privado en su nombre desde la red privada regional de Front Door. Azure Front Door administra este punto de conexión. Recibirá una solicitud del punto de conexión privado de Azure Front Door para el mensaje de aprobación en el origen. Una vez aprobada la solicitud, se asigna una dirección IP privada desde la red virtual de Front Door, y el tráfico entre Azure Front Door y el origen atraviesa el vínculo privado establecido con la red troncal de Azure. El tráfico entrante a su origen está ahora protegido cuando procede de Azure Front Door.

:::image type="content" source="../media/concept-private-link/enable-private-endpoint.png" alt-text="Habilitación del punto de conexión privado":::

> [!NOTE]
> Una vez habilitado un origen de Private Link y aprobada la conexión del punto de conexión privado, la conexión tarda unos minutos en establecerse. Durante este tiempo, las solicitudes al origen recibirán un mensaje de error de Front Door. El mensaje de error desaparecerá una vez establecida la conexión.

## <a name="limitations"></a>Limitaciones

Los puntos de conexión privados de Azure Front Door están disponibles en las siguientes regiones durante la versión preliminar pública: Este de EE. UU., Oeste de EE. UU. 2, Centro y Sur de EE. UU y Sur de Reino Unido.

Para lograr la mejor latencia, siempre debe elegir una región de Azure más cercana a su origen al habilitar el punto de conexión de vínculo privado de Front Door.

Los puntos de conexión privados de Azure Front Door se administran en la plataforma y en la suscripción de Azure Front Door. Azure Front Door permite conexiones de vínculo privado a la misma suscripción de cliente que se usa para crear el perfil de Front Door.

## <a name="next-steps"></a>Pasos siguientes

* Para conectar Azure Front Door Prémium a la aplicación web mediante el servicio Private Link, vea [Conexión a una aplicación web mediante un punto de conexión privado de Azure](../../private-link/tutorial-private-endpoint-webapp-portal.md).
* Para conectar Azure Front Door Prémium a la cuenta de almacenamiento mediante el servicio Private Link, vea [Conexión a una cuenta de almacenamiento mediante un punto de conexión privado de Azure](../../private-link/tutorial-private-endpoint-storage-portal.md).
