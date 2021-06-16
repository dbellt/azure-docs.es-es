---
title: Operadores lógicos de Bicep
description: Se describen los operadores lógicos de Bicep que evalúan condiciones.
ms.topic: conceptual
ms.date: 04/07/2021
ms.openlocfilehash: 1eecbe589ecf08e32246dd97d137c60fa7b5078f
ms.sourcegitcommit: 5c136a01bddfccb2cc9f7e7e7741e2cf2651ddbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111353559"
---
# <a name="bicep-logical-operators"></a>Operadores lógicos de Bicep

Los operadores lógicos evalúan valores booleanos y expresiones condicionales o devuelven valores no NULL. Para ejecutar los ejemplos, use la CLI de Azure o Azure PowerShell para [implementar un archivo de Bicep](bicep-tutorial-create-first-bicep.md#deploy-bicep-file).

| Operator | Nombre |
| ---- | ---- |
| `&&` | [Y](#and-) |
| `||` | [O](#or-) |
| `!` | [Not](#not-) |
| `??` | [Coalesce](#coalesce-) |
| `?` `:` | [Expresión condicional](#conditional-expression--) |

## <a name="and-"></a>And &&

`operand1 && operand2`

Determina si ambos valores son verdaderos.

### <a name="operands"></a>Operandos

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `operand1` | boolean | Primer valor que se comprueba si es verdadero. |
| `operand2` | boolean | Segundo valor que se comprueba si es verdadero. |
| Más operandos | boolean | Se pueden incluir más operandos. |

### <a name="return-value"></a>Valor devuelto

`True` cuando ambos valores son verdaderos; de lo contrario, se devuelve `false`.

### <a name="example"></a>Ejemplo

Evalúa un conjunto de valores de parámetro y un conjunto de expresiones.

```bicep
param operand1 bool = true
param operand2 bool = true

output andResultParm bool = operand1 && operand2
output andResultExp bool = bool(10 >= 10) && bool(5 > 2)
```

Salida del ejemplo:

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `andResultParm` | boolean | true |
| `andResultExp` | boolean | true |

## <a name="or-"></a>Or ||

`operand1 || operand2`

Determina si alguno de los valores es verdadero.

### <a name="operands"></a>Operandos

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `operand1` | boolean | Primer valor que se comprueba si es verdadero. |
| `operand2` | boolean | Segundo valor que se comprueba si es verdadero. |
| Más operandos | boolean | Se pueden incluir más operandos. |

### <a name="return-value"></a>Valor devuelto

`True` cuando alguno de los valores es verdadero; de lo contrario, se devuelve `false`.

### <a name="example"></a>Ejemplo

Evalúa un conjunto de valores de parámetro y un conjunto de expresiones.

```bicep
param operand1 bool = true
param operand2 bool = false

output orResultParm bool = operand1 || operand2
output orResultExp bool = bool(10 >= 10) || bool(5 < 2)
```

Salida del ejemplo:

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `orResultParm` | boolean | true |
| `orResultExp` | boolean | true |

## <a name="not-"></a>Not !

`!boolValue`

Niega un valor booleano.

### <a name="operand"></a>Operando

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `boolValue` | boolean | Valor booleano que se niega. |

### <a name="return-value"></a>Valor devuelto

Niega el valor inicial y devuelve un valor booleano. Si el valor inicial es `true`, se devuelve `false`.

### <a name="example"></a>Ejemplo

El operador `not` niega un valor. Los valores se pueden incluir entre paréntesis.

```bicep
param initTrue bool = true
param initFalse bool = false

output startedTrue bool = !(initTrue)
output startedFalse bool = !initFalse
```

Salida del ejemplo:

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `startedTrue` | boolean | false |
| `startedFalse` | boolean | true |

## <a name="coalesce-"></a>Coalesce ??

`operand1 ?? operand2`

Devuelve el primer valor no nulo de los operandos.

### <a name="operands"></a>Operandos

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `operand1` | string, integer, boolean, object, array | Valor que se va a probar para `null`. |
| `operand2` | string, integer, boolean, object, array | Valor que se va a probar para `null`. |
| Más operandos | string, integer, boolean, object, array | Valor que se va a probar para `null`. |

### <a name="return-value"></a>Valor devuelto

Devuelve el primer valor no NULL. Las cadenas, matrices y objetos vacíos no son `null` y se devuelve un valor \<empty>.

### <a name="example"></a>Ejemplo

Las instrucciones de salida devuelven los valores no NULL. El tipo de salida debe coincidir con el tipo de la comparación o se genera un error.

```bicep
param myObject object = {
  'isnull1': null
  'isnull2': null
  'string': 'demoString'
  'emptystr': ''
  'integer': 10
  }

output nonNullStr string = myObject.isnull1 ?? myObject.string ?? myObject.isnull2
output nonNullInt int = myObject.isnull1 ?? myObject.integer ?? myObject.isnull2
output nonNullEmpty string = myObject.isnull1 ?? myObject.emptystr ?? myObject.string ?? myObject.isnull2
```

Salida del ejemplo:

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `nonNullStr` | string | demoString |
| `nonNullInt` | int | 10 |
| `nonNullEmpty` | string | \<empty> |

## <a name="conditional-expression--"></a>Expresión condicional ? :

`condition ? true-value : false-value`

Evalúa una condición y devuelve un valor si la condición es verdadera o falsa.

### <a name="operands"></a>Operandos

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `condition` | boolean | Condición que se debe evaluar como verdadera o falsa. |
| `true-value` | string, integer, boolean, object, array | Valor cuando la condición es verdadera. |
| `false-value` | string, integer, boolean, object, array | Valor cuando la condición es falsa. |

### <a name="example"></a>Ejemplo

En este ejemplo se evalúa la inicial de un parámetro y se devuelve un valor si la condición es verdadera o falsa.

```bicep
param initValue bool = true

output outValue string = initValue ? 'true value' : 'false value'
```

Salida del ejemplo:

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `outValue` | string | true value |

## <a name="next-steps"></a>Pasos siguientes

- Para crear un archivo de Bicep, consulte [Tutorial: Creación e implementación del primer archivo de Bicep de Azure Resource Manager](bicep-tutorial-create-first-bicep.md).
- Para obtener información sobre cómo resolver errores de tipos de Bicep, consulte [Función any para Bicep](template-functions-any.md).
- Para comparar la sintaxis de Bicep y JSON, consulte [Comparación de JSON y Bicep para plantillas](compare-template-syntax.md).
- Para ver ejemplos de las funciones de plantilla de Bicep y ARM, consulte [Funciones de plantilla de ARM](template-functions.md).
