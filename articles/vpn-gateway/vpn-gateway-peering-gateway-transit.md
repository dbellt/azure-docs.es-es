---
title: Configuración del tránsito de la puerta de enlace de VPN para el emparejamiento de red virtual
description: Aprenda cómo configurar el tránsito de puerta de enlace para el emparejamiento de red virtual a fin de conectar de forma sencilla dos redes virtuales de Azure a una con fines de conectividad.
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 04/28/2021
ms.author: cherylmc
ms.openlocfilehash: 7573f33082e1915bdf75ffe015afedfedbfda469
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108205294"
---
# <a name="configure-vpn-gateway-transit-for-virtual-network-peering"></a>Configuración del tránsito de la puerta de enlace de VPN para el emparejamiento de red virtual

Este artículo le ayuda a configurar el tránsito de la puerta de enlace para el emparejamiento de red virtual. El [emparejamiento de red virtual](../virtual-network/virtual-network-peering-overview.md) conecta perfectamente dos redes virtuales de Azure, combinando las dos redes virtuales en una sola para favorecer la conectividad. El [tránsito de puerta de enlace](../virtual-network/virtual-network-peering-overview.md#gateways-and-on-premises-connectivity) es una propiedad del emparejamiento que permite que una red virtual utilice la puerta de enlace de VPN en la red virtual emparejada para la conectividad entre locales o entre redes virtuales. El siguiente diagrama muestra cómo funciona el tránsito de la puerta de enlace con el emparejamiento de red virtual.

![Diagrama del tránsito de puerta de enlace](./media/vpn-gateway-peering-gateway-transit/gatewaytransit.png)

En el diagrama, el tránsito de la puerta de enlace permite que las redes virtuales emparejadas usen Azure VPN Gateway en Hub-RM. La conectividad disponible en la puerta de enlace de VPN, que incluye las conexiones S2S, P2S y entre redes virtuales, se aplica a las tres redes virtuales. La opción de tránsito está disponible para el emparejamiento entre modelos de implementación iguales o diferentes. Si está configurando el tránsito entre distintos modelos de implementación, la red virtual del centro y la puerta de enlace de red virtual deben estar en el modelo de implementación de Resource Manager, no en el modelo de implementación clásica.
>

En la arquitectura de red del tipo hub-and-spoke, el tránsito de la puerta de enlace permite que las redes virtuales de radio compartan la puerta de enlace de VPN en el concentrador, en lugar de implementar puertas de enlace de VPN en todas las redes virtuales de radio. Las rutas a las redes virtuales conectadas a la puerta de enlace o a las redes locales se propagarán a las tablas de enrutamiento de las redes virtuales emparejadas mediante el tránsito de la puerta de enlace. Puede deshabilitar la propagación automática de rutas de la puerta de enlace de VPN. Cree una tabla de enrutamiento con la opción "**Deshabilitar la propagación de rutas BGP**" y asóciela con las subredes para evitar la distribución de rutas en dichas subredes. Para más información, consulte [Tabla de enrutamiento de redes virtuales](../virtual-network/manage-route-table.md).

En este artículo, hay dos escenarios:

* **Mismo modelo de implementación**: ambas redes virtuales se crean en el modelo de implementación de Resource Manager.
* **Diferentes modelos de implementación**: la red virtual de radio se crea en el modelo de implementación clásica y la red virtual de centro y la puerta de enlace se encuentran en el modelo de implementación de Resource Manager.

>[!NOTE]
> Si realiza un cambio en la topología de la red y tiene clientes VPN de Windows, el paquete de cliente VPN para clientes de Windows se debe descargar e instalar nuevamente para que los cambios se apliquen en el cliente.
>

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, compruebe que dispone de las siguientes redes virtuales y permisos:

### <a name="virtual-networks"></a><a name="vnet"></a>Redes virtuales

|VNet|Modelo de implementación| Puerta de enlace de red virtual|
|---|---|---|---|
| Hub-RM| [Resource Manager](./tutorial-site-to-site-portal.md)| [Sí](tutorial-create-gateway-portal.md)|
| Spoke-RM | [Resource Manager](./tutorial-site-to-site-portal.md)| No |
| Spoke-Classic | [Clásico](vpn-gateway-howto-site-to-site-classic-portal.md#CreatVNet) | No |

### <a name="permissions"></a><a name="permissions"></a>Permisos

Las cuentas que use para crear un emparejamiento de redes virtuales deben tener los rol o permisos necesarios. En el ejemplo siguiente, si fuera a emparejar dos redes virtuales llamadas **Hub-RM** y **Spoke-Classic**, la cuenta debería tener los siguientes roles o permisos mínimos para cada red virtual:

|VNet|Modelo de implementación|Role|Permisos|
|---|---|---|---|
|Hub-RM|Resource Manager|[Colaborador de la red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)|Microsoft.Network/virtualNetworks/virtualNetworkPeerings/write|
| |Clásico|[Colaborador de la red clásica](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#classic-network-contributor)|N/D|
|Spoke-Classic|Resource Manager|[Colaborador de la red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)|Microsoft.Network/virtualNetworks/peer|
||Clásico|[Colaborador de la red clásica](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#classic-network-contributor)|Microsoft.ClassicNetwork/virtualNetworks/peer|

Obtenga más información sobre los [roles integrados](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) y la asignación de permisos específicos a [roles personalizados](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (solo Resource Manager).

## <a name="same-deployment-model"></a><a name="same"></a>Mismo modelo de implementación

En este escenario, las redes virtuales están ambas en el modelo de implementación de Resource Manager. Use los pasos siguientes para crear o actualizar los emparejamientos de red virtual para habilitar el tránsito de puerta de enlace.

### <a name="to-add-a-peering-and-enable-transit"></a>Para agregar un emparejamiento y habilitar el tránsito:

1. En [Azure Portal](https://portal.azure.com), cree o actualice el emparejamiento de red virtual de Hub-RM. Vaya a la red virtual **Hub-RM**. Seleccione **Emparejamientos** y, después, **+ Agregar** para abrir **Agregar emparejamiento**.
1. En la página **Agregar emparejamiento**, configure los valores de **This virtual network** (Esta red virtual).

   * Nombre del vínculo de emparejamiento: Asigne un nombre al vínculo. Ejemplo: **HubRMToSpokeRM**
   * Tráfico hacia la red virtual remota: **Permitir**
   * Tráfico reenviado desde la red virtual remota: **Permitir**
   * Puerta de enlace de red virtual: **Use this virtual network's gateway** (Usar la puerta de enlace de esta red virtual)

     :::image type="content" source="./media/vpn-gateway-peering-gateway-transit/peering-vnet.png" alt-text="Captura de pantalla que muestra Agregar emparejamiento.":::

1. En la misma página, continúe con la configuración de los valores de la **red virtual remota**.

   * Nombre del vínculo de emparejamiento: Asigne un nombre al vínculo. Ejemplo: **SpokeRMtoHubRM**
   * Modelo de implementación: **Resource Manager**
   * Red virtual: **Spoke-RM**
   * Tráfico hacia la red virtual remota: **Permitir**
   * Tráfico reenviado desde la red virtual remota: **Permitir**
   * Puerta de enlace de red virtual: **Use the remote virtual network's gateway** (Usar la puerta de enlace de esta red virtual)

     :::image type="content" source="./media/vpn-gateway-peering-gateway-transit/peering-remote.png" alt-text="Captura de pantalla que muestra los valores de la red virtual remota.":::

1. Seleccione **Agregar** para crear el emparejamiento.
1. Compruebe que el estado de emparejamiento sea **Conectado** en ambas redes virtuales.

### <a name="to-modify-an-existing-peering-for-transit"></a>Para modificar un emparejamiento existente para el tránsito:

Si ya se ha creado el emparejamiento, puede modificarlo para el tránsito.

1. Vaya a la red virtual. Seleccione **Emparejamientos** y elija el emparejamiento que quiere modificar.

   :::image type="content" source="./media/vpn-gateway-peering-gateway-transit/peering-modify.png" alt-text="Captura de pantalla que muestra la selección de emparejamientos.":::

1. Actualice el emparejamiento de red virtual.

   * Tráfico hacia la red virtual remota: **Permitir**
   * Tráfico reenviado a la red virtual; **Permitir**
   * Puerta de enlace de red virtual: **Use remote virtual network's gateway** (Usar la puerta de enlace de esta red virtual)

     :::image type="content" source="./media/vpn-gateway-peering-gateway-transit/modify-peering-settings.png" alt-text="Captura de pantalla que muestra la modificación de la puerta de enlace de emparejamiento.":::

1. **Guarde** la configuración de emparejamiento.

### <a name="powershell-sample"></a><a name="ps-same"></a>Ejemplo de PowerShell

También puede usar PowerShell para crear o actualizar el emparejamiento con el ejemplo anterior. Reemplace las variables por los nombres de las redes virtuales y los grupos de recursos.

```azurepowershell-interactive
$SpokeRG = "SpokeRG1"
$SpokeRM = "Spoke-RM"
$HubRG   = "HubRG1"
$HubRM   = "Hub-RM"

$spokermvnet = Get-AzVirtualNetwork -Name $SpokeRM -ResourceGroup $SpokeRG
$hubrmvnet   = Get-AzVirtualNetwork -Name $HubRM -ResourceGroup $HubRG

Add-AzVirtualNetworkPeering `
  -Name SpokeRMtoHubRM `
  -VirtualNetwork $spokermvnet `
  -RemoteVirtualNetworkId $hubrmvnet.Id `
  -UseRemoteGateways

Add-AzVirtualNetworkPeering `
  -Name HubRMToSpokeRM `
  -VirtualNetwork $hubrmvnet `
  -RemoteVirtualNetworkId $spokermvnet.Id `
  -AllowGatewayTransit
```

## <a name="different-deployment-models"></a><a name="different"></a>Diferentes modelos de implementación

En esta configuración, la red virtual de radio **Spoke-Classic** está en el modelo de implementación clásica y la red virtual de centro **Hub-RM** está en el modelo de implementación de Resource Manager. Al configurar el tránsito entre los modelos de implementación, la puerta de enlace de red virtual debe estar configurada para la red virtual de Resource Manager, no la red virtual clásica.

Con esta configuración, solo tiene que configurar la red virtual **Hub-RM**. No es necesario configurar nada en la red virtual **Spoke-Classic**.

1. En Azure Portal, vaya a la red virtual **Hub-RM**, seleccione **Emparejamientos** y, luego, elija **+ Agregar**.
1. En la página **Agregar emparejamiento**, configure los siguientes valores:

   * Nombre del vínculo de emparejamiento: Asigne un nombre al vínculo. Ejemplo: **HubRMToClassic**
   * Tráfico hacia la red virtual remota: **Permitir**
   * Tráfico reenviado desde la red virtual remota: **Permitir**
   * Puerta de enlace de red virtual: **Use this virtual network's gateway** (Usar la puerta de enlace de esta red virtual)
   * Red virtual remota: **Clásico**

     :::image type="content" source="./media/vpn-gateway-peering-gateway-transit/peering-classic.png" alt-text="Página Agregar emparejamiento para Spoke-Classic":::

1. Compruebe que la suscripción es correcta y, a continuación, seleccione la red virtual en la lista desplegable.
1. Seleccione **Agregar** para agregar el emparejamiento.
1. Compruebe que el estado de emparejamiento sea **Conectado** en la red virtual Hub-RM. 

Con esta configuración, no es necesario configurar nada en la red virtual **Spoke-Classic**. Una vez que el estado muestra **Conectado**, la red virtual de radio puede usar la conectividad a través de la puerta de enlace de VPN en la red virtual de centro.

### <a name="powershell-sample"></a><a name="ps-different"></a>Ejemplo de PowerShell

También puede usar PowerShell para crear o actualizar el emparejamiento con el ejemplo anterior. Reemplace las variables y el identificador de la suscripción por los valores de la red virtual y de los grupos de recursos, así como de la suscripción. Basta con crear un emparejamiento de red virtual en la red virtual del concentrador.

```azurepowershell-interactive
$HubRG   = "HubRG1"
$HubRM   = "Hub-RM"

$hubrmvnet   = Get-AzVirtualNetwork -Name $HubRM -ResourceGroup $HubRG

Add-AzVirtualNetworkPeering `
  -Name HubRMToClassic `
  -VirtualNetwork $hubrmvnet `
  -RemoteVirtualNetworkId "/subscriptions/<subscription Id>/resourceGroups/Default-Networking/providers/Microsoft.ClassicNetwork/virtualNetworks/Spoke-Classic" `
  -AllowGatewayTransit
```

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información acerca de las [restricciones y comportamientos importantes del emparejamiento de redes virtuales](../virtual-network/virtual-network-manage-peering.md#requirements-and-constraints), así como la [configuración del emparejamiento de redes virtuales](../virtual-network/virtual-network-manage-peering.md#create-a-peering) antes de crear uno para usarlo en un entorno de producción.
* Aprenda a [crear una topología del tipo hub-and-spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke#virtual-network-peering) con emparejamiento de redes virtuales y tránsito de la puerta de enlace.
* [Creación de un emparejamiento de red virtual con el mismo modelo de implementación](../virtual-network/tutorial-connect-virtual-networks-portal.md).
* [Creación de un emparejamiento de red virtual con distintos modelos de implementación](../virtual-network/create-peering-different-deployment-models.md).