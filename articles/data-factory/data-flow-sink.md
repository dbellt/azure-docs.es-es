---
title: Transformación de receptor en el flujo de datos de asignación
description: Aprenda a configurar una transformación de receptor en el flujo de datos de asignación.
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 04/06/2021
ms.openlocfilehash: 8970fc2d5f11d106f34cf2f4c05a1c61bdc6c438
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107107421"
---
# <a name="sink-transformation-in-mapping-data-flow"></a>Transformación de receptor en el flujo de datos de asignación

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cuando termine de transformar los datos, escríbalos en un almacén de destino mediante la transformación del receptor. Cada flujo de datos requiere al menos una transformación de receptor, pero puede agregar tantos receptores como sea necesario para completar el flujo de transformación. Para escribir en receptores adicionales, cree nuevas secuencias a través de nuevas ramas y divisiones condicionales.

Cada transformación del receptor se asocia exactamente con un servicio vinculado o un objeto de conjunto de objetos de Azure Data Factory. La transformación del receptor determina la forma y la ubicación de los datos en los que se desea escribir.

## <a name="inline-datasets"></a>Conjuntos de datos insertado

Al crear una transformación de receptor, elija si la información del receptor se define dentro de un objeto de conjunto de datos o en la transformación. La mayoría de los formatos están disponibles solo en una opción o en la otra. Para obtener información sobre cómo usar un conector específico, consulte el documento adecuado del conector.

Cuando un formato se admita tanto en la opción en línea como en un objeto de conjunto de datos, existen ventajas para ambos. Los objetos de conjunto de datos son entidades reutilizables que se pueden usar en otros flujos de datos y actividades, como en la copia. Estas entidades reutilizables son especialmente útiles cuando se usa un esquema protegido. Los conjuntos de datos no se basan en Spark. En ocasiones, es posible que necesite reemplazar determinados valores o la proyección del esquema en la transformación del receptor.

Se recomiendan los conjuntos de datos insertados cuando se usan esquemas flexibles, instancias de receptor único u receptores con parámetros. Si el receptor contiene muchos parámetros, los conjuntos de datos insertados permiten no crear un objeto "ficticio". Los conjuntos de datos insertados se basan en Spark y sus propiedades son nativas para el flujo de datos.

Para usar un conjunto de datos insertado, seleccione el formato que desee en el selector **Tipo de receptor**. En lugar de seleccionar un conjunto de datos de receptor, seleccione el servicio vinculado al que desee conectarse.

![Captura de pantalla que muestra la opción Insertado seleccionada.](media/data-flow/inline-selector.png "Captura de pantalla que muestra la opción Insertado seleccionada.")

##  <a name="supported-sink-types"></a><a name="supported-sinks"></a> Tipos de receptores admitidos

El flujo de datos de asignación sigue un enfoque de extracción, carga y transformación (ELT) y funciona con conjuntos de datos de un *almacenamiento provisional* que están todos en Azure. Actualmente, se pueden usar los siguientes conjuntos de datos en una transformación de origen.

