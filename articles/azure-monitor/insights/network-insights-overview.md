---
title: Azure Monitor Network Insights
description: Una introducción a Azure Monitor Network Insights, que ofrece una vista completa del estado y las métricas de todos los recursos de red implementados sin ninguna configuración.
ms.topic: conceptual
author: KumudD
ms.author: kumud
ms.date: 11/25/2020
ms.openlocfilehash: f401c143f1f5a9352b56b80f9e473bd7fa819245
ms.sourcegitcommit: f5448fe5b24c67e24aea769e1ab438a465dfe037
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105968261"
---
# <a name="azure-monitor-network-insights"></a>Azure Monitor Network Insights

Azure Monitor Network Insights ofrece una vista completa del [estado](../../service-health/resource-health-checks-resource-types.md) y las [métricas](../essentials/metrics-supported.md) de todos los recursos de red implementados sin necesitar ninguna configuración. También proporciona acceso a funcionalidades de supervisión de red como [Connection Monitor](../../network-watcher/connection-monitor-overview.md), [registro de flujo para los grupos de seguridad de red (NSG)](../../network-watcher/network-watcher-nsg-flow-logging-overview.md) y [Análisis de tráfico](../../network-watcher/traffic-analytics.md). Además, proporciona otras características de [diagnóstico](../../network-watcher/network-watcher-monitoring-overview.md#diagnostics).

Azure Monitor Network Insights se estructura en torno a estos componentes clave de supervisión:
- [Estado y métricas de red](#networkhealth)
- [Conectividad](#connectivity)
- [Tráfico](#traffic)
- [Kit de herramientas de diagnóstico](#diagnostictoolkit)

## <a name="network-health-and-metrics"></a><a name="networkhealth"></a>Estado y métricas de la red

La página de **información general** de Azure Monitor Network Insights ofrece una manera fácil de visualizar el inventario de los recursos de red junto con las alertas y el estado de los recursos. Se divide en cuatro áreas funcionales principales: búsqueda y filtrado, estado y métricas de los recursos, alertas y vista de dependencias.

[![Captura de pantalla que muestra la página de información general](media/network-insights-overview/overview.png)](media/network-insights-overview/overview.png#lightbox)

### <a name="search-and-filtering"></a>Búsqueda y filtrado
Para personalizar la vista del estado de los recursos y las alertas, use filtros como **Suscripción**, **Grupo de recursos** y **Tipo**.

Puede usar el cuadro de búsqueda para buscar los recursos y sus recursos asociados. Por ejemplo, una IP pública está asociada a una puerta de enlace de aplicación. La búsqueda del nombre DNS de la IP pública devolverá tanto la IP pública como la puerta de enlace de aplicación asociada:

[![Captura de pantalla que muestra los resultados de la búsqueda de Azure Monitor Network Insights](media/network-insights-overview/search.png)](media/network-insights-overview/search.png#lightbox)


### <a name="resource-health-and-metrics"></a>Métricas y estado de los recursos
En el ejemplo siguiente, cada icono representa un tipo de recurso. El icono muestra el número de instancias de ese tipo de recurso implementadas en todas las suscripciones seleccionadas. También se muestra el estado de mantenimiento del recurso. En este ejemplo, hay implementadas 105 conexiones de ER y VPN. 103 son correctas y hay 2 no disponibles.

![Captura de pantalla que muestra el estado y las métricas de los recursos en Azure Monitor Network Insights.](media/network-insights-overview/resource-health.png)

Si selecciona las conexiones de ER y VPN que no está disponibles, verá una vista de métrica: 

![Captura de pantalla que muestra la vista de métrica de Azure Monitor Network Insights.](media/network-insights-overview/metric-view.png)

Puede seleccionar cualquier elemento en la vista de cuadrícula. Seleccione el icono de la columna **Estado** para obtener el estado de los recursos de esa conexión. Seleccione el valor de la columna **Alerta** para ir a la página de alertas y métricas correspondiente a la conexión. 

### <a name="alerts"></a>Alertas
El cuadro **Alerta** que se encuentra al lado derecho de la página ofrece una vista de todas las alertas generadas para los recursos seleccionados en todas las suscripciones. Seleccione el número de alertas para ir a una página de alertas detalladas.

### <a name="dependency-view"></a>Vista de dependencias
La vista de dependencias ayuda a visualizar cómo está configurado un recurso. Actualmente, la vista de dependencias está disponible para Azure Application Gateway, Azure Virtual WAN y Azure Load Balancer. Por ejemplo, para Application Gateway, puede acceder a la vista de dependencias si selecciona el nombre del recurso de Application Gateway en la vista de cuadrícula de métricas. Puede hacer lo mismo para Virtual WAN y Load Balancer.

![Captura de pantalla que muestra la vista de Application Gateway en Azure Monitor Network Insights.](media/network-insights-overview/application-gateway.png)

La vista de dependencias de Application Gateway ofrece una vista simplificada de cómo se conectan las direcciones IP de front-end a los agentes de escucha, las reglas y el grupo de back-end. Las líneas de conexión están codificadas por colores y ofrecen detalles adicionales según el estado del grupo de back-end. La vista también ofrece información detallada de las métricas de Application Gateway y de las métricas de todos los grupos de back-end relacionados, como las instancias de VM y de conjunto de escalado de máquinas virtuales.

[![Captura de pantalla que muestra la vista de dependencias de Azure Monitor Network Insights](media/network-insights-overview/dependency-view.png)](media/network-insights-overview/dependency-view.png#lightbox).

El gráfico de dependencias facilita la navegación a las opciones de configuración. Haga clic con el botón derecho en un grupo de back-end para acceder a otra información. Por ejemplo, si el grupo de back-end es una máquina virtual, puede acceder directamente a VM Insights y a la solución de problemas de conexión de Azure Network Watcher para identificar problemas de conectividad:

![Captura de pantalla que muestra el menú de la vista de dependencias en Azure Monitor Network Insights.](media/network-insights-overview/dependency-view-menu.png)

La barra de búsqueda y filtrado de la vista de dependencias ofrece una forma sencilla de buscar en el gráfico. Por ejemplo, si busca **AppGWTestRule** en el ejemplo anterior, la vista se reducirá verticalmente a todos los nodos conectados a través de AppGWTestRule:

![Captura de pantalla que muestra un ejemplo de búsqueda en Azure Monitor Network Insights.](media/network-insights-overview/search-example.png)

Varios filtros lo ayudan a reducir verticalmente a una ruta de acceso y un estado específicos. Por ejemplo, seleccione solo **Incorrecto** en la lista **Estado de mantenimiento** para mostrar todos los bordes con estado incorrecto.

Seleccione **Ver métricas detalladas** para abrir un libro configurado previamente que proporciona métricas detalladas para la puerta de enlace de aplicación, todos los recursos del grupo de back-end y las direcciones IP de front-end. 

## <a name="connectivity"></a><a name="connectivity"></a>Conectividad

La pestaña **Conectividad** proporciona una forma sencilla de visualizar todas las pruebas configuradas mediante [Connection Monitor](../../network-watcher/connection-monitor-overview.md) y Connection Monitor (clásico) para el conjunto de suscripciones seleccionado.

![Captura de pantalla que muestra la pestaña Conectividad de Azure Monitor Network Insights.](media/network-insights-overview/azure-monitor-for-networks-connectivity-tab.png)

Las pruebas se agrupan por iconos de **Origen** y **Destino** y muestran el estado de disponibilidad de cada una. La configuración accesible proporciona un acceso sencillo para configurar los criterios de disponibilidad en función de las comprobaciones con errores (%) y RTT (ms). Una vez que se establecen los valores, el estado de cada prueba se actualiza en función de los criterios de selección.

[![Captura de pantalla que muestra las pruebas de conectividad en Azure Monitor Network Insights](media/network-insights-overview/azure-monitor-for-networks-connectivity-tests.png)](media/network-insights-overview/azure-monitor-for-networks-connectivity-tests.png#lightbox).

Puede seleccionar cualquier icono de origen o de destino para abrir una vista de métrica:

[![Captura de pantalla que muestra las métricas de conectividad de Azure Monitor Network Insights](media/network-insights-overview/azure-monitor-for-networks-connectivity-metrics.png)](media/network-insights-overview/azure-monitor-for-networks-connectivity-metrics.png#lightbox).


Puede seleccionar cualquier elemento en la vista de cuadrícula. Seleccione el icono de la columna **Disponibilidad** para ir a la página del portal de Connection Monitor y ver la topología de salto a salto y la conectividad que afectan a los problemas identificados. Seleccione el valor en la columna **Alerta** para ir a las alertas. Seleccione los gráficos en las columnas **Porcentaje de comprobaciones con error** y **Tiempo de ida y vuelta (ms)** para ir a la página de métricas correspondientes al monitor de conexión seleccionado.

El cuadro  **Alerta** que se encuentra al lado derecho de la página ofrece una vista de todas las alertas generadas para las pruebas de conectividad configuradas en todas las suscripciones. Seleccione el número de alertas para ir a una página de alertas detalladas.

## <a name="traffic"></a><a name="traffic"></a>Tráfico
La pestaña **Tráfico** ofrece acceso a todos los grupos de seguridad de red configurados para [Registros de flujo de NSG](../../network-watcher/network-watcher-nsg-flow-logging-overview.md) y [Análisis de tráfico](../../network-watcher/traffic-analytics.md) para el conjunto seleccionado de suscripciones, agrupados por ubicación. La funcionalidad de búsqueda proporcionada en esta pestaña permite identificar los grupos de seguridad de red configurados para la dirección IP buscada. Puede buscar cualquier dirección IP en su entorno. La vista regional en mosaico mostrará todos los NSG junto con los registros de flujo de NSG y el estado de configuración de Análisis de tráfico.

[![Captura de pantalla que muestra la pestaña Tráfico de Azure Monitor Network Insights](media/network-insights-overview/azure-monitor-for-networks-traffic-view.png)](media/network-insights-overview/azure-monitor-for-networks-traffic-view.png#lightbox).

Si selecciona algún icono de región, aparece una vista de cuadrícula. La cuadrícula proporciona registros de flujo de NSG y Análisis de tráfico en una vista que es fácil de leer y configurar:  

[![Captura de pantalla que muestra la vista de la región de tráfico de Azure Monitor Network Insights](media/network-insights-overview/azure-monitor-for-networks-traffic-region-view.png)](media/network-insights-overview/azure-monitor-for-networks-traffic-region-view.png#lightbox).

Puede seleccionar cualquier elemento en la vista de cuadrícula. Seleccione el icono en la columna **Flowlog Configuration Status** (Estado de configuración de registro de flujo) para editar la configuración del registro de flujo de NSG y de Análisis de tráfico. Seleccione el valor en la columna **Alerta** para ir a las alertas de tráfico configuradas para el NSG seleccionado. También puede ir a la vista Análisis de tráfico si selecciona el **área de trabajo de Análisis de tráfico**.  

El cuadro  **Alerta** que se encuentra al lado derecho de la página ofrece una vista de todas las alertas basadas en el área de trabajo de Análisis de tráfico en todas las suscripciones. Seleccione el número de alertas para ir a una página de alertas detalladas.

## <a name="diagnostic-toolkit"></a><a name="diagnostictoolkit"></a> Kit de herramientas de diagnóstico
El kit de herramientas de diagnóstico proporciona acceso a todas las características de diagnóstico disponibles para solucionar problemas de la red. Puede usar esta lista desplegable para acceder a características como la [captura de paquetes](../../network-watcher/network-watcher-packet-capture-overview.md), la [solución de problemas de VPN](../../network-watcher/network-watcher-troubleshoot-overview.md), la [solución de problemas de conexión](../../network-watcher/network-watcher-connectivity-overview.md), el [próximo salto](../../network-watcher/network-watcher-next-hop-overview.md) y la [comprobación del flujo de IP](../../network-watcher/network-watcher-ip-flow-verify-overview.md):

![Captura de pantalla que muestra la pestaña Kit de herramientas de diagnóstico.](media/network-insights-overview/azure-monitor-for-networks-diagnostic-toolkit.png)

## <a name="onboarded-resources"></a>Recursos incorporados 

Los recursos incorporados tienen libros integrados y vistas de dependencia. Actualmente, los recursos incorporados son Virtual WAN, Application Gateway, Load Balancer y ExpressRoute.

## <a name="troubleshooting"></a>Solución de problemas 
Para instrucciones generales sobre la solución de problemas, consulte el [artículo dedicado de solución de problemas](troubleshoot-workbooks.md) de conclusiones basadas en libros.
Esta sección lo ayudará a diagnosticar y solucionar algunos problemas comunes que puede encontrar al usar Azure Monitor Network Insights. 

### <a name="how-do-i-resolve-performance-problems-or-failures"></a>¿Cómo resuelvo problemas o errores de rendimiento?

Para información sobre cómo solucionar cualquier problema relacionado con la red que identifique con Azure Monitor Network Insights, consulte la documentación para la solución de problemas del recurso que no funciona correctamente. 

Estos son algunos vínculos a los artículos de solución de problemas de los servicios de uso frecuente. Para más artículos de solución de problemas sobre estos servicios, consulte los otros artículos que aparecen en la sección de solución de problemas de la tabla de contenido del servicio.
* [Azure Virtual Network](../../virtual-network/virtual-network-troubleshoot-peering-issues.md)
* [Introducción a Puerta de enlace de aplicaciones](../../application-gateway/create-gateway-internal-load-balancer-app-service-environment.md)
* [Azure VPN Gateway](../../vpn-gateway/vpn-gateway-troubleshoot.md)
* [Información técnica de ExpressRoute](../../expressroute/expressroute-troubleshooting-expressroute-overview.md) 
* [Equilibrador de carga de Azure](../../load-balancer/load-balancer-troubleshoot.md) 

### <a name="why-dont-i-see-the-resources-for-all-the-subscriptions-ive-selected"></a>¿Por qué no veo los recursos de todas las suscripciones que he seleccionado?

Azure Monitor Network Insights solo puede mostrar los recursos de cinco suscripciones a la vez. 

### <a name="how-do-i-make-changes-or-add-visualizations-to-azure-monitor-network-insights"></a>¿Cómo hago cambios o agrego visualizaciones a Azure Monitor Network Insights?

Si quiere hacer cambios, seleccione **Modo de edición** para modificar el libro. Después puede guardar los cambios como un libro nuevo que está vinculado a una suscripción y a un grupo de recursos designados.

### <a name="whats-the-time-grain-after-i-pin-any-part-of-the-workbooks"></a>¿Cuál es el intervalo de agregación después de anclar cualquier parte de los libros?

En Azure Monitor Network Insights se usa el intervalo de agregación **Automático**, por lo que se basa en el intervalo de tiempo seleccionado.

### <a name="whats-the-time-range-when-any-part-of-a-workbook-is-pinned"></a>¿Cuál es el intervalo de tiempo cuando se ancla cualquier parte de un libro?

El intervalo de tiempo depende de la configuración del panel.

### <a name="what-if-i-want-to-see-other-data-or-make-my-own-visualizations-how-can-i-make-changes-to-azure-monitor-network-insights"></a>¿Qué ocurre si deseo ver otros datos o crear mis propias visualizaciones? ¿Cómo puedo realizar cambios en Azure Monitor Network Insights?

Puede evitar el libro que ve en cualquier vista de métrica detallada o panel lateral mediante el modo de edición. Después puede guardar los cambios como un libro nuevo.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre la supervisión de red: [¿Qué es Azure Network Watcher?](../../network-watcher/network-watcher-monitoring-overview.md)
- Conozca los escenarios para los que están concebidos los libros, aprenda a crear informes y a personalizar los ya existentes, y mucho más: [Crear informes interactivos con libros de Azure Monitor](../visualize/workbooks-overview.md)
