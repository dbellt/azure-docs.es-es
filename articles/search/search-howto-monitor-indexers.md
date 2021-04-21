---
title: Supervisión del estado y los resultados del indizador
titleSuffix: Azure Cognitive Search
description: Supervise el estado, el progreso y los resultados de los indizadores de Azure Cognitive Search en Azure Portal mediante la API REST o el SDK de .NET.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/28/2021
ms.openlocfilehash: ed79831559802efdf7e3813f48b74c2fd3c63e87
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106581861"
---
# <a name="monitor-indexer-status-and-results-in-azure-cognitive-search"></a>Supervisión del estado y los resultados del indizador en Azure Cognitive Search

Puede supervisar el procesamiento del indexador en Azure Portal o mediante programación a través de llamadas REST o un SDK de Azure. Además del estado sobre el indexador mismo, puede revisar las horas de inicio y finalización, así como los errores y advertencias detallados de una ejecución determinada.

## <a name="monitor-using-azure-portal"></a>Supervisión mediante Azure Portal

Puede consultar el estado actual de todos los indexadores en la página de información general del servicio de búsqueda. Las páginas del portal se actualizan cada pocos minutos, por lo que no verá evidencia de que una ejecución de indexador nueva se ejecute de inmediato.

   ![Lista de indizadores](media/search-monitor-indexers/indexers-list.png "Lista de indizadores")

| Estado | Descripción |
|--------|-------------|
| **En curso** | Indica la ejecución activa. El portal entregará información parcial. A medida que avanza la indexación, puede ver que el valor **Documentos correctos** aumenta en respuesta. Los indizadores que procesan grandes volúmenes de datos pueden tardar mucho en ejecutarse. Por ejemplo, los indizadores que tratan millones de documentos de origen pueden ejecutarse durante 24 horas y, después, reiniciarse casi de inmediato. El estado de los indizadores de gran volumen podría indicar siempre **En curso** en el portal. Incluso cuando se está ejecutando un indizador, los detalles sobre el progreso actual y las ejecuciones anteriores están disponibles. |
| **Success** | Indica que la ejecución se completó correctamente. Una ejecución del indizador puede ser correcta incluso si los documentos individuales tienen errores, siempre y cuando el número de errores sea inferior al valor **Máx. de elementos con error** del indizador. |
| **Erróneo** | El número de errores superó el valor **Máx. de elementos con error** y la indexación se detuvo. |
| **Reset** | Se restableció el estado de seguimiento de cambios internos del indexador. El indexador se ejecutará en su totalidad, actualizando todos los documentos, no solo aquellos con marcas de tiempo más recientes. |

Puede hacer clic en un indexador de la lista para ver más detalles sobre sus ejecuciones actuales y recientes.

   ![Historial de ejecución y resumen del indizador](media/search-monitor-indexers/indexer-summary.png "Historial de ejecución y resumen del indizador")

En el gráfico **Resumen del indizador** se muestra el número de documentos procesados en las ejecuciones más recientes.

En la lista **Detalles de la ejecución** aparecen como máximo los 50 resultados de las ejecuciones más recientes. Haga clic en un resultado de la lista para ver los detalles de una ejecución en concreto. Se incluyen las horas de inicio y finalización, así como los errores y las advertencias que se han producido.

   ![Detalles de ejecución del indizador](media/search-monitor-indexers/indexer-execution.png "Detalles de ejecución del indizador")

Si se produjeron problemas específicos del documento durante la ejecución, se mostrarán en el campo Errores o advertencias.

   ![Detalles del indizador con errores](media/search-monitor-indexers/indexer-execution-error.png "Detalles del indizador con errores")

Las advertencias son comunes con algunos tipos de indizadores y no siempre indican un problema. Por ejemplo, los indizadores que usan Cognitive Services pueden notificar advertencias cuando las imágenes o los archivos PDF no contienen texto para procesar. 

Para obtener más información sobre cómo investigar los errores y las advertencias del indizador, vea [Solución de problemas comunes con el indizador en Azure Search](search-indexer-troubleshooting.md).

## <a name="monitor-using-get-indexer-status-rest-api"></a>Supervisión mediante Obtener estado del indexador (API REST)

Puede recuperar el estado y el historial de ejecución de un indizador con el [comando Obtener estado del indizador](/rest/api/searchservice/get-indexer-status):

```http
GET https://[service name].search.windows.net/indexers/[indexer name]/status?api-version=2020-06-30
api-key: [Search service admin key]
```

La respuesta contiene el estado general del indexador, la última vez que se invocó al indexador (o en curso) y el historial de las recientes invocaciones al indexador.

