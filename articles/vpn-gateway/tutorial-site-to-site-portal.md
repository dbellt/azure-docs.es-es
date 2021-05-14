---
title: 'Tutorial: Conexión de una red local a una red virtual: Azure portal'
description: En este tutorial, aprenderá a crear una conexión de puerta de enlace VPN de sitio a sitio de IPsec desde la red local a una red virtual.
titleSuffix: Azure VPN Gateway
author: cherylmc
ms.author: cherylmc
ms.service: vpn-gateway
ms.topic: tutorial
ms.date: 04/28/2021
ms.openlocfilehash: 83c2e42db4cc3e1a287341592f4917afce7309e2
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108204232"
---
# <a name="tutorial-create-a-site-to-site-connection-in-the-azure-portal"></a>Tutorial: Creación de una conexión de sitio a sitio mediante Azure Portal

Las puertas de enlace de VPN de Azure proporcionan conectividad entre locales entre el entorno local del cliente y Azure. En este tutorial se muestra cómo usar Azure Portal para crear una conexión de puerta de enlace VPN de sitio a sitio desde la red local a la red virtual. También puede crear esta configuración mediante [Azure PowerShell](vpn-gateway-create-site-to-site-rm-powershell.md) o la [CLI de Azure](vpn-gateway-howto-site-to-site-resource-manager-cli.md).

:::image type="content" source="./media/tutorial-site-to-site-portal/diagram.png" alt-text="Diagrama de la conexión entre locales de VPN Gateway de sitio a sitio":::

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de una red virtual
> * Creación de una puerta de enlace de VPN
> * Creación de una puerta de enlace de red local
> * Creación de una conexión VPN
> * Comprobación de la conexión
> * Conexión a una máquina virtual

## <a name="prerequisites"></a>Prerrequisitos

