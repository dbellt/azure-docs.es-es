---
title: 'Azure VPN Gateway: Administración de sesiones de VPN de punto a sitio'
description: Este artículo le ayuda a ver y desconectar sesiones VPN de punto a sitio.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 04/26/2021
ms.author: cherylmc
ms.openlocfilehash: 0b0a4c712333516dcdd20dcde9ef4b9c74b5f228
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108018662"
---
# <a name="point-to-site-vpn-session-management"></a>Administración de sesiones de VPN de punto a sitio

Las puertas de enlace de red virtual de Azure proporcionan una manera fácil de ver y desconectar las sesiones actuales de VPN de punto a sitio. Este artículo le ayuda a ver y desconectar las sesiones actuales. El estado de la sesión se actualiza cada cinco minutos. No lo hace inmediatamente. 


## <a name="portal"></a>Portal

>[!NOTE]
> La información del origen de conexión se proporciona solo para conexiones IKEv2 y OpenVPN.
> 

Para ver y desconectar una sesión en el portal:

1. Vaya a la puerta de enlace de VPN.
1. En la sección **Supervisión**, seleccione **Point-to-site Sessions** (Sesiones de punto a sitio).

   :::image type="content" source="./media/p2s-session-management/portal.png" alt-text="Ejemplo del portal":::
1. Puede ver todas las sesiones actuales en el panel de la ventana.
1. Seleccione **"..."** en la sesión que quiera desconectar y, luego, elija **Desconectar**.

Actualmente, no puede usar esta característica en el portal para las SKU VpnGw4 y VpnGw5. Si tiene una de estas puertas de enlace, use el método de PowerShell que se describe en la sección siguiente.

## <a name="powershell"></a>PowerShell

Para ver y desconectar una sesión mediante PowerShell:

1. Ejecute el siguiente comando de PowerShell para mostrar las sesiones activas:

   ```azurepowershell-interactive
   Get-AzVirtualNetworkGatewayVpnClientConnectionHealth -VirtualNetworkGatewayName <name of the gateway>  -ResourceGroupName <name of the resource group>
   ```
1. Copie el valor de **VpnConnectionId** de la sesión que quiere desconectar.

   :::image type="content" source="./media/p2s-session-management/powershell.png" alt-text="Ejemplo de PowerShell":::
1. Para desconectar la sesión, ejecute el siguiente comando:

   ```azurepowershell-interactive
   Disconnect-AzVirtualNetworkGatewayVpnConnection -VirtualNetworkGatewayName <name of the gateway> -ResourceGroupName <name of the resource group> -VpnConnectionId <VpnConnectionId of the session>
   ```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las conexiones de punto a sitio, consulte [Acerca de las conexiones VPN de punto a sitio](point-to-site-about.md).
