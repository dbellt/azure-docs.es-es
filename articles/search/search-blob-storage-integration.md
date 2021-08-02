---
title: Búsqueda en el contenido de Azure Blob Storage
titleSuffix: Azure Cognitive Search
description: Aprenda a extraer texto de blobs de Azure y a que se puedan realizar búsquedas de texto completo en un índice de Azure Cognitive Search.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 05/14/2021
ms.openlocfilehash: 6646a2e5a074219df13f3bf373edfc53310c8104
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111556611"
---
# <a name="search-over-azure-blob-storage-content"></a>Búsqueda en el contenido de Azure Blob Storage

La búsqueda en los diversos tipos de contenido almacenado en Azure Blob Storage puede ser un problema difícil de resolver. En este artículo se examina el flujo de trabajo básico para extraer contenido y metadatos de blobs y enviarlo a un índice de búsqueda en Azure Cognitive Search. En el índice resultante se pueden realizar consultas mediante la búsqueda de texto completo.

> [!NOTE]
> ¿Conoce el flujo de trabajo y su composición? El siguiente paso es consultar [Configuración de un indexador de blobs](search-howto-indexing-azure-blob-storage.md).

## <a name="what-it-means-to-add-full-text-search-to-blob-data"></a>Qué implica agregar búsqueda de texto completo a los datos de blob

Azure Cognitive Search es un servicio de búsqueda que admite las cargas de trabajo de indexación y consulta a través de índices definidos por el usuario que contienen el contenido en que se pueden hacer búsquedas remotas hospedado en la nube. Para mejorar el rendimiento es necesario colocalizar el contenido que permite búsquedas, lo que devuelve resultados a la velocidad a la que los usuarios esperan de las consultas de búsqueda.

Cognitive Search se integra con Azure Blob Storage en el nivel de indexación e importa el contenido de los blobs en forma de documentos de búsqueda que se indexan en *índices invertidos* y otras estructuras de consulta que admiten consultas de texto en forma libre y expresiones de filtro. Como el contenido de los blobs se indexa en un índice de búsqueda, puede usar toda la gama de características de consulta de Azure Cognitive Search para encontrar información en el contenido de los blobs.

Las entradas son los blobs, en un solo contenedor, de Azure Blob Storage. Los blobs pueden ser casi prácticamente cualquier tipo de datos de texto. Si los blobs contienen imágenes, puede agregar [enriquecimiento con IA a la indexación de blobs](search-blob-ai-integration.md) para crear y extraer texto de imágenes.

La salida siempre es un índice de Azure Cognitive Search, que se usa para operaciones rápidas de búsqueda, recuperación y exploración de texto en aplicaciones cliente. Entre las entradas y la salida se encuentra la propia arquitectura de canalización de la indexación. La canalización se basa en la característica de *indexador*, que se explica más adelante en este artículo.

Una vez que el índice se crea y se rellena, pasa a existir con independencia del contenedor de blobs, pero se pueden volver a ejecutar operaciones de indexación para actualizar el índice según los documentos cambiados. Para detectar los cambios se usa la información de marca de tiempo de los blobs individuales. Como mecanismo de actualización, puede optar por una ejecución programada o por una indexación a petición.

## <a name="required-resources"></a>Recursos necesarios

Se necesitan tanto Azure Cognitive Search como Azure Blob Storage. En el almacenamiento de blobs, se necesita un contenedor que proporcione el contenido de origen.

Puede empezar directamente en la página del portal de la cuenta de almacenamiento. En la página de navegación izquierda, en **Blob service** haga clic en **Agregar Azure Cognitive Search** para crear un nuevo servicio o seleccionar uno existente. 

Después de agregar Azure Cognitive Search a la cuenta de almacenamiento, puede seguir el proceso estándar para indexar los datos de blob. Se recomienda utilizar el asistente para **Importar datos** de Azure Cognitive Search para comenzar con una introducción inicial sencilla, o bien llamar a las API de REST mediante una herramienta como, por ejemplo, Postman. Este tutorial le guiará por los pasos necesarios para llamar a la API REST en Postman: [Indexación y búsqueda de datos semiestructurados (blobs JSON) en Azure Cognitive Search](search-semi-structured-data.md) 

## <a name="use-a-blob-indexer"></a>Uso de un indexador de blobs

Un *indexador* es un subservicio con reconocimiento del origen de datos de Cognitive Search y equipado con lógica interna para el muestreo de datos, la lectura de datos de los metadatos, la recuperación de datos y la serialización de datos procedentes de formatos nativos en documentos JSON para su posterior importación. 

