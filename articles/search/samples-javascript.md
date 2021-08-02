---
title: Ejemplos de JavaScript
titleSuffix: Azure Cognitive Search
description: Busque ejemplos de código de JavaScript de demostración de Azure Cognitive Search que usan el SDK de .NET de Azure para JavaScript.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/11/2021
ms.openlocfilehash: 073095d590e61d9e7e561339c17501f0ccee844d
ms.sourcegitcommit: 942a1c6df387438acbeb6d8ca50a831847ecc6dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112018664"
---
# <a name="javascript-code-samples-for-azure-cognitive-search"></a>Ejemplos de código de JavaScript para Azure Cognitive Search

Obtenga información sobre los ejemplos de código de JavaScript que muestran la funcionalidad y el flujo de trabajo de una solución de Azure Cognitive Search. En estos ejemplos se usa la [**biblioteca cliente de Azure Cognitive Search**](/javascript/api/overview/azure/search-documents-readme) para el [**SDK de Azure para JavaScript**](/azure/developer/javascript/), que puede explorar por medio de los vínculos siguientes.

| Destino | Vínculo |
|--------|------|
| Descarga del paquete | [www.npmjs.com/package/@azure/search-documents](https://www.npmjs.com/package/@azure/search-documents) |
| Referencia de API | [@azure/search-documents](/javascript/api/@azure/search-documents/)  |
| Casos de prueba de la API | [github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/test](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/test) |
| Código fuente | [github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents)  |

## <a name="sdk-samples"></a>Ejemplos del SDK

Los ejemplos de código del equipo de desarrollo del SDK de Azure muestran el uso de la API. Puede encontrar estos ejemplos en [**azure-sdk-for-js/tree/master/sdk/search/search-documents/samples**](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples) en GitHub.

### <a name="javascript-sdk-samples"></a>Ejemplos del SDK de JavaScript

| Ejemplos | Descripción |
|---------|-------------|
| [índices](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/javascript/src/indexes) | Muestra cómo crear, actualizar, obtener, enumerar y eliminar [índices de búsqueda](search-what-is-an-index.md). Esta categoría de ejemplos también incluye un ejemplo de estadísticas de servicio. |
| [dataSourceConnections (para indizadores)](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/javascript/src/dataSourceConnections) | Muestra cómo crear, actualizar, obtener, enumerar y eliminar orígenes de datos de indizadores, necesarios para la indización basada en indizadores de [orígenes de datos de Azure compatibles](search-indexer-overview.md#supported-data-sources). |
| [Indizadores](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/javascript/src/indexers) |  Muestra cómo crear, actualizar, obtener, enumerar y eliminar [indizadores](search-indexer-overview.md).|
| [skillSet](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/javascript/src/skillSets) |   Muestra cómo crear, actualizar, obtener, enumerar y eliminar [conjuntos de aptitudes](cognitive-search-working-with-skillsets.md) que son indizadores adjuntos, y que realizan el enriquecimiento basado en IA durante la indexación. |
| [synonymMaps](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/javascript/src/synonymMaps) | Muestra cómo crear, actualizar, obtener, enumerar y eliminar [asignaciones de sinónimos](search-synonyms.md).  |

### <a name="typescript-samples"></a>Ejemplos de TypeScript

| Ejemplos | Descripción |
|---------|-------------|
| [índices](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/typescript/src/indexes) | Muestra cómo crear, actualizar, obtener, enumerar y eliminar [índices de búsqueda](search-what-is-an-index.md). Esta categoría de ejemplos también incluye un ejemplo de estadísticas de servicio. |
| [dataSourceConnections (para indizadores)](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/typescript/src/dataSourceConnections) | Muestra cómo crear, actualizar, obtener, enumerar y eliminar orígenes de datos de indizadores, necesarios para la indización basada en indizadores de [orígenes de datos de Azure compatibles](search-indexer-overview.md#supported-data-sources). |
| [Indizadores](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/typescript/src/indexers) |  Muestra cómo crear, actualizar, obtener, enumerar y eliminar [indizadores](search-indexer-overview.md).|
| [skillSet](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/typescript/src/skillSets) |   Muestra cómo crear, actualizar, obtener, enumerar y eliminar [conjuntos de aptitudes](cognitive-search-working-with-skillsets.md) que son indizadores adjuntos, y que realizan el enriquecimiento basado en IA durante la indexación. |
| [synonymMaps](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/typescript/src/synonymMaps) | Muestra cómo crear, actualizar, obtener, enumerar y eliminar [asignaciones de sinónimos](search-synonyms.md).  |

## <a name="doc-samples"></a>Ejemplos de documentación

Los ejemplos de código del equipo de Cognitive Search muestran características y flujos de trabajo. En tutoriales, inicios rápidos y artículos de procedimientos se hace referencia a muchos de estos ejemplos. Puede encontrar estos ejemplos en [**Azure-Samples/azure-search-javascript-samples**](https://github.com/Azure-Samples/azure-search-javascript-samples) en GitHub.

| Ejemplos | Artículo |
|---------|---------|
| [quickstart](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/quickstart/v11) | Código fuente de [Inicio rápido: Creación de un índice de búsqueda en JavaScript](search-get-started-javascript.md). En este artículo se habla del flujo de trabajo básico para crear, cargar y consultar un índice de búsqueda con datos de ejemplo. |
| [search-website](https://github.com/azure-samples/azure-search-javascript-samples/tree/master/search-website) | Código fuente de [Tutorial: Incorporación de la funcionalidad de búsqueda a las aplicaciones web](tutorial-javascript-overview.md). Muestra una aplicación de búsqueda de un extremo a otro que incluye un cliente enriquecido más componentes para hospedar la aplicación y controlar las solicitudes de búsqueda.|

> [!Tip]
> Pruebe el [Explorador de ejemplos](/samples/browse/?languages=javascript&products=azure-cognitive-search) para buscar ejemplos de código de Microsoft en GitHub, filtrados por producto, servicio y lenguaje.

## <a name="other-samples"></a>Otros ejemplos

Los siguientes ejemplos también se publican en el equipo de Cognitive Search, pero no se hace referencia a ellos en la documentación. Los archivos Léame asociados proporcionan instrucciones de uso.

| Ejemplos | Descripción |
|---------|-------------|
| [azure-search-react-template](https://github.com/dereklegenzoff/azure-search-react-template) | Plantilla d React para Azure Cognitive Search (github.com) |
