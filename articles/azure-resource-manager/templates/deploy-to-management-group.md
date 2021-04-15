---
title: Implementación de recursos en el grupo de administración
description: Se describe cómo implementar recursos en el ámbito de un grupo de administración en una plantilla de Azure Resource Manager.
ms.topic: conceptual
ms.date: 03/18/2021
ms.openlocfilehash: dc7418d9e93fb50590c5e2502b3a3ffb3847273f
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105043315"
---
# <a name="management-group-deployments-with-arm-templates"></a>Implementaciones de grupos de administración con plantillas de Resource Manager

A medida que la organización madura, puede implementar una plantilla de Azure Resource Manager (plantilla de ARM) para crear recursos a nivel de grupo de administración. Por ejemplo, es posible que deba definir y asignar [directivas](../../governance/policy/overview.md) o el [control de acceso basado en rol de Azure (RBAC de Azure)](../../role-based-access-control/overview.md) para un grupo de administración. Con las plantillas de nivel de grupo de administración, puede aplicar directivas y asignar roles mediante declaración en el nivel de grupo de administración.

## <a name="supported-resources"></a>Recursos compatibles

No todos los tipos de recursos se pueden implementar en el nivel de grupo de administración. En esta sección se enumeran los tipos de recursos que se admiten.

Para Azure Blueprints, use:

* [artifacts](/azure/templates/microsoft.blueprint/blueprints/artifacts)
* [blueprints](/azure/templates/microsoft.blueprint/blueprints)
* [blueprintAssignments](/azure/templates/microsoft.blueprint/blueprintassignments)
* [versions](/azure/templates/microsoft.blueprint/blueprints/versions)

En Azure Policy, use:

* [policyAssignments](/azure/templates/microsoft.authorization/policyassignments)
* [policyDefinitions](/azure/templates/microsoft.authorization/policydefinitions)
* [policySetDefinitions](/azure/templates/microsoft.authorization/policysetdefinitions)
* [remediations](/azure/templates/microsoft.policyinsights/remediations)

Para el control de acceso basado en rol de Azure (Azure RBAC), use:

* [roleAssignments](/azure/templates/microsoft.authorization/roleassignments)
* [roleDefinitions](/azure/templates/microsoft.authorization/roledefinitions)

Para plantillas anidadas que se implementan en suscripciones o grupos de recursos, use:

* [deployments](/azure/templates/microsoft.resources/deployments)

Para administrar los recursos, use:

* [etiquetas](/azure/templates/microsoft.resources/tags)

