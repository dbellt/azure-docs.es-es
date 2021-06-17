---
title: 'Filtrado avanzado: Azure Event Grid en IoT Edge | Microsoft Docs'
description: Filtrado avanzado en Event Grid en IoT Edge.
manager: rajarv
ms.reviewer: spelluru
ms.subservice: iot-edge
ms.date: 05/10/2021
ms.topic: article
ms.openlocfilehash: d0f146f66e2779a9ee8187f3ac06c72d884e9d8a
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111755634"
---
# <a name="advanced-filtering"></a>Filtrado avanzado
Event Grid permite especificar filtros por cualquier propiedad de la carga JSON. Estos filtros se modelan como un conjunto de condiciones `AND`, donde cada condición externa tiene condiciones `OR` internas opcionales. Para cada condición `AND`, se especifican los siguientes valores:

* `OperatorType`: el tipo de comparación.
* `Key`: la ruta de acceso JSON a la propiedad en la que se va a aplicar el filtro.
* `Value`: el valor de referencia con el que se ejecuta el filtro (o) `Values`: el conjunto de valores de referencia con el que se ejecuta el filtro.

## <a name="json-syntax"></a>Sintaxis JSON

La sintaxis JSON de un filtro avanzado es la siguiente:

```json
{
    "filter": {
        "advancedFilters": [{
                "operatorType": "NumberGreaterThanOrEquals",
                "key": "Data.Key1",
                "value": 5
            }, {
                "operatorType": "StringContains",
                "key": "Subject",
                "values": ["container1", "container2"]
            }
        ]
    }
}
```

## <a name="filtering-on-array-values"></a>Filtrado por los valores de matriz

Actualmente, Event Grid no admite el filtrado por una matriz de valores. Si un evento entrante tiene un valor de matriz para la clave del filtro avanzado, se produce un error en la operación de búsqueda de coincidencia. El evento entrante termina por no coincidir con la suscripción del evento.

## <a name="and-or-not-semantics"></a>Semántica AND-OR-NOT

Observe que en el ejemplo JSON proporcionado anteriormente, `AdvancedFilters` es una matriz. Considere cada elemento de la matriz `AdvancedFilter` como una condición `AND`.

En el caso de los operadores que admiten varios valores (como `NumberIn`, `NumberNotIn`, `StringIn`, etc.), cada valor se trata como una condición `OR`. Por lo tanto, un objeto `StringBeginsWith("a", "b", "c")` coincidirá con cualquier valor de cadena que empiece por `a`, `b` o `c`.

> [!CAUTION]
> Los operadores NOT `NumberNotIn` y `StringNotIn` se comportan como condiciones AND en cada valor proporcionado en el campo `Values`.
>
> Si no lo hacen, el filtro se convierte en un filtro Accept-All y se anula el propósito del filtrado.

## <a name="floating-point-rounding-behavior"></a>Comportamiento de redondeo de punto flotante

Event Grid usa el tipo .NET `decimal` para administrar todos los valores numéricos. Los valores numéricos especificados en el archivo JSON de suscripción de eventos no están sujetos al comportamiento de redondeo de punto flotante.

## <a name="case-sensitivity-of-string-filters"></a>Distinción de mayúsculas y minúsculas en los filtros de cadena

Ninguna comparación de cadena distingue mayúsculas de minúsculas. Actualmente no hay ninguna manera de cambiar este comportamiento.

## <a name="allowed-advanced-filter-keys"></a>Claves de filtro avanzadas permitidas

La propiedad `Key` puede ser una propiedad de nivel superior conocida o una ruta de acceso JSON con varios puntos, donde cada punto indica la ejecución paso a paso de un objeto JSON anidado.

Event Grid no tiene ningún significado especial para el carácter `$` de la clave, a diferencia de la especificación JSONPath.

### <a name="event-grid-schema"></a>Esquema de Event Grid

Para eventos del esquema de Event Grid:

* ID
* Tema
* Asunto
* EventType
* DataVersion
* Data.Prop1
* Data.Prop*Prop2.Prop3.Prop4.Prop5

### <a name="custom-event-schema"></a>Esquema de evento personalizado

No hay ninguna restricción en la propiedad `Key` del esquema de evento personalizado, ya que Event Grid no impone ningún esquema de sobre en la carga.

## <a name="numeric-single-value-filter-examples"></a>Ejemplos de filtros de un solo valor numérico

* NumberGreaterThan
* NumberGreaterThanOrEquals
* NumberLessThan
* NumberLessThanOrEquals

```json
{
    "filter": {
        "advancedFilters": [
            {
                "operatorType": "NumberGreaterThan",
                "key": "Data.Key1",
                "value": 5
            },
            {
                "operatorType": "NumberGreaterThanOrEquals",
                "key": "Data.Key2",
                "value": *456
            },
            {
                "operatorType": "NumberLessThan",
                "key": "Data.P*P2.P3",
                "value": 1000
            },
            {
                "operatorType": "NumberLessThanOrEquals",
                "key": "Data.P*P2",
                "value": 999
            }
        ]
    }
}
```

## <a name="numeric-range-value-filter-examples"></a>Ejemplos de filtros de intervalo de valores numérico

* NumberIn
* NumberNotIn

```json
{
    "filter": {
        "advancedFilters": [
            {
                "operatorType": "NumberIn",
                "key": "Data.Key1",
                "values": [1, 10, 100]
            },
            {
                "operatorType": "NumberNotIn",
                "key": "Data.Key2",
                "values": [2, 3, 4.56]
            }
        ]
    }
}
```

## <a name="string-range-value-filter-examples"></a>Ejemplos de filtros de intervalo de valores de cadena

* StringContains
* StringBeginsWith
* StringEndsWith
* StringIn
* StringNotIn

```json
{
    "filter": {
        "advancedFilters": [
            {
                "operatorType": "StringContains",
                "key": "Data.Key1",
                "values": ["microsoft", "azure"]
            },
            {
                "operatorType": "StringBeginsWith",
                "key": "Data.Key2",
                "values": ["event", "grid"]
            },
            {
                "operatorType": "StringEndsWith",
                "key": "Data.P3.P4",
                "values": ["jpg", "jpeg", "png"]
            },
            {
                "operatorType": "StringIn",
                "key": "RootKey",
                "values": ["exact", "string", "matches"]
            },
            {
                "operatorType": "StringNotIn",
                "key": "RootKey",
                "values": ["aws", "bridge"]
            }
        ]
    }
}
```

## <a name="boolean-single-value-filter-examples"></a>Ejemplos de filtros de valor único booleano

* BoolEquals

```json
{
    "filter": {
        "advancedFilters": [
            {
                "operatorType": "BoolEquals",
                "key": "BoolKey1",
                "value": true
            },
            {
                "operatorType": "BoolEquals",
                "key": "BoolKey2",
                "value": false
            }
        ]
    }
}
```
