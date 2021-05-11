---
title: 'Referencia del lenguaje de consulta de Azure Digital Twins: cláusulas FROM'
titleSuffix: Azure Digital Twins
description: Documentación de referencia para la cláusula FROM del lenguaje de consulta de Azure Digital Twins
author: baanders
ms.author: baanders
ms.date: 03/31/2021
ms.topic: article
ms.service: digital-twins
ms.openlocfilehash: 07dd3319416f6b8cad4cd77d69da81020569788f
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108296575"
---
# <a name="azure-digital-twins-query-language-reference-from-clause"></a>Referencia del lenguaje de consulta de Azure Digital Twins: cláusula FROM

Este documento contiene información de referencia sobre la **cláusula FROM** para el [lenguaje de consulta de Azure Digital Twins](concepts-query-language.md).

La cláusula FROM es la segunda parte de una consulta. Especifica la colección y las combinaciones en las que actuará la consulta.

Esta cláusula es necesaria para todas las consultas.

## <a name="select--from-digitaltwins"></a>SELECT ... FROM DIGITALTWINS

Use `FROM DIGITALTWINS` (no distingue mayúsculas de minúsculas) para hacer referencia a toda la colección de gemelos digitales de una instancia.

Opcionalmente, puede agregar un nombre a la colección de gemelos digitales agregando el nombre al final de la instrucción.

### <a name="syntax"></a>Sintaxis

Básico:

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="FromDigitalTwinsSyntax":::

Para dar nombre a la colección:

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="FromDigitalTwinsNamedSyntax":::

### <a name="examples"></a>Ejemplos

Esta es una consulta básica. La consulta siguiente devuelve todos los gemelos digitales de la instancia. 

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="FromDigitalTwinsExample":::

Esta es una consulta con una colección con nombre. La consulta siguiente asigna un nombre `T` a la colección y sigue devolviendo todos los gemelos digitales de la instancia.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="FromDigitalTwinsNamedExample":::

## <a name="select--from-relationships"></a>SELECT ... FROM RELATIONSHIPS

Use `FROM RELATIONSHIPS` (no distingue mayúsculas de minúsculas) para hacer referencia a toda la colección de relaciones de una instancia.

Opcionalmente, puede agregar un nombre a la colección de relaciones agregando el nombre al final de la instrucción.

>[!NOTE]
> Esta característica no se puede combinar con `JOIN`.

### <a name="syntax"></a>Sintaxis

Básico:

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="FromRelationshipsSyntax":::

Para dar nombre a la colección:

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="FromRelationshipsNamedSyntax":::

### <a name="examples"></a>Ejemplos

Esta es una consulta que devuelve todas las relaciones de la instancia. 

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="FromRelationshipsExample":::

Esta es una consulta que devuelve todas las relaciones procedentes de gemelos `A`, `B`, `C` o `D`.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="FromRelationshipsFilteredExample":::

## <a name="using-from-and-join-together"></a>Uso de FROM y JOIN juntos

La cláusula `FROM` se puede combinar con la cláusula `JOIN` para expresar recorridos entre entidades en el grafo de Azure Digital Twins.

Para obtener más información sobre la cláusula `JOIN` y la creación de consultas transversales de grafos, vea [Referencia del lenguaje de consulta de Azure Digital Twins: cláusula JOIN](reference-query-clause-join.md).

## <a name="limitations"></a>Limitaciones

Los límites siguientes se aplican a las consultas que usan la cláusula `FROM`.
* [Sin subconsultas](#no-subqueries)
* [Elección entre FROM RELATIONSHIPS o JOIN](#choose-from-relationships-or-join)

Consulte las siguientes secciones para obtener más detalles.

### <a name="no-subqueries"></a>Sin subconsultas

No se admiten subconsultas en la instrucción `FROM`.

#### <a name="example-negative"></a>Ejemplo (negativo)

En la consulta siguiente se muestra un ejemplo de lo que **no** se puede hacer según esta limitación.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="FromNegativeExample":::

### <a name="choose-from-relationships-or-join"></a>Elección entre FROM RELATIONSHIPS o JOIN

La característica `FROM RELATIONSHIPS` no se puede combinar con `JOIN`. Tendrá que seleccionar cuál de estas opciones funciona mejor para la información que desea seleccionar.


