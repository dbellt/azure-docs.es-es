---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 12/03/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 72ddd0b6cd6c3e12417d3698c403f89312b531f4
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "67186136"
---
Las preguntas más frecuentes sobre red virtual a red virtual se aplican a las conexiones de VPN Gateway. Para más información sobre el emparejamiento de redes virtuales, consulte [Emparejamiento de redes virtuales](../articles/virtual-network/virtual-network-peering-overview.md).

### <a name="does-azure-charge-for-traffic-between-vnets"></a>¿Cobra Azure por el tráfico entre redes virtuales?

El tráfico entre redes virtuales dentro de la misma región es gratuito en ambas direcciones cuando se usa una conexión de puerta de enlace de VPN. El tráfico de salida de red virtual a red virtual entre regiones se cobra según las tarifas de transferencia de datos de salida entre redes virtuales en función de las regiones de origen. Para más información, consulte la [página de precios de VPN Gateway](https://azure.microsoft.com/pricing/details/vpn-gateway/). Si conecta las redes virtuales mediante el emparejamiento de redes virtuales en lugar de una puerta de enlace de red virtual, consulte los [precios de redes virtuales](https://azure.microsoft.com/pricing/details/virtual-network/).

### <a name="does-vnet-to-vnet-traffic-travel-across-the-internet"></a>¿Viaja el tráfico entre dos redes virtuales a través de Internet?

No. Viaja por la red troncal de Microsoft Azure, no por Internet.

### <a name="can-i-establish-a-vnet-to-vnet-connection-across-azure-active-directory-aad-tenants"></a>¿Puedo establecer una conexión entre dos redes virtuales a través de los inquilinos de Azure Active Directory (AAD)?

Sí, las conexiones entre dos redes virtuales que usan puertas de enlace de VPN de Azure funcionan en los inquilinos de AAD.

### <a name="is-vnet-to-vnet-traffic-secure"></a>¿Es seguro el tráfico entre dos redes virtuales?

Sí, se protege mediante cifrado IPsec/IKE.

### <a name="do-i-need-a-vpn-device-to-connect-vnets-together"></a>¿Necesito un dispositivo VPN para conectar redes virtuales?

No. La conexión simultánea de varias redes virtuales de Azure no requiere dispositivos VPN, a menos que sea necesaria la conectividad entre locales.

### <a name="do-my-vnets-need-to-be-in-the-same-region"></a>¿Deben estar mis redes virtuales en la misma región?

No. Las redes virtuales pueden estar en la misma región de Azure o en regiones distintas (ubicaciones).

### <a name="if-the-vnets-arent-in-the-same-subscription-do-the-subscriptions-need-to-be-associated-with-the-same-active-directory-tenant"></a>Si las redes virtuales no están en la misma suscripción, ¿las suscripciones tienen que estar asociadas con el mismo inquilino de Active Directory?

No.

### <a name="can-i-use-vnet-to-vnet-to-connect-virtual-networks-in-separate-azure-instances"></a>¿Puedo usar una conexión entre redes virtuales para conectar redes virtuales en instancias independientes de Azure? 

No. La conexión entre redes virtuales permite conectar redes virtuales dentro de la misma instancia de Azure. Por ejemplo, no puede crear una conexión entre instancias de Azure del gobierno estadounidense, chino o alemán con una instancia global de Azure. Considere la posibilidad de usar una conexión VPN de sitio a sitio en estos escenarios.

### <a name="can-i-use-vnet-to-vnet-along-with-multi-site-connections"></a>¿Puedo usar las conexiones entre dos redes virtuales con conexiones multisitio?

Sí. La conectividad de red virtual se puede usar de forma simultánea con VPN de varios sitios.

### <a name="how-many-on-premises-sites-and-virtual-networks-can-one-virtual-network-connect-to"></a>¿A cuántos sitios locales y redes virtuales se puede conectar una red virtual?

Consulte la tabla [Requisitos de la puerta de enlace](../articles/vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md#requirements).

### <a name="can-i-use-vnet-to-vnet-to-connect-vms-or-cloud-services-outside-of-a-vnet"></a>¿Puedo usar la conexión entre dos redes virtuales para conectar máquinas virtuales o servicios en la nube fuera de una red virtual?

No. VNet a VNet admite la conexión de redes virtuales. No admite la conexión de máquinas virtuales ni servicios en la nube que no estén en una red virtual.

### <a name="can-a-cloud-service-or-a-load-balancing-endpoint-span-vnets"></a>¿Abarca redes virtuales un servicio en la nube o un punto de conexión de equilibrio de carga?

No. Un servicio en la nube o un punto de conexión de equilibrio de carga no puede abarcar varias redes virtuales, aunque estas estén conectadas entre sí.

### <a name="can-i-use-a-policybased-vpn-type-for-vnet-to-vnet-or-multi-site-connections"></a>¿Puedo usar un tipo de VPN PolicyBased para las conexiones entre dos redes virtuales o multisitio?

No. Las conexiones entre dos redes virtuales y multisitio requieren puertas de enlace de VPN de Azure con tipos de VPN RouteBased (antes denominado enrutamiento dinámico).

### <a name="can-i-connect-a-vnet-with-a-routebased-vpn-type-to-another-vnet-with-a-policybased-vpn-type"></a>¿Puedo conectar una red virtual con un tipo de VPN RouteBased a otra red virtual con un tipo de VPN PolicyBased?

No, ambas redes virtuales TIENEN que usar VPN basadas en enrutamiento (antes denominado enrutamiento dinámico).

### <a name="do-vpn-tunnels-share-bandwidth"></a>¿Comparten ancho de banda los túneles de VPN?

Sí. Todos los túneles de VPN de la red virtual comparten el ancho de banda disponible en la puerta de enlace de VPN de Azure y el mismo SLA de tiempo de actividad de puerta de enlace de VPN en Azure.

### <a name="are-redundant-tunnels-supported"></a>¿Se admiten túneles redundantes?

Los túneles redundantes entre dos redes virtuales se admiten cuando la puerta de enlace de una red virtual está configurada como activa-activa.

### <a name="can-i-have-overlapping-address-spaces-for-vnet-to-vnet-configurations"></a>¿Puedo tener espacios de direcciones superpuestos para configuraciones de red virtual a red virtual?

No. No puede tener intervalos de direcciones de IP superpuestos.

### <a name="can-there-be-overlapping-address-spaces-among-connected-virtual-networks-and-on-premises-local-sites"></a>¿Puede haber espacios de direcciones superpuestos entre las redes virtuales conectadas y los sitios locales?

No. No puede tener intervalos de direcciones de IP superpuestos.



