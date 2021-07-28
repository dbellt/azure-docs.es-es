---
title: Información general del indexador
titleSuffix: Azure Cognitive Search
description: Rastree Azure SQL Database, SQL Managed Instance, Azure Cosmos DB o Azure Storage para extraer los datos utilizables en búsquedas y rellenar un índice de Azure Cognitive Search.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/29/2021
ms.openlocfilehash: ac9708e496fd0ee84d6e225ff8a63807bbe34fcd
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110468397"
---
# <a name="indexers-in-azure-cognitive-search"></a>Indexadores de Azure Cognitive Search

En Azure Cognitive Search, un *indexador* es un rastreador que extrae texto y metadatos que permiten búsquedas de un origen de datos de Azure externo y rellena un índice de búsqueda mediante asignaciones de campo a otro entre el origen de datos y el índice. Este enfoque se denomina a veces "modelo de extracción", porque el servicio extrae datos sin que sea preciso escribir código que agregue datos en un índice. Los indexadores también controlan las funcionalidades de [enriquecimiento con AI](cognitive-search-concept-intro.md) de Cognitive Search, lo que integra el procesamiento externo del contenido en ruta en un índice.

Los indexadores son solo de Azure, y hay indexadores individuales para [Azure SQL](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md), [Azure Cosmos DB](search-howto-index-cosmosdb.md) [Azure Table Storage](search-howto-indexing-azure-tables.md) y [Blob Storage](search-howto-indexing-azure-blob-storage.md). Al configurar un indexador, debe especificar un origen de datos (procedencia), así como un índice (destino). Varios orígenes, como Blob Storage, tienen propiedades de configuración adicionales específicas de ese tipo de contenido.

Puede ejecutar los indexadores a petición o con una programación de actualización periódica de datos que se ejecuta con una frecuencia de incluso cada cinco minutos. Las actualizaciones más frecuentes requieren un ["modelo de inserción"](search-what-is-data-import.md) que actualice simultáneamente los datos tanto en Azure Cognitive Search como en su origen de datos externo.

## <a name="usage-scenarios"></a>Escenarios de uso

Puede usar un indexador como único medio para la ingesta de datos o como parte de una combinación de técnicas que cargan y, opcionalmente, transforman o enriquecen el contenido a lo largo del proceso. En la tabla siguiente se resumen los principales escenarios.

