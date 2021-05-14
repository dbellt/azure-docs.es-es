---
title: 'Referencia del lenguaje de consulta de Azure Digital Twins: cláusulas SELECT'
titleSuffix: Azure Digital Twins
description: Documentación de referencia para la cláusula SELECT del lenguaje de consulta de Azure Digital Twins
author: baanders
ms.author: baanders
ms.date: 03/31/2021
ms.topic: article
ms.service: digital-twins
ms.openlocfilehash: 58e37cb1cffadf841e6d9450f1ce908cf6f22c69
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108296494"
---
# <a name="azure-digital-twins-query-language-reference-select-clause"></a>Referencia del lenguaje de consulta de Azure Digital Twins: cláusula SELECT

Este documento contiene información de referencia sobre la **cláusula SELECT** para el [lenguaje de consulta de Azure Digital Twins](concepts-query-language.md).

La cláusula SELECT es la primera parte de una consulta. Especifica la lista de columnas que devolverá la consulta.

Esta cláusula es necesaria para todas las consultas.

## <a name="select-"></a>SELECT *

Use el carácter `*` de una instrucción SELECT para proyectar el documento del gemelo digital tal y como está, sin asignarlo a una propiedad del conjunto de resultados.

>[!NOTE]
> `SELECT *` solo es una sintaxis válida si la consulta no usa cláusula `JOIN`. Para obtener más información sobre las consultas que usan una cláusula `JOIN`, consulte [Referencia del lenguaje de consulta de Azure Digital Twins: cláusula JOIN](reference-query-clause-join.md).

### <a name="syntax"></a>Sintaxis

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="SelectSyntax":::

### <a name="returns"></a>Devoluciones

Conjunto de propiedades que devuelve la consulta.

### <a name="example"></a>Ejemplo

La consulta siguiente devuelve todos los gemelos digitales de la instancia. 

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="SelectExample":::

## <a name="select-columns-with-projections"></a>Columnas SELECT con proyecciones

Puede usar las proyecciones de la cláusula SELECT para elegir las columnas que devolverá una consulta. Puede especificar colecciones con nombre de gemelos y relaciones, o propiedades de gemelos y relaciones.

Ahora se admite la proyección de propiedades primitivas y propiedades complejas.

### <a name="syntax"></a>Sintaxis

Para proyectar una colección:

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="SelectProjectCollectionSyntax":::

Para proyectar una propiedad:

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="SelectProjectPropertySyntax":::

### <a name="returns"></a>Devoluciones

Colección de gemelos, propiedades o relaciones especificados en la proyección.

