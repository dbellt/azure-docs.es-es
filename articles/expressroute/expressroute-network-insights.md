---
title: Azure ExpressRoute Insights mediante Network Insights
description: Obtenga información de Azure ExpressRoute Insights mediante Network Insights.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 04/05/2021
ms.author: duau
ms.openlocfilehash: a5e01a4fc84f852b43e5a76114bebe145da10793
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106443596"
---
# <a name="azure-expressroute-insights-using-network-insights"></a>Azure ExpressRoute Insights mediante Network Insights

En este artículo se explica cómo Network Insights puede ayudarle a ver las métricas y configuraciones de ExpressRoute en un solo lugar. A través de Network Insights, puede ver mapas topológicos y paneles de mantenimiento que contienen información importante de ExpressRoute sin necesidad de realizar ninguna configuración adicional.

:::image type="content" source="./media/expressroute-network-insights/network-monitor-page.png" alt-text="Captura de pantalla de la página de aterrizaje del Monitor de redes." lightbox="./media/expressroute-network-insights/monitor-landing-page-expanded.png":::

## <a name="visualize-functional-dependencies"></a>Visualización de las dependencias funcionales

1. Vaya a la página *Azure Monitor* y seleccione *Redes*.

    :::image type="content" source="./media/expressroute-network-insights/monitor-page.png" alt-text="Captura de pantalla de la página de aterrizaje Monitor.":::

1. Seleccione la tarjeta *Circuitos ExpressRoute*. 

1. A continuación, seleccione el botón de topología del circuito que desea ver.

   :::image type="content" source="./media/expressroute-network-insights/monitor-landing-page.png" alt-text="Captura de pantalla de la página de aterrizaje de supervisión de ExpressRoute." lightbox="./media/expressroute-network-insights/monitor-landing-page-expanded.png"::: 

1. La vista de dependencia funcional proporciona una imagen clara de la configuración de ExpressRoute y describe la relación entre distintos componentes de ExpressRoute (emparejamientos, conexiones, puertas de enlace).

    :::image type="content" source="./media/expressroute-network-insights/topology-view.png" alt-text="Captura de pantalla de la vista de topología para Network Insights." lightbox="./media/expressroute-network-insights/topology-view-expanded.png":::

1. Mantenga el mouse sobre cualquier componente del mapa de topología para ver la información de configuración. Por ejemplo, mantenga el mouse sobre un componente de emparejamiento de ExpressRoute para ver detalles como el ancho de banda del circuito y la habilitación de la Global Reach.

    :::image type="content" source="./media/expressroute-network-insights/topology-hovered.png" alt-text="Captura de pantalla manteniendo el mouse sobre los recursos de la vista de topología." lightbox="./media/expressroute-network-insights/topology-hovered-expanded.png":::

## <a name="view-a-detailed-and-pre-loaded-metrics-dashboard"></a>Visualización de un panel de métricas detallado y cargado previamente

Una vez que revise la topología de la instalación de ExpressRoute mediante la vista de dependencia funcional, seleccione **Ver métricas detalladas** para navegar a la vista de métricas detalladas para comprender el rendimiento del circuito. Esta vista ofrece una lista organizada de recursos vinculados y un completo panel de métricas importantes de ExpressRoute.

En la sección **Recursos vinculados** se enumeran las puertas de enlace de ExpressRoute conectadas y los emparejamientos configurados, que puede seleccionar para ir a la página de recursos correspondiente.

:::image type="content" source="./media/expressroute-network-insights/linked-resources.png" alt-text="Captura de pantalla del recurso vinculado en la página supervisar.":::


La sección de **métricas de ExpressRoute** incluye gráficos de métricas importantes del circuito en todas las categorías de **Disponibilidad**, **Rendimiento**, **Caídas de paquetes** y **Métricas de puerta de enlace**.

### <a name="availability"></a>Disponibilidad

La pestaña *Disponibilidad* realiza un seguimiento de la disponibilidad de ARP y BGP, y traza los datos para el circuito como un conjunto y una conexión individual (principal y secundaria). 

:::image type="content" source="./media/expressroute-network-insights/arp-bgp-availability.png" alt-text="Captura de pantalla de los gráficos de métricas de disponibilidad." lightbox="./media/expressroute-network-insights/arp-bgp-availability-expanded.png":::

### <a name="throughput"></a>Throughput

Del mismo modo, la pestaña *Rendimiento* traza el rendimiento total del tráfico de entrada y salida del circuito en bits/segundo. También puede ver el rendimiento de las conexiones individuales y cada tipo de emparejamiento configurado.

:::image type="content" source="./media/expressroute-network-insights/throughput.png" alt-text="Captura de pantalla de gráficos de métricas de rendimiento." lightbox="./media/expressroute-network-insights/throughput-expanded.png":::

### <a name="packet-drops"></a>Pérdida de paquetes

La pestaña *Caídas de paquetes* traza los bits colocados por segundo para el tráfico de entrada y salida a través del circuito. Esta pestaña proporciona una manera sencilla de supervisar los problemas de rendimiento que pueden producirse si necesita o supera el ancho de banda del circuito.

:::image type="content" source="./media/expressroute-network-insights/dropped-packets.png" alt-text="Captura de pantalla de los gráficos de paquetes quitados." lightbox="./media/expressroute-network-insights/dropped-packets-expanded.png":::

### <a name="gateway-metrics"></a>Métricas de puerta de enlace

Por último, la pestaña métricas de puerta de enlace se rellena con los gráficos de métricas clave de una puerta de enlace de ExpressRoute seleccionada (en la sección recursos vinculados). Use esta pestaña cuando necesite supervisar la conectividad a redes virtuales específicas.

:::image type="content" source="./media/expressroute-network-insights/gateway-metrics.png" alt-text="Captura de pantalla de rendimiento de puerta de enlace y métricas de CPU." lightbox="./media/expressroute-network-insights/gateway-metrics-expanded.png":::

## <a name="next-steps"></a>Pasos siguientes

Configure su conexión ExpressRoute.
  
* Más información sobre [Azure ExpressRoute](expressroute-introduction.md), [Network insights](../azure-monitor/insights/network-insights-overview.md)y [Network Watcher](../network-watcher/network-watcher-monitoring-overview.md)
* [Creación y modificación de un circuito](expressroute-howto-circuit-arm.md)
* [Creación y modificación de la configuración de emparejamiento](expressroute-howto-routing-arm.md)
* [Vinculación de redes virtuales a circuitos ExpressRoute](expressroute-howto-linkvnet-arm.md)
* [Personalización de las métricas](expressroute-monitoring-metrics-alerts.md) y creación de un [Connection Monitor](../network-watcher/connection-monitor-overview.md)
