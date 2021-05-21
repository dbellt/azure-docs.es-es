---
title: 'Incorporación o edición de condiciones de asignación de roles de Azure mediante la CLI de Azure (versión preliminar): Control de acceso basado en roles de Azure (RBAC de Azure)'
description: Aprenda a agregar, editar, mostrar o eliminar condiciones de control de acceso basado en atributos (ABAC) en las asignaciones de roles de Azure mediante la CLI de Azure y el control de acceso basado en roles (RBAC) de Azure.
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.subservice: conditions
ms.topic: how-to
ms.workload: identity
ms.date: 05/07/2021
ms.author: rolyon
ms.openlocfilehash: c7baafbd34e9af488fbbee6237625aa469dff12c
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109656639"
---
# <a name="add-or-edit-azure-role-assignment-conditions-using-azure-cli-preview"></a>Incorporación o edición de condiciones de asignación de roles de Azure mediante la CLI de Azure (versión preliminar)

> [!IMPORTANT]
> Las condiciones de asignación de roles de Azure y ABAC se encuentran actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Una [condición de asignación de roles de Azure](conditions-overview.md) es una comprobación adicional que puede agregar opcionalmente a la asignación de roles para proporcionar un control de acceso más preciso. Por ejemplo, puede agregar una condición que requiera que un objeto tenga una etiqueta específica para poder leerlo. En este artículo se describe cómo agregar, editar, mostrar o eliminar condiciones para las asignaciones de roles mediante la CLI de Azure.

## <a name="prerequisites"></a>Requisitos previos

Para más información sobre los requisitos previos para agregar o editar condiciones de asignación de roles, consulte [Requisitos previos de las condiciones](conditions-prerequisites.md).

## <a name="add-a-condition"></a>Agregar una condición

Para agregar una condición de asignación de roles , use [az role assignment create](/cli/azure/role/assignment#az_role_assignment_create). El comando [az role assignment create](/cli/azure/role/assignment#az_role_assignment_create) incluye los siguientes parámetros relacionados con las condiciones.

| Parámetro | Tipo | Descripción |
| --- | --- | --- |
| `condition` | String | Condición bajo la que se puede conceder permiso al usuario. |
| `condition-version` | String | Versión de la sintaxis de la condición. Si `--condition` se especifica sin `--condition-version`, la versión se establece en el valor predeterminado 2.0. |

En el siguiente ejemplo se muestra cómo asignar el rol [Lector de datos de Storage Blob](built-in-roles.md#storage-blob-data-reader) con una condición. La condición comprueba si el nombre del contenedor es igual a "blobs-example-container".

```azurecli
az role assignment create --role "Storage Blob Data Reader" --assignee "user1@contoso.com" --resource-group {resourceGroup} \
--description "Read access if container name equals blobs-example-container" \
--condition "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container'))" \
--condition-version "2.0"
```

El siguiente texto muestra un ejemplo de la salida:

```azurecli
{
    "canDelegate": null,
    "condition": "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container'))",
    "conditionVersion": "2.0",
    "description": "Read access if container name equals blobs-example-container",
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
    "name": "{roleAssignmentId}",
    "principalId": "{userObjectId}",
    "principalType": "User",
    "resourceGroup": "{resourceGroup}",
    "roleDefinitionId": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
    "scope": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}",
    "type": "Microsoft.Authorization/roleAssignments"
}
```

## <a name="edit-a-condition"></a>Edición de una condición

Para editar una condición de asignación de roles existente, use [az role assignment update](/cli/azure/role/assignment#az_role_assignment_update) y un archivo JSON como entrada. A continuación se muestra un archivo JSON de ejemplo en el que la condición y la descripción están actualizadas. Solo se pueden editar las propiedades `condition`, `conditionVersion` y `description`. Debe especificar todas las propiedades para actualizar la condición de asignación de roles.

```json
{
    "canDelegate": null,
    "condition": "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container' OR @Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container2'))",
    "conditionVersion": "2.0",
    "description": "Read access if container name equals blobs-example-container or blobs-example-container2",
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
    "name": "{roleAssignmentId}",
    "principalId": "{userObjectId}",
    "principalType": "User",
    "resourceGroup": "{resourceGroup}",
    "roleDefinitionId": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
    "scope": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}",
    "type": "Microsoft.Authorization/roleAssignments"
}
```

Use [az role assignment update](/cli/azure/role/assignment#az_role_assignment_update) para actualizar la condición de la asignación de roles.

```azurecli
az role assignment update --role-assignment "./path/roleassignment.json"
```

El siguiente texto muestra un ejemplo de la salida:

```azurecli
{
    "canDelegate": null,
    "condition": "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container' OR @Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container2'))",
    "conditionVersion": "2.0",
    "description": "Read access if container name equals blobs-example-container or blobs-example-container2",
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
    "name": "{roleAssignmentId}",
    "principalId": "{userObjectId}",
    "principalType": "User",
    "resourceGroup": "{resourceGroup}",
    "roleDefinitionId": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
    "scope": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}",
    "type": "Microsoft.Authorization/roleAssignments"
}
```

## <a name="list-a-condition"></a>Mostrar una condición

Para mostrar una condición de asignación de roles, use [az role assignment list](/cli/azure/role/assignment#az_role_assignment_list). Para más información, consulte [Enumeración de asignaciones de roles de Azure mediante la CLI de Azure](role-assignments-list-cli.md).

## <a name="delete-a-condition"></a>Eliminar una condición

Para eliminar una condición de asignación de roles, edítela y establezca las propiedades `condition` y `condition-version` en una cadena vacía (`""`) o `null`.

Como alternativa, si desea eliminar la asignación de roles y la condición, puede usar el comando [az role assignment delete](/cli/azure/role/assignment#az_role_assignment_delete). Para más información, consulte [Eliminación de asignaciones de roles de Azure](role-assignments-remove.md).

## <a name="next-steps"></a>Pasos siguientes

- [Condiciones de asignación de roles de Azure de ejemplo (versión preliminar)](../storage/common/storage-auth-abac-examples.md)
- [Tutorial: Adición de una condición de asignación de roles para restringir el acceso a blobs mediante la CLI de Azure (versión preliminar)](../storage/common/storage-auth-abac-cli.md)
- [Solución de problemas de las condiciones de asignación de roles de Azure (versión preliminar)](conditions-troubleshoot.md)