* Una cuenta de Azure con una suscripción activa. Si no tiene ninguna, [cree una gratis](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
* Asegúrese de tener un dispositivo VPN compatible y alguien que pueda configurarlo. Para más información acerca de los dispositivos VPN compatibles y su configuración, consulte [Acerca de los dispositivos VPN](vpn-gateway-about-vpn-devices.md).
* Compruebe que tiene una dirección IPv4 pública externa para el dispositivo VPN.
* Si no está familiarizado con los intervalos de direcciones IP ubicados en la red local, necesita trabajar con alguien que pueda proporcionarle estos detalles. Al crear esta configuración, debe especificar los prefijos del intervalo de direcciones IP al que Azure enrutará la ubicación local. Ninguna de las subredes de la red local puede superponerse con las subredes de la red virtual a la que desea conectarse.

## <a name="create-a-virtual-network"></a><a name="CreatVNet"></a>Creación de una red virtual

Cree una red virtual (VNet) con los siguientes valores:

* **Grupo de recursos:** TestRG1
* **Nombre:** VNet1
* **Región:** (EE. UU.) Este de EE. UU.
* **Espacio de direcciones IPv4:** 10.1.0.0/16
* **Nombre de subred:** FrontEnd
* **Espacio de direcciones de subred:** 10.1.0.0/24

[!INCLUDE [About cross-premises addresses](../../includes/vpn-gateway-cross-premises.md)]

[!INCLUDE [Create a virtual network](../../includes/vpn-gateway-basic-vnet-rm-portal-include.md)]

## <a name="create-a-vpn-gateway"></a><a name="VNetGateway"></a>Creación de una puerta de enlace de VPN

En este paso, se crea la puerta de enlace para la red virtual. La creación de una puerta de enlace suele tardar 45 minutos o más, según la SKU de la puerta de enlace seleccionada.

### <a name="about-the-gateway-subnet"></a>Acerca de la subred de puerta de enlace

[!INCLUDE [About gateway subnets](../../includes/vpn-gateway-about-gwsubnet-portal-include.md)]

### <a name="create-the-gateway"></a>Creación de la puerta de enlace

Cree una puerta de enlace de VPN con los siguientes valores:

* **Nombre:** VNet1GW
* **Región:** Este de EE. UU.
* **Tipo de puerta de enlace:** VPN
* **Tipo de VPN:** basada en rutas
* **SKU:** VpnGw1
* **Generación:** Generación 1
* **Red virtual:** VNet1
* **Intervalo de direcciones de subred de puerta de enlace:** 10.1.255.0/27
* **Dirección IP pública**: Crear nuevo
* **Dirección IP pública:** VNet1GWpip
* **Habilitar el modo activo-activo:** Disabled
* **Configuración de BGP:** Disabled

[!INCLUDE [Create a vpn gateway](../../includes/vpn-gateway-add-gw-rm-portal-include.md)]

[!INCLUDE [NSG warning](../../includes/vpn-gateway-no-nsg-include.md)]

### <a name="view-the-public-ip-address"></a><a name="view"></a>Visualización de la dirección IP pública

Puede ver la dirección IP pública de la puerta de enlace en la página **Información general** de la puerta de enlace.

:::image type="content" source="./media/tutorial-create-gateway-portal/address.png" alt-text="Página de información general":::

Para ver más información acerca del objeto de dirección IP pública, haga clic en el vínculo del nombre/dirección IP que hay junto a **Dirección IP pública**.

## <a name="create-a-local-network-gateway"></a><a name="LocalNetworkGateway"></a>Creación de una puerta de enlace de red local

La puerta de enlace de red local es un objeto específico que representa la ubicación local (el sitio) para fines de enrutamiento. Asigne al sitio un nombre al que Azure pueda hacer referencia y, luego, especifique la dirección IP del dispositivo VPN local con la que creará una conexión. Especifique también los prefijos de dirección IP que se enrutarán a través de la puerta de enlace VPN al dispositivo VPN. Los prefijos de dirección que especifique son los prefijos que se encuentran en la red local. Si la red local cambia o necesita cambiar la dirección IP pública del dispositivo VPN, puede actualizar fácilmente los valores más adelante.

Cree una puerta de enlace de red local con los siguientes valores:

* **Nombre:** Site1
* **Grupos de recursos:** TestRG1
* **Ubicación:** Este de EE. UU.

[!INCLUDE [Add a local network gateway](../../includes/vpn-gateway-add-local-network-gateway-portal-include.md)]

## <a name="configure-your-vpn-device"></a><a name="VPNDevice"></a>Configuración del dispositivo VPN

Las conexiones de sitio a sitio a una red local requieren un dispositivo VPN. En este paso, se configura el dispositivo VPN. Para configurar el dispositivo VPN, necesita los valores siguientes:

* Una clave compartida. Se trata de la misma clave compartida que se especifica al crear la conexión VPN de sitio a sitio. En estos ejemplos se utiliza una clave compartida básica. Se recomienda que genere y utilice una clave más compleja.
* La dirección IP pública de la puerta de enlace de red virtual. Puede ver la dirección IP pública mediante Azure Portal, PowerShell o la CLI. Para buscar la dirección IP pública de la puerta de enlace de VPN mediante Azure Portal, vaya a **Puertas de enlace de red virtual** y seleccione el nombre de su puerta de enlace.

[!INCLUDE [Configure a VPN device](../../includes/vpn-gateway-configure-vpn-device-include.md)]

## <a name="create-a-vpn-connection"></a><a name="CreateConnection"></a>Creación de una conexión VPN

Creación de la conexión VPN de sitio a sitio entre la puerta de enlace de la red virtual y el dispositivo VPN local.

Cree una conexión con los valores siguientes:

* **Nombre de la puerta de enlace de red local:** Site1
* **Nombre de la conexión:** VNet1toSite1
* **Clave compartida:** en este ejemplo, usamos abc123. Sin embargo, puede usar cualquiera compatible con el hardware VPN. Lo importante es que los valores coincidan en ambos lados de la conexión.

[!INCLUDE [Add a site-to-site connection](../../includes/vpn-gateway-add-site-to-site-connection-portal-include.md)]

## <a name="verify-the-vpn-connection"></a><a name="VerifyConnection"></a>Comprobación de la conexión VPN

[!INCLUDE [Verify the connection](../../includes/vpn-gateway-verify-connection-portal-include.md)]

## <a name="connect-to-a-virtual-machine"></a><a name="connectVM"></a>Conexión a una máquina virtual

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm.md)]

