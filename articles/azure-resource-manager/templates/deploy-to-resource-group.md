---
title: Implementación de recursos en grupos de recursos
description: Se describe cómo implementar recursos en una plantilla de Azure Resource Manager. Se muestra cómo seleccionar más de un grupo de recursos como destino.
ms.topic: conceptual
ms.date: 01/13/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 77985020324efde1b6df47bd10a70015cc0aa2a4
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108754080"
---
# <a name="resource-group-deployments-with-arm-templates"></a>Implementaciones de grupos de recursos con plantillas de Resource Manager

En este artículo se describe cómo limitar la implementación a un grupo de recursos. Para la implementación puede usar una plantilla de Azure Resource Manager. En el artículo también se muestra cómo expandir el ámbito más allá del grupo de recursos en la operación de implementación.

## <a name="supported-resources"></a>Recursos compatibles

La mayoría de los recursos se pueden implementar en un grupo de recursos. Para obtener una lista de los recursos disponibles, vea [Referencia de plantillas de Resource Manager](/azure/templates).

## <a name="schema"></a>Schema

Para las plantillas, usa el esquema siguiente:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    ...
}
```

Para los archivos de parámetros, use:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    ...
}
```

## <a name="deployment-commands"></a>Comandos de implementación

Para implementar en un grupo de recursos, use los comandos de implementación de grupos de recursos.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para la CLI de Azure, use [az deployment group create](/cli/azure/deployment/group#az_deployment_group_create). El ejemplo siguiente implementa una plantilla para crear un grupo de recursos:

```azurecli-interactive
az deployment group create \
  --name demoRGDeployment \
  --resource-group ExampleGroup \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.storage/storage-account-create/azuredeploy.json" \
  --parameters storageAccountType=Standard_GRS
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para el comando de implementación de PowerShell, use [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment). El ejemplo siguiente implementa una plantilla para crear un grupo de recursos:

```azurepowershell-interactive
New-AzResourceGroupDeployment `
  -Name demoRGDeployment `
  -ResourceGroupName ExampleGroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.storage/storage-account-create/azuredeploy.json `
  -storageAccountType Standard_GRS `
```

---

Para obtener información más detallada sobre los comandos y las opciones de implementación para la implementación de plantillas de Resource Manager, vea:

* [Implementación de recursos con Azure Portal y plantillas de Resource Manager](deploy-portal.md)
* [Implementación de recursos con plantillas de ARM y la CLI de Azure](deploy-cli.md)
* [Implementación de recursos con las plantillas de ARM y Azure PowerShell](deploy-powershell.md)
* [Implementación de recursos con plantillas de Resource Manager y la API REST de Azure Resource Manager](deploy-rest.md)
* [Usar un botón de implementación para implementar plantillas desde el repositorio de GitHub](deploy-to-azure-button.md)
* [Implementación de plantillas de Resource Manager desde Cloud Shell](deploy-cloud-shell.md)

## <a name="deployment-scopes"></a>Ámbitos de implementación

Al implementar en un grupo de recursos, puede implementar los recursos en:

* el grupo de recursos de destino de la operación
* otros grupos de recursos de la misma suscripción o de otras
* cualquier suscripción en el inquilino
* el inquilino del grupo de recursos

Un [recurso de extensión](scope-extension-resources.md) se puede limitar a un destino distinto del destino de implementación.

El usuario que implementa la plantilla debe tener acceso al ámbito especificado.

En esta sección se muestra cómo especificar distintos ámbitos. Puede combinar estos ámbitos diferentes en una sola plantilla.

### <a name="scope-to-target-resource-group"></a>Ámbito de destino del grupo de recursos

Para implementar recursos en el recurso de destino, agregue esos recursos a la sección de recursos de la plantilla.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/default-rg.json" highlight="5":::

Para ver una plantilla de ejemplo, consulte [Implementación en el grupo de recursos de destino](#deploy-to-target-resource-group).

### <a name="scope-to-resource-group-in-same-subscription"></a>Ámbito en el grupo de recursos de la misma suscripción

Para implementar recursos en otro grupo de recursos de la misma suscripción, agregue una implementación anidada e incluya la propiedad `resourceGroup`. Si no se especifica el grupo de recursos o el identificador de la suscripción, se utilizan los de la plantilla primaria. Todos los grupos de recursos deben existir antes de que se ejecute la implementación.

En el ejemplo siguiente, la implementación anidada tiene como destino un grupo de recursos denominado `demoResourceGroup`.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/same-sub-to-resource-group.json" highlight="9,13":::

Para ver una plantilla de ejemplo, consulte [Implementación en varios grupos de recursos](#deploy-to-multiple-resource-groups).

### <a name="scope-to-resource-group-in-different-subscription"></a>Ámbito en el grupo de recursos de otra suscripción

Para implementar recursos en un grupo de recursos de otra suscripción, agregue una implementación anidada e incluya las propiedades `subscriptionId` y `resourceGroup`. En el ejemplo siguiente, la implementación anidada tiene como destino un grupo de recursos denominado `demoResourceGroup`.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/different-sub-to-resource-group.json" highlight="9,10,14":::

Para ver una plantilla de ejemplo, consulte [Implementación en varios grupos de recursos](#deploy-to-multiple-resource-groups).

### <a name="scope-to-subscription"></a>Ámbito de la suscripción

Para implementar recursos en una suscripción, agregue una implementación anidada e incluya la propiedad `subscriptionId`. La suscripción puede ser la del grupo de recursos de destino, o cualquier otra suscripción del inquilino. Además, establezca la propiedad `location` para la implementación anidada.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/resource-group-to-subscription.json" highlight="9,10,14":::

Para ver una plantilla de ejemplo, consulte [Creación de un grupo de recursos](#create-resource-group).

### <a name="scope-to-tenant"></a>Ámbito del inquilino

Para crear recursos en el inquilino, establezca `scope` en `/`. El usuario que implementa la plantilla debe tener el [acceso necesario para realizar implementaciones en el inquilino](deploy-to-tenant.md#required-access).

Para usar una implementación anidada, establezca `scope` y `location`.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/resource-group-to-tenant.json" highlight="9,10,14":::

O bien, puede establecer el ámbito en `/` para algunos tipos de recursos, como los grupos de administración.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/resource-group-create-mg.json" highlight="12,15":::

Para más información, consulte [Grupo de administración](deploy-to-management-group.md#management-group).

## <a name="deploy-to-target-resource-group"></a>Implementación en el grupo de recursos de destino

Para implementar recursos en el grupo de recursos de destino, defina esos recursos en la sección `resources` de la plantilla. La plantilla siguiente crea una cuenta de almacenamiento en el grupo de recursos que se especifica en la operación de implementación.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-outputs/azuredeploy.json":::

## <a name="deploy-to-multiple-resource-groups"></a>Implementación de varios grupos de recursos

Puede implementar en más de un grupo de recursos en una misma plantilla de Resource Manager. Para establecer como destino un grupo de recursos diferente al de la plantilla primaria, use una [plantilla anidada o vinculada](linked-templates.md). En el tipo de recurso de implementación, especifique los valores para el identificador de suscripción y el grupo de recursos en que quiere realizar la implementación de la plantilla anidada. Los grupos de recursos pueden existir en suscripciones distintas.

> [!NOTE]
> Puede implementar hasta **800 grupos de recursos** en una única implementación. Normalmente, esta limitación significa que puede implementar en un grupo de recursos especificado para la plantilla primaria y hasta en 799 grupos de recursos en implementaciones anidadas o vinculadas. Sin embargo, si la plantilla principal contiene solo plantillas anidadas o vinculadas y no implementa por sí misma ningún recurso, puede incluir hasta 800 grupos de recursos en las implementaciones anidadas o vinculadas.

En el ejemplo siguiente se implementan dos cuentas de almacenamiento. La primera cuenta de almacenamiento se implementa en el grupo de recursos especificado en la operación de implementación. La segunda cuenta de almacenamiento se implementa en el grupo de recursos especificado en los parámetros `secondResourceGroup` y `secondSubscriptionID`:

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/crosssubscription.json":::

Si establece `resourceGroup` en el nombre de un grupo de recursos que no existe, la implementación generará un error.

Para probar la plantilla anterior y ver los resultados, use PowerShell o la CLI de Azure.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para implementar dos cuentas de almacenamiento en dos grupos de recursos de la **misma suscripción**, use:

```azurepowershell-interactive
$firstRG = "primarygroup"
$secondRG = "secondarygroup"

New-AzResourceGroup -Name $firstRG -Location southcentralus
New-AzResourceGroup -Name $secondRG -Location eastus

New-AzResourceGroupDeployment `
  -ResourceGroupName $firstRG `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crosssubscription.json `
  -storagePrefix storage `
  -secondResourceGroup $secondRG `
  -secondStorageLocation eastus
```

Para implementar dos cuentas de almacenamiento en **dos suscripciones**, use:

```azurepowershell-interactive
$firstRG = "primarygroup"
$secondRG = "secondarygroup"

$firstSub = "<first-subscription-id>"
$secondSub = "<second-subscription-id>"

Set-AzContext -Subscription $secondSub
New-AzResourceGroup -Name $secondRG -Location eastus

Set-AzContext -Subscription $firstSub
New-AzResourceGroup -Name $firstRG -Location southcentralus

New-AzResourceGroupDeployment `
  -ResourceGroupName $firstRG `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crosssubscription.json `
  -storagePrefix storage `
  -secondResourceGroup $secondRG `
  -secondStorageLocation eastus `
  -secondSubscriptionID $secondSub
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para implementar dos cuentas de almacenamiento en dos grupos de recursos de la **misma suscripción**, use:

```azurecli-interactive
firstRG="primarygroup"
secondRG="secondarygroup"

az group create --name $firstRG --location southcentralus
az group create --name $secondRG --location eastus
az deployment group create \
  --name ExampleDeployment \
  --resource-group $firstRG \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crosssubscription.json \
  --parameters storagePrefix=tfstorage secondResourceGroup=$secondRG secondStorageLocation=eastus
```

Para implementar dos cuentas de almacenamiento en **dos suscripciones**, use:

```azurecli-interactive
firstRG="primarygroup"
secondRG="secondarygroup"

firstSub="<first-subscription-id>"
secondSub="<second-subscription-id>"

az account set --subscription $secondSub
az group create --name $secondRG --location eastus

az account set --subscription $firstSub
az group create --name $firstRG --location southcentralus

az deployment group create \
  --name ExampleDeployment \
  --resource-group $firstRG \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crosssubscription.json \
  --parameters storagePrefix=storage secondResourceGroup=$secondRG secondStorageLocation=eastus secondSubscriptionID=$secondSub
```

---

## <a name="create-resource-group"></a>Creación de un grupo de recursos

En una implementación de grupo de recursos, puede cambiar al nivel de una suscripción y crear un grupo de recursos. La siguiente plantilla implementa una cuenta de almacenamiento en el grupo de recursos de destino y crea un grupo de recursos nuevo en la suscripción especificada.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storagePrefix": {
            "type": "string",
            "maxLength": 11
        },
        "newResourceGroupName": {
            "type": "string"
        },
        "nestedSubscriptionID": {
            "type": "string"
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "variables": {
        "storageName": "[concat(parameters('storagePrefix'), uniqueString(resourceGroup().id))]"
    },
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "name": "[variables('storageName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {
            }
        },
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2020-06-01",
            "name": "demoSubDeployment",
            "location": "westus",
            "subscriptionId": "[parameters('nestedSubscriptionID')]",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "type": "Microsoft.Resources/resourceGroups",
                            "apiVersion": "2020-10-01",
                            "name": "[parameters('newResourceGroupName')]",
                            "location": "[parameters('location')]",
                            "properties": {}
                        }
                    ],
                    "outputs": {}
                }
            }
        }
    ]
}
```

## <a name="next-steps"></a>Pasos siguientes

* Para un ejemplo de implementación de la configuración del área de trabajo para Azure Security Center, consulte [deployASCwithWorkspaceSettings.json](https://github.com/krnese/AzureDeploy/blob/master/ARM/deployments/deployASCwithWorkspaceSettings.json).
