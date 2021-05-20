---
title: Visualización y configuración de alertas para el mantenimiento del circuito de Azure ExpressRoute
description: Aprenda a configurar alertas personalizadas para el mantenimiento del circuito de ExpressRoute mediante la página Service Health de Azure Portal.
services: expressroute
author: mialdrid
ms.service: expressroute
ms.topic: how-to
ms.date: 05/10/2021
ms.author: mialdrid
ms.openlocfilehash: 43e30ddd601640b61c42c5ba5d6bb2aaf41eb73d
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109739069"
---
# <a name="how-to-view-and-configure-alerts-for-azure-expressroute-circuit-maintenance"></a>Visualización y configuración de alertas para el mantenimiento del circuito de Azure ExpressRoute

ExpressRoute usa Azure Service Health para notificarle el mantenimiento planeado y próximo del circuito de ExpressRoute. Con Service Health, puede ver el mantenimiento planeado y anterior en Azure Portal junto con la configuración de alertas y notificaciones que mejor se adapte a sus necesidades. Para más información sobre Azure Service Health, consulte [¿Qué es Azure Service Health?](../service-health/overview.md)

## <a name="view-planned-maintenance"></a>Visualización del mantenimiento planeado

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Vaya a la página Service Health. 

    :::image type="content" source="./media/maintenance-alerts/search-service-health.png" alt-text="Captura de pantalla de la búsqueda de la página Service Health."::: 

1. Seleccione **Mantenimiento planeado** en *Eventos activos* al lado izquierdo de la página. En esta página, puede ver eventos de mantenimiento individuales si filtra según una suscripción de destino, una región de Azure y un servicio de Azure.

    :::image type="content" source="./media/maintenance-alerts/view-maintenance.png" alt-text="Captura de pantalla de la página de mantenimiento planeado de Service Health." lightbox="./media/maintenance-alerts/view-maintenance-expanded.png"::: 

1. Seleccione **ExpressRoute** en el menú desplegable *Servicios* para ver solo el mantenimiento relacionado con ExpressRoute. Luego, seleccione una incidencia de la lista para ver el resumen del evento. Seleccione la pestaña **Issues updates** (Actualizaciones de incidencias) si desea obtener más detalles sobre un mantenimiento en curso.

    :::image type="content" source="./media/maintenance-alerts/summary.png" alt-text="Captura de pantalla de resumen del mantenimiento de ExpressRoute." lightbox="./media/maintenance-alerts/summary-expanded.png"::: 

## <a name="view-past-maintenance"></a>Visualización del mantenimiento anterior

1. Para ver eventos de mantenimiento anterior, seleccione **Historial de estado**, que se encuentra en la sección *Historial* a la izquierda de la página. 

    :::image type="content" source="./media/maintenance-alerts/health-history.png" alt-text="Captura de pantalla de la selección del historial de estado en Service Health." lightbox="./media/maintenance-alerts/health-history-expanded.png"::: 

1. En esta página, puede revisar eventos de mantenimiento individuales si filtra según una suscripción de destino y una región de Azure. Para restringir aún más el ámbito de los eventos del historial de estado, puede seleccionar el tipo de evento de mantenimiento y definir un intervalo de tiempo anterior. Para solicitar un mantenimiento planeado del circuito de ExpressRoute, establezca el tipo de evento de mantenimiento en **Mantenimiento planeado**.

    :::image type="content" source="./media/maintenance-alerts/past-maintenance.png" alt-text="Captura de pantalla del mantenimiento anterior en la página Historial de estado." lightbox="./media/maintenance-alerts/past-maintenance-expanded.png"::: 

## <a name="create-alerts-and-notifications-for-maintenance-events"></a>Creación de alertas y notificaciones para eventos de mantenimiento

1. Azure Service Health admite alertas personalizadas para los eventos de mantenimiento. Si desea configurar una alerta para el mantenimiento del circuito de ExpressRoute, vaya a **Alertas de estado** en la sección *Alertas* en el lado izquierdo de la página. Aquí verá una tabla de las alertas que se configuraron anteriormente.

1.  Para crear una alerta, seleccione **Agregar alerta de Service Health** que se encuentra en la parte superior de la página.

    :::image type="content" source="./media/maintenance-alerts/health-alerts.png" alt-text="Captura de pantalla de la selección de alertas de mantenimiento en Service Health." lightbox="./media/maintenance-alerts/health-alerts-expanded.png"::: 

1. Si desea crear una regla de alertas, seleccione o escriba esta información.

    | Category | Configuración | Value | 
    | --- | -------- | ----- |
    | **Condition** | Suscripción | Seleccione la suscripción de destino. |
    |               | Servicio(s) | *ExpressRoute \ Circuitos ExpressRoute* |
    |               | Regiones | Seleccione una región o deje la opción *Global* para los eventos de mantenimiento de todas las regiones.
    |               | Tipo de evento | Seleccione *Mantenimiento planeado*. |
    | **Acciones** | Nombre del grupo de acciones | El *grupo de acciones* determina el tipo de notificación y define la audiencia a la que se envía la notificación. Si desea ayuda para crear y administrar el grupo de acciones, consulte [Creación y administración de grupos de acciones](../azure-monitor/alerts/action-groups.md) en Azure Portal. |
    | **Detalles de la regla de alertas** | Nombre de la regla de alertas | Escriba un *nombre* para identificar la regla de alertas. |
    |                        | Descripción | Proporcione una descripción de lo que hace esta regla de alertas. | 
    |                        | Guardar regla de alertas en el grupo de recursos | Seleccione un *grupo de recursos* en el que se va a crear esta regla de alertas. |
    |                        | Habilitación de una regla de alertas tras la creación | Active esta casilla para habilitar esta regla de alertas una vez creada. |

1. Seleccione **Crear regla de alertas** para guardar la configuración.

    :::image type="content" source="./media/maintenance-alerts/create-alert-rule.png" alt-text="Captura de pantalla de la página Crear regla de alertas."::: 

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre [Azure ExpressRoute](expressroute-introduction.md), [Network insights](../azure-monitor/insights/network-insights-overview.md)y [Network Watcher](../network-watcher/network-watcher-monitoring-overview.md)
* [Personalización de las métricas](expressroute-monitoring-metrics-alerts.md) y creación de un [Connection Monitor](../network-watcher/connection-monitor-overview.md)

