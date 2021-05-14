---
title: Creación de alertas con SQL Insights (versión preliminar)
description: Creación de alertas con SQL Insights en Azure Monitor
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/12/2021
ms.openlocfilehash: bb42f74f6ac8487a93479bdf980c66ef87e8e742
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107726869"
---
# <a name="create-alerts-with-sql-insights-preview"></a>Creación de alertas con SQL Insights (versión preliminar)
SQL Insights incluye un conjunto de plantillas de reglas de alertas que puede usar para crear [reglas de alertas en Azure Monitor](../alert/../alerts/alerts-overview.md) para problemas comunes de SQL. Las reglas de alertas de SQL Insights son reglas de alertas de registro basadas en los datos de rendimiento almacenados en la tabla *InsightsMetrics* de los registros de Azure Monitor.  

> [!NOTE]
> Para crear una alerta para SQL Insights mediante una plantilla de Resource Manager, vea [Ejemplos de plantilla de Resource Manager para SQL Insights](resource-manager-sql-insights.md#create-an-alert-rule-for-sql-insights).


> [!NOTE]
> Si tiene solicitudes de más plantillas de reglas de alertas de SQL Insights, envíe sus comentarios con el vínculo de la parte inferior de esta página o con el vínculo de comentarios de SQL Insights de Azure Portal.

## <a name="enable-alert-rules"></a>Habilitar reglas de alertas 
Realice los pasos siguientes para habilitar las alertas en Azure Monitor desde Azure Portal. Las reglas de alertas que se creen se limitarán a todos los recursos de SQL supervisados bajo el perfil de supervisión seleccionado.  Cuando una regla de alertas se desencadena, lo hace en la base de datos o la instancia de SQL específica.

> [!NOTE]
> También puede crear [reglas de alertas de registro](../alerts/alerts-log.md) personalizadas mediante la ejecución de consultas en los conjuntos de datos de la tabla *InsightsMetrics* y, a continuación, guardar dichas consultas como una regla de alertas. 

Seleccione **SQL (versión preliminar)** en la sección **Información** del menú Azure Monitor en Azure Portal. Haga clic en **Alertas**.

:::image type="content" source="media/sql-insights-alerts/alerts-button.png" alt-text="Botón Alertas":::

Se abre el panel **Alertas** en el lado derecho de la página. De forma predeterminada, se mostrarán las alertas desencadenadas para los recursos de SQL en el perfil de supervisión seleccionado en función de las reglas de alertas que ya ha creado. Seleccione **Plantillas de alerta** y se mostrará la lista de plantillas disponibles que puede usar para crear una regla de alertas.

:::image type="content" source="media/sql-insights-alerts/alert-templates.png" alt-text="Plantillas de alerta":::

En la página **Crear regla de alertas**, revise la configuración predeterminada de la regla y edítela según sea necesario. También puede seleccionar un [grupo de acciones](../alerts/action-groups.md) para crear notificaciones y acciones cuando se desencadene la regla de alertas. Haga clic en **Enable alert rule** (Habilitar regla de alertas) para crear la regla de alertas una vez que haya comprobado todas sus propiedades.


:::image type="content" source="media/sql-insights-alerts/alert-rule.png" alt-text="Página Reglas de alertas":::

Para implementar la regla de alertas inmediatamente, haga clic en **Deploy alert rule** (Implementar regla de alertas). Haga clic en **Ver plantilla** si desea ver la plantilla de regla antes de implementarla realmente.

:::image type="content" source="media/sql-insights-alerts/alert-rule-deploy.png" alt-text="Implementar regla de alertas":::

Si opta por ver las plantillas, seleccione **Implementar** en la página de la plantilla para crear la regla de alertas.

:::image type="content" source="media/sql-insights-alerts/view-template-deploy.png" alt-text="Implementar desde Ver plantilla":::


## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre [alertas en Azure Monitor](../alerts/alerts-overview.md).

