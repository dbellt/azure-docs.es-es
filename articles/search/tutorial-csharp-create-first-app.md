---
title: Tutorial de C# para crear la primera aplicación
titleSuffix: Azure Cognitive Search
description: Aprenda a crear su primera aplicación de búsqueda con C# paso a paso. En el tutorial se proporcionan un vínculo de descarga a una aplicación en funcionamiento en GitHub y el proceso completo para crear la aplicación desde cero.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 02/26/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 68fa8b6d798bb649e7eb3646443bd5edd8f17147
ms.sourcegitcommit: 42ac9d148cc3e9a1c0d771bc5eea632d8c70b92a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2021
ms.locfileid: "109847968"
---
# <a name="tutorial-create-your-first-search-app-using-the-net-sdk"></a>Tutorial: Creación de la primera aplicación de búsqueda con el SDK de .NET

En este tutorial se muestra cómo crear una aplicación web que consulta y devuelve los resultados de un índice de búsqueda mediante Azure Cognitive Search y Visual Studio.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar entorno de desarrollo
> * Modelar estructuras de datos
> * Crear una página web para recopilar entradas de consulta y mostrar resultados
> * Definir un método de búsqueda
> * Prueba de la aplicación

También verá lo sencillo que es una llamada de búsqueda. Las instrucciones clave en el código que desarrollará están encapsuladas en las siguientes líneas.

```csharp
var options = new SearchOptions()
{
    // The Select option specifies fields for the result set
    options.Select.Add("HotelName");
    options.Select.Add("Description");
};

var searchResult = await _searchClient.SearchAsync<Hotel>(model.searchText, options).ConfigureAwait(false);
model.resultList = searchResult.Value.GetResults().ToList();
```

Solo una llamada consulta el índice y devuelve los resultados.

:::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-pool.png" alt-text="Búsqueda de *pool*" border="true":::

## <a name="overview"></a>Información general

En este tutorial se usa hotels-sample-index, que puede crear rápidamente en su propio servicio de búsqueda con el [inicio rápido de importación de datos](search-get-started-portal.md). El índice contiene datos de hoteles ficticios, disponibles como un origen de datos integrado en cada servicio de búsqueda.

En la primera lección de este tutorial se crea una página de búsqueda y una estructura de consulta básica, que se mejorará en las lecciones siguientes para incluir la paginación, las facetas y una experiencia de escritura anticipada.

En el proyecto siguiente se puede encontrar una versión finalizada del código:

* [1-basic-search-page (GitHub)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11/1-basic-search-page)

Este tutorial se ha actualizado para usar el paquete Azure.Search.Documents (versión 11). Para obtener una versión anterior del SDK de .NET, consulte el [código de ejemplo de Microsoft.Azure.Search (versión 10)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v10).

## <a name="prerequisites"></a>Prerrequisitos

* [Creación](search-create-service-portal.md) o [búsqueda de un servicio de búsqueda existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices)

* Creación del índice hotels-sample-index mediante las instrucciones del [inicio rápido de creación de un índice de búsqueda](search-get-started-portal.md)

