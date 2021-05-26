---
title: Consultas JOIN de SQL para Azure Cosmos DB
description: Obtenga información sobre cómo combinar (JOIN) varias tablas en Azure Cosmos DB para consultar los datos.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 01/07/2021
ms.author: tisande
ms.openlocfilehash: c792a967805bdda1e51ab3a0f5521c5b1baa1be2
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110466891"
---
# <a name="joins-in-azure-cosmos-db"></a>Combinaciones en Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

En una base de datos relacional, las combinaciones entre tablas son la consecuencia lógica de diseñar esquemas normalizados. En cambio, la API de SQL usa el modelo de datos desnormalizado de los elementos sin esquemas, que es el equivalente lógico de una *autocombinación*.

Las combinaciones derivan en un producto cruzado completo de los conjuntos participantes en la combinación. El resultado de una combinación de N es un conjunto de tuplas de N elementos, donde cada valor de la tupla se asocia con el conjunto del alias participante en la combinación, accesible mediante una referencia a ese alias en otras cláusulas.

## <a name="syntax"></a>Sintaxis

El lenguaje admite la sintaxis `<from_source1> JOIN <from_source2> JOIN ... JOIN <from_sourceN>`. Esta consulta devuelve un conjunto de tuplas con valores `N`. Cada tupla tiene valores generados mediante la iteración de todos los alias de contenedor en los conjuntos correspondientes. 

Observemos la siguiente cláusula FROM: `<from_source1> JOIN <from_source2> JOIN ... JOIN <from_sourceN>`  
  
 Los orígenes definirán `input_alias1, input_alias2, …, input_aliasN`. Esta cláusula FROM devuelve un conjunto de N tuplas (tupla con N valores). Cada tupla tiene valores generados mediante la iteración de todos los alias de contenedor en los conjuntos correspondientes.  
  
**Ejemplo 1**: 2 orígenes  
  
- `<from_source1>` tendrá ámbito de contenedor y representa el conjunto {A, B, C}.  
  
- `<from_source2>` tendrá ámbito de documento, hará referencia a input_alias1 y representará los conjuntos:  
  
    {1, 2} para `input_alias1 = A,`  
  
    {3} para `input_alias1 = B,`  
  
    {4, 5} para `input_alias1 = C,`  
  
- La cláusula FROM `<from_source1> JOIN <from_source2>` dará como resultado las siguientes tuplas:  
  
    (`input_alias1, input_alias2`):  
  
    `(A, 1), (A, 2), (B, 3), (C, 4), (C, 5)`  
  
**Ejemplo 2**: 3 orígenes  
  
- `<from_source1>` tendrá ámbito de contenedor y representa el conjunto {A, B, C}.  
  
- `<from_source2>` tendrá ámbito de documento, hará referencia a `input_alias1` y representará los conjuntos:  
  
    {1, 2} para `input_alias1 = A,`  
  
    {3} para `input_alias1 = B,`  
  
    {4, 5} para `input_alias1 = C,`  
  
- `<from_source3>` tendrá ámbito de documento, hará referencia a `input_alias2` y representará los conjuntos:  
  
    {100, 200} para `input_alias2 = 1,`  
  
    {300} para `input_alias2 = 3,`  
  
- La cláusula FROM `<from_source1> JOIN <from_source2> JOIN <from_source3>` dará como resultado las siguientes tuplas:  
  
    (input_alias1, input_alias2, input_alias3):  
  
    (A, 1, 100), (A, 1, 200), (B, 3, 300)  
  
  > [!NOTE]
  > Faltan tuplas para otros valores de `input_alias1` y `input_alias2`, para los cuales `<from_source3>` no devolvió ningún valor.  
  
**Ejemplo 3**: 3 orígenes  
  
- `<from_source1>` tendrá ámbito de contenedor y representa el conjunto {A, B, C}.  
  
- `<from_source2>` tendrá ámbito de documento, hará referencia a `input_alias1` y representará los conjuntos:  
  
    {1, 2} para `input_alias1 = A,`  
  
    {3} para `input_alias1 = B,`  
  
    {4, 5} para `input_alias1 = C,`  
  
- `<from_source3>` tendrá el ámbito `input_alias1` y representará los conjuntos:  
  
    {100, 200} para `input_alias2 = A,`  
  
    {300} para `input_alias2 = C,`  
  
- La cláusula FROM `<from_source1> JOIN <from_source2> JOIN <from_source3>` dará como resultado las siguientes tuplas:  
  
    (`input_alias1, input_alias2, input_alias3`):  
  
    (A, 1, 100), (A, 1, 200), (A, 2, 100), (A, 2, 200), (C, 4, 300), (C, 5, 300)  
  
  > [!NOTE]
  > Esto dio como resultado un producto cruzado entre `<from_source2>` y `<from_source3>`, ya que ambos tienen como ámbito el mismo `<from_source1>`.  Esto dio como resultado 4 (2 x 2) tuplas con valor A, 0 tuplas con valor B (1 x 0) y 2 (2 x 1) tuplas con valor C.  
  
## <a name="examples"></a>Ejemplos

