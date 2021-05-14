---
title: Asignación de flujos de datos
description: Introducción a los flujos de datos de asignación en Azure Data Factory
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.custom: references_regions
ms.date: 04/14/2021
ms.openlocfilehash: 826183e09f2aa7f3f22ace8b5ce3e16767d49863
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107515668"
---
# <a name="mapping-data-flows-in-azure-data-factory"></a>Flujos de datos de asignación en Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

## <a name="what-are-mapping-data-flows"></a>¿Qué son los flujos de datos de asignación?

La asignación de flujos de datos es una transformación de datos diseñada visualmente en Azure Data Factory. Los flujos de datos permiten a los ingenieros de datos desarrollar lógica de transformación de datos sin necesidad de escribir código. Los flujos de datos resultantes se ejecutan como actividades en las canalizaciones de Azure Data Factory que usan clústeres de Apache Spark con escalabilidad horizontal. Las actividades de flujo de datos pueden ponerse en marcha mediante las capacidades de programación, control, flujo y supervisión existentes de Azure Data Factory.

Los flujos de datos de asignación proporcionan una experiencia completamente visual que no requiere programación. Los flujos de datos se ejecutan en clústeres de ejecución administrados por ADF durante el procesamiento de datos de escalabilidad horizontal. Asimismo, Azure Data Factory controla toda la traducción de código, la optimización de rutas de acceso y la ejecución de los trabajos de flujo de datos.

## <a name="getting-started"></a>Introducción

Los flujos de datos se crean desde el panel Factory Resources (Recursos de fábrica) como canalizaciones y conjuntos de datos. Para crear un flujo de datos, seleccione el signo más junto a **Factory Resources** (Recursos de fábrica) y luego **Data Flow**. 

![Nuevo flujo de datos](media/data-flow/new-data-flow.png)

Con esta acción, accederá al lienzo de flujo de datos, donde podrá crear la lógica de transformación. Seleccione **Agregar origen** para comenzar a configurar la transformación de origen. Para más información, consulte [Transformación de origen](data-flow-source.md).

## <a name="authoring-data-flows&quot;></a>Creación de flujos de datos

El flujo de datos de asignación tiene un lienzo de creación único diseñado para facilitar la creación de lógica de transformación. El lienzo de flujo de datos está dividido en tres partes: la barra superior, el gráfico y el panel de configuración. 

