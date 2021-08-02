---
title: 'Escenario: universal'
titleSuffix: Azure Virtual WAN
description: Obtenga información sobre los escenarios de enrutamiento de tipo "any-to-any" de Virtual WAN, donde cualquier radio puede comunicarse con otro radio.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 04/27/2021
ms.author: cherylmc
ms.custom: fasttrack-edit
ms.openlocfilehash: b27d1327eac8e108c462fd3c0f19a257a5385428
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110575996"
---
# <a name="scenario-any-to-any"></a>Escenario: Universal

Al trabajar con el enrutamiento de centros virtuales de Virtual WAN, hay bastantes escenarios disponibles. En un escenario universal, cualquier radio puede comunicarse con otro radio. Cuando existen varios centros, el enrutamiento de centro a centro (también conocido como hub-to-hub o interhub) está habilitado de forma predeterminada en Virtual WAN Estándar. Puede crear esta configuración mediante diferentes métodos, como Azure Portal o una [plantilla de inicio rápido de Azure](quickstart-any-to-any-template.md). Para obtener más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md). 

## <a name="design"></a><a name="design"></a>Diseño

Para averiguar el número de tablas de rutas que se necesitarán en un escenario de WAN virtual, puede crear una matriz de conectividad, donde cada celda representa si un origen (fila) puede comunicarse con un destino (columna).

| De |   A |  *Redes virtuales* | *Ramas* |
| -------------- | -------- | ---------- | ---|
| Redes virtuales     | &#8594;| Directo | Directo |
| Ramas   | &#8594;| Directo  | Directo |

En cada una de las celdas de la tabla anterior se describe si una conexión de Virtual WAN (el lado "De" del flujo, los encabezados de fila) se comunica con un prefijo de destino (el lado "A" del flujo, los encabezados de columna en cursiva). En este escenario no hay ningún firewall o dispositivo virtual de red, por lo que la comunicación fluye directamente a través de Virtual WAN (de ahí la palabra "Directo" en la tabla).

Dado que todas las conexiones de redes virtuales y ramas (VPN, ExpressRoute y VPN de usuario) tienen los mismos requisitos de conectividad, se requiere una sola tabla de rutas. Como resultado, todas las conexiones se asociarán y se propagarán a la misma tabla de rutas, la tabla de rutas predeterminada:

* Redes virtuales:
  * Tabla de rutas asociada: **Valor predeterminado**
  * Propagación a tablas de rutas: **Valor predeterminado**
* Ramas:
  * Tabla de enrutamiento asociada: **Valor predeterminado**
  * Propagación a tablas de rutas: **Valor predeterminado**

Para obtener más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

## <a name="architecture"></a><a name="architecture"></a>Arquitectura

En la **Ilustración 1**, todas las redes virtuales y ramas (VPN, ExpressRoute, P2S) pueden comunicarse entre sí. En un centro virtual, las conexiones funcionan de la siguiente manera:

* Una conexión VPN conecta un sitio VPN a una puerta de enlace de VPN.
* La conexión de red virtual conecta una red virtual a un centro virtual. El enrutador del centro virtual proporciona la funcionalidad de tránsito entre las redes virtuales.
* Una conexión ExpressRoute conecta un circuito de ExpressRoute a una puerta de enlace de ExpressRoute.

Estas conexiones (de forma predeterminada en la creación) están asociadas a la tabla de rutas predeterminada, a menos que defina la configuración de enrutamiento de la conexión en **Ninguno** o en una tabla de rutas personalizada. De forma predeterminada, estas conexiones también se propagan a la tabla de rutas predeterminada. Esto es lo que permite un escenario universal, en el que cualquier radio (red virtual, VPN, ER, P2S) puede comunicarse con cualquier otro.

**Ilustración 1**

:::image type="content" source="./media/routing-scenarios/any-any/figure-1.png" alt-text="ilustración 1":::

## <a name="workflow"></a><a name="workflow"></a>Flujo de trabajo

Este escenario está habilitado de forma predeterminada para Virtual WAN Estándar. Si la configuración de rama a rama está deshabilitada en la configuración de WAN, no se permitirá la conectividad entre los radios de la rama. VPN/ExpressRoute/VPN de usuario se consideran radios de ramas en Virtual WAN.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre Virtual WAN, consulte las [preguntas más frecuentes](virtual-wan-faq.md).
* Para obtener más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).
