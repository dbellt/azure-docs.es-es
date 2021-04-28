---
title: Inicio rápido de la biblioteca cliente para C# de Bing Custom Search
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 02/27/2020
ms.author: aahi
ms.openlocfilehash: 504057be85902307a121a57dd421254d94167341
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107879674"
---
Introducción a la biblioteca cliente de Bing Custom Search para C#. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas. Bing Custom Search API le permite crear experiencias de búsqueda a medida y sin anuncios para los temas que le interesan. El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7/BingCustomWebSearch).

Use la biblioteca cliente de Bing Custom Search para C# para:
* Buscar los resultados de la búsqueda en la Web, desde la instancia de Bing Custom Search.

[Documentación de referencia](/dotnet/api/overview/azure/cognitiveservices/bing-custom-search-readme) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Search.BingCustomSearch) | [Paquete (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.CustomSearch/1.2.0) | [Ejemplos](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples)


## <a name="prerequisites"></a>Prerrequisitos

- Una instancia de Bing Custom Search. Consulte [Quickstart: Creación de la primera instancia de Bing Custom Search](../../quick-start.md) para obtener más información.
- Microsoft [.NET Core](https://dotnet.microsoft.com/download)
- Cualquier edición de [Visual Studio 2017 o versiones posteriores](https://www.visualstudio.com/downloads/).
- Si usa Linux/MacOS, esta aplicación puede ejecutarse con [Mono](https://www.mono-project.com/).
- El paquete [Bing Custom Search](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.CustomSearch/1.2.0) de NuGet. 
    - En el **Explorador de soluciones** de Visual Studio, haga clic con el botón derecho en el proyecto y seleccione **Administrar paquetes NuGet** en el menú. Instale el paquete `Microsoft.Azure.CognitiveServices.Search.CustomSearch`. Al instalar el paquete NuGet Custom Search, también se instalarán los ensamblados siguientes:
        - Microsoft.Rest.ClientRuntime
        - Microsoft.Rest.ClientRuntime.Azure
        - Newtonsoft.Json

[!INCLUDE [cognitive-services-bing-custom-search-prerequisites](~/includes/cognitive-services-bing-custom-search-signup-requirements.md)]


## <a name="create-and-initialize-the-application"></a>Creación e inicialización de la aplicación

1. Cree una aplicación de consola en C# mediante Visual Studio. Agregue los siguientes paquetes al proyecto.

    ```csharp
    using System;
    using System.Linq;
    using Microsoft.Azure.CognitiveServices.Search.CustomSearch;
    ```

2. En el método Main de la aplicación, cree una instancia del cliente de búsqueda con la clave de API.

    ```csharp
    var client = new CustomSearchAPI(new ApiKeyServiceClientCredentials("YOUR-SUBSCRIPTION-KEY"));
    ```

## <a name="send-the-search-request-and-receive-a-response"></a>Envío de la solicitud de búsqueda y recepción de la respuesta
    
1. Envíe una consulta de búsqueda con el método `SearchAsync()` del cliente y guarde la respuesta. No olvide reemplazar `YOUR-CUSTOM-CONFIG-ID` por el identificador de configuración de la instancia (encontrará el identificador en el [portal de Bing Custom Search](https://www.customsearch.ai/)). En este ejemplo se busca "Xbox".

    ```csharp
    // This will look up a single query (Xbox).
    var webData = client.CustomInstance.SearchAsync(query: "Xbox", customConfig: Int32.Parse("YOUR-CUSTOM-CONFIG-ID")).Result;
    ```

2. El método `SearchAsync()` devuelve un objeto `WebData`. Use el objeto para iterar por cualquier `WebPages` que se encuentre. Este código busca el primer resultado de página web e imprime los valores de `Name` y `URL` de la página web.

    ```csharp
    if (webData?.WebPages?.Value?.Count > 0)
    {
        // find the first web page
        var firstWebPagesResult = webData.WebPages.Value.FirstOrDefault();

        if (firstWebPagesResult != null)
        {
            Console.WriteLine("Number of webpage results {0}", webData.WebPages.Value.Count);
            Console.WriteLine("First web page name: {0} ", firstWebPagesResult.Name);
            Console.WriteLine("First web page URL: {0} ", firstWebPagesResult.Url);
        }
        else
        {
            Console.WriteLine("Couldn't find web results!");
        }
    }
    else
    {
        Console.WriteLine("Didn't see any Web data..");
    }
    ```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una página web de Custom Search](../../tutorials/custom-search-web-page.md)
