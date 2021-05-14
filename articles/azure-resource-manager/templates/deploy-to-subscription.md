---
title: Implementación de recursos en una suscripción
description: Se describe cómo crear un grupo de recursos en una plantilla de Azure Resource Manager. También se muestra cómo implementar recursos en el ámbito de la suscripción de Azure.
ms.topic: conceptual
ms.date: 01/13/2021
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: a2b9fedbd2916953b0ff2166bc7fddb5f877ee07
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109754093"
---
# <a name="subscription-deployments-with-arm-templates"></a>Implementaciones de suscripción con plantillas de Resource Manager

Para simplificar la administración de recursos, puede usar una plantilla de Azure Resource Manager (plantilla de ARM) para implementar recursos en el nivel de la suscripción de Azure. Por ejemplo, puede implementar [directivas](../../governance/policy/overview.md) y el [control de acceso basado en rol (RBAC de Azure)](../../role-based-access-control/overview.md) en su suscripción, y se aplicarán a ella en su totalidad. También puede crear grupos de recursos dentro de la suscripción e implementar recursos en grupos de recursos de la suscripción.

> [!NOTE]
> Puede implementar en 800 grupos de recursos distintos en una implementación de nivel de suscripción.

Para implementar plantillas en el nivel de suscripción, use la CLI de Azure, PowerShell, la API de REST o el portal.

## <a name="supported-resources"></a>Recursos compatibles

No todos los tipos de recursos se pueden implementar en el nivel de suscripción. En esta sección se enumeran los tipos de recursos que se admiten.

Para Azure Blueprints, use:

* [artifacts](/azure/templates/microsoft.blueprint/blueprints/artifacts)
* [blueprints](/azure/templates/microsoft.blueprint/blueprints)
* [blueprintAssignments](/azure/templates/microsoft.blueprint/blueprintassignments)
* [versions (Blueprints)](/azure/templates/microsoft.blueprint/blueprints/versions)

Para las directivas de Azure, use:

* [policyAssignments](/azure/templates/microsoft.authorization/policyassignments)
* [policyDefinitions](/azure/templates/microsoft.authorization/policydefinitions)
* [policySetDefinitions](/azure/templates/microsoft.authorization/policysetdefinitions)
* [remediations](/azure/templates/microsoft.policyinsights/remediations)

Para el control de acceso basado en rol de Azure (Azure RBAC), use:

* [roleAssignments](/azure/templates/microsoft.authorization/roleassignments)
* [roleDefinitions](/azure/templates/microsoft.authorization/roledefinitions)

Para plantillas anidadas que se implementan en grupos de recursos, use:

* [deployments](/azure/templates/microsoft.resources/deployments)

Para crear grupos de recursos, use:

* [resourceGroups](/azure/templates/microsoft.resources/resourcegroups)

Para administrar su suscripción, use:

* [Configuraciones de Advisor](/azure/templates/microsoft.advisor/configurations)
* [budgets](/azure/templates/microsoft.consumption/budgets)
* [Perfil de Change Analysis](/azure/templates/microsoft.changeanalysis/profile)
* [supportPlanTypes](/azure/templates/microsoft.addons/supportproviders/supportplantypes)
* [etiquetas](/azure/templates/microsoft.resources/tags)

Otros tipos admitidos incluyen:

* [scopeAssignments](/azure/templates/microsoft.managednetwork/scopeassignments)
* [eventSubscriptions](/azure/templates/microsoft.eventgrid/eventsubscriptions)
* [peerAsns](/azure/templates/microsoft.peering/2019-09-01-preview/peerasns)

## <a name="schema"></a>Schema

El esquema que se usa para las implementaciones de nivel de suscripción es diferente del esquema de las implementaciones de grupo de recursos.

Para las plantillas, use:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
    ...
}
```

El esquema de un archivo de parámetros es el mismo para todos los ámbitos de implementación. Para los archivos de parámetros, use:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    ...
}
```

## <a name="deployment-commands"></a>Comandos de implementación

