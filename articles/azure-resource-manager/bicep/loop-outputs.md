---
title: Definición de varias instancias de un valor de salida en Bicep
description: Use un bucle de salida de Bicep para iterar y devolver valores de implementación.
author: mumian
ms.author: jgao
ms.topic: conceptual
ms.date: 06/01/2021
ms.openlocfilehash: cc931b7e0d65804892176a2965f87022a2becb7b
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "111026756"
---
# <a name="output-iteration-in-bicep"></a>Iteración de la salida en Bicep

En este artículo, se muestra cómo puede crear más de un valor para una salida en el archivo Bicep. Puede agregar un bucle a la sección `output` del archivo y devolver dinámicamente varios elementos durante la implementación.

También puede usar un bucle con [recursos](loop-resources.md), [propiedades de un recurso](loop-properties.md) y [variables](loop-variables.md).

## <a name="syntax"></a>Sintaxis

Los bucles se pueden usar para devolver muchos elementos durante la implementación mediante lo siguiente:

- Iteración en una matriz.

  ```bicep
  output <output-name> array = [for <item> in <collection>: {
    <properties>
  }]

  ```

- Iteración en los elementos de una matriz.

  ```bicep
  output <output-name> array = [for <item>, <index> in <collection>: {
    <properties>
  }]
  ```

- Uso de un índice de bucle.

  ```bicep
  output <output-name> array = [for <index> in range(<start>, <stop>): {
    <properties>
  }]
  ```

## <a name="loop-limits"></a>Límites del bucle

Las iteraciones de bucle del archivo Bicep no pueden ser un número negativo ni superar las 800 iteraciones. Para implementar archivos Bicep, instale la versión más reciente de las [herramientas de Bicep](install.md).

## <a name="output-iteration"></a>Iteración de salida

En el ejemplo siguiente se crea un número variable de cuentas de almacenamiento y se devuelve un punto de conexión para cada cuenta de almacenamiento.

```bicep
param rgLocation string = resourceGroup().location
param storageCount int = 2

var baseNameVar = 'storage${uniqueString(resourceGroup().id)}'

resource baseName 'Microsoft.Storage/storageAccounts@2021-02-01' = [for i in range(0, storageCount): {
  name: '${i}${baseNameVar}'
  location: rgLocation
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'Storage'
}]

output storageEndpoints array = [for i in range(0, storageCount): reference('${i}${baseNameVar}').primaryEndpoints.blob]
```

La salida devuelve una matriz con los valores siguientes:

```json
[
  "https://0storagecfrbqnnmpeudi.blob.core.windows.net/",
  "https://1storagecfrbqnnmpeudi.blob.core.windows.net/"
]
```

En el ejemplo siguiente se devuelven tres propiedades de las nuevas cuentas de almacenamiento.

```bicep
param rgLocation string = resourceGroup().location
param storageCount int = 2

var baseNameVar = 'storage${uniqueString(resourceGroup().id)}'

resource baseName 'Microsoft.Storage/storageAccounts@2021-02-01' = [for i in range(0, storageCount): {
  name: '${i}${baseNameVar}'
  location: rgLocation
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'Storage'
}]

output storageInfo array = [for i in range(0, storageCount): {
  id: reference('${i}${baseNameVar}', '2021-02-01', 'Full').resourceId
  blobEndpoint: reference('${i}${baseNameVar}').primaryEndpoints.blob
  status: reference('${i}${baseNameVar}').statusOfPrimary
}]
```

La salida devuelve una matriz con los valores siguientes:

```json
[
  {
    "id": "Microsoft.Storage/storageAccounts/0storagecfrbqnnmpeudi",
    "blobEndpoint": "https://0storagecfrbqnnmpeudi.blob.core.windows.net/",
    "status": "available"
  },
  {
    "id": "Microsoft.Storage/storageAccounts/1storagecfrbqnnmpeudi",
    "blobEndpoint": "https://1storagecfrbqnnmpeudi.blob.core.windows.net/",
    "status": "available"
  }
]
```

En este ejemplo se usa un índice de matriz porque las referencias directas a un módulo de recurso o una colección de módulos no se admiten en bucles de salida.

```bicep
param rgLocation string = resourceGroup().location

var stgNames = [
  'demostg1'
  'demostg2'
  'demostg3'
]

resource stg 'Microsoft.Storage/storageAccounts@2021-02-01' = [for name in stgNames: {
  name: name
  location: rgLocation
  kind: 'Storage'
  sku: {
    name: 'Standard_LRS'
    tier: 'Standard'
  }
}]

output stgOutput array = [for (name, i) in stgNames: {
  name: stg[i].name
  resourceId: stg[i].id
}]
```

La salida devuelve una matriz con los valores siguientes:

```json
[
  {
    "name": "demostg1",
    "resourceId": "/subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.Storage/storageAccounts/demostg1"
  },
  {
    "name": "demostg2",
    "resourceId": "/subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.Storage/storageAccounts/demostg2"
  },
  {
    "name": "demostg3",
    "resourceId": "/subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.Storage/storageAccounts/demostg3"
  }
]
```

## <a name="next-steps"></a>Pasos siguientes

- Para otros usos de bucles, consulte:
  - [Iteración de recursos en archivos Bicep](loop-resources.md)
  - [Iteración de propiedades en archivos Bicep](loop-properties.md)
  - [Iteración de variables en archivos Bicep](loop-variables.md)
- Si quiere conocer las secciones de una plantilla, consulte [Nociones sobre la estructura y la sintaxis de los archivos Bicep](file.md).
- Para información sobre cómo implementar varios recursos, vea [Uso de módulos de Bicep.](modules.md)
- Para establecer las dependencias de los recursos creados en un bucle, consulte [Establecimiento de dependencias de recursos](./resource-declaration.md#set-resource-dependencies).
- Para información sobre la implementación con PowerShell, consulte [Implementación de recursos con Bicep y Azure PowerShell](deploy-powershell.md).
- Para información sobre la implementación con la CLI de Azure, consulte [Implementación de recursos con Bicep y la CLI de Azure](deploy-cli.md).
