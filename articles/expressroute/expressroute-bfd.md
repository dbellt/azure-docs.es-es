---
title: 'Azure ExpressRoute: Configuración de BFD'
description: En este artículo se proporcionan instrucciones sobre cómo configurar BFD (detección de reenvío bidireccional) sobre el emparejamiento privado de un circuito ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: article
ms.date: 12/14/2020
ms.author: duau
ms.openlocfilehash: 1ad91b00c14744ab1f0cbb414defeaffb85559fe
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108127807"
---
# <a name="configure-bfd-over-expressroute"></a>Configuración de BFD a través de ExpressRoute

ExpressRoute admite la detección de reenvío bidireccional (BFD) a través del emparejamiento privado y el emparejamiento de Microsoft. Al habilitar BFD en ExpressRoute, puede acelerar la detección de errores de los vínculos entre los dispositivos perimetrales de Microsoft Enterprise (MSEE) y los enrutadores que el circuito de ExpressRoute haya configurado (CE/PE). Puede configurar ExpressRoute a través de los dispositivos de enrutamiento perimetrales o los dispositivos de enrutamiento perimetrales asociados (si ha optado por usar el servicio administrado de conexión de nivel 3). Este documento explica en detalle la necesidad de BFD y cómo habilitar BFD a través de ExpressRoute.

## <a name="need-for-bfd"></a>Necesidad de BFD

El siguiente diagrama muestra la ventaja de habilitar BFD a través del circuito de ExpressRoute: [![1]][1]

Puede habilitar el circuito de ExpressRoute mediante conexiones de nivel 2 o de nivel 3 administradas. En ambos casos, si hay más de un dispositivo de nivel 2 en la ruta de acceso de conexión de ExpressRoute, la responsabilidad de detectar los errores de los vínculos en la ruta de acceso es de la sesión BGP superpuesto.

En los dispositivos MSEE, el protocolo de puerta de enlace de borde para mantener la conexión y de tiempo de espera, generalmente se configura en 60 y 180 segundos respectivamente. Por esa razón, cuando se produce un error de vínculo, el proceso puede tardar hasta tres minutos en detectar cualquier error de vínculo y cambiar el tráfico a una conexión alternativa.

Puede controlar los temporizadores de BGP mediante una configuración con valores inferiores de BGP para mantener la conexión y de tiempo de espera en el dispositivo de emparejamiento perimetral. Si los temporizadores de BGP no son los mismos entre los dos dispositivos de emparejamiento, la sesión BGP se establecerá con el valor de hora más bajo. El tipo de BGP para mantener la conexión se puede establecer con un valor tan bajo como tres segundos, y el de tipo de tiempo de espera, en unos diez segundos. Sin embargo, no se recomienda establecer un temporizador de BGP demasiado agresivo, ya que el protocolo requiere un uso intensivo del proceso.

En este escenario, BFD puede ayudar. BFD proporciona detección de errores de vínculo de baja sobrecarga en un intervalo de tiempo de subsegundos. 


## <a name="enabling-bfd"></a>Habilitación del BFD

BFD está configurado de forma predeterminada en todas las recién creadas interfaces de emparejamiento privadas de ExpressRoute en los MSEE. Por lo tanto, para habilitar BFD, solo tiene que configurar BFD en los dispositivos principal y secundario. Tenga en cuenta que la configuración de BFD es un proceso de dos pasos. Tiene que configurar BFD en la interfaz y luego vincularlo a la sesión de BGP.

A continuación se muestra un ejemplo de configuración de CE/PE (con Cisco IOS XE). 

```console
interface TenGigabitEthernet2/0/0.150
   description private peering to Azure
   encapsulation dot1Q 15 second-dot1q 150
   ip vrf forwarding 15
   ip address 192.168.15.17 255.255.255.252
   bfd interval 300 min_rx 300 multiplier 3


router bgp 65020
   address-family ipv4 vrf 15
      network 10.1.15.0 mask 255.255.255.128
      neighbor 192.168.15.18 remote-as 12076
      neighbor 192.168.15.18 fall-over bfd
      neighbor 192.168.15.18 activate
      neighbor 192.168.15.18 soft-reconfiguration inbound
   exit-address-family
```

>[!NOTE]
>Para habilitar BFD en un emparejamiento privado ya existente; tiene que restablecer el emparejamiento. Consulte [Restablecimiento de emparejamientos de ExpressRoute][ResetPeering].
>

## <a name="bfd-timer-negotiation"></a>Negociación de temporizador BFD

Entre pares BFD, el más lento de los dos pares determina la velocidad de transmisión. Los intervalos de transmisión y recepción de los MSEE BFD se establecen en 300 milisegundos. En determinados escenarios, el intervalo puede establecerse en un valor superior a 750 milisegundos. Al configurar un valor mayor, puede forzar que estos intervalos sean más largos, pero no es posible hacerlos más cortos.

>[!NOTE]
>Si ha configurado circuitos de ExpressRoute con redundancia geográfica o usa la conectividad VPN de IPSec de sitio a sitio como copia de seguridad, habilitar BFD le permitirá realizar la conmutación por error más rápido después de que se haya producido un error de conectividad de ExpressRoute. 
>

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte los siguientes artículos:

- [Creación y modificación de un circuito ExpressRoute][CreateCircuit]
- [Creación y modificación del enrutamiento de un circuito ExpressRoute][CreatePeering]

<!--Image References-->
[1]: ./media/expressroute-bfd/bfd-need.png "BFD acelera el tiempo de deducción de errores de vínculo"

<!--Link References-->
[CreateCircuit]: ./expressroute-howto-circuit-portal-resource-manager.md
[CreatePeering]: ./expressroute-howto-routing-portal-resource-manager.md
[ResetPeering]: ./expressroute-howto-reset-peering.md