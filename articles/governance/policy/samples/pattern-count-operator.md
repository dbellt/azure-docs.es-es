---
title: 'Patrón: operador count en una definición de directiva'
description: Este patrón de Azure Policy proporciona un ejemplo de cómo usar el operador count en una definición de directiva.
ms.date: 03/31/2021
ms.topic: sample
ms.openlocfilehash: 4b4a2615491632caf1a7abddd1b6ace96ccc608d
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108752532"
---
# <a name="azure-policy-pattern-the-count-operator"></a>Patrón de Azure Policy: operador count

El operador [count](../concepts/definition-structure.md#count) evalúa los miembros de un \[\*\] alias.

## <a name="sample-policy-definition"></a>Definición de directiva de ejemplo

Esta definición de directiva [audita](../concepts/effects.md#audit) los grupos de seguridad de red configurados para permitir el tráfico del Protocolo de escritorio remoto (RDP) entrante.

:::code language="json" source="~/policy-templates/patterns/pattern-count-operator.json":::

### <a name="explanation"></a>Explicación

Los componentes principales del operador **count** son _field_, _where_ y la condición. Cada uno de ellos se resalta en el fragmento de código siguiente.

- _field_ indica a count de qué [alias](../concepts/definition-structure.md#aliases) debe evaluar los miembros. Aquí, observamos la _matriz_ de alias **securityRules\[\*\]** del grupo de seguridad de red.
- _where_ utiliza el lenguaje de directivas para definir qué miembros de la _matriz_ cumplen los criterios. En este ejemplo, el operador lógico **allOf** agrupa tres evaluaciones de condiciones diferentes de las propiedades de la _matriz_ de alias: _direction_, _access_ y _destinationPortRange_.
- La condición de count en este ejemplo es **greater** (mayor que). Count se evalúa como verdadero cuando uno o más miembros de la _matriz_ de alias coincide con la cláusula _where_.

:::code language="json" source="~/policy-templates/patterns/pattern-count-operator.json" range="12-32" highlight="3,4,20":::

## <a name="next-steps"></a>Pasos siguientes

- Consulte otros [patrones y definiciones integradas](./index.md).
- Revise la [estructura de definición de Azure Policy](../concepts/definition-structure.md).
- Vea la [Descripción de los efectos de directivas](../concepts/effects.md).
