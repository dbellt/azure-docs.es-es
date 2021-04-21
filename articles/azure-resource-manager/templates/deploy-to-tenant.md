---
title: Implementación de recursos en el inquilino
description: Se describe cómo implementar recursos en el ámbito de un inquilino en una plantilla de Azure Resource Manager.
ms.topic: conceptual
ms.date: 01/13/2021
ms.openlocfilehash: 0b17b8741d1701720de86d8039be3b6cd28ace5c
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107781908"
---
# <a name="tenant-deployments-with-arm-templates"></a>Implementaciones de inquilino con plantillas de Resource Manager

A medida que crece la organización, es posible que necesite definir y asignar [directivas](../../governance/policy/overview.md) o el [control de acceso basado en rol (RBAC de Azure)](../../role-based-access-control/overview.md) en el inquilino de Azure AD. Con las plantillas de nivel de inquilino, puede aplicar directivas y asignar roles a nivel global de forma declarativa.

## <a name="supported-resources"></a>Recursos compatibles

No todos los tipos de recursos se pueden implementar a nivel de inquilino. En esta sección se enumeran los tipos de recursos que se admiten.

Para las directivas de Azure, use:

* [policyAssignments](/azure/templates/microsoft.authorization/policyassignments)
* [policyDefinitions](/azure/templates/microsoft.authorization/policydefinitions)
* [policySetDefinitions](/azure/templates/microsoft.authorization/policysetdefinitions)

Para el control de acceso basado en rol de Azure (Azure RBAC), use:

* [roleAssignments](/azure/templates/microsoft.authorization/roleassignments)

Para plantillas anidadas que se implementan en grupos de administración, suscripciones o grupos de recursos, use:

* [deployments](/azure/templates/microsoft.resources/deployments)

Para crear grupos de administración, use:

* [managementGroups](/azure/templates/microsoft.management/managementgroups)

Para crear suscripciones, use:

* [alias](/azure/templates/microsoft.subscription/aliases)

Para la administración de costos, use:

* [billingProfiles](/azure/templates/microsoft.billing/billingaccounts/billingprofiles)
* [instructions](/azure/templates/microsoft.billing/billingaccounts/billingprofiles/instructions)
* [invoiceSections](/azure/templates/microsoft.billing/billingaccounts/billingprofiles/invoicesections)

Para configurar el portal, use:

* [tenantConfigurations](/azure/templates/microsoft.portal/tenantconfigurations)

## <a name="schema"></a>Schema

El esquema que se usa para las implementaciones de nivel de inquilino es diferente del esquema de las implementaciones de grupo de recursos.

Para las plantillas, use:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/tenantDeploymentTemplate.json#",
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

## <a name="required-access"></a>Acceso necesario

La entidad de seguridad que implementa la plantilla debe tener permisos para crear recursos en el ámbito del inquilino. La entidad de seguridad debe tener permiso para ejecutar las acciones de implementación (`Microsoft.Resources/deployments/*`) y para crear los recursos definidos en la plantilla. Por ejemplo, para crear un grupo de administración, la entidad de seguridad debe tener permiso de colaborador en el ámbito del inquilino. Para crear asignaciones de roles, la entidad de seguridad debe tener permiso de propietario.

El administrador global de Azure Active Directory no tiene permiso para asignar roles de forma automática. Para habilitar las implementaciones de plantilla en el ámbito del inquilino, el administrador global debe realizar los siguientes pasos:

1. Eleve el acceso a la cuenta para que el administrador global pueda asignar roles. Para más información, consulte [Elevación de los privilegios de acceso para administrar todas las suscripciones y los grupos de administración de Azure](../../role-based-access-control/elevate-access-global-admin.md).

1. Asigne el rol de propietario o colaborador a la entidad de seguridad que necesita implementar las plantillas.

   ```azurepowershell-interactive
   New-AzRoleAssignment -SignInName "[userId]" -Scope "/" -RoleDefinitionName "Owner"
   ```

   ```azurecli-interactive
   az role assignment create --assignee "[userId]" --scope "/" --role "Owner"
   ```

La entidad de seguridad tiene ahora los permisos necesarios para implementar la plantilla.

## <a name="deployment-commands"></a>Comandos de implementación

