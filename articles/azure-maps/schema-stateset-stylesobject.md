---
title: Guía de referencia del esquema de StylesObject para Azure Maps dinámico
description: Guía de referencia de la sintaxis y el esquema de StylesObject para Azure Maps dinámico
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: reference
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: f213976d0e61706b2d8a9c8465be1ff0a4065b2e
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112031896"
---
# <a name="stylesobject-schema-reference-guide-for-dynamic-maps"></a>Guía de referencia del esquema de StylesObject para Azure Maps dinámico

 `StylesObject` es una matriz de objetos `StyleObject` que representa los estilos del conjunto de estados. Use la [característica Servicio de estado](/rest/api/maps/v2/feature-state) de Azure Maps Creator para aplicar los estilos del conjunto de estados a las características de los datos de mapas de interiores. Una vez que haya creado los estilos del conjunto de estados y los haya asociado con las características del mapa de interiores, puede usarlos para crear mapas de interiores dinámicos. Para más información sobre la creación de mapas de interiores dinámicos, consulte [Implementación de estilos dinámicos para mapas de interiores de Creator](indoor-map-dynamic-styling.md).

## <a name="styleobject"></a>StyleObject

`StyleObject` es una de las siguientes reglas de estilo:

 * [`BooleanTypeStyleRule`](#booleantypestylerule)
 * [`NumericTypeStyleRule`](#numerictypestylerule)
 * [`StringTypeStyleRule`](#stringtypestylerule)

El código JSON siguiente muestra el uso de ejemplo de cada uno de los tres tipos de estilo.  `BooleanTypeStyleRule` se utiliza para determinar el estilo dinámico de las características cuya propiedad `occupied` es true y false.  `NumericTypeStyleRule` se utiliza para determinar el estilo de las características cuya propiedad `temperature` se encuentra dentro de un intervalo determinado. Por último, `StringTypeStyleRule` se usa para hacer coincidir estilos específicos con `meetingType`.



```json
 "styles": [
     {
        "keyname": "occupied",
        "type": "boolean",
        "rules": [
            {
                "true": "#FF0000",
                "false": "#00FF00"
            }
        ]
    },
    {
        "keyname": "temperature",
        "type": "number",
        "rules": [
             {
                "range": {
                "minimum": 50,
                "exclusiveMaximum": 70
                },
                "color": "#343deb"
            },
            {
                "range": {
                "maximum": 70,
                "exclusiveMinimum": 30
              },
              "color": "#eba834"
            }
        ]
    },
    {
      "keyname": "meetingType",
      "type": "string",
      "rules": [
        {
          "private": "#FF0000",
          "confidential": "#FF00AA",
          "allHands": "#00FF00",
          "brownBag": "#964B00"
        }
      ]
    }
]
```

## <a name="numerictypestylerule"></a>NumericTypeStyleRule

 Un elemento `NumericTypeStyleRule` es un objeto [`StyleObject`](#styleobject) y consta de las siguientes propiedades:

| Propiedad | Tipo | Descripción | Obligatorio |
|-----------|----------|-------------|-------------|
| `keyName` | string | *Estado* o nombre de la propiedad dinámica. `keyName` debe ser único dentro de una matriz de elementos `StyleObject`.| Sí |
| `type` | string | El valor es "numérico". | Sí |
| `rules` | [`NumberRuleObject`](#numberruleobject)[]| Matriz de intervalos de estilos numéricos con colores asociados. Cada intervalo define un color que se va a utilizar cuando el valor de *estado* está en el intervalo.| Sí |

### <a name="numberruleobject"></a>NumberRuleObject

Un elemento `NumberRuleObject` consta de un objeto [`RangeObject`](#rangeobject) y una propiedad `color`. Si el valor de *estado* entra en el intervalo, su color para la presentación será el color especificado en la propiedad `color`.

Si define varios intervalos superpuestos, el color elegido será el color que se define en el primer intervalo que se cumple.

En el siguiente código JSON de ejemplo, ambos intervalos contendrán True cuando el valor de *estado* está entre 50 y 60. Sin embargo, el color que se utilizará es `#343deb`, porque es el primer intervalo de la lista que se cumple.

```json

    {
        "rules":[
            {
                "range": {
                "minimum": 50,
                "exclusiveMaximum": 70
                },
                "color": "#343deb"
            },
            {
                "range": {
                "minimum": 50,
                "maximum": 60
                },
                "color": "#eba834"
            }
        ]
    }
]
```

| Propiedad | Tipo | Descripción | Obligatorio |
|-----------|----------|-------------|-------------|
| `range` | [RangeObject](#rangeobject) | El elemento [RangeObject](#rangeobject) define un conjunto de condiciones de intervalo lógico, que, si es `true`, cambia el color de presentación de *estado* al color especificado en la propiedad `color`. Si no se especifica `range`, se usará siempre el color definido en la propiedad `color`.   | No |
| `color` | string | Color que se va a usar cuando el valor de estado está en el intervalo. La propiedad `color` es una cadena JSON en cualquiera de los formatos siguientes: <ul><li> Valores hexadecimales de estilo HTML </li><li> RGB ("#ff0", "#ffff00", "rgb(255, 255, 0)")</li><li> RGBA ("rgba(255, 255, 0, 1)")</li><li> HSL("hsl(100, 50%, 50%)")</li><li> HSLA("hsla(100, 50%, 50%, 1)")</li><li> Nombres de colores HTML predefinidos, como yellow (amarillo) y blue (azul).</li></ul> | Sí |

### <a name="rangeobject"></a>RangeObject

`RangeObject` define un valor de intervalo numérico de un objeto [`NumberRuleObject`](#numberruleobject). Para que el valor de *estado* entre en el intervalo, todas las condiciones definidas deben ser verdaderas.

| Propiedad | Tipo | Descripción | Obligatorio |
|-----------|----------|-------------|-------------|
| `minimum` | double | Todo número x en el que x ≥ `minimum`.| No |
| `maximum` | double | Todo número x en el que x ≤ `maximum`. | No |
| `exclusiveMinimum` | double | Todo número x en el que x > `exclusiveMinimum`.| No |
| `exclusiveMaximum` | double | Todo número x en el que x < `exclusiveMaximum`.| No |

### <a name="example-of-numerictypestylerule"></a>Ejemplo de NumericTypeStyleRule

El siguiente código JSON muestra un *estado* de tipo `NumericTypeStyleRule` llamado `temperature`. En este ejemplo, [`NumberRuleObject`](#numberruleobject) contiene dos intervalos de temperatura definidos y sus estilos de color asociados. Si el intervalo de temperatura está entre 50 y 69, la representación debe utilizar el color `#343deb`.  Si el intervalo de temperatura está entre 31 y 70, la representación debe utilizar el color `#eba834`.

```json
{
    "keyname": "temperature",
    "type": "number",
    "rules":[
        {
            "range": {
            "minimum": 50,
            "exclusiveMaximum": 70
            },
            "color": "#343deb"
        },
        {
            "range": {
            "maximum": 70,
            "exclusiveMinimum": 30
            },
            "color": "#eba834"
        }
    ]
}
```

## <a name="stringtypestylerule"></a>StringTypeStyleRule

Un elemento `StringTypeStyleRule` es un objeto [`StyleObject`](#styleobject) y consta de las siguientes propiedades:

| Propiedad | Tipo | Descripción | Obligatorio |
|-----------|----------|-------------|-------------|
| `keyName` | string |  *Estado* o nombre de la propiedad dinámica.  `keyName` debe ser único dentro de una matriz de `StyleObject`.| Sí |
| `type` | string |El valor es "string". | Sí |
| `rules` | [`StringRuleObject`](#stringruleobject)[]| Matriz de N número de valores de *estado*.| Sí |

### <a name="stringruleobject"></a>StringRuleObject

`StringRuleObject` consta de hasta N números de valores de estado que son los posibles valores de cadena de la propiedad de una característica. Si el valor de propiedad de la característica no coincide con ninguno de los valores de estado definidos, esa característica no tendrá un estilo dinámico. Si se proporcionan valores de estado duplicados, el primero tiene prioridad.

La coincidencia de valores de cadena distingue entre mayúsculas y minúsculas.

| Propiedad | Tipo | Descripción | Obligatorio |
|-----------|----------|-------------|-------------|
| `stateValue1` | string | El color cuando la cadena de valor es stateValue1. | No |
| `stateValue2` | string | El color cuando la cadena de valor es stateValue. | No |
| `stateValueN` | string | El color cuando la cadena de valor es stateValueN. | No |

### <a name="example-of-stringtypestylerule"></a>Ejemplo de StringTypeStyleRule

El siguiente JSON muestra una regla `StringTypeStyleRule` que define los estilos asociados a tipos de reuniones concretos.

```json
    {
      "keyname": "meetingType",
      "type": "string",
      "rules": [
        {
          "private": "#FF0000",
          "confidential": "#FF00AA",
          "allHands": "#00FF00",
          "brownBag": "#964B00"
        }
      ]
    }

```

## <a name="booleantypestylerule"></a>BooleanTypeStyleRule

Un elemento `BooleanTypeStyleRule` es un objeto [`StyleObject`](#styleobject) y consta de las siguientes propiedades:

| Propiedad | Tipo | Descripción | Obligatorio |
|-----------|----------|-------------|-------------|
| `keyName` | string |  *Estado* o nombre de la propiedad dinámica.  `keyName` debe ser único dentro de una matriz de elementos `StyleObject`.| Sí |
| `type` | string |El valor es "booleano". | Sí |
| `rules` | [`BooleanRuleObject`](#booleanruleobject)[1]| Un par booleano con colores para los valores de *estado* `true` y `false`.| Sí |

### <a name="booleanruleobject"></a>BooleanRuleObject

Un elemento `BooleanRuleObject` define los colores para los valores `true` y `false`.

| Propiedad | Tipo | Descripción | Obligatorio |
|-----------|----------|-------------|-------------|
| `true` | string | Color que se va a usar cuando el valor de *estado* sea `true`. La propiedad `color` es una cadena JSON en cualquiera de los formatos siguientes: <ul><li> Valores hexadecimales de estilo HTML </li><li> RGB ("#ff0", "#ffff00", "rgb(255, 255, 0)")</li><li> RGBA ("rgba(255, 255, 0, 1)")</li><li> HSL("hsl(100, 50%, 50%)")</li><li> HSLA("hsla(100, 50%, 50%, 1)")</li><li> Nombres de colores HTML predefinidos, como yellow (amarillo) y blue (azul).</li></ul>| Sí |
| `false` | string | Color que se va a usar cuando el valor de *estado* sea `false`. | Sí |

### <a name="example-of-booleantypestylerule"></a>Ejemplo de BooleanTypeStyleRule

El siguiente código JSON muestra un *estado* de tipo `BooleanTypeStyleRule` llamado `occupied`. Un elemento [`BooleanRuleObject`](#booleanruleobject) define los colores para los valores `true` y `false`.

```json
{
    "keyname": "occupied",
    "type": "boolean",
    "rules": [
    {
        "true": "#FF0000",
        "false": "#00FF00"
    }
    ]
}
```