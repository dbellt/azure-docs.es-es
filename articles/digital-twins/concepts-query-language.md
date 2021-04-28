---
title: Lenguaje de consulta
titleSuffix: Azure Digital Twins
description: Información sobre los aspectos básicos del lenguaje de consulta de Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 11/19/2020
ms.topic: conceptual
ms.service: digital-twins
ms.custom: contperf-fy21q2
ms.openlocfilehash: ef583d21647681cb1be730ea93a67a9339c74aae
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107930990"
---
# <a name="about-the-query-language-for-azure-digital-twins"></a>Acerca del lenguaje de consulta para Azure Digital Twins

Recuerde que el centro de Azure Digital Twins es el [grafo de gemelos](concepts-twins-graph.md), que se crea partir de gemelos digitales y relaciones. 

Este grafo se puede consultar para obtener información sobre los gemelos digitales y las relaciones que contiene. Estas consultas se escriben en un lenguaje de consulta personalizado similar a SQL al que se conoce como **lenguaje de consulta de Azure Digital Twins**. Es similar al [lenguaje de consulta de IoT Hub](../iot-hub/iot-hub-devguide-query-language.md) con muchas características comparables.

En este artículo se describen los aspectos básicos del lenguaje de consulta y sus funcionalidades. Para ver ejemplos más detallados de la sintaxis de consulta y cómo ejecutar solicitudes de consulta, consulte [ *Consulta del grafo gemelo de Azure Digital Twins*](how-to-query-graph.md).

## <a name="about-the-queries"></a>Acerca de las consultas

Puede usar el lenguaje de consulta de Azure Digital Twins para recuperar gemelos digitales según sus...
* propiedades (incluidas [propiedades de etiqueta](how-to-use-tags.md))
* modelos
* relationships
  - propiedades de las relaciones

Para enviar una consulta al servicio desde una aplicación cliente, usará la [**API de consulta**](/rest/api/digital-twins/dataplane/query) de Azure Digital Twins. Una manera de usar la API es a mediante uno de los [SDK](how-to-use-apis-sdks.md#overview-data-plane-apis) de Azure Digital Twins.

### <a name="considerations-for-querying"></a>Consideraciones para la consulta

Al escribir consultas para Azure Digital Twins, tenga en cuenta las consideraciones siguientes:
* **Recuerde la distinción entre mayúsculas y minúsculas**: todas las operaciones de consulta de Azure Digital Twins distinguen mayúsculas de minúsculas, por lo que debe tener cuidado de usar los nombres exactos definidos en los modelos. Si los nombres de propiedad están mal escritos o usan las mayúsculas de forma incorrecta, el conjunto de resultados está vacío y no se devuelven errores.
* **Comillas simples de escape**: si el texto de la consulta incluye un carácter de comilla simple en los datos, la comilla deberá utilizar el carácter `\` como escape. Este es un ejemplo que trata sobre el valor de una propiedad de *D'Souza*:

  :::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="EscapedSingleQuote":::

## <a name="reference-expressions-and-conditions"></a>Referencia: Expresiones y condiciones

En esta sección se describen los operadores y las funciones que están disponibles para escribir consultas de Azure Digital Twins. Para ver consultas de ejemplo que ilustre el uso de estas características, consulte [ *Consulta del grafo gemelo de Azure Digital Twins*](how-to-query-graph.md).

### <a name="operators"></a>Operadores

Se admiten los siguientes operadores:

| Familia | Operadores |
| --- | --- |
| Lógicos |`AND`, `OR`, `NOT` |
| De comparación | `=`, `!=`, `<`, `>`, `<=`, `>=` |
| Contiene | `IN`, `NIN` |

### <a name="functions"></a>Functions

Se admiten las funciones de conversión y comprobación de tipos siguientes:

| Función | Descripción |
| -------- | ----------- |
| `IS_DEFINED` | Devuelve un valor booleano que indica si se ha asignado un valor a la propiedad. Solo se admite cuando el valor es de tipo primitivo. Los tipos primitivos son cadena, booleano, numérico o `null`. No se admiten `DateTime`, los tipos de objeto ni las matrices. |
| `IS_OF_MODEL` | Devuelve un valor booleano que indica si el gemelo especificado coincide con el tipo de modelo especificado. |
| `IS_BOOL` | Devuelve un valor booleano que indica si el tipo de la expresión especificada es un valor booleano. |
| `IS_NUMBER` | Devuelve un valor booleano que indica si el tipo de la expresión especificada es un número. |
| `IS_STRING` | Devuelve un valor booleano que indica si el tipo de la expresión especificada es una cadena. |
| `IS_NULL` | Devuelve un valor booleano que indica si el tipo de la expresión especificada es nulo. |
| `IS_PRIMITIVE` | Devuelve un valor booleano que indica si el tipo de la expresión especificada es un tipo primitivo (cadena, booleano, numérico o `null`). |
| `IS_OBJECT` | Devuelve un valor booleano que indica si el tipo de la expresión especificada es un objeto JSON. |

Se admiten las siguientes funciones de cadena:

| Función | Descripción |
| -------- | ----------- |
| `STARTSWITH(x, y)` | Devuelve un valor booleano que indica si la primera expresión de cadena empieza con la segunda. |
| `ENDSWITH(x, y)` | Devuelve un valor booleano que indica si la primera expresión de cadena finaliza con la segunda. |

## <a name="query-limitations"></a>Limitaciones de las consultas

En esta sección se describen las limitaciones del lenguaje de consulta.

* Intervalo: Puede haber un retraso de hasta 10 segundos antes de que los cambios en la instancia se reflejen en las consultas. Por ejemplo, si completa una operación como la creación o la eliminación de gemelos con la API de DigitalTwins, es posible que el resultado no se refleje inmediatamente en las solicitudes de la API de consulta. Para que se resuelva debería bastar con esperar un poco.
* No se admiten subconsultas en la instrucción `FROM`.
* No se admite la semántica `OUTER JOIN`, lo que significa que, si la relación tiene un rango de cero, se elimina toda la "fila" del conjunto de resultados de salida.
* La profundidad transversal del grafo está restringida a cinco niveles `JOIN` por consulta.
* Las relaciones en Azure Digital Twins no se pueden consultar como entidades independientes; también debe proporcionar información sobre el gemelo de origen del que procede la relación. Esto significa que hay algunas restricciones en la operación `JOIN`, que se utiliza para consultar las relaciones, para tener la seguridad de que la consulta declara los gemelos donde esta comienza. Puede encontrar ejemplos de este escenario en [*Consulta por relación*](how-to-query-graph.md#query-by-relationship) en el artículo *Consulta del grafo de gemelos*.

## <a name="next-steps"></a>Pasos siguientes

Aprenda a escribir consultas y ver ejemplos de código de cliente en [*Procedimientos: Consulta del grafo gemelo*](how-to-query-graph.md).