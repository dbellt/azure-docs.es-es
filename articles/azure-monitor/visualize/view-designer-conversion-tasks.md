---
title: Tareas comunes de conversión del diseñador de vistas de Azure Monitor en libros
description: Tareas comunes para realizar la transición desde vistas a libros en Azure Monitor.
author: shijatsu
ms.author: shijain
ms.topic: conceptual
ms.date: 02/07/2020
ms.openlocfilehash: 3b1147b4a16fd453605bb4bab6d48605e110b995
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108318088"
---
# <a name="view-designer-to-workbooks-conversion-common-tasks"></a>Tareas comunes de conversión del diseñador de vistas en libros
El [diseñador de vistas](view-designer.md) es una característica de Azure Monitor que permite crear vistas personalizadas para ayudar a visualizar datos en el área de trabajo de Log Analytics, con gráficos, listas y escalas de tiempo. Estas vistas se están retirando paulatinamente y están siendo reemplazadas por libros, que proporcionan funcionalidad adicional. En este artículo se detallan las tareas comunes para la conversión de vistas en libros.


## <a name="quickstart-with-preset-view-designer-templates"></a>Guía de inicio rápido con plantillas preestablecidas del diseñador de vistas

Los libros de áreas de trabajo de Log Analytics ya tienen plantillas que coinciden con algunas de las vistas en el diseñador de vistas. En la categoría **Guías del diseñador de vistas**, seleccione **Guía de transición del diseñador de vistas** para obtener información sobre las opciones, o bien seleccione una de las plantillas preestablecidas.

![Plantillas de ejemplo](media/view-designer-conversion-tasks/templates.png)

## <a name="enabling-time-range-filter"></a>Habilitación del filtro de intervalo de tiempo
El diseñador de vistas tiene un filtro de intervalo de tiempo predeterminado integrado; sin embargo, en los libros, esta configuración no está habilitada de manera predeterminada. Los libros permiten a los usuarios crear sus propios filtros de intervalo de tiempo, que podrían ajustarse mejor a sus registros de datos. A continuación se enumeran los pasos para generar un filtro:

Seleccione la opción **Agregar parámetros**. El **Estilo** predeterminado está establecido en *Pills*.

![Agregar parámetro](media/view-designer-conversion-tasks/add-param.png)

 Seleccione el botón **Agregar parámetro**.

![Agregar parámetro](media/view-designer-conversion-tasks/add-parameter.png)

En el menú de la barra lateral, dentro del cuadro de texto **Nombre de parámetro**, escriba *TimeRange*. Establezca el **Tipo de parámetro** como *Selector de intervalo de tiempo*. Marque la casilla **¿Obligatorio?**.

![Menú de parámetro](media/view-designer-conversion-tasks/parameter-menu.png)

Guarde el parámetro desde la esquina superior izquierda del menú de la barra lateral. Puede dejar la lista desplegable *sin establecer* de forma predeterminada, o bien seleccionar un valor predeterminado de **TimeRange**, como *24 horas*. Seleccione **Edición finalizada**.