| Escenario |Estrategia |
|----------|---------|
| Origen de datos único | Este patrón es el más simple: un origen de datos es el único proveedor de contenido de un índice de búsqueda. A partir del origen, identificará un campo que contenga valores únicos que sirvan como clave de documento del índice de búsqueda. El valor único se usará como identificador. Todos los demás campos de origen se asignan implícita o explícitamente a los campos correspondientes de un índice. </br></br>Una ventaja importante es que el valor de una clave de documento se origina a partir de los datos de origen. Un servicio de búsqueda no genera valores de clave. En las ejecuciones sucesivas, se agregan los documentos entrantes con nuevas claves, mientras que los documentos entrantes con claves existentes se combinan o se sobrescriben, en función de si los campos de índice son nulos o se han rellenado. |
| Varios orígenes de datos | Un índice puede aceptar contenido de varios orígenes, donde cada ejecución aporta nuevo contenido de un origen diferente. </br></br>Uno de los resultados puede ser un índice que obtenga documentos después de cada ejecución del indexador, de forma que se crean documentos completos íntegramente desde cada origen. Por ejemplo, los documentos del 1 al 100 son de Blob Storage, los documentos del 101 al 200 son de Azure SQL, etc. El reto de este escenario radica en diseñar un esquema de índice que funcione con todos los datos entrantes y una estructura de clave de documento que sea uniforme en el índice de búsqueda. De forma nativa, los valores que identifican de forma única un documento son metadata_storage_path en un contenedor de blobs y una clave principal en una tabla de SQL. Puede suponer que uno o ambos orígenes se deben modificar para proporcionar valores de clave en un formato común, sin importar el origen del contenido. En este escenario, cabría esperar realizar algún nivel de procesamiento previo para homogeneizar los datos de forma que se puedan extraer en un solo índice. </br></br>Un resultado alternativo podrían ser los documentos de búsqueda que se rellenan parcialmente en la primera ejecución y, después, adicionalmente con las sucesivas ejecuciones para incorporar valores de otros orígenes. Por ejemplo, los campos del 1 al 10 son de Blob Storage, del 11 al 20 de Azure SQL, etc. El reto de este patrón es asegurarse de que cada ejecución de indexación tenga como destino el mismo documento. La combinación de campos en un documento existente requiere una coincidencia en la clave de documento. Para ver una demostración de este escenario, consulte el [Tutorial: Indexación de varios orígenes de datos](tutorial-multiple-data-sources.md). |
| Varios indexadores | Si utiliza varios orígenes de datos, es posible que también necesite varios indexadores si necesita variar los parámetros del tiempo de ejecución, la programación o las asignaciones de campos. Aunque varios conjuntos de orígenes de datos de indexador pueden tener como destino el mismo índice, tenga cuidado con las ejecuciones del indexador que puedan sobrescribir los valores existentes en el índice. Si un segundo origen de datos de indexador tiene como destino los mismos documentos y campos, se sobrescribirán los valores de la primera ejecución. Los valores de campo se reemplazan por completo; un indexador no puede combinar valores de varias ejecuciones en el mismo campo.</br></br>Otro caso de uso de varios indexadores es el [escalado horizontal entre regiones de Cognitive Search](search-performance-optimization.md#data-sync). Es posible que tenga copias del mismo índice de búsqueda en diferentes regiones. Para sincronizar el contenido del índice de búsqueda, puede tener varios indexadores que extraigan datos del mismo origen, donde cada indexador tiene como destino un índice de búsqueda diferente.</br></br>La [indexación en paralelo](search-howto-large-index.md#parallel-indexing) de conjuntos de datos muy grandes también requiere una estrategia de varios indexadores. Cada indexador tiene como destino un subconjunto de los datos. |
| Transformación de contenido | Cognitive Search admite comportamientos de [enriquecimiento con IA](cognitive-search-concept-intro.md) que agregan análisis de imágenes y procesamiento del lenguaje natural para crear contenidos y estructuras de búsqueda. El enriquecimiento con IA está controlado por indexador, mediante un [conjunto de aptitudes](cognitive-search-working-with-skillsets.md) asociado. Para realizar el enriquecimiento con IA, el indexador necesita un índice y un origen de datos de Azure; sin embargo, en este escenario, se agrega el procesamiento del conjunto de aptitudes a la ejecución del indexador. |

<a name="supported-data-sources"></a>

## <a name="supported-data-sources"></a>Orígenes de datos admitidos

Los indexadores rastrean los almacenes de datos en Azure y fuera de Azure.

+ [Amazon Redshift](search-how-to-index-power-query-data-sources.md) (en versión preliminar)
+ [Azure Blob Storage](search-howto-indexing-azure-blob-storage.md)
+ [Azure Cosmos DB](search-howto-index-cosmosdb.md)
+ [Azure Data Lake Storage Gen2](search-howto-index-azure-data-lake-storage.md)
+ [Azure MySQL](search-howto-index-mysql.md) (en versión preliminar)
+ [Azure SQL Database](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)
+ [Azure Table Storage](search-howto-indexing-azure-tables.md)
+ [Elasticsearch](search-how-to-index-power-query-data-sources.md) (en versión preliminar)
+ [PostgreSQL](search-how-to-index-power-query-data-sources.md) (en versión preliminar)
+ [Objectos de Salesforce](search-how-to-index-power-query-data-sources.md) (en versión preliminar)
+ [Informes de Salesforce](search-how-to-index-power-query-data-sources.md) (en versión preliminar)
+ [Smartsheet](search-how-to-index-power-query-data-sources.md) (en versión preliminar)
+ [Snowflake](search-how-to-index-power-query-data-sources.md) (en versión preliminar)
+ [Instancia administrada de SQL](search-howto-connecting-azure-sql-mi-to-azure-search-using-indexers.md)
+ [SQL Server en Azure Virtual Machines](search-howto-connecting-azure-sql-iaas-to-azure-search-using-indexers.md)

Las conexiones de indexador a orígenes de datos remotos se pueden realizar mediante conexiones de Internet estándar (públicas) o conexiones privadas cifradas, cuando se usan redes virtuales de Azure para las aplicaciones cliente. También puede configurar conexiones para autenticarse mediante una identidad de servicio de confianza. Para más información sobre las conexiones seguras, consulte [Concesión de acceso a través de puntos de conexión privados](search-indexer-securing-resources.md#granting-access-via-private-endpoints) y [Configuración de una conexión de indexador a un origen de datos mediante una identidad administrada](search-howto-managed-identities-data-sources.md).

## <a name="stages-of-indexing"></a>Fases de la indexación

En una ejecución inicial, cuando el índice está vacío, un indexador lee todos los datos proporcionados en la tabla o el contenedor. En las ejecuciones posteriores, el indexador normalmente puede detectar y recuperar solo los datos que han cambiado. En el caso de los datos de blob, la detección de cambios es automática. En otros orígenes de datos, como Azure SQL o Cosmos DB, la detección de cambios se debe habilitar.

En todos los documentos que recibe, un indexador implementa o coordina varios pasos, desde la recuperación del documento hasta una "entrega" final del motor de búsqueda para la indexación. Opcionalmente, un indexador también es fundamental para impulsar la ejecución y los resultados del conjunto de aptitudes, siempre que se haya definido uno.

:::image type="content" source="media/search-indexer-overview/indexer-stages.png" alt-text="Fases del indexador" border="false":::

### <a name="stage-1-document-cracking"></a>Fase 1 Descifrado de documentos

El descifrado de documentos es el proceso de abrir archivos y extraer contenido. Según el tipo de origen de datos, el indexador intenta realizar diferentes operaciones para extraer contenido que posiblemente se pueda indexar.  

Ejemplos:  

+ Si el documento es un registro de un [origen de datos de Azure SQL](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md), el indexador extrae cada uno de los campos del registro.
+ Si el documento es un archivo PDF de un [origen de datos de Azure Blob Storage](search-howto-indexing-azure-blob-storage.md), el indexador extrae el texto, las imágenes y los metadatos.
+ Si el documento es un registro de un [origen de datos de Cosmos DB](search-howto-index-cosmosdb.md), el indexador extrae los campos y subcampos del documento de Cosmos DB.

### <a name="stage-2-field-mappings"></a>Fase 2: Asignaciones de campos 

Un indexador extrae texto de un campo de origen y lo envía a un campo de destino de un índice o un almacén de conocimiento. Si los nombres de los campos y los tipos coinciden, la ruta de acceso está clara. Pero es posible que quiera nombres o tipos diferentes en el resultado, en cuyo caso debe indicar al indexador cómo asignar el campo. 

Este paso se produce después del descifrado de documentos, pero antes de las transformaciones, cuando el indexador lee los documentos de origen. Al definir una [asignación de campos](search-indexer-field-mappings.md), el valor del campo de origen se envía tal cual al campo de destino, sin modificaciones. 

### <a name="stage-3-skillset-execution"></a>Fase 3: Ejecución del conjunto de aptitudes

La ejecución del conjunto de aptitudes es un paso opcional que invoca al procesamiento de IA integrado o personalizado. Es posible que lo necesite para el reconocimiento óptico de caracteres (OCR) en forma de análisis de imágenes, si los datos de origen es una imagen binaria, o puede que necesite traducción de idiomas, si el contenido está en varios idiomas. 

Sea cual sea la transformación, la ejecución del conjunto de aptitudes es donde se produce el enriquecimiento. Si un indexador es una canalización, puede imaginarse un [conjunto de aptitudes](cognitive-search-defining-skillset.md) como una "canalización dentro de la canalización".

### <a name="stage-4-output-field-mappings"></a>Fase 4: Asignaciones de campos de salida

Si incluye un conjunto de aptitudes, lo más probable es que tenga que incluir asignaciones de campos de salida. El resultado de un conjunto de aptitudes es en realidad un árbol de información denominado *documento enriquecido*. Las asignaciones de campos de salida permiten seleccionar qué partes de este árbol se asignan a campos del índice. Aprenda a [definir asignaciones de campos de salida](cognitive-search-output-field-mapping.md).

Mientras que las asignaciones de campos asocian valores textuales de los datos de origen a los campos de destino, las asignaciones de campos de salida indican al indexador cómo asociar los valores transformados del documento enriquecido a los campos de destino del índice. A diferencia de las asignaciones de campos, que se consideran opcionales, siempre debe definir una asignación de campos de salida para cualquier contenido transformado que deba residir en un índice.

En la siguiente imagen se muestra una representación de [sesión de depuración](cognitive-search-debug-session.md) de un indexador de ejemplo de las fases del indexador: descifrado de documentos, asignaciones de campos, ejecución del conjunto de aptitudes y asignaciones de campos de salida.

:::image type="content" source="media/search-indexer-overview/sample-debug-session.png" alt-text="Sesión de depuración de ejemplo" lightbox="media/search-indexer-overview/sample-debug-session.png":::

## <a name="basic-workflow"></a>Flujo de trabajo básico

Los indexadores pueden ofrecer características que son exclusivas del origen de datos. En este sentido, algunos aspectos de la configuración de orígenes de datos o indexadores varían según el tipo de indexador. No obstante, todos los indexadores comparten composición básica y requisitos. Más adelante, se explican los pasos que son comunes a todos los indexadores.

### <a name="step-1-create-a-data-source"></a>Paso 1: Creación de un origen de datos

Los indexadores requieren un objeto *origen de datos* que proporcione una cadena de conexión y, posiblemente, credenciales. Para crear el recurso, llame a [Create Data Source (REST)](/rest/api/searchservice/create-data-source) o a la [clase SearchIndexerDataSourceConnection](/dotnet/api/azure.search.documents.indexes.models.searchindexerdatasourceconnection).

Los orígenes de datos se configuran y administran independientemente de los indexadores que los usan, lo que significa que varios indexadores pueden usar un origen de datos para cargar más de un índice a la vez.

### <a name="step-2-create-an-index"></a>Paso 2: Creación de un índice

Un indexador automatizará algunas tareas relacionadas con la ingesta de datos, pero la creación de un índice no suele ser una de ellas. Como requisito previo, debe tener un índice predefinido con campos que coincidan con los del origen de datos externo. Los campos deben coincidir por nombre y tipo de datos. Si no es así, puede [definir asignaciones de campos](search-indexer-field-mappings.md) para establecer la asociación. Para más información sobre cómo estructurar índices, consulte [Create an Index (REST)](/rest/api/searchservice/Create-Index) o a la [clase SearchIndex ](/dotnet/api/azure.search.documents.indexes.models.searchindex).

> [!Tip]
> Aunque los indexadores no pueden generar un índice, el Asistente para **importar datos** del portal puede ayudarle. En la mayoría de los casos, el asistente puede inferir un esquema de índice de los metadatos existentes en el origen y presentar un esquema de índice preliminar que se pueda modificar en línea mientras el asistente esté activo. Una vez que el índice se crea en el servicio, la mayoría de las posteriores modificaciones se limitan a agregar nuevos campos. Tenga en cuenta al asistente para crear índices, pero no para revisarlos. Para obtener conocimientos prácticos, recorra el [tutorial del portal](search-get-started-portal.md).

### <a name="step-3-create-and-run-or-schedule-the-indexer"></a>Paso 3: Creación y ejecución (o programación) del indexador

Un indexador se ejecuta la primera vez que se [crea un indexador](/rest/api/searchservice/Create-Indexer) en el servicio de búsqueda. Hasta que se crea o ejecuta el indexador no se sabe si se puede acceder al origen de datos es accesible o si el conjunto de aptitudes es válido. Después de la primera ejecución, se puede volver a ejecutar a petición mediante [Run Indexer](/rest/api/searchservice/run-indexer), o bien se puede [definir una programación periódica](search-howto-schedule-indexers.md). 

El estado del indexador se puede supervisar en el portal o mediante [Get Indexer Status API](/rest/api/searchservice/get-indexer-status). También hay que [ejecutar consultas en el índice](search-query-create.md) para comprobar que el resultado es el esperado.

## <a name="next-steps"></a>Pasos siguientes

Tras esta introducción, el siguiente paso es revisar las propiedades y los parámetros del indexador, su programación y su supervisión. Como alternativa, puede volver a la lista de [orígenes de datos admitidos](#supported-data-sources), donde podrá obtener más información acerca de cualquier origen concreto.

+ [Creación de indexadores](search-howto-create-indexers.md)
+ [Restablecimiento y ejecución de indexadores](search-howto-run-reset-indexers.md)
+ [Programación de indexadores](search-howto-schedule-indexers.md)
+ [Definición de asignaciones de campos](search-indexer-field-mappings.md)
+ [Supervisión del estado del indexador](search-howto-monitor-indexers.md)