* [Visual Studio](https://visualstudio.microsoft.com/)

* [Biblioteca cliente de Azure Cognitive Search (versión 11)](https://www.nuget.org/packages/Azure.Search.Documents/)

### <a name="install-and-run-the-project-from-github"></a>Instalar y ejecutar el proyecto de GitHub

Si quiere ir directamente a una aplicación en funcionamiento, siga estos pasos para descargar y ejecutar el código terminado.

1. Busque el ejemplo en GitHub: [Create first app](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11) (Crear la primera aplicación).

1. En la [carpeta raíz](https://github.com/Azure-Samples/azure-search-dotnet-samples), seleccione **Código**, seguido de **Clonar** o **Descargar archivo ZIP** para realizar su copia local privada del proyecto.

1. Con Visual Studio, vaya a la solución y ábrala para la página de búsqueda básica ("1-basic-search-page") y seleccione **Iniciar sin depurar** (o presione F5) para crear y ejecutar el programa.

1. Se trata de un índice de hoteles, por lo que debe escribir algunas palabras que se pueden usar para buscar hoteles (por ejemplo, "wifi", "vistas", "bar", "aparcamiento") y examinar los resultados.

    :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-wifi.png" alt-text="Búsqueda de *wifi*" border="true":::

Con suerte, este proyecto se ejecutará sin problemas y la aplicación web estará en funcionamiento. En esta aplicación única se incluyen muchos de los componentes esenciales para búsquedas más sofisticadas, por lo que es una buena idea repasarlo y volver a crearlo paso a paso. Las secciones siguientes tratan sobre estos pasos.

## <a name="set-up-a-development-environment"></a>Configurar entorno de desarrollo

Para crear este proyecto desde cero y, por lo tanto, reforzar los conceptos de Azure Cognitive Search, empiece por un proyecto de Visual Studio.

1. En Visual Studio, seleccione **Nuevo** > **Proyecto** y, luego, **Aplicación web ASP.NET Core**.

    :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-project1.png" alt-text="Crear un proyecto en la nube" border="true":::

1. Asigne un nombre al proyecto como "FirstSearchApp" y establezca la ubicación. Seleccione **Crear**.

1. Elija la plantilla de proyecto **Aplicación web (controlador de vista de modelos)** .

    :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-project2.png" alt-text="Crear un proyecto Modelo-Vista-Controlador" border="true":::

1. Instale la biblioteca cliente. En **Herramientas** > **Administrador de paquetes NuGet** > **Administrar paquetes NuGet para la solución...** , seleccione **Examinar** y busque "azure.search.documents". Instale **Azure.Search.Documents** (versión 11 o posterior) y acepte los contratos de licencia y las dependencias.

    :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-nuget-azure.png" alt-text="Uso de NuGet para agregar bibliotecas de Azure" border="true":::

### <a name="initialize-azure-cognitive-search"></a>Inicialización de Azure Cognitive Search

Para este ejemplo, usará datos de hotel disponibles públicamente. Estos datos son una colección arbitraria de 50 nombres de hotel y descripciones ficticios, creados exclusivamente para proporcionar datos de demostración. Para acceder a estos datos, especifique un nombre y una clave de API.

1. Abra **appsettings.js** y reemplace las líneas predeterminadas por la dirección URL del servicio de búsqueda (en el formato `https://<service-name>.search.windows.net`) y una [clave de administrador o de API de consulta](search-security-api-keys.md) del servicio de búsqueda. Dado que no es necesario crear ni actualizar un índice, puede usar la clave de consulta para este tutorial.

    ```csharp
    {
        "SearchServiceUri": "<YOUR-SEARCH-SERVICE-URI>",
        "SearchServiceQueryApiKey": "<YOUR-SEARCH-SERVICE-API-KEY>"
    }
    ```

1. En Explorador de soluciones, seleccione el archivo y, en Propiedades, cambie la configuración de **Copiar en el directorio de salida** a **Copiar si es más reciente**.

    :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-copy-if-newer.png" alt-text="Copiar la configuración de la aplicación a la salida" border="true":::

## <a name="model-data-structures"></a>Modelar estructuras de datos

Los modelos (clases de C#) se usan para comunicar datos entre el cliente (vista), el servidor (controlador) y también la nube de Azure mediante la arquitectura MVC (modelo, vista, controlador). Normalmente, estos modelos reflejan la estructura de los datos a los que se accede.

En este paso, modelará las estructuras de datos del índice de búsqueda, así como la cadena de búsqueda usada en las comunicaciones de controlador y vista. En el índice de hoteles, cada hotel tiene muchas habitaciones y una dirección con varios componentes. En conjunto, la representación completa de un hotel es una estructura de datos jerárquica y anidada. Necesitará tres clases para crear cada componente.

El conjunto de clases **Hotel**, **Address** y **Room** se conoce como [*tipos complejos*](search-howto-complex-data-types.md), una característica importante de Azure Cognitive Search. Los tipos complejos pueden tener una profundidad de varios niveles de clases y subclases, y permiten la representación de estructuras de datos mucho más complejas que con el uso de *tipos simples* (una clase que contiene solo miembros primitivos).

1. En el Explorador de soluciones, haga clic con el botón derecho en **Modelos** > **Agregar** > **Nuevo elemento**.

1. Seleccione **Clase** y asigne al elemento el nombre Hotel.cs. Reemplace todo el contenido de Hotel.cs por el código siguiente. Observe los miembros **Address** y **Room** de la clase; estos campos son clases en sí, por lo que necesitaremos modelos para ellos también.

    ```csharp
    using Azure.Search.Documents.Indexes;
    using Azure.Search.Documents.Indexes.Models;
    using Microsoft.Spatial;
    using System;
    using System.Text.Json.Serialization;

    namespace FirstAzureSearchApp.Models
    {
        public partial class Hotel
        {
            [SimpleField(IsFilterable = true, IsKey = true)]
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

            public Address Address { get; set; }

            [SimpleField(IsFilterable = true, IsSortable = true)]
            public GeographyPoint Location { get; set; }

            public Room[] Rooms { get; set; }
        }
    }
    ```

1. Repita el mismo proceso de creación de un modelo para la clase **Address**, y asigne el nombre Address.cs al archivo. Reemplace el contenido por el siguiente.

    ```csharp
    using Azure.Search.Documents.Indexes;

    namespace FirstAzureSearchApp.Models
    {
        public partial class Address
        {
            [SearchableField]
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

1. Nuevamente, siga el mismo proceso para crear la clase **Room**, asignando el nombre Room.cs al archivo.

    ```csharp
    using Azure.Search.Documents.Indexes;
    using Azure.Search.Documents.Indexes.Models;
    using System.Text.Json.Serialization;

    namespace FirstAzureSearchApp.Models
    {
        public partial class Room
        {
            [SearchableField(AnalyzerName = LexicalAnalyzerName.Values.EnMicrosoft)]
            public string Description { get; set; }

            [SearchableField(AnalyzerName = LexicalAnalyzerName.Values.FrMicrosoft)]
            [JsonPropertyName("Description_fr")]
            public string DescriptionFr { get; set; }

            [SearchableField(IsFilterable = true, IsFacetable = true)]
            public string Type { get; set; }

            [SimpleField(IsFilterable = true, IsFacetable = true)]
            public double? BaseRate { get; set; }

            [SearchableField(IsFilterable = true, IsFacetable = true)]
            public string BedOptions { get; set; }

            [SimpleField(IsFilterable = true, IsFacetable = true)]
            public int SleepsCount { get; set; }

            [SimpleField(IsFilterable = true, IsFacetable = true)]
            public bool? SmokingAllowed { get; set; }

            [SearchableField(IsFilterable = true, IsFacetable = true)]
            public string[] Tags { get; set; }
        }
    }
    ```

1. El último modelo que creará en este tutorial es una clase denominada **SearchData** y representa la entrada del usuario (**searchText**) y la salida de la búsqueda (**resultList**). El tipo de la salida es fundamental, **SearchResults&lt;Hotel&gt;** , ya que este tipo coincide exactamente con los resultados de la búsqueda y necesita pasar esta referencia por la vista. Reemplace la plantilla predeterminada por el siguiente código.

    ```csharp
    using Azure.Search.Documents.Models;

    namespace FirstAzureSearchApp.Models
    {
        public class SearchData
        {
            // The text to search for.
            public string searchText { get; set; }

            // The list of results.
            public SearchResults<Hotel> resultList;
        }
    }
    ```

## <a name="create-a-web-page"></a>Crear una aplicación web

Las plantillas de proyecto incluyen varias vistas de cliente ubicadas en la carpeta **Views**. Las vistas exactas dependen de la versión de .NET Core que use (en este ejemplo, se usa la versión 3.1). En este tutorial, modificará **Index.cshtml** para incluir los elementos de una página de búsqueda.

Elimine el contenido de Index.cshtml en su totalidad y vuelva a generar el archivo con los pasos siguientes.

1. En el tutorial se usan dos imágenes pequeñas en la vista: un logotipo de Azure y un icono del ampliador de la búsqueda (azure-logo.png y search.png). Copie las imágenes del proyecto de GitHub a la carpeta **wwwroot/images** del proyecto.

1. La primera línea de Index.cshtml debe hacer referencia al modelo que se va a usar para comunicar datos entre el cliente (vista) y el servidor (controlador), que es el modelo **SearchData** que se creó anteriormente. Agregue esta línea al archivo Index.cshtml.

    ```csharp
    @model FirstAzureSearchApp.Models.SearchData
    ```

1. La práctica habitual es escribir un título para la vista, por lo que las líneas siguientes deberían ser:

    ```csharp
    @{
        ViewData["Title"] = "Home Page";
    }
    ```

1. Después del título, escriba una referencia a una hoja de estilos HTML que creará en breve.

    ```csharp
    <head>
        <link rel="stylesheet" href="~/css/hotels.css" />
    </head>
    ```

1. El cuerpo de la vista controla dos casos de uso. En primer lugar, debe proporcionar una página vacía para su uso por primera vez, antes de escribir cualquier texto de búsqueda. En segundo lugar, debe controlar los resultados, además del cuadro de texto de búsqueda, para consultas repetidas.

   Para controlar ambos casos, es necesario comprobar si el modelo proporcionado a la vista es null. Un modelo null indica el primer caso de uso (la ejecución inicial de la aplicación). Agregue lo siguiente al archivo Index.cshtml y lea los comentarios.

    ```csharp
    <body>
    <h1 class="sampleTitle">
        <img src="~/images/azure-logo.png" width="80" />
        Hotels Search
    </h1>

    @using (Html.BeginForm("Index", "Home", FormMethod.Post))
    {
        // Display the search text box, with the search icon to the right of it.
        <div class="searchBoxForm">
            @Html.TextBoxFor(m => m.searchText, new { @class = "searchBox" }) <input class="searchBoxSubmit" type="submit" value="">
        </div>

        @if (Model != null)
        {
            // Show the result count.
            <p class="sampleText">
                @Model.resultList.TotalCount Results
            </p>

            var results = Model.resultList.GetResults().ToList();

            @for (var i = 0; i < results.Count; i++)
            {
                // Display the hotel name and description.
                @Html.TextAreaFor(m => results[i].Document.HotelName, new { @class = "box1" })
                @Html.TextArea($"desc{i}", results[i].Document.Description, new { @class = "box2" })
            }
        }
    }
    </body>
    ```

1. Agregue la hoja de estilos. En Visual Studio, en el menú **Archivo**> **Nuevo** > **Archivo**, seleccione **Hoja de estilos** (con la opción **General** resaltada).

   Reemplace el código predeterminado por el siguiente. No entraremos en más detalles en este archivo; los estilos son estilos HTML estándar.

    ```html
    textarea.box1 {
        width: 648px;
        height: 30px;
        border: none;
        background-color: azure;
        font-size: 14pt;
        color: blue;
        padding-left: 5px;
    }

    textarea.box2 {
        width: 648px;
        height: 100px;
        border: none;
        background-color: azure;
        font-size: 12pt;
        padding-left: 5px;
        margin-bottom: 24px;
    }

    .sampleTitle {
        font: 32px/normal 'Segoe UI Light',Arial,Helvetica,Sans-Serif;
        margin: 20px 0;
        font-size: 32px;
        text-align: left;
    }

    .sampleText {
        font: 16px/bold 'Segoe UI Light',Arial,Helvetica,Sans-Serif;
        margin: 20px 0;
        font-size: 14px;
        text-align: left;
        height: 30px;
    }

    .searchBoxForm {
        width: 648px;
        box-shadow: 0 0 0 1px rgba(0,0,0,.1), 0 2px 4px 0 rgba(0,0,0,.16);
        background-color: #fff;
        display: inline-block;
        border-collapse: collapse;
        border-spacing: 0;
        list-style: none;
        color: #666;
    }

    .searchBox {
        width: 568px;
        font-size: 16px;
        margin: 5px 0 1px 20px;
        padding: 0 10px 0 0;
        border: 0;
        max-height: 30px;
        outline: none;
        box-sizing: content-box;
        height: 35px;
        vertical-align: top;
    }

    .searchBoxSubmit {
        background-color: #fff;
        border-color: #fff;
        background-image: url(/images/search.png);
        background-repeat: no-repeat;
        height: 20px;
        width: 20px;
        text-indent: -99em;
        border-width: 0;
        border-style: solid;
        margin: 10px;
        outline: 0;
    }
    ```

1. Guarde el archivo de hoja de estilos como hotels.css, en la carpeta **wwwroot/css**, junto con el archivo site.css predeterminado.

Esto completa nuestra vista. En este momento, se completan los modelos y las vistas. Solo se permite al controlador vincular todo.

## <a name="define-methods"></a>Definición de métodos

En este paso, modifique el contenido de **Home Controller**.

1. Abra el archivo HomeController.cs y reemplace las instrucciones **using** por lo siguiente.

    ```csharp
    using Azure;
    using Azure.Search.Documents;
    using Azure.Search.Documents.Indexes;
    using FirstAzureSearchApp.Models;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Configuration;
    using System;
    using System.Diagnostics;
    using System.Linq;
    using System.Threading.Tasks;
    ```

### <a name="add-index-methods"></a>Agregar métodos de índice

En una aplicación MVC, el método **Index()** es un método de acción predeterminado para cualquier controlador. Con él se puede abrir la página HTML de índice. El método predeterminado, que no toma ningún parámetro, se usa en este tutorial para el caso de uso de inicio de la aplicación: representación de una página de búsqueda vacía.

En esta sección, ampliaremos el método para que admita un segundo caso de uso: la representación de la página cuando un usuario ha escrito texto de búsqueda. Para admitir este caso, el método de índice se amplía para tomar un modelo como parámetro.

1. Agregue el método siguiente, después del método **Index()** predeterminado.

    ```csharp
        [HttpPost]
        public async Task<ActionResult> Index(SearchData model)
        {
            try
            {
                // Ensure the search string is valid.
                if (model.searchText == null)
                {
                    model.searchText = "";
                }

                // Make the Azure Cognitive Search call.
                await RunQueryAsync(model);
            }

            catch
            {
                return View("Error", new ErrorViewModel { RequestId = "1" });
            }
            return View(model);
        }
    ```

    Observe la declaración **async** del método y la llamada **await** a **RunQueryAsync**. Estas palabras clave se encargan de realizar las llamadas asincrónicas y así evitar el bloqueo de subprocesos en el servidor.

    El bloque **catch** usa el modelo de error que se creó automáticamente de manera predeterminada.

### <a name="note-the-error-handling-and-other-default-views-and-methods"></a>Observar el control de errores y otras vistas y métodos predeterminados

Según la versión de .NET Core que usa, de manera predeterminada se crea un conjunto diferente de vistas predeterminadas. Para .NET Core 3.1, las vistas predeterminadas son Índice, Privacidad y Error. Puede ver estas páginas predeterminadas al ejecutar la aplicación y examinar cómo se controlan en el controlador.

Va a probar la vista Error más adelante en este tutorial.

En el ejemplo de GitHub, se eliminan las vistas sin usar y sus acciones asociadas.

### <a name="add-the-runqueryasync-method"></a>Agregar el método RunQueryAsync

La llamada a Azure Cognitive Search se encapsula en nuestro método **RunQueryAsync**.

1. En primer lugar, agregue algunas variables estáticas para configurar el servicio de Azure y una llamada para iniciarlas.

    ```csharp
        private static SearchClient _searchClient;
        private static SearchIndexClient _indexClient;
        private static IConfigurationBuilder _builder;
        private static IConfigurationRoot _configuration;

        private void InitSearch()
        {
            // Create a configuration using appsettings.json
            _builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
            _configuration = _builder.Build();

            // Read the values from appsettings.json
            string searchServiceUri = _configuration["SearchServiceUri"];
            string queryApiKey = _configuration["SearchServiceQueryApiKey"];

            // Create a service and index client.
            _indexClient = new SearchIndexClient(new Uri(searchServiceUri), new AzureKeyCredential(queryApiKey));
            _searchClient = _indexClient.GetSearchClient("hotels");
        }
    ```

2. Ahora, agregue el método **RunQueryAsync** en sí.

    ```csharp
    private async Task<ActionResult> RunQueryAsync(SearchData model)
    {
        InitSearch();

        var options = new SearchOptions() 
        { 
            IncludeTotalCount = true
        };

        // Enter Hotel property names into this list so only these values will be returned.
        // If Select is empty, all values will be returned, which can be inefficient.
        options.Select.Add("HotelName");
        options.Select.Add("Description");

        // For efficiency, the search call should be asynchronous, so use SearchAsync rather than Search.
        model.resultList = await _searchClient.SearchAsync<Hotel>(model.searchText, options).ConfigureAwait(false);          

        // Display the results.
        return View("Index", model);
    }
    ```

    En este método, primero asegúrese de que la configuración de Azure se haya iniciado y luego configure algunas opciones de búsqueda. La opción **Seleccionar** especifica qué campos se devuelven en los resultados y, por tanto, coinciden con los nombres de propiedad de la clase **Hotel**. Si omite **Seleccionar**, se devolverán todos los campos visibles, lo cual puede ser ineficaz si solo está interesado en un subconjunto de todos los campos posibles.

    La llamada asincrónica a Search formula la solicitud (modelada como **searchText**) y la respuesta (modelada como **searchResult**). Si está depurando este código, la clase **SearchResult** es una buena candidata para establecer un punto de interrupción si necesita examinar el contenido de **model.resultList**. Verá que este proceso es intuitivo, ya que le proporciona los datos que se piden y no mucho más.

### <a name="test-the-app"></a>Prueba de la aplicación

Comprobemos si la aplicación se ejecuta correctamente.

1. Seleccione **Depurar** > **Iniciar sin depurar** o presione **F5**. Si la aplicación se ejecuta según lo esperado, debe obtener la vista Índice inicial.

     :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-index.png" alt-text="Abrir la aplicación" border="true":::

1. Escriba una cadena de consulta, como "beach" (o cualquier texto que le venga a la mente) y haga clic en el icono de búsqueda para enviar la solicitud.

     :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-beach.png" alt-text="Búsqueda de *beach*" border="true":::

1. Pruebe a escribir "five star". Observe que esta consulta no devuelve ningún resultado. Una búsqueda más sofisticada sería tratar "five star" como sinónimo de "luxury" y hacer que se devuelvan esos resultados. La posibilidad de admitir [sinónimos](search-synonyms.md) está disponible en Azure Cognitive Search, pero no se describe en esta serie de tutoriales.

1. Pruebe a escribir "hot" como texto de búsqueda. _No_ devuelve entradas con la palabra "hotel" en ellas. Nuestra búsqueda solo busca palabras completas, aunque se devuelven algunos resultados.

1. Pruebe otras palabras: "pool", "sunshine", "view" y lo que sea. Verá que Azure Cognitive Search funciona en su nivel más sencillo, pero no por ello menos convincente.

## <a name="test-edge-conditions-and-errors"></a>Probar las condiciones de borde y los errores

Es importante comprobar que las características de control de errores funcionan del modo esperado, incluso cuando todo funciona perfectamente. 

1. En el método **Index**, después de la llamada **try {** , escriba la línea **Throw new Exception()** . Esta excepción fuerza un error cuando se realizan búsquedas de texto.

2. Ejecute la aplicación, escriba "bar" como texto de búsqueda y haga clic en el icono de búsqueda. La excepción debería provocar una vista de error.

     :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-error.png" alt-text="Forzar un error" border="true":::

    > [!Important]
    > Se considera un riesgo de seguridad devolver números de error interno en las páginas de error. Si la aplicación está pensada para uso general, investigue la seguridad y los procedimientos recomendados sobre lo que se debe devolver cuando se produce un error.

3. Cuando esté satisfecho del funcionamiento previsto del control de errores, quite **Throw new Exception()** .

## <a name="takeaways"></a>Puntos clave

Tenga en cuenta las siguientes conclusiones de este proyecto:

* Una llamada a Azure Cognitive Search es concisa, y los resultados son fáciles de interpretar.
* Las llamadas asincrónicas agregan una pequeña cantidad de complejidad al controlador, pero son el procedimiento recomendado si va a desarrollar aplicaciones de calidad.
* Esta aplicación realizó una búsqueda de texto sencillo, definida por lo que se configura en **searchOptions**. Sin embargo, esta clase única se puede rellenar con muchos miembros que agregan sofisticación a una búsqueda. No hace falta mucho trabajo adicional para hacer que esta aplicación sea mucho más potente.

## <a name="next-steps"></a>Pasos siguientes

Para mejorar la experiencia del usuario, agregue más características, especialmente, la paginación (ya sea mediante números de página o desplazamiento infinito), así como las opciones de autocompletar y sugerencias. También puede tener en cuenta opciones de búsqueda más sofisticadas (por ejemplo, búsquedas geográficas de hoteles dentro de un radio especificado de un punto dado y el ordenamiento de los resultados de la búsqueda).

Los tutoriales restantes tratan sobre los pasos siguientes. Comencemos por la paginación.

> [!div class="nextstepaction"]
> [Tutorial de C#: Paginación de los resultados de Azure Cognitive Search](tutorial-csharp-paging.md)
