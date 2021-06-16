---
title: Automatizar la adición de un usuario de laboratorio en Azure DevTest Labs | Microsoft Docs
description: En este artículo se muestra cómo automatizar la incorporación de un usuario a un laboratorio en Azure DevTest Labs mediante plantillas de Azure Resource Manager, PowerShell y la CLI.
ms.topic: article
ms.date: 06/26/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 70f8e2740a53c7bb855d3796efa438e9c9ff0ffa
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111965222"
---
# <a name="automate-adding-a-lab-user-to-a-lab-in-azure-devtest-labs"></a>Automatizar la adición de un usuario de laboratorio a un laboratorio en Azure DevTest Labs
Azure DevTest Labs le permite crear rápidamente entornos de desarrollo y pruebas de autoservicio mediante Azure Portal. Pero si tiene varios equipos y varias instancias de DevTest Labs, automatizar el proceso de creación puede ahorrar tiempo. Las [plantillas de Azure Resource Manager](https://github.com/Azure/azure-devtestlab/tree/master/Environments) le permiten crear laboratorios, máquinas virtuales de laboratorio, imágenes personalizadas, fórmulas y agregar usuarios de forma automática. Este artículo se centra específicamente en la adición de usuarios a una instancia de DevTest Labs.

Para agregar un usuario a un laboratorio, agrega el usuario al rol de **usuario de DevTest Labs** para el laboratorio. Este artículo le muestra cómo automatizar la adición de un usuario a un laboratorio mediante una de las maneras siguientes:

- Plantillas del Administrador de recursos de Azure
- Cmdlets de Azure PowerShell 
- CLI de Azure.

## <a name="use-azure-resource-manager-templates"></a>Utilización de plantillas del Administrador de recursos de Azure
La siguiente plantilla de ejemplo de Resource Manager especifica un usuario que se agregará al rol de **usuario de DevTest Labs** de un laboratorio. 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "principalId": {
      "type": "string",
      "metadata": {
        "description": "The objectId of the user, group, or service principal for the role."
      }
    },
    "labName": {
      "type": "string",
      "metadata": {
        "description": "The name of the lab instance to be created."
      }
    },
    "roleAssignmentGuid": {
      "type": "string",
      "metadata": {
        "description": "Guid to use as the name for the role assignment."
      }
    }
  },
  "variables": {
    "devTestLabUserRoleId": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/111111111-0000-0000-11111111111111111')]",
    "fullDevTestLabUserRoleName": "[concat(parameters('labName'), '/Microsoft.Authorization/', parameters('roleAssignmentGuid'))]",
    "roleScope": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.DevTestLab/labs/', parameters('labName'))]"
  },
  "resources": [
    {
      "apiVersion": "2016-05-15",
      "type": "Microsoft.DevTestLab/labs",
      "name": "[parameters('labName')]",
      "location": "[resourceGroup().location]"
    },
    {
      "apiVersion": "2016-07-01",
      "type": "Microsoft.DevTestLab/labs/providers/roleAssignments",
      "name": "[variables('fullDevTestLabUserRoleName')]",
      "properties": {
        "roleDefinitionId": "[variables('devTestLabUserRoleId')]",
        "principalId": "[parameters('principalId')]",
        "scope": "[variables('roleScope')]"
      },
      "dependsOn": [
        "[resourceId('Microsoft.DevTestLab/labs', parameters('labName'))]"
      ]
    }
  ]
}

