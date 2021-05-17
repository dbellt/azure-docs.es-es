---
title: 'Referencia del lenguaje de consulta de Azure Digital Twins: cláusulas WHERE'
titleSuffix: Azure Digital Twins
description: Documentación de referencia para la cláusula WHERE del lenguaje de consulta de Azure Digital Twins
author: baanders
ms.author: baanders
ms.date: 03/31/2021
ms.topic: article
ms.service: digital-twins
ms.openlocfilehash: 58c0d35178c2683bd2626ad0b9d9209c2819e463
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109789486"
---
# <a name="azure-digital-twins-query-language-reference-where-clause"></a>Referencia del lenguaje de consulta de Azure Digital Twins: cláusula WHERE

Este documento contiene información de referencia sobre la **cláusula WHERE** para el [lenguaje de consulta de Azure Digital Twins](concepts-query-language.md).

La cláusula WHERE es la última parte de una consulta. Se usa para filtrar los elementos que se devuelven en función de condiciones específicas.

Esta cláusula es opcional durante la consulta.

## <a name="core-syntax-where"></a>Sintaxis principal: WHERE

La cláusula WHERE se usa junto con una condición booleana para filtrar los resultados de la consulta. 

Una condición puede ser una [función](reference-query-functions.md) que se evalúa como un resultado booleano. También puede crear su propia instrucción booleana mediante las propiedades de los gemelos y las relaciones (a las que se accede con `.`) mediante un [operador](reference-query-operators.md) de comparación o de tipo contenido.

### <a name="syntax"></a>Sintaxis

Con propiedades y operadores:

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="WhereSyntax":::

Con una función:

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="WhereFunctionSyntax":::

### <a name="arguments"></a>Argumentos

Condición que se evalúa como un valor `Boolean`.

### <a name="examples"></a>Ejemplos

Este es un ejemplo que usa propiedades y operadores. La siguiente consulta especifica en la cláusula WHERE que solo se debe devolver el gemelo cuyo valor de `$dtId` sea Room1.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="WhereExample":::

Este es un ejemplo que usa una función. La consulta siguiente usa la función `IS_OF_MODEL` para especificar en la cláusula WHERE que solo se deben devolver los gemelos con un modelo de `dtmi:sample:Room;1`. Para obtener más información sobre la función `IS_OF_MODEL`, consulte la [Referencia del lenguaje de consulta de Azure Digital Twins: Funciones](reference-query-functions.md#is_of_model).

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="WhereFunctionExample":::