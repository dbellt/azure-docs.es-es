---
title: 'Conexión a una red virtual mediante una VPN P2S y varios tipos de autenticación: portal'
titleSuffix: Azure VPN Gateway
description: Obtenga información sobre cómo conectarse a una red virtual mediante P2S usando varios tipos de autenticación.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 02/22/2021
ms.author: cherylmc
ms.openlocfilehash: 8141d44aa4da394726d8c45a1f8514bdcaac09aa
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108229385"
---
# <a name="configure-a-point-to-site-vpn-connection-to-a-vnet-using-multiple-authentication-types-azure-portal"></a>Configuración de una conexión VPN de punto a sitio a una red virtual mediante varios tipos de autenticación: Azure Portal

Este artículo le ayudará con la conexión segura de clientes que ejecutan Windows, Linux o macOS a una red virtual de Azure. Las conexiones VPN de punto a sitio son útiles cuando desea conectarse a la red virtual desde una ubicación remota, como desde casa o desde una conferencia. También puede usar P2S en lugar de una VPN de sitio a sitio cuando son pocos los clientes que necesitan conectarse a una red virtual. Las conexiones de punto a sitio no requieren un dispositivo VPN ni una dirección IP de acceso público. P2S crea la conexión VPN sobre SSTP (Protocolo de túnel de sockets seguros) o IKEv2. Para más información sobre las conexiones VPN de punto a sitio, consulte [Acerca de las conexiones VPN de punto a sitio](point-to-site-about.md).

:::image type="content" source="./media/vpn-gateway-howto-point-to-site-resource-manager-portal/point-to-site-diagram.png" alt-text="Conexión de un equipo a una red virtual de Azure: diagrama de conexión de punto a sitio":::

Para más información sobre las conexiones VPN de punto a sitio, consulte [Acerca de las conexiones VPN de punto a sitio](point-to-site-about.md). Para crear esta configuración mediante Azure PowerShell, consulte [Configuración de una conexión VPN de punto a sitio con Azure PowerShell](vpn-gateway-howto-point-to-site-rm-ps.md).

## <a name="prerequisites"></a>Requisitos previos

Compruebe que tiene una suscripción a Azure. Si todavía no la tiene, puede activar sus [ventajas como suscriptor de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial).

Solo se admiten varios tipos de autenticación en la misma puerta de enlace de VPN con el tipo de túnel OpenVPN.

### <a name="example-values"></a><a name="example"></a>Valores del ejemplo

Puede usar los siguientes valores para crear un entorno de prueba o hacer referencia a ellos para comprender mejor los ejemplos de este artículo:

* **Nombre de la red virtual:** VNet1
* **Espacio de direcciones**: 10.1.0.0/16<br>En este ejemplo, se utiliza solo un espacio de direcciones. Puede tener más de un espacio de direcciones para la red virtual.
* **Nombre de subred:** FrontEnd
* **Intervalo de direcciones de subred:** 10.1.0.0/24
* **Subscription** (Suscripción): si tiene más de una suscripción, compruebe que usa la correcta.
* **Grupos de recursos:** TestRG1
* **Ubicación:** Este de EE. UU.
* **GatewaySubnet:** 10.1.255.0/27<br>
* **Nombre de la puerta de enlace de red virtual:** VNet1GW
* **Tipo de puerta de enlace:** VPN
* **Tipo de VPN:** basada en rutas
* **Nombre de dirección IP pública:** VNet1GWpip
* **Tipo de conexión**: De punto a sitio
* **Grupo de direcciones de clientes:** 172.16.201.0/24<br>Los clientes de VPN que se conectan a la red virtual mediante esta conexión de punto a sitio reciben una dirección IP del grupo de clientes.

## <a name="create-a-virtual-network"></a><a name="createvnet"></a>Creación de una red virtual

Antes de empezar, compruebe que tiene una suscripción a Azure. Si todavía no la tiene, puede activar sus [ventajas como suscriptor de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial).

[!INCLUDE [About cross-premises addresses](../../includes/vpn-gateway-cross-premises.md)]

[!INCLUDE [Basic Point-to-Site VNet](../../includes/vpn-gateway-basic-vnet-rm-portal-include.md)]

## <a name="virtual-network-gateway"></a><a name="creategw"></a>Puerta de enlace de red virtual

En este paso, se crea la puerta de enlace para la red virtual. La creación de una puerta de enlace suele tardar 45 minutos o más, según la SKU de la puerta de enlace seleccionada.

>[!NOTE]
>La SKU de puerta de enlace básica no admite el tipo de túnel OpenVPN.
>

[!INCLUDE [About gateway subnets](../../includes/vpn-gateway-about-gwsubnet-portal-include.md)]

[!INCLUDE [Create a gateway](../../includes/vpn-gateway-add-gw-rm-portal-include.md)]

## <a name="client-address-pool"></a><a name="addresspool"></a>Grupo de direcciones de clientes

