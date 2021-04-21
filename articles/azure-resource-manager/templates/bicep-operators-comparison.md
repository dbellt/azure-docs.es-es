---
title: Operadores de comparación de Bicep
description: Describe los operadores de comparación de Bicep, que sirven para comparar valores.
ms.topic: conceptual
ms.date: 04/07/2021
ms.openlocfilehash: c382ef355131d271d9f4fa4a978a807b9aac1e4f
ms.sourcegitcommit: c3739cb161a6f39a9c3d1666ba5ee946e62a7ac3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107211256"
---
# <a name="bicep-comparison-operators"></a>Operadores de comparación de Bicep

Los operadores de comparación comparan valores y devuelven `true` o `false`. Si desea ejecutar los ejemplos, use la CLI de Azure o Azure PowerShell para [implementar un archivo de Bicep](bicep-tutorial-create-first-bicep.md#deploy-bicep-file).

| Operator | Nombre |
| ---- | ---- |
| `>=` | [Mayor que o igual a](#greater-than-or-equal-) |
| `>`  | [Mayor que](#greater-than-) |
| `<=` | [Menor que o igual a](#less-than-or-equal-) |
| `<`  | [Menor que](#less-than-) |
| `==` | [Es igual a](#equals-) |
| `!=` | [No igual a](#not-equal-) |
| `=~` | [Igual sin distinción de mayúsculas y minúsculas](#equal-case-insensitive-) |
| `!~` | [No es igual sin distinción de mayúsculas y minúsculas](#not-equal-case-insensitive-) |

## <a name="greater-than-or-equal-"></a>Mayor o igual que >=

`operand1 >= operand2`

Determina si el primer valor es mayor o igual que el segundo.

### <a name="operands"></a>Operandos

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `operand1` | entero, cadena | Primer valor de la comparación. |
| `operand2` | entero, cadena | Segundo valor de la comparación. |

### <a name="return-value"></a>Valor devuelto

Si el primer valor es mayor o igual que el segundo, devuelve `true`. En caso contrario, devuelve `false`.

### <a name="example"></a>Ejemplo

Comparación de un par de enteros y un par de cadenas.

```bicep
param firstInt int = 10
param secondInt int = 5

param firstString string = 'A'
param secondString string = 'A'

output intGtE bool = firstInt >= secondInt
output stringGtE bool = firstString >= secondString
```

Salida del ejemplo:

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `intGtE` | boolean | true |
| `stringGtE` | boolean | true |

## <a name="greater-than-"></a>Mayor que >

`operand1 > operand2`

Determina si el primer valor es mayor que el segundo.

### <a name="operands"></a>Operandos

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `operand1` | entero, cadena | Primer valor de la comparación. |
| `operand2` | entero, cadena | Segundo valor de la comparación. |

### <a name="return-value"></a>Valor devuelto

Si el primer valor es mayor que el segundo, devuelve `true`. En caso contrario, devuelve `false`.

### <a name="example"></a>Ejemplo

Comparación de un par de enteros y un par de cadenas.

```bicep
param firstInt int = 10
param secondInt int = 5

param firstString string = 'bend'
param secondString string = 'band'

output intGt bool = firstInt > secondInt
output stringGt bool = firstString > secondString
```

Salida del ejemplo:

La **e** de **bend** hace que la primera cadena sea mayor.

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `intGt` | boolean | true |
| `stringGt` | boolean | true |

## <a name="less-than-or-equal-"></a>Menor o igual que <=

`operand1 <= operand2`

Determina si el primer valor es menor o igual que el segundo.

### <a name="operands"></a>Operandos

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `operand1` | entero, cadena | Primer valor de la comparación. |
| `operand2` | entero, cadena | Segundo valor de la comparación. |

### <a name="return-value"></a>Valor devuelto

Si el primer valor es menor o igual que el segundo, devuelve `true`. En caso contrario, devuelve `false`.

### <a name="example"></a>Ejemplo

Comparación de un par de enteros y un par de cadenas.

```bicep
param firstInt int = 5
param secondInt int = 10

param firstString string = 'demo'
param secondString string = 'demo'

output intLtE bool = firstInt <= secondInt
output stringLtE bool = firstString <= secondString
```

Salida del ejemplo:

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `intLtE` | boolean | true |
| `stringLtE` | boolean | true |

## <a name="less-than-"></a>Menor que <

`operand1 < operand2`

Determina si el primer valor es menor que el segundo.

### <a name="operands"></a>Operandos

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `operand1` | entero, cadena | Primer valor de la comparación. |
| `operand2` | entero, cadena | Segundo valor de la comparación. |

### <a name="return-value"></a>Valor devuelto

Si el primer valor es menor que el segundo, devuelve `true`. En caso contrario, devuelve `false`.

### <a name="example"></a>Ejemplo

Comparación de un par de enteros y un par de cadenas.

```bicep
param firstInt int = 5
param secondInt int = 10

param firstString string = 'demo'
param secondString string = 'Demo'

output intLt bool = firstInt < secondInt
output stringLt bool = firstString < secondString
```

Salida del ejemplo:

La cadena es `true`, ya que el valor de las minúsculas es inferior al de las mayúsculas.

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `intLt` | boolean | true |
| `stringLt` | boolean | true |

## <a name="equals-"></a>Es igual a ==

`operand1 == operand2`

Determina si los valores son iguales.

### <a name="operands"></a>Operandos

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `operand1` | cadena, entero, booleano, objeto, matriz | Primer valor de la comparación. |
| `operand2` | cadena, entero, booleano, objeto, matriz | Segundo valor de la comparación. |

### <a name="return-value"></a>Valor devuelto

Si los operandos son iguales, devuelve `true`. Si los operandos son diferentes, devuelve `false`.

### <a name="example"></a>Ejemplo

Comparación de pares de enteros, cadenas y booleanos.

```bicep
param firstInt int = 5
param secondInt int = 5

param firstString string = 'demo'
param secondString string = 'demo'

param firstBool bool = true
param secondBool bool = true

output intEqual bool = firstInt == secondInt
output stringEqual bool = firstString == secondString
output boolEqual bool = firstBool == secondBool
```

Salida del ejemplo:

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `intEqual` | boolean | true |
| `stringEqual` | boolean | true |
| `boolEqual` | boolean | true |

## <a name="not-equal-"></a>No es igual !=

`operand1 != operand2`

Determina si dos valores **no** son iguales.

### <a name="operands"></a>Operandos

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `operand1` | cadena, entero, booleano, objeto, matriz | Primer valor de la comparación. |
| `operand2` | cadena, entero, booleano, objeto, matriz | Segundo valor de la comparación. |

### <a name="return-value"></a>Valor devuelto

Si los operandos **no** son iguales, devuelve `true`. Si los operandos son iguales, devuelve `false`.

### <a name="example"></a>Ejemplo

Comparación de pares de enteros, cadenas y booleanos.

```bicep
param firstInt int = 10
param secondInt int = 5

param firstString string = 'demo'
param secondString string = 'test'

param firstBool bool = false
param secondBool bool = true

output intNotEqual bool = firstInt != secondInt
output stringNotEqual bool = firstString != secondString
output boolNotEqual bool = firstBool != secondBool
```

Salida del ejemplo:

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `intNotEqual` | boolean | true |
| `stringNotEqual` | boolean | true |
| `boolNotEqual` | boolean | true |

## <a name="equal-case-insensitive-"></a>Igual sin distinción de mayúsculas y minúsculas =~

`operand1 =~ operand2`

Determina si los dos valores son iguales sin tener en cuenta la distinción entre mayúsculas y minúsculas.

### <a name="operands"></a>Operandos

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `operand1`  | string | Primera cadena de la comparación. |
| `operand2`  | string | Segunda cadena de la comparación. |

### <a name="return-value"></a>Valor devuelto

Si las cadenas son iguales, devuelve `true`. En caso contrario, devuelve `false`.

### <a name="example"></a>Ejemplo

Compara cadenas que utilizan letras mayúsculas y minúsculas mezcladas.

```bicep
param firstString string = 'demo'
param secondString string = 'DEMO'

param thirdString string = 'demo'
param fourthString string = 'TEST'

output strEqual1 bool = firstString =~ secondString
output strEqual2 bool = thirdString =~ fourthString
```

Salida del ejemplo:

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `strEqual1` | boolean | true |
| `strEqual2` | boolean | false |

## <a name="not-equal-case-insensitive-"></a>No es igual sin distinción de mayúsculas y minúsculas !~

`operand1 !~ operand2`

Determina si los dos valores **no** son iguales sin tener en cuenta la distinción de mayúsculas y minúsculas.

### <a name="operands"></a>Operandos

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `operand1` | string | Primera cadena de la comparación. |
| `operand2` | string | Segunda cadena de la comparación. |

### <a name="return-value"></a>Valor devuelto

Si las cadenas no **son** iguales, devuelve `true`. En caso contrario, devuelve `false`.

### <a name="example"></a>Ejemplo

Compara cadenas que utilizan letras mayúsculas y minúsculas mezcladas.

```bicep
param firstString string = 'demo'
param secondString string = 'TEST'

param thirdString string = 'demo'
param fourthString string = 'DeMo'

output strNotEqual1 bool = firstString !~ secondString
output strEqual2 bool = thirdString !~ fourthString
```

Salida del ejemplo:

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `strNotEqual1` | boolean | true |
| `strNotEqual2` | boolean | false |

## <a name="next-steps"></a>Pasos siguientes

- Para crear un archivo de Bicep, consulte [Tutorial: Creación e implementación del primer archivo Bicep de Azure Resource Manager](bicep-tutorial-create-first-bicep.md).
- Para más información acerca de cómo resolver errores de tipos de Bicep, consulte [Función any para Bicep](template-functions-any.md).
- Para comparar la sintaxis de Bicep y JSON, consulte [Comparación de JSON y Bicep para plantillas](compare-template-syntax.md).
- Para ver funciones de plantillas de Bicep y ARM, consulte [Funciones de plantilla de ARM](template-functions.md).
