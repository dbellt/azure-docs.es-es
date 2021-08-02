---
title: 'Funciones de Bicep: numéricas'
description: Se describen las funciones que se usan en un archivo de Bicep para trabajar con números.
author: mumian
ms.author: jgao
ms.topic: conceptual
ms.date: 06/01/2021
ms.openlocfilehash: 2eec27d0317b6cfae4969244d069be6881566942
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "111027109"
---
# <a name="numeric-functions-for-bicep"></a>Funciones numéricas de Bicep

Resource Manager ofrece las siguientes funciones para trabajar con números enteros en el archivo de Bicep:

* [int](#int)
* [max](#max)
* [min](#min)

Algunas de las funciones numéricas JSON de Azure Resource Manager se reemplazan por [operadores numéricos de Bicep](./operators-numeric.md).

## <a name="int"></a>int

`int(valueToConvert)`

Convierte el valor especificado en un entero.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| valueToConvert |Sí |cadena o entero |Valor que se convierte en entero. |

### <a name="return-value"></a>Valor devuelto

Un entero del valor convertido.

### <a name="example"></a>Ejemplo

En el siguiente ejemplo se convierte el valor del parámetro proporcionado por el usuario en entero.

```bicep
param stringToConvert string = '4'

output inResult int = int(stringToConvert)
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| intResult | Int | 4 |

## <a name="max"></a>max

`max (arg1)`

Devuelve el valor máximo de una matriz de enteros o una lista separada por comas de enteros.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| arg1 |Sí |matriz de enteros, o lista separada por comas de enteros |La colección para obtener el valor máximo. |

### <a name="return-value"></a>Valor devuelto

Un entero que representa el valor máximo de la colección.

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

output arrayOutPut int = max(arrayToTest)
output intOutput int = max(0,3,2,5,4)
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| arrayOutput | Int | 5 |
| intOutput | Int | 5 |

## <a name="min"></a>Min

`min (arg1)`

Devuelve el valor mínimo de una matriz de enteros o una lista separada por comas de enteros.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| arg1 |Sí |matriz de enteros, o lista separada por comas de enteros |La colección para obtener el valor mínimo. |

### <a name="return-value"></a>Valor devuelto

Un entero que representa el valor mínimo de la colección.

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

output arrayOutPut int = min(arrayToTest)
output intOutput int = min(0,3,2,5,4)
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| arrayOutput | Int | 0 |
| intOutput | Int | 0 |

## <a name="next-steps"></a>Pasos siguientes

* Puede encontrar una descripción de las secciones de un archivo de Bicep en [Nociones sobre la estructura y la sintaxis de los archivos de Bicep](./file.md).
