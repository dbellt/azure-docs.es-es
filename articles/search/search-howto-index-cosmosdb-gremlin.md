---
title: Búsqueda de datos mediante Gremlin API de Azure Cosmos DB (versión preliminar)
titleSuffix: Azure Cognitive Search
description: Importe datos de Gremlin API de Azure Cosmos DB a un índice que permita búsquedas en Azure Cognitive Search. Los indexadores automatizan la ingesta de datos para orígenes de datos seleccionados, como Azure Cosmos DB.
author: vkurpad
manager: luisca
ms.author: vikurpad
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/11/2021
ms.openlocfilehash: a1181af6bf7f8c734d7cb90c7e9bb28c22e00b08
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109517989"
---
# <a name="how-to-index-data-available-through-cosmos-db-gremlin-api-using-an-indexer-preview"></a>Cómo indexar los datos disponibles mediante Gremlin API de Cosmos DB con un indexador (versión preliminar)

> [!IMPORTANT]
> El indexador de Gremlin API de Cosmos DB está actualmente en versión preliminar. La funcionalidad de versión preliminar se ofrece sin contrato de nivel de servicio y no es aconsejable usarla con cargas de trabajo de producción. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
> Para solicitar acceso a las versión preliminar, puede rellenar [este formulario](https://aka.ms/azure-cognitive-search/indexer-preview).
> En esta versión preliminar, se recomienda usar la [API REST, versión 2020-06-30-Preview](search-api-preview.md). Actualmente hay compatibilidad limitada con el portal y no la hay con .NET SDK.

> [!WARNING]
> Para que Azure Cognitive Search indexe los datos de Cosmos DB mediante Gremlin API, la [propia indexación de Cosmos DB](../cosmos-db/index-overview.md) también debe habilitarse y establecerse en [Coherente](../cosmos-db/index-policy.md#indexing-mode). Esta es la configuración predeterminada en Cosmos DB. La indexación de Azure Cognitive Search no funcionará sin no está ya habilitada la indexación de Cosmos DB.

La [indexación de Azure Cosmos DB](../cosmos-db/index-overview.md) y la [indexación de Azure Cognitive Search](search-what-is-an-index.md) son operaciones distintas, específicas de cada servicio. Antes de comenzar la indexación de Azure Cognitive Search, ya debe existir la base de datos de Azure Cosmos DB.

En este artículo se muestra cómo configurar Azure Cognitive Search para indexar el contenido de Azure Cosmos DB mediante Gremlin API. Este flujo de trabajo crea un índice de Azure Cognitive Search y lo carga con texto existente extraído de Azure Cosmos DB mediante Gremlin API.

## <a name="get-started"></a>Primeros pasos

Puede usar la [API REST en versión preliminar](https://docs.microsoft.com/rest/api/searchservice/index-preview) para indexar datos de Azure Cosmos DB que están disponibles mediante Gremlin API siguiendo un flujo de trabajo de tres partes común a todos los indexadores de Azure Cognitive Search: se crea un origen de datos, se crea un índice y se crea un indexador. En el proceso siguiente, la extracción de datos de Cosmos DB se inicia al enviar la solicitud Crear indexador.

De forma predeterminada, el indexador de Gremlin API de Cosmos DB de Azure Cognitive Search hará que cada vértice del grafo sea un documento en el índice. Los bordes se omitirán. Como alternativa, puede establecer la consulta para indexar solo los bordes.

### <a name="step-1---assemble-inputs-for-the-request"></a>Paso 1: Ensamblado de las entradas de la solicitud

Para cada solicitud, debe proporcionar el nombre del servicio y la clave de administrador de Azure Cognitive Search (en el encabezado POST). Puede usar [Postman](search-get-started-postman.md) o cualquier cliente de API REST para enviar solicitudes HTTPS a Azure Cognitive Search.

Copie y guarde los siguientes valores para usarlos en la solicitud:

+ El nombre del servicio Azure Cognitive Search
+ La clave de administrador de Azure Cognitive Search
+ Cadena de conexión de Gremlin API de Cosmos DB

También puede encontrar estos valores en Azure Portal:

1. En las páginas del portal de Azure Cognitive Search, copie la URL del servicio de búsqueda desde la página de información general.

2. En el panel de navegación izquierdo, haga clic en **Claves** y, luego, copie la clave primaria o la secundaria.

3. Cambie a las páginas del portal en la cuenta de Cosmos DB. En el panel de navegación de la izquierda, en **Configuración**, haga clic en **Claves**. En esta página se proporciona un URI, dos conjuntos de cadenas de conexión y dos conjuntos de claves. En el Bloc de notas, copie una de las cadenas de conexión.

### <a name="step-2---create-a-data-source"></a>Paso 2: creación de un origen de datos

A **origen de datos** especifica los datos para indexar, las credenciales y las directivas para identificar cambios en los datos (por ejemplo, los documentos modificados o eliminados dentro de la colección). El origen de datos se define como un recurso independiente para que puedan usarlo múltiples indizadores.

Para crear un origen de datos, formule una solicitud POST:

```http
    POST https://[service name].search.windows.net/datasources?api-version=2020-06-30-Preview
    Content-Type: application/json
    api-key: [Search service admin key]
    
    {
        "name": "mycosmosdbgremlindatasource",
        "type": "cosmosdb",
        "credentials": {
            "connectionString": "AccountEndpoint=https://myCosmosDbEndpoint.documents.azure.com;AccountKey=myCosmosDbAuthKey;ApiKind=Gremlin;Database=myCosmosDbDatabaseId"
        },
        "container": { "name": "myGraphId", "query": null }
    }
```

El cuerpo de la solicitud contiene la definición del origen de datos, que debe incluir los siguientes campos:

| Campo   | Descripción |
|---------|-------------|
| **name** | Necesario. Elija un nombre para representar el objeto de origen de datos. |
|**type**| Necesario. Debe ser `cosmosdb`. |
|**credentials** | Necesario. **connectionString** debe incluir un objeto AccountEndpoint, AccountKey, ApiKind y Database. ApiKind es **Gremlin**.</br></br>Por ejemplo:<br/>`AccountEndpoint=https://<Cosmos DB account name>.documents.azure.com;AccountKey=<Cosmos DB auth key>;Database=<Cosmos DB database id>;ApiKind=Gremlin`<br/><br/>AccountEndpoint debe usar el punto de conexión `*.documents.azure.com`.
| **container** | Contiene los siguientes elementos: <br/>**name**: Necesario. Especifique el identificador del grafo.<br/>**query**: Opcional. El valor predeterminado es `g.V()`. Para indexar los bordes, establezca la consulta en `g.E()`. |
| **dataChangeDetectionPolicy** | El progreso incremental se habilitará de forma predeterminada usando `_ts` como columna de límite superior. |
|**dataDeletionDetectionPolicy** | Opcional. Consulte la sección [Indexación de documentos eliminados](#DataDeletionDetectionPolicy).|

### <a name="step-3---create-a-target-search-index"></a>Paso 3: Creación de un índice de búsqueda de destino

Si aún no tiene un [índice de Azure Cognitive Search de destino](/rest/api/searchservice/create-index), cree uno. En el ejemplo siguiente se crea un índice con campos de identificador, etiqueta y descripción:

```http
    POST https://[service name].search.windows.net/indexes?api-version=2020-06-30-Preview
    Content-Type: application/json
    api-key: [Search service admin key]

    {
       "name": "mysearchindex",
       "fields": [
        {
            "name": "rid",
            "type": "Edm.String",
            "facetable": false,
            "filterable": false,
            "key": true,
            "retrievable": true,
            "searchable": true,
            "sortable": false,
            "analyzer": "standard.lucene",
            "indexAnalyzer": null,
            "searchAnalyzer": null,
            "synonymMaps": [],
            "fields": []
        },{
            "name": "id",
            "type": "Edm.String",
            "searchable": true,
            "filterable": false,
            "retrievable": true,
            "sortable": false,
            "facetable": false,
            "key": false,
            "indexAnalyzer": null,
            "searchAnalyzer": null,
            "analyzer": "standard.lucene",
            "synonymMaps": []
        }, {
            "name": "label",
            "type": "Edm.String",
            "searchable": true,
            "filterable": false,
            "retrievable": true,
            "sortable": false,
            "facetable": false,
            "key": false,
            "indexAnalyzer": null,
            "searchAnalyzer": null,
            "analyzer": "standard.lucene",
            "synonymMaps": []
       }]
     }
```

Asegúrese de que el esquema del índice de destino sea compatible con el grafo.

En el caso de las colecciones con particiones, la clave de documento predeterminada es la propiedad `_rid` de Azure Cosmos DB, a la que Azure Cognitive Search cambia el nombre automáticamente a `rid`, porque los nombres de los campos no pueden comenzar por un carácter de guion bajo. Además, los valores `_rid` de Azure Cosmos DB contienen caracteres que no son válidos en las claves de Azure Cognitive Search. Por esta razón, los valores `_rid` deben estar codificados en Base64 si quiere convertirlo en la clave del documento.

### <a name="mapping-between-json-data-types-and-azure-cognitive-search-data-types"></a>Asignación entre tipos de datos de JSON y de Azure Cognitive Search

| Tipo de datos JSON | Tipos de campos de índice de destino compatibles |
| --- | --- |
| Bool |Edm.Boolean, Edm.String |
| Números que parecen enteros |Edm.Int32, Edm.Int64, Edm.String |
| Números que parecen puntos flotantes |Edm.Double, Edm.String |
| String |Edm.String |
| Matrices de tipos primitivos, por ejemplo ["a", "b", "c"] |Collection(Edm.String) |
| Cadenas que parecen fechas |Edm.DateTimeOffset, Edm.String |
| Objetos GeoJSON, por ejemplo {"type": "Point", "coordinates": [long, lat] } |Edm.GeographyPoint |
| Otros objetos JSON |N/D |

### <a name="step-4---configure-and-run-the-indexer"></a>Paso 4: configuración y ejecución del indexador

Una vez creados el origen de datos y los índices, ya podrá crear el indizador:

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      "name": "mycosmosdbgremlinindexer",
      "description": "My Cosmos DB Gremlin API indexer",
      "dataSourceName": "mycosmosdbgremlindatasource",
      "targetIndexName": "mysearchindex"
    }
```

Este indexador comenzará a ejecutarse después de crearlo y solo lo hará una vez. Puede agregar el parámetro de programación opcional a la solicitud para establecer que el indexador se ejecute según una programación. Para más información sobre cómo definir las programaciones del indexador, consulte [Programación de indexadores para Azure Cognitive Search](search-howto-schedule-indexers.md).

Para más información sobre la API Create Indexer, consulte [Crear indexador](https://docs.microsoft.com/rest/api/searchservice/create-indexer).

<a name="DataDeletionDetectionPolicy"></a>

## <a name="indexing-deleted-documents"></a>Indexación de documentos eliminados

Cuando se eliminen los datos del grafo, es posible que quiera eliminar también su documento correspondiente del índice de búsqueda. El propósito de una directiva de detección de eliminación de datos es identificar eficazmente los elementos de datos eliminados y eliminar el documento completo del índice. La directiva de detección de eliminación de datos no está diseñada para eliminar información parcial del documento. Actualmente, solo es compatible la directiva `Soft Delete` (la eliminación se indica con algún tipo de marca), especificada de la siguiente forma:

```http
    {
        "@odata.type" : "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
        "softDeleteColumnName" : "the property that specifies whether a document was deleted",
        "softDeleteMarkerValue" : "the value that identifies a document as deleted"
    }
```

En el ejemplo siguiente se crea un origen de datos con una directiva de eliminación temporal:

```http
    POST https://[service name].search.windows.net/datasources?api-version=2020-06-30-Preview
    Content-Type: application/json
    api-key: [Search service admin key]
    
    {
        "name": "mycosmosdbgremlindatasource",
        "type": "cosmosdb",
        "credentials": {
            "connectionString": "AccountEndpoint=https://myCosmosDbEndpoint.documents.azure.com;AccountKey=myCosmosDbAuthKey;ApiKind=Gremlin;Database=myCosmosDbDatabaseId"
        },
        "container": { "name": "myCollection" },
        "dataChangeDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
            "highWaterMarkColumnName": "`_ts`"
        },
        "dataDeletionDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
            "softDeleteColumnName": "isDeleted",
            "softDeleteMarkerValue": "true"
        }
    }
```

<a name="MappingGraphData"></a>

## <a name="mapping-graph-data-to-a-search-index"></a>Asignación de datos del grafo a un índice de búsqueda

El indexador de Gremlin API de Cosmos DB asignará automáticamente un par de fragmentos de datos del grafo:

1. El indexador asignará `_rid` a un campo `rid` del índice si existe. Tenga en cuenta que, si quiere usar el valor `rid` como clave en el índice, debe codificar la clave en Base64, ya que `_rid` puede contener caracteres que no son válidos en las claves de documento de Azure Cognitive Search.

1. El indexador asignará `_id` a un campo `id` del índice si existe.

1. Al consultar la base de datos de Cosmos DB mediante Gremlin API, es posible que observe que la salida JSON de cada propiedad tiene un campo `id` y un campo `value`. El indexador de Cosmos DB de Azure Cognitive Search asignará automáticamente las propiedades `value` a un campo del índice de búsqueda que tenga el mismo nombre que la propiedad, si existe. En el ejemplo siguiente, 450 se asignaría a un campo `pages` del índice de búsqueda.

```http
    {
        "id": "Cookbook",
        "label": "book",
        "type": "vertex",
        "properties": {
          "pages": [
            {
              "id": "48cf6285-a145-42c8-a0aa-d39079277b71",
              "value": "450"
            }
          ]
        }
    }
```

Es posible que necesite usar [asignaciones de campos de salida](cognitive-search-output-field-mapping.md) para asignar la salida de la consulta a los campos del índice. Es probable que quiera usar asignaciones de campos de salida en lugar de [asignaciones de campos](search-indexer-field-mappings.md), ya que es existe la posibilidad de que la consulta personalizada tenga datos complejos.

Por ejemplo, supongamos que la consulta genera esta salida:

```json
    [
      {
        "vertex": {
          "id": "Cookbook",
          "label": "book",
          "type": "vertex",
          "properties": {
            "pages": [
              {
                "id": "48cf6085-a211-42d8-a8ea-d38642987a71",
                "value": "450"
              }
            ],
          }
        },
        "written_by": [
          {
            "yearStarted": "2017"
          }
        ]
      }
    ]
```

Si quiere asignar el valor de `pages` del archivo JSON anterior a un campo `totalpages` del índice, puede agregar la siguiente [asignación de campos de salida](cognitive-search-output-field-mapping.md) a la definición del indexador:

```json
    ... // rest of indexer definition 
    "outputFieldMappings": [
        {
          "sourceFieldName": "/document/vertex/pages",
          "targetFieldName": "totalpages"
        }
    ]
```

Observe que la asignación de campos de salida comienza con `/document` y no incluye una referencia a la clave de propiedades en el archivo JSON. Esto se debe a que el indexador coloca cada documento en el nodo `/document` al ingerir los datos del grafo y también le permite hacer referencia automáticamente al valor de `pages` con una referencia simple a `pages` en lugar de tener que hacer referencia al primer objeto de la matriz de `pages`.

## <a name="next-steps"></a>Pasos siguientes

+ Para obtener más información sobre Gremlin API de Azure Cosmos DB, vea [Introducción a Gremlin API de Azure Cosmos DB]()../cosmos-db/graph-introduction.md).

+ Para obtener más información sobre escenarios y precios de Azure Cognitive Search, vea la [página del servicio Search en azure.microsoft.com](https://azure.microsoft.com/services/search/).