Para implementar en una suscripción, use los comandos de implementación de nivel de suscripción.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para la CLI de Azure, use [az deployment sub create](/cli/azure/deployment/sub#az_deployment_sub_create). El ejemplo siguiente implementa una plantilla para crear un grupo de recursos:

```azurecli-interactive
az deployment sub create \
  --name demoSubDeployment \
  --location centralus \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/emptyRG.json" \
  --parameters rgName=demoResourceGroup rgLocation=centralus
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para el comando de implementación de PowerShell, use [New-AzDeployment](/powershell/module/az.resources/new-azdeployment) o su alias `New-AzSubscriptionDeployment`. El ejemplo siguiente implementa una plantilla para crear un grupo de recursos:

```azurepowershell-interactive
New-AzSubscriptionDeployment `
  -Name demoSubDeployment `
  -Location centralus `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/emptyRG.json" `
  -rgName demoResourceGroup `
  -rgLocation centralus
```

---

Para obtener información más detallada sobre los comandos y las opciones de implementación para la implementación de plantillas de Resource Manager, vea:

* [Implementación de recursos con Azure Portal y plantillas de Resource Manager](deploy-portal.md)
* [Implementación de recursos con plantillas de ARM y la CLI de Azure](deploy-cli.md)
* [Implementación de recursos con las plantillas de ARM y Azure PowerShell](deploy-powershell.md)
* [Implementación de recursos con plantillas de Resource Manager y la API REST de Azure Resource Manager](deploy-rest.md)
* [Usar un botón de implementación para implementar plantillas desde el repositorio de GitHub](deploy-to-azure-button.md)
* [Implementación de plantillas de Resource Manager desde Cloud Shell](deploy-cloud-shell.md)

## <a name="deployment-location-and-name"></a>Ubicación y nombre de la implementación

En el caso de las implementaciones de nivel de suscripción, debe proporcionar una ubicación para la implementación. La ubicación de la implementación es independiente de la ubicación de los recursos que se implementan. La ubicación de implementación especifica dónde se almacenarán los datos de la implementación. Las implementaciones de [grupo de administración](deploy-to-management-group.md) e [inquilino](deploy-to-tenant.md) también requieren una ubicación. En las implementaciones de [grupo de recursos](deploy-to-resource-group.md), la ubicación del grupo de recursos se usa para almacenar los datos de implementación.

Puede proporcionar un nombre para la implementación o usar el nombre de implementación predeterminado. El nombre predeterminado es el nombre del archivo de plantilla. Por ejemplo, al implementar una plantilla llamada _azuredeploy.json_, se crea un nombre de predeterminado **azuredeploy**.

Para cada nombre de implementación, la ubicación es inmutable. No se puede crear una implementación en una ubicación si ya existe una implementación con el mismo nombre en otra ubicación. Por ejemplo, si crea una implementación de suscripción con el nombre **deployment1** en **centralus**, no podrá crear otra implementación con el nombre **deployment1**, sino una ubicación de **westus**. Si recibe el código de error `InvalidDeploymentLocation`, use un nombre diferente o utilice la ubicación de la implementación anterior que tenía ese mismo nombre.

## <a name="deployment-scopes"></a>Ámbitos de implementación

Al implementar en una suscripción, puede implementar los recursos en:

* la suscripción de destino de la operación
* cualquier suscripción en el inquilino
* grupos de recursos en la suscripción o en otras
* inquilino para la suscripción

Un [recurso de extensión](scope-extension-resources.md) se puede aplicar al ámbito de un destino distinto del destino de implementación.

El usuario que implementa la plantilla debe tener acceso al ámbito especificado.

En esta sección se muestra cómo especificar distintos ámbitos. Puede combinar estos distintos ámbitos en una sola plantilla.

### <a name="scope-to-target-subscription"></a>Ámbito de la suscripción de destino

Para implementar recursos en la suscripción de destino, agregue esos recursos a la sección de recursos de la plantilla.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/default-sub.json" highlight="5":::

Para obtener ejemplos de cómo implementar en la suscripción, vea [Creación de grupos de recursos](#create-resource-groups) y [Asignación de definición de directiva](#assign-policy-definition).

### <a name="scope-to-other-subscription"></a>Ámbito de otra suscripción

Para implementar recursos en una suscripción diferente de la suscripción de la operación, agregue una implementación anidada. Establezca la propiedad `subscriptionId` en el id. de la suscripción en la que quiere realizar la implementación. Establezca la propiedad `location` para la implementación anidada.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/sub-to-sub.json" highlight="9,10,14":::

### <a name="scope-to-resource-group"></a>Ámbito del grupo de recursos

Para implementar recursos en un grupo de recursos dentro de la suscripción, agregue una implementación anidada e incluya la propiedad `resourceGroup`. En el ejemplo siguiente, la implementación anidada tiene como destino un grupo de recursos denominado `demoResourceGroup`.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/sub-to-resource-group.json" highlight="9,13":::

Para obtener un ejemplo de cómo implementar en un grupo de recursos, vea [Creación de un grupo de recursos y recursos](#create-resource-group-and-resources).

### <a name="scope-to-tenant"></a>Ámbito del inquilino

Para crear recursos en el inquilino, establezca `scope` en `/`. El usuario que implementa la plantilla debe tener el [acceso necesario para realizar implementaciones en el inquilino](deploy-to-tenant.md#required-access).

Para usar una implementación anidada, establezca `scope` y `location`.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/subscription-to-tenant.json" highlight="9,10,14":::

O bien, puede establecer el ámbito en `/` para algunos tipos de recursos, como los grupos de administración.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/subscription-create-mg.json" highlight="12,15":::

Para más información, consulte [Grupo de administración](deploy-to-management-group.md#management-group).

## <a name="resource-groups"></a>Grupos de recursos

### <a name="create-resource-groups"></a>Crear grupos de recursos

Para crear un grupo de recursos en una plantilla de ARM, defina un recurso [Microsoft.Resources/resourceGroups](/azure/templates/microsoft.resources/allversions) con un nombre y una ubicación para el grupo de recursos.

En la plantilla siguiente se crea un grupo de recursos vacío.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "rgName": {
      "type": "string"
    },
    "rgLocation": {
      "type": "string"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Resources/resourceGroups",
      "apiVersion": "2020-10-01",
      "name": "[parameters('rgName')]",
      "location": "[parameters('rgLocation')]",
      "properties": {}
    }
  ],
  "outputs": {}
}
```

Use el [elemento copy](copy-resources.md) con grupos de recursos para crear más de un grupo de recursos.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "rgNamePrefix": {
      "type": "string"
    },
    "rgLocation": {
      "type": "string"
    },
    "instanceCount": {
      "type": "int"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Resources/resourceGroups",
      "apiVersion": "2020-10-01",
      "location": "[parameters('rgLocation')]",
      "name": "[concat(parameters('rgNamePrefix'), copyIndex())]",
      "copy": {
        "name": "rgCopy",
        "count": "[parameters('instanceCount')]"
      },
      "properties": {}
    }
  ],
  "outputs": {}
}
```

Para más información sobre la iteración de recursos, consulte [Iteración de recursos en las plantillas de ARM](./copy-resources.md) y [Tutorial: Creación de varias instancias de recursos con plantillas de Resource Manager](./template-tutorial-create-multiple-instances.md).

### <a name="create-resource-group-and-resources"></a>Creación del grupo de recursos y los recursos

Para crear el grupo de recursos e implementar recursos en él, utilice una plantilla anidada. La plantilla anidada define los recursos que se van a implementar en el grupo de recursos. Establezca la plantilla anidada como dependiente del grupo de recursos para asegurarse de que el grupo de recursos existe antes de implementar los recursos. Puede implementar en hasta 800 grupos de recursos.

En el ejemplo siguiente se crea un grupo de recursos y se implementa una cuenta de almacenamiento en el grupo de recursos.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "rgName": {
      "type": "string"
    },
    "rgLocation": {
      "type": "string"
    },
    "storagePrefix": {
      "type": "string",
      "maxLength": 11
    }
  },
  "variables": {
    "storageName": "[concat(parameters('storagePrefix'), uniqueString(subscription().id, parameters('rgName')))]"
  },
  "resources": [
    {
      "type": "Microsoft.Resources/resourceGroups",
      "apiVersion": "2020-10-01",
      "name": "[parameters('rgName')]",
      "location": "[parameters('rgLocation')]",
      "properties": {}
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-10-01",
      "name": "storageDeployment",
      "resourceGroup": "[parameters('rgName')]",
      "dependsOn": [
        "[resourceId('Microsoft.Resources/resourceGroups/', parameters('rgName'))]"
      ],
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {},
          "variables": {},
          "resources": [
            {
              "type": "Microsoft.Storage/storageAccounts",
              "apiVersion": "2019-06-01",
              "name": "[variables('storageName')]",
              "location": "[parameters('rgLocation')]",
              "sku": {
                "name": "Standard_LRS"
              },
              "kind": "StorageV2"
            }
          ],
          "outputs": {}
        }
      }
    }
  ],
  "outputs": {}
}
```

## <a name="azure-policy"></a>Azure Policy

### <a name="assign-policy-definition"></a>Asignación de una definición de directiva

En el ejemplo siguiente se asigna una definición de directiva existente a la suscripción. Si la definición de directiva toma parámetros, proporciónelos como un objeto. Si la definición de directiva no toma parámetros, use el objeto vacío predeterminado.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "policyDefinitionID": {
      "type": "string"
    },
    "policyName": {
      "type": "string"
    },
    "policyParameters": {
      "type": "object",
      "defaultValue": {}
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Authorization/policyAssignments",
      "apiVersion": "2018-03-01",
      "name": "[parameters('policyName')]",
      "properties": {
        "scope": "[subscription().id]",
        "policyDefinitionId": "[parameters('policyDefinitionID')]",
        "parameters": "[parameters('policyParameters')]"
      }
    }
  ]
}
```

