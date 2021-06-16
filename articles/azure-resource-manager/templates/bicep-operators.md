---
title: Operadores de Bicep
description: Describe los operadores de Bicep disponibles para las implementaciones de Azure Resource Manager.
ms.topic: conceptual
ms.date: 04/15/2021
ms.openlocfilehash: 0838ebf6bc03f4237ef76e07f1eb6f25aa996fc0
ms.sourcegitcommit: 5c136a01bddfccb2cc9f7e7e7741e2cf2651ddbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111353551"
---
# <a name="bicep-operators"></a>Operadores de Bicep

En este artículo, se describen los operadores de Bicep que están disponibles cuando se crea una plantilla de Bicep y se utiliza Azure Resource Manager para implementar recursos. Los operadores se usan para calcular valores, comparar valores o evaluar condiciones. Hay tres tipos de operadores:

- [de comparación](#comparison)
- [lógicos](#logical)
- [numeric](#numeric)

Encerrar una expresión entre `(` y `)` permite invalidar la precedencia predeterminada del operador Bicep. Por ejemplo, la expresión x + y/z evalúa primero la división y, después, la suma. Sin embargo, la expresión (x + y)/z evalúa la suma primero y la división después.

## <a name="comparison"></a>De comparación

Los operadores de comparación comparan valores y devuelven `true` o `false`.

| Operator | Nombre | Descripción |
| ---- | ---- | ---- |
| `>=` | [Mayor que o igual a](bicep-operators-comparison.md#greater-than-or-equal-) | Determina si el primer valor es mayor o igual que el segundo. |
| `>`  | [Mayor que](bicep-operators-comparison.md#greater-than-) | Determina si el primer valor es mayor que el segundo. |
| `<=` | [Menor que o igual a](bicep-operators-comparison.md#less-than-or-equal-) | Determina si el primer valor es menor o igual que el segundo. |
| `<`  | [Menor que](bicep-operators-comparison.md#less-than-) | Determina si el primer valor es menor que el segundo. |
| `==` | [Es igual a](bicep-operators-comparison.md#equals-) | Determina si dos valores son iguales. |
| `!=` | [No es igual a](bicep-operators-comparison.md#not-equal-) | Determina si dos valores **no** son iguales. |
| `=~` | [Igual sin distinción de mayúsculas y minúsculas](bicep-operators-comparison.md#equal-case-insensitive-) | Determina si dos valores son iguales sin tener en cuenta la distinción entre mayúsculas y minúsculas. |
| `!~` | [No es igual sin distinción de mayúsculas y minúsculas](bicep-operators-comparison.md#not-equal-case-insensitive-) | Determina si dos valores **no** son iguales sin tener en cuenta la distinción entre mayúsculas y minúsculas. |

## <a name="logical"></a>Lógicos

Los operadores lógicos evalúan valores booleanos, devuelven valores que no son NULL o evalúan expresiones condicionales.

| Operator | Nombre | Descripción |
| ---- | ---- | ---- |
| `&&` | [Y](bicep-operators-logical.md#and-) | Devuelve `true` si todos los valores son TRUE. |
| `||`| [O](bicep-operators-logical.md#or-) | Devuelve `true` si alguno de los valores es TRUE. |
| `!` | [Not](bicep-operators-logical.md#not-) | Niega un valor booleano. |
| `??` | [Fusionar](bicep-operators-logical.md#coalesce-) | Devuelve el primer valor que no es NULL. |
| `?` `:` | [Expresión condicional](bicep-operators-logical.md#conditional-expression--) | Determina si una condición es TRUE o FALSE y devuelve un valor. |

## <a name="numeric"></a>Numeric

Los operadores numéricos usan enteros para realizar cálculos y devolver valores enteros.

| Operator | Nombre | Descripción |
| ---- | ---- | ---- |
| `*` | [Multiplicar](bicep-operators-numeric.md#multiply-) | Multiplica dos enteros. |
| `/` | [Dividir](bicep-operators-numeric.md#divide-) | Divide un entero por otro. |
| `%` | [Módulo](bicep-operators-numeric.md#modulo-) | Divide un entero por otro y devuelve el resto. |
| `+` | [Add (Agregar)](bicep-operators-numeric.md#add-) | Suma dos enteros. |
| `-` | [Restar](bicep-operators-numeric.md#subtract--) | Resta un entero de otro. |
| `-` | [Menos](bicep-operators-numeric.md#minus--) | Multiplica un entero por `-1`. |

> [!NOTE]
> "Restar" y "Menos" utilizan el mismo operador. La funcionalidad es diferente porque "Restar" utiliza dos operandos, mientras que "Menos" solo utiliza uno.

## <a name="next-steps"></a>Pasos siguientes

- Para crear un archivo de Bicep, consulte [Tutorial: Creación e implementación del primer archivo de Bicep de Azure Resource Manager](bicep-tutorial-create-first-bicep.md).
- Para obtener información sobre cómo resolver errores de tipos de Bicep, consulte [Función any para Bicep](template-functions-any.md).
- Para comparar la sintaxis de Bicep y JSON, consulte [Comparación de JSON y Bicep para plantillas](compare-template-syntax.md).
- Para ver ejemplos de las funciones de plantilla de Bicep y ARM, consulte [Funciones de plantilla de ARM](template-functions.md).