Los comandos para las implementaciones de inquilino son diferentes de los comandos para las implementaciones de grupos de recursos.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para la CLI de Azure, use [az deployment tenant create](/cli/azure/deployment/tenant#az_deployment_tenant_create):

```azurecli-interactive
az deployment tenant create \
  --name demoTenantDeployment \
  --location WestUS \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/tenant-deployments/new-mg/azuredeploy.json"
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

En el caso de Azure PowerShell, use [New-AzTenantDeployment](/powershell/module/az.resources/new-aztenantdeployment).

```azurepowershell-interactive
New-AzTenantDeployment `
  -Name demoTenantDeployment `
  -Location "West US" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/tenant-deployments/new-mg/azuredeploy.json"
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

En el caso de las implementaciones de nivel de inquilino, debe proporcionar una ubicación para la implementación. La ubicación de la implementación es independiente de la ubicación de los recursos que se implementan. La ubicación de implementación especifica dónde se almacenarán los datos de la implementación. Las implementaciones de [suscripción](deploy-to-subscription.md) y [grupo de administración](deploy-to-management-group.md) también requieren una ubicación. En las implementaciones de [grupo de recursos](deploy-to-resource-group.md), la ubicación del grupo de recursos se usa para almacenar los datos de implementación.

Puede proporcionar un nombre para la implementación o usar el nombre de implementación predeterminado. El nombre predeterminado es el nombre del archivo de plantilla. Por ejemplo, al implementar una plantilla llamada _azuredeploy.json_, se crea un nombre de predeterminado **azuredeploy**.

Para cada nombre de implementación, la ubicación es inmutable. No se puede crear una implementación en una ubicación si ya existe una implementación con el mismo nombre en otra ubicación. Por ejemplo, si crea una implementación de inquilino con el nombre **deployment1** en **centralus**, no podrá crear otra implementación con el nombre **deployment1**, sino una ubicación de **westus**. Si recibe el código de error `InvalidDeploymentLocation`, use un nombre diferente o utilice la ubicación de la implementación anterior que tenía ese mismo nombre.

## <a name="deployment-scopes"></a>Ámbitos de implementación

Al implementar en un inquilino, puede implementar los recursos en:

* el inquilino
* grupos de administración dentro del inquilino
* subscriptions
* grupos de recursos

Un [recurso de extensión](scope-extension-resources.md) se puede limitar a un destino distinto del destino de implementación.

El usuario que implementa la plantilla debe tener acceso al ámbito especificado.

En esta sección se muestra cómo especificar distintos ámbitos. Puede combinar estos distintos ámbitos en una sola plantilla.

### <a name="scope-to-tenant"></a>Ámbito del inquilino

Los recursos definidos en la sección de recursos de la plantilla se aplican al inquilino.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/default-tenant.json" highlight="5":::

### <a name="scope-to-management-group"></a>Ámbito del grupo de administración

Para establecer como destino un grupo de administración dentro del inquilino, agregue una implementación anidada y especifique la propiedad `scope`.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/tenant-to-mg.json" highlight="10,17,18,22":::

### <a name="scope-to-subscription"></a>Ámbito de la suscripción

También puede dirigirse a las suscripciones dentro del inquilino. El usuario que implementa la plantilla debe tener acceso al ámbito especificado.

Para establecer como destino una suscripción dentro del inquilino, use una implementación anidada y la propiedad `subscriptionId`.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/tenant-to-subscription.json" highlight="9,10,18":::

### <a name="scope-to-resource-group"></a>Ámbito del grupo de recursos

También puede dirigirse a los grupos de recursos dentro del inquilino. El usuario que implementa la plantilla debe tener acceso al ámbito especificado.

Para establecer como destino un grupo de recursos dentro del inquilino, use una implementación anidada. Establezca las propiedades `subscriptionId` y `resourceGroup`. No establezca una ubicación para la implementación anidada porque se implementa en la ubicación del grupo de recursos.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/tenant-to-rg.json" highlight="9,10,18":::

## <a name="create-management-group"></a>Creación de un grupo de administración

La siguiente plantilla crea un grupo de administración.

:::code language="json" source="~/quickstart-templates/tenant-deployments/new-mg/azuredeploy.json":::

Si su cuenta no tiene permiso para la implementación en el inquilino, puede crear grupos de administración mediante la implementación en otro ámbito. Para más información, consulte [Grupo de administración](deploy-to-management-group.md#management-group).

## <a name="assign-role"></a>Asignación de un rol

La siguiente plantilla asigna un rol en el ámbito del inquilino.

:::code language="json" source="~/quickstart-templates/tenant-deployments/tenant-role-assignment/azuredeploy.json":::

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre la asignación de roles, consulte [Incorporación de asignaciones de roles mediante plantillas de Azure Resource Manager](../../role-based-access-control/role-assignments-template.md).
* También puede implementar plantillas en el [nivel de suscripción](deploy-to-subscription.md) o el [nivel de grupo de administración](deploy-to-management-group.md).
