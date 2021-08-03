---
title: Conexión e indexación de contenido de Azure MySQL con un indexador de Azure Cognitive Search (versión preliminar)
titleSuffix: Azure Cognitive Search
description: Importe datos de Azure MySQL a un índice que permita búsquedas en Azure Cognitive Search. Los indexadores automatizan la ingesta de datos para orígenes de datos seleccionados, como MySQL.
author: markheff
manager: luisca
ms.author: maheff
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 05/17/2021
ms.openlocfilehash: 4dee2250d49d437d47148b873cfe8c7ce1e8f5ea
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111754446"
---
# <a name="connect-to-and-index-azure-mysql-content-using-an-azure-cognitive-search-indexer-preview"></a>Conexión e indexación de contenido de Azure MySQL con un indexador de Azure Cognitive Search (versión preliminar)

> [!IMPORTANT] 
> La compatibilidad con MySQL se encuentra actualmente en versión preliminar pública. La funcionalidad de versión preliminar se ofrece sin un Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Para solicitar acceso a las versiones preliminares, rellene [este formulario](https://aka.ms/azure-cognitive-search/indexer-preview). En la [API de REST, versión 2020-06-30-Preview](search-api-preview.md) se proporciona esta característica. Actualmente no hay compatibilidad con el SDK ni con el portal.

El indexador de Azure Cognitive Search para MySQL rastreará la base de datos MySQL en Azure, extraerá los datos en los que se pueden realizar búsquedas y los indexará en Azure Cognitive Search. El indexador realizará todos los cambios, cargas y eliminaciones de la base de datos MySQL y reflejará estos cambios en Azure Cognitive Search.

## <a name="create-an-azure-mysql-indexer"></a>Creación de un indexador de Azure MySQL

Para indexar MySQL en Azure, siga estos pasos.

### <a name="step-1-create-a-data-source"></a>Paso 1: Creación de un origen de datos

Para crear el origen de datos, envíe la siguiente solicitud:

```http

    POST https://[search service name].search.windows.net/datasources?api-version=2020-06-30-Preview
    Content-Type: application/json
    api-key: [admin key]
    
    {   
        "name" : "[Data source name]"
        "description" : "[Description of MySQL data source]",
        "type" : "mysql",
        "credentials" : { 
            "connectionString" : 
                "Server=[MySQLServerName].MySQL.database.azure.com; Port=3306; Database=[DatabaseName]; Uid=[UserName]; Pwd=[Password]; SslMode=Preferred;" 
        },
        "container" : { 
            "name" : "[TableName]" 
        },
        "dataChangeDetectionPolicy" : { 
            "@odata.type": "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
            "highWaterMarkColumnName": "[HighWaterMarkColumn]"
        }
    }

```

### <a name="step-2-create-an-index"></a>Paso 2: Creación de un índice

Si aún no tiene un índice de Búsqueda cognitiva de Azure de destino, créelo.

```http

    POST https://[service name].search.windows.net/indexes?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
       "name": "[Index name]",
       "fields": [{
         "name": "id",
         "type": "Edm.String",
         "key": true,
         "searchable": false
       }, {
         "name": "description",
         "type": "Edm.String",
         "filterable": false,
         "searchable": true,
         "sortable": false,
         "facetable": false
       }]
    }

```

### <a name="step-3-create-the-indexer"></a>Paso 3: Creación del indexador

Una vez creados el índice y el origen de datos, ya podrá crear el indexador.

```http

    POST https://[search service name].search.windows.net/indexers?api-version=2020-06-30-Preview
    Content-Type: application/json
    api-key: [admin key]
    
    {
        "name" : "[Indexer name]"
        "description" : "[Description of MySQL indexer]",
        "dataSourceName" : "[Data source name]",
        "targetIndexName" : "[Index name]"
    }

```

## <a name="run-indexers-on-a-schedule"></a>Ejecutar indexadores según una programación
También puede disponer que el indizador se ejecute periódicamente según una programación. Para ello, agregue la propiedad **schedule** al crear o actualizar el indexador. El ejemplo siguiente muestra una solicitud PUT para actualizar el indizador:

```http
    PUT https://[search service name].search.windows.net/indexers/[Indexer name]?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin-key]

    {
        "dataSourceName" : "[Data source name]",
        "targetIndexName" : "[Index name]",
        "schedule" : { 
            "interval" : "PT10M", 
            "startTime" : "2021-01-01T00:00:00Z"
        }
    }
```

El parámetro **interval** es obligatorio. El intervalo se refiere al tiempo entre el inicio de dos ejecuciones consecutivas de indizador. El intervalo mínimo permitido es de 5 minutos y el máximo de un día. Debe tener el formato de un valor "dayTimeDuration" XSD (subconjunto restringido de un valor de [duración ISO 8601](https://www.w3.org/TR/xmlschema11-2/#dayTimeDuration) ). El patrón de este es: `P(nD)(T(nH)(nM))`. Ejemplos: `PT15M` para cada 15 minutos, `PT2H` para cada 2 horas.

Para más información sobre cómo definir las programaciones del indexador, consulte [Programación de indexadores para Azure Cognitive Search](search-howto-schedule-indexers.md).

## <a name="capture-new-changed-and-deleted-rows"></a>Capturar filas nuevas, cambiadas y eliminadas

Búsqueda cognitiva de Azure usa la **indexación incremental** para evitar tener que volver a indexar la tabla o vista completa cada vez que se ejecuta un indexador.

<a name="HighWaterMarkPolicy"></a>

### <a name="high-water-mark-change-detection-policy"></a>Directiva de detección de cambios de límite superior

Esta directiva de detección de cambios se basa en una columna de "marca de límite superior" que captura la versión o la hora en que se actualizó por última vez una fila. Si usa una vista, debe usar una directiva de marca de límite superior. La columna de marca de límite superior debe cumplir los siguientes requisitos.

#### <a name="requirements"></a>Requisitos 

* Todas las inserciones especifican un valor para la columna.
* Todas las actualizaciones de un elemento también cambian el valor de la columna.
* El valor de esta columna aumenta con cada inserción o actualización.
* Las consultas con las cláusulas WHERE y ORDER BY siguientes se pueden ejecutar de forma eficaz: `WHERE [High Water Mark Column] > [Current High Water Mark Value] ORDER BY [High Water Mark Column]`.

#### <a name="usage"></a>Uso

Para usar una directiva de marca de límite superior, cree o actualice el origen de datos de la siguiente manera:

```http
    {
        "name" : "[Data source name]",
        "type" : "mysql",
        "credentials" : { "connectionString" : "[connection string]" },
        "container" : { "name" : "[table or view name]" },
        "dataChangeDetectionPolicy" : {
           "@odata.type" : "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
           "highWaterMarkColumnName" : "[last_updated column name]"
      }
    }
```

> [!WARNING]
> Si la tabla de origen no tiene un índice en la columna de límite superior, las consultas utilizadas por el indexador MySQL pueden agotar el tiempo de espera. En concreto, la cláusula `ORDER BY [High Water Mark Column]` requiere un índice para ejecutarse de forma eficaz cuando la tabla contiene muchas filas.

### <a name="soft-delete-column-deletion-detection-policy"></a>Directiva de detección de eliminación de columna de eliminación temporal
Cuando se eliminan filas de la tabla de origen, probablemente le interesará eliminar dichas filas del índice de búsqueda. Si las filas se quitaron físicamente de la tabla, Búsqueda cognitiva de Azure no tiene forma de deducir la presencia de registros que ya no existen.  Sin embargo, puede utilizar la técnica de "eliminación temporal" para eliminar filas lógicamente sin quitarlas de la tabla. Agregue una columna a la tabla o vista y marque filas como eliminadas mediante esa columna.

Cuando use la técnica de la eliminación temporal, puede especificar la directiva de eliminación temporal de la manera que se indica a continuación al crear o actualizar el origen de datos:

```http
    {
        …,
        "dataDeletionDetectionPolicy" : {
           "@odata.type" : "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
           "softDeleteColumnName" : "[a column name]",
           "softDeleteMarkerValue" : "[the value that indicates that a row is deleted]"
        }
    }
```

**softDeleteMarkerValue** debe ser una cadena. Use la representación de cadena del valor real. Por ejemplo, si tiene una columna de enteros donde las filas eliminadas se marcan con el valor 1, use `"1"`. Si tiene una columna BIT donde las filas eliminadas se marcan con el valor booleano true, use el literal de cadena `True` o `true`, no importan las mayúsculas y minúsculas.

<a name="TypeMapping"></a>

## <a name="mapping-between-mysql-and-azure-cognitive-search-data-types"></a>Asignación entre tipos de datos de MySQL y Azure Cognitive Search

> [!NOTE]
> La versión preliminar no admite tipos de geometría ni blobs.

| Tipo de datos de MySQL | Tipos de campos de índice de destino permitidos |
| --- | --- |
| bool, boolean | Edm.Boolean, Edm.String |
| tinyint, smallint, mediumint, int, integer, year | Edm.Int32, Edm.Int64, Edm.String |
| bigint | Edm.Int64, Edm.String |
| float, double, real | Edm.Double, Edm.String |
| date, datetime, timestamp | Edm.DateTimeOffset, Edm.String |
| char, varchar, tinytext, mediumtext, text, longtext, enum, set, time | Edm.String |
| unsigned numerical data, serial, decimal, dec, bit, blob, binary, geometry | N/D |


## <a name="next-steps"></a>Pasos siguientes

Felicidades. En este artículo, aprendió a integrar MySQL con Azure Cognitive Search con un indexador.

+ Para más información sobre los indexadores, consulte [Creación de indexadores en Azure Cognitive Search](search-howto-create-indexers.md).
