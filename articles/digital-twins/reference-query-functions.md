---
title: 'Referencia del lenguaje de consulta de Azure Digital Twins: funciones'
titleSuffix: Azure Digital Twins
description: Documentación de referencia para las funciones de lenguaje de consulta de Azure Digital Twins
author: baanders
ms.author: baanders
ms.date: 03/22/2021
ms.topic: article
ms.service: digital-twins
ms.openlocfilehash: 80b32907499d3c8588a7a565972f49923abbe5c7
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108296478"
---
# <a name="azure-digital-twins-query-language-reference-functions"></a>Referencia del lenguaje de consulta de Azure Digital Twins: funciones

Este documento contiene información de referencia sobre las **funciones** del [lenguaje de consulta de Azure Digital Twins](concepts-query-language.md).

## <a name="endswith"></a>ENDSWITH

Función de cadena que se usa para determinar si una cadena proporcionada termina en otra cadena determinada. 

### <a name="syntax"></a>Sintaxis

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="EndsWithSyntax":::

### <a name="arguments"></a>Argumentos

* `<string-to-check>`: cadena de la que se comprobará el final.
* `<ending-string>`: cadena que representa el final que se va a comprobar.

### <a name="returns"></a>Devoluciones

Valor booleano que indica si la primera expresión de cadena finaliza con la segunda.

### <a name="example"></a>Ejemplo

La consulta siguiente devuelve todos los gemelos digitales cuyos identificadores terminan en `-small`.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="EndsWithExample":::

## <a name="is_defined"></a>IS_DEFINED

Función de comprobación y conversión de tipos para comprobar si una propiedad está definida.

Solo se admite cuando el valor de la propiedad es de tipo primitivo. Los tipos primitivos son cadena, booleano, numérico o `null`. No se admiten `DateTime`, los tipos de objeto ni las matrices.

### <a name="syntax"></a>Sintaxis

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="IsDefinedSyntax":::

### <a name="arguments"></a>Argumentos

`<property>`, una propiedad para determinar si se ha definido. La propiedad debe ser de un tipo primitivo.

### <a name="returns"></a>Devoluciones

Valor booleano que indica si se ha asignado un valor a la propiedad.

### <a name="example"></a>Ejemplo

La consulta siguiente devuelve todos los gemelos digitales que tienen una propiedad *Location* definida.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="IsDefinedExample":::

## <a name="is_of_model"></a>IS_OF_MODEL

Función de comprobación y conversión de tipos para determinar si un gemelo es de un tipo de modelo determinado. Incluye los modelos que se heredan del modelo especificado.

### <a name="syntax"></a>Sintaxis

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="IsOfModelSyntax":::

### <a name="arguments"></a>Argumentos

Requerido:
* `<model-ID>`: identificador del modelo que se va a comprobar.

Opcional:
* `<twin-collection>`: especifique la colección gemela que desea buscar cuando haya más de una (por ejemplo, cuando se usa una cláusula `JOIN`).
* `exact`: requiere una coincidencia exacta. Si no se establece este parámetro, el conjunto de resultados incluirá gemelos con modelos que se heredan del modelo especificado.

### <a name="returns"></a>Devoluciones

Valor booleano que indica si el gemelo especificado coincide con el tipo de modelo especificado.

### <a name="example"></a>Ejemplo

La consulta siguiente devuelve gemelos de la colección de DT que son exactamente del tipo de modelo `dtmi:example:room;1`.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="IsOfModelExample":::

## <a name="is_bool"></a>IS_BOOL

Función de comprobación y conversión de tipos para determinar si una expresión tiene un valor booleano.

Esta función se suele combinar con otros predicados si el programa que procesa los resultados de la consulta requiere un valor booleano y desea filtrar los casos en los que la propiedad no es un valor booleano.

### <a name="syntax"></a>Sintaxis

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="IsBoolSyntax":::

### <a name="arguments"></a>Argumentos

`<expression>`, expresión para comprobar si se trata de un valor booleano.

### <a name="returns"></a>Devoluciones

Valor booleano que indica si el tipo de la expresión especificada es un valor booleano.

### <a name="example"></a>Ejemplo

La consulta siguiente selecciona los gemelos digitales que tienen una propiedad `HasTemperature` booleana.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="IsBoolExample":::

La consulta siguiente se basa en el ejemplo anterior para seleccionar los gemelos digitales que tienen una propiedad `HasTemperature` booleana y el valor de esa propiedad es distinto de `false`.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="IsBoolNotFalseExample":::

## <a name="is_number"></a>IS_NUMBER

Función de comprobación y conversión de tipos para determinar si una expresión tiene un valor numérico.

Esta función suele combinarse con otros predicados si el programa que procesa los resultados de la consulta requiere un valor numérico y desea filtrar los casos en que la propiedad no es un número.

### <a name="syntax"></a>Sintaxis

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="IsNumberSyntax":::

### <a name="arguments"></a>Argumentos

