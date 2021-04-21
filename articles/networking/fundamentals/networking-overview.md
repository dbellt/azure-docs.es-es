---
title: Información general sobre los servicios de red de Azure
description: Aprenda sobre los servicios de red de Azure, por ejemplo, los servicios de conectividad, protección de aplicaciones, entrega de aplicaciones y supervisión de la red.
services: networking
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.workload: infrastructure-services
ms.date: 04/07/2021
ms.author: kumud
ms.openlocfilehash: 34e31bacbb276feb5b11d1b955096877376e989b
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107107710"
---
# <a name="azure-networking-services-overview"></a>Información general sobre los servicios de red de Azure

Los servicios de red de Azure proporcionan diversas funcionalidades de red que se pueden usar conjuntamente o por separado. Haga clic en cualquiera de las siguientes funcionalidades principales para obtener más información acerca de ellas:
- [**Servicios de conectividad**](#connect). Conecte recursos de Azure y recursos del entorno local con cualquier combinación de estos servicios de red en Azure: Virtual Network (VNet), Virtual WAN, ExpressRoute, VPN Gateway, NAT Gateway de Virtual Network, Azure DNS, Peering Service y Azure Bastion.
- [**Servicios de protección de aplicaciones**](#protect). proteja sus aplicaciones con una combinación de estos servicios de red en Azure: Load Balancer, Private Link, protección contra DDoS, firewall, grupos de seguridad de red, Web Application Firewall o puntos de conexión de red virtual.
- [**Servicios de entrega de aplicaciones**](#deliver): Entregue aplicaciones en la red de Azure usando uno o varios de estos servicios de red en Azure: Content Delivery Network (CDN), Azure Front Door Service, Traffic Manager, Application Gateway, Internet Analyzer y Load Balancer.
- [**Supervisión de red**](#monitor): Supervise los recursos de red mediante uno o varios de estos servicios de red en Azure: Network Watcher, Supervisión de ExpressRoute, Azure Monitor o Punto de acceso de terminal de red virtual (TAP).

## <a name="connectivity-services"></a><a name="connect"></a>Servicios de conectividad
 
En esta sección se describen los servicios que proporcionan conectividad entre los recursos de Azure, conectividad desde una red en el entorno local a los recursos de Azure y conectividad de rama a rama en Azure: Virtual Network (VNet), ExpressRoute, VPN Gateway, Virtual WAN, Virtual Network NAT Gateway de Virtual Network, Azure DNS, Azure Peering Service y Azure Bastion.


### <a name="virtual-network"></a><a name="vnet"></a>Virtual Network

Azure Virtual Network (VNet) es el bloque de creación fundamental de una red privada en Azure. Puede usar una red virtual para:
- **Comunicarse entre recursos de Azure**: tanto las máquinas virtuales como otros tipos de recursos de Azure se pueden implementar en una red virtual, como Azure App Service Environment, Azure Kubernetes Service (AKS) y Azure Virtual Machine Scale Sets. Para ver una lista completa de los recursos de Azure que puede implementar en una red virtual, consulte [Integración de red virtual para los servicios de Azure](../../virtual-network/virtual-network-for-azure-services.md).
- **Comunicarse entre sí**: Las redes virtuales se pueden conectar entre sí, lo que permite que los recursos de cualquiera de ellas se comuniquen entre sí mediante el emparejamiento de red virtual. Las redes virtuales que conecte pueden estar en la misma región de Azure o en regiones distintas. Para más información, consulte [Emparejamiento de redes virtuales](../../virtual-network/virtual-network-peering-overview.md).
- **Comunicarse en Internet**: De manera predeterminada, todos los recursos de una red virtual tienen comunicación de salida hacia Internet. Para comunicarse con un recurso entrante, asígnele una dirección IP pública o un equilibrador de carga público. También puede usar [Direcciones IP públicas](../../virtual-network/virtual-network-public-ip-address.md) o [Load Balancer](../../load-balancer/load-balancer-overview.md) público para administrar las conexiones salientes.
- **Comunicarse con otras redes en el entorno local**: Puede conectar sus redes y equipos en el entorno local a una red virtual mediante [VPN Gateway](../../vpn-gateway/vpn-gateway-about-vpngateways.md) o [ExpressRoute](../../expressroute/expressroute-introduction.md).

Para más información, consulte [¿Qué es Azure Virtual Network?](../../virtual-network/virtual-networks-overview.md).

### <a name="expressroute"></a><a name="expressroute"></a>ExpressRoute
ExpressRoute permite ampliar las redes en el entorno local en la nube de Microsoft Azure a través de una conexión privada que facilita un proveedor de conectividad. Esta conexión es privada. El tráfico no pasa por Internet. Con ExpressRoute, se pueden establecer conexiones con servicios en la nube de Microsoft, como Microsoft Azure, Microsoft 365 y Dynamics 365.  Para más información, consulte [¿Qué es ExpressRoute?](../../expressroute/expressroute-introduction.md)

:::image type="content" source="./media/networking-overview/expressroute-connection-overview.png" alt-text="Información técnica de ExpressRoute" border="false":::

### <a name="vpn-gateway"></a><a name="vpngateway"></a>VPN Gateway
VPN Gateway ayuda a crear conexiones cifradas entre entornos locales a la red virtual desde ubicaciones locales, o bien a crear conexiones cifradas entre las redes virtuales. Hay distintas configuraciones disponibles para las conexiones de VPN Gateway, por ejemplo, de sitio a sitio, de punto a sitio o de red virtual a red virtual.
En el siguiente diagrama se ilustran varias conexiones VPN de sitio a sitio a la misma red virtual.

:::image type="content" source="./media/networking-overview/vpngateway-multisite-connection-diagram.png" alt-text="Conexiones de Azure VPN Gateway de sitio a sitio":::

Para más información acerca de los distintos tipos de conexiones VPN, consulte [VPN Gateway](../../vpn-gateway/vpn-gateway-about-vpngateways.md).

### <a name="virtual-wan"></a><a name="virtualwan"></a>Virtual WAN
Azure Virtual WAN es un servicio de redes que ofrece conectividad entre ramas automatizada y optimizada a y mediante Azure. Las regiones de Azure sirven como centros que se pueden elegir para conectar las distintas ramas. Puede aprovechar la red troncal de Azure para conectar también ramas y disfrutar de la conectividad de rama a red virtual. Azure Virtual WAN reúne muchos servicios de conectividad en la nube de Azure como la VPN de sitio a sitio, ExpressRoute y la VPN de usuario de punto a sitio en una única interfaz operativa. La conectividad con las redes virtuales de Azure se establece mediante el uso de conexiones de red virtual. Para más información, consulte [¿Qué es Azure Virtual WAN?](../../virtual-wan/virtual-wan-about.md).

:::image type="content" source="./media/networking-overview/virtualwan1.png" alt-text="Diagrama de Virtual WAN":::

### <a name="azure-dns"></a><a name="dns"></a>Azure DNS
Azure DNS es un servicio de hospedaje para dominios DNS que ofrece resolución de nombres mediante la infraestructura de Microsoft Azure. Al hospedar dominios en Azure, puede administrar los registros de DNS con las mismas credenciales, API, herramientas y facturación que con los demás servicios de Azure. Para más información, consulte [¿Qué es Azure DNS?](../../dns/dns-overview.md)

### <a name="azure-bastion"></a><a name="bastion"></a>Azure Bastion
Azure Bastion es un nuevo servicio PaaS totalmente administrado por la plataforma que se aprovisiona en redes virtuales. Proporciona una conectividad RDP/SSH segura e ininterrumpida a las máquinas virtuales directamente en Azure Portal a través de TLS. Cuando se conecta a través de Azure Bastion, las máquinas virtuales no necesitan una dirección IP pública. Para más información, consulte [¿Qué es Azure Bastion?](../../bastion/bastion-overview.md).

:::image type="content" source="./media/networking-overview/architecture.png" alt-text="Arquitectura de Azure Bastion":::

### <a name="virtual-network-nat-gateway"></a><a name="nat"></a>NAT Gateway de Virtual Network
La NAT (traducción de direcciones de red) de Virtual Network simplifica la conectividad a Internet de solo salida para redes virtuales. Cuando se configura en una subred, todas las conexiones salientes usan las direcciones IP públicas estáticas que se hayan especificado. La conectividad saliente es posible sin que el equilibrador de carga ni las direcciones IP públicas estén conectados directamente a máquinas virtuales. Para obtener más información, vea [¿Qué es la puerta de enlace NAT de Virtual Network?](../../virtual-network/nat-overview.md)

:::image type="content" source="./media/networking-overview/flow-map.png" alt-text="Puerta de enlace NAT de Virtual Network":::

### <a name="azure-peering-service"></a><a name="azurepeeringservice"></a> Azure Peering Service
Azure Peering Service mejora la conectividad de los clientes a Servicios en la nube de Microsoft, como Microsoft 365, Dynamics 365, servicios de software como servicio (SaaS), Azure o cualquier otro servicio de Microsoft accesible a través de la red pública de Internet. Para más información, vea [¿Qué es Azure Peering Service?](../../peering-service/about.md)

### <a name="azure-edge-zones"></a><a name="edge-zones"></a>Azure Edge Zones

Azure Edge Zone es una familia de ofertas de Microsoft Azure que permite el procesamiento de datos cerca del usuario. Puede implementar máquinas virtuales, contenedores y otros servicios de Azure seleccionados en Azure Edge Zone para satisfacer los requisitos de baja latencia y alto rendimiento de las aplicaciones.

### <a name="azure-orbital"></a><a name="orbital"></a>Azure Orbital

Azure Orbital es una estación terrestre basada en la nube totalmente administrada como servicio que le permite comunicarse con su nave espacial o constelaciones de satélites, descargar los datos y crear un vínculo superior, procesar los datos en la nube, encadenar servicios con servicios de Azure en escenarios únicos y generar productos para sus clientes. Este sistema se basa en la infraestructura global de Azure y en la red de fibra global de latencia baja.

## <a name="application-protection-services"></a><a name="protect"></a>Servicios de protección de aplicaciones

En esta sección se describen los servicios de red en Azure que ayudan a proteger los recursos de red. Proteja sus aplicaciones con una combinación de estos servicios de red en Azure: DDoS Protection, Private Link, Firewall, Web Application Firewall, grupos de seguridad de red y puntos de conexión de servicio de red virtual.

### <a name="ddos-protection"></a><a name="ddosprotection"></a>DDoS Protection 
[Azure DDoS Protection](../../ddos-protection/manage-ddos-protection.md) ofrece técnicas defensivas contra las amenazas de DDoS más sofisticadas. El servicio proporciona funcionalidades mejoradas de mitigación de DDoS a su aplicación y los recursos implementados en las redes virtuales. Además, los clientes que usan Azure DDoS Protection tienen acceso a la compatibilidad con la respuesta rápida de DDoS para atraer a expertos de DDoS durante un ataque activo.

:::image type="content" source="./media/networking-overview/ddos-protection.png" alt-text="Protección contra DDOS":::

### <a name="azure-private-link"></a><a name="privatelink"></a>Azure Private Link
[Azure Private Link](../../private-link/private-link-overview.md) permite acceder a los servicios PaaS de Azure (por ejemplo, Azure Storage y SQL Database) y a los servicios hospedados en Azure que son propiedad de los clientes, o a los servicios de asociados, a través de un punto de conexión privado de la red virtual.
El tráfico entre la red virtual y el servicio viaja por la red troncal de Microsoft. Ya no es necesario exponer el servicio a la red pública de Internet. Puede crear su propio servicio de vínculo privado en la red virtual y enviarlo a los clientes.

:::image type="content" source="./media/networking-overview/private-endpoint.png" alt-text="Introducción al punto de conexión privado":::

### <a name="azure-firewall"></a><a name="firewall"></a>Azure Firewall
Azure Firewall es un servicio de seguridad de red administrado y basado en la nube que protege los recursos de Azure Virtual Network. Con Azure Firewall, puede crear, aplicar y registrar directivas de conectividad de red y de aplicaciones de forma centralizada en suscripciones y redes virtuales. Azure Firewall usa una dirección IP pública estática para los recursos de red virtual, que permite que los firewall externos identifiquen el tráfico procedente de la red virtual. 

Para más información acerca de Azure Firewall, consulte la [documentación de Azure Firewall](../../firewall/overview.md).

:::image type="content" source="./media/networking-overview/firewall-threat.png" alt-text="Información general de firewalls":::

### <a name="web-application-firewall"></a><a name="waf"></a>Firewall de aplicaciones web
[Azure Web Application Firewall](../../web-application-firewall/overview.md) (WAF) de Azure proporciona protección a las aplicaciones web frente a vulnerabilidades de seguridad web comunes, como la inyección de SQL y el scripting entre sitios. WAF de Azure ofrece protección frente a las diez principales vulnerabilidades OWASP a través de reglas administradas. Además, los clientes también pueden configurar reglas personalizadas, que son reglas administradas por el cliente para proporcionar una protección adicional basada en el intervalo IP de origen, y atributos de solicitud como encabezados, cookies, campos de datos de formulario o parámetros de cadena de consulta.

Los clientes pueden optar por implementar [WAF de Azure con Application Gateway](../../web-application-firewall/ag/ag-overview.md), que proporciona protección regional para las entidades en el espacio de direcciones público y privado. Los clientes pueden optar por implementar [WAF de Azure con Front Door](../../web-application-firewall/afds/afds-overview.md), que proporciona protección en el perímetro de la red para los puntos de conexión públicos.

:::image type="content" source="./media/networking-overview/waf-overview.png" alt-text="Firewall de aplicaciones web":::

### <a name="network-security-groups"></a><a name="nsg"></a>Grupos de seguridad de red
Puede filtrar el tráfico de red hacia y desde los recursos de Azure en una red virtual de Azure con un grupo de seguridad de red. Para más información, consulte[Grupo de seguridad de red](../../virtual-network/network-security-groups-overview.md).

### <a name="service-endpoints"></a><a name="serviceendpoints"></a>Puntos de conexión de servicio
Los puntos de conexión del servicio Virtual Network (red virtual) extienden el espacio de direcciones privadas de la red virtual y la identidad de la red virtual a los servicios de Azure a través de una conexión directa. Los puntos de conexión permiten proteger los recursos de servicio de Azure críticos únicamente para las redes virtuales. El tráfico desde la red virtual al servicio de Azure siempre permanece en la red troncal de Microsoft Azure. Para obtener más información, consulte [puntos de conexión de servicio de red virtual](../../virtual-network/virtual-network-service-endpoints-overview.md).

:::image type="content" source="./media/networking-overview/vnet-service-endpoints-overview.png" alt-text="Puntos de conexión de servicio de red virtual":::

## <a name="application-delivery-services"></a><a name="deliver"></a>Servicios de entrega de aplicaciones

En esta sección se describen los servicios de red de Azure que ayudan a entregar aplicaciones: Content Delivery Network, Azure Front Door Service, Traffic Manager, Load Balancer y Application Gateway.

### <a name="content-delivery-network"></a><a name="cdn"></a>Content Delivery Network
Azure Content Delivery Network (CDN) ofrece a los desarrolladores una solución global para la entrega rápida de contenido con alto ancho de banda a los usuarios mediante el almacenamiento en caché del contenido en nodos físicos estratégicamente situados en todo el mundo. Para obtener más información sobre Azure CDN, vea [Azure Content Delivery Network](../../cdn/cdn-overview.md).

:::image type="content" source="./media/networking-overview/cdn-overview.png" alt-text="Azure CDN":::

### <a name="azure-front-door-service"></a><a name="frontdoor"></a>Azure Front Door Service
Azure Front Door Service permite definir, administrar y supervisar el enrutamiento global para el tráfico web mediante la optimización para obtener el mejor rendimiento y la conmutación por error global instantánea para alta disponibilidad. Con Front Door, las aplicaciones empresariales y de consumidor globales (de varias regiones) se pueden transformar en aplicaciones modernas personalizadas, sólidas y de alto rendimiento, API y contenido que lleguen a un público global mediante Azure. Para más información, consulte [Azure Front Door](../../frontdoor/front-door-overview.md).

:::image type="content" source="./media/networking-overview/front-door-visual-diagram.png" alt-text="Introducción a Front Door Service":::

### <a name="traffic-manager"></a><a name="trafficmanager"></a>Traffic Manager

Azure Traffic Manager es un equilibrador de carga de tráfico basado en DNS que le permite distribuir el tráfico de forma óptima a servicios de regiones de Azure globales, al tiempo que proporciona una alta disponibilidad y capacidad de respuesta. Traffic Manager proporciona diversos métodos de enrutamiento de tráfico para distribuir el tráfico, como la prioridad, la ponderación, el rendimiento, el método geográfico, el multivalor o la subred. Para más información sobre los métodos de enrutamiento del tráfico, consulte [Métodos de enrutamiento de Traffic Manager](../../traffic-manager/traffic-manager-routing-methods.md).

En el diagrama siguiente se muestra el enrutamiento basado en la prioridad del punto de conexión con Traffic Manager:

:::image type="content" source="./media/networking-overview/priority.png" alt-text="Método de enrutamiento de tráfico de &quot;prioridad&quot; de Azure Traffic Manager":::

Para más información sobre Traffic Manager, consulte [¿Qué es Azure Traffic Manager?](../../traffic-manager/traffic-manager-overview.md)

### <a name="load-balancer"></a><a name="loadbalancer"></a>Load Balancer
Azure Load Balancer proporciona equilibrio de carga de nivel 4 con latencia baja y rendimiento alto para todos los protocolos UDP y TCP. Administra conexiones entrantes y salientes. Puede configurar los puntos de conexión de carga equilibrada públicos e internos. Puede definir reglas para asignar conexiones entrantes a destinos de grupo de back-end con opciones de sondeo de estado TCP y HTTP para administrar la disponibilidad del servicio. Para más información sobre Load Balancer, consulte el artículo [Introducción a Load Balancer](../../load-balancer/load-balancer-overview.md).

La siguiente imagen muestra una aplicación de niveles múltiples con conexión a Internet que utiliza los equilibradores de carga externos e internos:

:::image type="content" source="./media/networking-overview/load-balancer.png" alt-text="Ejemplo de Azure Load Balancer":::

### <a name="application-gateway"></a><a name="applicationgateway"></a>Application Gateway
Azure Application Gateway es un equilibrador de carga de tráfico web que permite administrar el tráfico a las aplicaciones web. Se trata de un controlador de entrega de aplicaciones (ADC) como servicio que ofrece diversas funcionalidades de equilibrio de carga de capa 7 para las aplicaciones. Para más información, consulte [¿Qué es Azure Application Gateway?](../../application-gateway/overview.md).

En el diagrama siguiente se muestra el enrutamiento basado en la ruta de acceso de la dirección URL con Application Gateway.

:::image type="content" source="./media/networking-overview/figure1-720.png" alt-text="Ejemplo de Application Gateway":::

## <a name="network-monitoring-services"></a><a name="monitor"></a>Servicios de supervisión de red
En esta sección se describen los servicios de red de Azure que ayudan a supervisar los recursos de red: Network Watcher, Azure Monitor Network Insights, Azure Monitor, Supervisión de ExpressRoute y TAP de red virtual.

### <a name="network-watcher"></a><a name="networkwatcher"></a>Network Watcher
Azure Network Watcher proporciona herramientas para supervisar, diagnosticar, ver las métricas y habilitar o deshabilitar registros de recursos en una red virtual de Azure. Para más información, consulte [¿Qué es Azure Network Watcher?](../../network-watcher/network-watcher-monitoring-overview.md?toc=%2fazure%2fnetworking%2ftoc.json).

### <a name="azure-monitor-network-insights"></a>Azure Monitor Network Insights
Azure Monitor para redes ofrece una vista completa del estado y las métricas de todos los recursos de red implementados sin necesitar ninguna configuración. También proporciona acceso a funcionalidades de supervisión de red como [Connection Monitor](../../network-watcher/connection-monitor-overview.md), [registro de flujo para los grupos de seguridad de red](../../network-watcher/network-watcher-nsg-flow-logging-overview.md) y [Análisis de tráfico](../../network-watcher/traffic-analytics.md). Para más información, consulte [Azure Monitor Network Insights](../../azure-monitor/insights/network-insights-overview.md?toc=%2fazure%2fnetworking%2ftoc.json).

### <a name="expressroute-monitor"></a><a name="expressroutemonitor"></a>Supervisión de ExpressRoute
Para información sobre cómo ver las métricas de los circuitos ExpressRoute, los registros de recursos y las alertas, consulte [Supervisión, métricas y alertas de ExpressRoute](../../expressroute/expressroute-monitoring-metrics-alerts.md?toc=%2fazure%2fnetworking%2ftoc.json).
### <a name="azure-monitor"></a><a name="azuremonitor"></a>Azure Monitor
Azure Monitor maximiza la disponibilidad y el rendimiento de las aplicaciones con una completa solución que permite recopilar, analizar y administrar datos telemétricos tanto en la nube como en entornos locales. Esta solución le ayudará a entender cómo funcionan las aplicaciones y le permitirá identificar de manera proactiva los problemas que les afectan y los recursos de los que dependen. Para más información, consulte [Introducción a Azure Monitor](../../azure-monitor/overview.md?toc=%2fazure%2fnetworking%2ftoc.json).
### <a name="virtual-network-tap"></a><a name="vnettap"></a>TAP de red virtual
Azure Virtual Network TAP (punto de acceso del terminal) permite transmitir continuamente el tráfico de red de la máquina virtual a un recopilador de paquetes de red o a una herramienta de análisis. Un asociado de la [aplicación virtual de red](https://azure.microsoft.com/solutions/network-appliances/) proporciona el recopilador o la herramienta de análisis de la herramienta.

En la siguiente imagen se muestra cómo funciona TAP de red virtual:

:::image type="content" source="./media/networking-overview/virtual-network-tap-architecture.png" alt-text="Cómo funciona Virtual Network TAP":::

Para más información, consulte [TAP de red virtual](../../virtual-network/virtual-network-tap-overview.md).

## <a name="next-steps"></a>Pasos siguientes

- Cree su primera red virtual y conecte algunas máquinas virtuales a ella, siguiendo los pasos descritos en el artículo [Creación de su primera red virtual](../../virtual-network/quick-create-portal.md?toc=%2fazure%2fnetworking%2ftoc.json).
- Conecte su equipo a una red virtual siguiendo los pasos descritos en el artículo [Configuración de una conexión de punto a sitio](../../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md?toc=%2fazure%2fnetworking%2ftoc.json).
- Equilibre la carga de tráfico de Internet en los servidores públicos siguiendo los pasos descritos en el artículo [Creación de un equilibrador de carga con conexión a Internet](../../load-balancer/quickstart-load-balancer-standard-public-portal.md?toc=%2fazure%2fnetworking%2ftoc.json).