---
author: alexeyo26
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 03/19/2021
ms.author: alexeyo
ms.openlocfilehash: 8a1253e7ac88d2fe8b557c48dd846b48de59fba3
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "110059219"
---
## <a name="simultaneous-use-of-private-endpoints-and-vnet-service-endpoints"></a>Uso simultáneo de puntos de conexión privados y puntos de conexión de servicio de red virtual

Los [puntos de conexión privados](../speech-services-private-link.md) y los [puntos de conexión de servicio de red virtual](../speech-service-vnet-service-endpoint.md) se pueden usar para acceder al mismo recurso de voz simultáneamente. Sin embargo, para habilitar los puntos de conexión privados y los puntos de conexión de servicio de red virtual simultáneamente, debe usar la opción **Redes y puntos de conexión privados seleccionados** en la configuración de red del recurso de voz en Azure Portal. El resto de opciones no se admiten en este escenario.
