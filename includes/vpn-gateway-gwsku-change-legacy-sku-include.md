---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/15/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 4c232e1ce183c6935d625b5bc9987a4981865ae4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96010842"
---
Si está trabajando con el modelo de implementación de Resource Manager, puede cambiar a las nuevas SKU de puerta de enlace. Al cambiar de una SKU de puerta de enlace heredada a una nueva SKU, se elimina la puerta de enlace de la red virtual existente y se crea una nueva.

Flujo de trabajo:

1. Elimine las conexiones a la puerta de enlace de la red virtual.
2. Elimine la puerta de enlace VPN Gateway antigua.
3. Cree la puerta de enlace VPN Gateway nueva.
4. Actualice los dispositivos VPN locales con la nueva dirección IP de la puerta de enlace VPN Gateway (para las conexiones de sitio a sitio).
5. Actualice el valor de la dirección IP de puerta de enlace para las puertas de enlace de red local de red virtual a red virtual que se conectarán a esta puerta de enlace.
6. Descargue los nuevos paquetes de configuración de VPN de cliente para los clientes de P2S que se vayan a conectar a la red virtual a través de esta puerta de enlace VPN Gateway.
7. Vuelva a crear las conexiones a la puerta de enlace de la red virtual.

Consideraciones:

* Para pasar a las nuevas SKU, la puerta de enlace de la red virtual debe estar en el modelo de implementación de Resource Manager.
* Si tiene una puerta de enlace VPN clásica, debe seguir utilizando las SKU heredadas más antiguas para esa puerta de enlace; sin embargo, puede cambiar el tamaño entre las SKU heredadas. No se puede cambiar as las nuevas SKU.
* Tendrá un tiempo de inactividad de conectividad cuando cambie de una SKU heredada a una nueva.
* Al cambiar a una nueva SKU de puerta de enlace, cambiará la dirección IP pública de la puerta de enlace de la red virtual. Esto sucede incluso si especifica el mismo objeto de dirección IP pública que usó anteriormente.