---
title: Próximos cambios en las reglas de ingesta y simplificación en Azure Time Series Insights Gen2 | Microsoft Docs
description: Cambios en la regla de ingesta
ms.service: time-series-insights
services: time-series-insights
author: tedvilutis
ms.author: tvilutis
manager: cnovak
ms.reviewer: orspodek
ms.workload: big-data
ms.topic: conceptual
ms.date: 10/02/2020
ms.custom: lyhughes
ms.openlocfilehash: f89e7196bcc0bf5dfce94ea0198e6fd255270941
ms.sourcegitcommit: 6323442dbe8effb3cbfc76ffdd6db417eab0cef7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110615445"
---
# <a name="upcoming-changes-to-json-flattening-and-escaping-rules-for-new-environments"></a>Próximos cambios en las reglas de aplanamiento y escape de JSON para los entornos nuevos

> [!IMPORTANT]
> Estos cambios solo se aplicarán a los entornos de Microsoft Azure Time Series Insights Gen2 *recién creados*. Los cambios no se aplican a los entornos de Gen1.

El entorno de Azure Time Series Insights Gen2 crea dinámicamente las columnas de almacenamiento, siguiendo un conjunto determinado de convenciones de nomenclatura. Cuando se ingiere un evento, Time Series Insights aplica un conjunto de reglas a la carga y los nombres de propiedad de JSON. Los cambios en la forma en que los datos JSON se aplanan y se almacenan entraron en vigor en los entornos nuevos de Azure Time Series Insights Gen2 en julio de 2020. Este cambio afecta su entorno en los siguientes casos:

* Si la carga de JSON contiene objetos anidados.
* Si la carga de JSON contiene matrices.
* Use cualquiera de los cuatro caracteres especiales siguientes en un nombre de propiedad de JSON: `[` `\` `.` `'`
* Una o varias de las propiedades del identificador de Time Series (TS) se encuentran dentro de un objeto anidado.

Si crea un entorno nuevo y uno o varios de estos casos se aplican a la carga del evento, los datos se habrán aplanado y almacenado de forma diferente. En la tabla siguiente se resumen los cambios:

| Regla actual | Nueva regla | Ejemplo de JSON | Nombre de columna anterior | Nuevo nombre de columna
|---|---| ---| ---|  ---|
| Para aplanar un objeto JSON anidado, se usa un carácter de subrayado como delineador. |Para aplanar un objeto JSON anidado, se usa un punto como delineador.  | ``{"series" : { "value" : 19.338 }}`` | `series_value_double` |`series.value_double` |
| Los caracteres especiales no incluyen caracteres de escape. | Los nombres de propiedad JSON que incluyen los caracteres especiales `.` `[`   `\` y `'`incluyen los caracteres de escape `['` y `']`. Dentro de `['` y `']`, hay un escape adicional de comillas simples y barras diagonales inversas. Una comilla simple se escribirá así, `\'`, mientras que una barra diagonal inversa se escribirá así `\\`.  | ```"Foo's Law Value": "17.139999389648"``` | `Foo's Law Value_double` | `['Foo\'s Law Value']_double` |
| Las matrices de primitivas se almacenan como una cadena. | Las matrices de tipos primitivos se almacenan como un tipo dinámico.  | `"values": [154, 149, 147]` | `values_string`  | `values_dynamic` |
Las matrices de objetos siempre están aplanadas, lo que genera varios eventos. | Si los objetos dentro de una matriz no tienen las propiedades de marca de tiempo ni del identificador de TS, la matriz de objetos se almacena completamente como un tipo dinámico. | `"values": [{"foo" : 140}, {"bar" : 149}]` | `values_foo_long | values_bar_long` | `values_dynamic` |

## <a name="recommended-changes-for-new-environments"></a>Cambios recomendados para los entornos nuevos

### <a name="if-your-ts-id-andor-timestamp-property-is-nested-within-an-object"></a>Si la propiedad de marca de tiempo o de identificador de TS está anidada dentro de un objeto:

Es preciso que las implementaciones nuevas coincidan con las reglas de ingesta nuevas. Por ejemplo, si el identificador de TS es `telemetry_tagId`, deberá actualizar las plantillas de Azure Resource Manager o los scripts de implementación automatizados para configurar `telemetry.tagId` como identificador de TS del entorno. Este cambio también es necesario para las marcas de tiempo del origen del evento en el objeto JSON anidado.

### <a name="if-your-payload-contains-nested-json-or-special-characters-and-you-automate-authoring-time-series-model-variable-expressions"></a>Si la carga contiene un objeto JSON anidado o caracteres especiales y se automatiza la creación de expresiones variables del [modelo de Time Series](./concepts-model-overview.md)

Actualice el código de cliente que ejecuta [TypesBatchPut](/rest/api/time-series-insights/dataaccessgen2/timeseriestypes/executebatch#typesbatchput) para que coincida con las reglas de ingesta nuevas. Por ejemplo, debería actualizar una [expresión de Time Series](/rest/api/time-series-insights/reference-time-series-expression-syntax) de `"value": {"tsx": "$event.series_value.Double"}` a una de las opciones siguientes:

* `"value": {"tsx": "$event.series.value.Double"}`
* `"value": {"tsx": "$event['series']['value'].Double"}`

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre el [almacenamiento y la entrada de Azure Time Series Insights Gen2](./concepts-ingestion-overview.md).

* Aprenda a realizar consultas de datos mediante las [API de consulta de Time Series](./concepts-query-overview.md).

* Lea más información sobre la [nueva sintaxis de Time Series Expression](/rest/api/time-series-insights/reference-time-series-expression-syntax).