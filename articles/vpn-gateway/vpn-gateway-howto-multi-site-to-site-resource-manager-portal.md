---
title: 'Adición de varias conexiones de sitio a sitio de una instancia de VPN Gateway a una red virtual: Azure Portal'
description: Aprenda a agregar conexiones de sitio a sitio adicionales a una instancia de VPN Gateway.
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 04/29/2021
ms.author: cherylmc
ms.openlocfilehash: d65367b561ed94c4872ce86edede2043f99cced4
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108292691"
---
# <a name="add-additional-s2s-connections-to-a-vnet-azure-portal"></a>Adición de conexiones S2S adicionales a una red virtual: Azure portal

> [!div class="op_single_selector"]
> * [Azure Portal](vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md)
> * [PowerShell (clásico)](vpn-gateway-multi-site.md)
>

Los pasos de este artículo le ayudan a agregar conexiones de sitio a sitio (S2S) adicionales a VPN Gateway con una conexión existente. Esta arquitectura se denomina con frecuencia configuración "multisitio". Puede agregar una conexión S2S a una red virtual que ya tiene una conexión S2S, una conexión de punto a sitio o una conexión entre dos redes virtuales. Existen algunas limitaciones al agregar conexiones. Vea la sección [Requisitos previos](#before) de este artículo para comprobarlas antes de iniciar la configuración.

Este artículo se aplica a redes virtuales de Resource Manager que tienen una puerta de enlace de VPN RouteBased. Estos pasos no se aplican a las nuevas configuraciones de conexión coexistentes de ExpressRoute/de sitio a sitio. Sin embargo, si solo va a agregar una nueva conexión VPN a una configuración coexistente, puede seguir estos pasos. Consulte [conexiones coexistentes de ExpressRoute/S2S](../expressroute/expressroute-howto-coexist-resource-manager.md) para obtener información sobre las conexiones coexistentes.

## <a name="prerequisites"></a><a name="before"></a>Requisitos previos

Compruebe los siguientes aspectos:

* No va a configurar una configuración de ExpressRoute y VPN Gateway coexistente nueva.
* Tiene una red virtual que se ha creado con el modelo de implementación de Resource Manager con una conexión existente.
* La puerta de enlace de red virtual para su red virtual es RouteBased. Si tiene una VPN Gateway PolicyBased, necesita eliminar la puerta de enlace de red virtual y crear una VPN Gateway nueva como RouteBased.
* Ninguno de los intervalos de direcciones se superpone con ninguna de las redes virtuales a la que se está conectando esta red virtual.
* Tiene un dispositivo VPN compatible y alguien que pueda configurarlo. Consulte [Acerca de los dispositivos VPN para conexiones de red virtual de sitio a sitio](vpn-gateway-about-vpn-devices.md). Si no conoce la configuración de su dispositivo VPN o los intervalos de direcciones IP ubicados en la configuración de la red local, necesita trabajar con alguien que pueda proporcionarle estos detalles.
* Tiene una dirección IP pública externa para el dispositivo VPN.

## <a name="configure-a-connection"></a><a name="configure"></a>Configuración de una conexión

1. Desde un explorador, vaya [Azure Portal](https://portal.azure.com) y, si fuera necesario, inicie sesión con su cuenta de Azure.
1. Seleccione **Todos los recursos**, localice su **puerta de enlace de red virtual** en la lista de recursos y selecciónela.
1. En la página **Puerta de enlace de red virtual**, seleccione **Conexiones**.

   :::image type="content" source="./media/vpn-gateway-howto-multi-site-to-site-resource-manager-portal/connections.png" alt-text="Conexiones de VPN Gateway":::
1. En la página **Conexiones**, seleccione **+ Agregar**.
1. De este modo se abrirá la página **Agregar conexión**.

   :::image type="content" source="./media/vpn-gateway-howto-multi-site-to-site-resource-manager-portal/add-connection.png" alt-text="Página Agregar conexión":::
1. En la página **Agregar conexión**, rellene los campos siguientes:

   * **Nombre:** el nombre que quiere darle al sitio para el que está creando la conexión.
   * **Tipo de conexión**: seleccione **De sitio a sitio (IPsec)** .

## <a name="add-a-local-network-gateway"></a><a name="local"></a>Adición de una puerta de enlace de red local

1. En el campo **Puerta de enlace de red local**, seleccione **_Elegir una puerta de enlace de red local_ *_. Se abre la página _* Elegir puerta de enlace de red local**.
1. Seleccione **+ Crear nueva** para abrir la página **Crear puerta de enlace de red local**.

   :::image type="content" source="./media/vpn-gateway-howto-multi-site-to-site-resource-manager-portal/create-local-network-gateway.png" alt-text="Página Crear puerta de enlace de red local":::
1. En la página **Crear puerta de enlace de red local**, rellene los campos siguientes:

   * **Nombre:** el nombre que quiere darle al recurso de puerta de enlace de red local.
   * **Punto de conexión:** Dirección IP pública del dispositivo VPN del sitio al que quiere conectarse o FQDN del punto de conexión.
   * **Espacio de direcciones:** el espacio de direcciones al que quiere que se enrute el nuevo sitio de red local.
1. Seleccione **Aceptar** en la página **Crear puerta de enlace de red local** para guardar los cambios.

## <a name="add-the-shared-key"></a><a name="part3"></a>Adición de la clave compartida

1. Después de crear la puerta de enlace de red local, vuelva a la página **Agregar conexión**.
1. Rellene los demás campos. En **Clave compartida (PSK)** , puede obtener la clave compartida de su dispositivo VPN o crear una aquí y, después, configurar su dispositivo VPN para que use la misma clave compartida. Lo importante es que las claves sean exactamente iguales.

## <a name="create-the-connection"></a><a name="create"></a>Creación de la conexión

1. En la parte inferior de la página, seleccione **Aceptar** para crear la conexión. La conexión empieza a crearse inmediatamente.
1. Una vez completada la conexión, puede verla y comprobarla.

## <a name="view-and-verify-the-vpn-connection"></a><a name="verify"></a>Visualización y comprobación de la conexión VPN

[!INCLUDE [Verify the connection](../../includes/vpn-gateway-verify-connection-portal-include.md)]

## <a name="next-steps"></a>Pasos siguientes

Una vez completada la conexión, puede agregar máquinas virtuales a las redes virtuales. Para más información, consulte las [rutas de aprendizaje de las máquinas virtuales](/learn/paths/deploy-a-website-with-azure-virtual-machines/).
