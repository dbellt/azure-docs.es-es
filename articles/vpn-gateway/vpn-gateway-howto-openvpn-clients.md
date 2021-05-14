---
title: Configuración de clientes OpenVPN para VPN Gateway de punto a sitio
titleSuffix: Azure VPN Gateway
description: Obtenga información sobre cómo configurar los clientes OpenVPN para Azure VPN Gateway. Este artículo le ayuda a configurar clientes Windows, Linux, iOS y Mac.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 04/29/2021
ms.author: cherylmc
ms.openlocfilehash: 5a7b125ef89d5d598e22fc4458b5b1c7f8c4a41c
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108289361"
---
# <a name="configure-openvpn-clients-for-azure-vpn-gateway"></a>Configuración de los clientes OpenVPN de Azure VPN Gateway

Este artículo le ayuda a configurar clientes del **protocolo &reg;OpenVPN**.

## <a name="before-you-begin"></a>Antes de empezar

Compruebe que ha completado los pasos para configurar OpenVPN para VPN Gateway. Para obtener más información, consulte [Configuración de los clientes OpenVPN de Azure VPN Gateway](vpn-gateway-howto-openvpn.md).

[!INCLUDE [configuration steps](../../includes/vpn-gateway-vwan-config-openvpn-clients.md)]

## <a name="next-steps"></a>Pasos siguientes

Si quiere que los clientes VPN tengan acceso a recursos de otra red virtual, siga las instrucciones del artículo [De red virtual a red virtual](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md) para configurar una conexión de red virtual a red virtual. Asegúrese de habilitar BGP en las puertas de enlace y las conexiones; en caso contrario, el tráfico no fluirá.

**"OpenVPN" es una marca comercial de OpenVPN Inc.**
