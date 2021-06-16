---
title: Módulos de Bicep
description: Describe cómo definir y utilizar un módulo, y cómo usar ámbitos de módulo.
ms.topic: conceptual
ms.date: 03/30/2021
ms.openlocfilehash: 6c325bbbe265e13241119761373985ca4552b158
ms.sourcegitcommit: 5c136a01bddfccb2cc9f7e7e7741e2cf2651ddbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111353560"
---
# <a name="use-bicep-modules-preview"></a>Uso de módulos de Bicep (versión preliminar)

Bicep permite dividir una solución compleja en módulos. Un módulo de Bicep es un conjunto de uno o más recursos que se van a implementar juntos. Los módulos abstraen los detalles complejos de la declaración de recursos sin procesar, lo que puede aumentar la legibilidad. Puede volver a usar estos módulos y compartirlos con otras personas. En combinación con las [especificaciones de plantilla](./template-specs.md), un módulo de Bicep ofrece modularidad y la capacidad de reutilizar el código. Los módulos de Bicep se transpilan en una sola plantilla de ARM con [plantillas anidadas](./linked-templates.md#nested-template) para su implementación. En Bicep, [_dependsOn_](./template-syntax.md#resources) se administra automáticamente.

Para ver un tutorial, consulte [Tutorial: Adición de módulos de Bicep](./bicep-tutorial-add-modules.md).

## <a name="define-modules"></a>Definición de los módulos

Cada archivo de Bicep puede usarse como un módulo. Un módulo solo expone parámetros y salidas como un contrato a otros archivos de Bicep. Tanto los parámetros como las salidas son opcionales.

El siguiente archivo de Bicep se puede implementar directamente para crear una cuenta de almacenamiento o usarse como un módulo.  En la sección siguiente se muestra cómo usar módulos:

```bicep
@minLength(3)
@maxLength(11)
param storagePrefix string

@allowed([
  'Standard_LRS'
  'Standard_GRS'
  'Standard_RAGRS'
  'Standard_ZRS'
  'Premium_LRS'
  'Premium_ZRS'
  'Standard_GZRS'
  'Standard_RAGZRS'
])
param storageSKU string = 'Standard_LRS'
param location string

var uniqueStorageName = '${storagePrefix}${uniqueString(resourceGroup().id)}'

resource stg 'Microsoft.Storage/storageAccounts@2019-04-01' = {
  name: uniqueStorageName
  location: location
  sku: {
    name: storageSKU
  }
  kind: 'StorageV2'
  properties: {
    supportsHttpsTrafficOnly: true
  }
}

output storageEndpoint object = stg.properties.primaryEndpoints
```

La salida se usa para pasar valores a los archivos de Bicep primarios.

## <a name="consume-modules"></a>Uso de módulos

Utilice la palabra clave _module_ para usar un módulo. El siguiente archivo de Bicep implementa el recurso definido en el archivo de módulo al que se hace referencia:

```bicep
@minLength(3)
@maxLength(11)
param namePrefix string
param location string = resourceGroup().location

module stgModule './storageAccount.bicep' = {
  name: 'storageDeploy'
  params: {
    storagePrefix: namePrefix
    location: location
  }
}

output storageEndpoint object = stgModule.outputs.storageEndpoint
```

- **modulo**: palabra clave.
- **nombre simbólico** (stgModule): identificador del módulo.
- **archivo de módulo**: en este ejemplo, la ruta de acceso al módulo se especifica mediante una ruta de acceso relativa (./storageAccount.bicep). Todas las rutas de acceso de Bicep deben especificarse mediante el separador de directorios de barra diagonal (/), con el fin de garantizar una compilación coherente entre plataformas. El carácter de barra diagonal inversa de Windows (\\) no se admite.
- La propiedad **_name_** (storageDeploy) es necesaria cuando se utiliza un módulo. Cuando Bicep genera la plantilla IL, este campo se usa como el nombre del recurso de implementación anidado, que se genera para el módulo:

    ```json
    ...
    ...
    "resources": [
      {
        "type": "Microsoft.Resources/deployments",
        "apiVersion": "2020-10-01",
        "name": "storageDeploy",
        "properties": {
          ...
        }
      }
    ]
    ...
    ```

Para obtener un valor de salida de un módulo, recupere el valor de la propiedad con la siguiente sintaxis: `stgModule.outputs.storageEndpoint`, donde `stgModule` es el identificador del módulo.

## <a name="configure-module-scopes"></a>Configuración de ámbitos de módulo

Al declarar un módulo, puede proporcionar una propiedad _scope_ para establecer el ámbito en el que se implementará el módulo:

```bicep
module stgModule './storageAccount.bicep' = {
  name: 'storageDeploy'
  scope: resourceGroup('someOtherRg') // pass in a scope to a different resourceGroup
  params: {
    storagePrefix: namePrefix
    location: location
  }
}
```

Se puede omitir la propiedad _scope_ cuando el ámbito de destino del módulo y el ámbito de destino del elemento primario sean el mismo. Cuando no se proporciona la propiedad scope, el módulo se implementa en el ámbito de destino del elemento primario.

En el siguiente archivo de Bicep se muestra cómo crear un grupo de recursos e implementar un módulo en el grupo de recursos:

```bicep
// set the target scope for this file
targetScope = 'subscription'

@minLength(3)
@maxLength(11)
param namePrefix string

param location string = deployment().location

var resourceGroupName = '${namePrefix}rg'
resource myResourceGroup 'Microsoft.Resources/resourceGroups@2020-01-01' = {
  name: resourceGroupName
  location: location
  scope: subscription()
}

module stgModule './storageAccount.bicep' = {
  name: 'storageDeploy'
  scope: myResourceGroup
  params: {
    storagePrefix: namePrefix
    location: location
  }
}

output storageEndpoint object = stgModule.outputs.storageEndpoint
```

## <a name="next-steps"></a>Pasos siguientes

- Para completar un tutorial, consulte [Tutorial: Adición de módulos de Bicep](./bicep-tutorial-add-modules.md).
