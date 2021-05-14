---
title: Indexación en Azure Cosmos DB
description: Conozca el funcionamiento de la indexación en Azure Cosmos DB, y cómo se admiten los diferentes tipos de índices como los compuestos, espaciales o de intervalo.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 04/27/2021
ms.author: tisande
ms.openlocfilehash: fec7ed32b236dd0a5f9c0663209b5c2f44e05b29
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108166746"
---
# <a name="indexing-in-azure-cosmos-db---overview"></a>Indexación en Azure Cosmos DB: introducción
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB es una base de datos independiente del esquema que le permite iterar en la aplicación sin tener que tratar con la administración de esquemas o índices. De forma predeterminada, Azure Cosmos DB indexa automáticamente todas las propiedades de todos los elementos de su [contenedor](account-databases-containers-items.md#azure-cosmos-containers) sin tener que definir ningún esquema ni configurar índices secundarios.

El objetivo de este artículo es explicar cómo Azure Cosmos DB indexa datos y cómo usa índices para mejorar el rendimiento de las consultas. Se recomienda revisar esta sección antes de explorar cómo personalizar las [directivas de indexación](index-policy.md).

## <a name="from-items-to-trees"></a>De elementos a árboles

Cada vez que un elemento se almacena en un contenedor, su contenido se proyecta como un documento JSON y luego se convierte en una representación de árbol. Eso significa que todas las propiedades de ese elemento se representan como nodos de un árbol. Para todas las propiedades de primer nivel del elemento, se crea un pseudonodo raíz como elemento primario. Los nodos hoja contienen los valores escalares reales pertenecientes a un elemento.

Por ejemplo, considere este elemento:

```json
    {
        "locations": [
            { "country": "Germany", "city": "Berlin" },
            { "country": "France", "city": "Paris" }
        ],
        "headquarters": { "country": "Belgium", "employees": 250 },
        "exports": [
            { "city": "Moscow" },
            { "city": "Athens" }
        ]
    }
```

Se podría representar mediante el árbol siguiente:

:::image type="content" source="./media/index-overview/item-as-tree.png" alt-text="El elemento anterior representado como árbol" border="false":::

Observe cómo se codifican las matrices en el árbol: cada entrada de una matriz obtiene un nodo intermedio etiquetado con el índice de esa entrada dentro de la matriz (0, 1, etc.).

## <a name="from-trees-to-property-paths"></a>De árboles a rutas de acceso de propiedades

El motivo por el que Azure Cosmos DB transforma los elementos en árboles es porque permite hacer referencia a las propiedades mediante sus rutas de acceso dentro de esos árboles. Para obtener la ruta de acceso de una propiedad, podemos recorrer el árbol desde el nodo raíz hasta esa propiedad y concatenar las etiquetas de cada nodo recorrido.

Estas son las rutas de acceso de cada propiedad del elemento de ejemplo descrito anteriormente:

- /locations/0/country: "Germany"
- /locations/0/city: "Berlin"
- /locations/1/country: "France"
- /locations/1/city: "Paris"
- /headquarters/country: "Belgium"
- /headquarters/employees: 250
- /exports/0/city: "Moscow"
- /exports/1/city: "Athens"

Cuando se escribe un elemento, Azure Cosmos DB indexa eficazmente la ruta de acceso de cada propiedad y su valor correspondiente.

## <a name="types-of-indexes"></a><a id="index-types"></a>Tipos de índice

Actualmente, Azure Cosmos DB admite tres tipos de índices. Puede configurar estos tipos de índice al definir la directiva de indexación.

### <a name="range-index"></a>Índice de rangos

El **índice de rangos** se basa en una estructura de árbol ordenada. El tipo de índice de rango se usa para:

- Consultas de igualdad:

    ```sql
       SELECT * FROM container c WHERE c.property = 'value'
    ```


   ```sql
   SELECT * FROM c WHERE c.property IN ("value1", "value2", "value3")
   ```

   Coincidencia de igualdad numérica en un elemento de matriz
   ```sql
    SELECT * FROM c WHERE ARRAY_CONTAINS(c.tags, "tag1")
   ```

- Consultas por rango:

   ```sql
   SELECT * FROM container c WHERE c.property > 'value'
   ```
    (funciona para `>`, `<`, `>=`, `<=`, `!=`).

- Comprobación de la presencia de una propiedad:

   ```sql
   SELECT * FROM c WHERE IS_DEFINED(c.property)
   ```

- Funciones del sistema de cadena:

   ```sql
   SELECT * FROM c WHERE CONTAINS(c.property, "value")
   ```

   ```sql
   SELECT * FROM c WHERE STRINGEQUALS(c.property, "value")
   ```

- Consultas `ORDER BY`:

   ```sql
   SELECT * FROM container c ORDER BY c.property
   ```

- Consultas `JOIN`:

   ```sql
   SELECT child FROM container c JOIN child IN c.properties WHERE child = 'value'
   ```

Los índices de rango se pueden usar en valores escalares (cadena o número). La directiva de indexación predeterminada para los contenedores recién creados exige que se usen índices de intervalo para todas las cadenas o números. Para obtener información sobre cómo configurar los índices de rango, consulte los [ejemplos de la directiva de indexación de rangos](how-to-manage-indexing-policy.md#range-index).

> [!NOTE]
> Una cláusula `ORDER BY` que ordena por una sola propiedad *siempre* necesita un índice de rango y dará error si la ruta de acceso a la que hace referencia no tiene uno. Del mismo modo, una consulta `ORDER BY` que se ordena por varias propiedades *siempre* necesita un índice compuesto.

### <a name="spatial-index"></a>Índice espacial

Los índices **espaciales** permiten realizar consultas eficaces en objetos geoespaciales como puntos, líneas, polígonos y multipolígonos. Estas consultas usan las palabras clave ST_DISTANCE, ST_WITHIN, ST_INTERSECTS. A continuación se muestran algunos ejemplos que usan el tipo de índice espacial:

- Consultas de distancia geoespaciales:

   ```sql
   SELECT * FROM container c WHERE ST_DISTANCE(c.property, { "type": "Point", "coordinates": [0.0, 10.0] }) < 40
   ```

- Información geoespacial dentro de consultas:

   ```sql
   SELECT * FROM container c WHERE ST_WITHIN(c.property, {"type": "Point", "coordinates": [0.0, 10.0] })
   ```

- Consultas de intersecciones geoespaciales:

   ```sql
   SELECT * FROM c WHERE ST_INTERSECTS(c.property, { 'type':'Polygon', 'coordinates': [[ [31.8, -5], [32, -5], [31.8, -5] ]]  })  
   ```

Los índices espaciales pueden usarse en objetos [GeoJSON](./sql-query-geospatial-intro.md) con un formato correcto. Actualmente, se admiten Points, LineStrings, Polygons y MultiPolygons. Para usar este tipo de índice, establézcalo mediante la propiedad `"kind": "Range"` al configurar la directiva de indexación. Para obtener información sobre cómo configurar los índices espaciales, vea [ejemplos de la directiva de indexación espacial](how-to-manage-indexing-policy.md#spatial-index).

### <a name="composite-indexes"></a>Índices compuestos

Los índices **compuestos** aumentan la eficacia al realizar operaciones en varios campos. El tipo de índice compuesto se usa con:

- Consultas `ORDER BY` en varias propiedades:

```sql
 SELECT * FROM container c ORDER BY c.property1, c.property2
```

- Consultas con un filtro y `ORDER BY`. Estas consultas pueden emplear un índice compuesto si la propiedad de filtro se agrega a la cláusula `ORDER BY`.

```sql
 SELECT * FROM container c WHERE c.property1 = 'value' ORDER BY c.property1, c.property2
```

- Consultas con un filtro en dos o más propiedades en las que al menos una propiedad es un filtro de igualdad

```sql
 SELECT * FROM container c WHERE c.property1 = 'value' AND c.property2 > 'value'
```

Siempre y cuando un predicado de filtro use un tipo de índice, el motor de consultas evaluará ese primero antes de examinar el resto. Por ejemplo, si tiene una consulta SQL como `SELECT * FROM c WHERE c.firstName = "Andrew" and CONTAINS(c.lastName, "Liu")`

* La consulta anterior primero filtrará las entradas en las que firstName = "Andrew" mediante el índice. Después, pasa todas las entradas firstName = "Andrew" a través de una canalización subsiguiente para evaluar el predicado de filtro CONTAINS.

* Puede acelerar las consultas y evitar exámenes de todo el contenedor cuando emplee funciones que no usen el índice (por ejemplo, CONTAINS) mediante la incorporación de predicados de filtro adicionales que utilicen el índice. El orden de las cláusulas de filtro no es importante. El motor de consultas determinará qué predicados son más selectivos y ejecutará la consulta en consecuencia.

Para obtener información sobre cómo configurar los índices compuestos, consulte los [ejemplos de la directiva de indexación compuesta](how-to-manage-indexing-policy.md#composite-index).

## <a name="index-usage"></a>Uso de índices

El motor de consultas tiene cinco maneras de evaluar los filtros de consulta, ordenados del más eficaz al menos eficaz:

- Búsqueda de índice
- Examen preciso del índice
- Examen expandido del índice
- Examen completo del índice
- Examen completo

Al indexar las rutas de acceso de propiedad, el motor de consultas usará automáticamente el índice de la manera más eficaz posible. Además de indexar rutas de acceso de propiedad nuevas, no es necesario configurar nada a fin de optimizar el modo en que las consultas usan el índice. El cargo por RU de una consulta es una combinación del cargo por RU del uso del índice y el cargo por RU de la carga de elementos.

En esta tabla se resumen las distintas formas en que se usan los índices en Azure Cosmos DB:

| Tipo de búsqueda de índice  | Descripción                                                  | Ejemplos comunes                                 | Cargo por RU del uso del índice                                   | Cargo por RU de la carga de elementos desde el almacén de datos transaccionales                   |
| ------------------ | ------------------------------------------------------------ | ----------------------------------------------- | ------------------------------------------------------------ | --------------------------------------------------- |
| Búsqueda de índice         | Lectura solo de los valores indexados necesarios y carga solo de los elementos coincidentes desde el almacén de datos transaccionales | Filtros de igualdad, IN                            | Constante por filtro de igualdad                                                     | Aumenta en función de la cantidad de elementos en los resultados de la consulta |
| Examen preciso del índice | Búsqueda binaria de los valores indexados necesarios y carga solo de los elementos coincidentes desde el almacén de datos transaccionales | Comparaciones de rangos (>, <, <= o >=), StartsWith | Comparable a la búsqueda de índice, aumenta ligeramente en función de la cardinalidad de las propiedades indexadas | Aumenta en función de la cantidad de elementos en los resultados de la consulta |
| Examen expandido del índice | Búsqueda optimizada (pero menos eficaz que una búsqueda binaria) de valores indexados y carga solo de los elementos coincidentes desde el almacén de datos transaccionales | StartsWith (sin distinguir mayúsculas de minúsculas), StringEquals (sin distinguir mayúsculas de minúsculas) | Aumenta ligeramente en función de la cardinalidad de las propiedades indexadas | Aumenta en función de la cantidad de elementos en los resultados de la consulta |
| Examen completo del índice    | Lectura de un conjunto distinto de valores indexados y carga solo de los elementos coincidentes desde el almacén de datos transaccionales                                              | Contains, EndsWith, RegexMatch, LIKE                                    | Aumenta de manera lineal en función de la cardinalidad de las propiedades indexadas | Aumenta en función de la cantidad de elementos en los resultados de la consulta |
| Examen completo          | Carga de todos los elementos                                               | Upper, Lower                                    | N/D                                                          | Aumenta en función de la cantidad de elementos del contenedor |

Al escribir consultas, debe usar el predicado de filtro que use el índice de la manera más eficaz posible. Por ejemplo, si en su caso de uso funcionaría `StartsWith` o `Contains`, debería optar por `StartsWith`, porque realizará un examen preciso del índice en lugar de un examen completo del índice.

## <a name="index-usage-details"></a>Detalles de uso del índice

En esta sección, se tratarán más detalles sobre cómo las consultas usan los índices. Esto no es necesario para aprender a empezar a trabajar con Azure Cosmos DB, pero se documenta en detalle para satisfacer la curiosidad de los usuarios. Haremos referencia al elemento de ejemplo que ya se compartió en este documento:

Elementos de ejemplo:

```json
    {
        "id": 1,
        "locations": [
            { "country": "Germany", "city": "Berlin" },
            { "country": "France", "city": "Paris" }
        ],
        "headquarters": { "country": "Belgium", "employees": 250 },
        "exports": [
            { "city": "Moscow" },
            { "city": "Athens" }
        ]
    }
```

```json
    {
        "id": 2,
        "locations": [
            { "country": "Ireland", "city": "Dublin" }
        ],
        "headquarters": { "country": "Belgium", "employees": 200 },
        "exports": [
            { "city": "Moscow" },
            { "city": "Athens" },
            { "city": "London" }
        ]
    }
```

Azure Cosmos DB usa un índice invertido. El índice funciona asignando cada ruta de acceso JSON al conjunto de elementos que contienen ese valor. La asignación del id. de elemento se representa en muchas páginas de índice diferentes para el contenedor. Este es un diagrama de ejemplo de un índice invertido para un contenedor que incluye los dos elementos de ejemplo:

| Ruta de acceso                    | Value   | Lista de los id. de elemento   |
| ----------------------- | ------- | ---------- |
| /locations/0/country    | Alemania | 1          |
| /locations/0/country    | Irlanda | 2          |
| /locations/0/city       | Berlín  | 1          |
| /locations/0/city       | Dublín  | 1          |
| /locations/1/country    | Francia  | 1          |
| /locations/1/city       | Paris   | 1          |
| /headquarters/country   | Bélgica | 2          |
| /headquarters/employees | 200     | 2          |
| /headquarters/employees | 250     | 1          |

El índice invertido tiene dos atributos importantes:
- Para una ruta de acceso determinada, los valores se ordenan de manera ascendente. Por lo tanto, el motor de consultas puede procesar `ORDER BY` fácilmente desde el índice.
- Para una ruta de acceso determinada, el motor de consultas puede examinar el conjunto distinto de valores posibles para identificar las páginas de índice donde hay resultados.

El motor de consultas puede utilizar el índice invertido de cuatro maneras distintas:

### <a name="index-seek"></a>Búsqueda de índice

Considere la consulta siguiente: 

```sql
SELECT location
FROM location IN company.locations
WHERE location.country = 'France'`
```

El predicado de consulta (filtrado por elementos, donde cualquier ubicación tiene "France" como su país o región) coincidiría con la ruta de acceso resaltada en rojo a continuación:

:::image type="content" source="./media/index-overview/matching-path.png" alt-text="Coincidencia con una ruta de acceso específica dentro de un árbol" border="false":::

Puesto que esta consulta tiene un filtro de igualdad, después de recorrer este árbol, podemos identificar rápidamente las páginas de índice que contienen los resultados de la consulta. En este caso, el motor de consultas leería las páginas de índice que contienen el elemento 1. Una búsqueda de índice es la manera más eficaz de usar el índice. Con una búsqueda de índice solo se leen las páginas de índice necesarias y solo se cargan los elementos de los resultados de la consulta. Por lo tanto, el tiempo de búsqueda de índice y el cargo por RU de la búsqueda de índice son increíblemente bajos, independientemente del volumen total de los datos. 

### <a name="precise-index-scan"></a>Examen preciso del índice

Considere la consulta siguiente: 

```sql
SELECT *
FROM company
WHERE company.headquarters.employees > 200
```

El predicado de consulta (filtrado por elementos en los que hay más de 200 empleados) se puede evaluar con un examen preciso del índice de la ruta de acceso `headquarters/employees`. Al realizar un examen preciso del índice, el motor de consultas comienza al realizar una búsqueda binaria del conjunto distinto de valores posibles para encontrar la ubicación del valor `200` de la ruta de acceso `headquarters/employees`. Como los valores de cada ruta de acceso se ordenan de manera ascendente, resulta fácil para el motor de consultas realizar una búsqueda binaria. Una vez que el motor de consultas encuentra el valor `200`, comienza a leer todas las páginas de índice restantes (en dirección ascendente).

Debido a que el motor de consultas puede realizar una búsqueda binaria para evitar examinar páginas de índice innecesarias, los exámenes precisos del índice tienden a tener una latencia comparable y cargos por RU para las operaciones de búsqueda de índice.

### <a name="expanded-index-scan"></a>Examen expandido del índice

Considere la consulta siguiente: 

```sql
SELECT *
FROM company
WHERE StartsWith(company.headquarters.country, "United", true)
```

El predicado de consulta (filtrado por elementos que tienen oficinas centrales en un país que comienza con "United" y que distingue mayúsculas de minúsculas) se puede evaluar con un examen expandido del índice de la ruta de acceso `headquarters/country`. Las operaciones que realizan un examen expandido del índice cuentan con optimizaciones que pueden ayudar a evitar la necesidad de examinar cada página de índice, pero son ligeramente más costosas que la búsqueda binaria de un examen preciso del índice.

Por ejemplo, al evaluar `StartsWith` que no distingue mayúsculas de minúsculas, el motor de consultas comprobará las distintas combinaciones posibles de valores en mayúsculas y minúsculas del índice. Esta optimización permite al motor de consultas evitar leer la mayoría de las páginas de índice. Las distintas funciones del sistema tienen optimizaciones diferentes que pueden usar para evitar leer cada página de índice, por lo que las clasificaremos ampliamente como un examen expandido del índice. 

### <a name="full-index-scan"></a>Examen completo del índice

Considere la consulta siguiente: 

```sql
SELECT *
FROM company
WHERE Contains(company.headquarters.country, "United")
```

El predicado de consulta (filtrado por elementos que tienen oficinas centrales en un país que incluye "United") se puede evaluar con un examen del índice de la ruta de acceso `headquarters/country`. A diferencia de un examen preciso del índice, un examen completo del índice siempre examinará el conjunto distinto de valores posibles para identificar las páginas de índice donde hay resultados. En este caso, `Contains` se ejecuta en el índice. El tiempo de búsqueda de índice y el cargo por RU para los exámenes del índice aumentan a medida que aumenta la cardinalidad de la ruta de acceso. En otras palabras, cuanto más valores distintos necesite examinar el motor de consultas, mayores serán la latencia y el cargo por RU implicados en realizar un examen completo del índice.  

Por ejemplo, considere dos propiedades: ciudad y país. La cardinalidad de ciudad es 5000 y la de país es 200. A continuación, se muestras dos consultas de ejemplo que tienen una función del sistema [Contains](sql-query-contains.md) que realiza un examen del índice en la propiedad `town`. La primera consulta usará más RU que la segunda, porque la cardinalidad de ciudad es mayor que la de país.

```sql
    SELECT *
    FROM c
    WHERE CONTAINS(c.town, "Red", false)
```

```sql
    SELECT *
    FROM c
    WHERE CONTAINS(c.country, "States", false)
```

### <a name="full-scan"></a>Examen completo

En algunos casos, es posible que el motor de consultas no pueda evaluar un filtro de consulta mediante el índice. En este caso, el motor de consultas deberá cargar todos los elementos del almacén transaccional para evaluar el filtro de consulta. Los exámenes completos no usan el índice y tienen un cargo por RU que aumenta linealmente con el tamaño total de los datos. Afortunadamente, las operaciones que requieren exámenes completos son poco frecuentes. 

### <a name="queries-with-complex-filter-expressions"></a>Consultas con expresiones de filtro complejas

En los ejemplos anteriores, solo se tenían en cuenta las consultas que tenían expresiones de filtro simples (por ejemplo, consultas con un solo filtro de igualdad o rango). En realidad, la mayoría de las consultas tienen expresiones de filtro mucho más complejas.

Considere la consulta siguiente:

```sql
SELECT *
FROM company
WHERE company.headquarters.employees = 200 AND CONTAINS(company.headquarters.country, "United")
```

Para ejecutar esta consulta, el motor de consultas debe realizar una búsqueda de índice precisa en `headquarters/employees` y un examen completo del índice en `headquarters/country`. El motor de consultas cuenta con heurística interna que usa para evaluar la expresión de filtro de consulta de la manera más eficaz posible. En este caso, el motor de consultas evitaría tener que leer páginas de índice innecesarias al realizar primero la búsqueda de índice. Por ejemplo, si solo 50 elementos coincidían con el filtro de igualdad, el motor de consultas solo necesitaría evaluar `Contains` en las páginas de índice que contenían esos 50 elementos. No sería necesario realizar un examen completo del índice de todo el contenedor.

## <a name="index-utilization-for-scalar-aggregate-functions"></a>Uso del índice para funciones de agregado escalares

Las consultas con funciones de agregado deben basarse exclusivamente en el índice para poder usarlo. 

En algunos casos, el índice puede devolver falsos positivos. Por ejemplo, al evaluar `Contains` en el índice,el número de coincidencias en el índice puede superar el número de resultados de la consulta. El motor de consultas cargará todas las coincidencias de índice, evaluará el filtro en los elementos cargados y devolverá solo los resultados correctos.

Para la mayoría de las consultas, la carga de coincidencias de índice con falsos positivos no tendrá ningún impacto perceptible en el uso del índice.

Por ejemplo, considere la siguiente consulta:

```sql
SELECT *
FROM company
WHERE Contains(company.headquarters.country, "United")
```

La función del sistema `Contains` puede devolver algunas coincidencias de falsos positivos, por lo que el motor de consultas deberá comprobar si cada elemento cargado coincide con la expresión de filtro. En este ejemplo, es posible que el motor de consultas solo necesite cargar algunos elementos adicionales, por lo que el impacto en el uso del índice y el cargo por RU es mínimo.

Sin embargo, las consultas con funciones de agregado deben basarse exclusivamente en el índice para poder usarlo. Por ejemplo, considere la consulta siguiente con un agregado `Count`:

```sql
SELECT COUNT(1)
FROM company
WHERE Contains(company.headquarters.country, "United")
```

Al igual que en el primer ejemplo, la función del sistema `Contains` puede devolver algunas coincidencias de falsos positivos. Sin embargo, a diferencia de la consulta `SELECT *`, la consulta `Count` no puede evaluar la expresión de filtro en los elementos cargados para comprobar todas las coincidencias del índice. La consulta `Count` debe basarse exclusivamente en el índice, por lo que si hay alguna posibilidad de que una expresión de filtro devuelva coincidencias de falsos positivos, el motor de consultas recurrirá a un examen completo.

Las consultas con las funciones de agregado siguientes deben basarse exclusivamente en el índice, por lo que la evaluación de algunas funciones del sistema requiere un examen completo.

- [Avg](sql-query-aggregate-avg.md)
- [Recuento](sql-query-aggregate-count.md)
- [Máx.](sql-query-aggregate-max.md)
- [Mín.](sql-query-aggregate-min.md)
- [Sum](sql-query-aggregate-sum.md)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información acerca de la indexación en los siguientes artículos:

- [Directiva de indexación](index-policy.md)
- [Cómo administrar la directiva de indexación](how-to-manage-indexing-policy.md)
