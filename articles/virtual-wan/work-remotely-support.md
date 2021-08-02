---
title: Azure Virtual WAN y cómo trabajar de forma remota
description: Conozca cómo puede aprovechar Azure Virtual WAN para permitir el trabajo remoto debido a la pandemia de COVID-19.
services: virtual-wan
author: reyandap
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 05/26/2021
ms.author: cherylmc
ms.openlocfilehash: fb3af5f19e81ccf21a9ca38fe45514aa0bdf471e
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110580256"
---
# <a name="azure-virtual-wan-and-supporting-remote-work"></a>Azure Virtual WAN y compatibilidad para trabajar de forma remota

>[!NOTE]
>En este artículo se describe cómo puede aprovechar Azure Virtual WAN, Azure, la red de Microsoft y el ecosistema de asociados de Azure para trabajar de forma remota y mitigar los problemas de red que está experimentando debido a la crisis del COVID-19.
>

¿Tiene problemas para proporcionar conectividad a usuarios remotos?
¿De repente ve la necesidad de admitir un aumento de usuarios más allá de lo que tenía planeado?
¿Necesita que el usuario se conecte desde casa y que no solo acceda a la nube, sino que también pueda conectarse al entorno local?
¿Necesita que los usuarios remotos puedan obtener acceso a recursos detrás de una red WAN privada?
¿Tiene la necesidad de que los usuarios obtengan acceso a recursos dentro de la nube sin tener que configurar la conectividad entre regiones?
Dado que esta pandemia global ha comportado unos cambios sin precedentes, el equipo de Azure Virtual WAN está aquí para que pueda satisfacer sus necesidades de conectividad.

Azure Virtual WAN es un servicio de red que aporta muchas funciones de red, seguridad y enrutamiento para proporcionar una única interfaz operativa. Estas funcionalidades incluyen la conectividad de ramas (a través de la automatización de la conectividad desde dispositivos de asociados de Virtual WAN, como SD-WAN o VPN CPE), la conectividad VPN de sitio a sitio, la conectividad VPN de usuarios remotos (de punto a sitio), la conectividad privada (ExpressRoute), la conectividad en la nube (conectividad transitiva para redes virtuales), la interconectividad de VPN ExpressRoute, el enrutamiento, el firewall de Azure, el cifrado para la conectividad privada, etc. No es necesario disponer de todos estos casos de uso para empezar a usar Virtual WAN. Puede empezar a trabajar con un solo caso de uso y ajustar la red a medida que crezca.

![Diagrama de Virtual WAN](./media/virtual-wan-about/virtualwan1.png)

Ahora que hablamos de usuarios remotos, veamos lo que necesita para poner en marcha su red:

## <a name="set-up-remote-user-connectivity"></a><a name="connectivity"></a>Configuración de la conectividad de usuarios remotos

Puede conectarse a los recursos en Azure por medio de una conexión de IPsec/IKE (IKEv2) u OpenVPN. Este tipo de conexión requiere que se configure un cliente VPN para el usuario remoto. Este cliente puede ser el [cliente VPN de Azure](https://go.microsoft.com/fwlink/?linkid=2117554) o el cliente OpenVPN, o cualquier cliente que admita IKEv2. Para más información, consulte [Creación de una conexión de punto a sitio](virtual-wan-point-to-site-portal.md).

## <a name="connectivity-from-the-remote-user-to-on-premises"></a><a name="remote user connectivity"></a>Conectividad del usuario remoto al entorno local

Tiene dos opciones aquí:

* Configurar la conectividad de sitio a sitio con cualquier dispositivo VPN existente. Al conectar el dispositivo VPN con IPsec al centro de conectividad de Azure Virtual WAN, la interconectividad entre la VPN de usuario de punto a sitio (usuario remoto) y la VPN de sitio a sitio es automática. Para obtener más información sobre cómo configurar una VPN de sitio a sitio del dispositivo VPN local a Azure Virtual WAN, consulte [Creación de una conexión de sitio a sitio mediante Virtual WAN](virtual-wan-site-to-site-portal.md).

* Conectar el circuito de ExpressRoute al centro de conectividad de Virtual WAN. La conexión de un circuito de ExpressRoute requiere la implementación de una puerta de enlace de ExpressRoute en Virtual WAN. En cuanto haya implementado una, la interconectividad entre la VPN de usuario de punto a sitio y el usuario de ExpressRoute será automática. Para crear la conexión de ExpressRoute, consulte [Create an ExpressRoute connection using Virtual WAN](virtual-wan-expressroute-portal.md) (Creación de una conexión de ExpressRoute mediante Virtual WAN). Puede usar un circuito de ExpressRoute existente para conectarse a Azure Virtual WAN.

## <a name="existing-basic-virtual-wan-customer"></a><a name="basic vWAN"></a>Cliente existente de Virtual WAN Básico

Virtual WAN Básico solo proporciona una VPN de sitio a sitio. Para que los usuarios remotos se puedan conectar, deberá actualizar la red WAN virtual a Virtual WAN Estándar. Para conocer los pasos para actualizar una WAN virtual, consulte [Actualización de una WAN virtual de plan Básico a Estándar](upgrade-virtual-wan.md).

## <a name="additional-information"></a><a name="other considerations"></a>Información adicional

Virtual WAN admite un centro de conectividad por región o ubicación. Para obtener información de las ubicaciones, consulte el de ubicación, consulte el artículo acerca de los [asociados y ubicaciones de Virtual WAN](virtual-wan-locations-partners.md). Cada centro de conectividad admite un máximo de 10.000 conexiones de usuarios remotos, 1.000 conexiones de ramas, cuatro circuitos de ExpressRoute y un máximo de 500 conexiones de redes virtuales. A medida que escale verticalmente los usuarios remotos, si tiene alguna pregunta, no dude en buscar ayuda enviando un correo electrónico a azurevirtualwan@microsoft.com. Si necesita soporte técnico, asegúrese de abrir una incidencia de soporte técnico en Azure Portal y recibirá ayuda en breve.

## <a name="faq"></a><a name="faq"></a>P+F

[!INCLUDE [Virtual WAN FAQ](../../includes/virtual-wan-faq-include.md)]

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre Virtual WAN, consulte la [información general sobre Virtual WAN](virtual-wan-about.md).