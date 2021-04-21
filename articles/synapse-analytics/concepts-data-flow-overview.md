---
title: Flujos de datos
description: Información general sobre los flujos de datos en Azure Synapse Analytics
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
ms.service: synapse-analytics
ms.subservice: pipeline
ms.topic: conceptual
ms.custom: references_regions
ms.date: 12/16/2020
ms.openlocfilehash: 4769cc8abe121625f3bf77785cd681c0f649d166
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107567696"
---
# <a name="data-flows-in-azure-synapse-analytics"></a>Flujos de datos en Azure Synapse Analytics

## <a name="what-are-data-flows"></a>¿Qué son los flujos de datos?

Los flujos de datos son transformaciones de datos diseñada visualmente en Azure Synapse Analytics. Los flujos de datos permiten a los ingenieros de datos desarrollar lógica de transformación de datos sin necesidad de escribir código. Los flujos de datos resultantes se ejecutan como actividades en las canalizaciones de Azure Synapse Analytics que usan clústeres de Apache Spark con escalabilidad horizontal. Las actividades de flujo de datos pueden ponerse en marcha mediante las funcionalidades de programación, control, flujo y supervisión existentes de Azure Synapse Analytics.

Los flujos de datos proporcionan una experiencia completamente visual que no requiere programación. Los flujos de datos se ejecutan en clústeres de ejecución administrados por Synapse durante el procesamiento de datos de escalabilidad horizontal. Asimismo, Azure Synapse Analytics controla toda la traducción de código, la optimización de rutas de acceso y la ejecución de los trabajos de flujo de datos.

## <a name="getting-started"></a>Introducción

Los flujos de datos se crean desde el panel Desarrollar de Synapse Studio. Para crear un flujo de datos, seleccione el signo más junto a **Desarrollar** y luego **Data Flow**. 

![Nuevo flujo de datos](media/data-flow/new-data-flow.png)

