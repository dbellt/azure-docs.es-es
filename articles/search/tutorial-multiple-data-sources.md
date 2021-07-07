---
title: Tutorial de C# sobre la indexación de varios orígenes de datos de Azure
titleSuffix: Azure Cognitive Search
description: Aprenda a importar datos desde varios orígenes en un único índice de Azure Cognitive Search mediante indexadores. Este tutorial y el código de ejemplo están en C#.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 01/23/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 415c980390faae86e27efa6b6f63ec0334d6d0f0
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111560229"
---
# <a name="tutorial-index-from-multiple-data-sources-using-the-net-sdk"></a>Tutorial: Indexación de varios orígenes de datos mediante el SDK de .NET

Azure Cognitive Search puede importar, analizar e indexar datos desde varios orígenes de datos en un único índice de búsqueda consolidado. 

En este tutorial se usa C# y la biblioteca cliente [Azure.Search.Documents](/dotnet/api/overview/azure/search) del SDK de Azure para .NET con el fin de indexar los datos de hotel de ejemplo de una instancia de Azure Cosmos DB y fusionarlos con los detalles de las habitaciones de hotel extraídos de los documentos de Azure Blob Storage. El resultado será un índice de búsqueda combinado de hoteles que contiene documentos de estos con las habitaciones como tipos de datos complejos.

En este tutorial, realizará las siguientes tareas:

> [!div class="checklist"]
> * Cargar datos de ejemplo y crear orígenes de datos
> * Identificar la clave del documento
> * Definir y crear el índice
> * Indexar los datos de los hoteles desde Azure Cosmos DB
> * Combinar los datos de las habitaciones de hotel desde Blob Storage

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="overview"></a>Introducción

