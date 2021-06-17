---
title: 'Tipo de datos admitidos: Azure Time Series Insights Gen2 | Microsoft Docs'
description: Información acerca de los tipo de datos admitidos en Azure Time Series Insights Gen2.
author: tedvilutis
ms.author: tvilutis
manager: cnovak
ms.reviewer: orspodek
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 01/19/2021
ms.openlocfilehash: eee8a3167c5c7b9be2ccf0c5f14895846bfd3d3a
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110796243"
---
# <a name="supported-data-types"></a>Tipos de datos admitidos

En la tabla siguiente se enumeran los tipos de datos admitidos por Azure Time Series Insights Gen2

| Tipo de datos | Descripción | Ejemplo | [Sintaxis de expresión de serie temporal](/rest/api/time-series-insights/reference-time-series-expression-syntax) | Nombre de la columna de propiedad en Parquet
|---|---|---|---|---|
| **bool** | Tipo de datos que tiene uno de dos estados: `true` o `false`. | `"isQuestionable" : true` | `$event.isQuestionable.Bool` o `$event['isQuestionable'].Bool` | `isQuestionable_bool`
| **datetime** | Representa un instante de tiempo, normalmente expresado en forma de fecha y hora del día. Se expresa en formato [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html). Las propiedades DateTime siempre se almacenan en formato UTC. Los desplazamientos de zona horaria, si tiene el formato correcto, se aplicarán y, luego, el valor se almacenará en UTC. Consulte [esta](concepts-streaming-ingestion-event-sources.md#event-source-timestamp) sección para obtener más información sobre la propiedad de marca de tiempo del entorno y los desplazamientos datetime | `"eventProcessedLocalTime": "2020-03-20T09:03:32.8301668Z"` |  Si "eventProcessedLocalTime" es la marca de tiempo del origen del evento: `$event.$ts`. Si es otra propiedad JSON: `$event.eventProcessedLocalTime.DateTime` o `$event['eventProcessedLocalTime'].DateTime` | `eventProcessedLocalTime_datetime`
| **double** | Un número de 64 bits de precisión doble  | `"value": 31.0482941` | `$event.value.Double` o `$event['value'].Double` |  `value_double`
| **long** | Un entero de 64 bits con signo  | `"value" : 31` | `$event.value.Long` o `$event['value'].Long` |  `value_long`
| **string** | Los valores de texto, deben constar de UTF8 válidos. Las cadenas NULL y vacías se tratan de la misma forma. |  `"site": "DIM_MLGGG"`| `$event.site.String` o `$event['site'].String`| `site_string`
| **dynamic** | Tipo complejo (no primitivo) que consta de una matriz o un contenedor de propiedades (diccionario). Actualmente, solo se almacenarán como dinámicas las matrices JSON de string de elementos primitivos o las matrices de objetos que no contienen el identificador de TS o las propiedades de marca de tiempo. Consulte este [artículo](./concepts-json-flattening-escaping-rules.md) para comprender cómo se aplanarán los objetos y se podrán expandir las matrices. Solo se puede tener acceso a las propiedades de carga almacenadas como este tipo si se selecciona `Explore Events` en el Explorador de Time Series Insights para ver eventos sin procesar, o mediante la API de consulta [`GetEvents`](/rest/api/time-series-insights/dataaccessgen2/query/execute#getevents) para el análisis del lado cliente. |  `"values": "[197, 194, 189, 188]"` | Todavía no se admite la referencia a tipos dinámicos en una expresión de serie temporal. | `values_dynamic`

> [!NOTE]
> Se admiten valores enteros de 64 bits, pero el número más grande que el Explorador de Azure Time Series Insights puede expresar con seguridad es 9 007 199 254 740 991 (2^53-1) debido a las limitaciones de JavaScript. Si trabaja con números en el modelo de datos anterior, puede reducir el tamaño creando una [variable de modelo de serie temporal](./concepts-variables.md#numeric-variables) y [convirtiendo](/rest/api/time-series-insights/reference-time-series-expression-syntax#conversion-functions) el valor.

> [!NOTE]
> El tipo **Cadena** no admite valores NULL:
>
> * Una [expresión de serie temporal (TSX)](/rest/api/time-series-insights/reference-time-series-expression-syntax) expresada en una [consulta de serie temporal](/rest/api/time-series-insights/reference-query-apis) que compara el valor de una cadena vacía ( **''** ) con **NULL** se comportará de la misma manera: `$event.siteid.String = NULL` es equivalente a `$event.siteid.String = ''`.
> * La API puede devolver valores **NULL** incluso si los eventos originales contenían cadenas vacías.
> * No cree dependencias de valores **NULL** en columnas **Cadena** para realizar comparaciones o evaluaciones, trátelas de la misma manera que a las cadenas vacías.

## <a name="sending-mixed-data-types"></a>Envío de tipos de datos mixtos

El entorno de Azure Time Series Insights Gen2 está fuertemente tipado. Si los dispositivos o etiquetas envían datos de diferentes tipos para una propiedad de dispositivo, los valores se almacenarán en dos columnas separadas y la [función coalesce()](/rest/api/time-series-insights/reference-time-series-expression-syntax#other-functions) deberá usarse al definir las expresiones de las variables del modelo de serie temporal en las llamadas API.

El explorador de Azure Time Series Insights ofrece una manera de fusionar automáticamente las columnas independientes de la misma propiedad de dispositivo. En el ejemplo siguiente, el sensor envía una propiedad `PresentValue` que puede ser un valor Long o Double. Para realizar una consulta a todos los valores almacenados (independientemente del tipo de datos) de la propiedad `PresentValue`, elija `PresentValue (Double | Long)` y las columnas se fusionarán automáticamente.

[![Fusión automática del explorador](media\concepts-supported-data-types/explorer-auto-coalesce-sample.png)](media\concepts-supported-data-types/explorer-auto-coalesce-sample.png#lightbox)

## <a name="objects-and-arrays"></a>Objetos y matrices

Puede enviar tipos complejos, como objetos y matrices, como parte de la carga del evento. Los objetos anidados se acoplarán y las matrices se almacenarán como `dynamic` o sin acoplar para generar varios eventos en función de la configuración del entorno y la forma de JSON. Para obtener más información, lea sobre las [Reglas de acoplamiento y de escape de JSON](./concepts-json-flattening-escaping-rules.md)

## <a name="next-steps"></a>Pasos siguientes

* Lea las [Reglas de acoplamiento y de escape de JSON](./concepts-json-flattening-escaping-rules.md) para comprender cómo se almacenarán los eventos.

* Comprender las [limitaciones de rendimiento](./concepts-streaming-ingress-throughput-limits.md) del entorno

* Obtenga información sobre [Orígenes de eventos](concepts-streaming-ingestion-event-sources.md) para ingerir datos en streaming.