Para implementar esta plantilla con la CLI de Azure, use:

```azurecli-interactive
# Built-in policy definition that accepts parameters
definition=$(az policy definition list --query "[?displayName=='Allowed locations'].id" --output tsv)

az deployment sub create \
  --name demoDeployment \
  --location centralus \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/policyassign.json" \
  --parameters policyDefinitionID=$definition policyName=setLocation policyParameters="{'listOfAllowedLocations': {'value': ['westus']} }"
```

Para implementar esta plantilla con PowerShell, use:

```azurepowershell-interactive
$definition = Get-AzPolicyDefinition | Where-Object { $_.Properties.DisplayName -eq 'Allowed locations' }

$locations = @("westus", "westus2")
$policyParams =@{listOfAllowedLocations = @{ value = $locations}}

New-AzSubscriptionDeployment `
  -Name policyassign `
  -Location centralus `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/policyassign.json" `
  -policyDefinitionID $definition.PolicyDefinitionId `
  -policyName setLocation `
  -policyParameters $policyParams
```

### <a name="create-and-assign-policy-definitions"></a>Creación y asignación de definiciones de directiva

Puede [definir](../../governance/policy/concepts/definition-structure.md) y asignar una definición de directiva en la misma plantilla.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Authorization/policyDefinitions",
      "apiVersion": "2018-05-01",
      "name": "locationpolicy",
      "properties": {
        "policyType": "Custom",
        "parameters": {},
        "policyRule": {
          "if": {
            "field": "location",
            "equals": "northeurope"
          },
          "then": {
            "effect": "deny"
          }
        }
      }
    },
    {
      "type": "Microsoft.Authorization/policyAssignments",
      "apiVersion": "2018-05-01",
      "name": "location-lock",
      "dependsOn": [
        "locationpolicy"
      ],
      "properties": {
        "scope": "[subscription().id]",
        "policyDefinitionId": "[subscriptionResourceId('Microsoft.Authorization/policyDefinitions', 'locationpolicy')]"
      }
    }
  ]
}
```

