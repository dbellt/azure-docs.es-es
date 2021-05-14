---
title: archivo de inclusión
description: archivo de inclusión
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: include
ms.date: 01/12/2021
ms.author: duau
ms.custom: include file
ms.openlocfilehash: 17d08d0ffbb33d1aef35374dd5ead518b27e9ba0
ms.sourcegitcommit: 43be2ce9bf6d1186795609c99b6b8f6bb4676f47
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108284973"
---
| Resource | Límite |
| --- | --- |
| Circuitos ExpressRoute por suscripción |10 |
| Circuitos ExpressRoute por región y suscripción, con Azure Resource Manager |10 |
| Número máximo de rutas anunciadas para emparejamiento privado de Azure con ExpressRoute estándar |4\.000 |
| Número máximo de rutas anunciadas para emparejamiento privado de Azure con complemento de ExpressRoute premium |10 000 |
| Número máximo de rutas anunciadas desde el emparejamiento privado de Azure a partir del espacio de direcciones de red virtual de una conexión ExpressRoute | 1,000 |
| Número máximo de rutas anunciadas para emparejamiento de Microsoft con ExpressRoute estándar |200 |
| Número máximo de rutas anunciadas para emparejamiento de Microsoft con el complemento de ExpressRoute premium |200 |
| Número máximo de circuitos ExpressRoute vinculados a la misma red virtual en la misma ubicación de emparejamiento |4 |
| Número máximo de circuitos ExpressRoute vinculado a la misma red virtual en distintas ubicaciones de emparejamiento |16 (Para más información, consulte [SKU de puertas de enlace](../articles/expressroute/expressroute-about-virtual-network-gateways.md#aggthroughput)). |
| Número de vínculos de red virtual permitidos por circuito ExpressRoute |Consulte la tabla [Número de redes virtuales por circuito ExpressRoute](#vnetpercircuit).  |

#### <a name="number-of-virtual-networks-per-expressroute-circuit"></a><a name="vnetpercircuit"></a> Número de redes virtuales por circuito ExpressRoute
| **Tamaño del circuito** | **Número de vínculos de red virtual para estándar** | **Número de vínculos de red virtual con complemento premium** |
| --- | --- | --- |
| 50 Mbps |10 |20 |
| 100 Mbps |10 |25 |
| 200 Mbps |10 |25 |
| 500 Mbps |10 |40 |
| 1 Gbps |10 |50 |
| 2 Gbps |10 |60 |
| 5 Gbps |10 |75 |
| 10 Gbps |10 |100 |
| 40 Gbps* |10 |100 |
| 100 Gbps* |10 |100 |

**Solo ExpressRoute Direct de 100 Gbps*

> [!NOTE]
> Las conexiones de Global Reach cuentan para el límite de conexiones de red virtual por circuito ExpressRoute. Por ejemplo, un circuito Premium de 10 Gbps permitiría 5 conexiones de Global Reach y 95 conexiones a las puertas de enlace de ExpressRoute, o bien 95 conexiones de Global Reach y 5 conexiones a las puertas de enlace de ExpressRoute, o cualquier otra combinación hasta alcanzar el límite de 100 conexiones para el circuito.