| Conector | Formato | Conjunto de datos/insertado |
| --------- | ------ | -------------- |
| [Azure Blob Storage](connector-azure-blob-storage.md#mapping-data-flow-properties) | [Avro](format-avro.md#mapping-data-flow-properties) <br>[Texto delimitado](format-delimited-text.md#mapping-data-flow-properties) <br>[Delta](format-delta.md) <br>[JSON](format-json.md#mapping-data-flow-properties) <br/>[ORC](format-orc.md#mapping-data-flow-properties)<br>[Parquet](format-parquet.md#mapping-data-flow-properties) | ✓/- <br>✓/- <br>-/✓ <br>✓/- <br>✓/✓<br>✓/- |
| [Azure Cosmos DB (API de SQL)](connector-azure-cosmos-db.md#mapping-data-flow-properties) | | ✓/- |
| [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md#mapping-data-flow-properties) | [Avro](format-avro.md#mapping-data-flow-properties) <br>[Texto delimitado](format-delimited-text.md#mapping-data-flow-properties) <br>[JSON](format-json.md#mapping-data-flow-properties) <br/>[ORC](format-orc.md#mapping-data-flow-properties)<br/>[Parquet](format-parquet.md#mapping-data-flow-properties) | ✓/- <br>✓/- <br>✓/- <br>✓/✓<br>✓/- |
| [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#mapping-data-flow-properties) | [Avro](format-avro.md#mapping-data-flow-properties) <br/>[Common Data Model](format-common-data-model.md#sink-properties)<br>[Texto delimitado](format-delimited-text.md#mapping-data-flow-properties) <br>[Delta](format-delta.md) <br>[JSON](format-json.md#mapping-data-flow-properties) <br/>[ORC](format-orc.md#mapping-data-flow-properties)<br/>[Parquet](format-parquet.md#mapping-data-flow-properties) | ✓/- <br>-/✓ <br>✓/- <br>-/✓ <br>✓/-<br>✓/✓ <br>✓/- |
| [Azure Database for MySQL](connector-azure-database-for-mysql.md) |  | ✓/✓ |
| [Azure Database para PostgreSQL](connector-azure-database-for-postgresql.md) |  | ✓/✓ |
| [Azure SQL Database](connector-azure-sql-database.md#mapping-data-flow-properties) | | ✓/- |
| [Azure SQL Managed Instance (versión preliminar)](connector-azure-sql-managed-instance.md#mapping-data-flow-properties) | | ✓/- |
| [Azure Synapse Analytics](connector-azure-sql-data-warehouse.md#mapping-data-flow-properties) | | ✓/- |
| [Snowflake](connector-snowflake.md) | | ✓/✓ |

La configuración específica de estos conectores se encuentra en la pestaña **Configuración**. La información y algunos ejemplos de script de flujo de datos sobre esta configuración se encuentran en la documentación del conector.

Azure Data Factory tiene acceso a más de [90 conectores nativos](connector-overview.md). Para escribir datos en esos otros orígenes desde el flujo de datos, use la actividad de copia para cargar los datos desde un receptor compatible.

## <a name="sink-settings"></a>Configuración del receptor

Una vez que haya agregado un receptor, configúrelo a través de la pestaña **Receptor**. Aquí puede elegir o crear el conjunto de datos en el que escribe el receptor. Los valores de desarrollo de los parámetros del conjunto de datos se pueden definir en la [configuración de depuración](concepts-data-flow-debug-mode.md). (Requiere que esté activado el modo Depuración).

En el siguiente vídeo se explican varias opciones de receptor diferentes para los tipos de archivo delimitados de texto.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4tf7T]

![Captura de pantalla que muestra la configuración del receptor.](media/data-flow/sink-settings.png "Captura de pantalla que muestra la configuración del receptor.")

**Desfase de esquema**: el [desfase de esquema](concepts-data-flow-schema-drift.md) es la capacidad de Data Factory de administrar de forma nativa los esquemas flexibles en los flujos de datos sin necesidad de definir explícitamente los cambios en las columnas. Habilite **Permitir el desfase de esquema** para escribir columnas adicionales sobre lo que se define en el esquema de datos del receptor.

**Validar esquema**: Si se selecciona que se valide el esquema, se producirá un error en el flujo de datos si no se encuentra ninguna columna del esquema de origen entrante en la proyección de origen o si los tipos de datos no coinciden. Use esta opción para exigir que los datos de origen cumplan el contrato de la proyección definida. Es útil en escenarios de origen de base de datos para indicar que los nombres o los tipos de columna han cambiado.

## <a name="cache-sink"></a>Receptor de caché

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4HKt1]

Un *receptor de caché* se utiliza cuando un flujo de datos escribe datos en la memoria caché de Spark en lugar de en un almacén de datos. En los flujos de datos de asignación, puede hacer referencia a estos datos en el mismo flujo muchas veces con una *búsqueda en caché*. Esto resulta útil si desea hacer referencia a los datos como parte de una expresión, pero no desea unir explícitamente las columnas. Algunos ejemplos comunes en los que un receptor de caché puede ayudar son: buscar un valor máximo en un almacén de datos y buscar coincidencias de códigos de error en una base de datos de mensajes de error. 

Para escribir en un receptor de caché, agregue una transformación de receptor y seleccione **Caché** como el tipo de receptor. A diferencia de otros tipos de receptor, no es necesario seleccionar un conjunto de datos ni un servicio vinculado porque no está escribiendo en un almacén externo. 

![Selección del receptor de caché](media/data-flow/select-cache-sink.png "Selección del receptor de caché")

En la configuración del receptor, tiene la opción de especificar las columnas de clave del receptor de caché. Se usan como condiciones de coincidencia cuando se utiliza la función `lookup()` en una búsqueda en caché. Si especifica columnas de clave, no puede usar la función `outputs()` en una búsqueda en caché. Para obtener más información sobre la sintaxis de búsqueda de caché, consulte [búsquedas en caché](concepts-data-flow-expression-builder.md#cached-lookup).

![Columnas de clave del receptor de caché](media/data-flow/cache-sink-key-columns.png "Columnas de clave del receptor de caché")

Por ejemplo, si se especifica una columna de clave única de `column1` en un receptor de caché denominado `cacheExample`, la llamada a `cacheExample#lookup()` tendría un parámetro para especificar con qué fila del receptor de caché debe coincidir. La función genera una única columna compleja con subcolumnas para cada columna asignada.

> [!NOTE]
> Un receptor de caché debe estar en un flujo de datos completamente independiente de cualquier transformación que haga referencia a este a través de una búsqueda en caché. Un receptor de caché también debe ser el primer receptor escrito. 

## <a name="field-mapping"></a>Asignación de campos

De forma similar a la transformación Selección, en la pestaña **Asignación** del receptor, puede decidir qué columnas de entrada se escribirán. De forma predeterminada, se asignan todas las columnas de entrada, incluidas las columnas desfasadas. Este comportamiento se conoce como *asignación automática*.

Cuando desactive la asignación automática, puede agregar asignaciones basadas en columnas o basadas en reglas. Con las asignaciones basadas en reglas, puede escribir expresiones con coincidencia de patrones. La asignación fija asigna nombres de columnas lógicas y físicas. Para más información sobre la asignación basada en reglas, consulte [Patrones de columna en flujos de datos de asignación](concepts-data-flow-column-pattern.md#rule-based-mapping-in-select-and-sink).

## <a name="custom-sink-ordering"></a>Ordenación de receptores personalizados

De forma predeterminada, los datos se escriben en varios receptores en un orden no determinista. El motor de ejecución escribe datos en paralelo a medida que se complete la lógica de transformación y el orden de los receptores puede variar en cada ejecución. Para especificar una ordenación de receptores exacta, habilite la opción **Ordenación de receptores personalizada** en la pestaña **General** del flujo de datos. Una vez habilitada, los receptores se escriben secuencialmente en orden ascendente.

![Captura de pantalla que muestra la ordenación del receptor personalizado.](media/data-flow/custom-sink-ordering.png "Captura de pantalla que muestra la ordenación del receptor personalizado.")

> [!NOTE]
> Al usar las [búsquedas almacenadas en caché](./concepts-data-flow-expression-builder.md#cached-lookup), asegúrese de que la ordenación del receptor tenga los receptores almacenado en caché establecidos en 1, el calor más bajo (o el primer valor) del orden.

![Ordenación de receptores personalizada](media/data-flow/cache-2.png "Ordenación de receptores personalizados")

### <a name="sink-groups"></a>Grupos de receptores

Si desea agrupar receptores, puede aplicar el mismo número de pedido para una serie de receptores. ADF tratará esos receptores como grupos que se pueden ejecutar en paralelo. Las opciones para la ejecución en paralelo se mostrarán en la actividad de flujo de datos de la canalización.

## <a name="error-row-handling"></a>Control de filas de errores

Al escribir en bases de datos, puede producirse un error en determinadas filas de datos debido a las restricciones establecidas por el destino. De forma predeterminada, la ejecución de un flujo de datos no funcionará al recibir el primer error. En algunos conectores, puede optar por **Continuar en caso de error**, que permite que el flujo de datos se complete, aunque haya filas individuales con errores. Actualmente, esta funcionalidad solo está disponible en Azure SQL Database. Para más información, consulte [Control de filas de error en Azure SQL DB](connector-azure-sql-database.md#error-row-handling).

A continuación, se muestra un tutorial en vídeo sobre cómo usar el control de filas de error de base de datos automáticamente en la transformación del receptor.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4IWne]

## <a name="data-preview-in-sink"></a>Vista previa de los datos en el receptor

Al obtener una vista previa de los datos en modo de depuración, no se escribirá ningún dato en el receptor. Se devolverá una instantánea de la apariencia de los datos, pero no se escribirá nada en el destino. Para probar la escritura de datos en el receptor, ejecute una depuración de canalización desde el lienzo de canalización.

## <a name="data-flow-script"></a>Script de flujo de datos

### <a name="example"></a>Ejemplo

A continuación se muestra un ejemplo de una transformación de receptor con el script de flujo de datos correspondiente:

```
sink(input(
        movie as integer,
        title as string,
        genres as string,
        year as integer,
        Rating as integer
    ),
    allowSchemaDrift: true,
    validateSchema: false,
    deletable:false,
    insertable:false,
    updateable:true,
    upsertable:false,
    keys:['movie'],
    format: 'table',
    skipDuplicateMapInputs: true,
    skipDuplicateMapOutputs: true,
    saveOrder: 1,
    errorHandlingOption: 'stopOnFirstError') ~> sink1
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha creado el flujo de datos, agregue una [actividad de Data Flow a la canalización](concepts-data-flow-overview.md).