Con esta acción, accederá al lienzo de flujo de datos, donde podrá crear la lógica de transformación. Seleccione **Agregar origen** para comenzar a configurar la transformación de origen. Para más información, consulte [Transformación de origen](../data-factory/data-flow-source.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="authoring-data-flows"></a>Creación de flujos de datos

El flujo de datos tiene un lienzo de creación único diseñado para facilitar la creación de lógica de transformación. El lienzo de flujo de datos está dividido en tres partes: la barra superior, el gráfico y el panel de configuración. 

![Captura de pantalla que muestra el lienzo del flujo de datos con la barra superior, el gráfico y el panel de configuración marcados.](media/data-flow/canvas-1.png)

### <a name="graph"></a>Grafo

En el gráfico se muestra el flujo de transformación. Muestra el linaje de los datos de origen a medida que fluyen hacia uno o varios receptores. Para agregar un nuevo origen, seleccione **Agregar origen**. Para agregar una nueva transformación, seleccione el signo más situado en la parte inferior derecha de una transformación existente. Obtenga más información sobre cómo [administrar el gráfico de flujo de datos](../data-factory/concepts-data-flow-manage-graph.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

![Captura de pantalla que muestra la parte del gráfico del lienzo con un cuadro de texto de búsqueda.](media/data-flow/canvas-2.png)

### <a name="configuration-panel"></a>Panel de configuración

En el panel configuración se muestra la configuración específica de la transformación seleccionada actualmente. Si no se ha seleccionado ninguna transformación, se muestra el flujo de datos. En la configuración de flujo de datos general, puede agregar parámetros a través de la pestaña **Parámetros**. Para obtener más información, consulte [Parámetros de Data Flow](../data-factory/parameters-data-flow.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

Cada transformación contiene al menos cuatro pestañas de configuración.

#### <a name="transformation-settings"></a>Configuración de la transformación

La primera pestaña del panel de configuración de cada transformación contiene los valores específicos de esa transformación. Para más información, consulte la página de documentación de la transformación.

![Pestaña de configuración de origen](media/data-flow/source-1.png)

#### <a name="optimize"></a>Optimización

La pestaña **Optimizar** contiene valores opcionales para configurar los esquemas de partición. Para obtener más información sobre cómo optimizar los flujos de datos, consulte la [guía de rendimiento de flujos de datos de asignación](../data-factory/concepts-data-flow-performance.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

![Captura de pantalla que muestra la pestaña Optimizar](media/data-flow/optimize.png)

#### <a name="inspect"></a>Inspeccionar

La pestaña **Inspeccionar** proporciona una vista de los metadatos del flujo de datos que se está transformando. Puede ver el número de columnas, las columnas que han cambiado, las columnas que se han agregado, los tipos de datos, el orden de las columnas y las referencias de las columnas. **Inspeccionar** es una vista de solo lectura de los metadatos. Para ver los metadatos en el panel **Inspeccionar**, no es preciso que el modo de depuración esté habilitado.

![Pestaña Inspeccionar](media/data-flow/inspect.png)

Al cambiar la forma de los datos mediante transformaciones, verá que los cambios de los metadatos fluyen por el panel **Inspeccionar**. Si no hay un esquema definido en la transformación de origen, los metadatos no estarán visibles en el panel **Inspeccionar**. La falta de metadatos es habitual en escenarios de desviación en el esquema.

#### <a name="data-preview"></a>Vista previa de datos

Si el modo de depuración está activado, en la pestaña **Vista previa de los datos**, podrá ver una instantánea interactiva de los datos en cada transformación. Para más información, consulte [Vista previa de los datos en modo de depuración](../data-factory/concepts-data-flow-debug-mode.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#data-preview).

### <a name="top-bar"></a>Barra superior

La barra superior contiene acciones que afectan a todo el flujo de datos, como la configuración de validación y depuración. También puede ver el código JSON subyacente y el script de flujo de datos de la lógica de transformación.

## <a name="available-transformations"></a>Transformaciones disponibles

Vea [Introducción a las transformaciones en el flujo de datos de asignación](../data-factory/data-flow-transformation-overview.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) para obtener una lista de las transformaciones disponibles.

## <a name="data-flow-activity"></a>Actividad de los flujos de datos

Los flujos de datos se han operado dentro de las canalizaciones de Azure Synapse Analytics mediante la [actividad de flujo de datos](../data-factory/control-flow-execute-data-flow-activity.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json). Todo lo que tiene que hacer el usuario es especificar qué entorno de ejecución de integración usar y pasar valores de parámetro. Para obtener más información, vea el [entorno de ejecución de integración de Azure](../data-factory/concepts-integration-runtime.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#azure-integration-runtime).

## <a name="debug-mode"></a>Modo de depuración

El modo de depuración permite ver de forma interactiva los resultados de cada paso de transformación mientras compila y depura flujos de datos. La sesión de depuración se puede usar al compilar la lógica de flujo de datos y al ejecutar series de depuración de canalización con actividades de flujo de datos. Para obtener más información, vea la [documentación del modo de depuración](../data-factory/concepts-data-flow-debug-mode.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="monitoring-data-flows"></a>Supervisión de flujos de datos

El flujo de datos se integra con las funcionalidades de supervisión existentes de Azure Synapse Analytics. Para obtener información para comprender la salida de la supervisión de flujos de datos, vea [Supervisión de flujos de datos de asignación](../data-factory/concepts-data-flow-monitoring.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

El equipo de Azure Synapse Analytics ha creado una [Guía para la optimización del rendimiento](../data-factory/concepts-data-flow-performance.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) para ayudar a optimizar el tiempo de ejecución de los flujos de datos después de compilar la lógica de negocios.

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a crear una [transformación de origen](../data-factory/data-flow-source.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).
* Aprenda a crear flujos de datos en [modo de depuración](../data-factory/concepts-data-flow-debug-mode.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).
