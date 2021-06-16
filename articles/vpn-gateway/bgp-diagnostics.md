---
title: Visualización del estado y las métricas de BGP
titleSuffix: Azure VPN Gateway
description: Aprenda a consultar información importante relacionada con BGP para la solución de problemas.
services: vpn-gateway
author: anzaman
ms.service: vpn-gateway
ms.topic: sample
ms.date: 03/10/2021
ms.author: alzam
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 65ebf90d2485969343092bbee1fd3935112b6466
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110672156"
---
# <a name="view-bgp-metrics-and-status"></a>Visualización de las métricas y el estado de BGP

Puede ver las métricas y el estado de BGP mediante Azure Portal o Azure PowerShell.

## <a name="azure-portal"></a>Azure Portal

En Azure Portal, puede ver los pares BGP, las rutas aprendidas y las rutas anunciadas. También puede descargar archivos .csv que contengan estos datos.

1. En [Azure Portal](https://portal.azure.com), vaya a su puerta de enlace de red.
1. En **Supervisión**, seleccione **Pares BGP** para abrir la página de pares BGP.

   :::image type="content" source="./media/bgp-diagnostics/bgp-portal.jpg" alt-text="Captura de pantalla de las métricas desde Azure Portal.":::

### <a name="learned-routes"></a>Rutas aprendidas

1. Puede ver hasta 50 rutas aprendidas en el portal.

   :::image type="content" source="./media/bgp-diagnostics/learned-routes.jpg" alt-text="Captura de pantalla de rutas aprendidas.":::

1. También puede descargar el archivo de rutas aprendidas. Si tiene más de 50 rutas aprendidas, la única manera de verlas todas es mediante la descarga y visualización del archivo .csv. Para descargarlo, seleccione **Download learned routes** (descargar rutas aprendidas).

   :::image type="content" source="./media/bgp-diagnostics/download-routes.jpg" alt-text="Captura de pantalla de la descarga de rutas aprendidas.":::
1. A continuación, vea el archivo.

   :::image type="content" source="./media/bgp-diagnostics/learned-routes-file.jpg" alt-text="Captura de pantalla de las rutas aprendidas descargadas.":::

### <a name="advertised-routes"></a>Rutas anunciadas

1. Para ver las rutas anunciadas, seleccione **...** al final de la red que quiera ver y, a continuación, haga clic en **View advertised routes** (ver rutas anunciadas).

   :::image type="content" source="./media/bgp-diagnostics/select-route.png" alt-text="Captura de pantalla que muestra cómo ver las rutas anunciadas." lightbox="./media/bgp-diagnostics/route-expand.png":::
1. En la página **Rutas anunciadas al par**, puede ver hasta 50 rutas anunciadas.

   :::image type="content" source="./media/bgp-diagnostics/advertised-routes.jpg" alt-text="Captura de pantalla de las rutas anunciadas.":::
1. También puede descargar el archivo de rutas anunciadas. Si tiene más de 50 rutas anunciadas, la única manera de verlas todas es mediante la descarga y visualización del archivo .csv. Para descargarlo, seleccione **Download advertised routes** (descargar rutas anunciadas).

   :::image type="content" source="./media/bgp-diagnostics/download-advertised.jpg" alt-text="Captura de pantalla de la selección de rutas anunciadas descargadas.":::
1. A continuación, vea el archivo.

   :::image type="content" source="./media/bgp-diagnostics/advertised-routes-file.jpg" alt-text="Captura de pantalla de las rutas anunciadas descargadas.":::

### <a name="bgp-peers"></a>Pares BGP

1. Puede ver hasta 50 pares BGP en el portal.

   :::image type="content" source="./media/bgp-diagnostics/peers.jpg" alt-text="Captura de pantalla de pares BGP.":::
1. También puede descargar el archivo de pares BGP. Si tiene más de 50 pares BGP, la única manera de verlas todas es mediante la descarga y visualización del archivo .csv. Para descargarlo, seleccione **Download BGP peers** (descargar pares BGP) en la página del portal.

   :::image type="content" source="./media/bgp-diagnostics/download-peers.jpg" alt-text="Captura de pantalla de la descarga de pares BGP.":::
1. A continuación, vea el archivo.

   :::image type="content" source="./media/bgp-diagnostics/peers-file.jpg" alt-text="Captura de pantalla de los pares BGP descargados.":::

## <a name="powershell"></a>PowerShell

Use **Get-AzVirtualNetworkGatewayBGPPeerStatus** para ver todos los pares BGP y su estado.

[!INCLUDE [VPN Gateway PowerShell instructions](../../includes/vpn-gateway-cloud-shell-powershell-about.md)]

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayBgpPeerStatus -ResourceGroupName resourceGroup -VirtualNetworkGatewayName gatewayName

Asn               : 65515
ConnectedDuration : 9.01:04:53.5768637
LocalAddress      : 10.1.0.254
MessagesReceived  : 14893
MessagesSent      : 14900
Neighbor          : 10.0.0.254
RoutesReceived    : 1
State             : Connected
```

Use **Get-AzVirtualNetworkGatewayLearnedRoute** para ver todas las rutas que la puerta de enlace aprendió a través de BGP.

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayLearnedRoute -ResourceGroupName resourceGroup -VirtualNetworkGatewayname gatewayName

AsPath       :
LocalAddress : 10.1.0.254
Network      : 10.1.0.0/16
NextHop      :
Origin       : Network
SourcePeer   : 10.1.0.254
Weight       : 32768

AsPath       :
LocalAddress : 10.1.0.254
Network      : 10.0.0.254/32
NextHop      :
Origin       : Network
SourcePeer   : 10.1.0.254
Weight       : 32768

AsPath       : 65515
LocalAddress : 10.1.0.254
Network      : 10.0.0.0/16
NextHop      : 10.0.0.254
Origin       : EBgp
SourcePeer   : 10.0.0.254
Weight       : 32768
```

Use **Get-AzVirtualNetworkGatewayAdvertisedRoute** para ver todas las rutas que la puerta de enlace está anunciando a sus pares a través de BGP.

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayAdvertisedRoute -VirtualNetworkGatewayName gatewayName -ResourceGroupName resourceGroupName -Peer 10.0.0.254
```

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de BGP, consulte [Configuración de BGP para Azure VPN Gateway](bgp-howto.md).
