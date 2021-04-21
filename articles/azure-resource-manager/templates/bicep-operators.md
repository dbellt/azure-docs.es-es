---
title: Operadores de Bicep
description: Describe los operadores de Bicep disponibles para las implementaciones de Azure Resource Manager.
ms.topic: conceptual
ms.date: 04/07/2021
ms.openlocfilehash: 4bf1005a11b1dcfea9f4b28d6bd3fa7c33e3278f
ms.sourcegitcommit: c3739cb161a6f39a9c3d1666ba5ee946e62a7ac3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107211271"
---
# <a name="bicep-operators"></a>Operadores de Bicep

En este artículo, se describen los operadores de Bicep que están disponibles cuando se crea una plantilla de Bicep y se utiliza Azure Resource Manager para implementar recursos. Los operadores se usan para calcular valores, comparar valores o evaluar condiciones. Hay tres tipos de operadores de Bicep: operadores de [comparación](#comparison), operadores [lógicos](#logical) y operadores [numéricos](#numeric).

## <a name="comparison"></a>De comparación

Los operadores de comparación comparan valores y devuelven `true` o `false`.

| Operator | Nombre | Descripción |
| ---- | ---- | ---- |
| `>=` | [Mayor que o igual a](bicep-operators-comparison.md#greater-than-or-equal-) | Determina si el primer valor es mayor o igual que el segundo. |
| `>`  | [Mayor que](bicep-operators-comparison.md#greater-than-) | Determina si el primer valor es mayor que el segundo. |
| `<=` | [Menor que o igual a](bicep-operators-comparison.md#less-than-or-equal-) | Determina si el primer valor es menor o igual que el segundo. |
| `<`  | [Menor que](bicep-operators-comparison.md#less-than-) | Determina si el primer valor es menor que el segundo. |
| `==` | [Es igual a](bicep-operators-comparison.md#equals-) | Determina si dos valores son iguales. |
| `!=` | [No igual a](bicep-operators-comparison.md#not-equal-) | Determina si dos valores **no** son iguales. |
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

- Para crear un archivo de Bicep, consulte [Tutorial: Creación e implementación del primer archivo Bicep de Azure Resource Manager](bicep-tutorial-create-first-bicep.md).
- Para más información acerca de cómo resolver errores de tipos de Bicep, consulte [Función any para Bicep](template-functions-any.md).
- Para comparar la sintaxis de Bicep y JSON, consulte [Comparación de JSON y Bicep para plantillas](compare-template-syntax.md).
- Para ver funciones de plantillas de Bicep y ARM, consulte [Funciones de plantilla de ARM](template-functions.md).
