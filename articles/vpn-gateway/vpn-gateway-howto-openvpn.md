---
title: Habilitación de OpenVPN para instancias de VPN Gateway de punto a sitio
titleSuffix: Azure VPN Gateway
description: Obtenga información sobre cómo habilitar el protocolo OpenVPN en instancias de VPN Gateway para configuraciones de punto a sitio.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 02/05/2021
ms.author: cherylmc
ms.openlocfilehash: 056e9a44009f90be23d66c5da005902ccc8ebebf
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108205438"
---
# <a name="configure-openvpn-for-point-to-site-vpn-gateways"></a>Configuración de OpenVPN para instancias de VPN Gateway de punto a sitio

En este artículo le ayudamos a configurar el **protocolo OpenVPN®** en Azure VPN Gateway. Puede usar las instrucciones del portal o de PowerShell.

## <a name="prerequisites"></a>Prerrequisitos

* Se supone que ya tiene un entorno de trabajo punto a sitio. Si no es así, cree uno con uno de los métodos siguientes.

  * [Portal: Creación de VPN de punto a sitio](vpn-gateway-howto-point-to-site-resource-manager-portal.md)

  * [PowerShell: Creación de VPN de punto a sitio](vpn-gateway-howto-point-to-site-rm-ps.md)

* Compruebe que la puerta de enlace de VPN no use la SKU de nivel Básico. La SKU de nivel Básico no es compatible con OpenVPN.

## <a name="portal"></a>Portal

1. En el portal, vaya a **Puerta de enlace de red virtual -> Configuración de punto a sitio**.
1. En **Tipo de túnel**, seleccione **OpenVPN (SSL)** en la lista desplegable.

   :::image type="content" source="./media/vpn-gateway-howto-openvpn/portal.png" alt-text="Selección de OpenVPN SSL en la lista desplegable":::
1. Guarde los cambios y continúe con los **pasos siguientes**.

## <a name="powershell"></a>PowerShell

1. Habilite OpenVPN en la puerta de enlace mediante el ejemplo siguiente:

   ```azurepowershell-interactive
   $gw = Get-AzVirtualNetworkGateway -ResourceGroupName $rgname -name $name
   Set-AzVirtualNetworkGateway -VirtualNetworkGateway $gw -VpnClientProtocol OpenVPN
   ```
1. Continúe con los **pasos siguientes**.

## <a name="next-steps"></a>Pasos siguientes

Para configurar clientes para OpenVPN, consulte [Configuración de clientes de OpenVPN](vpn-gateway-howto-openvpn-clients.md).

**"OpenVPN" es una marca comercial de OpenVPN Inc.**
