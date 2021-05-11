---
title: Migración de puertas de enlace de VPN del modelo clásico a Resource Manager
titleSuffix: Azure VPN Gateway
description: Obtenga información sobre cómo migrar recursos de VPN Gateway del modelo de implementación clásica al modelo de implementación de Resource Manager.
services: vpn-gateway
author: amsriva
manager: rossort
ms.service: vpn-gateway
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 02/06/2020
ms.author: amsriva
ms.openlocfilehash: 0d7d4dda75fba86bf5d624ea277b8866490ea608
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108291197"
---
# <a name="vpn-gateway-classic-to-resource-manager-migration"></a>Migración de VPN Gateway (clásico) a Resource Manager
Las instancias de VPN Gateway ahora se pueden migrar del modelo de implementación clásica al de Resource Manager. Puede leer más información sobre las [características y ventajas](../azure-resource-manager/management/overview.md) de Azure Resource Manager. En este artículo, se explica cómo migrar desde implementaciones clásicas al nuevo modelo basado en Resource Manager. 

Las instancias de VPN Gateway se migran como parte de la migración de red virtual del modelo clásico al de Resource Manager. Esta migración se realiza con una red virtual cada vez. No se necesita nada más en cuanto a herramientas o requisitos previos para la migración. Los pasos de migración son idénticos a los de la migración de red virtual existente y están documentados en la [página de migración de recursos de IaaS](../virtual-machines/migration-classic-resource-manager-ps.md). No existe ningún tiempo de inactividad para la ruta de acceso de datos durante la migración y, por tanto, las cargas de trabajo existentes seguirán funcionando sin que se pierda la conectividad local durante la migración. La dirección IP pública asociada a la puerta de enlace de VPN no cambia durante el proceso de migración. Esto significa que no tendrá que volver a configurar el enrutador local una vez completada la migración.  

El modelo en Resource Manager es diferente del modelo clásico y se compone de puertas de enlace de red virtual, puertas de enlace de red local y recursos de conexión. Estos representan la puerta de enlace de VPN en sí y el sitio local representa el espacio de direcciones local y la conectividad entre ambos respectivamente. Una vez completada la migración, las puertas de enlace no estarán disponibles en el modelo clásico y todas las operaciones de administración en puertas de enlace de red virtual, puertas de enlace de red local y objetos de conexión se deben llevar a cabo mediante el modelo de Resource Manager.

## <a name="supported-scenarios"></a>Escenarios admitidos
Los escenarios de conectividad VPN más habituales están cubiertos en la migración del modelo clásico al de Resource Manager. Entre los escenarios compatibles, se incluyen:

* Conectividad de punto a sitio
* Conectividad de sitio a sitio con conexión de VPN Gateway a una ubicación local
* Conectividad de red virtual a red virtual entre dos redes virtuales con puertas de enlace de VPN
* Varias redes virtuales conectadas a la misma ubicación local
* Conectividad multisitio
* Redes virtuales con la tunelización forzada habilitada

Entre los escenarios no admitidos, se incluyen:  

* No se admite actualmente la red virtual con la puerta de enlace de ExpressRoute y VPN Gateway.
* Escenarios de tránsito donde las extensiones de máquina virtual están conectadas a servidores locales. A continuación, se detallan las limitaciones de conectividad VPN de tránsito.

> [!NOTE]
> La validación de CIDR en el modelo de Resource Manager es más estricta que la del modelo clásico. Antes de comenzar la migración, asegúrese de que los intervalos de direcciones clásicas dadas se ajusten al formato CIDR válido. Se puede validar CIDR mediante alguno de los validadores de CIDR habituales. Si se migran sitios locales o redes virtuales con intervalos de CIDR no válidos, se producirá un estado de error.
> 
> 

