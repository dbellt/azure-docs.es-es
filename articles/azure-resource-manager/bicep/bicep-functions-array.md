---
title: 'Funciones de Bicep: array'
description: Describe las funciones que se usarán en un archivo de Bicep para trabajar con matrices.
author: mumian
ms.topic: conceptual
ms.author: jgao
ms.date: 06/01/2021
ms.openlocfilehash: e782ec0d77930651346cc0fc97e52d5c473b9245
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "111027125"
---
# <a name="array-functions-for-bicep"></a>Funciones de matriz para Bicep

Resource Manager ofrece varias funciones para trabajar con matrices en Bicep:

* [array](#array)
* [concat](#concat)
* [contains](#contains)
* [empty](#empty)
* [first](#first)
* [intersection](#intersection)
* [last](#last)
* [length](#length)
* [max](#max)
* [min](#min)
* [range](#range)
* [skip](#skip)
* [take](#take)
* [union](#union)

Para obtener una matriz de valores de cadena delimitada por un valor, consulte [split](./bicep-functions-string.md#split).

## <a name="array"></a>array

`array(convertToArray)`

Convierte el valor en una matriz.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| convertToArray |Sí |int, string, array u object |Valor que se convierte en matriz. |

### <a name="return-value"></a>Valor devuelto

Matriz .

### <a name="example"></a>Ejemplo

En el ejemplo siguiente se muestra cómo utilizar la función de matriz con diferentes tipos.

```bicep
param intToConvert int = 1
param stringToConvert string = 'efgh'
param objectToConvert object = {
  'a': 'b'
  'c': 'd'
}

output intOutput array = array(intToConvert)
output stringOutput array = array(stringToConvert)
output objectOutput array = array(objectToConvert)
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| intOutput | Array | [1] |
| stringOutput | Array | ["efgh"] |
| objectOutput | Array | [{"a": "b", "c": "d"}] |

## <a name="concat"></a>concat

`concat(arg1, arg2, arg3, ...)`

Combina varias matrices y devuelve la matriz concatenada.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| arg1 |Sí |array |La primera matriz para la concatenación. |
| argumentos adicionales |No |array |Matrices adicionales en orden secuencial para la concatenación. |

Esta función toma cualquier número de matrices y las combina.

### <a name="return-value"></a>Valor devuelto

Una cadena de valores concatenados.

### <a name="example"></a>Ejemplo

En el ejemplo siguiente se muestra cómo combinar dos matrices.

```bicep
param firstArray array = [
  '1-1'
  '1-2'
  '1-3'
]
param secondArray array = [
  '2-1'
  '2-2'
  '2-3'
]

output return array = concat(firstArray, secondArray)
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| return | Array | ["1-1", "1-2", "1-3", "2-1", "2-2", "2-3"] |

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

En el ejemplo siguiente se muestra cómo utilizar contains con diferentes tipos:

```bicep
param stringToTest string = 'OneTwoThree'
param objectToTest object = {
  'one': 'a'
  'two': 'b'
  'three': 'c'
}
param arrayToTest array = [
  'one'
  'two'
  'three'
]

output stringTrue bool = contains(stringToTest, 'e')
output stringFalse bool = contains(stringToTest, 'z')
output objectTrue bool = contains(objectToTest, 'one')
output objectFalse bool = contains(objectToTest, 'a')
output arrayTrue bool = contains(arrayToTest, 'three')
output arrayFalse bool = contains(arrayToTest, 'four')
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

En el ejemplo siguiente se comprueba si una matriz, un objeto y una cadena están vacíos.

```bicep
param testArray array = []
param testObject object = {}
param testString string = ''

output arrayEmpty bool = empty(testArray)
output objectEmpty bool = empty(testObject)
output stringEmpty bool = empty(testString)
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| arrayEmpty | Bool | True |
| objectEmpty | Bool | True |
| stringEmpty | Bool | True |

## <a name="first"></a>first

`first(arg1)`

Devuelve el primer elemento de la matriz o el primer carácter de la cadena.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| arg1 |Sí |matriz o cadena |El valor para recuperar el primer elemento o carácter. |

### <a name="return-value"></a>Valor devuelto

El tipo (cadena, entero, matriz u objeto) del primer elemento en una matriz o el primer carácter de una cadena.

### <a name="example"></a>Ejemplo

En el ejemplo siguiente se muestra cómo utilizar la primera función con una matriz y una cadena.

```bicep
param arrayToTest array = [
  'one'
  'two'
  'three'
]

output arrayOutput string = first(arrayToTest)
output stringOutput string = first('One Two Three')
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| arrayOutput | String | one |
| stringOutput | String | O |

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

En el ejemplo siguiente se muestra cómo utilizar la intersección con matrices y objetos:

```bicep
param firstObject object = {
  'one': 'a'
  'two': 'b'
  'three': 'c'
}

param secondObject object = {
  'one': 'a'
  'two': 'z'
  'three': 'c'
}

param firstArray array = [
  'one'
  'two'
  'three'
]

param secondArray array = [
  'two'
  'three'
]

output objectOutput object = intersection(firstObject, secondObject)
output arrayOutput array = intersection(firstArray, secondArray)
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| objectOutput | Object | {"one": "a", "three": "c"} |
| arrayOutput | Array | ["two", "three"] |

## <a name="last"></a>last

`last (arg1)`

Devuelve el último elemento de la matriz o el último carácter de la cadena.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| arg1 |Sí |matriz o cadena |El valor para recuperar el último elemento o carácter. |

### <a name="return-value"></a>Valor devuelto

El tipo (cadena, entero, matriz u objeto) del último elemento de una matriz o el último carácter de una cadena.

### <a name="example"></a>Ejemplo

En el ejemplo siguiente se muestra cómo utilizar la última función con una matriz y una cadena.

```bicep
param arrayToTest array = [
  'one'
  'two'
  'three'
]

output arrayOutput string = last(arrayToTest)
output stringOutput string = last('One Two three')
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| arrayOutput | String | three |
| stringOutput | String | e |

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

En el ejemplo siguiente se muestra cómo utilizar length con una matriz y una cadena:

```bicep
param arrayToTest array = [
  'one'
  'two'
  'three'
]
param stringToTest string = 'One Two Three'
param objectToTest object = {
  'propA': 'one'
  'propB': 'two'
  'propC': 'three'
  'propD': {
    'propD-1': 'sub'
    'propD-2': 'sub'
  }
}

output arrayLength int = length(arrayToTest)
output stringLength int = length(stringToTest)
output objectLength int = length(objectToTest)
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| arrayLength | Int | 3 |
| stringLength | Int | 13 |
| objectLength | Int | 4 |

## <a name="max"></a>max

`max(arg1)`

Devuelve el valor máximo de una matriz de enteros o una lista separada por comas de enteros.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| arg1 |Sí |matriz de enteros, o lista separada por comas de enteros |La colección para obtener el valor máximo. |

### <a name="return-value"></a>Valor devuelto

Un entero que representa el valor máximo.

### <a name="example"></a>Ejemplo

En el ejemplo siguiente se muestra cómo utilizar max con una matriz y una lista de enteros:

```bicep
param arrayToTest array = [
  0
  3
  2
  5
  4
]

output arrayOutput int = max(arrayToTest)
output intOutput int = max(0,3,2,5,4)
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| arrayOutput | Int | 5 |
| intOutput | Int | 5 |

## <a name="min"></a>Min

`min(arg1)`

Devuelve el valor mínimo de una matriz de enteros o una lista separada por comas de enteros.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| arg1 |Sí |matriz de enteros, o lista separada por comas de enteros |La colección para obtener el valor mínimo. |

### <a name="return-value"></a>Valor devuelto

Un entero que representa el valor mínimo.

### <a name="example"></a>Ejemplo

En el ejemplo siguiente se muestra cómo utilizar min con una matriz y una lista de enteros:

```bicep
param arrayToTest array = [
  0
  3
  2
  5
  4
]

output arrayOutput int = min(arrayToTest)
output intOutput int = min(0,3,2,5,4)
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| arrayOutput | Int | 0 |
| intOutput | Int | 0 |

## <a name="range"></a>range

`range(startIndex, count)`

Crea una matriz de enteros a partir de un entero de inicio y contiene un número de elementos.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| startIndex |Sí |int |El primer entero de la matriz. La suma de startIndex y count no debe ser mayor que 2 147 483 647. |
| count |Sí |int |El número de enteros en la matriz. Debe ser un entero no negativo de hasta 10 000. |

### <a name="return-value"></a>Valor devuelto

Una matriz de enteros.

### <a name="example"></a>Ejemplo

En el ejemplo siguiente se muestra cómo utilizar la función range:

```bicep
param startingInt int = 5
param numberOfElements int = 3

output rangeOutput array = range(startingInt, numberOfElements)
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| rangeOutput | Array | [5, 6, 7] |

## <a name="skip"></a>skip

`skip(originalValue, numberToSkip)`

Devuelve una matriz con todos los elementos después del número especificado de la matriz, o devuelve una cadena con todos los caracteres después del número especificado en la cadena.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| originalValue |Sí |matriz o cadena |La matriz o cadena que se usará para la omisión. |
| numberToSkip |Sí |int |El número de elementos o caracteres que se van a omitir. Si este valor es 0 o un valor inferior, se devuelven todos los elementos o caracteres del valor. Si es mayor que la longitud de la matriz o la cadena, se devuelve una matriz o cadena vacía. |

### <a name="return-value"></a>Valor devuelto

Una matriz o cadena.

### <a name="example"></a>Ejemplo

En el ejemplo siguiente se omite el número especificado de elementos de la matriz, y el número especificado de caracteres de la cadena.

```bicep
param testArray array = [
  'one'
  'two'
  'three'
]
param elementsToSkip int = 2
param testString string = 'one two three'
param charactersToSkip int = 4

output arrayOutput array = skip(testArray, elementsToSkip)
output stringOutput string = skip(testString, charactersToSkip)
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| arrayOutput | Array | ["three"] |
| stringOutput | String | two three |

## <a name="take"></a>take

`take(originalValue, numberToTake)`

Devuelve una matriz con el número especificado de elementos desde el inicio de la matriz, o una cadena con el número especificado de caracteres desde el inicio de la cadena.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| originalValue |Sí |matriz o cadena |La matriz o cadena de la que se van a tomar los elementos. |
| numberToTake |Sí |int |El número de elementos o caracteres que se van a tomar. Si este valor es 0 o un valor inferior, se devolverá una matriz o cadena vacía. Si es mayor que la longitud de la matriz o cadena especificada, se devuelven todos los elementos de la matriz o cadena. |

### <a name="return-value"></a>Valor devuelto

Una matriz o cadena.

### <a name="example"></a>Ejemplo

En el ejemplo siguiente se toma el número especificado de elementos de la matriz y de caracteres de la cadena.

```bicep
param testArray array = [
  'one'
  'two'
  'three'
]
param elementsToTake int = 2
param testString string = 'one two three'
param charactersToTake int = 2

output arrayOutput array = take(testArray, elementsToTake)
output stringOutput string = take(testString, charactersToTake)
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| arrayOutput | Array | ["one", "two"] |
| stringOutput | String | en |

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

En el ejemplo siguiente se muestra cómo utilizar la unión con matrices y objetos:

```bicep
param firstObject object = {
  'one': 'a'
  'two': 'b'
  'three': 'c1'
}

param secondObject object = {
  'three': 'c2'
  'four': 'd'
  'five': 'e'
}

param firstArray array = [
  'one'
  'two'
  'three'
]

param secondArray array = [
  'three'
  'four'
]

output objectOutput object = union(firstObject, secondObject)
output arrayOutput array = union(firstArray, secondArray)
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| objectOutput | Object | {"one": "a", "two": "b", "three": "c2", "four": "d", "five": "e"} |
| arrayOutput | Array | ["one", "two", "three", "four"] |

## <a name="next-steps"></a>Pasos siguientes

* Puede encontrar una descripción de las secciones de un archivo de Bicep en [Nociones sobre la estructura y la sintaxis de los archivos de Bicep](./file.md).
