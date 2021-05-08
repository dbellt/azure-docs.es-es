---
title: 'Inicio rápido: Creación y configuración de una instancia de Route Server mediante Azure PowerShell'
description: En esta guía de inicio rápido, aprenderá a crear y configurar una instancia de Route Server mediante Azure PowerShell.
services: route-server
author: duongau
ms.author: duau
ms.date: 04/23/2021
ms.topic: quickstart
ms.service: route-server
ms.custom:
- mode-api
ms.openlocfilehash: 7c28ad86b9194c8120b463581177f659a13d5c2b
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107949035"
---
# <a name="quickstart-create-and-configure-route-server-using-azure-powershell"></a>Inicio rápido: Creación y configuración de una instancia de Route Server mediante Azure PowerShell

Este artículo le ayuda a configurar Azure Route Server para su emparejamiento con una aplicación virtual de red (NVA) en la red virtual mediante PowerShell. Azure Route Server aprenderá las rutas de la NVA y las programará en las máquinas virtuales de la red virtual. Azure Route Server también anunciará las rutas de la red virtual a la NVA. Para más información, consulte [Azure Route Server](overview.md).

:::image type="content" source="media/quickstart-configure-route-server-portal/environment-diagram.png" alt-text="Diagrama del entorno de implementación de Route Server mediante Azure PowerShell." border="false":::

