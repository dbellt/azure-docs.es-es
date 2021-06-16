---
title: Definición de varias instancias de una variable
description: Utilice la operación de copia en una plantilla de Azure Resource Manager para realizar varias iteraciones al crear una variable.
ms.topic: conceptual
ms.date: 02/13/2020
ms.openlocfilehash: 5f6459335fa3f059a76128edde4ee7096790fe13
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111957952"
---
# <a name="variable-iteration-in-arm-templates"></a>Iteración de variables en las plantillas de ARM

En este artículo, se explica cómo crear varios valores para una variable de una plantilla de Azure Resource Manager. Al agregar el elemento `copy` a la sección de variables de la plantilla, puede establecer de forma dinámica el número de elementos de una variable durante la implementación. Asimismo, evitará tener que repetir la sintaxis de la plantilla.

También puede utilizar el elemento copy con [recursos](copy-resources.md), [propiedades de un recurso](copy-properties.md) y [salidas](copy-outputs.md).

## <a name="syntax"></a>Sintaxis

El elemento copy tiene el siguiente formato general:

```json
"copy": [
  {
    "name": "<name-of-loop>",
    "count": <number-of-iterations>,
    "input": <values-for-the-variable>
  }
]
```

La propiedad `name` es cualquier valor que identifique el bucle. La propiedad `count` especifica el número de iteraciones que desea realizar en la variable.

La propiedad `input` especifica las propiedades que desea repetir. Tiene que crear una matriz de elementos construida a partir del valor de la propiedad `input`. Puede tratarse de una propiedad única (como una cadena) o de un objeto con varias propiedades.

## <a name="copy-limits"></a>Límites de copia

El valor de count no puede superar 800.

El valor de count no puede ser un número negativo. Puede ser cero si implementa la plantilla con una versión reciente de la CLI de Azure, PowerShell o la API de REST. Concretamente, se debe usar:

- Azure PowerShell **2.6** o posterior
- CLI de Azure **2.0.74** o posterior
- API de REST versión **2019-05-10** o posterior
- Las [implementaciones vinculadas](linked-templates.md) deben usar la versión **10-05-2019** o posterior de la API para el tipo de recurso de implementación

Las versiones anteriores de PowerShell, la CLI y API REST no admiten un valor de count de cero.

## <a name="variable-iteration"></a>Iteración variable

En el ejemplo siguiente, se muestra cómo se crea una matriz de valores de cadena:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "itemCount": {
      "type": "int",
      "defaultValue": 5
    }
  },
  "variables": {
    "copy": [
      {
        "name": "stringArray",
        "count": "[parameters('itemCount')]",
        "input": "[concat('item', copyIndex('stringArray', 1))]"
      }
    ]
  },
  "resources": [],
  "outputs": {
    "arrayResult": {
      "type": "array",
      "value": "[variables('stringArray')]"
    }
  }
}
```

La plantilla anterior devuelve una matriz con los siguientes valores:

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

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "itemCount": {
      "type": "int",
      "defaultValue": 5
    }
  },
  "variables": {
    "copy": [
      {
        "name": "objectArray",
        "count": "[parameters('itemCount')]",
        "input": {
          "name": "[concat('myDataDisk', copyIndex('objectArray', 1))]",
          "diskSizeGB": "1",
          "diskIndex": "[copyIndex('objectArray')]"
        }
      }
    ]
  },
  "resources": [],
  "outputs": {
    "arrayResult": {
      "type": "array",
      "value": "[variables('objectArray')]"
    }
  }
}
```

El ejemplo anterior devuelve una matriz con los valores siguientes:

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

> [!NOTE]
> La iteración de variables admite argumentos de desplazamiento. El desplazamiento debe aparecer después del nombre de la iteración, por ejemplo, `copyIndex('diskNames', 1)`. Si no proporciona un valor de desplazamiento, este se establece en 0 de forma predeterminada en la primera instancia.
>

