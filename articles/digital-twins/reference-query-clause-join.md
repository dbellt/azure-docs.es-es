---
title: 'Referencia del lenguaje de consulta de Azure Digital Twins: cláusulas JOIN'
titleSuffix: Azure Digital Twins
description: Documentación de referencia para la cláusula JOIN del lenguaje de consulta de Azure Digital Twins
author: baanders
ms.author: baanders
ms.date: 03/31/2021
ms.topic: article
ms.service: digital-twins
ms.openlocfilehash: 7dc6827f7ebd7b034ffc00906629bafe04036fbd
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109789576"
---
# <a name="azure-digital-twins-query-language-reference-join-clause"></a>Referencia del lenguaje de consulta de Azure Digital Twins: cláusula JOIN

Este documento contiene información de referencia sobre la **cláusula JOIN** para el [lenguaje de consulta de Azure Digital Twins](concepts-query-language.md).

La cláusula `JOIN` se usa en el lenguaje de consulta de Azure Digital Twins como parte de la [cláusula FROM](reference-query-clause-from.md) cuando se desea consultar para recorrer el grafo de Azure Digital Twins.

Esta cláusula es opcional durante la consulta.

## <a name="core-syntax-join--related"></a>Sintaxis básica: JOIN... RELATED 
Dado que las relaciones de Azure Digital Twins forman parte de gemelos digitales, no de entidades independientes, la palabra clave `RELATED` se usa en las consultas `JOIN` para hacer referencia al conjunto de relaciones de un tipo determinado de la colección de gemelos. A este conjunto de relaciones se le puede asignar un nombre de colección.

Posteriormente, la consulta debe usar la cláusula `WHERE` para especificar qué gemelos o gemelos específicos se usan para admitir la consulta de relación. Para ello, se filtra por el valor `$dtId` del gemelo de origen o de destino.

### <a name="syntax"></a>Sintaxis

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="JoinSyntax":::

### <a name="example"></a>Ejemplo

La consulta siguiente selecciona todos los gemelos digitales que están relacionados con el gemelo con un identificador de *ABC* a través de una relación *contains*.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="JoinExample":::

## <a name="multiple-joins"></a>Varias cláusulas JOIN

Se admiten hasta cinco cláusulas `JOIN` en una sola consulta, lo que permite el recorrido de varios niveles de relaciones a la vez.

### <a name="syntax"></a>Sintaxis

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="MultiJoinSyntax":::

### <a name="example"></a>Ejemplo

La consulta siguiente se basa en objetos Room que contienen elementos LightPanel, y cada elemento LightPanel contiene varios elementos LightBulb. La consulta obtiene todos los elementos LightBulb de los elementos LightPanel de los objetos room1 y room2.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="MultiJoinExample":::

## <a name="limitations"></a>Limitaciones

Los límites siguientes se aplican a las consultas que usan la cláusula `JOIN`.
* [Límite de profundidad de cinco](#depth-limit-of-five)
* [Sin semántica OUTER JOIN](#no-outer-join-semantics)
* [Se requiere un gemelo de origen](#twins-required)

Consulte las siguientes secciones para obtener más detalles.

### <a name="depth-limit-of-five"></a>Límite de profundidad de cinco

La profundidad transversal del grafo está restringida a cinco niveles `JOIN` por consulta.

#### <a name="example"></a>Ejemplo

En la consulta siguiente se muestra el número máximo de `JOINs` posibles en una consulta de Azure Digital Twins. Obtiene todos los elementos LightBulb de Building1.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="MaxJoinExample":::

### <a name="no-outer-join-semantics"></a>Sin semántica OUTER JOIN

No se admite la semántica `OUTER JOIN`, lo que significa que, si la relación tiene un rango de cero, se elimina toda la "fila" del conjunto de resultados de salida.

#### <a name="example"></a>Ejemplo

Considere la posibilidad de utilizar la siguiente consulta, que ilustra un recorrido del edificio.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="NoOuterJoinExample":::

Si Building1 no contiene ninguna planta, esta consulta devolverá un conjunto de resultados vacío (en lugar de devolver una fila con un valor para Building y `undefined` para Floor).

### <a name="twins-required"></a>Gemelos necesarios

Las relaciones en Azure Digital Twins no se pueden consultar como entidades independientes; también debe proporcionar información sobre el gemelo de origen del que procede la relación. Esta se incluye como parte del uso de `JOIN` predeterminado en Azure Digital Twins a través de la palabra clave `RELATED`. 

Las consultas con una cláusula `JOIN` también deben filtrar por la propiedad `$dtId` de cualquier gemelo de la cláusula `WHERE`, a fin de aclarar qué gemelos se usan para admitir la consulta de relación.