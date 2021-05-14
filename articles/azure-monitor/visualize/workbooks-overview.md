---
title: Introducción a los libros de Azure Monitor
description: Obtenga información sobre cómo los libros proporcionan un lienzo flexible para el análisis de datos y la creación de informes visuales completos en el Azure Portal.
services: azure-monitor
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 07/23/2020
ms.openlocfilehash: 3d75d7605ba082aac84973aef247de79d55b4c9c
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107482779"
---
# <a name="azure-monitor-workbooks"></a>Libros de Azure Monitor

Los libros proporcionan un lienzo flexible para el análisis de datos y la creación de informes visuales completos en el Azure Portal. Permiten acceder a varios orígenes de datos desde Azure y combinarlos en experiencias interactivas unificadas.

Este es un tutorial en vídeo sobre la creación de libros.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4B4Ap]

## <a name="data-sources"></a>Orígenes de datos

Los libros pueden consultar datos de varios orígenes dentro de Azure. Los autores de los libros pueden transformar estos datos para proporcionar información sobre la disponibilidad, el rendimiento, el uso y el estado general de los componentes subyacentes. Por ejemplo, el análisis de los registros de rendimiento de las máquinas virtuales para identificar grandes instancias de CPU o de memoria insuficiente y mostrar los resultados como una cuadrícula en un informe interactivo.
  
Pero la eficacia real de los libros es la capacidad de combinar datos de orígenes dispares dentro de un único informe. Esto permite la creación de vistas de recursos compuestas o combinaciones en los recursos, lo que permite obtener datos e información más completos que, de lo contrario, serían imposibles.

Los libros son compatibles actualmente con los siguientes orígenes de datos:

