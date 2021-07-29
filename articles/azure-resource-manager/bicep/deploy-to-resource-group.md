---
title: Uso de Bicep para implementar recursos en grupos de recursos
description: Se describe cómo implementar recursos en un archivo Bicep. Se muestra cómo seleccionar más de un grupo de recursos como destino.
ms.topic: conceptual
ms.date: 06/01/2021
ms.openlocfilehash: f32ee9ce08b447a776ae74d19c0edabba233f345
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "111026820"
---
# <a name="resource-group-deployments-with-bicep-files"></a>Implementaciones de grupos de recursos con archivos de Bicep

En este artículo se describe cómo establecer el ámbito con Bicep al implementar en un grupo de administración.

## <a name="supported-resources"></a>Recursos compatibles

La mayoría de los recursos se pueden implementar en un grupo de recursos. Para obtener una lista de los recursos disponibles, vea [Referencia de plantillas de Resource Manager](/azure/templates).

## <a name="set-scope"></a>Ámbito de conjunto

De forma predeterminada, un archivo Bicep tiene como ámbito el grupo de recursos. Si desea establecer explícitamente el ámbito, use:

```bicep
targetScope = 'resourceGroup'
```

Sin embargo, no es necesario establecer el ámbito de destino en el grupo de recursos porque ese ámbito se usa de forma predeterminada.

## <a name="deployment-commands"></a>Comandos de implementación

