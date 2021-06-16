---
title: Uso de ExpressRoute para servicios de Microsoft 365 | Microsoft Docs
description: En este documento se describe de forma objetivo el uso del circuito ExpressRoute para servicios SaaS de Microsoft 365.
documentationcenter: na
services: expressroute
author: rambk
manager: tracsman
ms.service: expressroute
ms.topic: article
ms.workload: infrastructure-services
ms.date: 4/29/2021
ms.author: rambala
ms.openlocfilehash: 6e6b543741a27b7a99ef447e7e34fbec803403db
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111970505"
---
# <a name="using-expressroute-for-routing-microsoft-365-traffic"></a>Uso de ExpressRoute para el enrutamiento de tráfico de Microsoft 365

Un circuito ExpressRoute proporciona conectividad privada a la red troncal de Microsoft. 
* Ofrece *emparejamiento privado* para conectarse a puntos de conexión privados de la implementación de IaaS en regiones de Azure. 
* Además, ofrece *emparejamiento de Microsoft* para conectarse a puntos de conexión públicos de servicios IaaS, PaaS y SaaS en la red de Microsoft. 

Para más información sobre ExpressRoute, consulte el artículo [Introducción a ExpressRoute][ExR-Intro].


A menudo hay una confusión sobre si ExpressRoute se puede usar o no para enrutar tráfico SaaS de Microsoft 365. 

* Argumento a favor: ExpressRoute ofrece emparejamiento de Microsoft, con el que puede acceder a la mayoría de los puntos de conexión públicos de la red de Microsoft. De hecho, con un *filtro de ruta* puede seleccionar prefijos de servicio de Microsoft 365 que deben anunciarse a través del emparejamiento de Microsoft a la red local. Estos anuncios de rutas permiten enrutar tráfico del servicio de Microsoft 365 a través del circuito ExpressRoute. 
* Contraargumento: Microsoft 365 es un servicio distribuido. Está diseñado para permitir que los clientes de todo el mundo se conecten al servicio mediante Internet. Por lo tanto, se recomienda no usar ExpressRoute para Microsoft 365.

Los objetivos de este artículo son los siguientes: 
* Proporcionar un razonamiento técnico para los argumentos. 
* Analizar de forma objetivo cuándo usar ExpressRoute para enrutar tráfico de Microsoft 365 y cuándo no usarlo.

## <a name="network-requirements-of-microsoft-365-traffic"></a>Requisitos de red del tráfico de Microsoft 365
Microsoft 365 a menudo incluye tráfico en tiempo real, como llamadas de vídeo y voz, reuniones en línea y colaboración en tiempo real. Este tráfico en tiempo real tiene estrictos requisitos de rendimiento de red en términos de latencia y vibración. Dentro de ciertos límites de latencia de red, la vibración se puede controlar eficazmente mediante el búfer en el dispositivo cliente. La latencia de red es una función de distancia física que el tráfico necesita para viajar, vincular el ancho de banda y la latencia de procesamiento de red. 

## <a name="network-optimization-features-of-microsoft-365"></a>Características de optimización de red de Microsoft 365 

Microsoft se esfuerza por optimizar el rendimiento de red de todas las aplicaciones en la nube tanto en términos de arquitectura como de características. Para empezar, Microsoft posee una de las redes globales más grandes, que está optimizada para lograr el objetivo principal de ofrecer el mejor rendimiento de red. La red de Microsoft está definida por software y es una red de tipo "cold potato". En ese sentido, la red de tipo "cold potato", atrae y hace salir el tráfico lo más cerca posible de cliente-dispositivo o cliente-red. Además, la red de Microsoft es altamente redundante y de alta disponibilidad. Para obtener más información sobre la optimización de la arquitectura, vea [Cómo Microsoft crea su red global rápida y confiable][MGN].

Para abordar los estrictos requisitos de latencia de red, Microsoft 365 reduce la longitud de la ruta de las siguientes formas:
* mediante el enrutamiento dinámico de la conexión del usuario final al punto de entrada de Microsoft 365 más cercano, y 
* desde el punto de entrada, enrutándolos de forma eficaz dentro de la red global de Microsoft al centro de datos más cercano (y autorizado) de Microsoft 365.

Los puntos de entrada de Microsoft 365 los proporciona Azure Front Door (AFD). AFD es un servicio ampliamente distribuido presente en la red perimetral global de Microsoft y ayuda a crear aplicaciones SaaS rápidas, seguras y altamente escalables. Para comprender mejor cómo AFD acelera el rendimiento de las aplicaciones web, consulte [¿Qué es Azure Front Door?][AFD]. Al elegir el centro de datos de Microsoft 365 más cercano, Microsoft tiene en cuenta las regulaciones de soberanía de datos dentro de la región geopolítica.

## <a name="what-is-geo-pinning-connections"></a>¿Qué son las conexiones de asignación geográfica?