Los blobs de Azure Storage se indexan mediante el [indexador de Blob Storage para Azure Cognitive Search](search-howto-indexing-azure-blob-storage.md). Puede invocar este indexador mediante el asistente para la  **importación de datos**, una API REST o el SDK de .NET. En el código, este indexador se usa estableciendo el tipo y proporcionando información de conexión —que incluye una cuenta de Azure Storage junto con un contenedor de blobs—. Puede crear un subconjunto de los blobs mediante la creación de un directorio virtual, que luego puede pasar como parámetro, o filtrando por una extensión de tipo de archivo.

Un indexador realiza el "descifrado de documentos", abriendo un blob para inspeccionar el contenido. Después de conectarse al origen de datos, es el primer paso de la canalización. En el caso de los datos de blob, aquí es donde se detectan los archivos PDF, los documentos de Office y otros tipos de contenido. El descifrado de documentos con extracción de texto no tiene ningún cargo. Si los blobs incluyen contenido de imagen, las imágenes se omiten, a menos que [agregue enriquecimiento con inteligencia artificial](search-blob-ai-integration.md). La indexación estándar solo se aplica al contenido de texto.

El indexador de blobs incluye parámetros de configuración y admite el seguimiento de cambios si los datos subyacentes proporcionan información suficiente. Puede obtener más información sobre la funcionalidad básica en [Indexador en Azure Blob Storage con Azure Cognitive Search](search-howto-indexing-azure-blob-storage.md).

### <a name="supported-access-tiers"></a>Niveles de acceso admitidos

Entre los [niveles de acceso](../storage/blobs/storage-blob-storage-tiers.md) de Blob Storage se incluyen el nivel de acceso frecuente, esporádico y de archivo. Los indexadores solo tienen acceso frecuente y esporádico. 

### <a name="supported-content-types"></a>Tipos de contenido admitidos

Mediante la ejecución de un indexador de blobs en un contenedor, puede extraer texto y metadatos de los siguientes tipos de contenido con una sola consulta:

[!INCLUDE [search-blob-data-sources](../../includes/search-blob-data-sources.md)]

### <a name="indexing-blob-metadata"></a>Indexación de metadatos de blob

Una práctica común que permite ordenar fácilmente blobs con cualquier tipo de contenido consiste en indexar los metadatos personalizados y las propiedades de sistema de cada uno de los blobs. De esta manera, la información de todos los blobs se indexa con independencia del tipo de documento y se almacena en un índice en el servicio de búsqueda. Con el nuevo índice, puede continuar con la ordenación, filtrado y búsqueda por facetas en todo el contenido de Blob Storage.

> [!NOTE]
> Las etiquetas del índice de blobs se indexan de forma nativa mediante el servicio Blob Storage y se exponen para su consulta. Si los atributos de clave y valor de los blobs requieren funcionalidad de indexación y filtrado, se deben aprovechar las etiquetas del índice de blobs en lugar de los metadatos.
>
> Para más información sobre el índice de blobs, consulte [Administración y búsqueda de datos en Azure Blob Storage con el Índice de blobs (versión preliminar)](../storage/blobs/storage-manage-find-blobs.md).

### <a name="indexing-json-blobs"></a>Indexación de blobs JSON

Es posible configurar indexadores para extraer el contenido estructurado que se encuentra en los blobs que contienen JSON. Un indexador puede leer blobs JSON y analizar el contenido estructurado en los campos correspondientes de un documento de búsqueda. Los indexadores también pueden tomar blobs que contienen una matriz de objetos JSON y asignar cada elemento a un documento de búsqueda independiente. Puede establecer un modo de análisis que se aplique al tipo de objeto JSON creado por el indexador.

## <a name="search-blob-content-in-a-search-index"></a>Búsqueda de contenido de blobs en un índice de búsqueda 

La salida de un indexador es un índice de búsqueda que se usa para la exploración interactiva mediante consultas de texto libre y con filtros en una aplicación cliente. Para la exploración inicial y la comprobación del contenido, se recomienda comenzar con el [Explorador de búsqueda](search-explorer.md) en el portal, para examinar la estructura de los documentos. Puede usar una [sintaxis de consulta simple](query-simple-syntax.md), una [sintaxis de consulta completa](query-lucene-syntax.md) y una [sintaxis de expresiones de filtro](query-odata-filter-orderby-syntax.md) en el Explorador de búsqueda.

Una solución más permanente supone recopilar las entradas de la consulta y presentar la respuesta como resultados de la búsqueda en una aplicación cliente. En el siguiente tutorial para C# se explica cómo compilar una aplicación de búsqueda: [Crear su primera aplicación en Azure Cognitive Search](tutorial-csharp-create-first-app.md).

## <a name="next-steps"></a>Pasos siguientes

+ [Carga, descarga y enumeración de blobs con Azure Portal (Azure Blob Storage)](../storage/blobs/storage-quickstart-blobs-portal.md)
+ [Configuración de un indexador de blobs (Azure Cognitive Search)](search-howto-indexing-azure-blob-storage.md)