Los parámetros se pueden usar en las consultas; para ello, agregue llaves {} alrededor del nombre del parámetro. Puede encontrar más detalles sobre los parámetros en la [documentación sobre parámetros para libros](https://github.com/microsoft/Application-Insights-Workbooks/blob/master/Documentation/Parameters/Parameters.md).

## <a name="updating-queries-with-the-timerange-parameter"></a>Actualización de consultas con el parámetro TimeRange

### <a name="option-1-select-timerange-from-the-time-range-dropdown"></a>Opción 1: Seleccionar TimeRange en la lista desplegable del intervalo de tiempo

![Parámetro de tiempo](media/view-designer-conversion-tasks/time-parameter.png)

### <a name="option-2-update-your-log-queries"></a>Opción 2: Actualizar las consultas del registro

En la consulta, agregue la línea: `| where TimeGenerated {TimeRange}` como en el siguiente ejemplo:

Consulta original
```KQL
search * 
| summarize count() by Type
```

Consulta actualizada
```KQL
search * 
| where TimeGenerated {TimeRange} 
| summarize count() by Type
```

## <a name="including-a-list"></a>Incorporación de una lista
La mayoría de las vistas del diseñador de vistas incluyen una lista, y puede reproducir estas listas estándar en un libro.

![Icono de lista](media/view-designer-conversion-tasks/tile-list.png)

Agregue una visualización al hacer clic en **Agregar consulta** en las opciones de celda.

![Agregar parámetro](media/view-designer-conversion-tasks/add-param.png)

El diseñador de vistas emplea una consulta predeterminada que coincide con la sintaxis del ejemplo original. Puede actualizarla cambiando la consulta al formulario actualizado como se indica en el ejemplo siguiente:

Consulta original
```KQL
search * 
| summarize AggregatedValue = count() by Type
```

Consulta actualizada
```KQL
search * 
| summarize Count = count() by Type
```

Se generará una lista con un aspecto similar al siguiente:

![Ejemplo de lista](media/view-designer-conversion-tasks/list-example.png)

## <a name="enabling-sparklines"></a>Habilitación de minigráficos
Una característica común de las cuadrículas implica agregar minigráficos para resumir varios patrones de datos a lo largo del tiempo. El diseñador de vistas ofrece la característica **Habilitar los minigráficos** para todas las listas, al igual que los libros. Para incluir en los datos minigráficos que coinciden con el diseñador de vistas, combine los datos con la consulta original, como en el ejemplo siguiente:

Consulta original
```KQL
search *
| summarize AggregatedValue = count() by Type) on Type
```

Consulta actualizada
```KQL
search * 
| summarize AggregatedValue = count() by Type
| join kind = inner (search * 
    | make-series Trend = count() default=0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by Type) on Type
| project Type, AggregatedValue, Trend
```

Seleccione **Configuración de columnas**.
![Configuración de columnas](media/view-designer-conversion-tasks/column-settings.png)

Actualice la lista desplegable **Representador de columnas** para que sea un *Área de Spark*.
![Minigráficos](media/view-designer-conversion-tasks/sparkline.png)

Guarde la configuración y vuelva a ejecutar la consulta; de este modo, se actualizará la tabla para incluir un minigráfico.

La cuadrícula resultante será similar a la siguiente: ![Ejemplo de minigráficos](media/view-designer-conversion-tasks/sparkline-example.png)

## <a name="advanced-cell-settings"></a>Configuración avanzada de celda
Al igual que en el diseñador de vistas, puede realizar tareas como cambiar el tamaño de las celdas del libro o agregar pines y vínculos externos a los registros.

Para acceder a la **Configuración avanzada**, seleccione el icono de engranaje en la parte inferior de cada celda.

![Configuración avanzada](media/view-designer-conversion-tasks/advanced-settings.png)

Se mostrará un menú con varias opciones:

![Opciones de configuración avanzada](media/view-designer-conversion-tasks/advanced-settings-settings.png)

Para agregar un pin y un vínculo a una consulta externa, marque las casillas correspondientes. Para agregar un título a la celda, escriba el título deseado en la sección **Título del gráfico**.

De forma predeterminada, todas las celdas de los libros están configuradas para ocupar todo el ancho de la página, pero puede ajustar este comportamiento al reducir el tamaño de la celda en la pestaña **Estilo** del menú **Configuración avanzada**.

![Estilo de la configuración avanzada](media/view-designer-conversion-tasks/advanced-settings-style.png)

 
## <a name="additional-parameters"></a>Parámetros adicionales
Seleccione **Agregar parámetro** para crear un nuevo parámetro en el libro. 

Para seleccionar una suscripción, escriba *Subscription* en el campo **Nombre de parámetro** del menú lateral y seleccione *Selector de suscripciones* en la lista desplegable **Tipo de parámetro**.

![Menú de suscripción](media/view-designer-conversion-tasks/subscription-filter.png)

Para seleccionar un recurso, escriba *Resource* en el campo **Nombre de parámetro** del menú lateral y seleccione *Selector de recursos* en la lista desplegable **Tipo de parámetro**.

![Menú de recurso](media/view-designer-conversion-tasks/resource-filter.png)

Esta acción insertará listas desplegables para permitirle acceder a las distintas suscripciones y recursos.

![Lista desplegable para suscripción y recurso](media/view-designer-conversion-tasks/subscription-resource.png)


## <a name="next-steps"></a>Pasos siguientes
- [Conversiones de iconos](view-designer-conversion-tiles.md)
