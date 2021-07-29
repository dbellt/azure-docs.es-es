---
title: Definición de varias instancias de una variable en Bicep
description: Use el bucle de variables Bicep para iterar al crear una variable.
author: mumian
ms.author: jgao
ms.topic: conceptual
ms.date: 06/01/2021
ms.openlocfilehash: be97b88156600121deb1b870940a1a40af84fbfc
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "111026741"
---
# <a name="variable-iteration-in-bicep"></a>Iteración de variables en Bicep

En este artículo, se explica cómo puede crear varios valores para una variable en el archivo Bicep. Puede agregar un bucle a la sección `variables` y establecer dinámicamente el número de elementos de una variable durante la implementación. También evita repetir la sintaxis en el archivo Bicep.

También puede utilizar el elemento copy con [recursos](loop-resources.md), [propiedades de un recurso](loop-properties.md) y [salidas](loop-outputs.md).

## <a name="syntax"></a>Sintaxis

Los bucles se pueden usar para declarar varias variables mediante lo siguiente:

- Iteración en una matriz.

  ```bicep
  var <variable-name> = [for <item> in <collection>: {
    <properties>
  }]

  ```

- Iteración en los elementos de una matriz.

  ```bicep
  var <variable-name> = [for <item>, <index> in <collection>: {
    <properties>
  }]
  ```

- Uso de un índice de bucle.

  ```bicep
  var <variable-name> = [for <index> in range(<start>, <stop>): {
    <properties>
  }]
  ```

## <a name="loop-limits"></a>Límites del bucle

Las iteraciones de bucle del archivo Bicep no pueden ser un número negativo ni superar las 800 iteraciones. Para implementar archivos Bicep, instale la versión más reciente de las [herramientas de Bicep](install.md).

## <a name="variable-iteration"></a>Iteración variable

En el ejemplo siguiente, se muestra cómo se crea una matriz de valores de cadena:

```bicep
param itemCount int = 5

var stringArray = [for i in range(0, itemCount): 'item${(i + 1)}']

output arrayResult array = stringArray
```

La salida devuelve una matriz con los valores siguientes:

```json
[
  "item1",
  "item2",
  "item3",
  "item4",
  "item5"
]
```

En el ejemplo siguiente, se muestra cómo se crea una matriz de objetos con tres propiedades: `name`, `diskSizeGB` y `diskIndex`.

```bicep
param itemCount int = 5

var objectArray = [for i in range(0, itemCount): {
  name: 'myDataDisk${(i + 1)}'
  diskSizeGB: '1'
  diskIndex: i
}]

output arrayResult array = objectArray
```

La salida devuelve una matriz con los valores siguientes:

```json
[
  {
    "name": "myDataDisk1",
    "diskSizeGB": "1",
    "diskIndex": 0
  },
  {
    "name": "myDataDisk2",
    "diskSizeGB": "1",
    "diskIndex": 1
  },
  {
    "name": "myDataDisk3",
    "diskSizeGB": "1",
    "diskIndex": 2
  },
  {
    "name": "myDataDisk4",
    "diskSizeGB": "1",
    "diskIndex": 3
  },
  {
    "name": "myDataDisk5",
    "diskSizeGB": "1",
    "diskIndex": 4
  }
]
```

## <a name="example-templates"></a>Plantillas de ejemplo

En los ejemplos siguientes, se muestran escenarios comunes en los que se crean varios valores para una variable.

|Plantilla  |Descripción  |
|---------|---------|
|[Variables de bucle](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/loopvariables.bicep) | Muestra cómo iterar en variables. |
|[Varias reglas de seguridad](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/multiplesecurityrules.bicep) |Implementa varias reglas de seguridad en un grupo de seguridad de red. Crea las reglas de seguridad a partir de un parámetro. Para el parámetro, consulte el [archivo de parámetros de varios grupos de seguridad de red](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/multiplesecurityrules.parameters.json). |

## <a name="next-steps"></a>Pasos siguientes

- Para otros usos de bucles, consulte:
  - [Iteración de recursos en archivos Bicep](loop-resources.md)
  - [Iteración de propiedades en archivos Bicep](loop-properties.md)
  - [Iteración de la salida en archivos Bicep](loop-outputs.md)
- Si quiere conocer las secciones de una plantilla, consulte [Nociones sobre la estructura y la sintaxis de los archivos Bicep](file.md).
- Para información sobre cómo implementar varios recursos, vea [Uso de módulos de Bicep.](modules.md)
- Para establecer las dependencias de los recursos creados en un bucle, consulte [Establecimiento de dependencias de recursos](./resource-declaration.md#set-resource-dependencies).
- Para información sobre la implementación con PowerShell, consulte [Implementación de recursos con Bicep y Azure PowerShell](deploy-powershell.md).
- Para información sobre la implementación con la CLI de Azure, consulte [Implementación de recursos con Bicep y la CLI de Azure](deploy-cli.md).