> [!IMPORTANT]
> Azure Route Server (versión preliminar) está actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Asegúrese de que tiene los módulos más recientes de PowerShell, o bien, puede usar Azure Cloud Shell en el portal.
* Consulte [Límites de servicio de Azure Route Server](route-server-faq.md#limitations).

## <a name="create-a-route-server"></a>Creación de una instancia de Route Server

### <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Inicie sesión en la cuenta de Azure y seleccione su suscripción.

[!INCLUDE [sign in](../../includes/expressroute-cloud-shell-connect.md)]

### <a name="create-a-resource-group-and-virtual-network"></a>Creación de un grupo de recursos y red virtual

Para poder crear una instancia de Azure Route Server, necesitará una red virtual para hospedar la implementación. Use el comando siguiente para crear un grupo de recursos y una red virtual. Si ya tiene una red virtual, puede ir directamente a la sección siguiente.

```azurepowershell-interactive
New-AzResourceGroup –Name "RouteServerRG” -Location “West US"
New-AzVirtualNetwork –ResourceGroupName "RouteServerRG" -Location "West US" -Name myVirtualNetwork –AddressPrefix 10.0.0.0/16
```

### <a name="add-a-subnet"></a>Incorporación de una subred

1. Agregue una subred llamada *RouteServerSubnet* para implementar la instancia de Azure Route Server en ella. Esta subred es una subred dedicada solo para Azure Route Server. La subred RouteServerSubnet debe ser /27 o un prefijo más corto, (como /26 o /25). De lo contrario, recibirá un mensaje de error al agregar la instancia de Azure Route Server.

    ```azurepowershell-interactive
    $vnet = Get-AzVirtualNetwork –Name "myVirtualNetwork" - ResourceGroupName "RouteServerRG"
    Add-AzVirtualNetworkSubnetConfig –Name "RouteServerSubnet" -AddressPrefix 10.0.0.0/24 -VirtualNetwork $vnet
    $vnet | Set-AzVirtualNetwork
    ```

1. Obtenga el identificador de la subred RouteServerSubnet. Para ver el identificador de recurso de todas las subredes de la red virtual, use este comando:

    ```azurepowershell-interactive
    $vnet = Get-AzVirtualNetwork –Name "vnet_name" -ResourceGroupName "RouteServerRG"
    $vnet.Subnets
    ```

El identificador de RouteServerSubnet tiene un aspecto similar al siguiente:

`/subscriptions/<subscriptionID>/resourceGroups/RouteServerRG/providers/Microsoft.Network/virtualNetworks/myVirtualNetwork/subnets/RouteServerSubnet`

## <a name="create-the-route-server"></a>Creación de la instancia de Route Server

Cree la instancia de Route Server con este comando:

```azurepowershell-interactive 
New-AzRouteServer -RouteServerName myRouteServer -ResourceGroupName RouteServerRG -Location "West US" -HostedSubnet "RouteServerSubnet_ID"
```

La ubicación debe coincidir con la ubicación de la red virtual. HostedSubnet es el identificador de RouteServerSubnet que obtuvo en la sección anterior.

## <a name="create-peering-with-an-nva"></a>Creación del emparejamiento con una NVA

Use el siguiente comando para establecer el emparejamiento de BGP desde la instancia de Route Server a la NVA:

```azurepowershell-interactive 
Add-AzRouteServerPeer -PeerName "myNVA" -PeerIp "nva_ip" -PeerAsn "nva_asn" -RouteServerName myRouteServer -ResourceGroupName RouteServerRG
```

"nva_ip" es la dirección IP de la red virtual asignada a la NVA. "nva_asn" es el número de sistema autónomo (ASN) configurado en la NVA. El ASN puede ser cualquier número de 16 bits que no se encuentre en el intervalo del 65515 al 65520. Este intervalo de ASN está reservado por Microsoft.

Para configurar el emparejamiento con diferentes NVA u otra instancia de la misma NVA para la redundancia, use este comando:

```azurepowershell-interactive 
Add-AzRouteServerPeer -PeerName "NVA2_name" -PeerIp "nva2_ip" -PeerAsn "nva2_asn" -RouteServerName myRouteServer -ResourceGroupName RouteServerRG 
```

## <a name="complete-the-configuration-on-the-nva"></a>Finalización de la configuración en la NVA

Para completar la configuración en la NVA y habilitar las sesiones de BGP, necesita la dirección IP y el ASN de la instancia de Azure Route Server. Puede obtener esta información mediante este comando:

```azurepowershell-interactive 
Get-AzRouteServer -RouterServerName myRouteServer -ResourceGroupName RouteServerRG
```

La salida tiene la siguiente información:

``` 
RouteServerAsn : 65515
RouteServerIps : {10.5.10.4, 10.5.10.5}
```

## <a name="configure-route-exchange"></a><a name = "route-exchange"></a>Configuración del intercambio de rutas

Si tiene una puerta de enlace de ExpressRoute y una instancia de Azure VPN Gateway en la misma red virtual y quiere que intercambien rutas, puede habilitar el intercambio de rutas en la instancia de Azure Route Server.

1. Para habilitar el intercambio de rutas entre Azure Route Server y las puertas de enlace, use este comando:

```azurepowershell-interactive 
Update-AzRouteServer -RouteServerName myRouteServer -ResourceGroupName RouteServerRG -AllowBranchToBranchTraffic 
```

2. Para deshabilitar el intercambio de rutas entre Azure Route Server y las puertas de enlace, use este comando:

```azurepowershell-interactive 
Update-AzRouteServer -RouteServerName myRouteServer -ResourceGroupName RouteServerRG
```

## <a name="troubleshooting"></a>Solución de problemas

Puede ver las rutas anunciadas y recibidas por Azure Route Server con este comando:

```azurepowershell-interactive
Get-AzRouteServerPeerAdvertisedRoute
Get-AzRouteServerPeerLearnedRoute
```
## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no necesita la instancia de Azure Route Server, use estos comandos para quitar el emparejamiento de BGP y, a continuación, quite la instancia de Route Server. 

1. Elimine el emparejamiento de BGP entre Azure Route Server y una NVA con este comando:

```azurepowershell-interactive 
Remove-AzRouteServerPeer -PeerName "nva_name" -RouteServerName myRouteServer -ResourceGroupName RouteServerRG 
```

2. Elimine la instancia de Azure Route Server con este comando:

```azurepowershell-interactive 
Remove-AzRouteServer -RouteServerName myRouteServer -ResourceGroupName RouteServerRG
```

## <a name="next-steps"></a>Pasos siguientes

Después de crear la instancia de Azure Route Server, continúe para obtener información sobre cómo interactúa Azure Route Server con las puertas de enlace de VPN y ExpressRoute: 

> [!div class="nextstepaction"]
> [Compatibilidad de Azure Route Server (versión preliminar) con ExpressRoute y VPN de Azure](expressroute-vpn-support.md)
