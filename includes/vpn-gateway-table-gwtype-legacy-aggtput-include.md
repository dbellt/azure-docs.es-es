---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 9734859c0bf22201c146e5d8a220f3146f6051c4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186205"
---
En la tabla siguiente se muestran los tipos de puerta de enlace y el rendimiento agregado estimado por SKU de puerta de enlace. Esta tabla se aplica a los modelos de implementación tanto clásico como Resource Manager. 

Los precios difieren entre las SKU de puerta de enlace. Para obtener más información, vea [Precios de VPN Gateway](https://azure.microsoft.com/pricing/details/vpn-gateway).

Tenga en cuenta que la SKU de la puerta de enlace de UltraPerformance no se representa en esta tabla. Para obtener información acerca de la SKU de UltraPerformance, consulte la documentación de [ExpressRoute](../articles/expressroute/expressroute-about-virtual-network-gateways.md).

|  | **Rendimiento de VPN Gateway (1)** | **Túneles IPsec máx. de VPN Gateway (2)** | **Rendimiento de puerta de enlace de ExpressRoute** | **VPN Gateway y ExpressRoute coexisten** |
| --- | --- | --- | --- | --- |
| **SKU básica (3)(5)(6)** |100 Mbps |10 |500 Mbps (6) |No |
| **SKU estándar (4)(5)** |100 Mbps |10 |1000 Mbps |Sí |
| **SKU de alto rendimiento (4)** |200 Mbps |30 |2000 Mbps |Sí |


(1) El rendimiento de la VPN es una estimación aproximada basada en las medidas entre redes virtuales en la misma región de Azure. No es un rendimiento garantizado para las conexiones entre locales a través de Internet. Es el valor máximo posible del rendimiento.

(2) El número de túneles hace referencia a VPN basadas en enrutamiento. Una VPN basada en directivas solo puede admitir un túnel VPN de sitio a sitio.

(3) BGP no es compatible con la SKU básica.

(4) Las VPN basadas en directivas no son compatibles con esta SKU. Solo son compatibles con la SKU básica.

(5) Esta SKU no admite conexiones de VPN Gateway S2S activo/activo. Activo/activo solo es compatible con la SKU HighPerformance.

(6) La SKU de nivel Básico está obsoleta para su uso con ExpressRoute.