`<expression>`, expresión para comprobar si se trata de un número.

### <a name="returns"></a>Devoluciones

Valor booleano que indica si el tipo de la expresión especificada es un número.

### <a name="example"></a>Ejemplo

La consulta siguiente selecciona los gemelos digitales que tienen una propiedad `Capacity` numérica y su valor es distinto de 0.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="IsNumberExample":::

## <a name="is_string"></a>IS_STRING

Función de comprobación y conversión de tipos para determinar si una expresión tiene un valor de cadena. 

Esta función suele combinarse con otros predicados si el programa que procesa los resultados de la consulta requiere un valor de cadena y desea filtrar los casos en que la propiedad no es una cadena.

### <a name="syntax"></a>Sintaxis

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="IsStringSyntax":::

### <a name="arguments"></a>Argumentos

`<expression>`, expresión para comprobar si se trata de una cadena.

### <a name="returns"></a>Devoluciones

Valor booleano que indica si el tipo de la expresión especificada es una cadena.

### <a name="example"></a>Ejemplo

La consulta siguiente selecciona los gemelos digitales que tienen una propiedad `Status` de propiedad de cadena y su valor es distinto de `Completed`.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="IsStringExample":::

## <a name="is_null"></a>IS_NULL

Función de comprobación y conversión de tipos para determinar si el valor de una expresión es `null`.

### <a name="syntax"></a>Sintaxis

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="IsNullSyntax":::

### <a name="arguments"></a>Argumentos

`<expression>`, expresión para comprobar si se trata de un valor NULL.

### <a name="returns"></a>Devoluciones

Valor booleano que indica si el tipo de la expresión especificada es `null`.

### <a name="example"></a>Ejemplo

La consulta siguiente devuelve gemelos que no tienen un valor NULL para Temperature. Para obtener más información sobre el operador `NOT` utilizado en esta consulta, consulte [Referencia del lenguaje de consulta de Azure Digital Twins: operadores](reference-query-operators.md#logical-operators).

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="IsNullExample":::

## <a name="is_primitive"></a>IS_PRIMITIVE

Función de comprobación y conversión de tipos para determinar si el valor de una expresión es de un tipo primitivo (cadena, booleano, numérico o `null`).

Esta función suele combinarse con otros predicados si el programa que procesa los resultados de la consulta requiere un valor de tipo primitivo y desea filtrar los casos en que la propiedad no es primitiva.

### <a name="syntax"></a>Sintaxis

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="IsPrimitiveSyntax":::

### <a name="arguments"></a>Argumentos

`<expression>`, expresión para comprobar si se trata de un tipo primitivo.

### <a name="returns"></a>Devoluciones

Valor booleano que indica si el tipo de la expresión especificada es uno de los tipos primitivos (cadena, booleano, numérico o `null`).

### <a name="example"></a>Ejemplo

La consulta siguiente devuelve la propiedad `area` de la instancia de Factory con el identificador de "ABC", solo si la propiedad `area` es un tipo primitivo. Para obtener más información sobre la proyección de determinadas columnas en el resultado de la consulta (como hace esta consulta con `area`), consulte [Referencia del lenguaje de consulta de Azure Digital Twins: cláusula SELECT](reference-query-clause-select.md#select-columns-with-projections).

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="IsPrimitiveExample":::

## <a name="is_object"></a>IS_OBJECT

Función de comprobación y conversión de tipos para determinar si el valor de una expresión es de un tipo de objeto JSON.

Esta función se suele combinar con otros predicados si el programa que procesa los resultados de la consulta requiere un objeto JSON y desea filtrar los casos en que el valor no es un objeto JSON.

### <a name="syntax"></a>Sintaxis

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="IsObjectSyntax":::

### <a name="arguments"></a>Argumentos

`<expression>`, expresión para comprobar si se trata de un tipo de objeto.

### <a name="returns"></a>Devoluciones

Valor booleano que indica si el tipo de la expresión especificada es un objeto JSON.

### <a name="example"></a>Ejemplo

La consulta siguiente selecciona todos los gemelos digitales donde se trata de un objeto denominado `MapObject` y no tiene una propiedad secundaria `TemperatureReading`.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="IsObjectExample":::

## <a name="startswith"></a>STARTSWITH

Función de cadena que se usa para determinar si una cadena proporcionada comienza con otra cadena determinada. 

### <a name="syntax"></a>Sintaxis

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="StartsWithSyntax":::

### <a name="arguments"></a>Argumentos

* `<string-to-check>`: cadena de la que se va a comprobar el principio.
* `<beginning-string>`: cadena que representa el principio que se va a comprobar.

### <a name="returns"></a>Devoluciones

Valor booleano que indica si la primera expresión de cadena empieza con la segunda.

### <a name="example"></a>Ejemplo

La consulta siguiente devuelve todos los gemelos digitales cuyos identificadores comienzan por `area1-`.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" ID="StartsWithExample":::
