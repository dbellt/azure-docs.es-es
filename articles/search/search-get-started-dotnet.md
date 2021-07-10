---
title: 'Inicio rápido: Creación de un índice de búsqueda en .NET'
titleSuffix: Azure Cognitive Search
description: En este inicio rápido de C#, se explica cómo crear un índice, cargar datos y ejecutar consultas mediante la biblioteca cliente Azure.Search.Documents.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 06/11/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 11870ed112f4d342bf6df95185d1005004df3f67
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112029286"
---
# <a name="quickstart-create-a-search-index-using-the-azuresearchdocuments-client-library"></a>Inicio rápido: Creación de un índice de búsqueda mediante la biblioteca cliente Azure.Search.Documents

Use la nueva [biblioteca cliente Azure.Search.Documents (versión 11)](/dotnet/api/overview/azure/search.documents-readme) para crear una aplicación de consola de .NET Core en C# que cree, cargue y consulte un índice de búsqueda.

Puede [descargar el código fuente](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/quickstart/v11) para empezar con un proyecto terminado o seguir los pasos especificados en este artículo para crear el suyo propio.

> [!NOTE]
> ¿Busca una versión anterior? Consulte [Creación de un índice de búsqueda con Microsoft.Azure.Search v10](search-get-started-dotnet-v10.md).

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar, tenga preparadas las herramientas y los servicios siguientes:

+ Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/).

+ Un servicio de Azure Cognitive Search. [Cree un servicio](search-create-service-portal.md) o [busque uno existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices). Puede usar un servicio gratuito para este inicio rápido. 

