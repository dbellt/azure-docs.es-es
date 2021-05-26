---
title: Visualizar datos de Azure Monitor | Microsoft Docs
description: Proporciona un resumen de los métodos disponibles para visualizar los datos de métricas y de registro almacenados en Azure Monitor.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 01/25/2021
ms.openlocfilehash: 1da3f75fb95488d45a3f29284bdc8c7424b6bc9a
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110459111"
---
# <a name="visualizing-data-from-azure-monitor"></a>Visualizar datos de Azure Monitor
En este artículo se proporciona un resumen de los métodos disponibles para visualizar los datos de métricas y de registro almacenados en Azure Monitor.

Visualizaciones como gráficos y diagramas pueden ayudarlo a analizar los datos de supervisión para explorar en profundidad los problemas e identificar patrones. Según la herramienta que utilice, es posible que también tenga la opción de compartir visualizaciones con otros usuarios dentro y fuera de su organización.

## <a name="workbooks"></a>Workbooks
Los [libros](./visualize/workbooks-overview.md) son documentos interactivos que proporcionan información detallada sobre los datos, la investigación y colaboración en el equipo. Algunos ejemplos de libros útiles son las guías de resolución de problemas y los análisis posteriores a incidentes.

![Diagrama que muestra capturas de pantalla de varias páginas de un libro, entre las que se incluyen Análisis de vistas de página, Uso y Tiempo invertido en la página.](media/visualizations/workbook.png)

### <a name="advantages"></a>Ventajas
- Es compatible con las métricas y los registros.
- Admite parámetros que habilitan los informes interactivos en que, al seleccionar un elemento en una tabla, se actualizarán dinámicamente los gráficos y las visualizaciones asociados.
- Flujo similar al de los documentos.
- Opción de libros personales o compartidos.
- Experiencia de creación sencilla y colaborativa.
- Las plantillas admiten la galería de plantillas públicas basadas en GitHub.


## <a name="azure-dashboards"></a>Paneles de Azure
Los [paneles de Azure](../azure-portal/azure-portal-dashboards.md) son la tecnología de panel principal de Azure. Son especialmente útiles para proporcionar una hoja de cristal sobre su infraestructura de Azure y servicios, lo que permite identificar rápidamente problemas importantes.

![Captura de pantalla que muestra un ejemplo de un panel de Azure con información personalizable.](media/visualizations/dashboard.png)

Este es un tutorial en vídeo sobre la creación de paneles.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4AslH]

### <a name="advantages"></a>Ventajas
- Integración profunda en Azure. Las visualizaciones se pueden anclar a paneles desde varias páginas de Azure, que incluyen el [Explorador de métricas](essentials/metrics-charts.md), [Log Analytics](logs/log-analytics-overview.md) y [Application Insights](app/app-insights-overview.md).
- Es compatible con las métricas y los registros.
- Puede combinar datos de varios orígenes, incluidos los resultados del [Explorador de métricas](essentials/metrics-charts.md), las[ consultas del registro](logs/log-query-overview.md) y los [mapas](app/app-map.md), y la disponibilidad de [Application Insights](app/app-insights-overview.md).
- Opción de paneles personales o compartidos. Esta actualización también se integra con los [controles de acceso basado en roles (Azure RBAC) de Azure](../role-based-access-control/overview.md).
- Actualización automática. Las métricas se actualizan según el intervalo de tiempo con un mínimo de cinco minutos. Registra la actualización cada hora, con una opción de actualización manual a petición mediante un clic en el icono "Actualizar" en una visualización determinada o mediante la actualización del panel completo.
- Paneles de métricas parametrizadas con marca de tiempo y parámetros personalizados.
- Opciones de diseño flexibles.
- Modo de pantalla completa.


### <a name="limitations"></a>Limitaciones
- Control limitado sobre las visualizaciones de registros sin compatibilidad con tablas de datos. El número total de series de datos se limita a 50 con otras series de datos agrupadas en _otro_ depósito.
- Los gráficos de registro no son compatibles con los parámetros personalizados.
- Los gráficos de registro se limitan a los últimos 30 días.
- Los gráficos de registro solo se pueden anclar a los paneles compartidos.
- No hay interactividad con los datos del panel.
- Exploración en profundidad contextual limitada.


