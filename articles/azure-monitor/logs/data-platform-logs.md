---
title: Registros de Azure Monitor
description: Describe los registros de Azure Monitor que se usan para realizar análisis avanzados de los datos de supervisión.
documentationcenter: ''
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.date: 10/22/2020
ms.author: bwren
ms.openlocfilehash: 9794c5f048b8795652e4b31e0134b36a77715abe
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107873386"
---
# <a name="azure-monitor-logs-overview"></a>Introducción a los registros de Azure Monitor
Los registros de Azure Monitor son una característica de Azure Monitor que recopila y organiza los datos de registro y rendimiento de los [recursos supervisados](../monitor-reference.md). Los datos de orígenes diferentes, como los [registros de la plataforma](../essentials/platform-logs-overview.md) de los servicios de Azure, los datos de registro y de rendimiento de los [agentes de máquinas virtuales](../agents/agents-overview.md), y los datos de uso y rendimiento de las[aplicaciones](../app/app-insights-overview.md) se pueden consolidar en una sola área de trabajo para que se puedan analizar juntos con un lenguaje de consulta sofisticado que sea capaz de analizar rápidamente millones de registros. Puede realizar una consulta simple que solo recupere un conjunto específico de registros o realizar un análisis de datos sofisticado para identificar patrones críticos en los datos de supervisión. Trabaje con consultas de registros y sus resultados de forma interactiva mediante Log Analytics, úselas en una regla de alertas para recibir notificaciones proactivas de los problemas o visualice los resultados en un libro o panel.

> [!NOTE]
> Los registros de Azure Monitor son la mitad de la plataforma de datos de Azure Monitor. La otra mitad son las [métricas de Azure Monitor](../essentials/data-platform-metrics.md), que almacenan datos numéricos en una base de datos de serie temporal. Esto hace que estos datos sean más ligeros que los datos de los registros de Azure Monitor y capaces de admitir escenarios casi en tiempo real, lo que hace que sean especialmente útiles para las alertas y la detección rápida de problemas. Sin embargo, las métricas solo pueden almacenar datos numéricos en una estructura determinada, mientras que los registros pueden almacenar una variedad de tipos de datos distintos, cada uno con su propia estructura. También puede hacer análisis complejos en los datos de los registros mediante el uso de consultas de registros que no se pueden usar para el análisis de los datos de las métricas.


## <a name="what-can-you-do-with-azure-monitor-logs"></a>¿Qué puede hacer con los registros de Azure Monitor?
En la tabla siguiente se enumeran algunas de las distintas formas en que se pueden usar los registros en Azure Monitor:

