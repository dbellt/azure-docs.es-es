---
title: 'Tutorial: Supervisión de una máquina híbrida con VM Insights de Azure Monitor'
description: Aprenda a recopilar y analizar los datos de una máquina híbrida en Azure Monitor.
ms.topic: tutorial
ms.date: 04/21/2021
ms.openlocfilehash: f59ad448440110e2c5e6dd1fa1b2858d9cf42e91
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107834271"
---
# <a name="tutorial-monitor-a-hybrid-machine-with-vm-insights"></a>Tutorial: Supervisión de una máquina híbrida con VM Insights

[Azure Monitor](../../../azure-monitor/overview.md) puede recopilar datos directamente de las máquinas híbridas en un área de trabajo de Log Analytics para lograr una correlación y un análisis detallados. Normalmente, esto implicaría la instalación del [agente de Log Analytics](../../../azure-monitor/agents/agents-overview.md#log-analytics-agent) en la máquina mediante un script, un método manual o automatizado, según los estándares de administración de la configuración. Los servidores habilitados para Arc han incorporado recientemente compatibilidad para instalar las [extensiones de VM](../manage-vm-extensions.md) del agente de Log Analytics y de Dependency Agent para Windows y Linux, lo que permite a [VM Insights](../../../azure-monitor/vm/vminsights-overview.md) recopilar datos de máquinas virtuales que no son de Azure.

En este tutorial se muestra cómo configurar y recopilar datos de máquinas Linux o Windows mediante la habilitación de VM Insights siguiendo un conjunto simplificado de pasos, lo que optimiza la experiencia y exige un tiempo menor.  

## <a name="prerequisites"></a>Requisitos previos

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

* La funcionalidad de la extensión de VM solo está disponible en la lista de [regiones admitidas](../overview.md#supported-regions).

* Consulte [Sistemas operativos admitidos](../../../azure-monitor/vm/vminsights-enable-overview.md#supported-operating-systems) para asegurarse de que el sistema operativo de los servidores que está habilitando sea compatible con VM Insights.

* Revise los requisitos de firewall para el agente de Log Analytics, que se proporcionan en el artículo sobre [información general del agente de Log Analytics](../../../azure-monitor/agents/log-analytics-agent.md#network-requirements). La instancia de Dependency Agent de asignación de VM Insights no transmite ningún dato y no requiere ningún cambio en firewalls o puertos.

## <a name="sign-in-to-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="enable-vm-insights"></a>Habilitar VM Insights

1. Inicie el servicio Azure Arc en Azure Portal. Para ello, haga clic en **Todos los servicios** y, a continuación, busque y seleccione **Máquinas: Azure Arc**.

    :::image type="content" source="./media/quick-enable-hybrid-vm/search-machines.png" alt-text="Búsqueda de servidores habilitados para Arc en todos los servicios" border="false":::

1. En la página **Máquinas: Azure Arc**, seleccione la máquina conectada que creó en el artículo de [inicio rápido](quick-enable-hybrid-vm.md).

1. En el panel izquierdo, en la sección **Supervisión**, seleccione **Insights** y, luego, **Habilitar**.

    :::image type="content" source="./media/tutorial-enable-vm-insights/insights-option.png" alt-text="Selección de la opción Insights en el menú izquierdo" border="false":::

1. En la página de Azure Monitor **Incorporación a Insights**, se le pedirá que cree un área de trabajo. En este tutorial, no se recomienda que seleccione un área de trabajo de Log Analytics existente si tiene una. Seleccione el valor predeterminado que es un área de trabajo con un nombre único en la misma región que la máquina conectada registrada. Esta área de trabajo se crea y se configura automáticamente.

    :::image type="content" source="./media/tutorial-enable-vm-insights/enable-vm-insights.png" alt-text="Página de habilitación de VM Insights" border="false":::

1. Recibirá mensajes de estado mientras se lleva a cabo la configuración. Este proceso tarda unos minutos mientras se instalan las extensiones en la máquina conectada.

    :::image type="content" source="./media/tutorial-enable-vm-insights/onboard-vminsights-vm-portal-status.png" alt-text="Mensaje de estado de progreso de habilitación de VM Insights" border="false":::

    Cuando haya finalizado, recibirá un mensaje que indica que la máquina se ha incorporado correctamente y que la información se ha implementado sin problemas.

## <a name="view-data-collected"></a>Ver datos recopilados

Una vez completada la implementación y la configuración, seleccione **Insights** y, a continuación, seleccione la pestaña **Rendimiento**. En la pestaña Rendimiento, se muestra un grupo seleccionado de contadores de rendimiento recopilados desde el sistema operativo invitado de la máquina. Desplácese hacia abajo para ver más contadores, y mueva el mouse sobre un gráfico para ver el promedio y los percentiles que se han tomado desde el momento en que se instaló la extensión de máquina virtual de Log Analytics en la máquina.

:::image type="content" source="./media/tutorial-enable-vm-insights/insights-performance-charts.png" alt-text="Gráficos de rendimiento de VM Insights para la máquina seleccionada" border="false":::

Seleccione **Asignar** para abrir la característica de asignaciones que muestra los procesos que se ejecutan en la máquina y sus dependencias. Seleccione **Propiedades** para abrir el panel de propiedades si aún no está abierto.

:::image type="content" source="./media/tutorial-enable-vm-insights/insights-map.png" alt-text="Asignación de VM Insights para la máquina seleccionada" border="false":::

Expanda los procesos de la máquina. Seleccione uno de los procesos para ver sus detalles y resaltar sus dependencias.

Vuelva a seleccionar la máquina y, luego, **Registrar eventos**. Verá una lista de tablas que se almacenan en el área de trabajo de Log Analytics de la máquina. Esta lista será diferente en función de si utiliza una máquina Windows o Linux. Seleccione la tabla **Evento**. La tabla **Evento** incluye todos los eventos del registro de eventos de Windows. Se abre Log Analytics con una consulta simple para recuperar las entradas del registro de eventos recopiladas.

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de Azure Monitor, consulte el siguiente artículo:

> [!div class="nextstepaction"]
> [Introducción a Azure Monitor](../../../azure-monitor/overview.md)