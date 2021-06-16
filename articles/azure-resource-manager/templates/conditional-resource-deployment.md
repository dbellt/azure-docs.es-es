---
title: Implementación condicional con plantillas
description: En este artículo se describe cómo realizar la implementación condicional de un recurso en una plantilla de Azure Resource Manager (plantilla de ARM).
ms.topic: conceptual
ms.date: 05/07/2021
ms.openlocfilehash: 352ee71fea77608ae27552630a7d302b215374a1
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111969494"
---
# <a name="conditional-deployment-in-arm-templates"></a>Implementación condicional en las plantillas de ARM

A veces, tiene la opción de implementar un recurso en una plantilla de Azure Resource Manager (plantilla de ARM). Use el elemento `condition` para especificar si se implementó el recurso. El valor de la condición se resuelve como true o false. Cuando el valor es true, el recurso se crea. Cuando el valor es false, el recurso no se crea. El valor solo se puede aplicar a todo el recurso.

> [!NOTE]
> La implementación condicional no se aplica en cascada a los [recursos secundarios](child-resource-name-type.md). Si desea implementar condicionalmente un recurso y sus recursos secundarios, debe aplicar la misma condición a cada tipo de recurso.

## <a name="deploy-condition"></a>Condición de implementación

Puede pasar un valor de parámetro que indica si un recurso está implementado. En el ejemplo siguiente se implementa una zona DNS de forma condicional.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "deployZone": {
      "type": "bool"
    }
  },
  "functions": [],
  "resources": [
    {
      "condition": "[parameters('deployZone')]",
      "type": "Microsoft.Network/dnsZones",
      "apiVersion": "2018-05-01",
      "name": "myZone",
      "location": "global"
    }
  ]
}
```

Para obtener un ejemplo más complejo, consulte el [servidor lógico de Azure SQL](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.sql/sql-logical-server).

## <a name="new-or-existing-resource"></a>Recurso nuevo o existente

Puede usar la implementación condicional para crear un recurso nuevo o usar uno existente. En el ejemplo siguiente se muestra cómo implementar una cuenta de almacenamiento nueva o usar una existente.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": {
      "type": "string"
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    },
    "newOrExisting": {
      "type": "string",
      "defaultValue": "new",
      "allowedValues": [
        "new",
        "existing"
      ]
    }
  },
  "functions": [],
  "resources": [
    {
      "condition": "[equals(parameters('newOrExisting'), 'new')]",
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-06-01",
      "name": "[parameters('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_LRS",
        "tier": "Standard"
      },
      "kind": "StorageV2",
      "properties": {
        "accessTier": "Hot"
      }
    }
  ]
}
```

Cuando el parámetro `newOrExisting` está establecido en **new**, la condición se evalúa como true. Se implementa la cuenta de almacenamiento. Sin embargo, cuando `newOrExisting` está establecido **existing**, la condición se evalúa como false y no se implementa la cuenta de almacenamiento.

Para una plantilla de ejemplo completo que usa el elemento `condition`, consulte [VM with a new or existing Virtual Network, Storage, and Public IP](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/vm-new-or-existing-conditions) (Máquina virtual con una red virtual nueva o existente, almacenamiento y dirección IP pública).

## <a name="runtime-functions"></a>Funciones en tiempo de ejecución

Si usa una función [reference](template-functions-resource.md#reference) o [list](template-functions-resource.md#list) con un recurso que se implementa de forma condicional, se puede evaluar la función incluso si el recurso no está implementado. Se genera un error si la función hace referencia a un recurso que no existe.

Use la función [if](template-functions-logical.md#if) para asegurarse de que la función se evalúa solo para las condiciones en las que se implementa el recurso. Consulte la [función if](template-functions-logical.md#if) con una plantilla de ejemplo que use `if` y `reference` con un recurso implementado de forma condicional.

Puede establecer un [recurso como dependiente](./resource-dependency.md) en un recurso condicional exactamente como lo haría con cualquier otro recurso. Cuando un recurso condicional no está implementado, Azure Resource Manager lo quita automáticamente de las dependencias necesarias.

## <a name="complete-mode"></a>Modo completo

Si implementa una plantilla con el [modo completo](deployment-modes.md) y no se implementa un recurso porque `condition` se evalúa como false, el resultado depende de la versión de API REST que use para implementar la plantilla. Si usa una versión anterior a 2019-05-10, el recurso **no se elimina**. Con 2019-05-10 o posterior, el recurso **se elimina**. Las versiones más recientes de Azure PowerShell y la CLI de Azure eliminan el recurso cuando la condición es falsa.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener un módulo de Microsoft Learn en el que se describe la implementación condicional, vea [Administración de implementaciones complejas en la nube mediante características avanzadas de la plantilla de ARM](/learn/modules/manage-deployments-advanced-arm-template-features/).
* Para recomendaciones sobre la creación de platillas, consulte [Procedimientos recomendados de plantillas de ARM](./best-practices.md).
* Para crear varias instancias de un recurso, consulte [Iteración de recursos en plantillas de ARM](copy-resources.md).