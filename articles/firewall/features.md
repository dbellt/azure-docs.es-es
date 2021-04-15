---
title: Características de Azure Firewall
description: Conozca las características de Azure Firewall
services: firewall
author: vhorne
ms.service: firewall
ms.topic: conceptual
ms.date: 04/02/2021
ms.author: victorh
ms.openlocfilehash: 8dbfb23d4314f8ceb13ad36ca9733e446e176090
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106278191"
---
# <a name="azure-firewall-features"></a>Características de Azure Firewall

[Azure Firewall](overview.md) es un servicio de seguridad de red administrado y basado en la nube que protege los recursos de red virtual de Azure.

![Información general de firewalls](media/overview/firewall-threat.png)

Azure Firewall incluye las siguientes características:

- Alta disponibilidad integrada
- Zonas de disponibilidad
- Escalabilidad a la nube sin restricciones
- Reglas de filtrado de FQDN de aplicación
- Reglas de filtrado de tráfico de red
- Etiquetas FQDN
- Etiquetas de servicio
- Información sobre amenazas
- Compatibilidad con SNAT saliente
- Compatibilidad con DNAT entrante
- Varias direcciones IP públicas
- Registro de Azure Monitor
- Tunelización forzada
- Categorías web (versión preliminar)
- Certificaciones

## <a name="built-in-high-availability"></a>Alta disponibilidad integrada

Gracias a la alta disponibilidad integrada, no se necesita ningún equilibrador de carga adicional y no es necesario configurar nada.

## <a name="availability-zones"></a>Zonas de disponibilidad

