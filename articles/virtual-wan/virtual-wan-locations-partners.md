---
title: Asociados y ubicaciones de Azure Virtual WAN | Microsoft Docs
description: En este artículo se incluyen listas de ubicaciones de concentradores y de asociados de Azure Virtual WAN.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 04/27/2021
ms.author: cherylmc
ms.custom: references_regions
ms.openlocfilehash: bdcb675e379b5ab94f8404966477f4d2df56400c
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108163938"
---
# <a name="virtual-wan-partners-and-virtual-hub-locations"></a>Ubicaciones de concentradores virtuales y asociados de Azure Virtual WAN

En este artículo se proporciona información sobre las regiones admitidas de Virtual WAN y los asociados de conectividad en concentradores virtuales.

Azure Virtual WAN es un servicio de redes que ofrece conectividad de rama a rama automatizada y optimizada mediante Azure. Azure Virtual WAN permite conectar y configurar los dispositivos de rama para comunicarse con Azure. La conexión y la configuración se pueden realizar manualmente o con dispositivos de proveedores mediante un asociado de Virtual WAN. El uso de dispositivos de asociados permite la administración de la configuración, facilita el uso y simplifica la conectividad.

La conectividad desde el dispositivo local se establece de forma automática para el concentrador virtual. Un concentrador virtual es una red virtual administrada por Microsoft. El concentrador contiene varios puntos de conexión de servicio para habilitar la conectividad de la red local (vpnsite). Solo se puede tener un concentrador por región.

## <a name="branch-ipsec-connectivity-automation-from-partners"></a><a name="automation"></a>Automatización de la conectividad IPSec de ramas de los asociados

Los dispositivos que se conectan a Azure Virtual WAN tienen automatización integrada para conectarse. Normalmente, esto se configura en la interfaz de usuario de administración de dispositivos (o equivalente), donde se configura la administración de configuración y de conectividad entre el dispositivo de la rama VPN a un punto de conexión de VPN de concentrador virtual de Azure (puerta de enlace VPN).

La automatización de alto nivel siguiente se configura en la consola del dispositivo/centro de administración:

* Permisos adecuados para que el dispositivo acceda al grupo de recursos de Azure Virtual WAN
* Carga del dispositivo de la rama en Azure Virtual WAN
* Descarga automática de la información de conectividad de Azure
* Configuración del dispositivo de rama local 

Algunos asociados de conectividad pueden extender la automatización para incluir la creación de la red virtual del concentrador virtual de Azure y VPN Gateway. Si quiere obtener más información acerca de la automatización, consulte [Directrices de automatización para asociados de Azure Virtual WAN](virtual-wan-configure-automation-providers.md).

## <a name="branch-ipsec-connectivity-partners"></a><a name="partners"></a>Asociados de la conectividad de IPSec de ramas

[!INCLUDE [partners](../../includes/virtual-wan-partners-include.md)]

Los siguientes asociados están en nuestra hoja de ruta en función de una hoja de términos firmada por las empresas, que indica el ámbito del trabajo para automatizar la conectividad de IPsec entre el dispositivo asociado y las instancias de VPN Gateway de Azure Virtual WAN: 128 Technologies, Arista, F5 Networks, Oracle SD-WAN (Talari) y SharpLink.

## <a name="partners-with-integrated-virtual-hub-offerings"></a>Asociados con ofertas del Centro virtual integrado

Además de tener conectividad IPSec automatizada de sucursales, algunos asociados ofrecen **aplicaciones virtuales de red (NVA)** que se pueden integrar directamente en el centro de Azure Virtual WAN.  Esto permite a los clientes finalizar sus conexiones de rama en un dispositivo de terceros compatible con el centro virtual.  

Los asociados que ofrecen NVA en el centro de Virtual WAN deben:

* Tener implementada la automatización de conectividad IPSec desde su dispositivo de rama e incorporar su oferta NVA al centro de Azure Virtual WAN.
* Tener una oferta de Dispositivo virtual de red existente disponible en Azure Marketplace.

Si es un asociado y tiene preguntas sobre las NVA administradas en la oferta del centro virtual, póngase en contacto con nosotros por correo electrónico a través de la dirección vwannvaonboarding@microsoft.com.

## <a name="integrated-virtual-hub-nva-partners"></a>Asociados de NVA del Centro virtual integrado

Estos asociados tienen ofertas de **Aplicaciones administradas** que ahora están disponibles para su implementación en el centro de Virtual WAN.

|Asociados|Guía de implementación/instrucciones/configuración|
|---|---|
|[Barracuda Networks](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/barracudanetworks.barracuda_cloudgenwan_gateway?tab=Overviewus/marketplace/apps/barracudanetworks.barracuda_cloudgenwan_gateway?tab=Overview)| [Guía de implementación de WAN de Barracuda CloudGen](https://campus.barracuda.com/product/cloudgenwan/doc/91980640/deployment/)|
|[VWAN del enrutador del servicio en la nube de Cisco (CSR)](https://aka.ms/ciscoMarketPlaceOffer)| Durante la versión preliminar pública de la instancia de WAN de Cisco Cloud Services (CSR) en el Centro de VWAN, Cisco requiere que el cliente final se registre como un cliente de Cisco EFT (versión de prueba temprana); para ello, debe enviar un correo electrónico a vwan_public_preview@external.cisco.com y la solicitud de la guía de implementación de vManage. 
|[VMware SD-WAN en el centro de Virtual WAN](https://sdwan.vmware.com/partners/microsoft) | Durante la versión preliminar pública de VMware SD-WAN en el centro de VWAN, VMware requiere que el cliente se registre mediante el envío de un correo electrónico a vhubsupport@vmware.com. [Guía de implementación de VMware SD-WAN en el centro de Virtual WAN](https://kb.vmware.com/s/article/82746)|

Los siguientes asociados están preparados para agregar NVA en las ofertas del centro virtual en un futuro próximo: Redes de Aviatrix, Citrix y Versa.

## <a name="locations"></a><a name="locations"></a>Ubicaciones

[!INCLUDE [Virtual WAN regions file](../../includes/virtual-wan-regions-include.md)]

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre Virtual WAN, consulte [Preguntas más frecuentes sobre Virtual WAN](virtual-wan-faq.md).

* Para más información sobre cómo automatizar la conectividad a Azure Virtual WAN, consulte [Directrices de automatización de asociados de Azure Virtual WAN](virtual-wan-configure-automation-providers.md).
