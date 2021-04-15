---
title: ¿Qué son los proveedores de seguridad asociados de Azure Firewall Manager?
description: Más información sobre los proveedores de seguridad asociados de Azure Firewall Manager
author: vhorne
ms.service: firewall-manager
services: firewall-manager
ms.topic: conceptual
ms.date: 03/30/2021
ms.author: victorh
ms.openlocfilehash: 622fde49a31105b2f66a678d3e55d48fabea9487
ms.sourcegitcommit: f5448fe5b24c67e24aea769e1ab438a465dfe037
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105966918"
---
# <a name="what-are-security-partner-providers"></a>¿Qué son los proveedores de seguridad asociados?

*Los proveedores de seguridad asociados* en Azure Firewall Manager permiten usar las mejores ofertas de seguridad como servicio (SECaaS) de terceros que ya conoce para proteger el acceso a internet para los usuarios.

Con una configuración rápida, puede proteger un centro de conectividad con un asociado de seguridad compatible, así como enrutar y filtrar el tráfico de Internet de sus redes virtuales (VNet) o ubicaciones de rama en una región. Esto se hace mediante la administración de rutas automatizada, sin configurar ni administrar rutas definidas por el usuario (UDR).

Puede implementar centros de conectividad protegidos configurados con el asociado de seguridad que prefiera en varias regiones de Azure para obtener conectividad y seguridad para los usuarios en cualquier lugar del mundo en dichas regiones. Con la capacidad de usar la oferta del asociado de seguridad para el tráfico de la aplicación de Internet/SaaS y Azure Firewall para el tráfico privado en los centros de conectividad de protegidos, ahora puede empezar a crear el perímetro de seguridad en Azure próximo a las aplicaciones y los usuarios distribuidos globalmente.

Los asociados de seguridad admitidos son **Zscaler**, **[Check Point](check-point-overview.md)** y **iboss**.

![Proveedores de seguridad asociados](media/trusted-security-partners/trusted-security-partners.png)

## <a name="key-scenarios"></a>Escenarios principales

Puede usar los asociados de seguridad para filtrar el tráfico de Internet en los siguientes escenarios:

- De red virtual (VNet) a Internet

   Use la protección de Internet avanzada con reconocimiento del usuario para sus cargas de trabajo en la nube que se ejecutan en Azure.

- De rama a Internet

   Use la conectividad de Azure y la distribución global para agregar fácilmente el filtrado NSaaS de terceros para escenarios de rama a Internet. Puede crear la red de tránsito global y el perímetro de seguridad mediante Azure Virtual WAN.

Se admiten los escenarios siguientes:
- Dos proveedores de seguridad en el centro de conectividad

   De red virtual o rama a Internet a través de un proveedor de seguridad asociado y el resto del tráfico (de radio a radio, de radio a rama, de rama a radio) a través de Azure Firewall.
- Proveedor único en el centro de conectividad

   - Todo el tráfico (de radio a radio, de radio a rama, de rama a radio, de red virtual o de rama a Internet) protegido por Azure Firewall<br>
      or
   - De red virtual o rama a Internet a través del proveedor de seguridad asociado

## <a name="best-practices-for-internet-traffic-filtering-in-secured-virtual-hubs"></a>Procedimientos recomendados para el filtrado de tráfico de Internet en los centros virtuales protegidos

El tráfico de Internet suele incluir tráfico web. No obstante, también incluye tráfico destinado a aplicaciones SaaS como Microsoft 365 y servicios PaaS públicos de Azure, como Azure Storage, Azure SQL, etc. A continuación se muestran los procedimientos recomendados para controlar el tráfico a estos servicios:

### <a name="handling-azure-paas-traffic"></a>Control del tráfico de PaaS de Azure
 
- Use Azure Firewall para la protección si el tráfico es principalmente de PaaS de Azure y el acceso a los recursos para las aplicaciones se puede filtrar mediante direcciones IP, FQDN, etiquetas de servicio o etiquetas FQDN.

- Use una solución de asociados de terceros en sus centros de conectividad si el tráfico es de acceso a las aplicaciones SaaS, si necesita filtrado que tenga en cuenta al usuario [por ejemplo, para cargas de trabajo de infraestructura de escritorio virtual (VDI)] o si necesita capacidades de filtrado de Internet avanzadas.

![Todos los escenarios de Azure Firewall Manager](media/trusted-security-partners/all-scenarios.png)

## <a name="handling-microsoft-365-traffic"></a>Tratamiento del tráfico de Microsoft 365

En escenarios de ubicación de rama distribuida globalmente, debe redirigir el tráfico de Microsoft 365 directamente en la rama antes de enviar el tráfico de Internet restante a su centro de conectividad protegido de Azure.

Para Microsoft 365, la latencia y el rendimiento de la red son fundamentales con el fin de lograr una experiencia del usuario satisfactoria. Para lograr estos objetivos de rendimiento y experiencia del usuario óptimos, los clientes deben implementar el escape de Microsoft 365 directo y local antes de considerar el enrutamiento del resto del tráfico de Internet mediante Azure.

Los [principios de conectividad de red de Microsoft 365](/microsoft-365/enterprise/microsoft-365-network-connectivity-principles) requieren que las principales conexiones de red de Microsoft 365 se enruten localmente desde la rama del usuario o el dispositivo móvil y directamente por Internet al punto de presencia de la red de Microsoft más cercano.

Además, las conexiones de Microsoft 365 están cifradas para proteger la privacidad y usan protocolos propios eficaces por motivos de rendimiento. En consecuencia, el hecho de someter estas conexiones a las soluciones de seguridad de nivel de red tradicionales puede ser poco práctico y tener un gran impacto. Por estos motivos, se recomienda encarecidamente que los clientes envíen el tráfico de Microsoft 365 directamente desde las ramas, antes de enviar el resto del tráfico mediante Azure. Microsoft se ha asociado con varios proveedores de soluciones SD-WAN, que se integran con Azure y Microsoft 365, y que permiten a los clientes habilitar fácilmente el desglose de Internet de Microsoft 365 directo y local. Para más información, consulte [¿Qué es Azure Virtual WAN?](../virtual-wan/virtual-wan-about.md)

## <a name="next-steps"></a>Pasos siguientes

[Implementación de una oferta de asociado de seguridad en un centro de conectividad protegido mediante Azure Firewall Manager](deploy-trusted-security-partner.md).