Entre un servidor cliente cuando se fuerza el flujo del tráfico a través de determinados dispositivos de red ubicados en una ubicación geográfica, se conoce como asignación geográfica de las conexiones de red. La arquitectura de red tradicional, con el principio de diseño subyacente de que los servidores de clientes se ubican estáticamente, normalmente ancla geográficamente las conexiones.
Por ejemplo, cuando fuerza las conexiones a Internet empresariales para que atraviesen la red corporativa y la salida desde una ubicación central (normalmente a través de un conjunto de servidores proxy o firewalls), se anclan geográficamente las conexiones a Internet.  

De forma similar, en la arquitectura de aplicaciones SaaS si fuerza el enrutamiento del tráfico a través de un centro de datos intermedio (por ejemplo, la seguridad en la nube) en una región o a través de uno o varios dispositivos de red intermedios (por ejemplo, ExpressRoute) en una ubicación específica, está anclando geográficamente las conexiones SaaS.

## <a name="when-not-to-use-expressroute-for-microsoft-365"></a>¿Cuándo no usar ExpressRoute para Microsoft 365?

Debido a su capacidad para reducir dinámicamente la longitud de la ruta y elegir dinámicamente el centro de datos del servidor más cercano en función de la ubicación de los clientes, se dice que Microsoft 365 está diseñado para Internet. Además, cierto tráfico de Microsoft 365 se enruta solo a través de Internet.
Si tiene sus clientes SaaS ampliamente distribuidos en una región o globalmente, y si ancla geográficamente las conexiones a una ubicación determinada, obliga a los clientes que se encuentran más lejos de la ubicación anclada geográficamente a experimentar una mayor latencia de red. Una mayor latencia de red da como resultado un rendimiento de red poco óptimo y un rendimiento deficiente de las aplicaciones.

Por lo tanto, en escenarios en los que tiene clientes o clientes SaaS ampliamente distribuidos con mucha movilidad, no querrá anclar geográficamente las conexiones de ninguna manera, por ejemplo, al forzar el tráfico a través de un circuito ExpressRoute en una ubicación de emparejamiento específica.


## <a name="when-to-use-expressroute-for-microsoft-365"></a>¿Cuándo usar ExpressRoute para Microsoft 365?

Estos son algunos de los motivos por los que puede que desee usar ExpressRoute para enrutar el tráfico de Microsoft 365:
* Los clientes SaaS se encuentran centrados en una ubicación geográfica y la manera más óptima de conectarse a la red global de Microsoft es a través de circuitos ExpressRoute.
* Los clientes SaaS se concentran en varias ubicaciones globales y cada ubicación tiene sus propios circuitos ExpressRoute que proporcionan una conectividad óptima a la red global de Microsoft.
* La ley le exige enrutar el tráfico enlazado a la nube a través de conexiones privadas.
* Debe enrutar todo el tráfico de SaaS a una ubicación centralizada anclada geográficamente (ya sea un centro de datos privado o público) y la manera óptima de conectar la ubicación centralizada a la red global de Microsoft es a través de ExpressRoute.
* Para algunos de los clientes SaaS estáticos, solo ExpressRoute proporciona una conectividad óptima, mientras que para los demás clientes se usa Internet.

Mientras usa ExpressRoute, puede aplicar el filtro de ruta asociado al emparejamiento de Microsoft de ExpressRoute para enrutar solo un subconjunto de servicios de Microsoft 365 o servicios PaaS de Azure a través del circuito ExpressRoute. Para más información, consulte [Tutorial: Configuración de filtros de ruta para el emparejamiento de Microsoft][ExRRF].

## <a name="next-steps"></a>Pasos siguientes

* Para comprender cómo fluyen las llamadas de Microsoft Teams y cómo optimizar la conectividad de red en diferentes escenarios, incluido el uso de ExpressRoute para obtener mejores resultados, consulte [Flujos de llamadas de Microsoft Teams][Teams].
* Si desea realizar una prueba para comprender los problemas de conectividad de Microsoft 365 para ubicaciones de oficina individuales, consulte [Prueba de conectividad de red de Microsoft 365][Microsoft 365-Test].
* Para establecer la línea de base y el historial de rendimiento para ayudar a detectar problemas emergentes de rendimiento de Microsoft 365, consulte [Ajuste del rendimiento de Office 365 mediante líneas de base y el historial de rendimiento][Microsoft 365perf].

<!--Link References-->
[ExR-Intro]: ./expressroute-introduction.md
[CreatePeering]: ./expressroute-howto-routing-portal-resource-manager.md
[MGN]: https://azure.microsoft.com/blog/how-microsoft-builds-its-fast-and-reliable-global-network/
[AFD]: ../frontdoor/front-door-overview.md
[ExRRF]: ./how-to-routefilter-portal.md
[Teams]: /microsoftteams/microsoft-teams-online-call-flows
[Microsoft 365-Test]: https://connectivity.office.com/
[Microsoft 365perf]: /microsoft-365/enterprise/performance-tuning-using-baselines-and-history