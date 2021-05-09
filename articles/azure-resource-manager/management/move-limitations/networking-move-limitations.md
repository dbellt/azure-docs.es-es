---
title: Traslado de recursos de red de Azure a una suscripción o un grupo de recursos nuevos
description: Use Azure Resource Manager para trasladar redes virtuales y otros recursos de red a un nuevo grupo de recursos o a una nueva suscripción.
ms.topic: conceptual
ms.date: 10/16/2019
ms.openlocfilehash: 0ac503a778b3de8945b074628242233f9bcd6a3b
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108129970"
---
# <a name="move-guidance-for-networking-resources"></a>Directrices para el traslado de recursos de red

En este artículo, se explica cómo trasladar las redes virtuales y otros recursos de red en escenarios específicos.

## <a name="dependent-resources"></a>Recursos dependientes

> [!NOTE]
> Tenga en cuenta que las puertas de enlace de VPN asociadas con direcciones de SKU estándar de IP públicas no pueden moverse entre grupos de recursos o suscripciones.

Al mover un recurso, también debe mover sus recursos dependientes (por ejemplo, direcciones IP públicas, puertas de enlace de red virtual, todos los recursos de conexión asociados). Las puertas de enlace de red local pueden estar en otro grupo de recursos.

Para mover una máquina virtual con una tarjeta adaptadora de red a una suscripción nueva, debe mover también todos los recursos dependientes. Mueva la red virtual de la tarjeta de interfaz de red, todas las demás tarjetas de interfaz de la red virtual y las puertas de enlace VPN.

Para más información, consulte [Escenario para el traslado entre suscripciones](../move-resource-group-and-subscription.md#scenario-for-move-across-subscriptions).

## <a name="peered-virtual-network"></a>Red virtual emparejada

Para mover una red virtual emparejada, primero debe deshabilitar el emparejamiento de red virtual. Una vez deshabilitado, puede mover la red virtual. Después de moverla, vuelva a habilitar el emparejamiento de red virtual.

## <a name="subnet-links"></a>Vínculos de subredes

Si una red virtual contiene una subred con vínculos de navegación de los recursos, no se puede mover a otra suscripción. Por ejemplo, si un recurso de Azure Cache for Redis está implementado en una subred, esta contiene un vínculo de navegación de recursos.

## <a name="next-steps"></a>Pasos siguientes

Para ver los comandos para trasladar recursos, vea [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../move-resource-group-and-subscription.md).