+ [Visual Studio](https://visualstudio.microsoft.com/downloads/), cualquier edición. El código de ejemplo se probó en la edición gratuita Community de Visual Studio 2019.

Al configurar el proyecto, descargará el [paquete NuGet Azure.Search.Documents](https://www.nuget.org/packages/Azure.Search.Documents/).

El SDK de Azure para .NET se ajusta a [.NET Standard 2.0](/dotnet/standard/net-standard#net-implementation-support); es decir, .NET Framework 4.6.1 y .NET Core 2.0 como requisitos mínimos.

## <a name="set-up-your-project"></a>Configuración del proyecto

Ensamble la información de conexión de servicio e inicie Visual Studio para crear un proyecto de aplicación de consola que se pueda ejecutar en .NET Core.

<a name="get-service-info"></a>

### <a name="copy-a-key-and-endpoint"></a>Copia de una clave y un punto de conexión

Las llamadas al servicio requieren un punto de conexión de URL y una clave de acceso en cada solicitud. Como primer paso, busque la clave de API y la dirección URL que se vayan a agregar al proyecto. Deberá especificar ambos valores al crear el cliente en un paso posterior.

1. [Inicie sesión en Azure Portal](https://portal.azure.com/) y en la página **Introducción** del servicio de búsqueda, obtenga la dirección URL. Un punto de conexión de ejemplo podría ser similar a `https://mydemo.search.windows.net`.

2. En **Configuración** > **Claves**, obtenga una clave de administración para los derechos completos en el servicio, esta es necesaria si va a crear o eliminar objetos. Hay dos claves intercambiables una principal y otra secundaria. Puede usar cualquiera de ellas.

   ![Obtención de una clave de acceso y un punto de conexión HTTP](media/search-get-started-rest/get-url-key.png "Obtención de una clave de acceso y un punto de conexión HTTP")

Todas las solicitudes requieren una clave de API en cada solicitud enviada al servicio. Tener una clave válida genera la confianza, solicitud a solicitud, entre la aplicación que envía la solicitud y el servicio que se encarga de ella.

### <a name="install-the-nuget-package"></a>Instalación del paquete NuGet.

Una vez creado el proyecto, agregue la biblioteca de cliente. El [paquete Azure.Search.Documents](https://www.nuget.org/packages/Azure.Search.Documents/) consta de una biblioteca de cliente que proporciona todas las API que se usan para trabajar con un servicio de búsqueda en .NET.

1. Inicie Visual Studio y cree una aplicación de consola de .NET Core.

1. En **Herramientas** > **Administrador de paquetes NuGet** seleccione **Administrar paquetes NuGet para la solución....** . 

1. Haga clic en **Examinar**.

1. Busque `Azure.Search.Documents` y seleccione la versión 11.0 ou otra posterior.

1. Haga clic a la derecha en **Instalar** para agregar el ensamblado al proyecto y la solución.

### <a name="create-a-search-client"></a>Creación de un cliente de búsqueda

1. En **Program.cs**, cambie el espacio de nombres a `AzureSearch.SDK.Quickstart.v11` y, a continuación, agregue las siguientes directivas `using`.

   ```csharp
   using Azure;
   using Azure.Search.Documents;
   using Azure.Search.Documents.Indexes;
   using Azure.Search.Documents.Indexes.Models;
   using Azure.Search.Documents.Models;
   ```

1. Cree dos clientes: [SearchIndexClient](/dotnet/api/azure.search.documents.indexes.searchindexclient) crea el índice y [SearchClient](/dotnet/api/azure.search.documents.searchclient) carga y consulta uno existente. Ambos necesitan el punto de conexión de servicio y una clave de API de administración para la autenticación con derechos de creación y eliminación.

   ```csharp
   static void Main(string[] args)
   {
       string serviceName = "<YOUR-SERVICE-NAME>";
       string indexName = "hotels-quickstart";
       string apiKey = "<YOUR-ADMIN-API-KEY>";

        // Create a SearchIndexClient to send create/delete index commands
        Uri serviceEndpoint = new Uri($"https://{serviceName}.search.windows.net/");
        AzureKeyCredential credential = new AzureKeyCredential(apiKey);
        SearchIndexClient adminClient = new SearchIndexClient(serviceEndpoint, credential);

        // Create a SearchClient to load and query documents
        SearchClient srchclient = new SearchClient(serviceEndpoint, indexName, credential);
    ```

## <a name="1---create-an-index"></a>1 - Creación de un índice

En este inicio rápido se crea un índice de hoteles que se cargará con datos de hotel y en el que se ejecutarán consultas. En este paso, defina los campos en el índice. Cada definición de campo incluye un nombre, un tipo de datos y atributos que determinan cómo se usa el campo.

En este ejemplo, se usan métodos sincrónicos de la biblioteca Azure.Search.Documents para simplificar y mejorar la legibilidad. Sin embargo, para los escenarios de producción, debe usar métodos asincrónicos para mantener la aplicación escalable y receptiva. Por ejemplo, usaría [CreateIndexAsync](/dotnet/api/azure.search.documents.indexes.searchindexclient.createindexasync) en lugar de [CreateIndex](/dotnet/api/azure.search.documents.indexes.searchindexclient.createindex).

1. Agregue una definición de clase vacía al proyecto: **Hotel.cs**

1. Copie el código siguiente en **Hotel.cs** para definir la estructura de un documento de hotel. Los atributos del campo determinan cómo se utilizan en una aplicación. Por ejemplo, el atributo `IsFilterable` se tiene que asignar a cada campo que admita una expresión de filtro.

    ```csharp
    using System;
    using System.Text.Json.Serialization;
    using Azure.Search.Documents.Indexes;
    using Azure.Search.Documents.Indexes.Models;

    namespace AzureSearch.Quickstart
    {
        public partial class Hotel
        {
            [SimpleField(IsKey = true, IsFilterable = true)]
            public string HotelId { get; set; }

            [SearchableField(IsSortable = true)]
            public string HotelName { get; set; }

            [SearchableField(AnalyzerName = LexicalAnalyzerName.Values.EnLucene)]
            public string Description { get; set; }

            [SearchableField(AnalyzerName = LexicalAnalyzerName.Values.FrLucene)]
            [JsonPropertyName("Description_fr")]
            public string DescriptionFr { get; set; }

            [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public string Category { get; set; }

            [SearchableField(IsFilterable = true, IsFacetable = true)]
            public string[] Tags { get; set; }

            [SimpleField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public bool? ParkingIncluded { get; set; }

            [SimpleField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public DateTimeOffset? LastRenovationDate { get; set; }

            [SimpleField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public double? Rating { get; set; }

            [SearchableField]
            public Address Address { get; set; }
        }
    }
    ```

   En la biblioteca cliente Azure.Search.Documents puede usar [SearchableField](/dotnet/api/azure.search.documents.indexes.models.searchablefield) y [SimpleField](/dotnet/api/azure.search.documents.indexes.models.simplefield) para optimizar las definiciones de campo. Ambos son derivados de un [SearchField](/dotnet/api/azure.search.documents.indexes.models.searchfield) y pueden simplificar el código:

   + `SimpleField` puede ser cualquier tipo de datos, no permite búsquedas (se omite en las consultas de búsqueda de texto completo) y se puede recuperar (no está oculto). Otros atributos están desactivados de forma predeterminada, pero se pueden habilitar. Puede usar un `SimpleField` para los identificadores de documento o los campos que se usan solo en filtros, facetas o perfiles de puntuación. En caso de hacerlo, asegúrese de aplicar los atributos necesarios para el escenario, como `IsKey = true` para un identificador de documento. Para más información, consulte [SimpleFieldAttribute.cs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/src/Indexes/SimpleFieldAttribute.cs) en el código fuente.

   + `SearchableField` debe ser una cadena y siempre permite búsquedas y se puede recuperar. Otros atributos están desactivados de forma predeterminada, pero se pueden habilitar. Ya que este tipo de campo se puede buscar, admite sinónimos y el complemento completo de las propiedades del analizador. Para más información, consulte [SearchableFieldAttribute.cs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/src/Indexes/SearchableFieldAttribute.cs) en el código fuente.

   Tanto si usa la API básica de `SearchField`, como uno de los modelos auxiliares, debe habilitar explícitamente los atributos de filtro, faceta y ordenación. Por ejemplo, [IsFilterable](/dotnet/api/azure.search.documents.indexes.models.searchfield.isfilterable), [IsSortable](/dotnet/api/azure.search.documents.indexes.models.searchfield.issortable)y [IsFacetable](/dotnet/api/azure.search.documents.indexes.models.searchfield.isfacetable) se tienen que atribuir explícitamente, como se muestra en el ejemplo anterior. 

1. Agregue una segunda definición de clase vacía al proyecto: **Address.cs**.  Pegue el código siguiente en la clase.

   ```csharp
   using Azure.Search.Documents.Indexes;

    namespace AzureSearch.Quickstart
    {
        public partial class Address
        {
            [SearchableField(IsFilterable = true)]
            public string StreetAddress { get; set; }

            [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public string City { get; set; }

            [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public string StateProvince { get; set; }

            [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public string PostalCode { get; set; }

            [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public string Country { get; set; }
        }
    }
   ```

1. Cree dos clases más: **Hotel.Methods.cs** y **Address.Methods.cs** para las invalidaciones de ToString(). Estas clases se usan para representar los resultados de búsqueda en la salida de la consola.  El contenido de estas clases no se proporciona en este artículo, pero puede copiar el código de los [archivos de GitHub](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/quickstart/v11/AzureSearchQuickstart-v11).

1. En **Program.cs**, cree un objeto [SearchIndex](/dotnet/api/azure.search.documents.indexes.models.searchindex) y llame al método [CreateIndex](/dotnet/api/azure.search.documents.indexes.searchindexclient.createindex) para expresar el índice en el servicio de búsqueda. El índice también incluye [SearchSuggester](/dotnet/api/azure.search.documents.indexes.models.searchsuggester), que habilita la función autocompletar en los campos especificados.

   ```csharp
    // Create hotels-quickstart index
    private static void CreateIndex(string indexName, SearchIndexClient adminClient)
    {
        FieldBuilder fieldBuilder = new FieldBuilder();
        var searchFields = fieldBuilder.Build(typeof(Hotel));

        var definition = new SearchIndex(indexName, searchFields);

        var suggester = new SearchSuggester("sg", new[] { "HotelName", "Category", "Address/City", "Address/StateProvince" });
        definition.Suggesters.Add(suggester);

        adminClient.CreateOrUpdateIndex(definition);
    }
   ```

<a name="load-documents"></a>

## <a name="2---load-documents"></a>2 - Carga de documentos

Azure Cognitive Search busca en el contenido almacenado en el servicio. En este paso, el usuario cargará los documentos JSON que se ajustan al índice de hotel que acaba de crear.

En Azure Cognitive Search, los documentos de búsqueda son estructuras de datos que se pueden usar como entradas para la indexación y como salidas de las consultas. Tal como se obtienen del origen de datos externo, las entradas de documento pueden ser las filas de una base de datos, los blobs en Blob Storage o documentos JSON en el disco. En este ejemplo vamos a atajar e insertaremos los documentos JSON para cuatro hoteles en el propio código. 

Al cargar documentos, tiene que utilizar un objeto [IndexDocumentsBatch](/dotnet/api/azure.search.documents.models.indexdocumentsbatch-1). Un objeto `IndexDocumentsBatch` contiene una colección de [Actions](/dotnet/api/azure.search.documents.models.indexdocumentsbatch-1.actions) (Acciones), cada una de las cuales contiene un documento y una propiedad que indican a Azure Cognitive Search la acción que debe realizar ([carga, combinación, eliminación y mergeOrUpload](search-what-is-data-import.md#indexing-actions)).

1. En **Program.cs**, cree una matriz de documentos y las acciones de índice y pase la matriz a `IndexDocumentsBatch`. Los siguientes documentos se ajustan al índice hotels-quickstart, tal como se define mediante la clase hotel.

    ```csharp
    // Upload documents in a single Upload request.
    private static void UploadDocuments(SearchClient searchClient)
    {
        IndexDocumentsBatch<Hotel> batch = IndexDocumentsBatch.Create(
            IndexDocumentsAction.Upload(
                new Hotel()
                {
                    HotelId = "1",
                    HotelName = "Secret Point Motel",
                    Description = "The hotel is ideally located on the main commercial artery of the city in the heart of New York. A few minutes away is Time's Square and the historic centre of the city, as well as other places of interest that make New York one of America's most attractive and cosmopolitan cities.",
                    DescriptionFr = "L'hôtel est idéalement situé sur la principale artère commerciale de la ville en plein cœur de New York. A quelques minutes se trouve la place du temps et le centre historique de la ville, ainsi que d'autres lieux d'intérêt qui font de New York l'une des villes les plus attractives et cosmopolites de l'Amérique.",
                    Category = "Boutique",
                    Tags = new[] { "pool", "air conditioning", "concierge" },
                    ParkingIncluded = false,
                    LastRenovationDate = new DateTimeOffset(1970, 1, 18, 0, 0, 0, TimeSpan.Zero),
                    Rating = 3.6,
                    Address = new Address()
                    {
                        StreetAddress = "677 5th Ave",
                        City = "New York",
                        StateProvince = "NY",
                        PostalCode = "10022",
                        Country = "USA"
                    }
                }),
            IndexDocumentsAction.Upload(
                new Hotel()
                {
                    HotelId = "2",
                    HotelName = "Twin Dome Motel",
                    Description = "The hotel is situated in a  nineteenth century plaza, which has been expanded and renovated to the highest architectural standards to create a modern, functional and first-class hotel in which art and unique historical elements coexist with the most modern comforts.",
                    DescriptionFr = "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
                    Category = "Boutique",
                    Tags = new[] { "pool", "free wifi", "concierge" },
                    ParkingIncluded = false,
                    LastRenovationDate = new DateTimeOffset(1979, 2, 18, 0, 0, 0, TimeSpan.Zero),
                    Rating = 3.60,
                    Address = new Address()
                    {
                        StreetAddress = "140 University Town Center Dr",
                        City = "Sarasota",
                        StateProvince = "FL",
                        PostalCode = "34243",
                        Country = "USA"
                    }
                }),
            IndexDocumentsAction.Upload(
                new Hotel()
                {
                    HotelId = "3",
                    HotelName = "Triple Landscape Hotel",
                    Description = "The Hotel stands out for its gastronomic excellence under the management of William Dough, who advises on and oversees all of the Hotel’s restaurant services.",
                    DescriptionFr = "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
                    Category = "Resort and Spa",
                    Tags = new[] { "air conditioning", "bar", "continental breakfast" },
                    ParkingIncluded = true,
                    LastRenovationDate = new DateTimeOffset(2015, 9, 20, 0, 0, 0, TimeSpan.Zero),
                    Rating = 4.80,
                    Address = new Address()
                    {
                        StreetAddress = "3393 Peachtree Rd",
                        City = "Atlanta",
                        StateProvince = "GA",
                        PostalCode = "30326",
                        Country = "USA"
                    }
                }),
            IndexDocumentsAction.Upload(
                new Hotel()
                {
                    HotelId = "4",
                    HotelName = "Sublime Cliff Hotel",
                    Description = "Sublime Cliff Hotel is located in the heart of the historic center of Sublime in an extremely vibrant and lively area within short walking distance to the sites and landmarks of the city and is surrounded by the extraordinary beauty of churches, buildings, shops and monuments. Sublime Cliff is part of a lovingly restored 1800 palace.",
                    DescriptionFr = "Le sublime Cliff Hotel est situé au coeur du centre historique de sublime dans un quartier extrêmement animé et vivant, à courte distance de marche des sites et monuments de la ville et est entouré par l'extraordinaire beauté des églises, des bâtiments, des commerces et Monuments. Sublime Cliff fait partie d'un Palace 1800 restauré avec amour.",
                    Category = "Boutique",
                    Tags = new[] { "concierge", "view", "24-hour front desk service" },
                    ParkingIncluded = true,
                    LastRenovationDate = new DateTimeOffset(1960, 2, 06, 0, 0, 0, TimeSpan.Zero),
                    Rating = 4.60,
                    Address = new Address()
                    {
                        StreetAddress = "7400 San Pedro Ave",
                        City = "San Antonio",
                        StateProvince = "TX",
                        PostalCode = "78216",
                        Country = "USA"
                    }
                })
            );

        try
        {
            IndexDocumentsResult result = searchClient.IndexDocuments(batch);
        }
        catch (Exception)
        {
            // If for some reason any documents are dropped during indexing, you can compensate by delaying and
            // retrying. This simple demo just logs the failed document keys and continues.
            Console.WriteLine("Failed to index some of the documents: {0}");
        }
    }
    ```

    Una vez inicializado el objeto [IndexDocumentsBatch](/dotnet/api/azure.search.documents.models.indexdocumentsbatch-1), puede enviarlo al índice mediante una llamada a [IndexDocuments](/dotnet/api/azure.search.documents.searchclient.indexdocuments) en el objeto [SearchClient](/dotnet/api/azure.search.documents.searchclient).

1. Agregue las líneas siguientes a Main(). La carga de documentos se realiza mediante SearchClient, pero la operación también requiere derechos de administrador en el servicio, lo cual se suele asociar a SearchIndexClient. Una manera de configurar esta operación es obtener SearchClient mediante SearchIndexClient (adminClient en este ejemplo).

   ```csharp
    SearchClient ingesterClient = adminClient.GetSearchClient(indexName);

    // Load documents
    Console.WriteLine("{0}", "Uploading documents...\n");
    UploadDocuments(ingesterClient);
   ```

1. Dado que se trata de una aplicación de consola que ejecuta todos los comandos secuencialmente, agregue un tiempo de espera de 2 segundos entre la indexación y las consultas.

    ```csharp
    // Wait 2 seconds for indexing to complete before starting queries (for demo and console-app purposes only)
    Console.WriteLine("Waiting for indexing...\n");
    System.Threading.Thread.Sleep(2000);
    ```

    El retraso de 2 segundos compensa por la indexación, que es asincrónica, por lo que todos los documentos se pueden indexar antes de que se ejecutan las consultas. La codificación en un retraso solo suele ser necesaria en las pruebas, demostraciones y aplicaciones de ejemplo.

## <a name="3---search-an-index"></a>3 - Búsqueda en un índice

Puede obtener resultados de consulta tan pronto como se indexe el primer documento, pero las pruebas reales del índice deben esperar hasta que todos los documentos estén indexados.

Esta sección agrega dos funcionalidades: la lógica de consulta y los resultados. En el caso de las consultas, use el método [Search](/dotnet/api/azure.search.documents.searchclient.search). Este método admite texto de búsqueda (la cadena de consulta) además de otras [opciones](/dotnet/api/azure.search.documents.searchoptions).

La clase [SearchResults](/dotnet/api/azure.search.documents.models.searchresults-1) representa los resultados.

1. En **Program.cs**, cree un método **WriteDocuments**, que imprime los resultados de la búsqueda en la consola.

    ```csharp
    // Write search results to console
    private static void WriteDocuments(SearchResults<Hotel> searchResults)
    {
        foreach (SearchResult<Hotel> result in searchResults.GetResults())
        {
            Console.WriteLine(result.Document);
        }

        Console.WriteLine();
    }
    ```

1. Cree un método **RunQueries** para ejecutar consultas y devolver los resultados. Los resultados son objetos de Hotel. Este ejemplo muestra la firma del método y la primera consulta. Esta consulta muestra el parámetro Select, que permite crear el resultado con los campos seleccionados del documento.

    ```csharp
    // Run queries, use WriteDocuments to print output
    private static void RunQueries(SearchClient srchclient)
    {
        SearchOptions options;
        SearchResults<Hotel> response;

        Console.WriteLine("Query #1: Search on empty term '*' to return all documents, showing a subset of fields...\n");

        options = new SearchOptions()
        {
            IncludeTotalCount = true,
            Filter = "",
            OrderBy = { "" }
        };

        options.Select.Add("HotelId");
        options.Select.Add("HotelName");
        options.Select.Add("Address/City");

        response = srchclient.Search<Hotel>("*", options);
        WriteDocuments(response);
    ```

1. En la segunda consulta, busque un término, agregue un filtro que seleccione los documentos cuya clasificación sea mayor que 4 y, a continuación, ordene por clasificación en orden descendente. Un filtro es una expresión booleana que se evalúa en campos [IsFilterable](/dotnet/api/azure.search.documents.indexes.models.searchfield.isfilterable) en un índice. Las consultas de filtro incluyen o excluyen valores. Como tal, no hay puntuaciones de relevancia asociadas a una consulta de filtro. 

    ```csharp
    Console.WriteLine("Query #2: Search on 'hotels', filter on 'Rating gt 4', sort by Rating in descending order...\n");

    options = new SearchOptions()
    {
        Filter = "Rating gt 4",
        OrderBy = { "Rating desc" }
    };

    options.Select.Add("HotelId");
    options.Select.Add("HotelName");
    options.Select.Add("Rating");

    response = srchclient.Search<Hotel>("hotels", options);
    WriteDocuments(response);
    ```

1. La tercera consulta muestra searchFields, que se usa para delimitar una operación de búsqueda de texto completo a campos específicos.

    ```csharp
    Console.WriteLine("Query #3: Limit search to specific fields (pool in Tags field)...\n");

    options = new SearchOptions()
    {
        SearchFields = { "Tags" }
    };

    options.Select.Add("HotelId");
    options.Select.Add("HotelName");
    options.Select.Add("Tags");

    response = srchclient.Search<Hotel>("pool", options);
    WriteDocuments(response);
    ```

1. La cuarta consulta muestra facetas, que se pueden usar para estructurar una estructura de navegación por facetas. 

   ```csharp
    Console.WriteLine("Query #4: Facet on 'Category'...\n");

    options = new SearchOptions()
    {
        Filter = ""
    };

    options.Facets.Add("Category");

    options.Select.Add("HotelId");
    options.Select.Add("HotelName");
    options.Select.Add("Category");

    response = srchclient.Search<Hotel>("*", options);
    WriteDocuments(response);
   ```

1. En la quinta consulta se devuelve un documento específico. Una búsqueda de documentos es una respuesta típica al evento OnClick en un conjunto de resultados.

   ```csharp
    Console.WriteLine("Query #5: Look up a specific document...\n");

    Response<Hotel> lookupResponse;
    lookupResponse = srchclient.GetDocument<Hotel>("3");

    Console.WriteLine(lookupResponse.Value.HotelId);
   ```

1. La última consulta muestra la sintaxis de autocompletar; se simula una entrada de usuario parcial "sa" que se resuelve en dos posibles coincidencias en los sourceField asociados al que se ha definido en el índice.

   ```csharp
    Console.WriteLine("Query #6: Call Autocomplete on HotelName that starts with 'sa'...\n");

    var autoresponse = srchclient.Autocomplete("sa", "sg");
    WriteDocuments(autoresponse);
   ```

1. Agregue **RunQueries** a Main().

    ```csharp
    // Call the RunQueries method to invoke a series of queries
    Console.WriteLine("Starting queries...\n");
    RunQueries(srchclient);

    // End the program
    Console.WriteLine("{0}", "Complete. Press any key to end this program...\n");
    Console.ReadKey();
    ```

Las consultas anteriores muestran varias [maneras de buscar términos coincidentes en una consulta](search-query-overview.md#types-of-queries): búsqueda de texto completo, filtros y autocompletar.

Las búsquedas de texto completo y los filtros se realizan mediante el método [SearchClient.Search](/dotnet/api/azure.search.documents.searchclient.search). Se puede pasar una consulta de búsqueda en la cadena `searchText`, mientras que una expresión de filtro se puede pasar en la propiedad [Filter](/dotnet/api/azure.search.documents.searchoptions.filter) de la clase [SearchOptions](/dotnet/api/azure.search.documents.searchoptions). Para filtrar sin buscar, solo tiene que pasar `"*"` al parámetro `searchText` del método [Search](/dotnet/api/azure.search.documents.searchclient.search). Para buscar sin filtrar, deje la propiedad `Filter` sin establecer, o no pase ninguna instancia de `SearchOptions`.

## <a name="run-the-program"></a>Ejecución del programa

Presione F5 para recompilar la aplicación y ejecutar el programa en su totalidad. 

La salida incluye mensajes de [Console.WriteLIne](/dotnet/api/system.console.writeline), con la incorporación de la información de la consulta y los resultados.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando trabaje con su propia suscripción, es una buena idea al final de un proyecto identificar si todavía se necesitan los recursos que ha creado. Los recursos que se dejan en ejecución pueden costarle mucho dinero. Puede eliminar los recursos de forma individual o eliminar el grupo de recursos para eliminar todo el conjunto de recursos.

Puede encontrar y administrar recursos en el portal, mediante el vínculo **Todos los recursos** o **Grupos de recursos** en el panel de navegación izquierdo.

Si está usando un servicio gratuito, recuerde que está limitado a tres índices, indexadores y orígenes de datos. Puede eliminar elementos individuales en el portal para mantenerse por debajo del límite. 

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido de C# ha realizado un conjunto de tareas para crear un índice, cargar documentos en él y ejecutar consultas. En diferentes fases, hemos atajado para simplificar el código y mejorar la legibilidad y la comprensión. Ahora que está familiarizado con los conceptos básicos, pruebe el siguiente tutorial para llamar a las API de Cognitive Search en el contexto de una aplicación web.

> [!div class="nextstepaction"]
> [Tutorial: Incorporación de la funcionalidad de búsqueda a las aplicaciones web](tutorial-csharp-overview.md)