El grupo de direcciones de cliente es un intervalo de direcciones IP privadas que usted especifica. Los clientes que se conectan de forma dinámica a través de una VPN de punto a sitio reciben una dirección IP de este intervalo. Use un intervalo de direcciones IP privadas que no se superponga a la ubicación local desde la que se va a conectar ni a la red virtual a la que desea conectarse. Si configura varios protocolos y SSTP es uno de ellos, el grupo de direcciones configurado se divide equitativamente entre los protocolos configurados.

1. Una vez creada la puerta de enlace de red virtual, navegue hasta la sección **Valores** de la página de la puerta de enlace de red virtual. En **Configuración**, seleccione **Configuración de punto a sitio**. Seleccione **Configure now** (Configurar ahora) para abrir la página de configuración.

   :::image type="content" source="./media/vpn-gateway-howto-point-to-site-resource-manager-portal/configure-now.png" alt-text="Captura de pantalla de la página de configuración de punto a sitio." lightbox="./media/vpn-gateway-howto-point-to-site-resource-manager-portal/configure-now.png":::
1. En la página **Configuración de punto a sitio**, puede configurar diversas opciones. En el cuadro **Grupo de direcciones**, agregue el intervalo de direcciones IP privadas que quiere usar. Los clientes VPN reciben de forma dinámica una dirección IP del intervalo que especifique. La máscara de subred mínima es de 29 bits para la configuración activa/pasiva, y de 28 bits para la configuración activa/activa.

   :::image type="content" source="./media/howto-point-to-site-multi-auth/address.jpg" alt-text="Captura de pantalla del grupo de direcciones.":::

1. Continúe con la sección siguiente para configurar los tipos de autenticación y de túnel.

## <a name="authentication-and-tunnel-types"></a><a name="type"></a>Tipos de autenticación y de túnel

En esta sección, configurará el tipo de autenticación y el tipo de túnel. En la página **Configuración de punto a sitio**, si no ve **Tipo de túnel** o **Tipo de autenticación**, significa que la puerta de enlace usa la SKU básica. La SKU básica no admite la autenticación de IKEv2 o RADIUS. Si quiere usar esta configuración, debe eliminar y volver a crear la puerta de enlace con una SKU de puerta de enlace diferente.

   :::image type="content" source="./media/howto-point-to-site-multi-auth/multiauth.jpg" alt-text="Captura de pantalla del tipo de autenticación.":::

### <a name="tunnel-type"></a><a name="tunneltype"></a>Tipo de túnel

En la página **Configuración de punto a sitio**, seleccione **OpenVPN (SSL)** como tipo de túnel.

### <a name="authentication-type"></a><a name="authenticationtype"></a>Tipo de autenticación

En **Tipo de autenticación**, seleccione los tipos que quiera. Las opciones son:

* Certificado de Azure
* RADIUS
* Azure Active Directory

En función de los tipos de autenticación seleccionados, verá distintos campos de configuración que tendrán que rellenarse. Rellene la información necesaria y seleccione **Guardar** en la parte superior de la página para guardar todas las opciones de configuración.

Para obtener más información sobre el tipo de autenticación, consulte:

* [Certificado de Azure](vpn-gateway-howto-point-to-site-resource-manager-portal.md#type)
* [RADIUS](point-to-site-how-to-radius-ps.md)
* [Azure Active Directory](openvpn-azure-ad-tenant.md)

## <a name="vpn-client-configuration-package"></a><a name="clientconfig"></a>Paquete de configuración de cliente VPN

Los clientes VPN deben configurarse con las opciones de configuración de cliente. El paquete de configuración de cliente VPN contiene archivos con los valores para configurar clientes VPN con el fin de conectarse a una red virtual a través de una conexión P2S.

Para obtener instrucciones sobre cómo generar e instalar archivos de configuración de un cliente VPN, use el artículo relativo a su configuración:

* [Creación e instalación de archivos de configuración del cliente VPN para configuraciones de punto a sitio con autenticación con certificados nativos de Azure](point-to-site-vpn-client-configuration-azure-cert.md).
* [Autenticación de Azure Active Directory: Configuración de un cliente VPN para conexiones P2S de protocolo OpenVPN](openvpn-azure-ad-client.md).

## <a name="point-to-site-faq"></a><a name="faq"></a>Preguntas más frecuentes sobre la conexión de punto a sitio

Esta sección contiene información sobre las preguntas frecuentes relacionadas con las configuraciones de punto a sitio. También puede ver las [preguntas frecuentes sobre VPN Gateway](vpn-gateway-vpn-faq.md) para información adicional sobre VPN Gateway.

[!INCLUDE [Point-to-Site FAQ](../../includes/vpn-gateway-faq-p2s-azurecert-include.md)]

## <a name="next-steps"></a>Pasos siguientes

Una vez completada la conexión, puede agregar máquinas virtuales a las redes virtuales. Consulte [Virtual Machines](../index.yml) para más información. Para más información acerca de las redes y las máquinas virtuales, consulte [Información general sobre las redes de máquina virtual con Linux y Azure](../virtual-machines/network-overview.md).

Para obtener información sobre solución de problemas de P2S, vea [Solución de problemas: conexión de punto a sitio de Azure](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).
