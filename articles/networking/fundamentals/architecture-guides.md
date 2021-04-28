---
title: Documentación de la arquitectura de redes de Azure
description: Obtenga información sobre la documentación de arquitectura de referencia disponible para los servicios de redes de Azure.
services: networking
author: KumudD
ms.service: virtual-network
ms.topic: article
ms.date: 03/30/2021
ms.author: kumud
ms.openlocfilehash: c98bdbb9fba2a6ba01e4ce590c36d57e68390f17
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107484802"
---
# <a name="azure-networking-architecture-documentation"></a>Documentación de la arquitectura de redes de Azure

En este artículo se proporciona información sobre las guías de arquitectura que pueden ayudarle a explorar los distintos servicios de redes de Azure disponibles para crear sus aplicaciones.

## <a name="networking-overview"></a>Introducción a las redes

En la tabla siguiente se incluyen artículos que proporcionan una introducción a las redes de un centro de información virtual y una topología de concentrador y radio en Azure.

|Título |Descripción  |
|---------|---------|
|[Centros de datos virtuales](/azure/architecture/vdc/networking-virtual-datacenter)   | Proporciona una perspectiva de red de un centro de datos virtual de Azure.       |
|[Topología en estrella tipo hub-and-spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)  |Proporciona información general sobre la topología de red de concentrador y radio en Azure junto con información sobre los límites de suscripción y varios centros.          |

## <a name="connect-to-azure-resources"></a>Conexión a los recursos de Azure

En la tabla siguiente se incluyen artículos sobre los servicios de redes de Azure que proporcionan conectividad entre los recursos de Azure, la conectividad desde una red local a los recursos de Azure y la bifurcación a la conectividad de rama en Azure.

|Título |Descripción  |
|---------|---------|
|[Adición de espacios de direcciones IP a redes virtuales emparejadas](/azure/architecture/networking/prefixes/add-ip-space-peered-vnet)     | Proporciona scripts que ayudan a agregar espacios de direcciones IP a redes virtuales emparejadas.        |
|[Conexión de servidores independientes mediante el adaptador de red de Azure](/azure/architecture/hybrid/azure-network-adapter)   | Muestra cómo conectar un servidor independiente local a redes virtuales de Microsoft Azure mediante el adaptador de red de Azure que se implementa por medio de Windows Admin Center.        |
|[Elección entre el emparejamiento de red virtual y las puertas de enlace de VPN](/azure/architecture/reference-architectures/hybrid-networking/vnet-peering)   | Compara dos formas de conectar redes virtuales en Azure: emparejamiento de red virtual y puertas de enlace VPN.        |
|[Conexión de una red local a Azure](/azure/architecture/reference-architectures/hybrid-networking/)  | Compara las opciones para conectar una red local a una instancia de Azure Virtual Network (VNet). Para cada opción, hay disponible una arquitectura de referencia más detallada.        |
|[Arquitectura de conectividad SD WAN con Azure Virtual WAN](../../virtual-wan/sd-wan-connectivity-architecture.md)|Describe las diferentes opciones de conectividad para la interconexión de una WAN definida por software privado (SD-WAN) con Azure Virtual WAN.|

## <a name="deploy-highly-available-applications"></a>Implementación de aplicaciones de alta disponibilidad

En la tabla siguiente se incluyen artículos en los que se describe cómo implementar aplicaciones de alta disponibilidad mediante una combinación de servicios de redes de Azure.