Para crear la definición de directiva en su suscripción y asignarla a la suscripción, use el siguiente comando de la CLI:

```azurecli
az deployment sub create \
  --name demoDeployment \
  --location centralus \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/policydefineandassign.json"
```

Para implementar esta plantilla con PowerShell, use:

```azurepowershell
New-AzSubscriptionDeployment `
  -Name definePolicy `
  -Location centralus `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/policydefineandassign.json"
```

## <a name="azure-blueprints"></a>Azure Blueprint

### <a name="create-blueprint-definition"></a>Creación de una definición de plano técnico

Puede [crear](../../governance/blueprints/tutorials/create-from-sample.md) una definición de plano técnico a partir de una plantilla.

:::code language="json" source="~/quickstart-templates/subscription-deployments/blueprints-new-blueprint/azuredeploy.json":::

Para crear la definición de plano técnico en su suscripción, use el siguiente comando de la CLI:

```azurecli
az deployment sub create \
  --name demoDeployment \
  --location centralus \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/subscription-deployments/blueprints-new-blueprint/azuredeploy.json"
```

Para implementar esta plantilla con PowerShell, use:

```azurepowershell
New-AzSubscriptionDeployment `
  -Name demoDeployment `
  -Location centralus `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/subscription-deployments/blueprints-new-blueprint/azuredeploy.json"
```

## <a name="access-control"></a>Control de acceso

Para más información sobre la asignación de roles, consulte [Incorporación de asignaciones de roles mediante plantillas de Azure Resource Manager](../../role-based-access-control/role-assignments-template.md).

En el ejemplo siguiente, se crea un grupo de recursos, se le aplica un bloqueo y se asigna un rol a una entidad de seguridad.

:::code language="json" source="~/quickstart-templates/subscription-deployments/create-rg-lock-role-assignment/azuredeploy.json":::

## <a name="next-steps"></a>Pasos siguientes

* Para un ejemplo de implementación de la configuración del área de trabajo para Azure Security Center, consulte [deployASCwithWorkspaceSettings.json](https://github.com/krnese/AzureDeploy/blob/master/ARM/deployments/deployASCwithWorkspaceSettings.json).
* Puede encontrar plantillas de ejemplo en [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/subscription-deployments).
* También puede implementar plantillas en el [nivel de grupo de administración](deploy-to-management-group.md) y en el [nivel de inquilino](deploy-to-tenant.md).
