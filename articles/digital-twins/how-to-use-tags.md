---
title: Incorporación de etiquetas a gemelos digitales
titleSuffix: Azure Digital Twins
description: Visualización de la implementación de etiquetas en gemelos digitales
author: baanders
ms.author: baanders
ms.date: 7/22/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 5972882a582a0e612ecbed3f6a4ab70be8ba393c
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108207490"
---
# <a name="add-tags-to-digital-twins"></a>Incorporación de etiquetas a gemelos digitales 

Puede usar el concepto de etiquetas para detallar la identificación y la categorización de los gemelos digitales. En concreto, los usuarios podrían querer replicar etiquetas de sistemas existentes, como las [etiquetas de Haystack](https://project-haystack.org/doc/TagModel), en las instancias de Azure Digital Twins. 

En este documento se describen los patrones que se pueden usar para implementar etiquetas en gemelos digitales.

Las etiquetas se agregan primero como propiedades en el [modelo](concepts-models.md) que describe un gemelo digital. A continuación, esa propiedad se establece en el gemelo durante su creación en función del modelo. Después, las etiquetas se pueden usar en [consultas](concepts-query-language.md) para identificar y filtrar los gemelos.

## <a name="marker-tags"></a>Etiquetas de marcador 

Una **etiqueta de marcador** es una cadena simple que se usa para marcar o categorizar un gemelo digital, como "blue" o "red". Esta cadena es el nombre de la etiqueta y las etiquetas de marcador no tienen valor significativo; significa simplemente que está presente (o ausente). 

### <a name="add-marker-tags-to-model"></a>Incorporación de etiquetas de marcador al modelo 

Las etiquetas de marcador se modelan como asignación [DTDL](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) de `string` a `boolean`. Se omite el booleano `mapValue`, ya que la presencia de la etiqueta es lo único que importa. 

Este es un extracto de un modelo gemelo que implementa una etiqueta de marcador como propiedad:

:::code language="json" source="~/digital-twins-docs-samples/models/tags.json" range="2-16":::

### <a name="add-marker-tags-to-digital-twins"></a>Incorporación de etiquetas de marcador a gemelos digitales

Cuando la propiedad `tags` ya forme parte de un modelo de gemelos digitales, configure el valor de esta propiedad para establecer la etiqueta del marcador en el gemelo digital. 

Este es un ejemplo que rellena el marcador `tags` para tres gemelos:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_other.cs" id="TagPropertiesMarker":::

Este es un ejemplo de código de cómo establecer el marcador `tags` para un gemelo con el [SDK de .NET](/dotnet/api/overview/azure/digitaltwins/client):

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_other.cs" id="TagPropertiesCsharp":::

### <a name="query-with-marker-tags"></a>Consulta con etiquetas de marcador

Una vez que se han agregado etiquetas a gemelos digitales, estas se pueden usar para filtrar los gemelos en las consultas. 

Esta es una consulta para obtener todos los gemelos etiquetados como "red": 

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="QueryMarkerTags1":::

También puede combinar etiquetas para consultas más complejas. Esta es una consulta para obtener todos los gemelos etiquetados como "round" y no "red": 

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="QueryMarkerTags2":::

## <a name="value-tags"></a>Etiquetas de valor 

Una **etiqueta de valor** es un par clave-valor que se utiliza para asignar un valor a cada etiqueta, como `"color": "blue"` o `"color": "red"`. Una vez creada la etiqueta de valor, también se puede usar como etiqueta de marcador; para ello, hay que omitir el valor de la etiqueta. 

### <a name="add-value-tags-to-model"></a>Incorporación de etiquetas de valor al modelo 

Las etiquetas de valor se modelan como asignación [DTDL](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) de `string` a `string`. Tanto `mapKey` como `mapValue` tienen significado. 

Este es un extracto de un modelo gemelo que implementa una etiqueta de valor como propiedad:

:::code language="json" source="~/digital-twins-docs-samples/models/tags.json" range="17-31":::

### <a name="add-value-tags-to-digital-twins"></a>Incorporación de etiquetas de valor a gemelos digitales

Al igual que con las etiquetas de marcador, puede establecer la etiqueta de valor en un gemelo digital al configurar el valor de esta propiedad `tags` del modelo. Para usar una etiqueta de valor como etiqueta de marcador, puede establecer el campo `tagValue` en el valor de cadena vacía (`""`). 

Este es un ejemplo que rellena el valor `tags` para tres gemelos:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_other.cs" id="TagPropertiesValue":::

Tenga en cuenta que `red` y `purple` se usan como etiquetas de marcador en este ejemplo.

### <a name="query-with-value-tags"></a>Consulta con etiquetas de valor

Al igual que con las etiquetas de marcador, puede usar etiquetas de valor para filtrar gemelos en las consultas. También puede usar etiquetas de valor y etiquetas de marcador juntas.

En el ejemplo anterior, `red` se usa como etiqueta de marcador. Recuerde que esta es una consulta para obtener todos los gemelos etiquetados como "red": 

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="QueryMarkerTags1":::

Esta es una consulta para obtener todas las entidades que son "small" (etiqueta de valor) y no "red": 

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="QueryMarkerValueTags":::

## <a name="next-steps"></a>Pasos siguientes

Más información sobre el diseño y la administración de modelos de gemelos digitales:
* [Procedimiento: Administración de modelos DTDL](how-to-manage-model.md)

Más información sobre la consulta del grafo gemelo:
* [Procedimiento: Consulta del grafo de gemelos](how-to-query-graph.md)
