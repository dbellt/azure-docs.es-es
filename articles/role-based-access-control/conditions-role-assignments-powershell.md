---
title: 'Incorporación o edición de condiciones de asignación de roles de Azure mediante Azure PowerShell (versión preliminar): Azure RBAC'
description: Obtenga información sobre cómo agregar, editar, ver o eliminar condiciones de control de acceso basado en atributos (ABAC) en las asignaciones de roles de Azure mediante Azure PowerShell y el control de acceso basado en rol de Azure (Azure RBAC).
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.subservice: conditions
ms.topic: how-to
ms.workload: identity
ms.date: 05/07/2021
ms.author: rolyon
ms.openlocfilehash: 0ecc87a59d6db01c5c5dc4093bb42e4ee9c190d9
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109656585"
---
# <a name="add-or-edit-azure-role-assignment-conditions-using-azure-powershell-preview"></a>Incorporación o edición de condiciones de asignación de roles de Azure mediante Azure PowerShell (versión preliminar)

> [!IMPORTANT]
> Las condiciones de asignación de roles de Azure y Azure ABAC se encuentran actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Una [condición de asignación de roles de Azure](conditions-overview.md) es una comprobación adicional que puede agregar opcionalmente a la asignación de roles para proporcionar un control de acceso más preciso. Por ejemplo, puede agregar una condición que requiera que un objeto tenga una etiqueta específica para leer el objeto. En este artículo se describe cómo agregar, editar, ver o eliminar condiciones para las asignaciones de roles mediante Azure PowerShell.

## <a name="prerequisites"></a>Requisitos previos

Para obtener información sobre los requisitos previos para agregar o editar condiciones de asignación de roles, consulte [Requisitos previos de las condiciones](conditions-prerequisites.md).

## <a name="add-a-condition"></a>Agregar una condición

Para agregar una asignación de roles, use el comando [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment). El comando [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment) incluye los siguientes parámetros relacionados con las condiciones.

| Parámetro | Tipo | Descripción |
| --- | --- | --- |
| `Condition` | String | Condición bajo la que se puede conceder permiso al usuario. |
| `ConditionVersion` | String | Versión de la sintaxis de la condición. Se debe establecer en 2.0. Si se especifica `Condition`, se debe especificar también `ConditionVersion`. |