## <a name="vnet-to-vnet-connectivity-migration"></a>Migración de la conectividad de red virtual a red virtual
La conectividad de red virtual a red virtual del modelo clásico se lograba mediante la creación de una representación en el sitio local de la red virtual conectada. Los clientes tenían que crear dos sitios locales que representaban las dos redes virtuales que debían estar conectadas entre sí. Después, estos sitios se conectaban a las redes virtuales correspondientes mediante un túnel IPsec para establecer la conectividad entre ambas redes virtuales. Este modelo presenta problemas de facilidad de uso, ya que cualquier cambio en el intervalo de direcciones de una red virtual se debe mantener también en la representación en sitio local correspondiente. En el modelo de Resource Manager, esta solución ya no es necesaria. La conexión entre ambas redes virtuales se puede lograr directamente mediante el tipo de conexión "Vnet2Vnet" en el recurso de conexión. 

![Captura de pantalla de la migración de red virtual a red virtual.](./media/vpn-gateway-migration/migration1.png)

Durante la migración de red virtual, se detecta que la entidad conectada a la puerta de enlace de VPN de la red virtual actual es otra red virtual. Una vez completada la migración de ambas redes virtuales, asegúrese de que ya no se vean dos sitios locales que representen la otra red virtual. El modelo clásico con dos puertas de enlace de VPN, dos sitios locales y dos conexiones entre ellos se transforma en el modelo de Resource Manager con dos puertas de enlace de VPN y dos conexiones de tipo Vnet2Vnet.

## <a name="transit-vpn-connectivity"></a>Conectividad de VPN de tránsito
Puede configurar puertas de enlace de VPN en una topología de forma que la conectividad local para una red virtual se logre mediante la conexión a otra red virtual que esté conectada directamente a un sitio local. Se trata de la conectividad de VPN de tránsito, en la que las instancias en la primera red virtual se conectan a recursos locales mediante el tránsito a la puerta de enlace de VPN en la red virtual conectada que está en conexión directa con el sitio local. Para lograr esta configuración en el modelo de implementación clásica, debe crear un sitio local que cuente con prefijos agregados que representen tanto la red virtual conectada como el espacio de direcciones local. Este sitio local de representación se conecta después a la red virtual para lograr la conectividad de tránsito. Este modelo clásico también presenta problemas de facilidad de uso similares, ya que cualquier cambio en el intervalo de direcciones local también debe mantenerse en el sitio local que representa la agregación de la red virtual y el sitio local. La introducción de la compatibilidad con BGP en las puertas de enlace admitidas por Resource Manager simplifica el uso, ya que las puertas de enlace conectadas pueden aprender rutas desde el sitio local sin necesidad de modificar los prefijos manualmente.

![Captura de pantalla de escenario de enrutamiento de tránsito.](./media/vpn-gateway-migration/migration2.png)

Puesto que se transforma la conectividad de red virtual a red virtual sin necesidad de sitios locales, el escenario de tránsito pierde la conectividad local para la red virtual que está conectada de forma indirecta al sitio local. Se puede mitigar la pérdida de conectividad de las siguientes dos maneras, una vez completada la migración: 

* Habilite BGP en las puertas de enlace de VPN que están conectadas entre sí y con el sitio local. Al habilitarse BGP, se restaura la conectividad sin realizar ningún otro cambio de configuración, ya que las rutas se aprenden y anuncian entre las puertas de enlace de red virtual. Tenga en cuenta que la opción de BGP solo está disponible en las SKU Estándar y superiores.
* Establezca una conexión explícita desde la red virtual afectada hacia la puerta de enlace de red local que representa la ubicación local. Esto también podría requerir que se cambie la configuración del enrutador local para crear y configurar el túnel IPsec.

## <a name="next-steps"></a>Pasos siguientes
Después de leer sobre la compatibilidad con la migración de puerta de enlace de VPN, vaya a la página sobre la [migración de recursos de IaaS admitida por la plataforma del modelo clásico al de Resource Manager](../virtual-machines/migration-classic-resource-manager-ps.md) para empezar.