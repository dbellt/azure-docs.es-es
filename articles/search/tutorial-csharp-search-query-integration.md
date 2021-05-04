---
title: 'Tutorial de .NET: Aspectos destacados de la integración de funcionalidad de búsqueda'
titleSuffix: Azure Cognitive Search
description: Descripción de las consultas de integración de funcionalidad de búsqueda del SDK de .NET utilizadas en un sitio web habilitado para la búsqueda mediante esta hoja de referencia rápida.
manager: nitinme
author: diberry
ms.author: diberry
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 04/23/2021
ms.custom: devx-track-csharp
ms.devlang: dotnet
ms.openlocfilehash: 504116f9bbed178889cd2e6d370c0c912da0ad00
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107981991"
---
# <a name="4---net-search-integration-cheat-sheet"></a>4\. Hoja de información rápida de integración de funcionalidad de búsqueda de .NET

En las lecciones anteriores, agregó la búsqueda a una aplicación web estática. En esta lección se resaltan los pasos esenciales que establecen la integración. Si busca una hoja de referencia rápida sobre cómo integrar la funcionalidad de búsqueda en su aplicación web, en este artículo se explica lo que debe saber.

La aplicación está disponible: 
* [Ejemplo](https://github.com/azure-samples/azure-search-dotnet-samples/tree/master/search-website)
* [Sitio web de demostración: aka.ms/azs-good-books](https://aka.ms/azs-good-books)

## <a name="azure-sdk-azuresearchdocuments"></a>Azure SDK Azure.Search.Documents

La aplicación de funciones usa el SDK de Azure para Cognitive Search:

* NuGet: [Azure.Search.Documents](https://www.nuget.org/packages/Azure.Search.Documents/)
* Documentación de referencia: [Biblioteca cliente](/dotnet/api/overview/azure/search)

La aplicación de funciones se autentica a través del SDK para la API de Cognitive Search basada en la nube con el nombre del recurso, la clave de recurso y el nombre del índice. Los secretos se almacenan en la configuración estática de la aplicación web y se extraen a la función como variables de entorno. 

## <a name="configure-secrets-in-a-localsettingsjson-file"></a>Configuración de secretos en un archivo local.settings.jsen

1. Cree un nuevo archivo llamado `local.settings.json` en `./api/` y copie el siguiente objeto JSON en el archivo.

    ```json
    {
      "IsEncrypted": false,
      "Values": {
        "AzureWebJobsStorage": "",
        "FUNCTIONS_WORKER_RUNTIME": "dotnet",
        "SearchApiKey": "YOUR_SEARCH_QUERY_KEY",
        "SearchServiceName": "YOUR_SEARCH_RESOURCE_NAME",
        "SearchIndexName": "good-books"
      }
    }
    ```

1. Cambie lo siguiente para sus propios valores de recursos de búsqueda: 
    * YOUR_SEARCH_RESOURCE_NAME
    * YOUR_SEARCH_QUERY_KEY

## <a name="azure-function-search-the-catalog"></a>Azure Function: búsqueda en el catálogo

`Search` [API](https://github.com/Azure-Samples/azure-search-dotnet-samples/blob/master/search-website/api/Search.cs) toma un término de búsqueda y busca en los documentos del índice de búsqueda, y devuelve una lista de coincidencias. 

La función de Azure extrae la información de configuración de búsqueda y satisface la consulta.

:::code language="csharp" source="~/azure-search-dotnet-samples/search-website/api/Search.cs" highlight="22-24, 55" :::

## <a name="client-search-from-the-catalog"></a>Cliente: búsqueda en el catálogo

Llame a la función de Azure en el cliente de React con el código siguiente. 

:::code language="javascript" source="~/azure-search-dotnet-samples/search-website/src/pages/Search/Search.js" highlight="40-51" :::

## <a name="azure-function-suggestions-from-the-catalog"></a>Azure Function: sugerencias del catálogo

`Suggest` [API](https://github.com/Azure-Samples/azure-search-dotnet-samples/blob/master/search-website/api/Suggest.cs) toma un término de búsqueda mientras un usuario está escribiendo y sugiere términos de búsqueda como títulos de libros y autores en los documentos en el índice de búsqueda, y devuelve una pequeña lista de coincidencias. 

El proveedor de sugerencias de búsqueda, `sg`, se define en el [archivo de esquema](https://github.com/Azure-Samples/azure-search-dotnet-samples/blob/master/search-website/bulk-insert/BookSearchIndex.cs) usado durante la carga masiva.

:::code language="csharp" source="~/azure-search-dotnet-samples/search-website/api/Suggest.cs" highlight="21-23, 50-52" :::

## <a name="client-suggestions-from-the-catalog"></a>Cliente: sugerencias del catálogo

Se llama a la función de Suggest API en la aplicación React en `\src\components\SearchBar\SearchBar.js` como parte de la inicialización de componentes:

:::code language="javascript" source="~/azure-search-dotnet-samples/search-website/src/components/SearchBar/SearchBar.js" highlight="52-60" :::

## <a name="azure-function-get-specific-document"></a>Función de Azure: obtención de un documento específico 

`Lookup` [API](https://github.com/Azure-Samples/azure-search-dotent-samples/blob/master/search-website/api/Lookup.cs) toma un identificador y devuelve el objeto de documento del índice de búsqueda. 

:::code language="csharp" source="~/azure-search-dotnet-samples/search-website/api/Lookup.cs" highlight="19-21, 42" :::

## <a name="client-get-specific-document"></a>Cliente: obtención de un documento específico 

Se llama a esta API de función en la aplicación React en `\src\pages\Details\Detail.js` como parte de la inicialización de componentes:

:::code language="javascript" source="~/azure-search-dotnet-samples/search-website/src/pages/Details/Details.js" highlight="19-29" :::

## <a name="c-models-to-support-function-app"></a>Modelos de C# para admitir la aplicación de funciones

Los modelos siguientes se usan para dar cabida a las funciones de esta aplicación.

:::code language="csharp" source="~/azure-search-dotnet-samples/search-website/api/Models.cs" :::

## <a name="next-steps"></a>Pasos siguientes

* [Indexación de datos de Azure SQL](search-indexer-tutorial.md)
