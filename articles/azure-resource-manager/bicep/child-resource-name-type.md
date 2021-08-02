---
title: Recursos secundarios en Bicep
description: Describe cómo establecer el nombre y el tipo de los recursos secundarios en Bicep.
author: mumian
ms.author: jgao
ms.topic: conceptual
ms.date: 06/01/2021
ms.openlocfilehash: e19240a8050a7076a73c17f7e8ff3a9ab0d59591
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "111026885"
---
# <a name="set-name-and-type-for-child-resources-in-bicep"></a>Establecimiento del nombre y el tipo de los recursos secundarios en Bicep

Los recursos secundarios son recursos que solo existen en el contexto de otro recurso. Por ejemplo, una [extensión de máquina virtual](/azure/templates/microsoft.compute/virtualmachines/extensions) no puede existir sin una [máquina virtual](/azure/templates/microsoft.compute/virtualmachines). El recurso de extensión es un elemento secundario de la máquina virtual.

Cada recurso primario solo acepta determinados tipos de recursos como recursos secundarios. El tipo de recurso para el recurso secundario incluye el tipo de recurso para el recurso primario. Por ejemplo, `Microsoft.Web/sites/config` y `Microsoft.Web/sites/extensions` son recursos secundarios de `Microsoft.Web/sites`. Los tipos de recursos que se aceptan se especifican en el [esquema de plantilla](https://github.com/Azure/azure-resource-manager-schemas) del recurso principal.

En Bicep, puede especificar el recurso secundario dentro del recurso primario o fuera de él. Los valores proporcionados para el nombre y el tipo de recurso varían en función de si el recurso secundario se ha definido dentro del recurso primario o fuera de él.

## <a name="within-parent-resource"></a>Dentro del recurso primario

En el ejemplo siguiente se muestra el recurso secundario incluido en la propiedad resources del recurso primario.

```bicep
resource <parent-resource-symbolic-name> '<resource-type>@<api-version>' = {
  <parent-resource-properties>

  resource <child-resource-symbolic-name> '<child-resource-type>' = {
    <child-resource-properties>
  }
}
```

Debe aparecer una declaración de recursos anidada en el nivel superior de sintaxis del recurso primario. Las declaraciones pueden anidarse a una profundidad arbitraria, siempre y cuando cada nivel sea un tipo secundario de su recurso primario.

Cuando el recurso secundario se define dentro del tipo de recurso primario, los valores type y name se indican como un solo segmento sin barras diagonales. En el ejemplo siguiente se muestra una cuenta de almacenamiento con un servicio de archivos y un recurso compartido de archivos. El nombre del servicio de archivos se establece en **default** y su tipo se establece en **fileServices**. El nombre del recurso compartido de archivos se establece **en exampleshare** y su tipo en **shares**.

```bicep
resource storage 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: 'examplestorage'
  location: resourceGroup().location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }

  resource service 'fileServices' = {
    name: 'default'

    resource share 'shares' = {
      name: 'exampleshare'
    }
  }
}
```

Los tipos de recurso completos siguen siendo `Microsoft.Storage/storageAccounts/fileServices` y `Microsoft.Storage/storageAccounts/fileServices/shares`. No se proporciona `Microsoft.Storage/storageAccounts/` porque se supone que viene del tipo y versión del recurso primario. El recurso anidado puede declarar opcionalmente una versión de API mediante la sintaxis `<segment>@<version>`. Si el recurso anidado omite la versión de API, se usa la versión de API del recurso primario. Si el recurso anidado especifica una versión de API, se usa la versión de API especificada.

Los nombres de recursos secundarios se establecen en **default** y **exampleshare**, pero los nombres completos incluyen los nombres primarios. No se proporcionan **examplestorage** ni **default** porque se asumen a partir del recurso primario.

Un recurso anidado puede acceder a las propiedades de su recurso primario. Otros recursos declarados dentro del cuerpo del mismo recurso primario pueden hacerse referencia entre sí mediante los nombres simbólicos. Es posible que un recurso primario no tenga acceso a las propiedades de los recursos que contiene. Este intento provocaría una dependencia cíclica.

Para hacer referencia a un recurso anidado fuera del recurso primario, debe completarse con el nombre del recurso contenedor y el operador `::`. Por ejemplo, para generar una propiedad desde un recurso secundario:

```bicep
output childAddressPrefix string = VNet1::VNet1_Subnet1.properties.addressPrefix
```

## <a name="outside-parent-resource"></a>Fuera del recurso primario

En el ejemplo siguiente se muestra el recurso secundario fuera del recurso primario. Este enfoque puede usarse si el recurso primario no está implementado en la misma plantilla o si quiere usar [un bucle](loop-resources.md) para crear más de un recurso secundario. Especifique la propiedad primaria en el elemento secundario con el valor establecido en el nombre simbólico del elemento primario. Con esta sintaxis, todavía debe declarar el tipo de recurso completo, pero el nombre del recurso secundario es solo el nombre del elemento secundario.

```bicep
resource <parent-resource-symbolic-name> '<resource-type>@<api-version>' = {
  name: 'myParent'
  <parent-resource-properties>
}

resource <child-resource-symbolic-name> '<child-resource-type>@<api-version>' = {
  parent: 'myParent'
  name: 'myChild'
  <child-resource-properties>
}
```

Cuando el recurso secundario se define fuera del recurso primario, los valores type y name se indican con barras diagonales para incluir el tipo y el nombre primarios.

En el ejemplo siguiente se muestra una cuenta de almacenamiento, un servicio de archivos y un recurso compartido de archivos que se definen en el nivel raíz.

```bicep
resource storage 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: 'examplestorage'
  location: resourceGroup().location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }
}

resource service 'Microsoft.Storage/storageAccounts/fileServices@2021-02-01' = {
  name: 'default'
  parent: storage
}

resource share 'Microsoft.Storage/storageAccounts/fileServices/shares@2021-02-01' = {
  name: 'exampleshare'
  parent: service
}
```

Hacer referencia al nombre simbólico del recurso secundario funciona igual que hacer referencia al elemento primario.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información sobre cómo crear archivos de Bicep, consulte [Nociones sobre la estructura y la sintaxis de los archivos de Bicep](./file.md).
* Para información sobre el formato del nombre del recurso al hacer referencia a él, consulte la [función reference](./bicep-functions-resource.md#reference).
