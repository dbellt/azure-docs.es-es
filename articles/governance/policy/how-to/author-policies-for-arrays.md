---
title: Creación de directivas para propiedades de matriz en recursos
description: Aprenda a trabajar con parámetros de matriz y expresiones de lenguaje de matriz, evaluar el alias [*] y anexar elementos con las reglas de definición de Azure Policy.
ms.date: 03/31/2021
ms.topic: how-to
ms.openlocfilehash: 8953b67d0935ac81cffa20ee8fc48d6d9b091861
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108165432"
---
# <a name="author-policies-for-array-properties-on-azure-resources"></a>Creación de directivas para propiedades de matriz en recursos de Azure

Las propiedades de Azure Resource Manager se suelen definir como cadenas y valores booleanos. En cambio, cuando existe una relación uno a varios, las propiedades complejas se definen como matrices. En Azure Policy, las matrices se usan de varias maneras diferentes:

- Tipo de un [parámetro de definición](../concepts/definition-structure.md#parameters), para proporcionar varias opciones
- Parte de una [regla de directiva](../concepts/definition-structure.md#policy-rule) con las condiciones **in** o **notIn**
- Parte de una regla de directiva que cuenta el número de miembros de la matriz que satisfacen una condición
- En los efectos [append](../concepts/effects.md#append) y [modify](../concepts/effects.md#modify) para actualizar una matriz existente

En este artículo se explica cada uso por parte de Azure Policy y proporciona varias definiciones de ejemplo.

## <a name="parameter-arrays"></a>Matrices de parámetros

### <a name="define-a-parameter-array"></a>Definir una matriz de parámetros

Definir un parámetro como una matriz, permite la flexibilidad de la directiva cuando se necesita más de un valor.
Esta definición de directiva permite cualquier ubicación única para el parámetro **allowedLocations** y el valor predeterminado es _eastus2_:

```json
"parameters": {
    "allowedLocations": {
        "type": "string",
        "metadata": {
            "description": "The list of allowed locations for resources.",
            "displayName": "Allowed locations",
            "strongType": "location"
        },
        "defaultValue": "eastus2"
    }
}
```

Como **type** es _string_, solo se puede establecer un único valor al asignar la directiva. Si se asigna esta directiva, los recursos del ámbito solo se permiten en una única región de Azure. La mayoría de las definiciones de directivas deben permitir una lista de opciones aprobadas, como permitir _eastus2_, _eastus_ y _westus2_.

Para crear la definición de directiva a fin de permitir varias opciones, utilice el elemento **type** _array_. La misma directiva se puede reescribir del siguiente modo:

```json
"parameters": {
    "allowedLocations": {
        "type": "array",
        "metadata": {
            "description": "The list of allowed locations for resources.",
            "displayName": "Allowed locations",
            "strongType": "location"
        },
        "defaultValue": "eastus2",
        "allowedValues": [
            "eastus2",
            "eastus",
            "westus2"
        ]

    }
}
```

> [!NOTE]
> Una vez que se guarda una definición de directiva, no se puede cambiar la propiedad **type** en un parámetro.

Esta nueva definición de parámetro toma más de un valor durante la asignación de directiva. Con la propiedad de matriz **allowedValues** definida, los valores disponibles durante la asignación se limitan más a la lista predefinida de opciones. El uso de **allowedValues** es opcional.

### <a name="pass-values-to-a-parameter-array-during-assignment"></a>Pasar valores a una matriz de parámetros durante la asignación

Al asignar la directiva a través de Azure Portal, un parámetro de **type** _array_ se muestra como un único cuadro de texto. La sugerencia indica "Use ; para separar los valores (por ejemplo, Londres;Nueva York)". Para pasar los valores de ubicación permitidos _eastus2_, _eastus_ y _westus2_ al parámetro, use la siguiente cadena:

`eastus2;eastus;westus2`

El formato para el valor del parámetro es distinto cuando se usa la CLI de Azure, Azure PowerShell o la API de REST. Los valores se pasan a través de una cadena JSON que también incluye el nombre del parámetro.

```json
{
    "allowedLocations": {
        "value": [
            "eastus2",
            "eastus",
            "westus2"
        ]
    }
}
```

Para usar esta cadena con cada SDK, use los siguientes comandos:

- CLI de Azure: Comando [az policy assignment create](/cli/azure/policy/assignment#az_policy_assignment_create) con el parámetro **params**
- Azure PowerShell: Cmdlet [New AzPolicyAssignment](/powershell/module/az.resources/New-Azpolicyassignment) con el parámetro **PolicyParameter**
- API REST: en la operación _PUT_ [create](/rest/api/policy/policyassignments/create) como parte del cuerpo de la solicitud, como el valor de la propiedad **properties.parameters**

## <a name="using-arrays-in-conditions"></a>Uso de matrices en condiciones

### <a name="in-and-notin"></a>In y notIn

Las condiciones `in` y `notIn` solo funcionan con valores de matriz. Comprueban la existencia de un valor en una matriz. La matriz puede ser una matriz JSON literal o una referencia a un parámetro de matriz. Por ejemplo:

```json
{
      "field": "tags.environment",
      "in": [ "dev", "test" ]
}
```

```json
{
      "field": "location",
      "notIn": "[parameters('allowedLocations')]"
}
```

### <a name="value-count"></a>Recuento de valores

La expresión [value count](../concepts/definition-structure.md#value-count) cuenta el número de miembros de la matriz que cumplen una condición. Proporciona una manera de evaluar la misma condición varias veces, con valores diferentes en cada iteración. Por ejemplo, la siguiente condición comprueba si el nombre del recurso coincide con algún patrón de una matriz de patrones:

```json
{
    "count": {
        "value": [ "test*", "dev*", "prod*" ],
        "name": "pattern",
        "where": {
            "field": "name",
            "like": "[current('pattern')]"
        }
    },
    "greater": 0
}
```

Para evaluar la expresión, Azure Policy evalúa la condición `where` tres veces, una vez por cada miembro de `[ "test*", "dev*", "prod*" ]`, de modo que cuenta el número de veces que se ha evaluado como `true`. En cada iteración, el valor del miembro de la matriz actual se empareja con el nombre de índice `pattern` definido por `count.name`. Luego se puede hacer referencia a este valor dentro de la condición `where` si se llama a una función de plantilla especial: `current('pattern')`.

| Iteración | Valor devuelto por `current('pattern')` |
|:---|:---|
| 1 | `"test*"` |
| 2 | `"dev*"` |
| 3 | `"prod*"` |

La condición es true solo si el recuento resultante es mayor que 0.

Para que la condición anterior sea más genérica, use la referencia de parámetro en lugar de una matriz literal:

```json
{
    "count": {
        "value": "[parameters('patterns')]",
        "name": "pattern",
        "where": {
            "field": "name",
            "like": "[current('pattern')]"
        }
    },
    "greater": 0
}
```

Si la expresión **value count** no está en ninguna otra expresión **count**, `count.name` es opcional y se puede usar la función `current()` sin ningún argumento:

```json
{
    "count": {
        "value": "[parameters('patterns')]",
        "where": {
            "field": "name",
            "like": "[current()]"
        }
    },
    "greater": 0
}
```

**Value count** también admite matrices de objetos complejos, lo que permite condiciones más complejas. Por ejemplo, la siguiente condición define un valor de etiqueta deseado para cada patrón de nombre y comprueba si el nombre del recurso coincide con el patrón, pero no tiene el valor de etiqueta necesario:

```json
{
    "count": {
        "value": [
            { "pattern": "test*", "envTag": "dev" },
            { "pattern": "dev*", "envTag": "dev" },
            { "pattern": "prod*", "envTag": "prod" },
        ],
        "name": "namePatternRequiredTag",
        "where": {
            "allOf": [
                {
                    "field": "name",
                    "like": "[current('namePatternRequiredTag').pattern]"
                },
                {
                    "field": "tags.env",
                    "notEquals": "[current('namePatternRequiredTag').envTag]"
                }
            ]
        }
    },
    "greater": 0
}
```

Para obtener ejemplos útiles, vea [Ejemplos de value count](../concepts/definition-structure.md#value-count-examples).

## <a name="referencing-array-resource-properties"></a>Referencia a las propiedades de recursos de matriz

Muchos casos de uso requieren que trabaje con propiedades de la matriz en el recurso evaluado. Algunos escenarios requieren que haga referencia a una matriz completa (por ejemplo, para comprobar su longitud). Otros requieren que aplique una condición a cada miembro de la matriz individual (por ejemplo, asegúrese de que todas las reglas del firewall bloqueen el acceso desde Internet). Comprender las distintas formas en las que Azure Policy puede hacer referencia a las propiedades de los recursos y cómo se comportan estas referencias cuando se relacionan con las propiedades de la matriz es la clave para escribir condiciones que cubran estos escenarios.

### <a name="referencing-resource-properties"></a>Referencia a las propiedades de recursos

Azure Policy puede hacer referencia a las propiedades de los recursos mediante [alias](../concepts/definition-structure.md#aliases). Hay dos maneras de hacer referencia a los valores de una propiedad de recurso en Azure Policy:

- Use una condición de [campo](../concepts/definition-structure.md#fields) para comprobar si **todas** las propiedades de recursos seleccionadas cumplen una condición. Ejemplo:

  ```json
  {
    "field" : "Microsoft.Test/resourceType/property",
    "equals": "value"
  }
  ```

- Use la función `field()` para acceder al valor de una propiedad. Ejemplo:

  ```json
  {
    "value": "[take(field('Microsoft.Test/resourceType/property'), 7)]",
    "equals": "prefix_"
  }
  ```

La condición de campo tiene un comportamiento implícito de "todos los elementos". Si el alias representa una colección de valores, comprueba si todos los valores individuales cumplen la condición. La función `field()` devuelve los valores representados por el alias tal como están, que otras funciones de plantilla pueden manipular.

### <a name="referencing-array-fields"></a>Referencia de campos de matriz

Las propiedades de recursos de matriz normalmente se representan mediante dos tipos diferentes de alias. Un alias "normal" y los [alias de matriz](../concepts/definition-structure.md#understanding-the--alias) que tienen adjunto el carácter `[*]`:

- `Microsoft.Test/resourceType/stringArray`
- `Microsoft.Test/resourceType/stringArray[*]`

#### <a name="referencing-the-array"></a>Referencia a la matriz

El primer alias representa un valor único, el valor de la propiedad `stringArray` del contenido de la solicitud. Dado que el valor de esa propiedad es una matriz, no resulta útil en las condiciones de la directiva. Por ejemplo:

```json
{
  "field": "Microsoft.Test/resourceType/stringArray",
  "equals": "..."
}
```

Esta condición compara toda la matriz `stringArray` con un valor de cadena único. La mayoría de las condiciones, incluida `equals`, solo aceptan valores de cadena, por lo que no se usa mucho en la comparación de una matriz con una cadena. El principal escenario en el que se hace referencia a la propiedad de matriz es útil cuando se comprueba si existe:

```json
{
  "field": "Microsoft.Test/resourceType/stringArray",
  "exists": "true"
}
```

Con la función `field()`, el valor devuelto es la matriz del contenido de la solicitud, que se puede usar con cualquiera de las [funciones de plantilla admitidas](../concepts/definition-structure.md#policy-functions) que aceptan argumentos de matriz. Por ejemplo, la siguiente condición comprueba si la longitud de `stringArray` es mayor que 0:

```json
{
  "value": "[length(field('Microsoft.Test/resourceType/stringArray'))]",
  "greater": 0
}
```

#### <a name="referencing-the-array-members-collection"></a>Referencia a una colección de miembros de matriz

Los alias que usan la sintaxis `[*]` representan una **colección de valores de propiedad seleccionados de una propiedad de matriz**, que es diferente de seleccionar la propiedad de la matriz. En el caso de `Microsoft.Test/resourceType/stringArray[*]`, devuelve una colección que tiene todos los miembros de `stringArray`. Como se mencionó anteriormente, una condición `field` comprueba que todas las propiedades de recursos seleccionadas cumplan la condición; por lo tanto, la siguiente condición solo es verdadera si **todos** los miembros de `stringArray` son iguales al "valor".

```json
{
  "field": "Microsoft.Test/resourceType/stringArray[*]",
  "equals": "value"
}
```

Si la matriz contiene objetos, se puede usar un alias `[*]` para seleccionar el valor de una propiedad específica de cada miembro de la matriz. Ejemplo:

```json
{
  "field": "Microsoft.Test/resourceType/objectArray[*].property",
  "equals": "value"
}
```

Esta condición es true si los valores de todas las propiedades `property` en `objectArray` son iguales que `"value"`. Para obtener más ejemplos, vea [Ejemplos de alias \[\*\] adicionales](#additional--alias-examples).

Cuando use la función `field()` para hacer referencia a un alias de matriz, el valor devuelto es una matriz de todos los valores seleccionados. Este comportamiento implica que el caso de uso común de la función `field()`, la capacidad de aplicar funciones de plantilla a los valores de propiedad de los recursos, es limitado. Las únicas funciones de plantilla que se pueden usar en este caso son las que aceptan argumentos de matriz. Por ejemplo, se puede obtener la longitud de la matriz con `[length(field('Microsoft.Test/resourceType/objectArray[*].property'))]`. Sin embargo, los escenarios más complejos, como la aplicación de una función de plantilla a cada miembro de la matriz y su comparación con un valor deseado, solo son posibles cuando se usa la expresión `count`. Para obtener más información, vea [Expresiones de recuento de campos](#field-count-expressions).

En resumen, consulte el contenido de recurso del siguiente ejemplo y los valores seleccionados que devolvieron diversos alias:

```json
{
  "tags": {
    "env": "prod"
  },
  "properties":
  {
    "stringArray": [ "a", "b", "c" ],
    "objectArray": [
      {
        "property": "value1",
        "nestedArray": [ 1, 2 ]
      },
      {
        "property": "value2",
        "nestedArray": [ 3, 4 ]
      }
    ]
  }
}
```

Al usar la condición de campo en el contenido de recurso del ejemplo, los resultados son los siguientes:

| Alias | Valores seleccionados |
|:--- |:---|
| `Microsoft.Test/resourceType/missingArray` | `null` |
| `Microsoft.Test/resourceType/missingArray[*]` | Colección de valores vacía. |
| `Microsoft.Test/resourceType/missingArray[*].property` | Colección de valores vacía. |
| `Microsoft.Test/resourceType/stringArray` | `["a", "b", "c"]` |
| `Microsoft.Test/resourceType/stringArray[*]` | `"a"`, `"b"`, `"c"` |
| `Microsoft.Test/resourceType/objectArray[*]` |  `{ "property": "value1", "nestedArray": [ 1, 2 ] }`,<br/>`{ "property": "value2", "nestedArray": [ 3, 4 ] }`|
| `Microsoft.Test/resourceType/objectArray[*].property` | `"value1"`, `"value2"` |
| `Microsoft.Test/resourceType/objectArray[*].nestedArray` | `[ 1, 2 ]`, `[ 3, 4 ]` |
| `Microsoft.Test/resourceType/objectArray[*].nestedArray[*]` | `1`, `2`, `3`, `4` |

Al usar la función `field()` en el contenido de recurso del ejemplo, los resultados son los siguientes:

| Expression | Valor devuelto |
|:--- |:---|
| `[field('Microsoft.Test/resourceType/missingArray')]` | `""` |
| `[field('Microsoft.Test/resourceType/missingArray[*]')]` | `[]` |
| `[field('Microsoft.Test/resourceType/missingArray[*].property')]` | `[]` |
| `[field('Microsoft.Test/resourceType/stringArray')]` | `["a", "b", "c"]` |
| `[field('Microsoft.Test/resourceType/stringArray[*]')]` | `["a", "b", "c"]` |
| `[field('Microsoft.Test/resourceType/objectArray[*]')]` |  `[{ "property": "value1", "nestedArray": [ 1, 2 ] }, { "property": "value2", "nestedArray": [ 3, 4 ] }]`|
| `[field('Microsoft.Test/resourceType/objectArray[*].property')]` | `["value1", "value2"]` |
| `[field('Microsoft.Test/resourceType/objectArray[*].nestedArray')]` | `[[ 1, 2 ], [ 3, 4 ]]` |
| `[field('Microsoft.Test/resourceType/objectArray[*].nestedArray[*]')]` | `[1, 2, 3, 4]` |

### <a name="field-count-expressions"></a>Expresiones de recuento de campos

Las expresiones [field count](../concepts/definition-structure.md#field-count) cuentan el número de miembros de la matriz que cumplen una condición y comparan el recuento con un valor de destino. La expresión `Count` es más intuitiva y versátil para evaluar las matrices que las condiciones `field`. La sintaxis es:

```json
{
  "count": {
    "field": <[*] alias>,
    "where": <optional policy condition expression>
  },
  "equals|greater|less|any other operator": <target value>
}
```

Cuando se usa sin una condición `where`, `count` simplemente devuelve la longitud de una matriz. Con el contenido de recurso del ejemplo de la sección anterior, la siguiente expresión `count` se evalúa como `true`, porque `stringArray` tiene tres miembros:

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/stringArray[*]"
  },
  "equals": 3
}
```

Este comportamiento también funciona con matrices anidadas. Por ejemplo, la siguiente expresión `count` se evalúa como `true`, ya que hay cuatro miembros de matriz en las matrices `nestedArray`:

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/objectArray[*].nestedArray[*]"
  },
  "greaterOrEquals": 4
}
```

La eficacia de `count` reside en la condición `where`. Cuando esta se especifica, Azure Policy enumera los miembros de la matriz y los evalúa con la condición, de modo que cuenta el número de miembros de la matriz que se han evaluado como `true`. En concreto, en cada iteración de la evaluación de la condición `where`, Azure Policy selecciona un único miembro de la matriz ***i** _ y evalúa el contenido del recurso con la condición `where` _*como si ***i**_ fuera el único miembro de la matriz_*. Disponer de un solo miembro de la matriz en cada iteración proporciona una manera de aplicar condiciones complejas en cada miembro de la matriz individual.

Ejemplo:

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/stringArray[*]",
    "where": {
      "field": "Microsoft.Test/resourceType/stringArray[*]",
      "equals": "a"
    }
  },
  "equals": 1
}
```

Para evaluar la expresión `count`, Azure Policy evalúa la condición `where` tres veces, una vez por cada miembro de `stringArray`, de modo que cuenta el número de veces que se ha evaluado como `true`.
Cuando la condición `where` hace referencia a los miembros de la matriz `Microsoft.Test/resourceType/stringArray[*]`, en lugar de seleccionar todos los miembros de `stringArray`, solo selecciona un miembro de la matriz cada vez:

| Iteración | Valores seleccionados de `Microsoft.Test/resourceType/stringArray[*]` | Resultado de la evaluación de `where` |
|:---|:---|:---|
| 1 | `"a"` | `true` |
| 2 | `"b"` | `false` |
| 3 | `"c"` | `false` |

`count` devuelve `1`.

La siguiente es una expresión más compleja:

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/objectArray[*]",
    "where": {
      "allOf": [
        {
          "field": "Microsoft.Test/resourceType/objectArray[*].property",
          "equals": "value2"
        },
        {
          "field": "Microsoft.Test/resourceType/objectArray[*].nestedArray[*]",
          "greater": 2
        }
      ]
    }
  },
  "equals": 1
}
```

| Iteración | Valores seleccionados | Resultado de la evaluación de `where` |
|:---|:---|:---|
| 1 | `Microsoft.Test/resourceType/objectArray[*].property` => `"value1"` </br> `Microsoft.Test/resourceType/objectArray[*].nestedArray[*]` => `1`, `2` | `false` |
| 2 | `Microsoft.Test/resourceType/objectArray[*].property` => `"value2"` </br> `Microsoft.Test/resourceType/objectArray[*].nestedArray[*]` => `3`, `4`| `true` |

`count` devuelve `1`.

El hecho de que la expresión `where` se evalúe con **todo** el contenido de la solicitud (con cambios solo en el miembro de la matriz que se está enumerando actualmente) implica que la condición `where` también puede hacer referencia a los campos que se encuentran fuera de la matriz:

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/objectArray[*]",
    "where": {
      "field": "tags.env",
      "equals": "prod"
    }
  },
  "equals": 0
}
```

| Iteración | Valores seleccionados | Resultado de la evaluación de `where` |
|:---|:---|:---|
| 1 | `tags.env` => `"prod"` | `true` |
| 2 | `tags.env` => `"prod"` | `true` |

Las expresiones de recuento anidadas se pueden usar para aplicar condiciones a los campos de matriz anidados. Por ejemplo, la siguiente condición comprueba que la matriz `objectArray[*]` tiene exactamente 2 miembros con `nestedArray[*]` que contiene 1 o más miembros:

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/objectArray[*]",
    "where": {
      "count": {
        "field": "Microsoft.Test/resourceType/objectArray[*].nestedArray[*]"
      },
      "greaterOrEquals": 1
    }
  },
  "equals": 2
}
```

| Iteración | Valores seleccionados | Resultado de evaluación de recuento anidado |
|:---|:---|:---|
| 1 | `Microsoft.Test/resourceType/objectArray[*].nestedArray[*]` => `1`, `2` | `nestedArray[*]` tiene 2 miembros => `true` |
| 2 | `Microsoft.Test/resourceType/objectArray[*].nestedArray[*]` => `3`, `4` | `nestedArray[*]` tiene 2 miembros => `true` |

Puesto que ambos miembros de `objectArray[*]` tienen una matriz secundaria `nestedArray[*]` con 2 miembros, la expresión de recuento exterior devuelve `2`.

Ejemplo más complejo: compruebe que la matriz `objectArray[*]` tiene exactamente 2 miembros con `nestedArray[*]` con cualquier miembro igual a `2` o `3`:

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/objectArray[*]",
    "where": {
      "count": {
        "field": "Microsoft.Test/resourceType/objectArray[*].nestedArray[*]",
        "where": {
            "field": "Microsoft.Test/resourceType/objectArray[*].nestedArray[*]",
            "in": [ 2, 3 ]
        }
      },
      "greaterOrEquals": 1
    }
  },
  "equals": 2
}
```

| Iteración | Valores seleccionados | Resultado de evaluación de recuento anidado
|:---|:---|:---|
| 1 | `Microsoft.Test/resourceType/objectArray[*].nestedArray[*]` => `1`, `2` | `nestedArray[*]` contiene `2` => `true` |
| 2 | `Microsoft.Test/resourceType/objectArray[*].nestedArray[*]` => `3`, `4` | `nestedArray[*]` contiene `3` => `true` |

Dado que ambos miembros de `objectArray[*]` tienen una matriz secundaria `nestedArray[*]` que contiene `2` o `3`, se devuelve la expresión de recuento exterior `2`.

> [!NOTE]
> Las expresiones de recuento de campos anidados solo pueden hacer referencia a matrices anidadas. Por ejemplo, la expresión de recuento que hace referencia a `Microsoft.Test/resourceType/objectArray[*]` puede tener un recuento anidado que tenga como destino la matriz anidada `Microsoft.Test/resourceType/objectArray[*].nestedArray[*]`, pero no puede tener una expresión de recuento anidada como destino `Microsoft.Test/resourceType/stringArray[*]`.

#### <a name="accessing-current-array-member-with-template-functions"></a>Acceso al miembro de la matriz actual con funciones de plantilla

Si emplea funciones de plantilla, use la función `current()` para acceder al valor del miembro de la matriz actual o a los valores de cualquiera de sus propiedades. Para acceder al valor del miembro de la matriz actual, pase el alias definido en `count.field` o cualquiera de sus alias secundarios como argumento a la función `current()`. Por ejemplo:

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/objectArray[*]",
    "where": {
        "value": "[current('Microsoft.Test/resourceType/objectArray[*].property')]",
        "like": "value*"
    }
  },
  "equals": 2
}

```

| Iteración | Valor devuelto por `current()` | Resultado de la evaluación de `where` |
|:---|:---|:---|
| 1 | Valor de `property` en el primer miembro de `objectArray[*]`: `value1` | `true` |
| 2 | Valor de `property` en el primer miembro de `objectArray[*]`: `value2` | `true` |

#### <a name="the-field-function-inside-where-conditions"></a>Función field dentro de condiciones where

La función `field()` también se puede usar para acceder al valor del miembro de la matriz actual siempre que la expresión **count** no esté dentro de una **condición de existencia** (la función `field()` siempre hace referencia al recurso evaluado en la condición **if**). El comportamiento de `field()` cuando hace referencia a la matriz evaluada se basa en los siguientes conceptos:

1. Los alias de matriz se resuelven en una colección de valores seleccionados a partir de todos los miembros de la matriz.
1. Las funciones `field()` que hacen referencia a alias de matriz devuelven una matriz con los valores seleccionados.
1. Al hacer referencia al alias de matriz que se contó dentro de la condición `where`, se devuelve una colección con un solo valor seleccionado en el miembro de la matriz, que se evalúa en la iteración actual.

Este comportamiento implica que cuando se hace referencia al miembro de matriz contado con una función `field()` dentro de la condición `where`, **se devuelve una matriz con un solo miembro**. Aunque este comportamiento puede no ser intuitivo, es coherente con la idea de que los alias de matriz siempre devuelven una colección de propiedades seleccionadas. Este es un ejemplo:

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/stringArray[*]",
    "where": {
      "field": "Microsoft.Test/resourceType/stringArray[*]",
      "equals": "[field('Microsoft.Test/resourceType/stringArray[*]')]"
    }
  },
  "equals": 0
}
```

| Iteración | Valores de expresión | Resultado de la evaluación de `where` |
|:---|:---|:---|
| 1 | `Microsoft.Test/resourceType/stringArray[*]` => `"a"` </br>  `[field('Microsoft.Test/resourceType/stringArray[*]')]` => `[ "a" ]` | `false` |
| 2 | `Microsoft.Test/resourceType/stringArray[*]` => `"b"` </br>  `[field('Microsoft.Test/resourceType/stringArray[*]')]` => `[ "b" ]` | `false` |
| 3 | `Microsoft.Test/resourceType/stringArray[*]` => `"c"` </br>  `[field('Microsoft.Test/resourceType/stringArray[*]')]` => `[ "c" ]` | `false` |

Por lo tanto, cuando sea necesario acceder al valor del alias de matriz contado con una función `field()`, deberá encapsularlo con una función de plantilla `first()`:

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/stringArray[*]",
    "where": {
      "field": "Microsoft.Test/resourceType/stringArray[*]",
      "equals": "[first(field('Microsoft.Test/resourceType/stringArray[*]'))]"
    }
  }
}
```

| Iteración | Valores de expresión | Resultado de la evaluación de `where` |
|:---|:---|:---|
| 1 | `Microsoft.Test/resourceType/stringArray[*]` => `"a"` </br>  `[first(field('Microsoft.Test/resourceType/stringArray[*]'))]` => `"a"` | `true` |
| 2 | `Microsoft.Test/resourceType/stringArray[*]` => `"b"` </br>  `[first(field('Microsoft.Test/resourceType/stringArray[*]'))]` => `"b"` | `true` |
| 3 | `Microsoft.Test/resourceType/stringArray[*]` => `"c"` </br>  `[first(field('Microsoft.Test/resourceType/stringArray[*]'))]` => `"c"` | `true` |

Para obtener ejemplos útiles, vea [Ejemplos de field count](../concepts/definition-structure.md#field-count-examples).

## <a name="modifying-arrays"></a>Modificación de matrices

[append](../concepts/effects.md#append) y [modify](../concepts/effects.md#modify) alteran las propiedades de un recurso durante la creación o la actualización. Cuando se trabaja con propiedades de matriz, el comportamiento de estos efectos depende de si la operación intenta modificar el alias **\[\*\]** :

> [!NOTE]
> El uso del efecto `modify` con alias está actualmente en **versión preliminar**.

|Alias |Efecto | Resultado |
|-|-|-|
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules` | `append` | Azure Policy anexa toda la matriz especificada en los detalles del efecto, si falta. |
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules` | `modify` con la operación `add` | Azure Policy anexa toda la matriz especificada en los detalles del efecto, si falta. |
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules` | `modify` con la operación `addOrReplace` | Azure Policy anexa toda la matriz especificada en los detalles del efecto, si falta, o reemplaza la matriz existente. |
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]` | `append` | Azure Policy anexa el miembro de la matriz especificado en los detalles del efecto. |
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]` | `modify` con la operación `add` | Azure Policy anexa el miembro de la matriz especificado en los detalles del efecto. |
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]` | `modify` con la operación `addOrReplace` | Azure Policy quita todos los miembros de la matriz existentes y anexa el miembro de la matriz especificado en los detalles del efecto. |
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].action` | `append` | Azure Policy anexa un valor a la propiedad `action` de cada miembro de la matriz. |
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].action` | `modify` con la operación `add` | Azure Policy anexa un valor a la propiedad `action` de cada miembro de la matriz. |
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].action` | `modify` con la operación `addOrReplace` | Azure Policy anexa o reemplaza la propiedad `action` existente de cada miembro de la matriz. |

Para obtener más información, consulte [Ejemplos de append](../concepts/effects.md#append-examples).

## <a name="additional--alias-examples"></a>Ejemplos de alias [*] adicionales

Se recomienda usar las [expresiones de recuento de campos](#field-count-expressions) para comprobar si "todos" o "cualquiera" de los miembros de una matriz del contenido de la solicitud cumplen una condición. Pero en algunas condiciones sencillas es posible obtener el mismo resultado mediante un descriptor de acceso de campo con un alias de matriz (como se describe en [Referencia a una colección de miembros de matriz](#referencing-the-array-members-collection)). Este modelo puede ser útil en reglas de directiva que superan el límite de expresiones **count** permitidas. Estos son algunos ejemplos de casos de uso comunes:

La regla de directiva siguiente es un ejemplo para la tabla de escenario:

```json
"policyRule": {
    "if": {
        "allOf": [
            {
                "field": "Microsoft.Storage/storageAccounts/networkAcls.ipRules",
                "exists": "true"
            },
            <-- Condition (see table below) -->
        ]
    },
    "then": {
        "effect": "[parameters('effectType')]"
    }
}
```

La matriz **ipRules** es como sigue para la tabla de escenario:

```json
"ipRules": [
    {
        "value": "127.0.0.1",
        "action": "Allow"
    },
    {
        "value": "192.168.1.1",
        "action": "Allow"
    }
]
```

Para cada ejemplo de condición siguiente, reemplace `<field>` por `"field": "Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].value"`.

Los siguientes resultados se derivan de la combinación de la condición y la regla de directiva de ejemplo y la matriz de valores existentes anteriores:

|Condición |Resultado | Escenario |Explicación |
|-|-|-|-|
|`{<field>,"notEquals":"127.0.0.1"}` |Nada |Ninguna coincidencia |Un elemento de matriz se evalúa como false (127.0.0.1 != 127.0.0.1) y otro como true (127.0.0.1 != 192.168.1.1), por lo que la condición **notEquals** es _false_ y el efecto no se desencadena. |
|`{<field>,"notEquals":"10.0.4.1"}` |Efecto de la directiva |Ninguna coincidencia |Ambos elementos de matriz se evalúan como true (10.0.4.1 != 127.0.0.1 and 10.0.4.1 != 192.168.1.1), por lo que la condición **notEquals** es _true_ y el efecto se desencadena. |
|`"not":{<field>,"notEquals":"127.0.0.1" }` |Efecto de la directiva |Una o más coincidencias |Un elemento de matriz se evalúa como false (127.0.0.1 != 127.0.0.1) y otro como true (127.0.0.1 != 192.168.1.1), por lo que la condición **notEquals** es _false_. El operador lógico se evalúa como true (**not** _false_), por lo que se desencadena el efecto. |
|`"not":{<field>,"notEquals":"10.0.4.1"}` |Nada |Una o más coincidencias |Ambos elementos de matriz se evalúan como true (10.0.4.1 != 127.0.0.1 y 10.0.4.1 != 192.168.1.1), por lo que la condición **notEquals** es _true_. El operador lógico se evalúa como false (**not** _true_), por lo que no se desencadena el efecto. |
|`"not":{<field>,"Equals":"127.0.0.1"}` |Efecto de la directiva |Coincidencia parcial |Un elemento de matriz se evalúa como true (127.0.0.1 == 127.0.0.1) y otro como false (127.0.0.1 == 192.168.1.1), por lo que la condición **Equals** es _false_. El operador lógico se evalúa como true (**not** _false_), por lo que se desencadena el efecto. |
|`"not":{<field>,"Equals":"10.0.4.1"}` |Efecto de la directiva |Coincidencia parcial |Ambos elementos de matriz se evalúan como false (10.0.4.1 == 127.0.0.1 y 10.0.4.1 == 192.168.1.1), por lo que la condición **Equals** es _false_. El operador lógico se evalúa como true (**not** _false_), por lo que se desencadena el efecto. |
|`{<field>,"Equals":"127.0.0.1"}` |Nada |Coincidencia total |Un elemento de matriz se evalúa como true (127.0.0.1 == 127.0.0.1) y otro como false (127.0.0.1 == 192.168.1.1), por lo que la condición **Equals** es _false_ y el efecto no se desencadena. |
|`{<field>,"Equals":"10.0.4.1"}` |Nada |Coincidencia total |Ambos elementos de matriz se evalúan como false (10.0.4.1 == 127.0.0.1 y 10.0.4.1 == 192.168.1.1), por lo que la condición **Equals** es _false_ y el efecto no se desencadena. |

## <a name="next-steps"></a>Pasos siguientes

- Puede consultar ejemplos en [Ejemplos de Azure Policy](../samples/index.md).
- Revise la [estructura de definición de Azure Policy](../concepts/definition-structure.md).
- Vea la [Descripción de los efectos de directivas](../concepts/effects.md).
- Obtenga información acerca de cómo se pueden [crear directivas mediante programación](programmatically-create.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](remediate-resources.md).
- En [Organización de los recursos con grupos de administración de Azure](../../management-groups/overview.md), obtendrá información sobre lo que es un grupo de administración.