![Captura de pantalla que muestra el lienzo del flujo de datos con la barra superior, el gráfico y el panel de configuración marcados.](media/data-flow/canvas-1.png &quot;Lienzo")

### <a name="graph"></a>Grafo

En el gráfico se muestra el flujo de transformación. Muestra el linaje de los datos de origen a medida que fluyen hacia uno o varios receptores. Para agregar un nuevo origen, seleccione **Agregar origen**. Para agregar una nueva transformación, seleccione el signo más situado en la parte inferior derecha de una transformación existente. Obtenga más información sobre cómo [administrar el gráfico de flujo de datos](concepts-data-flow-manage-graph.md).

![Captura de pantalla que muestra la parte del gráfico del lienzo con un cuadro de texto de búsqueda.](media/data-flow/canvas-2.png)

### <a name="configuration-panel"></a>Panel de configuración

En el panel configuración se muestra la configuración específica de la transformación seleccionada actualmente. Si no se ha seleccionado ninguna transformación, se muestra el flujo de datos. En la configuración de flujo de datos general, puede agregar parámetros a través de la pestaña **Parámetros**. Para más información, consulte [Parámetros de asignación de Data Flow](parameters-data-flow.md).

Cada transformación contiene al menos cuatro pestañas de configuración.

#### <a name="transformation-settings"></a>Configuración de la transformación

La primera pestaña del panel de configuración de cada transformación contiene los valores específicos de esa transformación. Para más información, consulte la página de documentación de la transformación.

![Pestaña de configuración de origen](media/data-flow/source1.png "Pestaña de configuración de origen")

#### <a name="optimize"></a>Optimización

La pestaña **Optimizar** contiene valores opcionales para configurar los esquemas de partición. Para obtener más información sobre cómo optimizar los flujos de datos, consulte la [guía de rendimiento de flujos de datos de asignación](concepts-data-flow-performance.md).

![Captura de pantalla que muestra la pestaña Optimizar, que incluye la opción de partición, el tipo de partición y el número de particiones.](media/data-flow/optimize.png)

#### <a name="inspect&quot;></a>Inspeccionar

La pestaña **Inspeccionar** proporciona una vista de los metadatos del flujo de datos que se está transformando. Puede ver el número de columnas, las columnas que han cambiado, las columnas que se han agregado, los tipos de datos, el orden de las columnas y las referencias de las columnas. **Inspeccionar** es una vista de solo lectura de los metadatos. Para ver los metadatos en el panel **Inspeccionar**, no es preciso que el modo de depuración esté habilitado.

![Inspeccionar](media/data-flow/inspect1.png &quot;Inspeccionar")

Al cambiar la forma de los datos mediante transformaciones, verá que los cambios de los metadatos fluyen por el panel **Inspeccionar**. Si no hay un esquema definido en la transformación de origen, los metadatos no estarán visibles en el panel **Inspeccionar**. La falta de metadatos es habitual en escenarios de desviación en el esquema.

#### <a name="data-preview"></a>Vista previa de datos

Si el modo de depuración está activado, en la pestaña **Vista previa de los datos**, podrá ver una instantánea interactiva de los datos en cada transformación. Para más información, consulte [Vista previa de los datos en modo de depuración](concepts-data-flow-debug-mode.md#data-preview).

### <a name="top-bar"></a>Barra superior

La barra superior contiene acciones que afectan a todo el flujo de datos, como la operación de guardar y la validación. También puede ver el código JSON subyacente y el script de flujo de datos de la lógica de transformación. Para obtener más información, vea el [script de flujo de datos](data-flow-script.md).

## <a name="available-transformations"></a>Transformaciones disponibles

Vea [Introducción a las transformaciones en el flujo de datos de asignación](data-flow-transformation-overview.md) para obtener una lista de las transformaciones disponibles.

## <a name="data-flow-data-types"></a>Tipos de datos de flujo de datos

matriz, binario, booleano, complejo, decimal, fecha, float, entero, largo, asignación, corto, cadena, marca de tiempo

## <a name="data-flow-activity"></a>Actividad de los flujos de datos

Los flujos de datos de asignación se ponen en marcha en canalizaciones de ADF mediante la [actividad de flujo de datos](control-flow-execute-data-flow-activity.md). Todo lo que tiene que hacer el usuario es especificar qué entorno de ejecución de integración usar y pasar valores de parámetro. Para obtener más información, vea el [entorno de ejecución de integración de Azure](concepts-integration-runtime.md#azure-integration-runtime).

## <a name="debug-mode"></a>Modo de depuración

El modo de depuración permite ver de forma interactiva los resultados de cada paso de transformación mientras compila y depura flujos de datos. La sesión de depuración se puede usar al compilar la lógica de flujo de datos y al ejecutar series de depuración de canalización con actividades de flujo de datos. Para obtener más información, vea la [documentación del modo de depuración](concepts-data-flow-debug-mode.md).

## <a name="monitoring-data-flows"></a>Supervisión de flujos de datos

El flujo de datos de asignación se integra con las capacidades de supervisión existentes de Azure Data Factory. Para obtener información para comprender la salida de la supervisión de flujos de datos, vea [Supervisión de flujos de datos de asignación](concepts-data-flow-monitoring.md).

El equipo de Azure Data Factory ha creado una [Guía para la optimización del rendimiento](concepts-data-flow-performance.md) para ayudar a optimizar el tiempo de ejecución de los flujos de datos después de compilar la lógica de negocios.


## <a name="available-regions"></a>Regiones disponibles

Los flujos de datos de asignación están disponibles en las siguientes regiones de ADF:

| Región de Azure | Flujos de datos de ADF |
| ------------ | ----------------- |
| Centro de Australia | |
| Centro de Australia 2 | |
| Este de Australia | ✓ |
| Sudeste de Australia   | ✓ |
| Sur de Brasil  | ✓ |
| Centro de Canadá | ✓ |
| Centro de la India | ✓ |
| Centro de EE. UU.    | ✓ |
| Este de China |      |
| Este de China 2  |   |
| China no regional | |
| Norte de China | ✓ |
| Norte de China 2 | ✓ |
| Este de Asia | ✓ |
| Este de EE. UU.   | ✓ |
| Este de EE. UU. 2 | ✓ |
| Centro de Francia | ✓ |
| Sur de Francia  | |
| Centro de Alemania (soberana) | |
| Alemania no regional (soberana) | |
| Norte de Alemania (pública) | |
| Nordeste de Alemania (soberana) | |
| Centro-oeste de Alemania (soberana) |  |
| Japón Oriental | ✓ |
| Japón Occidental |  |
| Centro de Corea del Sur | ✓ |
| Corea del Sur | |
| Centro-Norte de EE. UU  | ✓ |
| Norte de Europa  | ✓ |
| Este de Noruega | ✓ |
| Oeste de Noruega | |
| Norte de Sudáfrica    | ✓ |
| Oeste de Sudáfrica |  |
| Centro-sur de EE. UU.  | |
| Sur de la India | |
| Sudeste de Asia    | ✓ |
| Norte de Suiza |   |
| Oeste de Suiza | |
| Centro de Emiratos Árabes Unidos | |
| Norte de Emiratos Árabes Unidos | ✓ |
| Sur de Reino Unido  | ✓ |
| Oeste de Reino Unido |     |
| US DoD (centro) | |
| US DoD (este) | |
| US Gov: Arizona | ✓ |
| US Gov no regional | |
| US Gov Texas | |
| US Gov - Virginia | ✓ |
| Centro-Oeste de EE. UU. |     |
| Oeste de Europa   | ✓ |
| Oeste de la India | |
| Oeste de EE. UU.   | ✓ |
| Oeste de EE. UU. 2 | ✓ |

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a crear una [transformación de origen](data-flow-source.md).
* Aprenda a crear flujos de datos en [modo de depuración](concepts-data-flow-debug-mode.md).