Los grupos de administración son recursos de nivel de inquilino. Sin embargo, puede crear grupos de administración en una implementación de tales grupos estableciendo el ámbito del nuevo grupo en el inquilino. Consulte [Grupo de administración](#management-group).

## <a name="schema"></a>Schema

El esquema que se usa para las implementaciones de nivel de grupo de administración es diferente del esquema de las implementaciones de grupo de recursos.

Para las plantillas, use:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/managementGroupDeploymentTemplate.json#",
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

Para implementar en un grupo de administración, use los comandos de implementación de grupos de administración.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para la CLI de Azure, use [az deployment mg create](/cli/azure/deployment/mg#az-deployment-mg-create):

```azurecli-interactive
az deployment mg create \
  --name demoMGDeployment \
  --location WestUS \
  --management-group-id myMG \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/management-level-deployment/azuredeploy.json"
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

En el caso de Azure PowerShell, use [New-AzManagementGroupDeployment](/powershell/module/az.resources/new-azmanagementgroupdeployment).

```azurepowershell-interactive
New-AzManagementGroupDeployment `
  -Name demoMGDeployment `
  -Location "West US" `
  -ManagementGroupId "myMG" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/management-level-deployment/azuredeploy.json"
```

---

Para obtener información más detallada sobre los comandos de implementación y las opciones para implementar plantillas de Resource Manager, consulte:

* [Implementación de recursos con Azure Portal y plantillas de Resource Manager](deploy-portal.md)
* [Implementación de recursos con plantillas de ARM y la CLI de Azure](deploy-cli.md)
* [Implementación de recursos con las plantillas de ARM y Azure PowerShell](deploy-powershell.md)
* [Implementación de recursos con plantillas de Resource Manager y la API REST de Azure Resource Manager](deploy-rest.md)
* [Usar un botón de implementación para implementar plantillas desde el repositorio de GitHub](deploy-to-azure-button.md)
* [Implementación de plantillas de Resource Manager desde Cloud Shell](deploy-cloud-shell.md)

## <a name="deployment-location-and-name"></a>Ubicación y nombre de la implementación

En el caso de las implementaciones de nivel de grupo de administración, debe proporcionar una ubicación para la implementación. La ubicación de la implementación es independiente de la ubicación de los recursos que se implementan. La ubicación de implementación especifica dónde se almacenarán los datos de la implementación. Las implementaciones de [suscripción](deploy-to-subscription.md) e [inquilino](deploy-to-tenant.md) también requieren una ubicación. En las implementaciones de [grupo de recursos](deploy-to-resource-group.md), la ubicación del grupo de recursos se usa para almacenar los datos de implementación.

Puede proporcionar un nombre para la implementación o usar el nombre de implementación predeterminado. El nombre predeterminado es el nombre del archivo de plantilla. Por ejemplo, al implementar una plantilla llamada _azuredeploy.json_, se crea un nombre de predeterminado **azuredeploy**.

Para cada nombre de implementación, la ubicación es inmutable. No se puede crear una implementación en una ubicación si ya existe una implementación con el mismo nombre en otra ubicación. Por ejemplo, si crea una implementación de grupo de administración con el nombre **deployment1** en **centralus**, no podrá crear otra implementación con el nombre **deployment1**, sino una ubicación de **westus**. Si recibe el código de error `InvalidDeploymentLocation`, use un nombre diferente o utilice la ubicación de la implementación anterior que tenía ese mismo nombre.

## <a name="deployment-scopes"></a>Ámbitos de implementación

Al implementar en un grupo de administración, puede implementar los recursos en:

* el grupo de administración de destino de la operación
* otro grupo de administración en el inquilino
* suscripciones en el grupo de administración
* grupos de recursos en el grupo de administración
* el inquilino del grupo de recursos

Un [recurso de extensión](scope-extension-resources.md) se puede limitar a un destino distinto del destino de implementación.

El usuario que implementa la plantilla debe tener acceso al ámbito especificado.

En esta sección se muestra cómo especificar distintos ámbitos. Puede combinar estos distintos ámbitos en una sola plantilla.

### <a name="scope-to-target-management-group"></a>Ámbito del grupo de administración de destino

Los recursos definidos en la sección de recursos de la plantilla se aplican al grupo de administración del comando de implementación.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/default-mg.json" highlight="5":::

### <a name="scope-to-another-management-group"></a>Ámbito de otro grupo de administración

Para establecer como destino otro grupo de administración, agregue una implementación anidada y especifique la propiedad `scope`. Establezca la propiedad `scope` en un valor con el formato `Microsoft.Management/managementGroups/<mg-name>`.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/scope-mg.json" highlight="10,17,18,22":::

### <a name="scope-to-subscription"></a>Ámbito de la suscripción

También puede establecer como destino suscripciones con un grupo de administración. El usuario que implementa la plantilla debe tener acceso al ámbito especificado.

Para establecer como destino una suscripción dentro del grupo de administración, use una implementación anidada y la propiedad `subscriptionId`.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/mg-to-subscription.json" highlight="9,10,18":::

### <a name="scope-to-resource-group"></a>Ámbito del grupo de recursos

También puede dirigirse a los grupos de recursos dentro del grupo de administración. El usuario que implementa la plantilla debe tener acceso al ámbito especificado.

Para establecer como destino un grupo de recursos dentro del grupo de administración, use una implementación anidada. Establezca las propiedades `subscriptionId` y `resourceGroup`. No establezca una ubicación para la implementación anidada porque se implementa en la ubicación del grupo de recursos.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/mg-to-resource-group.json" highlight="9,10,18":::

Para usar la implementación de un grupo de administración para crear un grupo de recursos dentro de una suscripción e implementar una cuenta de almacenamiento en ese grupo de recursos, vea [Implementación a una suscripción y a un grupo de recursos](#deploy-to-subscription-and-resource-group).

### <a name="scope-to-tenant"></a>Ámbito del inquilino

Para crear recursos en el inquilino, establezca `scope` en `/`. El usuario que implementa la plantilla debe tener el [acceso necesario para realizar implementaciones en el inquilino](deploy-to-tenant.md#required-access).

Para usar una implementación anidada, establezca `scope` y `location`.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/management-group-to-tenant.json" highlight="9,10,14":::

O bien, puede establecer el ámbito en `/` para algunos tipos de recursos, como los grupos de administración. La creación de un grupo de administración se describe en la sección siguiente.

## <a name="management-group"></a>Grupo de administración

Para crear un grupo de administración en una implementación de grupos de administración, debe establecer el ámbito en `/` para el grupo.

En el ejemplo siguiente se crea un grupo de administración en el grupo de administración raíz.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/management-group-create-mg.json" highlight="12,15":::

En el ejemplo siguiente se crea un grupo de administración en el grupo de administración especificado como primario. Observe que el ámbito se establece en `/`.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/managementGroupDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "mgName": {
            "type": "string",
            "defaultValue": "[concat('mg-', uniqueString(newGuid()))]"
        },
        "parentMG": {
            "type": "string"
        }
    },
    "resources": [
        {
            "name": "[parameters('mgName')]",
            "type": "Microsoft.Management/managementGroups",
            "apiVersion": "2020-05-01",
            "scope": "/",
            "location": "eastus",
            "properties": {
                "details": {
                    "parent": {
                        "id": "[tenantResourceId('Microsoft.Management/managementGroups', parameters('parentMG'))]"
                    }
                }
            }
        }
    ],
    "outputs": {
        "output": {
            "type": "string",
            "value": "[parameters('mgName')]"
        }
    }
}
```

## <a name="subscriptions"></a>Suscripciones

Para usar una plantilla de ARM para crear una nueva suscripción de Azure en un grupo de administración, consulte:

* [Creación de suscripciones de Contrato Enterprise de Azure mediante programación con las API más recientes](../../cost-management-billing/manage/programmatically-create-subscription-enterprise-agreement.md)
* [Creación de suscripciones de Azure mediante programación para el Contrato de cliente de Microsoft con las API más recientes](../../cost-management-billing/manage/programmatically-create-subscription-microsoft-customer-agreement.md)
* [Creación de suscripciones de Azure mediante programación para Microsoft Partner Agreement con las API más recientes](../../cost-management-billing/manage/programmatically-create-subscription-microsoft-partner-agreement.md)

Para implementar una plantilla que mueve una suscripción de Azure existente a un nuevo grupo de administración, consulte [Traslado de las suscripciones de la plantilla de ARM](../../governance/management-groups/manage.md#move-subscriptions-in-arm-template).

## <a name="azure-policy"></a>Azure Policy

Las definiciones de directivas personalizadas que se implementan en un grupo de administración son extensiones del grupo de administración. Para obtener el identificador de una definición de directiva personalizada, use la función [extensionResourceId()](template-functions-resource.md#extensionresourceid). Las definiciones de directivas integradas son recursos del nivel de inquilino. Para obtener el identificador de una definición de directiva integrada, use la función [tenantResourceId](template-functions-resource.md#tenantresourceid).

En el ejemplo siguiente se muestra cómo [definir](../../governance/policy/concepts/definition-structure.md) una directiva en el nivel de grupo de administración y cómo asignarla.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/managementGroupDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "targetMG": {
            "type": "string",
            "metadata": {
                "description": "Target Management Group"
            }
        },
        "allowedLocations": {
            "type": "array",
            "defaultValue": [
                "australiaeast",
                "australiasoutheast",
                "australiacentral"
            ],
            "metadata": {
                "description": "An array of the allowed locations, all other locations will be denied by the created policy."
            }
        }
    },
    "variables": {
        "mgScope": "[tenantResourceId('Microsoft.Management/managementGroups', parameters('targetMG'))]",
        "policyDefinition": "LocationRestriction"
    },
    "resources": [
        {
            "type": "Microsoft.Authorization/policyDefinitions",
            "name": "[variables('policyDefinition')]",
            "apiVersion": "2019-09-01",
            "properties": {
                "policyType": "Custom",
                "mode": "All",
                "parameters": {
                },
                "policyRule": {
                    "if": {
                        "not": {
                            "field": "location",
                            "in": "[parameters('allowedLocations')]"
                        }
                    },
                    "then": {
                        "effect": "deny"
                    }
                }
            }
        },
        {
            "type": "Microsoft.Authorization/policyAssignments",
            "name": "location-lock",
            "apiVersion": "2019-09-01",
            "dependsOn": [
                "[variables('policyDefinition')]"
            ],
            "properties": {
                "scope": "[variables('mgScope')]",
                "policyDefinitionId": "[extensionResourceId(variables('mgScope'), 'Microsoft.Authorization/policyDefinitions', variables('policyDefinition'))]"
            }
        }
    ]
}
```

## <a name="deploy-to-subscription-and-resource-group"></a>Implementación a una suscripción y a un grupo de recursos

En una implementación de nivel de grupo de administración, puede establecer como destino una suscripción dentro del grupo de administración. En el ejemplo siguiente se crea un grupo de recursos en una suscripción y se implementa una cuenta de almacenamiento en el grupo de recursos.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/managementGroupDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "nestedsubId": {
            "type": "string"
        },
        "nestedRG": {
            "type": "string"
        },
        "storageAccountName": {
            "type": "string"
        },
        "nestedLocation": {
            "type": "string"
        }
    },
    "resources": [
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2020-10-01",
            "name": "nestedSub",
            "location": "[parameters('nestedLocation')]",
            "subscriptionId": "[parameters('nestedSubId')]",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {
                    },
                    "variables": {
                    },
                    "resources": [
                        {
                            "type": "Microsoft.Resources/resourceGroups",
                            "apiVersion": "2020-10-01",
                            "name": "[parameters('nestedRG')]",
                            "location": "[parameters('nestedLocation')]"
                        }
                    ]
                }
            }
        },
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2020-10-01",
            "name": "nestedRG",
            "subscriptionId": "[parameters('nestedSubId')]",
            "resourceGroup": "[parameters('nestedRG')]",
            "dependsOn": [
                "nestedSub"
            ],
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "resources": [
                        {
                            "type": "Microsoft.Storage/storageAccounts",
                            "apiVersion": "2019-04-01",
                            "name": "[parameters('storageAccountName')]",
                            "location": "[parameters('nestedLocation')]",
                            "kind": "StorageV2",
                            "sku": {
                                "name": "Standard_LRS"
                            }
                        }
                    ]
                }
            }
        }
    ]
}
```

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre la asignación de roles, consulte [Incorporación de asignaciones de roles mediante plantillas de Azure Resource Manager](../../role-based-access-control/role-assignments-template.md).
* Para un ejemplo de implementación de la configuración del área de trabajo para Azure Security Center, consulte [deployASCwithWorkspaceSettings.json](https://github.com/krnese/AzureDeploy/blob/master/ARM/deployments/deployASCwithWorkspaceSettings.json).
* También puede implementar plantillas en el [nivel de suscripción](deploy-to-subscription.md) y en el [nivel de inquilino](deploy-to-tenant.md).