En este tutorial se usa la nueva biblioteca cliente, [Azure.Search.Documents](/dotnet/api/overview/azure/search), versión 11.x, para crear y ejecutar varios indexadores. En este tutorial se configuran dos orígenes de datos de Azure para que pueda configurar después un indexador que extraiga datos de ambos para rellenar un índice de búsqueda único. Los dos conjuntos de datos deben tener un valor en común para admitir la combinación. En este ejemplo, ese campo es un identificador. Siempre que haya un campo en común para admitir la asignación, se pueden combinar datos de recursos dispares con un indexador, por ejemplo, datos estructurados de Azure SQL, datos no estructurados de Blob Storage o cualquier combinación de [Orígenes de datos admitidos](search-indexer-overview.md#supported-data-sources) de Azure.

En el proyecto siguiente se puede encontrar una versión finalizada del código de este tutorial:

* [multiple-data-sources/v11 (GitHub)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/multiple-data-sources/v11)

Este tutorial se ha actualizado para usar el paquete Azure.Search.Documents (versión 11). Para una versión anterior del SDK de .NET, consulte el [código de ejemplo de Microsoft.Azure.Search (versión 10)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/multiple-data-sources/v10) en GitHub.

## <a name="prerequisites"></a>Prerrequisitos

+ [Azure Cosmos DB](../cosmos-db/create-cosmosdb-resources-portal.md)
+ [Almacenamiento de Azure](../storage/common/storage-account-create.md)
+ [Visual Studio](https://visualstudio.microsoft.com/)
+ [Paquete NuGet de Azure Cognitive Search (versión 11.x)](https://www.nuget.org/packages/Azure.Search.Documents/)
+ [Creación](search-create-service-portal.md) o [búsqueda de un servicio de búsqueda existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) 

> [!Note]
> Puede usar un servicio gratuito para este tutorial. Un servicio de búsqueda gratuito tiene una limitación de tres índices, tres indexadores y tres orígenes de datos. En este tutorial se crea uno de cada uno. Antes de empezar, asegúrese de que haya espacio en el servicio para aceptar los nuevos recursos.

## <a name="1---create-services"></a>1: Creación de servicios

En este tutorial se usa Azure Cognitive Search para la indexación y las consultas, Azure Cosmos DB para un conjunto de datos y Azure Blob Storage para el segundo conjunto de datos. 

Si es posible, cree todos los servicios en la misma región y grupo de recursos por proximidad y capacidad de administración. En la práctica, los servicios pueden estar en cualquier región.

Este ejemplo utiliza dos conjuntos pequeños de datos que describen las siete hoteles ficticios. Uno de ellos describe los hoteles en sí y se cargará en una base de datos de Azure Cosmos DB. El otro contiene los detalles de las habitaciones y se proporciona como siete archivos JSON independientes que se cargarán en Azure Blob Storage.

### <a name="start-with-cosmos-db"></a>Comenzar con Cosmos DB

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a la página Introducción de la cuenta de Azure Cosmos DB.

1. Seleccione **Explorador de datos** y luego **Nueva base de datos**.

   :::image type="content" source="media/tutorial-multiple-data-sources/cosmos-newdb.png" alt-text="Crear una base de datos nueva" border="false":::

1. Escriba el nombre **hotel-rooms-db**. Acepte los valores predeterminados para la configuración restante.

   :::image type="content" source="media/tutorial-multiple-data-sources/cosmos-dbname.png" alt-text="Configuración de la base de datos" border="false":::

1. Cree un contenedor nuevo. Use la base de datos existente que acaba de crear. Escriba **hotels** como nombre del contenedor y use **/HotelId** como clave de partición.

   :::image type="content" source="media/tutorial-multiple-data-sources/cosmos-add-container.png" alt-text="Agregar contenedor" border="false":::

1. Seleccione **Elementos** en **hotels** y luego haga clic en **Upload Item** (Cargar elemento) en la barra de comandos. Vaya al archivo **cosmosdb/HotelsDataSubset_CosmosDb.json** de la carpeta del proyecto y selecciónelo.

   :::image type="content" source="media/tutorial-multiple-data-sources/cosmos-upload.png" alt-text="Carga en la colección de Azure Cosmos DB" border="false":::

1. Utilice el botón Actualizar para actualizar la vista de los elementos de la colección de hoteles. Verá siete nuevos documentos de base de datos enumerados.

1. Copie una cadena de conexión de la página **Claves** en el Bloc de notas. Lo necesitará para **appsettings.json** en un paso posterior. Si no ha utilizado el nombre de base de datos sugerido, "hotel-rooms-dd", copie también el nombre de la base de datos.

### <a name="azure-blob-storage"></a>Azure Blob Storage

1. Inicie sesión en [Azure Portal](https://portal.azure.com), vaya a su cuenta de Azure Storage, haga clic en **Blobs** y, después, en **+Contenedor**.

1. [Cree un contenedor de blobs](../storage/blobs/storage-quickstart-blobs-portal.md) denominado **hotel-rooms** para almacenar los archivos JSON de las habitaciones de hotel de ejemplo. Puede establecer el nivel de acceso público a cualquiera de sus valores válidos.

   :::image type="content" source="media/tutorial-multiple-data-sources/blob-add-container.png" alt-text="Creación de un contenedor de blobs" border="false":::

1. Una vez creado el contenedor, ábralo y seleccione **Cargar** en la barra de comandos. Vaya a la carpeta que contiene los archivos de ejemplo. Selecciónelos todos ellos y haga clic en **Cargar**.

   :::image type="content" source="media/tutorial-multiple-data-sources/blob-upload.png" alt-text="Carga de archivos" border="false":::

1. Copie el nombre de la cuenta de almacenamiento y una cadena de conexión de la página **Claves de acceso** en el Bloc de notas. Necesitará estos dos valores para **appsettings.json** en un paso posterior.

### <a name="azure-cognitive-search"></a>Azure Cognitive Search

El tercer componente es Azure Cognitive Search, que se puede [crear en el portal](search-create-service-portal.md). 

### <a name="copy-an-admin-api-key-and-url-for-azure-cognitive-search"></a>Copia de una clave de API de administración y una dirección URL para Azure Cognitive Search

Para la autenticación en el servicio de búsqueda, necesitará la dirección URL del servicio y una clave de acceso.

1. [Inicie sesión en Azure Portal](https://portal.azure.com/) y en la página **Introducción** del servicio de búsqueda, obtenga la dirección URL. Un punto de conexión de ejemplo podría ser similar a `https://mydemo.search.windows.net`.

1. En **Configuración** > **Claves**, obtenga una clave de administrador para tener derechos completos en el servicio. Se proporcionan dos claves de administrador intercambiables para lograr la continuidad empresarial, por si necesitara sustituir una de ellas. Puede usar la clave principal o secundaria en las solicitudes para agregar, modificar y eliminar objetos.

   :::image type="content" source="media/search-get-started-javascript/service-name-and-keys.png" alt-text="Obtención del nombre del servicio y las claves de consulta y administrador" border="false":::

Tener una clave válida genera la confianza, solicitud a solicitud, entre la aplicación que envía la solicitud y el servicio que se encarga de ella.

## <a name="2---set-up-your-environment"></a>2: Configuración del entorno

1. Inicie Visual Studio y, en el menú **Herramientas**, seleccione **Administrador de paquetes NuGet** y **Administrar paquetes NuGet para la solución...** 

1. En la pestaña **Examinar**, busque e instale **Azure.Search.Documents** (versión 11.0 o posterior). Tendrá que hacer clic en algunos cuadros de diálogo adicionales para completar la instalación.

    :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-nuget-azure.png" alt-text="Uso de NuGet para agregar bibliotecas de Azure" border="false":::

1. Busque los paquetes NuGet **Microsoft.Extensions.Configuration** y **Microsoft.Extensions.Configuration.Json** e instálelos también.

1. Abra el archivo de solución **/v11/AzureSearchMultipleDataSources.sln**.

1. En el Explorador de soluciones, edite el archivo **appsettings.json** para agregar información de conexión.  

    ```json
    {
      "SearchServiceUri": "<YourSearchServiceURL>",
      "SearchServiceAdminApiKey": "<YourSearchServiceAdminApiKey>",
      "BlobStorageAccountName": "<YourBlobStorageAccountName>",
      "BlobStorageConnectionString": "<YourBlobStorageConnectionString>",
      "CosmosDBConnectionString": "<YourCosmosDBConnectionString>",
      "CosmosDBDatabaseName": "hotel-rooms-db"
    }
    ```

Las dos primeras entradas usan las claves de dirección URL y de administrador de un servicio de búsqueda. Use el punto de conexión completo, por ejemplo, `https://mydemo.search.windows.net`.

Las siguientes entradas especifican nombres de cuenta e información de la cadena de conexión de los orígenes de datos de Azure Blob Storage y Azure Cosmos DB.

## <a name="3---map-key-fields"></a>3: Asignación de campos de clave

La combinación de contenido requiere que ambos flujos de datos tengan como destino los mismos documentos en el índice de búsqueda. 

En Azure Cognitive Search, el campo de clave identifica de manera exclusiva cada documento. Cada índice de búsqueda debe tener exactamente un campo clave de tipo `Edm.String`. Ese campo de clave debe estar presente para cada documento de un origen de datos que se agregue al índice. (de hecho, es el único campo obligatorio).

Al indexar datos de varios orígenes de datos, asegúrese de que cada documento o fila entrante contiene una clave de documento común para combinar datos de dos documentos de origen físicamente distintos en un nuevo documento de búsqueda en el índice combinado. 

A menudo requiere planeación inicial para identificar una clave de documentación significativa para el índice; asegúrese de que existe en ambos orígenes de datos. En esta demostración, la clave `HotelId` de cada hotel de Cosmos DB también está presente en los blobs JSON de Blob Storage.

Los indexadores de Azure Cognitive Search pueden usar asignaciones de campo para cambiar el nombre e incluso el formato de campos de datos durante la indexación para poder dirigir los datos de origen al campo de índice correcto. Por ejemplo, en Cosmos DB, el identificador de los hoteles se denomina **`HotelId`** . Pero en los archivos de blob JSON de las habitaciones de los hoteles, se denomina **`Id`** . El programa se encarga de ello mediante la asignación del campo **`Id`** desde los blobs hasta el campo de clave **`HotelId`** del indexador.

> [!NOTE]
> En la mayoría de los casos, las claves de documento generadas automáticamente, como las que crean algunos indexadores de forma predeterminada, no son buenas claves de documento para los índices combinados. En general, es preferible usar un valor de clave único y significativo que ya exista en los orígenes de datos o que se pueda agregar fácilmente a ellos.

## <a name="4---explore-the-code"></a>4: Exploración del código

Una vez que los valores de configuración y los datos estén en su lugar, el ejemplo de programa de **/v11/AzureSearchMultipleDataSources.sln** estará listo para la compilación y la ejecución.

Esta sencilla aplicación de consola de C#/.NET realiza las siguientes tareas:

* Crea un nuevo índice basado en la estructura de datos de la clase Hotel de C# (que también hace referencia a las clases Address y Room).
* Crea un nuevo origen de datos y un indexador que asigna los datos de Azure Cosmos DB a campos de índice. Ambos son objetos de Azure Cognitive Search.
* Ejecuta el indexador para cargar datos de Hotel desde Cosmos DB.
* Crea un segundo origen de datos y un indexador que asigna datos de blob JSON a campos de índice.
* Ejecuta el segundo indexador para cargar datos de Rooms desde Blob Storage.

 Antes de ejecutar el programa, dedíquele un minuto a estudiar el código y las definiciones de índice e indexador de este ejemplo. El código pertinente aparece en dos archivos:

  + **Hotel.cs** contiene el esquema que define el índice.
  + **Program.cs** contiene las funciones que crean el índice de Azure Cognitive Search, los orígenes de datos y los indexadores, y cargan los resultados combinados en el índice.

### <a name="create-an-index"></a>Creación de un índice

Este programa de ejemplo usa [CreateIndexAsync](/dotnet/api/azure.search.documents.indexes.searchindexclient.createindexasync) para definir y crear un índice de Azure Cognitive Search. Aprovecha la clase [FieldBuilder](/dotnet/api/azure.search.documents.indexes.fieldbuilder) para generar una estructura de índice a partir de una clase de modelo de datos de C#.

El modelo de datos se define mediante la clase Hotel, que también contiene referencias a las clases Address y Room. FieldBuilder explora en profundidad varias definiciones de clase para generar una estructura de datos compleja para el índice. Se usan etiquetas de metadatos para definir los atributos de cada campo, como si se puede buscar u ordenar.

El programa eliminará los índices existentes con el mismo nombre antes de crear el nuevo, por si desea ejecutar este ejemplo más veces.

Los fragmentos de código siguientes del archivo **Hotel.cs** muestran campos únicos, seguidos de una referencia a otra clase de modelo de datos, Room[], que a su vez se define en el archivo **Room.cs** (no se muestra).

```csharp
. . .
[SimpleField(IsFilterable = true, IsKey = true)]
public string HotelId { get; set; }

[SearchableField(IsFilterable = true, IsSortable = true)]
public string HotelName { get; set; }
. . .
public Room[] Rooms { get; set; }
. . .
```

En el archivo **Program.cs**, se define [SearchIndex](/dotnet/api/azure.search.documents.indexes.models.searchindex) con un nombre y una colección de campos generados por el método `FieldBuilder.Build`, y se crea como sigue:

```csharp
private static async Task CreateIndexAsync(string indexName, SearchIndexClient indexClient)
{
    // Create a new search index structure that matches the properties of the Hotel class.
    // The Address and Room classes are referenced from the Hotel class. The FieldBuilder
    // will enumerate these to create a complex data structure for the index.
    FieldBuilder builder = new FieldBuilder();
    var definition = new SearchIndex(indexName, builder.Build(typeof(Hotel)));

    await indexClient.CreateIndexAsync(definition);
}
```

### <a name="create-azure-cosmos-db-data-source-and-indexer"></a>Creación de un origen de datos de Azure Cosmos DB y de un indexador

A continuación, el programa principal incluirá lógica para crear el origen de datos de Azure Cosmos DB para los datos de los hoteles.

En primer lugar, concatena el nombre de la base de datos de Azure Cosmos DB con la cadena de conexión. A continuación, define un objeto [SearchIndexerDataSourceConnection](/dotnet/api/azure.search.documents.indexes.models.searchindexerdatasourceconnection).

```csharp
private static async Task CreateAndRunCosmosDbIndexerAsync(string indexName, SearchIndexerClient indexerClient)
{
    // Append the database name to the connection string
    string cosmosConnectString =
        configuration["CosmosDBConnectionString"]
        + ";Database="
        + configuration["CosmosDBDatabaseName"];

    SearchIndexerDataSourceConnection cosmosDbDataSource = new SearchIndexerDataSourceConnection(
        name: configuration["CosmosDBDatabaseName"],
        type: SearchIndexerDataSourceType.CosmosDb,
        connectionString: cosmosConnectString,
        container: new SearchIndexerDataContainer("hotels"));

    // The Cosmos DB data source does not need to be deleted if it already exists,
    // but the connection string might need to be updated if it has changed.
    await indexerClient.CreateOrUpdateDataSourceConnectionAsync(cosmosDbDataSource);
```

Una vez creado el origen de datos, el programa configura un indexador de Azure Cosmos DB denominado **hotel-rooms-cosmos-indexer**.

El programa actualizará los indexadores existentes con el mismo nombre y sobrescribirá el indexador existente con el contenido del código anterior. También incluye acciones de restablecimiento y ejecución, por si desea ejecutar este ejemplo más de una vez.

En el siguiente ejemplo se define una programación para el indexador para que se ejecute una vez al día. Puede quitar la propiedad de programación de esta llamada si no desea que el indexador se vuelva a ejecutar automáticamente en el futuro.

```csharp
SearchIndexer cosmosDbIndexer = new SearchIndexer(
    name: "hotel-rooms-cosmos-indexer",
    dataSourceName: cosmosDbDataSource.Name,
    targetIndexName: indexName)
{
    Schedule = new IndexingSchedule(TimeSpan.FromDays(1))
};

// Indexers keep metadata about how much they have already indexed.
// If we already ran the indexer, it "remembers" and does not run again.
// To avoid this, reset the indexer if it exists.
try
{
    await indexerClient.GetIndexerAsync(cosmosDbIndexer.Name);
    // Reset the indexer if it exists.
    await indexerClient.ResetIndexerAsync(cosmosDbIndexer.Name);
}
catch (RequestFailedException ex) when (ex.Status == 404)
{
    // If the indexer does not exist, 404 will be thrown.
}

await indexerClient.CreateOrUpdateIndexerAsync(cosmosDbIndexer);

Console.WriteLine("Running Cosmos DB indexer...\n");

try
{
    // Run the indexer.
    await indexerClient.RunIndexerAsync(cosmosDbIndexer.Name);
}
catch (RequestFailedException ex) when (ex.Status == 429)
{
    Console.WriteLine("Failed to run indexer: {0}", ex.Message);
}
```

Este ejemplo incluye un bloque sencillo try-catch para notificar los errores que puedan producirse durante la ejecución.

Tras la ejecución del indexador de Azure Cosmos DB, el índice de búsqueda contendrá un conjunto integral de documentos de ejemplo de hoteles. Sin embargo, el campo de habitaciones de cada hotel será una matriz vacía, ya que el origen de datos de Azure Cosmos DB omitía los detalles de las habitaciones. A continuación, el programa realizará una extracción de Blob Storage para cargar y combinar los datos de las habitaciones.

### <a name="create-blob-storage-data-source-and-indexer"></a>Creación del indexador y del origen de datos de Blob Storage

Para obtener los detalles de las habitaciones, el programa primero configura un origen de datos de Blob Storage para hacer referencia a un conjunto de archivos de blob JSON individuales.

```csharp
private static async Task CreateAndRunBlobIndexerAsync(string indexName, SearchIndexerClient indexerClient)
{
    SearchIndexerDataSourceConnection blobDataSource = new SearchIndexerDataSourceConnection(
        name: configuration["BlobStorageAccountName"],
        type: SearchIndexerDataSourceType.AzureBlob,
        connectionString: configuration["BlobStorageConnectionString"],
        container: new SearchIndexerDataContainer("hotel-rooms"));

    // The blob data source does not need to be deleted if it already exists,
    // but the connection string might need to be updated if it has changed.
    await indexerClient.CreateOrUpdateDataSourceConnectionAsync(blobDataSource);
```

Una vez creado el origen de datos, el programa configura un indexador de blobs denominado **hotel-rooms-blob-indexer**, como se muestra a continuación.

Los blobs JSON contienen un campo de clave denominado **`Id`** en lugar de **`HotelId`** . El código usa la clase `FieldMapping` para indicar al indexador que dirija el valor del campo **`Id`** a la clave de documento **`HotelId`** del índice.

Los indexadores de Blob Storage pueden usar [IndexingParameters](/dotnet/api/azure.search.documents.indexes.models.indexingparameters) para especificar un modo de análisis. Debe establecer modos de análisis diferentes en función de si los blobs representan un único documento o varios en el mismo blob. En este ejemplo, cada blob representa un documento de JSON único, por lo que el código usa el modo de análisis `json`. Para más información sobre los parámetros de análisis del indexador, consulte el artículo sobre la [Indexación de blobs JSON](search-howto-index-json-blobs.md).

Este ejemplo define una programación para el indexador para que se ejecute una vez al día. Puede quitar la propiedad de programación de esta llamada si no desea que el indexador se vuelva a ejecutar automáticamente en el futuro.

```csharp
// Map the Id field in the Room documents to the HotelId key field in the index
List<FieldMapping> map = new List<FieldMapping> {
    new FieldMapping("Id")
    {
        TargetFieldName =  "HotelId"
    }
};

IndexingParameters parameters = new IndexingParameters();
parameters.Configuration.Add("parsingMode", "json");

SearchIndexer blobIndexer = new SearchIndexer(
    name: "hotel-rooms-blob-indexer",
    dataSourceName: blobDataSource.Name,
    targetIndexName: indexName)
{
    Parameters = parameters,
    Schedule = new IndexingSchedule(TimeSpan.FromDays(1))
};

blobIndexer.FieldMappings.Add(new FieldMapping("Id") { TargetFieldName = "HotelId" });

// Reset the indexer if it already exists
try
{
    await indexerClient.GetIndexerAsync(blobIndexer.Name);
    await indexerClient.ResetIndexerAsync(blobIndexer.Name);
}
catch (RequestFailedException ex) when (ex.Status == 404) { }

await indexerClient.CreateOrUpdateIndexerAsync(blobIndexer);

try
{
    // Run the indexer.
    await searchService.Indexers.RunAsync(blobIndexer.Name);
}
catch (CloudException e) when (e.Response.StatusCode == (HttpStatusCode)429)
{
    Console.WriteLine("Failed to run indexer: {0}", e.Response.Content);
}
```

Dado que el índice ya se ha rellenado con datos de los hoteles desde la base de datos de Azure Cosmos DB, el indexador de blobs actualiza los documentos existentes en el índice y agrega los detalles de las habitaciones.

> [!NOTE]
> Si tiene los mismos campos (distintos del clave) en ambos orígenes de datos y los datos de esos campos no coinciden, el índice contendrá los valores del indexador que se ejecutara más recientemente. En nuestro ejemplo, ambos orígenes de datos contienen un campo **HotelName**. Si, por alguna razón, los datos de este campo difieren para documentos con el mismo valor de calve, los datos de **HotelName** del origen de datos que se indexara más recientemente serán los que se almacenen en el índice como valor.

## <a name="5---search"></a>5: Búsqueda

Puede explorar el índice de búsqueda relleno tras la ejecución del programa con el [**Explorador de búsqueda**](search-explorer.md) del portal.

En Azure Portal, abra la página **Introducción** del servicio de búsqueda y busque el índice **hotel-rooms-sample** en la lista **Índices**.

  :::image type="content" source="media/tutorial-multiple-data-sources/index-list.png" alt-text="Lista de índices de Azure Cognitive Search" border="false":::

Haga clic en el índice hotel-rooms-sample en la lista. Verá una interfaz del Explorador de búsqueda para el índice. Escriba una consulta con un término, como "Luxury". Debería ver al menos un documento en los resultados y este documento debería mostrar una lista de objetos de las habitaciones de la matriz.

## <a name="reset-and-rerun"></a>Restablecer y volver a ejecutar

En las primeras etapas experimentales de desarrollo, el enfoque más práctico para las iteraciones del diseño es eliminar los objetos de Azure Cognitive Search y permitir que el código vuelva a generarlos. Los nombres de los recursos son únicos. La eliminación de un objeto permite volver a crearlo con el mismo nombre.

En el código de ejemplo se comprueban los objetos existentes y se eliminan o se actualizan para que pueda volver a ejecutar el programa.

También puede usar el portal para eliminar los índices, indexadores y orígenes de datos.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando trabaje con su propia suscripción, al final de un proyecto, es recomendable eliminar los recursos que ya no necesite. Los recursos que se dejan en ejecución pueden costarle mucho dinero. Puede eliminar los recursos de forma individual o eliminar el grupo de recursos para eliminar todo el conjunto de recursos.

Puede buscar y administrar los recursos en el portal, mediante el vínculo Todos los recursos o Grupos de recursos en el panel de navegación izquierdo.

## <a name="next-steps"></a>Pasos siguientes

Ahora que está familiarizado con el concepto de ingesta de datos de varios orígenes, echemos un vistazo más de cerca a la configuración del indexador, comenzando por Cosmos DB.

> [!div class="nextstepaction"]
> [Configuración de un indexador de Azure Cosmos DB](search-howto-index-cosmosdb.md)
