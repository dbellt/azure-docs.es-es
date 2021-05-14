---
title: Configuración de clientes de OpenVPN para Azure Virtual WAN
description: Este artículo lo ayuda a configurar clientes de OpenVPN para Azure Virtual WAN. Incluye los pasos de configuración de los clientes Windows, Mac, iOS y Linux.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 04/27/2021
ms.author: cherylmc
ms.openlocfilehash: 91b4a9a69912f7a5980348cc2b9a874673fa0eb2
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108163398"
---
# <a name="configure-an-openvpn-client-for-azure-virtual-wan"></a>Configuración de un cliente de OpenVPN para Azure virtual WAN

Este artículo le ayuda a configurar clientes del **protocolo &reg;OpenVPN**. También puede usar el cliente VPN de Azure para Windows 10 para conectarse a través del protocolo OpenVPN. Si necesita más instrucciones, las encontrará [aquí](openvpn-azure-ad-client.md).

## <a name="before-you-begin"></a>Antes de empezar

Cree una configuración de VPN de usuario (de punto a sitio). Asegúrese de seleccionar "OpenVPN" como el tipo de túnel. Para conocer los pasos, consulte [Creación de una configuración de P2S para Azure Virtual WAN](virtual-wan-point-to-site-portal.md#p2sconfig).

[!INCLUDE [configuration steps](../../includes/vpn-gateway-vwan-config-openvpn-clients.md)]

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre VPN de usuario (de punto a sitio), consulte [Creación de conexiones VPN de usuario](virtual-wan-point-to-site-portal.md).

**"OpenVPN" es una marca comercial de OpenVPN Inc.**