También puede usar el elemento `copy` dentro de una variable. En el ejemplo siguiente, se crea un objeto en el que uno de los valores es una matriz.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "itemCount": {
      "type": "int",
      "defaultValue": 5
    }
  },
  "variables": {
    "topLevelObject": {
      "sampleProperty": "sampleValue",
      "copy": [
        {
          "name": "disks",
          "count": "[parameters('itemCount')]",
          "input": {
            "name": "[concat('myDataDisk', copyIndex('disks', 1))]",
            "diskSizeGB": "1",
            "diskIndex": "[copyIndex('disks')]"
          }
        }
      ]
    }
  },
  "resources": [],
  "outputs": {
    "objectResult": {
      "type": "object",
      "value": "[variables('topLevelObject')]"
    }
  }
}
```

El ejemplo anterior devuelve un objeto con los valores siguientes:

```json
{
  "sampleProperty": "sampleValue",
  "disks": [
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
}
```

En el ejemplo siguiente, se muestran las distintas formas en que puede usar `copy` con las variables.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {
    "disk-array-on-object": {
      "copy": [
        {
          "name": "disks",
          "count": 5,
          "input": {
            "name": "[concat('myDataDisk', copyIndex('disks', 1))]",
            "diskSizeGB": "1",
            "diskIndex": "[copyIndex('disks')]"
          }
        },
        {
          "name": "diskNames",
          "count": 5,
          "input": "[concat('myDataDisk', copyIndex('diskNames', 1))]"
        }
      ]
    },
    "copy": [
      {
        "name": "top-level-object-array",
        "count": 5,
        "input": {
          "name": "[concat('myDataDisk', copyIndex('top-level-object-array', 1))]",
          "diskSizeGB": "1",
          "diskIndex": "[copyIndex('top-level-object-array')]"
        }
      },
      {
        "name": "top-level-string-array",
        "count": 5,
        "input": "[concat('myDataDisk', copyIndex('top-level-string-array', 1))]"
      },
      {
        "name": "top-level-integer-array",
        "count": 5,
        "input": "[copyIndex('top-level-integer-array')]"
      }
    ]
  },
  "resources": [],
  "outputs": {
    "exampleObject": {
      "value": "[variables('disk-array-on-object')]",
      "type": "object"
    },
    "exampleArrayOnObject": {
      "value": "[variables('disk-array-on-object').disks]",
      "type" : "array"
    },
    "exampleObjectArray": {
      "value": "[variables('top-level-object-array')]",
      "type" : "array"
    },
    "exampleStringArray": {
      "value": "[variables('top-level-string-array')]",
      "type" : "array"
    },
    "exampleIntegerArray": {
      "value": "[variables('top-level-integer-array')]",
      "type" : "array"
    }
  }
}
```

## <a name="example-templates"></a>Plantillas de ejemplo

En los ejemplos siguientes, se muestran escenarios comunes en los que se crean varios valores para una variable.

|Plantilla  |Descripción  |
|---------|---------|
|[Variables de copia](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/copyvariables.json) |Muestra las diferentes maneras de efectuar una iteración en las variables. |
|[Varias reglas de seguridad](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/multiplesecurityrules.json) |Implementa varias reglas de seguridad en un grupo de seguridad de red. Crea las reglas de seguridad a partir de un parámetro. Para el parámetro, consulte el [archivo de parámetros de varios grupos de seguridad de red](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/multiplesecurityrules.parameters.json). |

## <a name="next-steps"></a>Pasos siguientes

- Para realizar un tutorial, consulte [Tutorial: Creación de varias instancias de recursos con plantillas de Resource Manager](template-tutorial-create-multiple-instances.md).
- Para otros usos del elemento copy, consulte:
  - [Iteración de recursos en las plantillas de ARM](copy-resources.md)
  - [Iteración de propiedades en las plantillas de ARM](copy-properties.md)
  - [Iteración de salida en las plantillas de ARM](copy-outputs.md)
- Si quiere obtener más información sobre las secciones de una plantilla, vea [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](./syntax.md).
- Para información sobre cómo implementar una plantilla, consulte [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](deploy-powershell.md).