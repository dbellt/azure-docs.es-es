---
title: 'Incorporación o edición de condiciones de asignación de roles de Azure mediante la API REST (versión preliminar): Control de acceso basado en roles de Azure (RBAC de Azure)'
description: Aprenda a agregar, editar, mostrar o eliminar condiciones de control de acceso basado en atributos (ABAC) en las asignaciones de roles de Azure mediante la API REST y el control de acceso basado en roles (RBAC) de Azure.
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.subservice: conditions
ms.topic: how-to
ms.workload: identity
ms.date: 05/07/2021
ms.author: rolyon
ms.openlocfilehash: 4a929fd99f10e66ea07dacc0302dd6d0bba83a96
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109656567"
---
# <a name="add-or-edit-azure-role-assignment-conditions-using-the-rest-api-preview"></a>Incorporación o edición de condiciones de asignación de roles de Azure mediante la API REST (versión preliminar)

> [!IMPORTANT]
> Las condiciones de asignación de roles de Azure y ABAC de Azure se encuentran actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Una [condición de asignación de roles de Azure](conditions-overview.md) es una comprobación adicional que puede agregar opcionalmente a la asignación de roles para proporcionar un control de acceso más preciso. Por ejemplo, puede agregar una condición que requiera que un objeto tenga una etiqueta específica para leer el objeto. En este artículo se describe cómo agregar, editar, mostrar o eliminar condiciones para las asignaciones de roles mediante la API REST.

## <a name="prerequisites"></a>Requisitos previos

Para obtener información sobre los requisitos previos para agregar o editar condiciones de asignación de roles, consulte [Requisitos previos de las condiciones](conditions-prerequisites.md).

## <a name="add-a-condition"></a>Agregar una condición

Para agregar una condición de asignación de roles, use la API REST [Asignación de roles: creación](/rest/api/authorization/roleassignments/create). Establezca `api-version` en `2020-03-01-preview` u otra versión posterior. Si desea utilizar la propiedad `description` para las asignaciones de roles, use `2020-04-01-preview` o posterior.  [Asignación de roles: creación](/rest/api/authorization/roleassignments/create) incluye los siguientes parámetros relacionados con las condiciones.

| Parámetro | Tipo | Descripción |
| --- | --- | --- |
| `condition` | String | Condición bajo la que se puede conceder permiso al usuario. |
| `conditionVersion` | String | Versión de la sintaxis de la condición. Si `condition` se especifica sin `conditionVersion`, la versión se establece en el valor predeterminado 2.0. |

Use la solicitud y el cuerpo siguientes:

```http
PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}?api-version=2020-04-01-preview
```

```json
{
    "properties": {
        "roleDefinitionId": "/{scope}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}",
        "principalId": "{principalId}",
        "condition": "{condition}",
        "conditionVersion": "2.0",
        "description": "{description}"
    }
}
```

En el siguiente ejemplo se muestra cómo asignar el rol [Lector de datos de Storage Blob](built-in-roles.md#storage-blob-data-reader) con una condición. La condición comprueba si el nombre del contenedor es igual a "blobs-example-container".

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}?api-version=2020-04-01-preview
```

```json
{
    "properties": {
        "roleDefinitionId": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Authorization/roleDefinitions/2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
        "principalId": "{principalId}",
        "condition": "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container'))",
        "conditionVersion": "2.0",
        "description": "Read access if container name equals blobs-example-container"
    }
}
```

El siguiente texto muestra un ejemplo de la salida:

```json
{
    "properties": {
        "roleDefinitionId": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Authorization/roleDefinitions/2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
        "principalId": "{principalId}",
        "principalType": "User",
        "scope": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}",
        "condition": "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container'))",
        "conditionVersion": "2.0",
        "createdOn": "2021-04-20T06:20:44.0205560Z",
        "updatedOn": "2021-04-20T06:20:44.2955371Z",
        "createdBy": null,
        "updatedBy": "{updatedById}",
        "delegatedManagedIdentityResourceId": null,
        "description": "Read access if container name equals blobs-example-container"
    },
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
    "type": "Microsoft.Authorization/roleAssignments",
    "name": "{roleAssignmentId}"
}
```

## <a name="edit-a-condition"></a>Edición de una condición

Para editar una condición de asignación de roles existente, use la misma API REST [Asignaciones de roles: creación](/rest/api/authorization/roleassignments/create) que usó para agregar la condición de la asignación de roles. A continuación se muestra un archivo JSON de ejemplo donde se actualizan `condition` y `description`. Solo se pueden editar las propiedades `condition`, `conditionVersion` y `description`. Debe especificar las demás propiedades para que coincidan con la asignación de roles existente.

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}?api-version=2020-04-01-preview
```

```json
{
    "properties": {
        "roleDefinitionId": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Authorization/roleDefinitions/2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
        "principalId": "{principalId}",
        "condition": "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container' OR @Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container2'))",
        "conditionVersion": "2.0",
        "description": "Read access if container name equals blobs-example-container or blobs-example-container2"
    }
}
```


## <a name="list-a-condition"></a>Visualización de una condición

Para mostrar una condición de asignación de roles, use la API [Asignación de roles: lista](/rest/api/authorization/roleassignments/list). Establezca `api-version` en `2020-03-01-preview` u otra versión posterior. Si desea utilizar la propiedad `description` para las asignaciones de roles, use `2020-04-01-preview` o posterior. Para más información, consulte [Enumeración de asignaciones de roles de Azure mediante la API REST](role-assignments-list-rest.md).

## <a name="delete-a-condition"></a>Eliminar una condición

Para eliminar una condición de asignación de roles, edítela y establezca la condición y la versión de la condición en una cadena vacía o NULL.

Como alternativa, si desea eliminar la asignación de roles y la condición, puede usar la API [Asignaciones de roles: eliminar](/rest/api/authorization/roleassignments/delete). Para más información, consulte [Eliminación de asignaciones de roles](role-assignments-remove.md).

## <a name="next-steps"></a>Pasos siguientes

- [Condiciones de asignación de roles de Azure de ejemplo (versión preliminar)](../storage/common/storage-auth-abac-examples.md)
- [Tutorial: Adición de una condición de asignación de roles para restringir el acceso a blobs mediante Azure Portal (versión preliminar)](../storage/common/storage-auth-abac-portal.md)
- [Solución de problemas de las condiciones de asignación de roles de Azure (versión preliminar)](conditions-troubleshoot.md)
