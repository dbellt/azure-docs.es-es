---
title: 'Tutorial de Python: Aspectos destacados de la integración de funcionalidad de búsqueda'
titleSuffix: Azure Cognitive Search
description: Descripción de las consultas de integración de funcionalidad de búsqueda del SDK de Python utilizadas en un sitio web habilitado para la búsqueda mediante esta hoja de referencia rápida.
manager: nitinme
author: diberry
ms.author: diberry
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 05/25/2021
ms.custom: devx-track-python
ms.devlang: python
ms.openlocfilehash: a8f1078e714bd88d2b2e9911a9e1708c21aa91b2
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110580633"
---
# <a name="4---python-search-integration-cheat-sheet"></a>4\. Hoja de información rápida de integración de funcionalidad de búsqueda de Python

En las lecciones anteriores, agregó la búsqueda a una aplicación web estática. En esta lección se resaltan los pasos esenciales que establecen la integración. Si busca una hoja de referencia rápida sobre cómo integrar la funcionalidad de búsqueda en su aplicación de Python, en este artículo se explica lo que debe saber.

La aplicación está disponible: 
* [Ejemplo](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/search-website)
* [Sitio web de demostración: aka.ms/azs-good-books](https://aka.ms/azs-good-books)

## <a name="azure-sdk-azure-search-documents"></a>azure-search-documents del SDK de Azure

La aplicación de funciones usa el SDK de Azure para Cognitive Search:

* [azure-search-documents de paquete PYPI](https://pypi.org/project/azure-search-documents/)
* [Documentación de referencia](/python/api/azure-search-documents)

La aplicación de funciones se autentica a través del SDK para la API de Cognitive Search basada en la nube con el nombre del recurso, la clave de API y el nombre del índice. Los secretos se almacenan en la configuración estática de la aplicación web y se extraen a la función como variables de entorno. 

## <a name="configure-secrets-in-a-configuration-file"></a>Configuración de secretos en un archivo de configuración

Las variables de entorno de configuración de la aplicación de Azure Functions se extraen de un archivo, `__init__.py`, compartido entre las tres funciones de API. 

:::code language="python" source="~/azure-search-python-samples/search-website/api/shared_code/__init__.py" highlight="6-9" :::

## <a name="azure-function-search-the-catalog"></a>Azure Function: búsqueda en el catálogo

Search [API](https://github.com/Azure-Samples/azure-search-python-samples/blob/master/search-website/api/Search/__init__.py) toma un término de búsqueda y busca en los documentos del índice de búsqueda, y devuelve una lista de coincidencias. 

El enrutamiento de Search API se incluye en los enlaces de [function.json](https://github.com/Azure-Samples/azure-search-python-samples/blob/master/search-website/api/Search/function.json).

Azure Functions extrae la información de configuración de búsqueda y satisface la consulta.

:::code language="python" source="~/azure-search-python-samples/search-website/api/Search/__init__.py" highlight="8-18, 122" :::

## <a name="client-search-from-the-catalog"></a>Cliente: búsqueda en el catálogo

Llame a la función de Azure en el cliente de React con el código siguiente. 

:::code language="javascript" source="~/azure-search-python-samples/search-website/src/pages/Search/Search.js" highlight="42-55" :::

## <a name="azure-function-suggestions-from-the-catalog"></a>Azure Function: sugerencias del catálogo

`Suggest` [API](https://github.com/Azure-Samples/azure-search-python-samples/blob/master/search-website/api/Suggest/__init__.py) toma un término de búsqueda mientras un usuario está escribiendo y sugiere términos de búsqueda como títulos de libros y autores en los documentos en el índice de búsqueda, y devuelve una pequeña lista de coincidencias. 

El proveedor de sugerencias de búsqueda, `sg`, se define en el [archivo de esquema](https://github.com/Azure-Samples/azure-search-python-samples/blob/master/search-website/bulk-upload/good-books-index.json) usado durante la carga masiva.

El enrutamiento de Suggest API se incluye en los enlaces de [function.json](https://github.com/Azure-Samples/azure-search-python-samples/blob/master/search-website/api/Suggest/function.json).

:::code language="python" source="~/azure-search-python-samples/search-website/api/Suggest/__init__.py" highlight="8-23, 35" :::

## <a name="client-suggestions-from-the-catalog"></a>Cliente: sugerencias del catálogo

Se llama a la API de la función Suggest en la aplicación React en `\src\components\SearchBar\SearchBar.js` como parte de la inicialización de componentes:

:::code language="javascript" source="~/azure-search-python-samples/search-website/src/components/SearchBar/SearchBar.js" highlight="52-60" :::

## <a name="azure-function-get-specific-document"></a>Función de Azure: obtención de un documento específico 

`Lookup` [API](https://github.com/Azure-Samples/azure-search-python-samples/blob/master/search-website/api/Lookup/__init__.py) toma un identificador y devuelve el objeto de documento del índice de búsqueda. 

El enrutamiento de Lookup API se incluye en los enlaces de [function.json](https://github.com/Azure-Samples/azure-search-python-samples/blob/master/search-website/api/Lookup/function.json).

:::code language="python" source="~/azure-search-python-samples/search-website/api/Lookup/__init__.py" highlight="8-18, 27" :::

## <a name="client-get-specific-document"></a>Cliente: obtención de un documento específico 

Se llama a esta API de función en la aplicación React en `\src\pages\Details\Detail.js` como parte de la inicialización de componentes:

:::code language="javascript" source="~/azure-search-python-samples/search-website/src/pages/Details/Details.js" highlight="19-29" :::

## <a name="next-steps"></a>Pasos siguientes

* [Indexación de datos de Azure SQL](search-indexer-tutorial.md)
