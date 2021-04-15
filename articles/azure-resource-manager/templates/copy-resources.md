---
title: Implementación de varias instancias de recursos
description: Use la operación de copia y las matrices de una plantilla de Azure Resource Manager (plantilla de ARM) para realizar varias iteraciones al implementar recursos.
ms.topic: conceptual
ms.date: 04/01/2021
ms.openlocfilehash: 3af676cce544c125e441857f06556b9ff7eee697
ms.sourcegitcommit: 77d7639e83c6d8eb6c2ce805b6130ff9c73e5d29
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/05/2021
ms.locfileid: "106385722"
---
# <a name="resource-iteration-in-arm-templates"></a>Iteración de recursos en las plantillas de ARM

En este artículo se explica cómo se crean varias instancias de un recurso en la plantilla de Azure Resource Manager (plantilla de ARM). Al agregar el bucle de copia a la sección de recursos de la plantilla, puede establecer de forma dinámica el número de recursos que va a implementar. Asimismo, evitará tener que repetir la sintaxis de la plantilla.

También puede usar el bucle de copia con [propiedades](copy-properties.md), [variables](copy-variables.md) y [salidas](copy-outputs.md).

Si tiene que especificar si un recurso se implementa, consulte [Elemento condition](conditional-resource-deployment.md).

## <a name="syntax"></a>Sintaxis

# <a name="json"></a>[JSON](#tab/json)

Agregue el elemento `copy` a la sección de recursos de la plantilla para implementar varias instancias del recurso. El elemento `copy` tiene el siguiente formato general:

```json
"copy": {
  "name": "<name-of-loop>",
  "count": <number-of-iterations>,
  "mode": "serial" <or> "parallel",
  "batchSize": <number-to-deploy-serially>
}
```

La propiedad `name` es cualquier valor que identifique el bucle. La propiedad `count` especifica el número de iteraciones que desea realizar en el tipo de recurso.

