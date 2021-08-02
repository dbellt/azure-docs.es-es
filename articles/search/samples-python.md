---
title: Ejemplos de Python
titleSuffix: Azure Cognitive Search
description: Busque ejemplos de código de Python de demostración de Azure Cognitive Search que usan el SDK de .NET de Azure para Python o REST.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/11/2021
ms.openlocfilehash: fc27ed68f5dedefa0b4bf77a206faa15c94f083f
ms.sourcegitcommit: 942a1c6df387438acbeb6d8ca50a831847ecc6dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112018322"
---
# <a name="python-code-samples-for-azure-cognitive-search"></a>Ejemplos de código de Python de Azure Cognitive Search

Obtenga información sobre los ejemplos de código de Python que muestran la funcionalidad y el flujo de trabajo de una solución de Azure Cognitive Search. En estos ejemplos se usa la [**biblioteca cliente de Azure Cognitive Search**](/python/api/overview/azure/search-documents-readme) para el [**SDK de Azure para Python**](/azure/developer/python/), que puede examinar por medio de los vínculos siguientes.

| Destino | Vínculo |
|--------|------|
| Descarga del paquete | [pypi.org/project/azure-search-documents/](https://pypi.org/project/azure-search-documents/) |
| Referencia de API | [azure-search-documents](/python/api/azure-search-documents)  |
| Casos de prueba de la API | [github.com/Azure/azure-sdk-for-python/tree/master/sdk/search/azure-search-documents/tests](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/search/azure-search-documents/tests) |
| Código fuente | [github.com/Azure/azure-sdk-for-python/tree/master/sdk/search/azure-search-documents](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/search/azure-search-documents)  |

## <a name="sdk-samples"></a>Ejemplos del SDK

Los ejemplos de código del equipo de desarrollo del SDK de Azure muestran el uso de la API. Puede encontrar estos ejemplos en [**azure-sdk-for-python/tree/master/sdk/search/azure-search-documents/samples**](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/search/azure-search-documents/samples) en GitHub.

| Ejemplos | Descripción |
|---------|-------------|
| [Autenticar](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_authentication.py) | Muestra cómo configurar un cliente y autenticarse en el servicio. | 
| [Operaciones de creación, lectura, actualización y eliminación de índices](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_index_crud_operations.py) | Muestra cómo crear, actualizar, obtener, enumerar y eliminar [índices de búsqueda](search-what-is-an-index.md). |
| [Operaciones de creación, lectura, actualización y eliminación de indizadores](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_indexers_operations.py) | Muestra cómo crear, actualizar, obtener, enumerar y eliminar [indizadores](search-indexer-overview.md). |
| [Orígenes de datos de indizadores de búsqueda](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_indexer_datasource_skillset.py) | Muestra cómo crear, actualizar, obtener, enumerar y eliminar orígenes de datos de indizadores, necesarios para la indización basada en indizadores de [orígenes de datos de Azure compatibles](search-indexer-overview.md#supported-data-sources). |
| [Sinónimos](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_synonym_map_operations.py) | Muestra cómo crear, actualizar, obtener, enumerar y eliminar [asignaciones de sinónimos](search-synonyms.md).  |
| [Carga de documentos](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_crud_operations.py) | Muestra cómo cargar o combinar documentos en un índice en una operación de [importación de datos](search-what-is-data-import.md). |
| [Consulta sencilla](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_simple_query.py) | Muestra cómo configurar una [consulta básica](search-query-overview.md). |
| [Consulta con filtro](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_filter_query.py) | Muestra la configuración de una [expresión de filtro](search-filters.md). |
| [Consulta de facetas](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_facet_query.py) | Muestra el trabajo con [facetas](search-filters-facets.md). |

## <a name="doc-samples"></a>Ejemplos de documentación

Los ejemplos de código del equipo de Cognitive Search muestran características y flujos de trabajo. En tutoriales, inicios rápidos y artículos de procedimientos se hace referencia a muchos de estos ejemplos. Puede encontrar estos ejemplos en [**Azure-Samples/azure-search-python-samples**](https://github.com/Azure-Samples/azure-search-python-samples) en GitHub.

| Ejemplos | Artículo |
|---------|---------|
| [quickstart](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/Quickstart) | Código fuente de [Inicio rápido: Creación de un índice de Azure Cognitive Search en Python](search-get-started-python.md). En este artículo se habla del flujo de trabajo básico para crear, cargar y consultar un índice de búsqueda con datos de ejemplo. |
| [search-website](https://github.com/azure-samples/azure-search-python-samples/tree/master/search-website) | Código fuente de [Tutorial: Incorporación de la funcionalidad de búsqueda a las aplicaciones web](tutorial-python-overview.md). Muestra una aplicación de búsqueda de un extremo a otro que incluye un cliente enriquecido más componentes para hospedar la aplicación y controlar las solicitudes de búsqueda.|
| [tutorial-ai-enrichment](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/Tutorial-AI-Enrichment)  | Código fuente de [Tutorial: Uso de Python y AI para generar contenido que permita búsquedas desde blobs de Azure](cognitive-search-tutorial-blob-python.md). En este artículo se muestra cómo crear un indizador de blobs con un conjunto de aptitudes cognitivas, donde el conjunto de aptitudes crea y transforma contenido sin procesar para que pueda buscarse o consumirse. |
| [AzureML-Custom-Skill](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/AzureML-Custom-Skill)  | Código fuente de [Ejemplo: Creación de una aptitud personalizada con Python](cognitive-search-custom-skill-python.md). En este artículo se muestra la integración del indizador y el conjunto de aptitudes con modelos de aprendizaje profundo en Azure Machine Learning. |

> [!Tip]
> Pruebe el [Explorador de ejemplos](/samples/browse/?languages=python&products=azure-cognitive-search) para buscar ejemplos de código de Microsoft en GitHub, filtrados por producto, servicio y lenguaje.
