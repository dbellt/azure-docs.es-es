---
title: 'Patrón: operadores lógicos en una definición de directiva'
description: Este patrón de Azure Policy proporciona un ejemplo de cómo usar los operadores lógicos en una definición de directiva.
ms.date: 03/31/2021
ms.topic: sample
ms.openlocfilehash: 3daf5e5873566b1335178d50a4589ec7a6e1971c
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108752280"
---
# <a name="azure-policy-pattern-logical-operators"></a>Patrón de Azure Policy: operadores lógicos

Una definición de directiva puede contener varias instrucciones condicionales. Es posible que necesite que cada instrucción sea verdadera o que solo necesite que algunas de ellas sean verdaderas. Para satisfacer estas necesidades, el lenguaje tiene [operadores lógicos](../concepts/definition-structure.md#logical-operators) para **not**, **allOf** y **anyOf**. Son opcionales y se pueden anidar para crear escenarios complejos.

## <a name="sample-1-one-logical-operator"></a>Muestra 1: un operador lógico

Esta definición de directiva evalúa las cuentas de [Azure Cosmos DB](../../../cosmos-db/introduction.md) para ver si están configuradas las conmutaciones por error automáticas y varias ubicaciones de escritura. Cuando no lo están, se desencadena el efecto [audit](../concepts/effects.md#audit) y crea una entrada de registro cuando se crea o actualiza el recurso no compatible.

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-1.json":::

### <a name="sample-1-explanation"></a>Muestra 1: Explicación

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-1.json" range="6-22" highlight="3":::

El bloque **policyRule.if** usa un solo operador **allOf** para asegurarse de que las tres condiciones son verdaderas.
Solo cuando todas estas condiciones se evalúan como verdaderas, se desencadena el efecto **audit**.

## <a name="sample-2-multiple-logical-operators"></a>Ejemplo 2: varios operadores lógicos

Esta definición de directiva evalúa los recursos según un patrón de nomenclatura. Si un recurso no coincide, es [denegado](../concepts/effects.md#deny).

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-2.json":::

### <a name="sample-2-explanation"></a>Ejemplo 2: Explicación

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-2.json" range="7-21" highlight="2,3,9":::

Este bloque **policyRule.if** también incluye un único operador **allOf**, pero cada condición se encapsula con el operador lógico **not**. El condicional dentro del operador lógico **not** se evalúa primero y, a continuación, se evalúa el operador **not** para determinar si la cláusula completa es verdadera o falsa. Si ambos operadores lógicos **not** se evalúan como verdaderos, se desencadena el efecto de la directiva.

## <a name="sample-3-combining-logical-operators"></a>Ejemplo 3: Combinación de operadores lógicos

Esta definición de directiva evalúa las cuentas de [Spring en Azure](/azure/developer/java/spring-framework) para ver si el seguimiento no está habilitado o no se encuentra en un estado correcto.

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-3.json":::

### <a name="sample-3-explanation"></a>Ejemplo 3: Explicación

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-3.json" range="6-28" highlight="3,8":::

Este bloque **policyRule.if** incluye los operadores lógicos **allOf** y **anyOf**. El operador lógico **anyOf** se evalúa como True siempre que se cumpla una condición incluida. Dado que _type_ está en el núcleo de **allOf**, siempre se evalúa como True. Si _type_ y una de las condiciones de **anyOf** son True, se desencadena el efecto de la directiva.

## <a name="next-steps"></a>Pasos siguientes

- Consulte otros [patrones y definiciones integradas](./index.md).
- Revise la [estructura de definición de Azure Policy](../concepts/definition-structure.md).
- Vea la [Descripción de los efectos de directivas](../concepts/effects.md).