En los ejemplos siguientes se muestra cómo funciona la cláusula JOIN. Antes de ejecutar estos ejemplos, cargue los [datos de familia](sql-query-getting-started.md#upload-sample-data) de ejemplo. En el siguiente ejemplo, el resultado está vacío porque tanto el producto vectorial de cada elemento del origen como un conjunto vacío están vacíos:

```sql
    SELECT f.id
    FROM Families f
    JOIN f.NonExistent
```

El resultado es el siguiente:

```json
    [{
    }]
```

En el ejemplo siguiente, la combinación es un producto vectorial entre dos objetos JSON, la raíz del elemento `id` y la subraíz `children`. El hecho de que `children` sea una matriz no es eficaz en la combinación, porque se trata de una sola raíz que es la matriz `children`. El resultado contiene únicamente dos resultados, ya que el producto vectorial de cada elemento con la matriz genera exactamente solo un elemento.

```sql
    SELECT f.id
    FROM Families f
    JOIN f.children
```

Los resultados son:

```json
    [
      {
        "id": "AndersenFamily"
      },
      {
        "id": "WakefieldFamily"
      }
    ]
```

En el ejemplo siguiente se muestra una combinación más convencional:

```sql
    SELECT f.id
    FROM Families f
    JOIN c IN f.children
```

Los resultados son:

```json
    [
      {
        "id": "AndersenFamily"
      },
      {
        "id": "WakefieldFamily"
      },
      {
        "id": "WakefieldFamily"
      }
    ]
```

El origen FROM de la cláusula JOIN es un iterador. Por tanto, en el ejemplo anterior, el flujo es:  

1. Amplíe cada elemento secundario `c` en la matriz.
2. Aplique un producto vectorial con la raíz del elemento `f` con cada elemento hijo `c` cuyo formato se quitó en el primer paso.
3. Por último, proyecte solo la propiedad `f` `id` del objeto raíz.

El primer elemento, `AndersenFamily`, contiene solo un elemento `children`, por lo que el conjunto de resultados contiene solo un objeto único. El segundo elemento, `WakefieldFamily`, contiene dos `children`, por lo que el producto vectorial genera dos objetos, uno para cada elemento `children`. Los campos de raíz de ambos elementos son los mismos, tal como se esperaría en un cruce de productos.

La utilidad real de la cláusula JOIN es la formación de tuplas a partir del producto vectorial con una forma que, de otro modo, es difícil de proyectar. En el siguiente ejemplo se filtra por la combinación de una tupla que permite que el usuario elija una condición satisfecha por las tuplas en general.

```sql
    SELECT 
        f.id AS familyName,
        c.givenName AS childGivenName,
        c.firstName AS childFirstName,
        p.givenName AS petName
    FROM Families f
    JOIN c IN f.children
    JOIN p IN c.pets
```

Los resultados son:

```json
    [
      {
        "familyName": "AndersenFamily",
        "childFirstName": "Henriette Thaulow",
        "petName": "Fluffy"
      },
      {
        "familyName": "WakefieldFamily",
        "childGivenName": "Jesse",
        "petName": "Goofy"
      }, 
      {
       "familyName": "WakefieldFamily",
       "childGivenName": "Jesse",
       "petName": "Shadow"
      }
    ]
```

La extensión siguiente del ejemplo anterior realiza una combinación doble. Puede ver el producto vectorial como el pseudocódigo siguiente:

```
    for-each(Family f in Families)
    {
        for-each(Child c in f.children)
        {
            for-each(Pet p in c.pets)
            {
                return (Tuple(f.id AS familyName,
                  c.givenName AS childGivenName,
                  c.firstName AS childFirstName,
                  p.givenName AS petName));
            }
        }
    }
```

`AndersenFamily` tiene un hijo que tiene una mascota, entonces, el producto vectorial genera una fila (1\*1\*1) a partir de esta familia. `WakefieldFamily` tiene dos hijos, solo uno de los cuales tiene mascotas, pero ese hijo tiene dos mascotas. El producto vectorial para esta familia genera 1\*1\*2 = 2 filas.

En el ejemplo siguiente, hay un filtro adicional en `pet`, que excluye todas las tuplas en las que el nombre de la mascota no sea `Shadow`. Puede crear tuplas a partir de matrices, filtrar por cualquiera de los elementos de la tupla y proyectar cualquier combinación de los elementos.

```sql
    SELECT 
        f.id AS familyName,
        c.givenName AS childGivenName,
        c.firstName AS childFirstName,
        p.givenName AS petName
    FROM Families f
    JOIN c IN f.children
    JOIN p IN c.pets
    WHERE p.givenName = "Shadow"
```

Los resultados son:

```json
    [
      {
       "familyName": "WakefieldFamily",
       "childGivenName": "Jesse",
       "petName": "Shadow"
      }
    ]
```

Si la consulta contiene una combinación (JOIN) y filtros, puede volver a escribir parte de ella como una [subconsulta](sql-query-subquery.md#optimize-join-expressions) para mejorar el rendimiento.

## <a name="next-steps"></a>Pasos siguientes

- [Introducción](sql-query-getting-started.md)
- [Ejemplos de .NET de Azure Cosmos DB](https://github.com/Azure/azure-cosmosdb-dotnet)
- [Subconsultas](sql-query-subquery.md)