|Título |Descripción  |
|---------|---------|
|[Aplicación de niveles N para varias regiones](/azure/architecture/reference-architectures/n-tier/multi-region-sql-server))  | Describe una aplicación de niveles N para varias regiones que usa Traffic Manager para redirigir las solicitudes entrantes a una región principal. Si esa región deja de estar disponible, Traffic Manager conmuta por error en la región secundaria.      |
| [SaaS multiinquilino en Azure](https://docs.microsoft.com/azure/architecture/example-scenario/multi-saas/multitenant-saas)       |   Usa una solución de varios inquilinos que incluya una combinación de Front Door y Application Gateway.  Front Door ayuda a equilibrar la carga del tráfico entre regiones y Application Gateway redirige y equilibra la carga del tráfico internamente en la aplicación a varios servicios que satisfacen las necesidades empresariales de los clientes.  |
| [Aplicación web de varios niveles creada para lograr alta disponibilidad y recuperación ante desastres ](https://docs.microsoft.com/azure/architecture/example-scenario/infrastructure/multi-tier-app-disaster-recovery)        |      Implementa aplicaciones resistentes de varios niveles diseñadas para una alta disponibilidad y recuperación ante desastres. Si la región principal no está disponible, Traffic Manager conmuta por error en la región secundaria.  |
|[IaaS: aplicación web con base de datos relacional](/azure/architecture/high-availability/ref-arch-iaas-web-and-db)    |   Describe cómo usar los recursos distribuidos en varias zonas para proporcionar una arquitectura de alta disponibilidad para el hospedaje de una aplicación web de infraestructura como servicio (IaaS) y una base de datos de SQL Server.     |
|[Compartir ubicación en tiempo real mediante servicios de Azure sin servidor económicos](/azure/architecture/example-scenario/signalr/#azure-front-door)       |   Usa Azure Front Door para proporcionar a las aplicaciones una disponibilidad mayor que si se implementa en una única región. Si una interrupción regional afecta a la región primaria, puede usar Front Door realizar una conmutación por error a la región secundaria.      |
|[Aplicaciones virtuales de red de alta disponibilidad](/azure/architecture/reference-architectures/dmz/nva-ha)     | Muestra cómo implementar un conjunto de aplicaciones de red virtual (NVA) de alta disponibilidad en Azure.        |
|[Equilibrio de carga de varias regiones con Traffic Manager y Application Gateway](/azure/architecture/high-availability/reference-architecture-traffic-manager-application-gateway)     | Describe cómo implementar aplicaciones de varios niveles resistentes en varias regiones de Azure, con el fin de lograr disponibilidad y una sólida infraestructura de recuperación ante desastres.        |

## <a name="secure-your-network-resources"></a>Protección de los recursos de red

En la tabla siguiente se incluyen artículos que describen cómo proteger los recursos de red mediante los servicios de redes de Azure.

|Título |Descripción  |
|---------|---------|
|[Procedimientos recomendados de seguridad de red](../../security/fundamentals/network-best-practices.md) |Aborda un conjunto de procedimientos recomendados de Azure que sirven para mejorar la seguridad de la red.         |
[Guía de la arquitectura de Azure Firewall](/azure/architecture/example-scenario/firewalls/) | Proporciona un enfoque estructurado para el diseño de firewalls de alta disponibilidad en Azure mediante dispositivos virtuales de terceros.        |
|[Implementación de una red híbrida segura](/azure/architecture/reference-architectures/dmz/secure-vnet-dmz)     | Describe una arquitectura que implementa una DMZ, también conocida como red perimetral, entre la red local y una red virtual de Azure. Todo el tráfico entrante y saliente pasa a través de Azure Firewall.        |
|[Protección y control de cargas de trabajo con segmentación de nivel de red](/azure/architecture/reference-architectures/hybrid-networking/network-level-segmentation) | Describe los tres patrones comunes que se usan para organizar cargas de trabajo en Azure desde una perspectiva de red.   Cada uno de estos patrones proporciona un tipo diferente de aislamiento y conectividad.      |
|[Firewall y Application Gateway para redes virtuales](/azure/architecture/example-scenario/gateway/firewall-application-gateway) | Describe los servicios de seguridad de Azure Virtual Network, como Azure Firewall y Azure Application Gateway, cuándo usar cada servicio y las opciones de diseño de red que combinan ambos.      |

## <a name="next-steps"></a>Pasos siguientes

Más información sobre [Azure Virtual Network](../../virtual-network/virtual-networks-overview.md).
