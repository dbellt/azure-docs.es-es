---
title: 'Tutorial - Azure ExpressRoute: agregar una puerta de enlace a una red virtual (Azure PowerShell)'
description: En este tutorial, se muestra cómo agregar una puerta de enlace de red virtual a una red virtual de Resource Manager ya creada para ExpressRoute mediante Azure PowerShell.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 10/05/2020
ms.author: duau
ms.custom: seodec18, devx-track-azurepowershell
ms.openlocfilehash: c49409472c8ce9c5b9dab25cbb66df8e1b30f410
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110695161"
---
# <a name="tutorial-configure-a-virtual-network-gateway-for-expressroute-using-powershell"></a>Tutorial: Configuración de una puerta de enlace de red virtual para ExpressRoute con PowerShell
> [!div class="op_single_selector"]
> * [Resource Manager: Azure Portal](expressroute-howto-add-gateway-portal-resource-manager.md)
> * [Resource Manager - PowerShell](expressroute-howto-add-gateway-resource-manager.md)
> * [Clásico: PowerShell](expressroute-howto-add-gateway-classic.md)
> * [Vídeo: Azure Portal](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-vpn-gateway-for-your-virtual-network)
> 

En este tutorial se muestra cómo agregar, cambiar el tamaño y quitar una puerta de enlace de red virtual para una red virtual existente previamente. Los pasos de esta configuración se aplican a redes virtuales creadas con el modelo de implementación de Resource Manager de una configuración ExpressRoute. Para más información, consulte [Puertas de enlace de red virtual para ExpressRoute](expressroute-about-virtual-network-gateways.md).

En este tutorial, obtendrá información sobre cómo:
> [!div class="checklist"]
> - Cree una subred de puerta de enlace.
> - Cree una puerta de enlace de red virtual.

## <a name="prerequisites"></a>Requisitos previos

### <a name="configuration-reference-list"></a>Lista de referencia de configuración

Los pasos de esta tarea usan una red virtual que se basa en los valores de la siguiente lista de referencia de configuración. En esta lista también se enumeran nombres y valores de configuración adicionales. No se utiliza esta lista directamente en ninguno de los pasos, aunque se agregan variables basadas en los valores que aparecen en ella. Puede copiar la lista para utilizarla como referencia y reemplazar los valores por los suyos propios.

| Parámetro                   | Value                                              |
| ---                       | ---                                                |
| El nombre de la red virtual | *TestVNet* |    
| Espacio de direcciones de red virtual | *192.168.0.0/16* |
| Grupo de recursos | *TestRG* |
| Nombre de la subred 1 | *FrontEnd* |   
| Espacio de direcciones de la subred 1 | *192.168.1.0/24* |
| Nombre de la subred 1 | *FrontEnd* |
| Nombre de la subred de puerta de enlace | *GatewaySubnet* |    
| Espacio de direcciones de la subred de puerta de enlace | *192.168.200.0/26* |
| Region | *Este de EE. UU.* |
| Nombre de puerta de enlace | *GW* |   
| Nombre de IP de puerta de enlace | *GWIP* |
| Nombre de configuración de IP de puerta de enlace | *gwipconf* |
| Tipo | *ExpressRoute* |
| Nombre de IP pública de puerta de enlace  | *gwpip* |

> [!IMPORTANT]
> La compatibilidad de IPv6 con el emparejamiento privado se encuentra actualmente en **versión preliminar pública**. Si quiere conectar la red virtual a un circuito ExpressRoute con emparejamiento privado basado en IPv6, asegúrese de que la red virtual es de doble pila y sigue las instrucciones que se describen [aquí](../virtual-network/ipv6-overview.md).
> 
> 

## <a name="add-a-gateway"></a>Adición de una puerta de enlace

1. Para conectarse con Azure, ejecute `Connect-AzAccount`.

1. Declare las variables para este ejercicio. No olvide editar el ejemplo para reflejar la configuración que desea utilizar.

   ```azurepowershell-interactive 
   $RG = "TestRG"
   $Location = "East US"
   $GWName = "GW"
   $GWIPName = "GWIP"
   $GWIPconfName = "gwipconf"
   $VNetName = "TestVNet"
   ```
1. Almacene el objeto de red virtual como una variable.

   ```azurepowershell-interactive
   $vnet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $RG
   ```