```

Si va a asignar el rol en la misma plantilla que está creando el laboratorio, no olvide agregar una dependencia entre el recurso de asignación de roles y el laboratorio. Para obtener más información, consulte el artículo [Definición de dependencias en plantillas de Azure Resource Manager](../azure-resource-manager/templates/resource-dependency.md).

### <a name="role-assignment-resource-information"></a>Información sobre el recurso de asignación de roles
El recurso de asignación de roles debe especificar el tipo y nombre.

Lo primero que debe tener en cuenta es que el tipo del recurso no es `Microsoft.Authorization/roleAssignments`, como sería para un grupo de recursos.  En su lugar, el tipo de recurso sigue el patrón `{provider-namespace}/{resource-type}/providers/roleAssignments`. En este caso, el tipo de recurso será `Microsoft.DevTestLab/labs/providers/roleAssignments`.

El propio nombre de la asignación de roles debe ser único globalmente.  El nombre de la asignación usa el patrón `{labName}/Microsoft.Authorization/{newGuid}`. El `newGuid` es un valor de parámetro para la plantilla. Se asegura de que el nombre de la asignación de roles es único. Como no hay ninguna función de plantilla para crear GUID, deberá generar un GUID mediante cualquier herramienta generadora de GUID.  

En la plantilla, el nombre de la asignación de roles se define mediante la variable `fullDevTestLabUserRoleName`. La línea exacta de la plantilla es:

```json
"fullDevTestLabUserRoleName": "[concat(parameters('labName'), '/Microsoft.Authorization/', parameters('roleAssignmentGuid'))]"
```


### <a name="role-assignment-resource-properties"></a>Propiedades del recurso de asignación de roles
Una asignación de roles en sí define tres propiedades. Necesita `roleDefinitionId`, `principalId` y `scope`.

### <a name="role-definition"></a>Definición de roles
El identificador de definición de roles es el identificador de cadena para la definición de roles existente. El identificador de roles tiene el formato: `/subscriptions/{subscription-id}/providers/Microsoft.Authorization/roleDefinitions/{role-definition-id}`. 

El identificador de suscripción se obtiene mediante la función de plantilla `subscription().subscriptionId`.  

Deberá obtener la definición de roles para el rol integrado `DevTest Labs User`. Para obtener el GUID para el rol de [usuario de DevTest Labs](../role-based-access-control/built-in-roles.md#devtest-labs-user), puede usar la [API de REST de asignaciones de roles](/rest/api/authorization/roleassignments) o el cmdlet [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition).

```powershell
$dtlUserRoleDefId = (Get-AzRoleDefinition -Name "DevTest Labs User").Id
```

El identificador de rol se define en la sección de variables y se denomina `devTestLabUserRoleId`. En la plantilla, el identificador de rol se establece en: 111111111-0000-0000-11111111111111111. 

```json
"devTestLabUserRoleId": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/111111111-0000-0000-11111111111111111')]",
```

### <a name="principal-id"></a>Identificador de entidad de seguridad
El identificador de entidad de seguridad es el identificador de objeto del usuario, grupo o entidad de servicio de Active Directory que quiere agregar como usuario de laboratorio para el laboratorio. La plantilla usa `ObjectId` como un parámetro.

Puede obtener el valor de ObjectId mediante los cmdlets de PowerShell [Get-AzureRMADUser](/powershell/module/azurerm.resources/get-azurermaduser?view=azurermps-6.13.0&preserve-view=true), [Get-AzureRMADGroup o [Get-AzureRMADServicePrincipal](/powershell/module/azurerm.resources/get-azurermadserviceprincipal?view=azurermps-6.13.0&preserve-view=true). Estos cmdlets devuelven un objeto único o unas listas de objetos de Active Directory que tienen una propiedad de identificador, que es el identificador de objeto que necesita. El ejemplo siguiente muestra cómo obtener el identificador de objeto de un usuario único en una empresa.

```powershell
$userObjectId = (Get-AzureRmADUser -UserPrincipalName 'email@company.com').Id
```

También puede usar los cmdlets de PowerShell de Azure Active Directory que incluyen [Get-MsolUser](/powershell/module/msonline/get-msoluser?preserve-view=true&view=azureadps-1.0), [Get-MsolGroup](/powershell/module/msonline/get-msolgroup?preserve-view=true&view=azureadps-1.0) y [Get-MsolServicePrincipal](/powershell/module/msonline/get-msolserviceprincipal?preserve-view=true&view=azureadps-1.0).

### <a name="scope"></a>Ámbito
El ámbito especifica el recurso o grupo de recursos para el que se debe aplicar la asignación de roles. Para los recursos, el ámbito tiene el formato: `/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/{provider-namespace}/{resource-type}/{resource-name}`. La plantilla usa la función `subscription().subscriptionId` para rellenar el apartado `subscription-id` y la función de plantilla `resourceGroup().name` para rellenar el apartado `resource-group-name`. El uso de estas funciones implica que el laboratorio al que va a asignar un rol debe existir en la suscripción actual y en el mismo grupo de recursos en el que se realiza la implementación de plantilla. El último apartado, `resource-name`, es el nombre del laboratorio. Este valor se recibe a través del parámetro de plantilla de este ejemplo. 

El ámbito de rol en la plantilla: 

```json
"roleScope": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.DevTestLab/labs/', parameters('labName'))]"
```

### <a name="deploying-the-template"></a>Implementación de la plantilla
En primer lugar, cree un archivo de parámetros (por ejemplo: azuredeploy.parameters.json) que pase los valores de los parámetros en la plantilla de Resource Manager. 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "principalId": {
      "value": "11111111-1111-1111-1111-111111111111"
    },
    "labName": {
      "value": "MyLab"
    },
    "roleAssignmentGuid": {
      "value": "22222222-2222-2222-2222-222222222222"
    }
  }
}
```