|  | Descripción |
|:---|:---|
| **Análisis** | Use [Log Analytics](./log-analytics-tutorial.md) en Azure Portal para escribir [consultas de registros](./log-query-overview.md) y analizar los datos de registro de forma interactiva con un motor de análisis eficaz. |
| **Alerta** | Configurar un [regla de alerta de registro](../alerts/alerts-log.md) que envíe una notificación o realice [una acción automatizada](../alerts/action-groups.md) cuando los resultados de la consulta coincidan con un resultado concreto. |
| **Visualizar** | Ancle los resultados representados como tablas o gráficos a un [panel de Azure](../../azure-portal/azure-portal-dashboards.md).<br>Cree un [libro](../visualize/workbooks-overview.md) para combinar con varios conjuntos de datos en un informe interactivo. <br>Exportar los resultados de una consulta a [Power BI](../visualize/powerbi.md) para usar diferentes visualizaciones y compartirlos con usuarios fuera de Azure.<br>Exporte los resultados de una consulta a [Grafana](../visualize/grafana-plugin.md) para aprovechar sus paneles y combinar con otros orígenes de datos.|
| **Insights** | Compatibilidad con [Insights](../monitor-reference.md#insights-and-core-solutions), que proporciona una experiencia de supervisión personalizada para determinadas aplicaciones y servicios.  |
| **Recuperar** | Obtenga acceso a los resultados de la consulta de registro desde una línea de comandos mediante la [CLI de Azure](/cli/azure/monitor/log-analytics).<br>Obtenga acceso a los resultados de la consulta de registro desde una línea de comandos mediante [cmdlets de PowerShell](/powershell/module/az.operationalinsights).<br>Obtenga acceso a los resultados de la consulta de registro de una aplicación personalizada con la [API REST](https://dev.loganalytics.io/). |
| **Exportarar** | Configure la [exportación automatizada de datos de registro](./logs-data-export.md) a una cuenta de Azure Storage o a Azure Event Hubs.<br>Cree un flujo de trabajo para recuperar datos de registro y cópielo en una ubicación externa mediante [Logic Apps](./logicapp-flow-connector.md). |

![Información general de los registros](media/data-platform-logs/logs-overview.png)


## <a name="data-collection"></a>datos, recopilación
Una vez que cree un área de trabajo de Log Analytics, debe configurar orígenes diferentes para que envíen sus datos. No se recopilan datos automáticamente. Esta configuración será diferente en función del origen de datos. Por ejemplo, [cree una configuración de diagnóstico](../essentials/diagnostic-settings.md) para enviar los registros de recursos desde los recursos de Azure al área de trabajo. [Habilite VM Insights](../vm/vminsights-enable-overview.md) para recopilar datos de máquinas virtuales. Configure [orígenes de datos en el área de trabajo](../agents/data-sources.md) para recopilar eventos adicionales y datos de rendimiento.

- Consulte [¿Qué supervisa Azure Monitor?](../monitor-reference.md) para obtener una lista completa de los orígenes de datos que puede configurar para enviar datos a un área de trabajo de Log Analytics.


## <a name="log-analytics-workspaces"></a>Áreas de trabajo de Log Analytics
Los datos recopilados por los registros de Azure Monitor se almacenan en una o varias [áreas de trabajo de Log Analytics](./design-logs-deployment.md). El área de trabajo define la ubicación geográfica de los datos, derechos de acceso que definen qué usuarios pueden acceder a los datos y opciones de configuración, como el plan de tarifa y la retención de datos.  

Debe crear al menos un área de trabajo para usar los registros de Azure Monitor. Una sola área de trabajo puede ser suficiente para todos los datos de supervisión o puede optar por crear varias áreas de trabajo en función de sus requisitos. Por ejemplo, podría tener un área de trabajo para los datos de producción y otra para las pruebas. 

- Consulte [Creación de un área de trabajo de Log Analytics en Azure Portal](./quick-create-workspace.md) para crear una nueva área de trabajo.
- Consulte [Diseño de la implementación de registros de Azure Monitor](design-logs-deployment.md) para información sobre la creación de varias áreas de trabajo.

## <a name="data-structure"></a>Estructura de los datos
Las consultas de registro recuperan sus datos de un área de trabajo Log Analytics. Cada área de trabajo contiene varias tablas que están organizadas en columnas independientes con varias filas de datos. Cada tabla se define mediante un conjunto único de columnas compartidas por las filas de datos que proporciona el origen de datos. 

[![Estructura de los registros de Azure Monitor](media/data-platform-logs/logs-structure.png)](media/data-platform-logs/logs-structure.png#lightbox)


Los datos de registros de Application Insights también se almacenan en los registros de Azure Monitor, pero de manera distinta según la configuración de la aplicación. Si se trata de una aplicación basada en áreas de trabajo, los datos se almacenan en un área de trabajo de Log Analytics en un conjunto de tablas estándar para contener datos como solicitudes de aplicación, excepciones y vistas de página. Varias aplicaciones pueden usar la misma área de trabajo. Si se trata de una aplicación clásica, los datos no se almacenan en un área de trabajo de Log Analytics. Usa el mismo lenguaje de consulta y se crean y ejecutan consultas con la misma herramienta de Log Analytics en Azure Portal. Sin embargo, los datos de las aplicaciones clásicas se almacenan separados entre sí. Su estructura general es igual que la de las aplicaciones basadas en áreas de trabajo, a pesar de que los nombres de tabla y columna son distintos. Consulte [Cambios en los recursos basados en áreas de trabajo (versión preliminar)](../app/apm-tables.md) para obtener una comparación detallada del esquema de las aplicaciones clásicas y las basadas en áreas de trabajo.


> [!NOTE]
> Todavía se proporciona compatibilidad total con versiones anteriores con las consultas de recursos clásicas, los libros y las alertas basadas en registro de Application Insights dentro de la experiencia de Application Insights. Para consultar o ver en la [nueva estructura o esquema de tabla basados en área de trabajo](../app/apm-tables.md), primero debe ir al área de trabajo de Log Analytics. Durante la versión preliminar, al seleccionar **Registros** en los paneles de Application Insights, se proporciona acceso a la experiencia de consulta clásica de Application Insights. Consulte [Ámbito de la consulta](./scope.md) para más detalles.


[![Estructura de los registros de Azure Monitor para Application Insights](media/data-platform-logs/logs-structure-ai.png)](media/data-platform-logs/logs-structure-ai.png#lightbox)


## <a name="log-queries"></a>Consultas de registros
Los datos se recuperar de un área de trabajo de Log Analytics mediante una consulta de registros, que es una solicitud de solo lectura para procesar datos y devolver resultados. Las consultas de registros se escriben en el [lenguaje de consulta Kusto (KQL)](/azure/data-explorer/kusto/query/), que es el mismo lenguaje de consulta que usa Azure Data Explorer. Puede escribir consultas de registro en Log Analytics para analizar de forma interactiva los resultados, usarlas en reglas de alertas para notificar de forma proactiva los problemas o incluir los resultados en libros o paneles. La información detallada incluye las consultas precompiladas para admitir sus vistas y libros.

- Consulte [Introducción a las consultas de registro en Azure Monitor](./log-query-overview.md) para obtener una lista de dónde se utilizan las consultas de registro y referencias a tutoriales y otra documentación para ayudarle a comenzar.

![Log Analytics](media/data-platform-logs/log-analytics.png)

## <a name="log-analytics"></a>Log Analytics
Use Log Analytics, que es una herramienta de Azure Portal, para editar y ejecutar consultas de registros y analizar los resultados de forma interactiva. Luego puede usar las consultas que cree para admitir otras características en Azure Monitor como los libros y las alertas de consultas de registro. Acceda a Log Analytics desde la opción **Registros** del menú de Azure Monitor o desde la mayoría de los demás servicios de Azure Portal.

- Consulte [Introducción a Log Analytics en Azure Monitor](./log-analytics-overview.md) para obtener una descripción de Log Analytics. 
- Consulte [Tutorial de Log Analytics](./log-analytics-tutorial.md) para recorrer el uso de las características de Log Analytics para crear una consulta de registro simple y analizar los resultados.



## <a name="relationship-to-azure-data-explorer"></a>Relación con Azure Data Explorer
Los registros de Azure Monitor se basa en Azure Data Explorer. Un área de trabajo de Log Analytics es casi equivalente a una base de datos de Azure Data Explorer, las tablas están estructuradas de la misma manera y ambas usan el mismo lenguaje de consulta de Kusto (KQL). La experiencia de usar Log Analytics para trabajar con las consultas de Azure Monitor en Azure Portal es similar a la experiencia de usar la interfaz de usuario web de Azure Data Explorer. Incluso puede [incluir datos de un área de trabajo de Log Analytics en una consulta de Azure Data Explorer](/azure/data-explorer/query-monitor-data). 


## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre las [consultas de registros](./log-query-overview.md) para recuperar y analizar datos provenientes de un área de trabajo de Log Analytics.
- Obtenga más información acerca de las [métricas en Azure Monitor](../essentials/data-platform-metrics.md).
- Obtenga información sobre la [supervisión de datos disponible](../agents/data-sources.md) para diferentes recursos en Azure.