Azure Firewall se puede configurar durante la implementación para abarcar varias zonas de disponibilidad y aumentar la disponibilidad. Con Availability Zones, la disponibilidad aumenta a un tiempo de actividad del 99,99 %. Para más información, consulte el [Acuerdo de Nivel de Servicio (SLA)](https://azure.microsoft.com/support/legal/sla/azure-firewall/v1_0/) de Azure Firewall. El SLA de tiempo de actividad del 99,99 % se ofrece cuando se seleccionan dos o más zonas de Availability Zones.

También puede asociar Azure Firewall a una zona específica solo por motivos de proximidad, mediante el SLA del 99,95 % estándar del servicio.

No hay ningún costo adicional por el firewall implementado en una zona de disponibilidad. Sin embargo, hay costos adicionales para las transferencias de datos entrantes y salientes asociadas con Availability Zones. Para más información, consulte [Detalles de precios de ancho de banda](https://azure.microsoft.com/pricing/details/bandwidth/).

Availability Zones de Azure Firewall está disponible en las regiones que lo admiten. Para más información, consulte [Regiones que admiten Availability Zones en Azure](../availability-zones/az-region.md).

> [!NOTE]
> Availability Zones solo se puede configurar durante la implementación. No se puede configurar un firewall existente para incluirlo.

Para más información sobre Availability Zones, consulte [¿Qué son las zonas de disponibilidad en Azure?](../availability-zones/az-overview.md)

## <a name="unrestricted-cloud-scalability"></a>Escalabilidad a la nube sin restricciones

Azure Firewall puede escalarse verticalmente todo lo que sea necesario para acoger los flujos de tráfico de red cambiantes, por lo que no es necesario elaborar un presupuesto para el tráfico en su momento álgido.

## <a name="application-fqdn-filtering-rules"></a>Reglas de filtrado de FQDN de aplicación

Puede limitar el tráfico HTTP/S o el tráfico de Azure SQL saliente a una lista especificada de nombres de dominio completos (FQDN) que incluye caracteres comodín. Esta característica no requiere terminación de TLS.

## <a name="network-traffic-filtering-rules"></a>Reglas de filtrado de tráfico de red

Puede crear reglas de filtrado de red para *permitir* o *denegar* por dirección IP de origen y destino, puerto y protocolo. Azure Firewall tiene estado completo, de modo que puede distinguir los paquetes legítimos de diferentes tipos de conexiones. Las reglas se aplican y se registran en varias suscripciones y redes virtuales.

## <a name="fqdn-tags"></a>Etiquetas FQDN

Con las [etiquetas FQDN](fqdn-tags.md), puede permitir fácilmente que el tráfico de red conocido del servicio de Azure atraviese el firewall. Por ejemplo, supongamos que quiere permitir el tráfico de red de Windows Update a través del firewall. Puede crear una regla de aplicación e incluir la etiqueta de Windows Update. Ahora, el tráfico de red de Windows Update puede fluir a través del firewall.

## <a name="service-tags"></a>Etiquetas de servicio

Una [etiqueta de servicio](service-tags.md) representa un grupo de prefijos de direcciones IP que ayudan a reducir la complejidad de la creación de reglas de seguridad. No puede crear su propia etiqueta de servicio ni especificar qué direcciones IP se incluyen dentro de una etiqueta. Microsoft administra los prefijos de direcciones que incluye la etiqueta de servicio y actualiza automáticamente esta a medida que las direcciones cambian.

## <a name="threat-intelligence"></a>Información sobre amenazas

El filtrado basado en [inteligencia sobre amenazas](threat-intel.md) puede habilitarse para que el firewall alerte y deniegue el tráfico desde y hacia los dominios y las direcciones IP malintencionados. La direcciones IP y los dominios proceden de la fuente Inteligencia sobre amenazas de Microsoft.

## <a name="outbound-snat-support"></a>Compatibilidad con SNAT saliente

Todas las direcciones IP de tráfico de red virtual salientes se convierten a direcciones IP públicas de Azure Firewall (traducción de direcciones de red de origen). Puede identificar y permitir el tráfico que se origina en la red virtual y se dirige a los destinos de Internet remotos. Azure Firewall no aplica SNAT cuando la dirección IP de destino es un intervalo de direcciones IP privadas por [IANA RFC 1918](https://tools.ietf.org/html/rfc1918). 

Si su organización usa un intervalo de direcciones IP públicas para las redes privadas, Azure Firewall aplicará SNAT al tráfico para una de las direcciones IP privadas de firewall en AzureFirewallSubnet. Puede configurar Azure Firewall de modo que **no** aplique SNAT al intervalo de direcciones IP públicas. Para más información, consulte [Aplicación de SNAT por parte de Azure Firewall a intervalos de direcciones IP privadas](snat-private-range.md).

Puede supervisar el uso de los puertos SNAT en las métricas de Azure Firewall. Obtenga más información y vea nuestra recomendación sobre el uso de puertos SNAT en nuestra [documentación sobre los registros y las métricas de firewall](logs-and-metrics.md#metrics).

## <a name="inbound-dnat-support"></a>Compatibilidad con DNAT entrante

El tráfico entrante de Internet a la dirección IP pública de firewall se traduce (traducción de direcciones de red de destino) y se filtra para las direcciones IP privadas en las redes virtuales.

## <a name="multiple-public-ip-addresses"></a>Varias direcciones IP públicas

Puede asociar [varias direcciones IP públicas](deploy-multi-public-ip-powershell.md) (hasta 250) con el firewall.

Esto admite los siguientes escenarios:

- **DNAT**: puede traducir varias instancias de puerto estándar para los servidores back-end. Por ejemplo, si tiene dos direcciones IP públicas, puede traducir el puerto TCP 3389 (RDP) para ambas direcciones IP.
- **SNAT**: hay más puertos disponibles para las conexiones SNAT salientes, lo que reduce la posibilidad de que se agoten los puertos SNAT. En este momento, Azure Firewall selecciona aleatoriamente la dirección IP pública de origen que se usará para una conexión. Si dispone de algún filtro de nivel inferior de la red, deberá permitir todas las direcciones IP públicas asociadas con el firewall. Considere la posibilidad de usar un [prefijo de dirección IP pública](../virtual-network/public-ip-address-prefix.md) para simplificar esta configuración.

## <a name="azure-monitor-logging"></a>Registro de Azure Monitor

Todos los eventos se integran en Azure Monitor, lo que permite archivar registros en una cuenta de almacenamiento, transmitir eventos al centro de eventos o enviarlos a los registros de Azure Monitor. Para obtener ejemplos de registro de Azure Monitor, consulte [Registros de Azure Monitor para Azure Firewall](./firewall-workbook.md).

Para más información, consulte el [Tutorial: Supervisión de métricas y registros de Azure Firewall](./firewall-diagnostics.md). 

El libro de Azure Firewall proporciona un lienzo flexible para el análisis de datos de Azure Firewall. Puede utilizarlo para crear informes visuales completos en Azure Portal. Para más información, consulte [Supervisión de registros mediante el libro de Azure Firewall](firewall-workbook.md).

## <a name="forced-tunneling"></a>Tunelización forzada

Puede configurar Azure Firewall para enrutar todo el tráfico vinculado a Internet a un próximo salto designado, en lugar de ir directamente a Internet. Por ejemplo, puede tener un servidor perimetral local u otra aplicación virtual de red (NVA) para procesar el tráfico de red antes de que pase a Internet. Para más información, consulte [Tunelización forzada de Azure Firewall](forced-tunneling.md).

## <a name="web-categories-preview"></a>Categorías web (versión preliminar)

Las categorías web hacen que los administradores puedan permitir o denegar el acceso de los usuarios a categorías de sitios web como, por ejemplo, sitios web de apuestas, de redes sociales u otros. Las categorías web se incluyen en Azure Firewall Estándar, pero son más avanzadas en la versión preliminar de Azure Firewall Prémium. Frente a la funcionalidad de categorías web de la SKU Estándar que coincide con la categoría basada en un nombre de dominio completo, la SKU Prémium coincide con la categoría según la dirección URL completa tanto para el tráfico HTTP como para el HTTPS. Para más información sobre la versión preliminar de Azure Firewall Prémium, ve [Características en versión preliminar de Azure Firewall Prémium](premium-features.md).

Por ejemplo, si Azure Firewall intercepta una solicitud HTTPS para `www.google.com/news`, se espera la siguiente categorización: 

- Firewall Estándar: solo se examinará la parte del nombre de dominio completo, por lo que `www.google.com` se clasificará como *Motor de búsqueda*. 

- Firewall Prémium: se examinará la dirección URL completa, por lo que se `www.google.com/news` clasificará como *Noticias*.

Las categorías se organizan en función de la gravedad en **Responsabilidad**, **Ancho de banda alto**, **Uso empresarial**, **Pérdida de productividad**, **Navegación general** y **Sin categoría**.

### <a name="categorization-change"></a>Cambio de categorización

Puede solicitar un cambio de categorización si:

 - Cree que un FQDN o una dirección URL deben estar en otra categoría 
 
o bien 

- Tiene una categoría sugerida para un FQDN o una dirección URL sin categoría

Le agradecemos que envíe una solicitud en [https://aka.ms/azfw-webcategories-request](https://aka.ms/azfw-webcategories-request).

### <a name="category-exceptions"></a>Excepciones de las categorías

Puede crear excepciones a las reglas de la categoría web. Cree una colección de reglas de permiso o denegación independiente con una prioridad más alta dentro del grupo de recopilación de reglas. Por ejemplo, puede configurar una colección de reglas que permita `www.linkedin.com` con prioridad 100, con una colección de reglas que deniegue las **redes sociales** con prioridad 200. Esto crea la excepción para la categoría web **Redes sociales** predefinida.



## <a name="certifications"></a>Certificaciones

Azure Firewall es compatible con la industria de tarjetas de pago (PCI), los controles de organización de servicio (SOC), la Organización internacional de normalización (ISO) y con ICSA Labs. Para más información, consulte [Certificaciones de cumplimiento de Azure Firewall](compliance-certifications.md).

## <a name="next-steps"></a>Pasos siguientes

- [Características de la versión preliminar de Azure Firewall Prémium](premium-features.md)