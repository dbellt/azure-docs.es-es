---
title: Preguntas frecuentes de Azure DDoS Protection Standard
description: Preguntas frecuentes sobre el estándar de Azure DDoS Protection, que ayuda a proporcionar defensa contra ataques DDoS.
services: virtual-network
documentationcenter: na
author: aletheatoh
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/28/2020
ms.author: yitoh
ms.openlocfilehash: 894d7b07ee0d9b46b18bf42e50772af3fae427e1
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108072898"
---
# <a name="azure-ddos-protection-standard-frequently-asked-questions"></a>Preguntas frecuentes de Azure DDoS Protection Standard

En este artículo se responden las preguntas más frecuentes sobre Azure DDoS Protection Standard. 

## <a name="what-is-a-distributed-denial-of-service-ddos-attack"></a>¿Qué es un ataque de denegación de servicio distribuido (DDoS)?
La denegación de servicio distribuido, o DDoS, es un tipo de ataque en el que un atacante envía más solicitudes a una aplicación de los que la aplicación es capaz de gestionar. El efecto resultante es el agotamiento de los recursos, lo que afecta a la disponibilidad de la aplicación y a la capacidad de atender a sus clientes. En los últimos años, el sector ha experimentado un fuerte aumento de los ataques, que cada vez son más sofisticados y de mayor magnitud. Los ataques DDoS pueden ir dirigidos a cualquier punto de conexión que esté públicamente accesible a través de Internet.

## <a name="what-is-azure-ddos-protection-standard-service"></a>¿Qué es el servicio Azure DDoS Protection Standard?
Azure DDoS Protection, junto con los procedimientos recomendados de diseño de aplicaciones, proporciona características mejoradas de mitigación DDoS para protegerse de los ataques DDoS. Se ajusta de forma automática para proteger los recursos específicos de Azure de una red virtual. La protección se puede habilitar fácilmente en cualquier red virtual nueva o existente y no requiere cambios en las aplicaciones ni los recursos. Tiene varias ventajas sobre el servicio básico, incluidos el registro, las alertas y la telemetría. Para más información, vea [Introducción a Azure DDoS Protection Estándar](ddos-protection-overview.md). 

## <a name="how-does-pricing-work"></a>¿Cómo funcionan los precios?
Los planes de protección contra DDoS tienen un cargo mensual fijo de 2944 USD al mes que abarca hasta 100 direcciones IP públicas. La protección de recursos adicionales supondrá un costo adicional de 30 USD por recurso al mes. 

En un inquilino, se puede usar un único plan de protección contra DDoS en varias suscripciones, por lo que no es necesario crear más de un plan de protección contra DDoS.

