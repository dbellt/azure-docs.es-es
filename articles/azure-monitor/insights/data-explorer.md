---
title: Azure Data Explorer Insights (versión preliminar de ADX Insights)| Microsoft Docs
description: En este artículo se describe Azure Data Explorer Insights (ADX Insights)
services: azure-monitor
ms.topic: conceptual
ms.date: 01/05/2021
author: lgayhardt
ms.author: lagayhar
ms.openlocfilehash: 274d907c4fd8d09e444b938447365a4df64af3e4
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112061639"
---
# <a name="azure-data-explorer-insights-preview"></a>Azure Data Explorer Insights (versión preliminar)

Azure Data Explorer Insights (versión preliminar) proporciona una supervisión completa de los clústeres, ya que ofrece una vista unificada de su rendimiento, operaciones, uso y errores.
Este artículo le ayuda a saber cómo incorporar y usar Azure Data Explorer Insights (versión preliminar).

## <a name="introduction-to-azure-data-explorer-insights-preview"></a>Introducción a Azure Data Explorer Insights (versión preliminar)

Antes de saltar a la experiencia, debe entender cómo se presenta y se visualiza la información.
-    **Perspectiva a gran escala**, que muestra una instantánea de las métricas principales de los clústeres, con el fin de realizar un seguimiento sencillo del rendimiento de las consultas, la ingesta y las operaciones de exportación.
-   **Análisis en profundidad** de un clúster determinado de Azure Data Explorer para ayudar a realizar análisis detallados.
-    **Personalizable**, que le permite cambiar las métricas que quiere ver y modificar, o establecer umbrales en consonancia con sus límites y guardar sus propios libros personalizados. Los gráficos del libro se pueden anclar a los paneles de Azure.

## <a name="view-from-azure-monitor-at-scale-perspective"></a>Vista desde Azure Monitor (perspectiva a gran escala)

En Azure Monitor, puede ver las métricas principales de rendimiento del clúster, como las métricas de consultas, ingesta y operaciones de exportación de varios clústeres de su suscripción, y ayudar a identificar los problemas de rendimiento.

Para ver el rendimiento de los clústeres en todas las suscripciones, realice los pasos siguientes:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

2. Seleccione **Supervisión** en el panel izquierdo de Azure Portal y, en la sección, Insights Hub (Centro de conclusiones), seleccione **Azure Data Explorer Clusters (preview)** (Clústeres de Azure Data Explorer [versión preliminar]).

![Captura de pantalla de la experiencia de información general con varios gráficos](./media/data-explorer/insights-hub.png)

### <a name="overview-tab"></a>Pestaña Información general

En la pestaña **Información general** de la suscripción seleccionada, la tabla muestra las métricas interactivas de los clústeres de Azure Data Explorer agrupadas dentro de la suscripción. Puede filtrar los resultados en función de las opciones que seleccione en las siguientes listas desplegables:

* Suscripciones: solo se enumeran las suscripciones que tienen clústeres de Azure Data Explorer.

* Clústeres de Azure Data Explorer: de forma predeterminada, solo se preseleccionan hasta cinco clústeres. Si selecciona todos los clústeres o varios de ellos en el selector de ámbitos, se devolverán como máximo 200 clústeres.

* Intervalo de tiempo: de forma predeterminada, muestra las últimas veinticuatro horas de información en función de las selecciones correspondientes realizadas.

El icono de contador, que se encuentra bajo la lista desplegable, acumula el número total de clústeres de Azure Data Explorer de las suscripciones seleccionadas y refleja cuántas están seleccionadas. Hay códigos de colores condicionales para las columnas: Mantener conexión, CPU, Ingestion Utilization (Uso de ingesta) y Cache Utilization (Uso de caché). Las celdas con codificación naranja tienen valores que no son sostenibles para el clúster. 