Si una propiedad incluida en la proyección no está presente para una fila de datos determinada, la propiedad tampoco estará presente en el conjunto de resultados. Para obtener un ejemplo de este comportamiento, consulte [Ejemplo de propiedad del proyecto: propiedad no presente para una fila de datos](#project-property-example-property-not-present-for-a-data-row).

### <a name="examples"></a>Ejemplos

#### <a name="example-scenario"></a>Escenario de ejemplo

Para los ejemplos siguientes, considere la posibilidad de usar un gráfico gemelo que contenga los siguientes elementos de datos:
* Un gemelo de Factory denominado `FactoryA`.
    - Contiene una propiedad denominada `name` con un valor de `FactoryA`.
* Un gemelo de consumidor denominado `Contoso`.
    - Contiene una propiedad denominada `name` con un valor de `Contoso`.
* Una relación consumerRelationship de `FactoryA` a `Contoso`, denominada `FactoryA-consumerRelationship-Contoso`.
    - Contiene una propiedad denominada `managedBy` con un valor de `Jeff`.

El diagrama siguiente ilustra este escenario:

:::row:::
    :::column:::
        :::image type="content" source="media/reference-query-clause-select/projections-graph.png" alt-text="Diagrama que muestra el gráfico de ejemplo descrito anteriormente.":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

#### <a name="project-collection-example"></a>Ejemplo de colección de proyectos

A continuación se muestra una consulta de ejemplo que proyecta una colección a partir de este gráfico. La consulta siguiente devuelve todos los gemelos digitales de la instancia, asigna el nombre `T`a la colección de gemelos completa y proyecta `T` como la colección que se va a devolver. 

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="SelectProjectCollectionExample":::

Esta es la carga JSON que devuelve esta consulta:

```json
{
  "value": [
    {
      "result": [
        {
          "T": {
            "$dtId": "FactoryA",
            "$etag": "W/\"d22267a0-fd4f-4f6b-916d-4946a30453c9\"",
            "$metadata": {
              "$model": "dtmi:contosocom:DigitalTwins:Factory;1",
              "name": {
                "lastUpdateTime": "2021-04-19T17:15:54.4977151Z"
              }
            },
            "name": "FactoryA"
          }
        },
        {
          "T": {
            "$dtId": "Contoso",
            "$etag": "W/\"a96dc85e-56ae-4061-866b-058a149e03d8\"",
            "$metadata": {
              "$model": "dtmi:com:contoso:Consumer;1",
              "name": {
                "lastUpdateTime": "2021-04-19T17:16:30.2154166Z"
              }
            },
            "name": "Contoso"
          }
        }
      ]
    }
  ],
  "continuationToken": "null"
}
```

#### <a name="project-with-join-example"></a>Ejemplo de proyecto con una cláusula JOIN

La proyección se usa normalmente para devolver una colección especificada en una cláusula `JOIN`. La consulta siguiente usa la proyección para devolver los datos de Consumer, Factory y Relationship. Para obtener más información sobre la sintaxis `JOIN` usada en el ejemplo, consulte [Referencia del lenguaje de consulta de Azure Digital Twins: cláusula JOIN](reference-query-clause-join.md).

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="SelectProjectJoinExample":::

Esta es la carga JSON que devuelve esta consulta:

```json
{
  "value": [
    {
      "result": [
        {
          "Consumer": {
            "$dtId": "Contoso",
            "$etag": "W/\"a96dc85e-56ae-4061-866b-058a149e03d8\"",
            "$metadata": {
              "$model": "dtmi:com:contoso:Consumer;1",
              "name": {
                "lastUpdateTime": "2021-04-19T17:16:30.2154166Z"
              }
            },
            "name": "Contoso"
          },
          "Factory": {
            "$dtId": "FactoryA",
            "$etag": "W/\"d22267a0-fd4f-4f6b-916d-4946a30453c9\"",
            "$metadata": {
              "$model": "dtmi:contosocom:DigitalTwins:Factory;1",
              "name": {
                "lastUpdateTime": "2021-04-19T17:15:54.4977151Z"
              }
            },
            "name": "FactoryA"
          },
          "Relationship": {
            "$etag": "W/\"f01e07c1-19e4-4bbe-a12d-f5761e86d3e8\"",
            "$relationshipId": "FactoryA-consumerRelationship-Contoso",
            "$relationshipName": "consumerRelationship",
            "$sourceId": "FactoryA",
            "$targetId": "Contoso",
            "managedBy": "Jeff"
          }
        }
      ]
    }
  ],
  "continuationToken": "null"
}
```

#### <a name="project-property-example"></a>Ejemplo de propiedad del proyecto

A continuación se muestra un ejemplo que proyecta una propiedad. La consulta siguiente usa la proyección para devolver la propiedad `name` del gemelo Consumer y la propiedad `managedBy` de la relación.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="SelectProjectPropertyExample":::

Esta es la carga JSON que devuelve esta consulta:

```json
{
  "value": [
    {
      "result": [
        {
          "managedBy": "Jeff",
          "name": "Contoso"
        }
      ]
    }
  ],
  "continuationToken": "null"
}
```

#### <a name="project-property-example-property-not-present-for-a-data-row"></a>Ejemplo de propiedad del proyecto: propiedad no presente para una fila de datos

Si una propiedad incluida en la proyección no está presente para una fila de datos determinada, la propiedad tampoco estará presente en el conjunto de resultados.

Considere para este ejemplo un conjunto de gemelos que represente personas. Algunos de los gemelos tienen edades asociadas, pero otros no.

A continuación se muestra una consulta que proyecta las propiedades `name` y `age`:

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="SelectProjectPropertyNotPresentExample":::

El resultado podría tener un aspecto parecido al siguiente, donde falta la propiedad `age` de algunos gemelos en el resultado en el que los gemelos no tienen esta propiedad.

```json
{
  "value": [
    {
      "result": [
        {
          "name": "John",
          "age": 27
        },
        {
          "name": "Keanu"
        }
      ]
    }
  ],
  "continuationToken": "null"
}
```

## <a name="select-count"></a>SELECT COUNT

Use este método para que se cuente el número de elementos del conjunto de resultados y se devuelva ese número.

### <a name="syntax"></a>Sintaxis

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="SelectCountSyntax":::

### <a name="arguments"></a>Argumentos

Ninguno.

### <a name="returns"></a>Devoluciones

Valor `int`.

### <a name="example"></a>Ejemplo

La consulta siguiente devuelve el recuento de todos los gemelos digitales de la instancia.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="SelectCountExample":::

La consulta siguiente devuelve el recuento de todas las relaciones de la instancia.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="SelectCountRelationshipsExample":::

## <a name="select-top"></a>SELECT TOP

Use este método para que se devuelva solo un determinado número de elementos principales que cumplan los requisitos de consulta.

### <a name="syntax"></a>Sintaxis

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="SelectTopSyntax":::

### <a name="arguments"></a>Argumentos

Valor `int` que especifica el número de elementos principales que se van a seleccionar.

### <a name="returns"></a>Devoluciones

Colección de gemelos.

### <a name="example"></a>Ejemplo

La consulta siguiente devuelve solo los cinco primeros gemelos digitales de la instancia.

:::code language="sql" source="~/digital-twins-docs-samples/queries/reference.sql" id="SelectTopExample":::