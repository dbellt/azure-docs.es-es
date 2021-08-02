---
title: Configuración de clientes de OpenVPN para Azure Virtual WAN
description: Aprenda a configurar clientes de OpenVPN para Azure Virtual WAN. En este artículo se incluyen los pasos de configuración de los clientes Windows, Mac, iOS y Linux.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 04/27/2021
ms.author: cherylmc
ms.openlocfilehash: 83c70aca81eaa888186807d43fff5a7bbaccb700
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110579621"
---
# <a name="configure-an-openvpn-client-for-azure-virtual-wan"></a>Configuración de un cliente de OpenVPN para Azure virtual WAN

Este artículo le ayuda a configurar clientes del **protocolo &reg;OpenVPN**. También puede usar el cliente VPN de Azure para Windows 10 para conectarse a través del protocolo OpenVPN. Para más información, consulte [Configuración de un cliente VPN para conexiones P2S OpenVPN](openvpn-azure-ad-client.md).

## <a name="before-you-begin"></a>Antes de empezar

Cree una configuración de VPN de usuario (de punto a sitio). Asegúrese de seleccionar "OpenVPN" como el tipo de túnel. Para conocer los pasos, consulte [Creación de una configuración de P2S para Azure Virtual WAN](virtual-wan-point-to-site-portal.md#p2sconfig).

[!INCLUDE [configuration steps](../../includes/vpn-gateway-vwan-config-openvpn-clients.md)]

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre VPN de usuario (de punto a sitio), consulte [Creación de conexiones VPN de usuario](virtual-wan-point-to-site-portal.md).

**"OpenVPN" es una marca comercial de OpenVPN Inc.**
