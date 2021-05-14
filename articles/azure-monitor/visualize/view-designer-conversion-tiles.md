---
title: Conversiones de icono del diseñador de vistas de Azure Monitor en libros
description: Detalles para convertir iconos en libros cuando se realiza la transición desde vistas en Azure Monitor.
author: shijatsu
ms.author: shijain
ms.topic: conceptual
ms.date: 02/07/2020
ms.openlocfilehash: b73aa0569fb9889de687e7b3eacba5a0fcd54cc1
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108318016"
---
# <a name="azure-monitor-view-designer-tile-conversions"></a>Conversiones de icono del diseñador de vistas de Azure Monitor
El [diseñador de vistas](view-designer.md) es una característica de Azure Monitor que permite crear vistas personalizadas para ayudar a visualizar datos en el área de trabajo de Log Analytics, con gráficos, listas y escalas de tiempo. Estas vistas se están retirando paulatinamente y están siendo reemplazadas por libros, que proporcionan funcionalidad adicional. En este artículo se proporciona información detallada para convertir distintos iconos en libros.

## <a name="donut--list-tile"></a>Icono de anillo y lista

![Anillo y lista](media/view-designer-conversion-tiles/donut-list.png)

Para reproducir el icono de Anillo y lista en los libros se necesitan dos visualizaciones independientes. Para la parte del anillo, hay dos opciones.
Para empezar, seleccione **Agregar consulta** y pegue la consulta original del diseñador de vistas en la celda.

**Opción 1:** seleccionar **Gráfico circular** en el menú desplegable **Visualización**: ![Menú de visualización de gráfico circular](media/view-designer-conversion-tiles/pie-chart.png)

**Opción 2:** seleccionar **Establecer por consulta** en el menú desplegable **Visualización** y agregar `| render piechart` a la consulta:

 ![Menú Visualización](media/view-designer-conversion-tiles/set-by-query.png)

**Ejemplo**

Consulta original
```KQL
search * 
| summarize AggregatedValue = count() by Type 
| order by AggregatedValue desc
```

Consulta actualizada
```KQL
search * 
| summarize AggregatedValue = count() by Type 
| order by AggregatedValue desc 
| render piechart
```

Para crear una lista y habilitar los minigráficos, consulte el artículo sobre [tareas comunes](view-designer-conversion-tasks.md).

A continuación se muestra un ejemplo de cómo se podría ver el icono de anillo y lista en los libros:

![Libros de anillo y lista](media/view-designer-conversion-tiles/donut-workbooks.png)

## <a name="line-chart--list-tile"></a>Icono de gráfico de líneas y lista
![Gráfico de líneas y lista](media/view-designer-conversion-tiles/line-list.png) 

Para reproducir la parte del gráfico de líneas, actualice la consulta de la siguiente manera:

Consulta original
```KQL
search * 
| summarize AggregatedValue = count() by Type
```

Consulta actualizada
```KQL
search * 
| make-series Count = count() default=0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by Type
```

Hay dos opciones para visualizar el gráfico de líneas.

**Opción 1:** seleccionar **Gráfico de líneas** en el menú desplegable **Visualización**:
 
 ![Menú del gráfico de líneas](media/view-designer-conversion-tiles/line-visualization.png)

**Opción 2:** seleccionar **Establecer por consulta** en el menú desplegable **Visualización** y agregar `| render linechart` a la consulta:

 ![Menú Visualización](media/view-designer-conversion-tiles/set-by-query.png)

**Ejemplo**

```KQL
search * 
| make-series Count = count() default=0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by Type 
| render linechart_
```

Para crear una lista y habilitar los minigráficos, consulte el artículo sobre [tareas comunes](view-designer-conversion-tasks.md).

A continuación se muestra un ejemplo de cómo se podría ver el icono de gráfico de líneas y lista en los libros:

![Libros de gráfico de líneas y lista](media/view-designer-conversion-tiles/line-workbooks.png)

## <a name="number--list-tile"></a>Icono de número y lista

 ![Icono de lista](media/view-designer-conversion-tiles/tile-list-example.png)

Para el icono de número, actualice la consulta de la siguiente manera:

Consulta original
```KQL
search * 
| summarize AggregatedValue = count() by Computer | count
```

Consulta actualizada
```KQL
search *
| summarize AggregatedValue = count() by Computer 
| summarize Count = count()
```

Cambie la lista desplegable Visualización a **Iconos** y, después, seleccione **Configuración de icono**.
 ![Icono en Visualización](media/view-designer-conversion-tiles/tile-visualization.png)

Deje en blanco la sección **Título** y seleccione **Izquierda**. Cambie el valor de **Usar columna:** a **Recuento** y el de **Representador de columnas** a **Número grande**:

![Configuración de icono](media/view-designer-conversion-tiles/tile-settings.png)

 
Para crear una lista y habilitar los minigráficos, consulte el artículo sobre [tareas comunes](view-designer-conversion-tasks.md).

A continuación se muestra un ejemplo de cómo se podría ver el icono de número y lista en los libros:

![Libros de número y lista](media/view-designer-conversion-tiles/number-workbooks.png)

## <a name="timeline--list"></a>Escala de tiempo y lista

 ![Escala de tiempo y lista](media/view-designer-conversion-tiles/time-list.png)

Para la escala de tiempo, actualice la consulta de la siguiente manera:

Consulta original
```KQL
search * 
| sort by TimeGenerated desc
```

Consulta actualizada
```KQL
search * 
| summarize Count = count() by Computer, bin(TimeGenerated,{TimeRange:grain})
```

Hay dos opciones para visualizar la consulta como un gráfico de barras:

**Opción 1:** seleccionar **Gráfico de barras** en el menú desplegable **Visualización**: ![Menú de visualización de gráfico de barras](media/view-designer-conversion-tiles/bar-visualization.png)
 
**Opción 2:** seleccionar **Establecer por consulta** en el menú desplegable **Visualización** y agregar `| render barchart` a la consulta:

 ![Menú Visualización](media/view-designer-conversion-tiles/set-by-query.png)

 
Para crear una lista y habilitar los minigráficos, consulte el artículo sobre [tareas comunes](view-designer-conversion-tasks.md).

A continuación se muestra un ejemplo de cómo se podría ver el icono de escala de tiempo y lista en los libros:

![Libros de escala de tiempo y lista](media/view-designer-conversion-tiles/time-workbooks.png)

## <a name="next-steps"></a>Pasos siguientes

- [Información general de la transición del diseñador de vistas a libros](view-designer-conversion-overview.md)