## <a name="optional-steps"></a>Pasos opcionales

### <a name="add-additional-connections-to-the-gateway"></a><a name="addconnect"></a>Incorporación de conexiones adicionales a la puerta de enlace

Puede agregar conexiones adicionales, siempre que ninguno de los espacios de direcciones se superpongan entre las conexiones.

1. Para agregar otra conexión, vaya a la puerta de enlace de VPN y, luego, haga clic en **Conexiones** para abrir la página Conexiones.
1. Seleccione **+ Agregar** para agregar la conexión. Ajuste el tipo de conexión para reflejar una conexión entre redes virtuales (si se conecta a otra puerta de enlace de red virtual), o de sitio a sitio.
1. Si se conecta de sitio a sitio y aún no ha creado una puerta de enlace de red local para el sitio al que desea conectarse, puede crear una.
1. Especifique la clave compartida que quiera usar y, luego, seleccione **Aceptar** para crear la conexión.

### <a name="resize-a-gateway-sku"></a><a name="resize"></a>Cambio del tamaño de la SKU de una puerta de enlace

Hay reglas específicas relativas al cambio de tamaño de una SKU de puerta de enlace, en lugar de su cambio. En esta sección, se cambiará el tamaño de una SKU. Para más información, consulte [Configuración de una puerta de enlace: cambio de tamaño y cambio de SKU](vpn-gateway-about-vpn-gateway-settings.md#resizechange).

[!INCLUDE [resize a gateway](../../includes/vpn-gateway-resize-gw-portal-include.md)]

### <a name="reset-a-gateway"></a><a name="reset"></a>Restablecimiento de una puerta de enlace

Restablecer una puerta de enlace de VPN de Azure es útil si se pierde la conectividad VPN entre locales en uno o varios túneles VPN de sitio a sitio. En esta situación, todos tus dispositivos VPN locales funcionan correctamente, pero no pueden establecer túneles IPsec con las Puertas de enlace de VPN de Azure.

[!INCLUDE [reset a gateway](../../includes/vpn-gateway-reset-gw-portal-include.md)]

### <a name="additional-configuration-considerations"></a><a name="additional"></a>Consideraciones adicionales sobre la configuración

Las configuraciones S2S se pueden personalizar de varias maneras. Vea los siguientes artículos para más información:

* Para más información acerca de BGP, consulte [Información general de BGP](vpn-gateway-bgp-overview.md) y [Configuración de BGP](vpn-gateway-bgp-resource-manager-ps.md).
* Para información acerca de la tunelización forzada, consulte la [Acerca de la tunelización forzada](vpn-gateway-forced-tunneling-rm.md).
* Para obtener información acerca de las conexiones activo/activo de alta disponibilidad, consulte [Conectividad de alta disponibilidad entre locales y de red virtual a red virtual](vpn-gateway-highlyavailable.md).
* Para información acerca de cómo limitar el tráfico de red a los recursos de una red virtual, vea [Seguridad de red](../virtual-network/network-security-groups-overview.md).
* Para información acerca de cómo enruta Azure el tráfico entre los recursos locales, de Internet y de Azure, vea [Enrutamiento del tráfico de redes virtuales](../virtual-network/virtual-networks-udr-overview.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación o si va al siguiente tutorial, siga estos pasos para eliminar estos recursos:

1. Escriba el nombre del grupo de recursos en el cuadro **Buscar** de la parte superior del portal y selecciónelo en los resultados de la búsqueda.

1. Seleccione **Eliminar grupo de recursos**.

1. En **TYPE THE RESOURCE GROUP NAME** (ESCRIBIR EL NOMBRE DEL GRUPO DE RECURSOS), escriba el grupo de recursos y seleccione **Delete** (Eliminar).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurada la conexión S2S, podrá agregar una conexión P2S a la misma puerta de enlace.

> [!div class="nextstepaction"]
> [Conexión VPN de punto a sitio](vpn-gateway-howto-point-to-site-resource-manager-portal.md)
