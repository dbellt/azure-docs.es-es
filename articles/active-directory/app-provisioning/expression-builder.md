---
title: Descripción del funcionamiento del generador de expresiones con el aprovisionamiento de aplicaciones en Azure Active Directory
description: Describa el funcionamiento del generador de expresiones con el aprovisionamiento de aplicaciones en Azure Active Directory.
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: conceptual
ms.workload: identity
ms.date: 06/02/2021
ms.author: kenwith
ms.reviewer: arvinh
ms.openlocfilehash: 336005a94d4403d6a9e3fb125001b78a68179aa4
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111962087"
---
# <a name="understand-how-expression-builder-in-application-provisioning-works"></a>Descripción del funcionamiento del generador de expresiones con el aprovisionamiento de aplicaciones

Puede usar [expresiones](functions-for-customizing-application-data.md) para [asignar atributos](./customize-application-attributes.md). Anteriormente, tenía que crear estas expresiones manualmente y escribirlas en el cuadro de expresión. El generador de expresiones es una herramienta que puede usar para ayudarle a crear expresiones.

:::image type="content" source="media/expression-builder/expression-builder.png" alt-text="Página del generador de expresiones predeterminada antes de seleccionar una función." lightbox="media/expression-builder/expression-builder.png":::

En relación con la creación de expresiones, consulte [Referencia para escribir expresiones para las asignaciones de atributos](functions-for-customizing-application-data.md). 

## <a name="finding-the-expression-builder"></a>Búsqueda del generador de expresiones

En el aprovisionamiento de aplicaciones, se usan expresiones para las asignaciones de atributos. Para acceder al generador de expresiones en la página de asignación de atributos, seleccione **Mostrar opciones avanzadas** y, a continuación, seleccione **Generador de expresiones**.

:::image type="content" source="media/expression-builder/accessing-expression-builder.png" alt-text="La casilla para mostrar la configuración avanzada está activada y se muestra un vínculo que dice generador de expresiones" lightbox="media/expression-builder/accessing-expression-builder.png":::

## <a name="using-expression-builder"></a>Uso del generador de expresiones

Para usar el generador de expresiones, seleccione una función y un atributo y, a continuación, escriba un sufijo si es necesario. A continuación, seleccione **Agregar expresión** para agregar la expresión al cuadro del código. Para obtener más información sobre las funciones disponibles y cómo usarlas, consulte [Referencia para escribir expresiones para las asignaciones de atributos](functions-for-customizing-application-data.md).

Pruebe la expresión buscando un usuario o proporcionando valores y seleccionando **Expresión de prueba**. La salida de la prueba de expresión aparecerá en el cuadro **Ver la salida de la expresión**.

Cuando esté satisfecho con la expresión, muévala a una asignación de atributos. Cópiela y péguela en el cuadro de expresión para la asignación de atributos en la que trabaja.

## <a name="known-limitations"></a>Restricciones conocidas
* Los atributos de extensión no están disponibles para su selección en el generador de expresiones. Sin embargo, los atributos de extensión se pueden usar en la expresión de asignación de atributos. 

## <a name="next-steps"></a>Pasos siguientes

[Referencia para escribir expresiones para las asignaciones de atributos](functions-for-customizing-application-data.md)