En el ejemplo siguiente se muestra cómo inicializar las variables para asignar el rol [Lector de datos de Storage Blob](built-in-roles.md#storage-blob-data-reader) con una condición. La condición comprueba si el nombre del contenedor es igual a "blobs-example-container".

```azurepowershell
$subscriptionId = "<subscriptionId>"
$resourceGroup = "<resourceGroup>"
$roleDefinitionName = "Storage Blob Data Reader"
$roleDefinitionId = "2a2b9908-6ea1-4ae2-8e65-a410df84e7d1"
$userObjectId = "<userObjectId>"
$scope = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroup"
$description = "Read access if container name equals blobs-example-container"
$condition = "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container'))"
$conditionVersion = "2.0"
```

Use [New-AzRoleAssignment para](/powershell/module/az.resources/new-azroleassignment) asignar el rol con una condición.

```azurepowershell
New-AzRoleAssignment -ObjectId $userObjectId -Scope $scope -RoleDefinitionId $roleDefinitionId -Description $description -Condition $condition -ConditionVersion $conditionVersion
```

Este es un ejemplo de la salida:

```azurepowershell
RoleAssignmentId   : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.Authorization/roleAssignments/<roleAssignmentId>
Scope              : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>
DisplayName        : User1
SignInName         : user1@contoso.com
RoleDefinitionName : Storage Blob Data Reader
RoleDefinitionId   : 2a2b9908-6ea1-4ae2-8e65-a410df84e7d1
ObjectId           : <userObjectId>
ObjectType         : User
CanDelegate        : False
Description        : Read access if container name equals blobs-example-container
ConditionVersion   : 2.0
Condition          : ((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container'))
```

En PowerShell, si la condición incluye un signo de dólar ($), debe agregarle un carácter de acento grave (\`) como prefijo. Por ejemplo, esta condición usa signos de dólar para delinear el nombre de la clave de etiqueta. Para obtener más información sobre las reglas de comillas en PowerShell, vea [Acerca de las reglas de las comillas](/powershell/module/microsoft.powershell.core/about/about_quoting_rules).

```azurepowershell
$condition = "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Read.WithTagConditions'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<`$key_case_sensitive`$>] StringEquals 'Cascade'))"
```

## <a name="edit-a-condition"></a>Edición de una condición

Para editar una condición de asignación de roles existente, use [Set-AzRoleAssignment](/powershell/module/az.resources/set-azroleassignment). Solo se pueden editar las propiedades `Condition`, `ConditionVersion`y `Description`. El parámetro `-PassThru` hace que [Set-AzRoleAssignment](/powershell/module/az.resources/set-azroleassignment) devuelva la asignación de roles actualizada, lo que permite la visualización o el almacenamiento en una variable para su uso posterior.

Hay dos maneras de editar una condición. Puede usar el objeto `PSRoleAssignment` o un archivo JSON.

### <a name="edit-a-condition-using-the-psroleassignment-object"></a>Edición de una condición mediante el objeto PSRoleAssignment

1. Use [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment) para obtener la asignación de roles existente con una condición como objeto `PSRoleAssignment`.

    ```azurepowershell
    $testRa = Get-AzRoleAssignment -Scope $scope -RoleDefinitionName $roleDefinitionName -ObjectId $userObjectId
    ```

1. Edite la condición.

    ```azurepowershell
    $condition = "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container' OR @Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container2'))"
    ```

1. Inicialice la condición y la descripción.

    ```azurepowershell
    $testRa.Condition = $condition
    $testRa.Description = "Read access if container name equals blobs-example-container or blobs-example-container2"
    ```

1. Use [Set-AzRoleAssignment](/powershell/module/az.resources/set-azroleassignment) para actualizar la condición de la asignación de roles.

    ```azurepowershell
    Set-AzRoleAssignment -InputObject $testRa -PassThru
    ```

    Este es un ejemplo de la salida:

    ```azurepowershell
    RoleAssignmentId   : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.Authorization/roleAssignments/<roleAssignmentId>
    Scope              : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>
    DisplayName        : User1
    SignInName         : user1@contoso.com
    RoleDefinitionName : Storage Blob Data Reader
    RoleDefinitionId   : 2a2b9908-6ea1-4ae2-8e65-a410df84e7d1
    ObjectId           : <userObjectId>
    ObjectType         : User
    CanDelegate        : False
    Description        : Read access if container name equals blobs-example-container or blobs-example-container2
    ConditionVersion   : 2.0
    Condition          : ((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container' OR @Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container2'))
    ```

### <a name="edit-a-condition-using-a-json-file"></a>Edición de una condición mediante un archivo JSON

Para editar una condición, también puede proporcionar un archivo JSON como entrada. A continuación se muestra un archivo JSON de ejemplo donde se actualizan `Condition` y `Description`. Debe especificar todas las propiedades del archivo JSON para actualizar una condición.

```json
{
    "RoleDefinitionId": "2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
    "ObjectId": "<userObjectId>",
    "ObjectType": "User",
    "Scope": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>",
    "Condition": "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container' OR @Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container2'))",
    "ConditionVersion": "2.0",
    "CanDelegate": false,
    "Description": "Read access if container name equals blobs-example-container or blobs-example-container2",
    "RoleAssignmentId": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.Authorization/roleAssignments/<roleAssignmentId>"
}
```

Use [Set-AzRoleAssignment](/powershell/module/az.resources/set-azroleassignment) para actualizar la condición de la asignación de roles.

```azurepowershell
Set-AzRoleAssignment -InputFile "C:\path\roleassignment.json" -PassThru
```

Este es un ejemplo de la salida:

```azurepowershell
RoleAssignmentId   : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.Authorization/roleAssignments/<roleAssignmentId>
Scope              : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>
DisplayName        : User1
SignInName         : user1@contoso.com
RoleDefinitionName : Storage Blob Data Reader
RoleDefinitionId   : 2a2b9908-6ea1-4ae2-8e65-a410df84e7d1
ObjectId           : <userObjectId>
ObjectType         : User
CanDelegate        : False
Description        : Read access if container name equals blobs-example-container or blobs-example-container2
ConditionVersion   : 2.0
Condition          : ((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container' OR @Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container2'))
```

## <a name="list-a-condition"></a>Visualización de una condición

Para ver una condición de asignación de roles, use [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment). Para más información, consulte [Enumeración de asignaciones de roles de Azure mediante Azure PowerShell](role-assignments-list-powershell.md).

## <a name="delete-a-condition"></a>Eliminar una condición

Para eliminar una condición de asignación de roles, edite la condición de asignación de roles y establezca las propiedades `Condition` y `ConditionVersion` en una cadena vacía (`""`) o `$null`.

Como alternativa, si desea eliminar la asignación de roles y la condición, puede usar el comando [Remove-AzRoleAssignment](/powershell/module/az.resources/remove-azroleassignment). Para obtener más información, consulte [Eliminación de asignaciones de roles](role-assignments-remove.md).

## <a name="next-steps"></a>Pasos siguientes

- [Condiciones de asignación de roles de Azure de ejemplo (versión preliminar)](../storage/common/storage-auth-abac-examples.md)
- [Tutorial: Adición de una condición de asignación de roles para restringir el acceso a blobs mediante Azure PowerShell (versión preliminar)](../storage/common/storage-auth-abac-powershell.md)
- [Solución de problemas de las condiciones de asignación de roles de Azure (versión preliminar)](conditions-troubleshoot.md)
