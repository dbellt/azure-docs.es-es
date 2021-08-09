---
title: 'Tutorial: Creación y administración de una instancia de VPN Gateway: Azure portal'
description: En este tutorial aprenderá a crear, implementar y administrar una instancia de Azure VPN Gateway mediante Azure Portal.
titleSuffix: Azure VPN Gateway
author: cherylmc
ms.author: cherylmc
ms.service: vpn-gateway
ms.topic: tutorial
ms.date: 07/21/2021
ms.openlocfilehash: d0723d4a5e77fe9bcf52f515a1310dfc270338f6
ms.sourcegitcommit: bb1c13bdec18079aec868c3a5e8b33ef73200592
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/27/2021
ms.locfileid: "114721830"
---
# <a name="tutorial-create-and-manage-a-vpn-gateway-using-azure-portal"></a>Tutorial: Creación y administración de una puerta de enlace de VPN mediante Azure Portal

Las puertas de enlace de VPN de Azure proporcionan conectividad entre locales entre el entorno local del cliente y Azure. En este tutorial se tratan los elementos básicos de implementación de Azure VPN Gateway, como crear y administrar una puerta de enlace de VPN. También puede crear una puerta de enlace con [Azure PowerShell](create-routebased-vpn-gateway-cli.md) o la [CLI de Azure](create-routebased-vpn-gateway-powershell.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de una red virtual
> * Creación de una puerta de enlace de VPN
> * Visualización de la dirección IP pública de la puerta de enlace
> * Cambio de tamaño de una puerta de enlace de VPN (cambio de tamaño de la SKU)
> * Restablecimiento de una instancia de VPN Gateway

En el siguiente diagrama se muestran la red virtual y la puerta de enlace de VPN creadas como parte de este tutorial.

:::image type="content" source="./media/tutorial-create-gateway-portal/gateway-diagram.png" alt-text="Diagrama de VNET y VPN Gateway.":::

## <a name="prerequisites"></a>Requisitos previos

Una cuenta de Azure con una suscripción activa. Si no tiene ninguna, [cree una gratis](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

## <a name="create-a-virtual-network"></a><a name="CreatVNet"></a>Creación de una red virtual

Utilice estos valores para crear una red virtual:

* **Grupo de recursos:** TestRG1
* **Nombre:** VNet1
* **Región:** (EE. UU.) Este de EE. UU.
* **Espacio de direcciones IPv4:** 10.1.0.0/16
* **Nombre de subred:** FrontEnd
* **Espacio de direcciones de subred:** 10.1.0.0/24

[!INCLUDE [Create a virtual network](../../includes/vpn-gateway-basic-vnet-rm-portal-include.md)]

## <a name="create-a-vpn-gateway"></a><a name="VNetGateway"></a>Creación de una puerta de enlace de VPN

En este paso, se crea la puerta de enlace de red virtual (puerta de enlace de VPN). La creación de una puerta de enlace suele tardar 45 minutos o más, según la SKU de la puerta de enlace seleccionada.

Cree una puerta de enlace de red virtual con los siguientes valores:

* **Nombre:** VNet1GW
* **Región:** Este de EE. UU.
* **Tipo de puerta de enlace:** VPN
* **Tipo de VPN:** basada en rutas
* **SKU:** VpnGw2
* **Generación:** Generación 2
* **Red virtual:** VNet1
* **Intervalo de direcciones de subred de puerta de enlace:** 10.1.255.0/27
* **Dirección IP pública**: Crear nuevo
* **Dirección IP pública:** VNet1GWpip

[!INCLUDE [Create a vpn gateway](../../includes/vpn-gateway-add-gw-portal-include.md)]
[!INCLUDE [Configure PIP settings](../../includes/vpn-gateway-add-gw-pip-portal-include.md)]

Una puerta de enlace puede tardar 45 minutos aproximadamente en crearse e implementarse completamente. Puede ver el estado de implementación en la página Información general de la puerta de enlace. Una vez creada la puerta de enlace, puede ver la dirección IP que se le ha asignado consultando la red virtual en el portal. La puerta de enlace aparece como un dispositivo conectado.

[!INCLUDE [NSG warning](../../includes/vpn-gateway-no-nsg-include.md)]

## <a name="view-the-public-ip-address"></a><a name="view"></a>Visualización de la dirección IP pública

Puede ver la dirección IP pública de la puerta de enlace en la página **Información general** de la puerta de enlace.

:::image type="content" source="./media/tutorial-create-gateway-portal/address.png" alt-text="Captura de pantalla de la página de información general.":::

Para ver más información acerca del objeto de dirección IP pública, haga clic en el vínculo del nombre/dirección IP que hay junto a **Dirección IP pública**.

## <a name="resize-a-gateway-sku"></a><a name="resize"></a>Cambio del tamaño de la SKU de una puerta de enlace

Hay reglas específicas relativas al cambio de tamaño de una SKU de puerta de enlace, en lugar de su cambio. En esta sección, se cambiará el tamaño de una SKU. Para más información, consulte [Configuración de una puerta de enlace: cambio de tamaño y cambio de SKU](vpn-gateway-about-vpn-gateway-settings.md#resizechange).

[!INCLUDE [resize a gateway](../../includes/vpn-gateway-resize-gw-portal-include.md)]

## <a name="reset-a-gateway"></a><a name="reset"></a>Restablecimiento de una puerta de enlace

[!INCLUDE [reset a gateway](../../includes/vpn-gateway-reset-gw-portal-include.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación o si va al siguiente tutorial, siga estos pasos para eliminar estos recursos:

1. Escriba el nombre del grupo de recursos en el cuadro **Buscar** de la parte superior del portal y selecciónelo en los resultados de la búsqueda.

1. Seleccione **Eliminar grupo de recursos**.

1. En **TYPE THE RESOURCE GROUP NAME** (ESCRIBIR EL NOMBRE DEL GRUPO DE RECURSOS), escriba el grupo de recursos y seleccione **Delete** (Eliminar).

## <a name="next-steps"></a>Pasos siguientes

Una vez que tenga una puerta de enlace de VPN, puede configurar las conexiones. Los siguientes artículos le ayudarán a crear algunas de las configuraciones más comunes:

> [!div class="nextstepaction"]
> [Conexiones VPN de sitio a sitio](./tutorial-site-to-site-portal.md)

> [!div class="nextstepaction"]
> [Conexión VPN de punto a sitio](vpn-gateway-howto-point-to-site-resource-manager-portal.md)