Para implementar en un grupo de recursos, use los comandos de implementación de grupos de recursos.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para la CLI de Azure, use [az deployment group create](/cli/azure/deployment/group#az_deployment_group_create). El ejemplo siguiente implementa una plantilla para crear un grupo de recursos:

```azurecli-interactive
az deployment group create \
  --name demoRGDeployment \
  --resource-group ExampleGroup \
  --template-file main.bicep \
  --parameters storageAccountType=Standard_GRS
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para el comando de implementación de PowerShell, use [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment). El ejemplo siguiente implementa una plantilla para crear un grupo de recursos:

```azurepowershell-interactive
New-AzResourceGroupDeployment `
  -Name demoRGDeployment `
  -ResourceGroupName ExampleGroup `
  -TemplateFile main.bicep `
  -storageAccountType Standard_GRS `
```

---

Para obtener información más detallada sobre los comandos y las opciones de implementación para la implementación de plantillas de Resource Manager, vea:

* [Implementación de recursos con plantillas de ARM y la CLI de Azure](deploy-cli.md)
* [Implementación de recursos con las plantillas de ARM y Azure PowerShell](deploy-powershell.md)
* [Implementación de plantillas de Resource Manager desde Cloud Shell](deploy-cloud-shell.md)

## <a name="deployment-scopes"></a>Ámbitos de implementación

Al implementar en un grupo de recursos, puede implementar los recursos en:

* el grupo de recursos de destino para la operación de implementación
* otros grupos de recursos de la misma suscripción o de otras
* cualquier suscripción en el inquilino
* el inquilino del grupo de recursos

Un [recurso de extensión](scope-extension-resources.md) se puede limitar a un destino distinto del destino de implementación.

El usuario que implementa la plantilla debe tener acceso al ámbito especificado.

En esta sección se muestra cómo especificar distintos ámbitos. Puede combinar estos ámbitos diferentes en una sola plantilla.

### <a name="scope-to-target-resource-group"></a>Ámbito de destino del grupo de recursos

Para implementar recursos en el grupo de recursos de destino, agregue esos recursos al archivo Bicep.

```bicep
// resource deployed to target resource group
resource exampleResource 'Microsoft.Storage/storageAccounts@2019-06-01' = {
  ...
} 
```

Para ver una plantilla de ejemplo, consulte [Implementación en el grupo de recursos de destino](#deploy-to-target-resource-group).

### <a name="scope-to-different-resource-group"></a>Ámbito en un grupo de recursos diferente

Para implementar recursos en un grupo de recursos que no sea el grupo de recursos de destino, agregue un [módulo](modules.md). Use la [función resourceGroup](bicep-functions-scope.md#resourcegroup) para establecer la propiedad `scope` para ese módulo. 

Si el grupo de recursos está en otra suscripción, proporcione el id. de suscripción y el nombre del grupo de recursos. Si el grupo de recursos está en la misma suscripción que la implementación actual, proporcione solo el nombre del grupo de recursos. Si no especifica una suscripción en la [función resourceGroup](bicep-functions-scope.md#resourcegroup), se usa la suscripción actual. 

En el ejemplo siguiente se muestra un módulo que tiene como destino un grupo de recursos de una suscripción diferente.

```bicep
param otherResourceGroup string
param otherSubscriptionID string

// module deployed to different subscription and resource group
module exampleModule 'module.bicep' = {
  name: 'otherSubAndRG'
  scope: resourceGroup(otherSubscriptionID, otherResourceGroup)
}
```

En el ejemplo siguiente se muestra un módulo que tiene como destino un grupo de recursos de la misma suscripción.

```bicep
param otherResourceGroup string

// module deployed to resource group in the same subscription
module exampleModule 'module.bicep' = {
  name: 'otherRG'
  scope: resourceGroup(otherResourceGroup)
}
```

Para ver una plantilla de ejemplo, consulte [Implementación en varios grupos de recursos](#deploy-to-multiple-resource-groups).

### <a name="scope-to-subscription"></a>Ámbito de la suscripción

Para implementar recursos en una suscripción, agregue un módulo. Use la [función de suscripción](bicep-functions-scope.md#subscription) para establecer su propiedad `scope`. 

Para implementar en la suscripción actual, use la función de suscripción sin ningún parámetro. 

```bicep

// module deployed at subscription level
module exampleModule 'module.bicep' = {
  name: 'deployToSub'
  scope: subscription()
}
```

Para realizar la implementación en otra suscripción, especifique ese id. de suscripción como parámetro en la función de suscripción.

```bicep
param otherSubscriptionID string

// module deployed at subscription level but in a different subscription
module exampleModule 'module.bicep' = {
  name: 'deployToSub'
  scope: subscription(otherSubscriptionID)
}
```

Para ver una plantilla de ejemplo, consulte [Creación de un grupo de recursos](#create-resource-group).

### <a name="scope-to-tenant"></a>Ámbito del inquilino

Para crear recursos en el inquilino, agregue un módulo. Use la [función tenant](bicep-functions-scope.md#tenant) para establecer su propiedad `scope`.

El usuario que implementa la plantilla debe tener el [acceso necesario para realizar implementaciones en el inquilino](deploy-to-tenant.md#required-access).

En el ejemplo siguiente se incluye un módulo que se implementa en el inquilino.

```bicep
// module deployed at tenant level
module exampleModule 'module.bicep' = {
  name: 'deployToTenant'
  scope: tenant()
}
```

En lugar de usar un módulo, puede establecer el ámbito en `tenant()` para algunos tipos de recursos. En el ejemplo siguiente se implementa un grupo de administración en el inquilino.

```bicep
param mgName string = 'mg-${uniqueString(newGuid())}'

// ManagementGroup deployed at tenant
resource managementGroup 'Microsoft.Management/managementGroups@2020-05-01' = {
  scope: tenant()
  name: mgName
  properties: {}
}

output output string = mgName
```

Para más información, consulte [Grupo de administración](deploy-to-management-group.md#management-group).

## <a name="deploy-to-target-resource-group"></a>Implementación en el grupo de recursos de destino

Para implementar recursos en el grupo de recursos de destino, defina esos recursos en la sección `resources` de la plantilla. La plantilla siguiente crea una cuenta de almacenamiento en el grupo de recursos que se especifica en la operación de implementación.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-outputs/azuredeploy.bicep":::

## <a name="deploy-to-multiple-resource-groups"></a>Implementación de varios grupos de recursos

Puede implementar en más de un grupo de recursos de un único archivo Bicep.

> [!NOTE]
> Puede implementar hasta **800 grupos de recursos** en una única implementación. Normalmente, esta limitación significa que puede implementar en un grupo de recursos especificado para la plantilla primaria y hasta en 799 grupos de recursos en implementaciones anidadas o vinculadas. Sin embargo, si la plantilla principal contiene solo plantillas anidadas o vinculadas y no implementa por sí misma ningún recurso, puede incluir hasta 800 grupos de recursos en las implementaciones anidadas o vinculadas.

En el ejemplo siguiente se implementan dos cuentas de almacenamiento. La primera cuenta de almacenamiento se implementa en el grupo de recursos especificado en la operación de implementación. La segunda cuenta de almacenamiento se implementa en el grupo de recursos especificado en los parámetros `secondResourceGroup` y `secondSubscriptionID`:

```bicep
@maxLength(11)
param storagePrefix string

param firstStorageLocation string = resourceGroup().location

param secondResourceGroup string
param secondSubscriptionID string = ''
param secondStorageLocation string

var firstStorageName = '${storagePrefix}${uniqueString(resourceGroup().id)}'
var secondStorageName = '${storagePrefix}${uniqueString(secondSubscriptionID, secondResourceGroup)}'

module firstStorageAcct 'storage.bicep' = {
  name: 'storageModule1'
  params: {
    storageLocation: firstStorageLocation
    storageName: firstStorageName
  }
}

module secondStorageAcct 'storage.bicep' = {
  name: 'storageModule2'
  scope: resourceGroup(secondSubscriptionID, secondResourceGroup)
  params: {
    storageLocation: secondStorageLocation
    storageName: secondStorageName
  }
}
```

Ambos módulos usan el mismo archivo Bicep denominado **storage.bicep**.

```bicep
param storageLocation string
param storageName string

resource storageAcct 'Microsoft.Storage/storageAccounts@2019-06-01' = {
  name: storageName
  location: storageLocation
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'Storage'
  properties: {}
}
```

## <a name="create-resource-group"></a>Creación de un grupo de recursos

En una implementación de grupo de recursos, puede cambiar al nivel de una suscripción y crear un grupo de recursos. La siguiente plantilla implementa una cuenta de almacenamiento en el grupo de recursos de destino y crea un grupo de recursos nuevo en la suscripción especificada.

```bicep
@maxLength(11)
param storagePrefix string

param firstStorageLocation string = resourceGroup().location

param secondResourceGroup string
param secondSubscriptionID string = ''
param secondLocation string

var firstStorageName = '${storagePrefix}${uniqueString(resourceGroup().id)}'

// resource deployed to target resource group
module firstStorageAcct 'storage2.bicep' = {
  name: 'storageModule1'
  params: {
    storageLocation: firstStorageLocation
    storageName: firstStorageName
  }
}

// module deployed to subscription
module newRG 'resourceGroup.bicep' = {
  name: 'newResourceGroup'
  scope: subscription(secondSubscriptionID)
  params: {
    resourceGroupName: secondResourceGroup
    resourceGroupLocation: secondLocation
  }
}
```

En el ejemplo anterior se usa el siguiente archivo Bicep para el módulo que crea el nuevo grupo de recursos.

```bicep
targetScope='subscription'

param resourceGroupName string
param resourceGroupLocation string

resource newRG 'Microsoft.Resources/resourceGroups@2021-01-01' = {
  name: resourceGroupName
  location: resourceGroupLocation
}
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre otros ámbitos, consulte:

* [Implementaciones de suscripciones](deploy-to-subscription.md)
* [Implementaciones del grupo de administración](deploy-to-management-group.md)
* [Implementaciones de inquilinos](deploy-to-tenant.md)
