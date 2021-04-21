---
title: Análisis del rendimiento
titleSuffix: Azure Cognitive Search
description: TBD
author: LiamCavanagh
ms.author: liamca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/06/2021
ms.openlocfilehash: 298508f8068b47cbfc720f1d1e8ddf370323ed28
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106582145"
---
# <a name="analyze-performance-in-azure-cognitive-search"></a>Análisis del rendimiento en Azure Cognitive Search

En este artículo se describen las herramientas, los comportamientos y los enfoques para analizar el rendimiento de las consultas y la indexación en Cognitive Search.

## <a name="develop-baseline-numbers"></a>Desarrollo de números de línea de base

En cualquier implementación grande, es fundamental realizar una prueba de punto de referencia de rendimiento del servicio Cognitive Search antes de implementarlo en producción. Debe probar la carga de consulta de búsqueda que espera, pero también las cargas de trabajo de ingesta de datos esperadas (si es posible, ejecutarlas al mismo tiempo). Tener números de punto de referencia ayuda a validar el [nivel de búsqueda](search-sku-tier.md) adecuado, la [configuración del servicio](search-capacity-planning.md) y la [latencia de consulta](search-performance-analysis.md#average-query-latency) esperada.

Para desarrollar puntos de referencia, se recomienda la herramienta [azure-search-performance-testing (GitHub).](https://github.com/Azure-Samples/azure-search-performance-testing)

Para aislar los efectos de una arquitectura de servicio distribuida, pruebe las configuraciones de servicio de una réplica y una partición.

> [!NOTE]
> En el caso de los niveles de Almacenamiento optimizado (L1 y L2), debe esperar un rendimiento más bajo de las consultas y una latencia superior que en los niveles Estándar.
>

## <a name="use-diagnostic-logging"></a>Uso del registro de diagnóstico

La herramienta más importante que tiene un administrador para diagnosticar posibles problemas de rendimiento es el [registro de diagnóstico](search-monitor-logs.md), que recopila datos operativos y métricas sobre el servicio de búsqueda. El registro de diagnóstico se habilita a través de [Azure Monitor](../azure-monitor/overview.md). Hay costos asociados con el uso de Azure Monitor y el almacenamiento de datos, pero si lo habilita para el servicio, puede ser fundamental para investigar problemas de rendimiento.

Hay varias posibilidades de que se produzca latencia, ya sea durante una transferencia de red, durante el procesamiento de contenido en la capa de App Services o en un servicio de búsqueda. Una ventaja clave del registro de diagnóstico es que las actividades se registran desde la perspectiva del servicio de búsqueda, lo que significa que el registro puede ayudarle a determinar si los problemas de rendimiento se deben a problemas con la consulta o la indexación, o a algún otro punto de error.

:::image type="content" source="media/search-performance/perf-log-event-chain.png" alt-text="Cadena de eventos registrados" border="true":::

El registro de diagnóstico ofrece opciones para almacenar la información registrada. Se recomienda usar [Log Analytics](../azure-monitor/logs/log-analytics-overview.md) para que pueda ejecutar consultas avanzadas de Kusto en los datos para responder a muchas preguntas sobre el uso y el rendimiento. 

En las páginas del portal del servicio de búsqueda, puede habilitar el registro a través de la **configuración de diagnóstico** y, a continuación, emitir consultas de Kusto en Log Analytics si elige **Registro**. Para obtener más información, consulte [Recopilación y análisis de datos de registro](search-monitor-logs.md).

:::image type="content" source="media/search-performance/perf-log-menu-options.png" alt-text="Opciones del menú Registro" border="true":::

## <a name="throttling-behaviors"></a>Comportamiento de limitación

La limitación se produce cuando el servicio de búsqueda ha alcanzado el límite de lo que puede controlar desde una perspectiva de recursos. La limitación puede producirse durante las consultas o la indexación. Desde el lado cliente, una llamada API da como resultado una respuesta HTTP 503 cuando se ha limitado. Durante la indexación, también existe la posibilidad de recibir una respuesta HTTP 207, que indica que uno o varios elementos no se pudieron indexar. Este error es un indicador de que el servicio de búsqueda se acerca a su capacidad. 

Como regla general, es mejor cuantificar la cantidad de limitación que se ve. Por ejemplo, si una consulta de búsqueda de 500 000 está limitada, podría no suponer un problema. Sin embargo, si un gran porcentaje de consultas se limita durante un período, esto sería un problema mayor. Al observar la limitación durante un período, también ayuda a identificar los períodos de tiempo en los que es más probable que se produzca y le ayudará a decidir cómo adaptarlo mejor.

Una solución sencilla a la mayoría de los problemas de limitación es iniciar más recursos en el servicio de búsqueda (normalmente réplicas para la limitación basada en consultas o particiones para la limitación basada en la indexación). Sin embargo, aumentar las réplicas o particiones agrega costos, por lo que es importante saber el motivo por el que se produce la limitación. La investigación de las condiciones que provocan la limitación se explicará en las siguientes secciones. 

A continuación, se muestra un ejemplo de una consulta de Kusto que puede identificar el desglose de las respuestas HTTP del servicio de búsqueda que se ha cargado. Al realizar consultas durante un período de 7 días, el gráfico de barras representado muestra que se ha limitado un porcentaje relativamente grande de las consultas de búsqueda, en comparación con el número de respuestas correctas (200).

```kusto
AzureDiagnostics
| where TimeGenerated > ago(7d)
| summarize count() by resultSignature_d 
| render barchart 
```

:::image type="content" source="media/search-performance/perf-bar-chart-http-errors.png" alt-text="Gráfico de barras de recuentos de errores HTTP" border="true":::

Examinar la limitación durante un período de tiempo específico puede ayudarle a identificar las horas en las que la limitación puede producirse con más frecuencia. En el ejemplo siguiente, se usa un gráfico de serie temporal para mostrar el número de consultas limitadas que se produjeron durante un período de tiempo especificado. En este caso, se realizaron las consultas limitadas correlacionadas con los tiempos de los puntos de referencia de rendimiento.

```kusto
let ['_startTime']=datetime('2021-02-25T20:45:07Z');
let ['_endTime']=datetime('2021-03-03T20:45:07Z');
let intervalsize = 1m; 
AzureDiagnostics 
| where TimeGenerated > ago(7d)
| where resultSignature_d != 403 and resultSignature_d != 404 and OperationName in ("Query.Search", "Query.Suggest", "Query.Lookup", "Query.Autocomplete")
| summarize 
  ThrottledQueriesPerMinute=bin(countif(OperationName in ("Query.Search", "Query.Suggest", "Query.Lookup", "Query.Autocomplete") and resultSignature_d == 503)/(intervalsize/1m), 0.01)
  by bin(TimeGenerated, intervalsize)
| render timechart   
```

:::image type="content" source="media/search-performance/perf-line-chart-throttled-queries.png" alt-text="Gráfico de líneas de consultas limitadas" border="true":::

## <a name="measure-individual-queries"></a>Medición de consultas individuales

En algunos casos, puede ser útil probar consultas individuales para ver cómo funcionan. Para ello, es importante poder ver cuánto tiempo tarda el servicio de búsqueda en completar el trabajo, así como cuánto tiempo se tarda en realizar la solicitud de ida y vuelta desde el cliente y de vuelta a este. Los registros de diagnóstico se pueden usar para buscar operaciones individuales, pero puede ser más fácil hacerlo todo desde una herramienta cliente, como Postman.

En el ejemplo siguiente, se ejecutó una consulta de búsqueda basada en REST. Cognitive Search incluye en cada respuesta el número de milisegundos que se tarda en completar la consulta, visible en la pestaña Encabezados, en "tiempo transcurrido". Junto a Estado en la parte superior de la respuesta, encontrará la duración del recorrido de ida y vuelta. en este caso, 418 milisegundos. En la sección de resultados, se eligió la pestaña "Encabezados". Con estos dos valores resaltados con un cuadro rojo en la imagen siguiente, vemos que el servicio de búsqueda tardó 21 ms en completar la consulta de búsqueda y que toda la solicitud de ida y vuelta del cliente tardó 125 ms. Al restar estos dos números, podemos determinar que se tardó 104 ms más en transmitir la consulta de búsqueda al servicio de búsqueda y transferir los resultados de la búsqueda al cliente.

Esto puede ser muy útil para determinar si puede haber latencias de red u otros factores que afectan al rendimiento de las consultas.

:::image type="content" source="media/search-performance/perf-elapsed-time.png" alt-text="Métricas de duración de consulta" border="true":::

## <a name="query-rates"></a>Tasas de consulta

Una posible razón para que el servicio de búsqueda limite las solicitudes se debe al gran número de consultas que se realizan donde el volumen se captura como consultas por segundo (QPS) o consultas por minuto (QPM). A medida que el servicio de búsqueda recibe más QPS, normalmente tardará cada vez más tiempo en responder a esas consultas hasta que ya no pueda mantenerse al día, momento en el que enviará una respuesta HTTP 503 de limitación. 

La siguiente consulta de Kusto muestra el volumen de consulta medido en QPM, junto con la duración media de una consulta en milisegundos (AvgDurationMS) y el número medio de documentos (AvgDocCountReturned) devueltos en cada uno de ellos.

```kusto
AzureDiagnostics
| where OperationName == "Query.Search" and TimeGenerated > ago(1d)
| extend MinuteOfDay = substring(TimeGenerated, 0, 16) 
| project MinuteOfDay, DurationMs, Documents_d, IndexName_s
| summarize QPM=count(), AvgDuractionMs=avg(DurationMs), AvgDocCountReturned=avg(Documents_d)  by MinuteOfDay
| order by MinuteOfDay desc 
| render timechart
```

:::image type="content" source="media/search-performance/perf-line-chart-queries-per-minute.png" alt-text="Gráfico que muestra las consultas por minuto" border="true":::

> [!TIP]
> Para mostrar los datos que hay tras este gráfico, quite la línea `| render timechart` y vuelva a ejecutar la consulta.

## <a name="impact-of-indexing-on-queries"></a>Impacto de la indexación en las consultas

Un factor importante que se debe tener en cuenta al examinar el rendimiento es que la indexación usa los mismos recursos que las consultas de búsqueda. Si va a indexar una gran cantidad de contenido, puede esperar ver que la latencia aumenta a medida que el servicio intenta dar cabida a ambas cargas de trabajo.

Si las consultas se ralentizan, consulte el tiempo de la actividad de indexación para ver si coincide con la degradación de las consultas. Por ejemplo, quizás un indexador ejecuta un trabajo diario o cada hora que se correlaciona con el rendimiento reducido de las consultas de búsqueda. 

En esta sección se proporciona un conjunto de consultas que pueden ayudarle a visualizar las tasas de búsqueda e indexación. En estos ejemplos, el intervalo de tiempo se establece en la consulta. Asegúrese de indicar **Establecer en la consulta** al ejecutar las consultas en Azure Portal.

:::image type="content" source="media/search-performance/perf-set-query-time-range.png" alt-text="Establecimiento de intervalos de tiempo en la herramienta de consulta" border="true":::

<a name="average-query-latency"></a>

### <a name="average-query-latency"></a>Latencia media de consulta

En la consulta siguiente, se usa un tamaño de intervalo de 1 minuto para mostrar la latencia media de las consultas de búsqueda. En el gráfico podemos ver que la latencia media era baja hasta las 5:45 p. m. y duraba hasta las 5:53 p. m.

```kusto
let intervalsize = 1m; 
let _startTime = datetime('2021-02-23 17:40');
let _endTime = datetime('2021-02-23 18:00');
AzureDiagnostics
| where TimeGenerated between(['_startTime']..['_endTime']) // Time range filtering
| summarize AverageQueryLatency = avgif(DurationMs, OperationName in ("Query.Search", "Query.Suggest", "Query.Lookup", "Query.Autocomplete"))
    by bin(TimeGenerated, intervalsize)
| render timechart
```

:::image type="content" source="media/search-performance/perf-line-chart-average-query-latency.png" alt-text="Gráfico que muestra la latencia media de las consultas" border="true":::

### <a name="average-queries-per-minute-qpm"></a>Media de consultas por minuto (QPM)

La consulta siguiente nos permite ver el número medio de consultas por minuto para asegurarse de que no hubo ningún tipo de pico en las solicitudes de búsqueda que podrían haber afectado a la latencia. En el gráfico podemos ver que hay alguna varianza, pero nada que indique un pico en el número de solicitudes.

```kusto
let intervalsize = 1m; 
let _startTime = datetime('2021-02-23 17:40');
let _endTime = datetime('2021-02-23 18:00');
AzureDiagnostics
| where TimeGenerated between(['_startTime'] .. ['_endTime']) // Time range filtering
| summarize QueriesPerMinute=bin(countif(OperationName in ("Query.Search", "Query.Suggest", "Query.Lookup", "Query.Autocomplete"))/(intervalsize/1m), 0.01)
  by bin(TimeGenerated, intervalsize)
| render timechart
```

:::image type="content" source="media/search-performance/perf-line-chart-average-queries-per-minute.png" alt-text="Gráfico que muestra la media de consultas por minuto" border="true":::

### <a name="indexing-operations-per-minute-opm"></a>Operaciones de indexación por minuto (OPM)

A continuación, se verá el número de operaciones de indexación por minuto. En el gráfico podemos ver que una gran cantidad de datos se indexó a las 5:42 p. m. y finalizó a las 5:50 p. m. Esta indexación comenzó 3 minutos antes de que las consultas de búsqueda empezaran a estar latentes y finalizaron 3 minutos antes de que ya no estuvieran latentes.

A partir de esto, podemos ver que el servicio de búsqueda tardó unos 3 minutos en estar lo suficientemente ocupado desde la indexación para iniciarse y afectar a la latencia de la consulta de búsqueda. También podemos ver que, una vez completada la indexación, el servicio de búsqueda tardó otros 3 minutos en completar todo el trabajo desde el contenido recién indexado, hasta que las consultas de búsqueda empezaron a no estar latentes.

```kusto
let intervalsize = 1m; 
let _startTime = datetime('2021-02-23 17:40');
let _endTime = datetime('2021-02-23 18:00');
AzureDiagnostics
| where TimeGenerated between(['_startTime'] .. ['_endTime']) // Time range filtering
| summarize IndexingOperationsPerSecond=bin(countif(OperationName == "Indexing.Index")/ (intervalsize/1m), 0.01)
  by bin(TimeGenerated, intervalsize)
| render timechart 
```

:::image type="content" source="media/search-performance/perf-line-chart-indexing-operations.png" alt-text="Gráfico que muestra las operaciones de indexación por minuto" border="true":::

## <a name="background-service-processing"></a>Procesamiento del servicio en segundo plano

No es inusual ver picos periódicos en la latencia de consulta o indexación. Es posible que se produzcan picos en respuesta a la indexación o a tasas de consulta elevadas, pero también podrían producirse durante las operaciones Merge. Los índices de búsqueda se almacenan en fragmentos o particiones. Periódicamente, el sistema combina particiones más pequeñas en particiones grandes, lo que puede ayudar a optimizar el rendimiento del servicio. Este proceso de combinación también limpia los documentos que se han marcado previamente para su eliminación del índice, lo que da lugar a la recuperación del espacio de almacenamiento. 

La combinación de particiones es rápida, pero también consume muchos recursos y, por tanto, tiene el potencial de degradar el rendimiento del servicio. Por este motivo, si ve ráfagas cortas de latencia de consulta y esas ráfagas coinciden con los cambios recientes en el contenido indexado, es probable que atribuya esa latencia a las operaciones Merge de particiones. 

## <a name="next-steps"></a>Pasos siguientes

Revise estos artículos adicionales relacionados con el análisis del rendimiento del servicio.

+ [Consejos de rendimiento](search-performance-tips.md)
+ [Selección de un nivel de servicio](search-sku-tier.md)
+ [Administración de capacidad](search-capacity-planning.md)