```output
{
    "status":"running",
    "lastResult": {
        "status":"success",
        "errorMessage":null,
        "startTime":"2018-11-26T03:37:18.853Z",
        "endTime":"2018-11-26T03:37:19.012Z",
        "errors":[],
        "itemsProcessed":11,
        "itemsFailed":0,
        "initialTrackingState":null,
        "finalTrackingState":null
     },
    "executionHistory":[ {
        "status":"success",
         "errorMessage":null,
        "startTime":"2018-11-26T03:37:18.853Z",
        "endTime":"2018-11-26T03:37:19.012Z",
        "errors":[],
        "itemsProcessed":11,
        "itemsFailed":0,
        "initialTrackingState":null,
        "finalTrackingState":null
    }]
}
```

El historial de ejecución contiene como máximo las 50 ejecuciones más recientes, dispuestas en orden cronológico inverso (la más reciente en primer lugar).

Tenga en cuenta que hay dos valores de estado diferentes. El estado de nivel superior es para el propio indizador. El estado del indizador **En ejecución** significa que el indizador se ha configurado correctamente y está disponible para su ejecución, pero no que se esté ejecutando actualmente.

Cada ejecución del indizador también tiene su propio estado, que indica si la ejecución está en curso (**En ejecución**) o se ha completado con el estado **correcto**, **transientFailure** o **persistentFailure**. 

Cuando se restablece un indizador para actualizar su estado de seguimiento de cambios, se agrega una entrada independiente en el historial de ejecución con el estado **Restablecer**.

Para más detalles sobre los códigos de estado y los datos de supervisión del indexador, consulte [Obtención del estado del indexador](/rest/api/searchservice/get-indexer-status).

## <a name="monitor-using-net"></a>Supervisión mediante .NET

Con el SDK de .NET de Azure Cognitive Search, en el siguiente ejemplo de C# se escribe información sobre el estado de un indizador y los resultados de su ejecución más reciente, o en curso, en la consola.

```csharp
static void CheckIndexerStatus(SearchIndexerClient indexerClient, SearchIndexer indexer)
{
    try
    {
        string indexerName = "hotels-sql-idxr";
        SearchIndexerStatus execInfo = indexerClient.GetIndexerStatus(indexerName);

        Console.WriteLine("Indexer has run {0} times.", execInfo.ExecutionHistory.Count);
        Console.WriteLine("Indexer Status: " + execInfo.Status.ToString());

        IndexerExecutionResult result = execInfo.LastResult;

        Console.WriteLine("Latest run");
        Console.WriteLine("Run Status: {0}", result.Status.ToString());
        Console.WriteLine("Total Documents: {0}, Failed: {1}", result.ItemCount, result.FailedItemCount);

        TimeSpan elapsed = result.EndTime.Value - result.StartTime.Value;
        Console.WriteLine("StartTime: {0:T}, EndTime: {1:T}, Elapsed: {2:t}", result.StartTime.Value, result.EndTime.Value, elapsed);

        string errorMsg = (result.ErrorMessage == null) ? "none" : result.ErrorMessage;
        Console.WriteLine("ErrorMessage: {0}", errorMsg);
        Console.WriteLine(" Document Errors: {0}, Warnings: {1}\n", result.Errors.Count, result.Warnings.Count);
    }
    catch (Exception e)
    {
        // Handle exception
    }
}
```

La salida de la consola tendrá un aspecto similar al siguiente:

```output
Indexer has run 18 times.
Indexer Status: Running
Latest run
  Run Status: Success
  Total Documents: 7, Failed: 0
  StartTime: 11:29:31 PM, EndTime: 11:29:31 PM, Elapsed: 00:00:00.2560000
  ErrorMessage: none
  Document Errors: 0, Warnings: 0
```

Tenga en cuenta que hay dos valores de estado diferentes. El estado de nivel superior es el del propio indizador. El estado del indizador **En ejecución** significa que el indizador se ha configurado correctamente y está disponible para su ejecución, pero no que se esté ejecutando actualmente.

Cada ejecución del indizador también tiene su propio estado, que indica si la ejecución está en curso (**En ejecución**) o se ha completado con el estado **Correcto** o **TransientError**. 

Cuando se restablece un indizador para actualizar su estado de seguimiento de cambios, se agrega una entrada independiente en el historial con el estado **Restablecer**.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más detalles sobre los códigos de estado y la información de supervisión del indizador, vea la siguiente referencia de API:

* [GetIndexerStatus (API de REST)](/rest/api/searchservice/get-indexer-status)
* [IndexerStatus](/dotnet/api/azure.search.documents.indexes.models.indexerstatus)
* [IndexerExecutionStatus](/dotnet/api/azure.search.documents.indexes.models.indexerexecutionstatus)
* [IndexerExecutionResult](/dotnet/api/azure.search.documents.indexes.models.indexerexecutionresult)