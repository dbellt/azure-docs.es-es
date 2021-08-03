---
title: Operadores numéricos de Bicep
description: Se describen los operadores numéricos de Bicep que calculan valores.
author: mumian
ms.author: jgao
ms.topic: conceptual
ms.date: 06/01/2021
ms.openlocfilehash: 8d863d303ef0ed72b82bd65238f43a2d902c6003
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "111027229"
---
# <a name="bicep-numeric-operators"></a>Operadores numéricos de Bicep

Los operadores numéricos usan números enteros para realizar cálculos y devolver valores también enteros. Para ejecutar los ejemplos, use la CLI de Azure o Azure PowerShell para [implementar un archivo de Bicep](./quickstart-create-bicep-use-visual-studio-code.md#deploy-the-bicep-file).

| Operator | Nombre |
| ---- | ---- |
| `*` | [Multiplicar](#multiply-) |
| `/` | [Dividir](#divide-) |
| `%` | [Módulo](#modulo-) |
| `+` | [Add (Agregar)](#add-) |
| `-` | [Restar](#subtract--) |
| `-` | [Menos](#minus--) |

> [!NOTE]
> Restar y Menos usan el mismo operador. La funcionalidad es diferente, ya que Restar usa dos operandos y Menos, uno.

## <a name="multiply-"></a>Multiplicar (*)

`operand1 * operand2`

Multiplica dos enteros.

### <a name="operands"></a>Operandos

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `operand1`  | integer | Número que se va a multiplicar. |
| `operand2`  | integer  | Multiplicador del número. |

### <a name="return-value"></a>Valor devuelto

La multiplicación devuelve el producto en forma de entero.

### <a name="example"></a>Ejemplo

Se multiplican dos enteros y se devuelve el producto.

```bicep
param firstInt int = 5
param secondInt int = 2

output product int = firstInt * secondInt
```

Salida del ejemplo:

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `product` | integer | 10 |

## <a name="divide-"></a>Dividir (/)

`operand1 / operand2`

Divide un entero entre otro.

### <a name="operands"></a>Operandos

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `operand1` | integer | Entero dividido. |
| `operand2` | integer | Entero que se usa para la división. No puede ser cero. |

### <a name="return-value"></a>Valor devuelto

La división devuelve el cociente en forma de entero.

### <a name="example"></a>Ejemplo

Se dividen dos enteros y se devuelve el cociente.

```bicep
param firstInt int = 10
param secondInt int = 2

output quotient int = firstInt / secondInt
```

Salida del ejemplo:

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `quotient` | integer | 5 |

## <a name="modulo-"></a>Módulo (%)

`operand1 % operand2`

Divide un entero entre otro y devuelve el resto.

### <a name="operands"></a>Operandos

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `operand1`  | integer  | Entero dividido. |
| `operand2`  | integer  | Entero que se usa para la división. No puede ser 0. |

### <a name="return-value"></a>Valor devuelto

El resto se devuelve en forma de entero. Si la división no genera un resto, se devuelve “0”.

### <a name="example"></a>Ejemplo

Se dividen dos pares de enteros y se devuelven los restos.

```bicep
param firstInt int = 10
param secondInt int = 3

param thirdInt int = 8
param fourthInt int = 4

output remainder int = firstInt % secondInt
output zeroRemainder int = thirdInt % fourthInt
```

Salida del ejemplo:

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `remainder` | integer | 1 |
| `zeroRemainder` | integer | 0 |

## <a name="add-"></a>Sumar (+)

`operand1 + operand2`

Suma dos enteros.

### <a name="operands"></a>Operandos

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `operand1` | integer | Número que se va a sumar. |
| `operand2` | integer | Número sumado a otro. |

### <a name="return-value"></a>Valor devuelto

La suma devuelve el resultado en forma de entero.

### <a name="example"></a>Ejemplo

Se suman dos enteros y se devuelve el resultado.

```bicep
param firstInt int = 10
param secondInt int = 2

output sum int = firstInt + secondInt
```

Salida del ejemplo:

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `sum` | integer | 12 |

## <a name="subtract--"></a>Restar (–)

`operand1 - operand2`

Resta un entero de otro.

### <a name="operands"></a>Operandos

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `operand1` | integer | Número mayor del que se resta. |
| `operand2` | integer | Número que se resta del mayor. |

### <a name="return-value"></a>Valor devuelto

La resta devuelve la diferencia en forma de entero.

### <a name="example"></a>Ejemplo

Se resta un entero y se devuelve la diferencia.

```bicep
param firstInt int = 10
param secondInt int = 4

output difference int = firstInt - secondInt
```

Salida del ejemplo:

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `difference` | integer | 6 |

## <a name="minus--"></a>Menos (–)

`-integerValue`

Multiplica un entero por `-1`.

### <a name="operand"></a>Operando

| Operando | Tipo | Descripción |
| ---- | ---- | ---- |
| `integerValue` | integer | Entero multiplicado por `-1`. |

### <a name="return-value"></a>Valor devuelto

Un entero se multiplica por `-1`. Un entero positivo devuelve otro negativo, mientras que un entero negativo devuelve otro positivo. Los valores se pueden incluir entre paréntesis.

### <a name="example"></a>Ejemplo

```bicep
param posInt int = 10
param negInt int = -20

output startedPositive int = -posInt
output startedNegative int = -(negInt)
```

Salida del ejemplo:

| Nombre | Tipo | Value |
| ---- | ---- | ---- |
| `startedPositive` | integer | -10 |
| `startedNegative` | integer | 20 |

## <a name="next-steps"></a>Pasos siguientes

- Para crear un archivo de Bicep, consulte [Inicio rápido: Creación de plantillas de archivos de Bicep con Visual Studio Code](./quickstart-create-bicep-use-visual-studio-code.md).
- Para obtener información sobre cómo resolver errores de tipos de Bicep, consulte [Función any para Bicep](./bicep-functions-any.md).
- Para comparar la sintaxis de Bicep y JSON, consulte [Comparación de JSON y Bicep para plantillas](./compare-template-syntax.md).
- Para ejemplos de funciones de Bicep, consulte [Funciones de Bicep](./bicep-functions.md).