Utilice las propiedades `mode` y `batchSize` para especificar si los recursos van a implementarse simultáneamente o por orden. Estas propiedades se describen en el artículo [En serie o en paralelo](#serial-or-parallel).

# <a name="bicep"></a>[Bicep](#tab/bicep)

Los bucles se pueden usar para declarar varios recursos al:

- Iterar una matriz:

  ```bicep
  @batchSize(<number>)
  resource <resource-symbolic-name> '<resource-type>@<api-version>' = [for <item> in <collection>: {
    <resource-properties>
  }
  ```

- Iterar los elementos de una matriz:

  ```bicep
  @batchSize(<number>)
  resource <resource-symbolic-name> '<resource-type>@<api-version>' = [for (<item>, <index>) in <collection>: {
    <resource-properties>
  }
  ```

- Usar el índice de bucle:

  ```bicep
  @batchSize(<number>)
  resource <resource-symbolic-name> '<resource-type>@<api-version>' = [for <index> in range(<start>, <stop>): {
    <resource-properties>
  }
  ```

---

## <a name="copy-limits"></a>Límites de copia

El valor de count no puede superar 800.

El valor de count no puede ser un número negativo. Puede ser cero si implementa la plantilla con una versión reciente de la CLI de Azure, PowerShell o la API de REST. Concretamente, se debe usar:

- Azure PowerShell **2.6** o posterior
- CLI de Azure **2.0.74** o posterior
- API de REST versión **2019-05-10** o posterior
- Las [implementaciones vinculadas](linked-templates.md) deben usar la versión **10-05-2019** o posterior de la API para el tipo de recurso de implementación

Las versiones anteriores de PowerShell, la CLI y API REST no admiten un valor de count de cero.

Tenga cuidado al usar la [implementación de modo completo](deployment-modes.md) con un bucle de copia. Si vuelve a implementar con el modo completo en un grupo de recursos, se eliminan todos los recursos que no se especifican en la plantilla después de resolver el bucle de copia.

## <a name="resource-iteration"></a>Iteración de recursos

En el ejemplo siguiente, se crea el número de cuentas de almacenamiento especificado en el parámetro `storageCount`.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageCount": {
      "type": "int",
      "defaultValue": 2
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[concat(copyIndex(),'storage', uniqueString(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "properties": {},
      "copy": {
        "name": "storagecopy",
        "count": "[parameters('storageCount')]"
      }
    }
  ]
}
```

Tenga en cuenta que el nombre de cada recurso incluye la función `copyIndex()`, que devuelve la iteración actual del bucle. `copyIndex()` es de base cero. Así, en el ejemplo siguiente:

```json
"name": "[concat('storage', copyIndex())]",
```

Crea estos nombres:

- storage0
- storage1
- storage2.

Para desplazar el valor de índice, puede pasar un valor en la función `copyIndex()`. El número de iteraciones se sigue especificando en el elemento copy, pero el valor de `copyIndex` se desplaza el valor especificado. Así, en el ejemplo siguiente:

```json
"name": "[concat('storage', copyIndex(1))]",
```

Crea estos nombres:

- storage1
- storage2
- storage3

La operación de copia es útil al trabajar con matrices, ya que puede iterar a través de cada elemento de la matriz. Use la función `length` en la matriz para especificar el número de iteraciones, y `copyIndex` para recuperar el índice actual de la matriz.

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param storageCount int = 2

resource storage_id 'Microsoft.Storage/storageAccounts@2019-04-01' = [for i in range(0, storageCount): {
  name: '${i}storage${uniqueString(resourceGroup().id)}'
  location: resourceGroup().location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'Storage'
  properties: {}
}]
```

Observe que el índice `i` se usa para crear el nombre del recurso de la cuenta de almacenamiento.

---

En el ejemplo siguiente, se crea una cuenta de almacenamiento para cada uno de los nombres proporcionados en el parámetro.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "storageNames": {
          "type": "array",
          "defaultValue": [
            "contoso",
            "fabrikam",
            "coho"
          ]
      }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[concat(parameters('storageNames')[copyIndex()], uniqueString(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "properties": {},
      "copy": {
        "name": "storagecopy",
        "count": "[length(parameters('storageNames'))]"
      }
    }
  ],
  "outputs": {}
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param storageNames array = [
  'contoso'
  'fabrikam'
  'coho'
]

resource storageNames_id 'Microsoft.Storage/storageAccounts@2019-04-01' = [for name in storageNames: {
  name: concat(name, uniqueString(resourceGroup().id))
  location: resourceGroup().location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'Storage'
  properties: {}
}]
```

---

Si desea devolver valores de los recursos implementados, puede usar [copy en la sección de salidas](copy-outputs.md).

## <a name="serial-or-parallel"></a>En serie o en paralelo

Resource Manager crea los recursos en paralelo de forma predeterminada. No se aplica ningún límite en el número de recursos implementados en paralelo, aparte del límite total de 800 recursos de la plantilla. No se garantiza el orden en el que se crean.

Sin embargo, es posible que quiera especificar que los recursos se implementen en secuencia. Por ejemplo, al actualizar un entorno de producción, puede que quiera escalonar las actualizaciones para que solo una cantidad determinada se actualice al mismo tiempo.

Por ejemplo, para implementar en serie dos cuentas de almacenamiento a la vez, use lo siguiente:

# <a name="json"></a>[JSON](#tab/json)

Para implementar en serie varias instancias de un recurso, establezca `mode` en el valor **serial** y `batchSize` en el número de instancias que se implementarán a la vez. Con mode establecido en serial, Resource Manager crea una dependencia en las instancias anteriores del bucle, por lo que no se inicia ningún lote hasta que se completa el lote anterior.

El valor de `batchSize` no puede superar el valor de `count` en el elemento copy.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[concat(copyIndex(),'storage', uniqueString(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "copy": {
        "name": "storagecopy",
        "count": 4,
        "mode": "serial",
        "batchSize": 2
      },
      "properties": {}
    }
  ],
  "outputs": {}
}
```

La propiedad `mode` también acepta **parallel**, que es el valor predeterminado.

# <a name="bicep"></a>[Bicep](#tab/bicep)

Para implementar en serie más de una instancia de un recurso, establezca el [decorador](./bicep-file.md#resource-and-module-decorators) `batchSize` en el número de instancias que se implementarán a la vez. Con mode establecido en serial, Resource Manager crea una dependencia en las instancias anteriores del bucle, por lo que no se inicia ningún lote hasta que se completa el lote anterior.

```bicep
@batchSize(2)
resource storage_id 'Microsoft.Storage/storageAccounts@2019-04-01' = [for i in range(0, 4): {
  name: '${i}storage${uniqueString(resourceGroup().id)}'
  location: resourceGroup().location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'Storage'
  properties: {}
}]
```

---

## <a name="iteration-for-a-child-resource"></a>Iteración para un recurso secundario

No puede usar un bucle copy en un recurso secundario. Para crear varias instancias de un recurso que se define normalmente como anidado dentro de otro recurso, debe crear dicho recurso como uno de nivel superior. La relación con el recurso principal se define a través de las propiedades type y name.

Por ejemplo, supongamos que suele definir un conjunto de datos como un recurso secundario dentro de una factoría de datos.

```json
"resources": [
{
  "type": "Microsoft.DataFactory/factories",
  "name": "exampleDataFactory",
  ...
  "resources": [
    {
      "type": "datasets",
      "name": "exampleDataSet",
      "dependsOn": [
        "exampleDataFactory"
      ],
      ...
    }
  ]
```

Para crear más de un conjunto de datos, muévalos fuera de la factoría de datos. El conjunto de datos debe estar en el mismo nivel que la factoría de datos, pero seguirá siendo un recurso secundario de la factoría de datos. La relación entre el conjunto de datos y la factoría de datos se conserva a través de los parámetros type y name. Puesto que la propiedad de type ya no se puede inferir de su posición en la plantilla, debe proporcionar el nombre completo del tipo con el formato: `{resource-provider-namespace}/{parent-resource-type}/{child-resource-type}`.

Para establecer una relación de principal-secundario con una instancia de la factoría de datos, proporcione un nombre para el conjunto de datos que incluya el nombre de recurso principal. Utilice el formato: `{parent-resource-name}/{child-resource-name}`.

En el siguiente ejemplo se muestra la implementación:

# <a name="json"></a>[JSON](#tab/json)

```json
"resources": [
{
  "type": "Microsoft.DataFactory/factories",
  "name": "exampleDataFactory",
  ...
},
{
  "type": "Microsoft.DataFactory/factories/datasets",
  "name": "[concat('exampleDataFactory', '/', 'exampleDataSet', copyIndex())]",
  "dependsOn": [
    "exampleDataFactory"
  ],
  "copy": {
    "name": "datasetcopy",
    "count": "3"
  },
  ...
}]
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
resource dataFactoryName_resource 'Microsoft.DataFactory/factories@2018-06-01' = {
  name: "exampleDataFactory"
  ...
}

resource dataFactoryName_ArmtemplateTestDatasetIn 'Microsoft.DataFactory/factories/datasets@2018-06-01' = [for i in range(0, 3): {
  name: 'exampleDataFactory/exampleDataset${i}'
  ...
}
```

---

## <a name="example-templates"></a>Plantillas de ejemplo

En los ejemplos siguientes se muestran escenarios comunes para crear más de una instancia de un recurso o propiedad.

|Plantilla  |Description  |
|---------|---------|
|[Almacenamiento de copias](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/copystorage.json) |Implementa más de una cuenta de almacenamiento con un número de índice en el nombre. |
|[Almacenamiento de copias en serie](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/serialcopystorage.json) |Implementa varias cuentas de almacenamiento, una tras otra. El nombre incluye el número de índice. |
|[Almacenamiento de copias con una matriz](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/copystoragewitharray.json) |Implementa varias cuentas de almacenamiento. El nombre incluye el valor de una matriz. |

## <a name="next-steps"></a>Pasos siguientes

- Para establecer las dependencias de los recursos que se crean en un bucle de copia, consulte [Definición del orden de implementación de recursos en las plantillas de ARM](define-resource-dependency.md).
- Para realizar un tutorial, consulte [Tutorial: Creación de varias instancias de recursos con plantillas de Resource Manager](template-tutorial-create-multiple-instances.md).
- Para un módulo de Microsoft Learn que abarca la copia de recursos, consulte [Administración de implementaciones complejas en la nube mediante características avanzadas de la plantilla de ARM](/learn/modules/manage-deployments-advanced-arm-template-features/).
- Para otros usos del bucle de copia, consulte:
  - [Iteración de propiedades en las plantillas de ARM](copy-properties.md)
  - [Iteración de variables en las plantillas de ARM](copy-variables.md)
  - [Iteración de salida en las plantillas de ARM](copy-outputs.md)
- Para obtener información acerca del uso de copy con plantillas anidadas, consulte [Uso de copy](linked-templates.md#using-copy).