* [Registros](../visualize/workbooks-data-sources.md#logs)
* [Métricas](../visualize/workbooks-data-sources.md#metrics)
* [Azure Resource Graph](../visualize/workbooks-data-sources.md#azure-resource-graph)
* [Alertas (versión preliminar)](../visualize/workbooks-data-sources.md#alerts-preview)
* [Mantenimiento de la carga de trabajo](../visualize/workbooks-data-sources.md#workload-health)
* [Azure Resource Health](../visualize/workbooks-data-sources.md#azure-resource-health)
* [Azure Data Explorer](../visualize/workbooks-data-sources.md#azure-data-explorer)

## <a name="visualizations"></a>Visualizaciones

Los libros proporcionan un amplio conjunto de funciones para visualizar los datos. Para obtener ejemplos detallados de cada tipo de visualización, puede consultar los siguientes vínculos:

* [Texto](../visualize/workbooks-text-visualizations.md)
* [Gráficos](../visualize/workbooks-chart-visualizations.md)
* [Cuadrículas](../visualize/workbooks-grid-visualizations.md)
* [Iconos](../visualize/workbooks-tile-visualizations.md)
* [Árboles](../visualize/workbooks-tree-visualizations.md)
* [Gráficos](../visualize/workbooks-graph-visualizations.md)
* [Barra compuesta](../visualize/workbooks-composite-bar.md)

:::image type="content" source="./media/workbooks-overview/visualizations.png" alt-text="Ejemplo de visualizaciones de libros" border="false" lightbox="./media/workbooks-overview/visualizations.png":::

### <a name="pinning-visualizations"></a>Anclado de visualizaciones

Los pasos de texto, consulta y métricas de un libro se pueden anclar mediante el botón de anclaje en esos elementos, mientras el libro está en modo de anclaje, o bien, si el autor del libro tiene habilitada la configuración para ese elemento con el fin de que el icono para anclar esté visible.

Para acceder al modo de anclaje, haga clic en **editar** para entrar en el modo de edición, luego seleccione el icono para anclar azul en la barra superior. Luego aparecerá un icono de anclaje individual por encima de cada cuadro de *Editar* de la parte correspondiente del libro en el lado derecho de la pantalla.

:::image type="content" source="./media/workbooks-overview/pin-experience.png" alt-text="Captura de pantalla de la experiencia de anclado." border="false":::

> [!NOTE]
> El estado del libro se guarda en el momento del anclaje, y los libros anclados en un panel no se actualizarán si se modifica el libro subyacente. Para actualizar una parte del libro anclada, deberá eliminarla y volver a anclarla.

## <a name="getting-started"></a>Introducción

Para explorar la experiencia de los libros, navegue primero hasta el servicio Azure Monitor. Para ello, escriba **monitor** en el cuadro de búsqueda de Azure Portal.

A continuación, seleccione **Libros**.

:::image type="content" source="./media/workbooks-overview/workbooks.png" alt-text="Captura de pantalla con el botón Libros resaltado con un cuadro rojo." border="false":::

### <a name="gallery"></a>Galería

La galería facilita la organización, ordenación y administración de libros de todos los tipos.

:::image type="content" source="./media/workbooks-overview/gallery-all-tab.png" alt-text="Captura de pantalla de la galería en la pestaña Todos." lightbox="media/workbooks-overview/gallery-all-tab.png":::

#### <a name="gallery-tabs"></a>Pestañas de la galería

Hay cuatro pestañas en la galería para ayudar a organizar los tipos de libro.

| Pestaña              | Descripción                                       |
|------------------|---------------------------------------------------|
| Todo | Muestra los cuatro elementos principales de cada tipo: libros, plantillas públicas y mis plantillas. Los libros se ordenan por fecha de modificación, por lo que verá los ocho libros modificados más recientes.|
| Workbooks | Muestra la lista de todos los libros disponibles que creó o que se comparten con usted. |
| Plantillas públicas | Muestra la lista de todas las plantillas de libro funcionales disponibles listas para usar publicadas por Microsoft. Por categoría. |
| Mis plantillas | Muestra la lista de todas las plantillas de libro implementadas disponibles que creó o que se comparten con usted. Por categoría. |

#### <a name="features"></a>Características

* En cada pestaña, hay una cuadrícula con información sobre los libros. Incluye la descripción, la fecha de última modificación, las etiquetas, la suscripción, el grupo de recursos, la región y el estado compartido. También puede ordenar los libros por esta información.
* Filtre por grupo de recursos, suscripciones, nombre de libro o plantilla o categoría de plantilla.
* Seleccione varios libros para eliminar o eliminar de forma masiva.
* Cada libro tiene un menú contextual (puntos suspensivos/tres puntos al final), al seleccionarlo se abrirá una lista de acciones rápidas.
    * Ver recurso: acceda a la pestaña de recursos del libro para ver el identificador de recurso del libro, agregar etiquetas, administrar bloqueos, etc.
    * Elimine o cambie el nombre del libro.
    * Ancle el libro al panel.

### <a name="workbooks-versus-workbook-templates"></a>Libros frente a plantillas de libro

Puede ver un _libro_ en verde y varias _plantillas de libro_ en color púrpura. Las plantillas sirven como informes seleccionados que están diseñados para que varios usuarios y equipos puedan reutilizarlos de forma flexible. Al abrir una plantilla se crea un libro transitorio que se rellena con el contenido de la plantilla.

Puede ajustar los parámetros del libro basado en la plantilla y realizar el análisis sin miedo a interrumpir la futura experiencia de informes para colegas. Si abre una plantilla, realice algunos ajustes y luego seleccione el icono guardar para guardar la plantilla como un libro que se mostrará en verde y la plantilla original quedará intacta.

En el fondo, las plantillas también difieren de los libros guardados. Al guardar un libro, se crea un recurso de Azure Resource Manager asociado, mientras que el libro transitorio que se crea al abrir una plantilla no tiene asociado un recurso único. Para obtener más información sobre cómo se administra el control de acceso en los libros, consulte el [artículo control de acceso de los libros](../visualize/workbooks-access-control.md).

### <a name="exploring-a-workbook-template"></a>Exploración de una plantilla de libro

Seleccione **Análisis de los errores de la aplicación** para ver una de las plantillas de libro de la aplicación predeterminadas.

:::image type="content" source="./media/workbooks-overview/failure-analysis.png" alt-text="Captura de pantalla de la plantilla de análisis de los errores de la aplicación." border="false" lightbox="./media/workbooks-overview/failure-analysis.png":::

Como se indicó anteriormente, al abrir la plantilla se crea un libro temporal para poder interactuar con él. De forma predeterminada, el libro se abre en modo de lectura, lo que muestra solo la información de la experiencia de análisis previsto creada por el autor original de la plantilla.

En el caso de este libro concreto, la experiencia es interactiva. Puede ajustar la suscripción, las aplicaciones de destino y el intervalo de tiempo de los datos que desea mostrar. Una vez realizadas esas selecciones, la cuadrícula de solicitudes HTTP también es interactiva, por lo que seleccionar una fila individual cambiará los datos que se representan en los dos gráficos en la parte inferior del informe.

### <a name="editing-mode"></a>Modo de edición

Para comprender cómo se agrupa esta plantilla de libro, debe cambiar al modo de edición seleccionando **Editar**.

:::image type="content" source="./media/workbooks-overview/edit.png" alt-text="Captura de pantalla del botón en libros." border="false" :::

Una vez que haya cambiado al modo de edición, observará que aparece un número de cuadros de **Editar** a la derecha correspondiente con cada aspecto individual del libro.

:::image type="content" source="./media/workbooks-overview/edit-mode.png" alt-text="Captura de pantalla del botón de edición." border="false" lightbox="./media/workbooks-overview/edit-mode.png":::

Si seleccionamos el botón Editar inmediatamente debajo de la cuadrícula de los datos de la solicitud, podemos ver que esta parte del libro consta de una consulta Kusto en los datos de un recurso de Application Insights.

:::image type="content" source="./media/workbooks-overview/kusto.png" alt-text="Captura de pantalla de la consulta Kusto subyacente." border="false" lightbox="./media/workbooks-overview/kusto.png":::

Al seleccionar los demás botones **Editar** de la derecha, se mostrarán varios componentes principales que conforman los libros, como los [cuadros de texto](../visualize/workbooks-text-visualizations.md) basados en marcado, los elementos de la interfaz de usuario de la [selección de parámetros](../visualize/workbooks-parameters.md) y otros [tipos de gráficos y visualizaciones](#visualizations).

Explorar las plantillas predefinidas en modo de edición y después modificarlas para adaptarlas a sus necesidades y guardar su propio libro personalizado es una excelente manera de empezar a obtener información sobre lo que es posible con los libros de Azure Monitor.

## <a name="dashboard-time-ranges"></a>Intervalos de tiempo del panel

Los elementos de consulta del libro anclado respetarán el intervalo de tiempo del panel si el elemento anclado está configurado para usar un parámetro de *Intervalo de tiempo*. El valor del intervalo de tiempo del panel se usará como el valor del parámetro de intervalo de tiempo y cualquier cambio del intervalo de tiempo del panel hará que el elemento anclado se actualice. Si una parte anclada usa el intervalo de tiempo del panel, verá el subtítulo de la actualización de la parte anclada para mostrar el intervalo de tiempo del panel cada vez que cambie el intervalo de tiempo.

Además, los elementos del libro anclados con un parámetro de intervalo de tiempo se actualizarán automáticamente a una velocidad determinada por el intervalo de tiempo del panel. La última vez que se ejecutó la consulta aparecerá en el subtítulo de la parte anclada.

Si un paso anclado tiene un intervalo de tiempo establecido explícitamente (no usa un parámetro de intervalo de tiempo), ese intervalo de tiempo se usará siempre para el panel, independientemente de la configuración del panel. El subtítulo de la parte anclada no mostrará el intervalo de tiempo del panel y la consulta no se actualizará automáticamente en el panel. El subtítulo mostrará la última vez que se ejecutó la consulta.

> [!NOTE]
> Actualmente no se admiten consultas con el origen de datos de *combinación* al anclar a los paneles.

## <a name="sharing-workbook-templates"></a>Compartir plantillas de libro

Una vez que empiece a crear sus propias plantillas de libro, es posible que quiera compartirla con la comunidad. Para obtener más información y explorar otras plantillas que no forman parte de la vista predeterminada de la galería de Azure Monitor, visite nuestro [repositorio GitHub](https://github.com/Microsoft/Application-Insights-Workbooks/blob/master/README.md). Para examinar los libros existentes, visite la [biblioteca de libros](https://github.com/microsoft/Application-Insights-Workbooks/tree/master/Workbooks) en GitHub.

## <a name="next-step"></a>Paso siguiente

* [Comience](#visualizations) a aprender más sobre las muchas opciones de visualizaciones enriquecidas de los libros.
* [Controle](../visualize/workbooks-access-control.md) y comparta el acceso a los recursos del libro.