Después, use el cmdlet de PowerShell [New-AzureRmResourceGroupDeployment](/powershell/module/azurerm.resources/new-azurermresourcegroupdeployment) para implementar la plantilla de Resource Manager. El comando de ejemplo siguiente asigna una persona, grupo o una entidad de servicio al rol de usuario de DevTest Labs para un laboratorio.

```powershell
New-AzureRmResourceGroupDeployment -Name "MyLabResourceGroup-$(New-Guid)" -ResourceGroupName 'MyLabResourceGroup' -TemplateParameterFile .\azuredeploy.parameters.json -TemplateFile .\azuredeploy.json
```

Es importante tener en cuenta que el GUID de asignación de roles y el nombre de implementación del grupo deben ser únicos. Si intenta implementar una asignación de recursos con un GUID que no es único, obtendrá el error `RoleAssignmentUpdateNotPermitted`.

Si tiene previsto usar la plantilla varias veces para agregar varios objetos de Active Directory al rol de usuario de DevTest Labs para el laboratorio, considere la posibilidad de utilizar objetos dinámicos en el comando de PowerShell. En el ejemplo siguiente se usa el cmdlet [New-Guid](/powershell/module/Microsoft.PowerShell.Utility/New-Guid) para especificar el GUID de asignación de roles y el nombre de implementación del grupo de recursos dinámicamente.

```powershell
New-AzureRmResourceGroupDeployment -Name "MyLabResourceGroup-$(New-Guid)" -ResourceGroupName 'MyLabResourceGroup' -TemplateFile .\azuredeploy.json -roleAssignmentGuid "$(New-Guid)" -labName "MyLab" -principalId "11111111-1111-1111-1111-111111111111"
```

## <a name="use-azure-powershell"></a>Uso de Azure PowerShell
Como se describe en la introducción, crea una nueva asignación de roles de Azure para agregar un usuario al rol de **usuario de DevTest Labs** para el laboratorio. En PowerShell, lo hace mediante el cmdlet [New-AzureRMRoleAssignment](/powershell/module/azurerm.resources/new-azurermroleassignment). Este cmdlet tiene muchos parámetros opcionales para permitir flexibilidad. Se puede especificar `ObjectId`, `SigninName` o `ServicePrincipalName` como el objeto al que se va a conceder permisos.  

A continuación se muestra un comando de Azure PowerShell de ejemplo que agrega un usuario al rol de usuario de DevTest Labs en el laboratorio especificado.

```powershell
New-AzureRmRoleAssignment -UserPrincipalName <email@company.com> -RoleDefinitionName 'DevTest Labs User' -ResourceName '<Lab Name>' -ResourceGroupName '<Resource Group Name>' -ResourceType 'Microsoft.DevTestLab/labs'
```

Para especificar el recurso al que se está concediendo permisos puede usar una combinación de `ResourceName`, `ResourceType`, `ResourceGroup` o mediante el parámetro `scope`. Independientemente de la combinación de parámetros que use, proporcione suficiente información para que el cmdlet identifique de forma única el objeto de Active Directory (usuario, grupo o entidad de servicio), el ámbito (grupo de recursos o recurso) y la definición de rol.

## <a name="use-azure-command-line-interface-cli"></a>Usar la Interfaz de la línea de comandos (CLI) de Azure
En la CLI de Azure, la adición de un usuario de laboratorios a un laboratorio se realiza mediante el uso del comando `az role assignment create`. Para más información sobre los cmdlets de la CLI de Azure, consulte [Incorporación o eliminación de asignaciones de roles de Azure mediante la CLI de Azure](../role-based-access-control/role-assignments-cli.md).

Se puede especificar el objeto al que se va a conceder acceso mediante los parámetros `objectId`, `signInName`, `spn`. El laboratorio al que se va a conceder acceso al objeto puede identificarse por la dirección url de `scope` o por una combinación de los parámetros `resource-name`, `resource-type` y `resource-group`.

En el siguiente ejemplo de la CLI de Azure se muestra cómo agregar a una persona al rol de usuario de DevTest Labs del laboratorio de pruebas especificado.  

```azurecli
az role assignment create --roleName "DevTest Labs User" --signInName <email@company.com> -–resource-name "<Lab Name>" --resource-type "Microsoft.DevTestLab/labs" --resource-group "<Resource Group Name>"
```

## <a name="next-steps"></a>Pasos siguientes
Vea los artículos siguientes:

- [Creación y administración de máquinas virtuales con DevTest Labs mediante la CLI de Azure](devtest-lab-vmcli.md)
- [Crear una máquina virtual con DevTest Labs mediante Azure PowerShell](devtest-lab-vm-powershell.md)
- [Usar herramientas de línea de comandos para iniciar y detener las máquinas virtuales de Azure DevTest Labs](use-command-line-start-stop-virtual-machines.md)