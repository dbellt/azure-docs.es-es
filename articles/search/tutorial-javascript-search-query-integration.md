---
title: 'Tutorial de JavaScript: Aspectos destacados de la integración de búsqueda'
titleSuffix: Azure Cognitive Search
description: Descripción de las consultas de integración de funcionalidad de búsqueda del SDK de JavaScript utilizadas en un sitio web habilitado para la búsqueda mediante esta hoja de referencia rápida.
manager: nitinme
author: diberry
ms.author: diberry
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 03/09/2021
ms.custom: devx-track-js
ms.devlang: javascript
ms.openlocfilehash: f6c23c7b0870fe8caed1f82e8ce102539aa2e950
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107950232"
---
# <a name="4---javascript-search-integration-cheat-sheet"></a>4\. Hoja de información rápida de integración de funcionalidad de búsqueda de JavaScript

En las lecciones anteriores, agregó la búsqueda a una aplicación web estática. En esta lección se resaltan los pasos esenciales que establecen la integración. Si busca una hoja de referencia rápida sobre cómo integrar la búsqueda en su aplicación de JavaScript, en este artículo se explica lo que necesita saber.

La aplicación está disponible: 
* [Ejemplo](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/search-website)
* [Sitio web de demostración: aka.ms/azs-good-books](https://aka.ms/azs-good-books)

## <a name="azure-sdk-azuresearch-documents"></a>@azure/search-documents de SDK de Azure 

La aplicación de funciones usa el SDK de Azure para Cognitive Search:

* NPM: [@azure/search-documents](https://www.npmjs.com/package/@azure/search-documents)
* Documentación de referencia: [Biblioteca cliente](/javascript/api/overview/azure/search-documents-readme)

La aplicación de funciones se autentica a través del SDK para la API de Cognitive Search basada en la nube con el nombre del recurso, la clave de recurso y el nombre del índice. Los secretos se almacenan en la configuración estática de la aplicación web y se extraen a la función como variables de entorno. 

## <a name="configure-secrets-in-a-configuration-file"></a>Configuración de secretos en un archivo de configuración

:::code language="javascript" source="~/azure-search-javascript-samples/search-website/api/config.js" highlight="3,4" :::

## <a name="azure-function-search-the-catalog"></a>Azure Function: búsqueda en el catálogo

`Search` [API](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/search-website/api/Search/index.js) toma un término de búsqueda y busca en los documentos del índice de búsqueda, y devuelve una lista de coincidencias. 

El enrutamiento de Search API se incluye en los enlaces de [function.json](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/search-website/api/Search/function.json).

La función de Azure extrae la información de configuración de búsqueda y satisface la consulta.

:::code language="javascript" source="~/azure-search-javascript-samples/search-website/api/Search/index.js" highlight="4-9, 75" :::

## <a name="client-search-from-the-catalog"></a>Cliente: búsqueda en el catálogo

Llame a la función de Azure en el cliente de React con el código siguiente. 

:::code language="javascript" source="~/azure-search-javascript-samples/search-website/src/pages/Search/Search.js" highlight="40-51" :::

## <a name="azure-function-suggestions-from-the-catalog"></a>Azure Function: sugerencias del catálogo

`Suggest` [API](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/search-website/api/Suggest/index.js) toma un término de búsqueda mientras un usuario está escribiendo y sugiere términos de búsqueda como títulos de libros y autores en los documentos en el índice de búsqueda, y devuelve una pequeña lista de coincidencias. 

El proveedor de sugerencias de búsqueda, `sg`, se define en el [archivo de esquema](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/search-website/bulk-insert/good-books-index.json) usado durante la carga masiva.

El enrutamiento de Suggest API se incluye en los enlaces de [function.json](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/search-website/api/Suggest/function.json).

:::code language="javascript" source="~/azure-search-javascript-samples/search-website/api/Suggest/index.js" highlight="4-9, 21" :::

## <a name="client-suggestions-from-the-catalog"></a>Cliente: sugerencias del catálogo

Se llama a la función de Suggest API en la aplicación React en `\src\components\SearchBar\SearchBar.js` como parte de la inicialización de componentes:

:::code language="javascript" source="~/azure-search-javascript-samples/search-website/src/components/SearchBar/SearchBar.js" highlight="52-60" :::

## <a name="azure-function-get-specific-document"></a>Función de Azure: obtención de un documento específico 

`Lookup` [API](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/search-website/api/Lookup/index.js) toma un identificador y devuelve el objeto de documento del índice de búsqueda. 

El enrutamiento de Lookup API se incluye en los enlaces de [function.json](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/search-website/api/Lookup/function.json).

:::code language="javascript" source="~/azure-search-javascript-samples/search-website/api/Lookup/index.js" highlight="4-9, 17" :::

## <a name="client-get-specific-document"></a>Cliente: obtención de un documento específico 

Se llama a esta API de función en la aplicación React en `\src\pages\Details\Detail.js` como parte de la inicialización de componentes:

:::code language="javascript" source="~/azure-search-javascript-samples/search-website/src/pages/Details/Details.js" highlight="19-29" :::

## <a name="next-steps"></a>Pasos siguientes

* [Indexación de datos de Azure SQL](search-indexer-tutorial.md)
