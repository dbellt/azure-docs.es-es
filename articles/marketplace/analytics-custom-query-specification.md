---
title: Especificación de consulta personalizada
description: Obtenga información sobre cómo usar consultas personalizadas para extraer datos de tablas de análisis mediante programación para sus ofertas en el marketplace comercial de Microsoft.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: sayantanroy83
ms.author: sroy
ms.date: 3/08/2021
ms.openlocfilehash: b82fb574b134065f3c0f1a7dc5a4742258914ff6
ms.sourcegitcommit: 942a1c6df387438acbeb6d8ca50a831847ecc6dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112017422"
---
# <a name="custom-query-specification"></a>Especificación de consulta personalizada

Los asociados pueden usar esta especificación de consulta para formular fácilmente consultas personalizadas para extraer datos de las tablas de análisis. Las consultas se pueden usar para seleccionar solo las columnas deseadas y las métricas que coincidan con un determinado criterio. En el centro de la especificación del lenguaje se encuentra la definición del conjunto de datos sobre el que se puede escribir una consulta personalizada.

## <a name="datasets"></a>Conjuntos de datos

Del mismo modo que algunas consultas se ejecutan en una base de datos que tiene tablas y columnas, una consulta personalizada funciona en conjuntos de datos que tienen columnas y métricas. La lista completa de los conjuntos de datos disponibles para formular una consulta puede obtenerse mediante la API de conjuntos de datos.

Este es un ejemplo de un conjunto de datos que se muestra como JSON.

```json
{
     "datasetName": "ISVUsage",
     "selectableColumns": [
       "MarketplaceSubscriptionId",
       "OfferName",
            "CustomerId",
     "MonthStartDate",
     "SKU"
    ],
    "availableMetrics": [
     "NormalizedUsage", 
     "RawUsage"
        "EstimatedExtendedChargeCC"
    ],
    "availableDateRanges": [
     "LAST_MONTH",
     "LAST_3_MONTHS",
     "LAST_6_MONTHS",
     "LIFETIME"
    ]
}
```

### <a name="parts-of-a-dataset"></a>Elementos de un conjunto de datos

- Un nombre de conjunto de datos es como el nombre de una tabla de base de datos. Por ejemplo, ISVUsage. Un conjunto de datos tiene una lista de columnas que se pueden seleccionar, como MarketplaceSubscriptionId.
- Un conjunto de datos también tiene métricas, que son como funciones de agregación en una base de datos. Por ejemplo, NormalizedUsage.
- Hay intervalos de tiempo fijos durante los que se pueden exportar los datos.

### <a name="formulating-a-query-on-a-dataset"></a>Formulación de una consulta en un conjunto de datos

Estas son algunas consultas de ejemplo que muestran cómo extraer varios tipos de datos.

| Consultar | Descripción |
| ------------ | ------------- |
| **SELECT** MarketplaceSubscriptionId,CustomerId **FROM** ISVUsage **TIMESPAN LAST_MONTH** | Esta consulta obtiene cada `MarketplaceSubscriptionId` y su correspondiente `CustomerId` en el último mes. |
| **SELECT** MarketplaceSubscriptionId, EstimatedExtendedChargeCC **FROM** ISVUsage **ORDER BY** EstimatedExtendedChargeCC **LIMIT** 10 | Esta consulta obtendrá las 10 principales suscripciones en orden decreciente de las licencias vendidas de cada suscripción. |
| **SELECT** CustomerId, NormalizedUsage, RawUsage **FROM** ISVUsage **WHERE** NormalizedUsage > 100000 **ORDER BY** NormalizedUsage **TIMESPAN** LAST_6_MONTHS | Esta consulta obtendrá los elementos NormalizedUsage y RawUsage de todos los clientes cuyo elemento NormalizedUsage es mayor que 100 000. |
| **SELECT** MarketplaceSubscriptionId, MonthStartDate, NormalizedUsage **FROM** ISVUsage **WHERE** CustomerId IN (‘2a31c234-1f4e-4c60-909e-76d234f93161’, ‘80780748-3f9a-11eb-b378-0242ac130002’) | Esta consulta obtiene `MarketplaceSubscriptionId` y el uso normalizado de cada mes por los dos valores `CustomerId`: `2a31c234-1f4e-4c60-909e-76d234f93161` y `80780748-3f9a-11eb-b378-0242ac130002`. |
|||

## <a name="query-specification"></a>Especificación de consulta

En esta sección se describe la definición y la estructura de la consulta.

### <a name="grammar-reference"></a>Referencia de gramática

En esta tabla se describen los símbolos que se usan en las consultas.

| Símbolo | Significado |
| ------------ | ------------- |
| ? | Opcionales |
| * | Cero o más |
| + | Uno o más |
| &#124; | O/Uno de la lista |
| | |

### <a name="query-definition"></a>Definición de la consulta

La instrucción de consulta tiene las cláusulas siguientes: SelectClause, FromClause, WhereClause?, OrderClause?, LimitClause? y TimeSpan?.

- **SelectClause**: **SELECT** ColumOrMetricName (, ColumOrMetricName)*
    - **ColumOrMetricName**: columnas y métricas definidas en el conjunto de datos
- **FromClause**: **FROM** DatasetName
    - **DatasetName**: nombre del conjunto de datos definido en el conjunto de datos
