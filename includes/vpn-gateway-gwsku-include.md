---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 11/12/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 8087025810214f3edbb74e628698eb69558f3500
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96010869"
---
Al crear una puerta de enlace de red virtual, debe especificar la SKU de la puerta de enlace que desea usar. Seleccione las SKU que cumplan sus requisitos en función de los tipos de cargas de trabajo, rendimientos, características y Acuerdos de Nivel de Servicio. En relación con las SKU de puerta de enlace de red virtual en Azure Availability Zones, vea la información sobre las [SKU de puerta de enlace de Azure Availability Zones](../articles/vpn-gateway/about-zone-redundant-vnet-gateways.md).

###  <a name="gateway-skus-by-tunnel-connection-and-throughput"></a><a name="benchmark"></a>SKU de puerta de enlace por túnel, conexión y rendimiento

[!INCLUDE [Aggregated throughput by SKU](./vpn-gateway-table-gwtype-aggtput-include.md)]

> [!NOTE]
> Las SKU de VpnGw (VpnGw1, VpnGw1AZ, VpnGw2, VpnGw2AZ, VpnGw3, VpnGw3AZ, VpnGw4, VpnGw4AZ, VpnGw5 y VpnGw5AZ) se admiten para el modelo de implementación de Resource Manager únicamente. Las redes virtuales clásicas deben seguir utilizando las SKU antiguas (heredadas).
>  * Para obtener información sobre cómo trabajar con SKU de puerta de enlace heredadas (Basic, Standard, y HighPerformance), consulte [Trabajo con SKU de puerta de enlace de red virtual (SKU antiguas)](../articles/vpn-gateway/vpn-gateway-about-skus-legacy.md).
>  * Para las SKU de puerta de enlace de ExpressRoute, consulte [Acerca de las puertas de enlace de red virtual para ExpressRoute](../articles/expressroute/expressroute-about-virtual-network-gateways.md).
>

###  <a name="gateway-skus-by-feature-set"></a><a name="feature"></a>SKU de puerta de enlace por conjunto de características

Las nueva SKU de puerta de enlace de VPN simplifican los conjuntos de características que se ofrecen en las puertas de enlace:

| **SKU**| **Características**|
| ---    | ---         |
|**Basic** (\*\*)   | **VPN basada en ruta**: 10 túneles para conexiones o S2S, sin autenticación RADIUS para P2S, sin IKEv2 para P2S<br>**VPN basada en directivas**: (IKEv1) 1 túnel de S2S o conexión, sin P2S|
| **Todas las SKU de Generation1 y Generation2, excepto Basic** | **VPN basada en ruta**: hasta 30 túneles (\*), P2S, BGP, activo-activo, directiva de IPsec/IKE personalizada, coexistencia de VPN y ExpressRoute |
|        |             |

(*) Se puede configurar "PolicyBasedTrafficSelectors" para conectar una instancia de VPN Gateway basada en la ruta a varios dispositivos de firewall locales basados en directivas. Consulte [Connect VPN gateways to multiple on-premises policy-based VPN devices using PowerShell](../articles/vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps.md) (Conexión de puertas de enlace VPN Gateway a varios dispositivos de VPN locales basados en directivas con PowerShell) para más información.

(\*\*) La SKU Basic se considera una SKU heredada. La SKU Basic tiene ciertas limitaciones de características. No se puede cambiar el tamaño de una puerta de enlace que utiliza una SKU Basic a una de las SKU de puerta de enlace nuevas, sino que debe cambiar a una SKU nueva, lo que implica eliminar y volver a crear la puerta de enlace VPN.

###  <a name="gateway-skus---production-vs-dev-test-workloads"></a><a name="workloads"></a>SKU de puerta de enlace: carga de trabajo de producción frente a cargas de desarrollo y pruebas

Dadas las diferencias en los Acuerdos de Nivel de Servicio y los conjuntos de características, se recomiendan las siguientes SKU para la producción, en comparación el desarrollo y las pruebas:

| **Carga de trabajo**                       | **SKU**               |
| ---                                | ---                    |
| **Cargas de trabajo de producción, críticas** | Todas las SKU de Generation1 y Generation2, excepto Basic |
| **Desarrollo y pruebas o prueba de concepto**   | Basic (\*\*)                 |
|                                    |                        |

(\*\*) La SKU Basic se considera una SKU heredada y tiene limitaciones de características. Compruebe que se admite la característica que necesita antes de usar la SKU Basic.

Si continúa utilizando las SKU antiguas (heredadas), las recomendaciones de SKU de producción son Standard y HighPerformance. Para obtener información e instrucciones para las SKU antiguas, consulte [Trabajo con SKU de puerta de enlace de red virtual (SKU antiguas)](../articles/vpn-gateway/vpn-gateway-about-skus-legacy.md).