1. Agregue una subred de puerta de enlace a su red virtual. A la subred de puerta de enlace debe asignarle el nombre "GatewaySubnet". La subred de puerta de enlace debe ser /27 o superior (/26, /25, etc.). Si planea conectar 16 circuitos ExpressRoute a la puerta de enlace, **debe** crear una subred de puerta de enlace de tamaño /26 o mayor.

   ```azurepowershell-interactive
   Add-AzVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet -AddressPrefix 192.168.200.0/26
   ```
    Si usa una red virtual de doble pila y tiene previsto usar el emparejamiento privado basado en IPv6 en ExpressRoute, es mejor crear una subred de puerta de enlace de doble pila.

   ```azurepowershell-interactive
   Add-AzVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet -AddressPrefix "10.0.0.0/26","ace:daa:daaa:deaa::/64"
   ```
1. Establezca la configuración.

   ```azurepowershell-interactive
   $vnet = Set-AzVirtualNetwork -VirtualNetwork $vnet
   ```
1. Almacene la subred de puerta de enlace como una variable.

   ```azurepowershell-interactive
   $subnet = Get-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
   ```
1. Pida una dirección IP pública. La dirección IP se solicita antes de crear la puerta de enlace. No puede especificar la dirección IP que quiere usar; se asigna una dinámicamente. Utilizará esta dirección IP en la siguiente sección de configuración. El valor de AllocationMethod debe ser Dynamic.

   ```azurepowershell-interactive
   $pip = New-AzPublicIpAddress -Name $GWIPName  -ResourceGroupName $RG -Location $Location -AllocationMethod Dynamic
   ```
1. Cree la configuración para su puerta de enlace. La configuración de puerta de enlace define la subred y la dirección IP pública. En este paso, se especifica la configuración que se utilizará al crear la puerta de enlace. Use el ejemplo siguiente para crear la configuración de la puerta de enlace.

   ```azurepowershell-interactive
   $ipconf = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName -Subnet $subnet -PublicIpAddress $pip
   ```
1. Cree la puerta de enlace. En este paso, el elemento **-GatewayType** resulta especialmente importante. Debe utilizar el valor **ExpressRoute**. Después de ejecutar estos cmdlets, la puerta de enlace puede tardar 45 minutos o más en crearse.

   ```azurepowershell-interactive
   New-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG -Location $Location -IpConfigurations $ipconf -GatewayType Expressroute -GatewaySku Standard
   ```
> [!IMPORTANT]
> Si tiene previsto usar el emparejamiento privado basado en IPv6 en ExpressRoute, asegúrese de seleccionar una SKU AZ (ErGw1AZ, ErGw2AZ, ErGw3AZ) para **-GatewaySku**.
> 
> 

## <a name="verify-the-gateway-was-created"></a>Comprobación de la creación de la puerta de enlace
Utilice los siguientes comandos para comprobar si se ha creado la puerta de enlace:

```azurepowershell-interactive
Get-AzVirtualNetworkGateway -ResourceGroupName $RG
```

## <a name="resize-a-gateway"></a>Cambio del tamaño de una puerta de enlace
Hay varias [SKU de puerta de enlace](expressroute-about-virtual-network-gateways.md). Puede utilizar el siguiente comando para cambiar en cualquier momento la SKU de puerta de enlace.

> [!IMPORTANT]
> Este comando no funciona para la puerta de enlace de UltraPerformance. Para cambiar la puerta de enlace a una puerta de enlace de UltraPerformance, quite primero la puerta de enlace de ExpressRoute existente y, después, cree una nueva puerta de enlace de UltraPerformance. Para degradar la puerta de enlace desde una puerta de enlace de UltraPerformance, quite primero la puerta de enlace de UltraPerformance existente y, después, cree una nueva puerta de enlace.
> 

```azurepowershell-interactive
$gw = Get-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG
Resize-AzVirtualNetworkGateway -VirtualNetworkGateway $gw -GatewaySku HighPerformance
```

## <a name="clean-up-resources"></a>Limpieza de recursos
Para quitar la puerta de enlace, use el siguiente comando:

```azurepowershell-interactive
Remove-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG
```

## <a name="next-steps"></a>Pasos siguientes
Después de crear la puerta de enlace de red virtual, puede vincular la red virtual a un circuito ExpressRoute. 

> [!div class="nextstepaction"]
> [Vinculación de una red virtual a un circuito ExpressRoute](expressroute-howto-linkvnet-arm.md)
