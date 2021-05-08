---
title: Opciones de conversión del diseñador de vistas de Azure Monitor en libros
description: Ejemplos de conversión para la transición desde vistas a libros en Azure Monitor.
author: shijatsu
ms.author: shijain
ms.topic: conceptual
ms.date: 02/07/2020
ms.openlocfilehash: 1110a19f67c41e5cc170e056f6d337bdf8860588
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108318070"
---
# <a name="azure-monitor-view-designer-to-workbooks-conversion-options"></a>Opciones de conversión del diseñador de vistas de Azure Monitor en libros
El [diseñador de vistas](view-designer.md) es una característica de Azure Monitor que permite crear vistas personalizadas para ayudar a visualizar datos en el área de trabajo de Log Analytics, con gráficos, listas y escalas de tiempo. Estas vistas se están retirando paulatinamente y están siendo reemplazadas por libros, que proporcionan funcionalidad adicional. En este artículo se comparan los conceptos fundamentales entre las dos y opciones para convertir vistas en libros.

## <a name="basic-workbook-designs"></a>Diseños de libros básicos

El diseñador de vistas tiene un estilo estático fijo de representación, mientras que los libros permiten la libertad de incluir y modificar el modo en que se representan los datos. En las imágenes siguientes se describen dos ejemplos de cómo puede organizar los libros al convertir las vistas.

[Libro vertical](view-designer-conversion-examples.md#vertical)
![Vertical](media/view-designer-conversion-options/view-designer-vertical.png)

[Libro con pestañas](view-designer-conversion-examples.md#tabbed)
Pestaña Distribución de tipo de datos ![](media/view-designer-conversion-options/distribution-tab.png)
![Pestaña tipos de datos a lo largo del tiempo](media/view-designer-conversion-options/over-time-tab.png)

## <a name="tile-conversion"></a>Conversión de iconos
El diseñador de vistas utiliza la característica de icono de información general para representar y resumir el estado general. Se representan en siete iconos, que van desde números hasta gráficos. En los libros, los usuarios pueden crear visualizaciones similares y anclarlas de forma similar al estilo original de los iconos de información general. 

![Galería](media/view-designer-conversion-options/overview.png)


## <a name="view-dashboard-conversion"></a>Visualización de la conversión del panel
Los iconos del diseñador de vistas normalmente constan de dos secciones, una visualización y una lista que coincide con los datos de la visualización; por ejemplo, el icono **Gráfico de anillos y lista**.

![Anillo](media/view-designer-conversion-options/donut-example.png)

En los libros, se permite al usuario elegir si consultar una o ambas secciones de la vista. La formulación de consultas en los libros es un proceso simple de dos pasos. En primer lugar, los datos se generan a partir de la consulta y, en segundo lugar, los datos se representan como una visualización.  A continuación se muestra un ejemplo de cómo se volvería a crear esta vista en los libros:

![Convert](media/view-designer-conversion-options/convert-donut.png)


## <a name="next-steps"></a>Pasos siguientes
- [Acceso a libros y permisos](view-designer-conversion-access.md)