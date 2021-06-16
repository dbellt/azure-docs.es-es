---
title: Error de tamaño de trabajo superado
description: Describe cómo solucionar errores cuando el tamaño o la plantilla de trabajo son demasiado grandes.
ms.topic: troubleshooting
ms.date: 03/23/2021
ms.openlocfilehash: 5d7333e8faf44b5c2c872448960eff34ff61daf2
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111963260"
---
# <a name="resolve-errors-for-job-size-exceeded"></a>Resolución de errores de tamaño de trabajo superado

En este artículo se describe cómo resolver los errores **JobSizeExceededException** y **DeploymentJobSizeExceededException**.

## <a name="symptom"></a>Síntoma

Al implementar una plantilla, recibe un error que indica que la implementación ha superado los límites.

## <a name="cause"></a>Causa

Este error aparece cuando la implementación supera uno de los límites permitidos. Normalmente, este error aparece cuando la plantilla o el trabajo que ejecuta la implementación son demasiado grandes.

El trabajo de implementación no puede superar 1 MB. El trabajo incluye metadatos sobre la solicitud. En el caso de las plantillas grandes, los metadatos combinados con la plantilla pueden superar el tamaño permitido para un trabajo.

La plantilla no puede superar los 4 MB. El límite de 4 MB se aplica al estado final de la plantilla después de su ampliación para las definiciones de recursos que utilizan la [copia](copy-resources.md) para crear muchas instancias. El estado final también incluye los valores resueltos para variables y parámetros.

Otros límites de la plantilla son:

* 256 parámetros
* 256 variables
* 800 recursos (incluido el recuento de copia)
* 64 valores de salida
* 24 576 caracteres en una expresión de plantilla

Al usar bucles de copia para implementar el recurso, no use el nombre del bucle como una dependencia:

```json
dependsOn: [ "nicLoop" ]
```

En su lugar, use la instancia del recurso del bucle del que depende. Por ejemplo:

```json
dependsOn: [
    "[resourceId('Microsoft.Network/networkInterfaces', concat('nic-', copyIndex()))]"
]
```

## <a name="solution-1---simplify-template"></a>Solución 1: simplificar la plantilla

La primera opción es simplificar la plantilla. Esta opción funciona cuando la plantilla implementa muchos tipos de recursos diferentes. Considere la posibilidad de dividir la plantilla en [plantillas vinculadas](linked-templates.md). Divida los tipos de recursos en grupos lógicos y agregue una plantilla vinculada para cada grupo. Por ejemplo, si necesita implementar una gran cantidad de recursos de red, puede trasladar esos recursos a una plantilla vinculada.

Puede establecer otros recursos como dependientes de la plantilla vinculada y [obtener valores de la salida de la plantilla vinculada](linked-templates.md#get-values-from-linked-template).

## <a name="solution-2---reduce-name-size"></a>Solución 2: reducir el tamaño de los nombres

Intente acortar la longitud de los nombres que utiliza para [parámetros](./parameters.md), [variables](./variables.md) y [salidas](./outputs.md). Cuando estos valores se repiten a través de bucles de copia, un nombre largo se multiplica varias veces.