## <a name="power-bi"></a>Power BI
[Power BI](https://powerbi.microsoft.com/documentation/powerbi-service-get-started/) es particularmente útil para crear informes y paneles centrados en los negocios, así como informes de análisis de tendencias de KPI a largo plazo. Puede [importar los resultados de una consulta de registro](visualize/powerbi.md) en un conjunto de datos de Power BI para poder beneficiarse de sus características, como la combinación de datos de diferentes orígenes y el uso compartido de informes en la web y los dispositivos móviles.

![Power BI](media/visualizations/power-bi.png)

### <a name="advantages"></a>Ventajas
- Visualizaciones enriquecidas.
- Amplia interactividad, incluidas las funciones de acercamiento y filtrado cruzado.
- Fácil de compartir en toda la organización.
- Integración con otros datos desde varios orígenes de datos.
- Mejor rendimiento con los resultados almacenados en caché en un cubo.


### <a name="limitations"></a>Limitaciones
- Admiten registros, pero no las métricas.
- No hay integración de Azure. No puede administrar los paneles y los modelos a través de Azure Resource Manager.
- Los resultados de las consultas deben importarse en el modelo de Power BI para que se configuren. Limitación de tamaño de los resultados y de actualización.
- La actualización de datos se limita a ocho veces al día.


## <a name="grafana"></a>Grafana
[Grafana](https://grafana.com/) es una plataforma abierta que sobresale en paneles operativos. Resulta especialmente útil para detectar, aislar y priorizar los incidentes operativos. Puede agregar el [complemento de origen de datos de Azure Monitor en Grafana](visualize/grafana-plugin.md) a su suscripción de Azure para visualizar los datos de métricas de Azure.

![Captura de pantalla que muestra las visualizaciones de Grafana.](media/visualizations/grafana.png)

### <a name="advantages"></a>Ventajas
- Visualizaciones enriquecidas.
- Ecosistema enriquecido de orígenes de datos.
- Interactividad de datos que incluye la función de acercamiento.
- Es compatible con parámetros.

### <a name="limitations"></a>Limitaciones
- No hay integración de Azure. No puede administrar los paneles y los modelos a través de Azure Resource Manager.
- Costo para admitir una infraestructura Grafana adicional o costo adicional para Grafana Cloud.


## <a name="build-your-own-custom-application"></a>Compilar su propia aplicación personalizada
Puede tener acceso a datos de métricas y de registro en Azure Monitor mediante la API con cualquier cliente REST, lo que le permite crear sus propios sitios web y aplicaciones personalizados.

### <a name="advantages"></a>Ventajas
- Flexibilidad completa en cuanto a interfaz de usuario, visualización, interactividad y características.
- Combine las métricas y los datos de registro con otros orígenes de datos.

### <a name="disadvantages"></a>Inconvenientes
- Se requiere un trabajo de ingeniería importante.


## <a name="azure-monitor-views"></a>Vistas de Azure Monitor

> [!IMPORTANT]
> Las vistas están en proceso de quedar en desuso. Consulte la [Guía de transición del diseñador de vistas de Azure Monitor a libros](visualize/view-designer-conversion-overview.md) para obtener instrucciones sobre cómo convertir vistas en libros.

Las [vistas de Azure Monitor](visualize/view-designer.md) le permiten crear visualizaciones personalizadas con datos de registro. Las [soluciones de supervisión](insights/solutions.md) las utilizan para presentar los datos recopilados.


![Captura de pantalla que muestra un icono de Solución de supervisión de contenedores y la vista detallada de Azure Monitor que se abre al seleccionarlo.](media/visualizations/view.png)

### <a name="advantages"></a>Ventajas
- Visualizaciones enriquecidas para datos de registro.
- Exporte e importe las vistas para transferirlas a otros grupos de recursos y suscripciones.
- Se integran en el modelo de administración de Azure Monitor con áreas de trabajo y soluciones de supervisión.
- [Filtran](visualize/view-designer-filters.md) los parámetros personalizados.
- Interactivas, admiten varios niveles de obtención de detalles (vista que explora otra vista)

### <a name="limitations"></a>Limitaciones
- Admiten registros, pero no las métricas.
- No hay vistas personales. Disponibles para todos los usuarios con acceso al área de trabajo.
- No hay actualizaciones automáticas.
- Opciones de diseño limitadas.
- No se admiten las consultas a través de varias áreas de trabajo o aplicaciones de Application Insights.
- El tamaño de respuesta y el tiempo de ejecución de las consultas se limitan a 8 MB y 110 segundos, respectivamente.

## <a name="next-steps"></a>Pasos siguientes
- Obtenga información sobre los [datos que Azure Monitor recopila](data-platform.md).
- Obtenga información sobre los [paneles de Azure](../azure-portal/azure-portal-dashboards.md).
- Más información acerca de [Explorador de métricas](essentials/metrics-getting-started.md).
- Obtenga información sobre [Workbooks](./visualize/workbooks-overview.md).
- Obtenga información sobre cómo [importar datos de registro en Power BI](./visualize/powerbi.md).
- Obtenga información sobre el [complemento de origen de datos de Azure Monitor en Grafana](./visualize/grafana-plugin.md).
- Obtenga información sobre las [vistas de Azure Monitor](visualize/view-designer.md).