Para comprender mejor lo que representa cada una de estas métricas, se recomienda leer la documentación sobre las [métricas de Azure Data Explorer](/azure/data-explorer/using-metrics#cluster-metrics).

### <a name="query-performance-tab"></a>Pestaña Rendimiento de las consultas

Esta pestaña muestra la duración de las consultas, el número total de consultas simultáneas y el número total de consultas limitadas.

![Captura de pantalla de la pestaña Rendimiento de las consultas](./media/data-explorer/query-performance.png)

### <a name="ingestion-performance-tab"></a>Pestaña Ingestion Performance (Rendimiento de ingesta)

En esta pestaña se muestra la latencia de ingesta, los resultados de ingesta correctos, el volumen de ingesta y los eventos procesados para los centros de IoT o de eventos.

[![Captura de pantalla de la pestaña Ingestion Performance (Rendimiento de ingesta)](./media/data-explorer/ingestion-performance.png)](./media/data-explorer/ingestion-performance.png#lightbox)

### <a name="streaming-ingest-performance-tab"></a>Pestaña Streaming Ingest Performance (Rendimiento de ingesta de streaming)

Esta pestaña proporciona información sobre la velocidad de datos media, la duración media y la tasa de solicitudes.

### <a name="export-performance-tab"></a>Pestaña Export Performance (Rendimiento de exportación)

Esta pestaña proporciona información sobre los registros exportados, la demora, el recuento pendiente y el porcentaje de uso de las operaciones de exportación continua.

## <a name="view-from-an-azure-data-explorer-cluster-resource-drill-down-analysis"></a>Vista de un recurso de clúster de Azure Data Explorer (análisis en profundidad)

Para acceder a Azure Data Explorer Insights directamente desde un clúster de Azure Data Explorer:

1. En Azure Portal, seleccione **Clústeres de Azure Data Explorer**.

2. En la lista, elija un clúster de Azure Data Explorer. En la sección de supervisión, seleccione **Conclusiones (versión preliminar)** .

También se puede acceder a estas vistas seleccionando el nombre del recurso de un clúster de Azure Data Explorer desde la vista de conclusiones de Azure Monitor.

Azure Data Explorer Insights combina los registros y las métricas para proporcionar una solución de supervisión global. La inclusión de visualizaciones basadas en registros requiere que los usuarios [habiliten el registro de diagnóstico de su clúster de Azure Data Explorer y lo envíen a un área de trabajo de Log Analytics.](/azure/data-explorer/using-diagnostic-logs?tabs=commands-and-queries#enable-diagnostic-logs). Los registros de diagnóstico que se deben habilitar son: **Command**, **Query**, **TableDetails** y **TableUsageStatistics**.

![Captura de pantalla del botón azul que muestra el texto "Enable Logs for Monitoring" (Habilitar la supervisión de registros)](./media/data-explorer/enable-logs.png)


 La pestaña **Información general** muestra:

- Iconos de métricas que resaltan la disponibilidad y el estado general del clúster para evaluar rápidamente su estado.

- Un resumen del estado activo de las [recomendaciones de Advisor](/azure/data-explorer/azure-advisor) y del [estado de los recursos](/azure/data-explorer/monitor-with-resource-health).

- Gráficos que muestran los consumidores principales de CPU y memoria y el número de usuarios únicos a lo largo del tiempo.


[![Captura de pantalla de la vista de un recurso de clúster de Azure Data Explorer](./media/data-explorer/overview.png)](./media/data-explorer/overview.png#lightbox)

La pestaña **Key Metrics** (Métricas clave) muestra una vista unificada de algunas de las métricas del clúster, agrupadas por: métricas generales, relacionadas con consultas, relacionadas con la ingesta y relacionadas con la ingesta de streaming.

[![Captura de pantalla de la vista de errores](./media/data-explorer/key-metrics.png)](./media/data-explorer/key-metrics.png#lightbox)

La pestaña **Uso** permite a los usuarios profundizar en el rendimiento de los comandos y las consultas del clúster. En esta página, puede:
 
 - Ver qué grupos de cargas de trabajo, usuarios y aplicaciones envían la mayoría de las consultas o consumen la mayor parte de la CPU y la memoria (para que pueda saber qué cargas de trabajo envían las consultas más pesadas que va a procesar el clúster).
 - Identificar los grupos de cargas de trabajo, usuarios y aplicaciones principales por consultas con error.
 - Identificar los cambios recientes en el número de consultas, en comparación con la media diaria histórica (en los últimos 16 días), por grupo de cargas de trabajo, usuario y aplicación.
 - Identificar las tendencias y los picos en el número de consultas y el consumo de memoria y CPU por grupo de cargas de trabajo, usuario, aplicación y tipo de comando.

[![Captura de pantalla de la vista de operaciones con gráficos de anillos de las principales aplicaciones por recuento de comandos y consultas, principales entidades de seguridad por recuento de comandos y consultas y comandos principales por tipos de comandos](./media/data-explorer/usage.png)](./media/data-explorer/usage.png#lightbox)

[![Captura de pantalla de la vista de operaciones con gráficos de líneas del recuento de consultas por aplicación, memoria total por aplicación y CPU total por aplicación](./media/data-explorer/usage-2.png)](./media/data-explorer/usage-2.png#lightbox)

En la pestaña **Tablas** se muestran las propiedades históricas y más recientes de las tablas del clúster. Puede ver qué tablas consumen más espacio, realizar un seguimiento del historial de crecimiento por tamaño de tabla, los datos de acceso frecuente y el número de filas a lo largo del tiempo.

La pestaña **Caché** permite a los usuarios analizar los patrones de la ventana de recuperación de las consultas reales y compararlos con la directiva de caché configurada (para cada tabla). Puede identificar las tablas que usan la mayoría de las consultas y aquellas que no se consultan nunca, y adaptar la directiva de caché en consecuencia. Puede obtener recomendaciones específicas de la directiva de caché sobre tablas concretas de Azure Advisor (actualmente, las recomendaciones de caché solo están disponibles en el [panel principal de Azure Advisor](https://docs.microsoft.com/azure/data-explorer/azure-advisor#use-the-azure-advisor-recommendations)) basadas en la ventana de recuperación de las consultas reales en los últimos 30 días y una directiva de caché no optimizada para al menos el 95 % de las consultas. Existen recomendaciones de reducción de la caché en Azure Advisor para los clústeres que están "enlazados por datos" (lo que significa que el clúster tiene un uso bajo de CPU y de ingesta, pero que, debido a su elevada capacidad de datos, no ha podido escalarse ni reducirse verticalmente).

[![Captura de pantalla de detalles de la caché](./media/data-explorer/cache-tab.png)](./media/data-explorer/cache-tab.png#lightbox)

La pestaña **cluster boundaries** (límites del clúster) muestra los límites del clúster en función de su uso. En esta pestaña puede inspeccionar el uso de la CPU, la ingesta y la caché. Los valores de estas métricas pueden ser como "Baja", "Media" o "Alta". Estas métricas y valores son importantes a la hora de decidir el número óptimo de SKU e instancias para el clúster, y se tienen en cuenta en la recomendación de tamaño y SKU de Azure Advisor. En esta pestaña puede seleccionar el icono de una métrica y profundizar para conocer su tendencia y cómo se decide su puntuación. También puede ver la recomendación de tamaño y SKU de Azure Advisor del clúster. Por ejemplo, en la siguiente imagen puede ver que los valores de todas las métricas son "Baja" y, por tanto, el clúster recibe una recomendación de costos, lo que le permite reducir o reducir horizontalmente y ahorrar costos.

> [!div class="mx-imgBorder"]
> [![Captura de pantalla de los límites del clúster.](./media/data-explorer/cluster-boundaries.png)](./media/data-explorer/cluster-boundaries.png#lightbox)

## <a name="pin-to-azure-dashboard&quot;></a>Anclar al panel de Azure

Cualquiera de las secciones de métricas (de la perspectiva &quot; a gran escala") se puede anclar a un panel de Azure; para ello, seleccione el icono de chincheta que se encuentra en la parte superior derecha de la sección.

![Captura de pantalla del icono de chincheta seleccionado](./media/data-explorer/pin.png)

## <a name="customize-azure-data-explorer-insights"></a>Personalización de Azure Data Explorer Insights

En esta sección se resaltan los escenarios comunes para editar el libro y personalizarlo de acuerdo con sus necesidades de análisis de datos:
* Definir el ámbito del libro para seleccionar siempre una suscripción o unos clústeres de Azure Data Explorer determinados
* Cambiar las métricas de la cuadrícula
* Cambiar los umbrales o la representación o codificación del color

Puede iniciar las personalizaciones habilitando el modo de edición; para ello, seleccione el botón **Personalizar** de la barra de herramientas superior.

![Captura de pantalla del botón Personalizar](./media/data-explorer/customize.png)

Las personalizaciones se guardan en un libro personalizado para evitar sobrescribir la configuración predeterminada del libro publicado. Los libros se guardan en un recurso compartido, bien en la sección Mis informes, que es privada para usted, o en la sección Informes compartidos, a la que puede acceder cualquiera con acceso al grupo de recursos. Después de guardar el libro personalizado, debe ir a la galería de libros para iniciarlo.

![Captura de pantalla de la galería de libros](./media/data-explorer/gallery.png)

## <a name="troubleshooting"></a>Solución de problemas

Para obtener instrucciones generales para la solución de problemas, consulte el [artículo dedicado de solución de problemas](troubleshoot-workbooks.md) de conclusiones basadas en libros.

Esta sección le ayudará con el diagnóstico y la solución de algunos de los problemas habituales que pueden producirse en la aplicación al usar Azure Data Explorer Insights (versión preliminar). Utilice la siguiente lista para buscar la información relacionada con el problema específico.

### <a name="why-dont-i-see-all-my-subscriptions-in-the-subscription-picker"></a>¿Por qué veo todas mis suscripciones en el selector de suscripciones?

Solo se muestran las suscripciones que contienen clústeres de Azure Data Explorer, que se eligen con el filtro de suscripciones seleccionado y que se seleccionan en la sección "Directorio + suscripción" en el encabezado de Azure Portal.

![Captura de pantalla del filtro de suscripción](./media/key-vaults-insights-overview/Subscriptions.png)

### <a name="why-do-i-not-see-any-data-for-my-azure-data-explorer-cluster-under-the-usage-tables-or-cache-sections"></a>¿Por qué no veo ningún dato del clúster de Azure Data Explorer en las secciones Uso, Tablas o Caché?

Para ver los datos basados en registros, deberá [habilitar los registros de diagnóstico](/azure/data-explorer/using-diagnostic-logs?tabs=commands-and-queries#enable-diagnostic-logs) para cada uno de los clústeres de Azure Data Explorer que quiera supervisar. Esto puede realizarse en la configuración de diagnóstico de cada clúster. Tendrá que enviar los datos a un área de trabajo de Log Analytics. Los registros de diagnóstico que se deben habilitar son: Command, Query, TableDetails y TableUsageStatistics.

### <a name="i-have-already-enabled-logs-for-my-azure-data-explorer-cluster-why-am-i-still-unable-to-see-my-data-under-commands-and-queries"></a>Ya he habilitado los registros del clúster de Azure Data Explorer, ¿por qué todavía no puedo ver mis datos en las secciones Comandos y Consultas?

Actualmente, los registros de diagnóstico no funcionan de forma retroactiva, por lo que los datos solo comenzarán a aparecer una vez que se hayan realizado acciones en Azure Data Explorer. Por lo tanto, pueden tardar algún tiempo, desde horas hasta un día, en función de la actividad que presente el clúster de Azure Data Explorer.


## <a name="next-steps"></a>Pasos siguientes

Conozca los escenarios para los que están concebidos los libros, cómo crear informes y personalizar los ya existentes y otros muchos temas en el artículo [Crear informes interactivos con libros de Azure Monitor](../visualize/workbooks-overview.md).
