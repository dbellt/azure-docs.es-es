---
title: 'Funciones de plantillas: objetos'
description: Se describen las funciones que se usan en una plantilla de Azure Resource Manager para trabajar con objetos.
ms.topic: conceptual
ms.date: 05/13/2021
ms.openlocfilehash: aa52cee2236ac3eab5090b1caad27b6cbf516c7a
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111959612"
---
# <a name="object-functions-for-arm-templates"></a>Funciones de objeto para plantillas de Resource Manager

Resource Manager proporciona varias funciones para trabajar con objetos en la plantilla de Resource Manager:

* [contains](#contains)
* [createObject](#createobject)
* [empty](#empty)
* [intersection](#intersection)
* [json](#json)
* [length](#length)
* [null](#null)
* [union](#union)

## <a name="contains"></a>contains

`contains(container, itemToFind)`

Comprueba si una matriz contiene un valor, un objeto contiene una clave o una cadena contiene una subcadena. La comparación de cadena distingue mayúsculas de minúsculas. Pero, cuando se prueba si un objeto contiene una clave, la comparación no distingue mayúsculas de minúsculas.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| contenedor |Sí |matriz, objeto o cadena |El valor que contiene el valor para buscar. |
| itemToFind |Sí |cadena o entero |El valor para buscar. |

### <a name="return-value"></a>Valor devuelto

**True** si el elemento se encuentra; en caso contrario, **False**.

### <a name="example"></a>Ejemplo

En la [plantilla de ejemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/contains.json) siguiente se muestra cómo utilizar contains con diferentes tipos:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "stringToTest": {
      "type": "string",
      "defaultValue": "OneTwoThree"
    },
    "objectToTest": {
      "type": "object",
      "defaultValue": {
        "one": "a",
        "two": "b",
        "three": "c"
      }
    },
    "arrayToTest": {
      "type": "array",
      "defaultValue": [ "one", "two", "three" ]
    }
  },
  "resources": [
  ],
  "outputs": {
    "stringTrue": {
      "type": "bool",
      "value": "[contains(parameters('stringToTest'), 'e')]"
    },
    "stringFalse": {
      "type": "bool",
      "value": "[contains(parameters('stringToTest'), 'z')]"
    },
    "objectTrue": {
      "type": "bool",
      "value": "[contains(parameters('objectToTest'), 'one')]"
    },
    "objectFalse": {
      "type": "bool",
      "value": "[contains(parameters('objectToTest'), 'a')]"
    },
    "arrayTrue": {
      "type": "bool",
      "value": "[contains(parameters('arrayToTest'), 'three')]"
    },
    "arrayFalse": {
      "type": "bool",
      "value": "[contains(parameters('arrayToTest'), 'four')]"
    }
  }
}
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| stringTrue | Bool | True |
| stringFalse | Bool | False |
| objectTrue | Bool | True |
| objectFalse | Bool | False |
| arrayTrue | Bool | True |
| arrayFalse | Bool | False |

## <a name="createobject"></a>createObject

`createObject(key1, value1, key2, value2, ...)`

Crea un objeto a partir de las claves y los valores.

La función `createObject` no es compatible con Bicep.  Construya un objeto mediante `{}`. Vea [Objetos](../bicep/data-types.md#objects).

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| key1 |No |string |El nombre de la clave. |
| value1 |No |int, booleano, cadena, objeto o matriz |Valor de la clave. |
| claves adicionales |No |string |Nombres adicionales de las claves. |
| valores adicionales |No |int, booleano, cadena, objeto o matriz |Valores adicionales para las claves. |

La función solo acepta un número par de parámetros. Cada clave debe tener un valor coincidente.

### <a name="return-value"></a>Valor devuelto

Un objeto con cada par de clave y valor.

### <a name="example"></a>Ejemplo

En el ejemplo siguiente se crea un objeto a partir de distintos tipos de valores.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
  ],
  "outputs": {
    "newObject": {
      "type": "object",
      "value": "[createObject('intProp', 1, 'stringProp', 'abc', 'boolProp', true(), 'arrayProp', createArray('a', 'b', 'c'), 'objectProp', createObject('key1', 'value1'))]"
    }
  }
}
```

La salida del ejemplo anterior con los valores predeterminados es un objeto denominado `newObject`con el siguiente valor:

```json
{
  "intProp": 1,
  "stringProp": "abc",
  "boolProp": true,
  "arrayProp": ["a", "b", "c"],
  "objectProp": {"key1": "value1"}
}
```

## <a name="empty"></a>empty

`empty(itemToTest)`

Determina si una matriz, un objeto o una cadena están vacíos.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| itemToTest |Sí |matriz, objeto o cadena |El valor para comprobar si está vacío. |

### <a name="return-value"></a>Valor devuelto

Devuelve **True** si el valor está vacío; en caso contrario, **False**.

### <a name="example"></a>Ejemplo

En la [plantilla de ejemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/empty.json) siguiente se comprueba si una matriz, un objeto y una cadena están vacíos.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "testArray": {
      "type": "array",
      "defaultValue": []
    },
    "testObject": {
      "type": "object",
      "defaultValue": {}
    },
    "testString": {
      "type": "string",
      "defaultValue": ""
    }
  },
  "resources": [
  ],
  "outputs": {
    "arrayEmpty": {
      "type": "bool",
      "value": "[empty(parameters('testArray'))]"
    },
    "objectEmpty": {
      "type": "bool",
      "value": "[empty(parameters('testObject'))]"
    },
    "stringEmpty": {
      "type": "bool",
      "value": "[empty(parameters('testString'))]"
    }
  }
}
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| arrayEmpty | Bool | True |
| objectEmpty | Bool | True |
| stringEmpty | Bool | True |

## <a name="intersection"></a>intersección

`intersection(arg1, arg2, arg3, ...)`

Devuelve una única matriz u objeto con los elementos comunes de los parámetros.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| arg1 |Sí |matriz u objeto |El primer valor que se utilizará para buscar elementos comunes. |
| arg2 |Sí |matriz u objeto |El segundo valor que se utilizará para buscar elementos comunes. |
| argumentos adicionales |No |matriz u objeto |Valores adicionales que se utilizarán para buscar elementos comunes. |

### <a name="return-value"></a>Valor devuelto

Una matriz o un objeto con los elementos comunes.

### <a name="example"></a>Ejemplo

En la [plantilla de ejemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/intersection.json) siguiente se muestra cómo utilizar la intersección con matrices y objetos:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "firstObject": {
      "type": "object",
      "defaultValue": {
        "one": "a",
        "two": "b",
        "three": "c"
      }
    },
    "secondObject": {
      "type": "object",
      "defaultValue": {
        "one": "a",
        "two": "z",
        "three": "c"
      }
    },
    "firstArray": {
      "type": "array",
      "defaultValue": [ "one", "two", "three" ]
    },
    "secondArray": {
      "type": "array",
      "defaultValue": [ "two", "three" ]
    }
  },
  "resources": [
  ],
  "outputs": {
    "objectOutput": {
      "type": "object",
      "value": "[intersection(parameters('firstObject'), parameters('secondObject'))]"
    },
    "arrayOutput": {
      "type": "array",
      "value": "[intersection(parameters('firstArray'), parameters('secondArray'))]"
    }
  }
}
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| objectOutput | Object | {"one": "a", "three": "c"} |
| arrayOutput | Array | ["two", "three"] |

<a id="json"></a>

## <a name="json"></a>json

`json(arg1)`

Convierte una cadena JSON válida en un tipo de datos JSON.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| arg1 |Sí |string |Valor que se va a convertir en JSON. La cadena debe ser una cadena JSON con el formato correcto. |

### <a name="return-value"></a>Valor devuelto

El tipo de datos JSON de la cadena especificada o un valor vacío si se especifica **null**.

### <a name="remarks"></a>Observaciones

Si necesita incluir un valor de parámetro o variable en el objeto JSON, use la función [concat](template-functions-string.md#concat) para crear la cadena que se pasa a la función.

También puede usar [null ()](#null) para obtener un valor null.

### <a name="example"></a>Ejemplo

En la siguiente [plantilla de ejemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/json.json) se muestra cómo usar la función json. Tenga en cuenta que puede pasar **null** para un objeto vacío.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "jsonEmptyObject": {
      "type": "string",
      "defaultValue": "null"
    },
    "jsonObject": {
      "type": "string",
      "defaultValue": "{\"a\": \"b\"}"
    },
    "jsonString": {
      "type": "string",
      "defaultValue": "\"test\""
    },
    "jsonBoolean": {
      "type": "string",
      "defaultValue": "true"
    },
    "jsonInt": {
      "type": "string",
      "defaultValue": "3"
    },
    "jsonArray": {
      "type": "string",
      "defaultValue": "[[1,2,3 ]"
    },
    "concatValue": {
      "type": "string",
      "defaultValue": "demo value"
    }
  },
  "resources": [
  ],
  "outputs": {
    "emptyObjectOutput": {
      "type": "bool",
      "value": "[empty(json(parameters('jsonEmptyObject')))]"
    },
    "objectOutput": {
      "type": "object",
      "value": "[json(parameters('jsonObject'))]"
    },
    "stringOutput": {
      "type": "string",
      "value": "[json(parameters('jsonString'))]"
    },
    "booleanOutput": {
      "type": "bool",
      "value": "[json(parameters('jsonBoolean'))]"
    },
    "intOutput": {
      "type": "int",
      "value": "[json(parameters('jsonInt'))]"
    },
    "arrayOutput": {
      "type": "array",
      "value": "[json(parameters('jsonArray'))]"
    },
    "concatObjectOutput": {
      "type": "object",
      "value": "[json(concat('{\"a\": \"', parameters('concatValue'), '\"}'))]"
    }
  }
}
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| emptyObjectOutput | Boolean | True |
| objectOutput | Object | {"a": "b"} |
| stringOutput | String | test |
| booleanOutput | Boolean | True |
| intOutput | Entero | 3 |
| arrayOutput | Array | [ 1, 2, 3 ] |
| concatObjectOutput | Object | {"a": "demo value"} |

## <a name="length"></a>length

`length(arg1)`

Devuelve el número de elementos de una matriz, caracteres de una cadena o propiedades de nivel raíz de un objeto.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| arg1 |Sí |matriz, cadena u objeto |La matriz que se usará para obtener el número de elementos, la cadena que se usará para obtener el número de caracteres o el objeto que se usará para obtener el número de propiedades del nivel raíz. |

### <a name="return-value"></a>Valor devuelto

Un entero.

### <a name="example"></a>Ejemplo

En la [plantilla de ejemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/length.json) siguiente se muestra cómo utilizar length con una matriz y una cadena:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "arrayToTest": {
      "type": "array",
      "defaultValue": [
        "one",
        "two",
        "three"
      ]
    },
    "stringToTest": {
      "type": "string",
      "defaultValue": "One Two Three"
    },
    "objectToTest": {
      "type": "object",
      "defaultValue": {
        "propA": "one",
        "propB": "two",
        "propC": "three",
        "propD": {
          "propD-1": "sub",
          "propD-2": "sub"
        }
      }
    }
  },
  "resources": [],
  "outputs": {
    "arrayLength": {
      "type": "int",
      "value": "[length(parameters('arrayToTest'))]"
    },
    "stringLength": {
      "type": "int",
      "value": "[length(parameters('stringToTest'))]"
    },
    "objectLength": {
      "type": "int",
      "value": "[length(parameters('objectToTest'))]"
    }
  }
}
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| arrayLength | Int | 3 |
| stringLength | Int | 13 |
| objectLength | Int | 4 |

## <a name="null"></a>null

`null()`

Devuelve NULL.

La función `null` no está disponible en Bicep. En su lugar, use la palabra clave `null`.

### <a name="parameters"></a>Parámetros

La función null no acepta ningún parámetro.

### <a name="return-value"></a>Valor devuelto

Valor booleano que siempre es null.

### <a name="example"></a>Ejemplo

En el siguiente ejemplo se utiliza la función null.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [],
  "outputs": {
    "emptyOutput": {
      "type": "bool",
      "value": "[empty(null())]"
    }
  }
}
```

El resultado del ejemplo anterior es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| emptyOutput | Bool | True |

## <a name="union"></a>union

`union(arg1, arg2, arg3, ...)`

Devuelve una única matriz u objeto con todos los elementos de los parámetros. Los valores o las claves duplicados solo se incluyen una vez.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| arg1 |Sí |matriz u objeto |El primer valor que se utiliza para unir elementos. |
| arg2 |Sí |matriz u objeto |El segundo valor que se utiliza para unir elementos. |
| argumentos adicionales |No |matriz u objeto |Valores adicionales que se utilizan para unir elementos. |

### <a name="return-value"></a>Valor devuelto

Una matriz u objeto.

### <a name="example"></a>Ejemplo

En la [plantilla de ejemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/union.json) siguiente se muestra cómo utilizar la unión con matrices y objetos:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "firstObject": {
      "type": "object",
      "defaultValue": {
        "one": "a",
        "two": "b",
        "three": "c1"
      }
    },
    "secondObject": {
      "type": "object",
      "defaultValue": {
        "three": "c2",
        "four": "d",
        "five": "e"
      }
    },
    "firstArray": {
      "type": "array",
      "defaultValue": [ "one", "two", "three" ]
    },
    "secondArray": {
      "type": "array",
      "defaultValue": [ "three", "four" ]
    }
  },
  "resources": [
  ],
  "outputs": {
    "objectOutput": {
      "type": "object",
      "value": "[union(parameters('firstObject'), parameters('secondObject'))]"
    },
    "arrayOutput": {
      "type": "array",
      "value": "[union(parameters('firstArray'), parameters('secondArray'))]"
    }
  }
}
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| objectOutput | Object | {"one": "a", "two": "b", "three": "c2", "four": "d", "five": "e"} |
| arrayOutput | Array | ["one", "two", "three", "four"] |

## <a name="next-steps"></a>Pasos siguientes

* Para obtener una descripción de las secciones de una plantilla de ARM, vea [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](./syntax.md).