- **WhereClause**: **WHERE** FilterCondition (**AND** FilterCondition)*
    - **FilterCondition**: valor del operador ColumOrMetricName
        - **Operador**:  = | > | < | >= | <= | != | LIKE | NOT LIKE | IN | NOT IN
        - **Valor**: Number | StringLiteral | MultiNumberList | MultiStringList 
            - **Number**: -? [0-9]+ (. [0-9] [0-9]*)?
            - **StringLiteral**: "[a-zA-Z0-9_]*"
            - **MultiNumberList**: (Number  (,Number)*)
            - **MultiStringList**: (StringLiteral (,StringLiteral)*)
- **OrderClause**: **ORDER BY** OrderCondition (,OrderCondition)*
    - **OrderCondition**: ColumOrMetricName (**ASC** | **DESC**)*
    - **LimitClause**: **LIMIT** [0-9]+
    - **TimeSpan**: **TIMESPAN** ( TODAY | YESTERDAY | LAST_7_DAYS | LAST_14_DAYS | LAST_30_DAYS | LAST_90_DAYS | LAST_180_DAYS | LAST_365_DAYS | LAST_MONTH | LAST_3_MONTHS | LAST_6_MONTHS | LAST_1_YEAR | LIFETIME)

### <a name="query-structure"></a>Estructura de la consulta

Una consulta de informe se compone de varios elementos:

- SELECT
- FROM
- WHERE
- ORDER BY
- LIMIT
- TIMESPAN

A continuación, se describe cada uno de los elementos.

#### <a name="select"></a>SELECT

Este elemento de la consulta especifica las columnas que se exportarán. Las columnas que se pueden seleccionar son los campos enumerados en las secciones `selectableColumns` y `availableMetrics` de un conjunto de datos. Si hay una columna de métricas incluida en la lista de campos seleccionados, las métricas se calcularán para cada combinación distinta de las columnas que no son métricas. 

**Ejemplo**:
- **SELECT** `OfferName`, `NormalizedUsage`

#### <a name="distinct"></a>DISTINCT

Agregar la palabra clave DISTINCT después de SELECT garantiza que los datos exportados finales no tendrán filas duplicadas. La palabra clave DISTINCT funciona independientemente de si se selecciona o no una columna de métrica.

**Ejemplo**:
- **SELECT DISTINCT** `MarketplaceSubscriptionId, OfferType`

#### <a name="from"></a>FROM

Este elemento de la consulta indica el conjunto de datos desde el que se deben exportar los datos. El nombre del conjunto de datos que se proporciona aquí debe ser un nombre de conjunto de datos válido devuelto por la API de los conjuntos de datos.

**Ejemplo**:
- FROM `ISVUsage`
- FROM `ISVOrder`

#### <a name="where"></a>WHERE

Este elemento de la consulta se usa para especificar las condiciones de filtro en el conjunto de datos. Solo las filas que coincidan con todas las condiciones enumeradas en esta cláusula estarán presentes en el archivo final exportado. La condición de filtro puede estar en cualquiera de las columnas enumeradas en `selectableColumns` y `availableMetrics`. Los valores especificados en la condición de filtro pueden ser una lista de números o una lista de cadenas solo cuando el operador es `IN` o `NOT IN`. Siempre se pueden proporcionar los valores como una cadena literal y se convertirán en los tipos nativos de columnas. Se deben separar varias condiciones de filtro con una operación `AND`.

**Ejemplo**:

- MarketplaceSubscriptionId = "868368da-957d-4959-8992-3c12dc7e6260"
- CustomerName **LIKE** "%Contosso%"
- CustomerId **NOT IN** (1000, 1001, 1002)
- OrderQuantity=100
- OrderQuantity="100"
    - MarketplaceSubscriptionId="7b487ac0-ce12-b732-dcd6-91a1e4e74a50" AND CustomerId= "0f8b7fa0-eb83-a183-1225-ca153ef807aa"

#### <a name="order-by"></a>ORDER BY

En este elemento de la consulta se especifican los criterios de ordenación de las filas exportadas. Las columnas en las que se puede definir la ordenación deben ser de `selectableColumns` y `availableMetrics` del conjunto de datos. Si no se especifica ninguna dirección de ordenación, se establecerá `DESC` como valor predeterminado en la columna. La ordenación se puede definir en varias columnas separando los criterios con una coma.

**Ejemplo**:

- **ORDER BY** NormalizedUsage **ASC**, EstimatedExtendedCharge(CC) **DESC**
- **ORDER BY** CustomerName **ASC**, NormalizedUsage

#### <a name="limit"></a>LIMIT

Este elemento de la consulta especifica el número de filas que se exportarán. El número que especifique debe ser un entero positivo distinto de cero.

#### <a name="timespan"></a>TIMESPAN

Este elemento de la consulta especifica el período de tiempo durante el que se deben exportar los datos. Los valores posibles deben ser del campo `availableDateRanges` de la definición del conjunto de datos.

### <a name="case-sensitivity-in-query-specification"></a>Distinción de mayúsculas y minúsculas en la especificación de consultas

La especificación no distingue entre mayúsculas y minúsculas. Las palabras clave predefinidas, los nombres de columna y los valores se pueden especificar usando mayúsculas o minúsculas.

## <a name="see-also"></a>Consulte también

- [Lista de consultas del sistema](analytics-system-queries.md)