Para obtener más información, consulte [Precios de Azure DDoS Protection Estándar](https://azure.microsoft.com/pricing/details/ddos-protection/).

## <a name="is-the-service-zone-resilient"></a>¿Es el servicio resistente a zonas?
Sí. De manera predeterminada, Azure DDoS Protection es resistente a zonas.

## <a name="how-do-i-configure-the-service-to-be-zone-resilient"></a>¿Cómo se configura el servicio para que sea resistente a zonas?
No es necesaria ninguna configuración de parte del cliente para habilitar la resistencia de zonas. La resistencia de zonas para los recursos de Azure DDoS Protection está disponible de manera predeterminada y se administra desde el propio servicio.

## <a name="what-about-protection-at-the-service-layer-layer-7"></a>¿Qué ocurre con la protección en el nivel de servicio (nivel 7)?
Los clientes pueden usar el servicio Azure DDoS Protection en combinación con un firewall de aplicaciones web (WAF) para la protección tanto en el nivel de red (niveles 3 y 4, que Azure DDoS Protection Standard ofrece) como en el nivel de aplicación (nivel 7, que un WAF ofrece). Entre las ofertas de WAF se incluyen la [SKU de WAF de Application Gateway](../web-application-firewall/ag/ag-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) de Azure, así como ofertas de firewall de aplicaciones web de terceros disponibles en [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps?page=1&search=web%20application%20firewall).

## <a name="are-services-unsafe-in-azure-without-the-service"></a>¿Los servicios no son seguros en Azure sin el servicio?
Los servicios que se ejecutan en Azure están protegidos de forma inherente por Azure DDoS Protection Basic, que está implementado para proteger la infraestructura de Azure. Sin embargo, la protección que se aplica a la infraestructura tiene un umbral mucho más alto del que la mayoría de las aplicaciones puede controlar y no proporciona telemetría ni alertas, por lo que, aunque la plataforma pueda percibir un volumen de tráfico como inofensivo, puede ser devastador para la aplicación que lo recibe. 

Al incorporar el servicio Azure DDoS Protection Standard, la aplicación obtiene una supervisión dedicada para detectar ataques y umbrales específicos de la aplicación. Un servicio se protegerá con un perfil optimizado para el volumen de tráfico previsto, lo que proporciona una defensa mucho más intensa contra los ataques DDoS.

## <a name="what-are-the-supported-protected-resource-types"></a>¿Cuáles son los tipos de recursos protegidos admitidos?
Las direcciones IP públicas en redes virtuales basadas en ARM son actualmente el único tipo de recurso protegido. Los servicios de PaaS (multiinquilino) no se admiten actualmente. Vea [Arquitecturas de referencia de Azure DDoS Protection Estándar](ddos-protection-reference-architectures.md) para más información.

## <a name="are-classicrdfe-protected-resources-supported"></a>¿Se admiten los recursos clásicos/RDFE protegidos?
En la versión preliminar solo se admiten los recursos protegidos basados en ARM. No se admiten las VM de implementaciones clásicas/RDFE. La compatibilidad no está planeada actualmente para los recursos clásicos/RDFE. Vea [Arquitecturas de referencia de Azure DDoS Protection Estándar](ddos-protection-reference-architectures.md) para más información.

## <a name="can-i-protect-my-paas-resources-using-ddos-protection"></a>¿Puedo proteger mis recursos de PaaS mediante DDoS Protection?
Actualmente no se admiten las direcciones IP públicas conectadas a servicios de PaaS de una sola VIP y de varios inquilinos. Entre los ejemplos de recursos no admitidos se incluyen las VIP de almacenamiento, las VIP del centro de eventos y las aplicaciones de App Services y Cloud Services. Vea [Arquitecturas de referencia de Azure DDoS Protection Estándar](ddos-protection-reference-architectures.md) para más información.

## <a name="can-i-protect-my-on-premise-resources-using-ddos-protection"></a>¿Puedo proteger mis recursos locales mediante DDoS Protection?
Debe tener los puntos de conexión públicos de su servicio asociados a una red virtual en Azure para que se habiliten para DDoS Protection. Los diseños de ejemplo incluyen:
- Sitios web (IaaS) en Azure y bases de datos de back-end en el centro de datos local. 
- Application Gateway en Azure (DDoS Protection habilitado en App Gateway/WAF) y sitios web en centros de datos locales.

Vea [Arquitecturas de referencia de Azure DDoS Protection Estándar](ddos-protection-reference-architectures.md) para más información.

## <a name="can-i-register-a-domain-outside-of-azure-and-associate-that-to-a-protected-resource-like-vm-or-elb"></a>¿Puedo registrar un dominio fuera de Azure y asociarlo a un recurso protegido como VM o ELB?
En el caso de los escenarios de IP pública, el servicio DDoS Protection admitirá cualquier aplicación independientemente de dónde se registre u hospede el dominio asociado, siempre y cuando la dirección IP pública asociada se hospede en Azure. 

## <a name="can-i-manually-configure-the-ddos-policy-applied-to-the-vnetspublic-ips"></a>¿Puedo configurar manualmente la directiva de DDoS aplicada a las redes virtuales o las direcciones IP públicas?
No, lamentablemente, la personalización de la directiva no está disponible en este momento.

## <a name="can-i-allowlistblocklist-specific-ip-addresses"></a>¿Puedo autorizar o bloquear direcciones IP específicas?
No, lamentablemente, la configuración manual no está disponible en este momento.

## <a name="how-can-i-test-ddos-protection"></a>¿Cómo puedo probar DDoS Protection?
Consulte [Pruebas mediante simulaciones](test-through-simulations.md).

## <a name="how-long-does-it-take-for-the-metrics-to-load-on-portal"></a>¿Cuánto tiempo se tarda en cargar las métricas en el portal?
Las métricas deberían ser visibles en el portal en un plazo de 5 minutos. Si el recurso está sufriendo un ataque, se empezarán a mostrar otras métricas en el portal en un plazo de 5 a 7 minutos. 

## <a name="does-the-service-store-customer-data"></a>¿Almacena el servicio datos de los clientes?
No, Azure DDoS Protection no almacena datos de los clientes.
    
