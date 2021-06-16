---
title: 'Eliminación de asignaciones de roles de Azure: RBAC de Azure'
description: Aprenda a quitar acceso a recursos de Azure para usuarios, grupos, entidades de servicio e identidades administradas mediante Azure Portal, Azure PowerShell, la CLI de Azure o API REST.
services: active-directory
author: rolyon
manager: daveba
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 02/15/2021
ms.author: rolyon
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: df37d64e634d2764666e29300e5157ceb3f3ec39
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110690845"
---
# <a name="remove-azure-role-assignments"></a>Eliminación de asignaciones de roles de Azure

El [control de acceso basado en rol (RBAC) de Azure](../../articles/role-based-access-control/overview.md) es el sistema de autorización que puede utilizar para administrar el acceso a los recursos de Azure. Para quitar el acceso de un recurso de Azure, se quita una asignación de roles. En este artículo se describe cómo quitar las asignaciones de roles mediante Azure Portal, Azure PowerShell, la CLI de Azure y la API REST.

## <a name="prerequisites"></a>Prerrequisitos

Para quitar asignaciones de roles, debe tener:

- Permisos `Microsoft.Authorization/roleAssignments/delete`, como [Administrador de acceso de usuarios](../../articles/role-based-access-control/built-in-roles.md#user-access-administrator) o [Propietario](../../articles/role-based-access-control/built-in-roles.md#owner)

## <a name="azure-portal"></a>Azure Portal

1. Abra **Control de acceso (IAM)** en un ámbito como, por ejemplo, grupo de administración, suscripción, grupo de recursos o recurso, en donde quiere quitar el acceso.

1. Haga clic en la pestaña **Asignaciones de roles** para ver todas las asignaciones de roles en este ámbito.

1. En la lista de asignaciones de roles, agregue una marca de verificación a la entidad de seguridad con la asignación de roles que desee quitar.

   ![Asignación de roles seleccionada que se va a quitar](./media/role-assignments-remove/rg-role-assignments-select.png)

1. Haga clic en **Quitar**.

   ![Mensaje de eliminación de asignación de roles](./media/role-assignments-remove/remove-role-assignment.png)

1. En el mensaje de eliminación de asignación de roles que aparece, haga clic en **Sí**.

    Si ve un mensaje que indica que las asignaciones de roles heredadas no se pueden quitar, significa que intenta quitar una asignación de roles en un ámbito secundario. Debe abrir el control de acceso (IAM) en el ámbito en el que se asignó el rol e intentarlo de nuevo. Una forma rápida de abrir el control de acceso (IAM) en el ámbito correcto es examinar la columna **Ámbito** y hacer clic en el vínculo situado junto a **(Heredado)** .

   ![Quitar el mensaje de asignación de roles de las asignaciones de roles heredadas](./media/role-assignments-remove/remove-role-assignment-inherited.png)

## <a name="azure-powershell"></a>Azure PowerShell

En Azure PowerShell, para quitar una asignación de roles, use [Remove-AzRoleAssignment](/powershell/module/az.resources/remove-azroleassignment).

En el ejemplo siguiente, se quita la asignación de roles [Colaborador de la máquina virtual](built-in-roles.md#virtual-machine-contributor) del usuario *patlong\@contoso.com* en el grupo de recursos *pharma-sales*:

```azurepowershell
PS C:\> Remove-AzRoleAssignment -SignInName patlong@contoso.com `
-RoleDefinitionName "Virtual Machine Contributor" `
-ResourceGroupName pharma-sales
```

Quita el rol [Lector](built-in-roles.md#reader) del grupo *Ann Mack Team* con el id. 22222222-2222-2222-2222-222222222222 en el ámbito de suscripción.

```azurepowershell
PS C:\> Remove-AzRoleAssignment -ObjectId 22222222-2222-2222-2222-222222222222 `
-RoleDefinitionName "Reader" `
-Scope "/subscriptions/00000000-0000-0000-0000-000000000000"
```

Quita el rol [Lector de facturación](built-in-roles.md#billing-reader) al usuario *alain\@example.com* en el ámbito de grupo de administración.

```azurepowershell
PS C:\> Remove-AzRoleAssignment -SignInName alain@example.com `
-RoleDefinitionName "Billing Reader" `
-Scope "/providers/Microsoft.Management/managementGroups/marketing-group"
```

Si se obtiene un mensaje de error parecido al siguiente: "La información proporcionada no se asigna a una asignación de roles", asegúrese de especificar también los parámetros `-Scope` o `-ResourceGroupName`. Para más información, consulte [Solución de problemas de RBAC de Azure](troubleshooting.md#role-assignments-with-identity-not-found).

## <a name="azure-cli"></a>Azure CLI

En la CLI de Azure, para quitar una asignación de roles, use [az role assignment delete](/cli/azure/role/assignment#az_role_assignment_delete).

En el ejemplo siguiente, se quita la asignación de roles [Colaborador de la máquina virtual](built-in-roles.md#virtual-machine-contributor) del usuario *patlong\@contoso.com* en el grupo de recursos *pharma-sales*:

```azurecli
az role assignment delete --assignee "patlong@contoso.com" \
--role "Virtual Machine Contributor" \
--resource-group "pharma-sales"
```

Quita el rol [Lector](built-in-roles.md#reader) del grupo *Ann Mack Team* con el id. 22222222-2222-2222-2222-222222222222 en el ámbito de suscripción.

```azurecli
az role assignment delete --assignee "22222222-2222-2222-2222-222222222222" \
--role "Reader" \
--subscription "00000000-0000-0000-0000-000000000000"
```

Quita el rol [Lector de facturación](built-in-roles.md#billing-reader) al usuario *alain\@example.com* en el ámbito de grupo de administración.

```azurecli
az role assignment delete --assignee "alain@example.com" \
--role "Billing Reader" \
--scope "/providers/Microsoft.Management/managementGroups/marketing-group"
```

## <a name="rest-api"></a>API DE REST

En la API REST, para quitar una asignación de roles, use [Asignaciones de roles - Eliminar](/rest/api/authorization/roleassignments/delete).

1. Obtenga el identificador de asignación de roles (GUID). Este identificador se devuelve cuando se crea por primera vez la asignación de roles o puede obtenerlo enumerando las asignaciones de roles.

1. Empiece con la solicitud siguiente:

    ```http
    DELETE https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}?api-version=2015-07-01
    ```

1. En el identificador URI, reemplace *{scope}* por el ámbito para eliminar la asignación de roles.

    > [!div class="mx-tableFixed"]
    > | Ámbito | Tipo |
    > | --- | --- |
    > | `providers/Microsoft.Management/managementGroups/{groupId1}` | Grupo de administración |
    > | `subscriptions/{subscriptionId1}` | Subscription |
    > | `subscriptions/{subscriptionId1}/resourceGroups/myresourcegroup1` | Resource group |
    > | `subscriptions/{subscriptionId1}/resourceGroups/myresourcegroup1/providers/microsoft.web/sites/mysite1` | Resource |

1. Reemplace *{roleAssignmentId}* por el identificador GUID de la asignación de roles.

La siguiente solicitud quita la asignación de roles especificada en el ámbito de la suscripción:

```http
DELETE https://management.azure.com/subscriptions/{subscriptionId1}/providers/microsoft.authorization/roleassignments/{roleAssignmentId1}?api-version=2015-07-01
```

El siguiente texto muestra un ejemplo de la salida:

```json
{
    "properties": {
        "roleDefinitionId": "/subscriptions/{subscriptionId1}/providers/Microsoft.Authorization/roleDefinitions/a795c7a0-d4a2-40c1-ae25-d81f01202912",
        "principalId": "{objectId1}",
        "scope": "/subscriptions/{subscriptionId1}",
        "createdOn": "2020-05-06T23:55:24.5379478Z",
        "updatedOn": "2020-05-06T23:55:24.5379478Z",
        "createdBy": "{createdByObjectId1}",
        "updatedBy": "{updatedByObjectId1}"
    },
    "id": "/subscriptions/{subscriptionId1}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId1}",
    "type": "Microsoft.Authorization/roleAssignments",
    "name": "{roleAssignmentId1}"
}
```

## <a name="arm-template"></a>Plantilla ARM

No hay ninguna manera de quitar una asignación de roles mediante una plantilla de Azure Resource Manager (plantilla de ARM). Para quitar una asignación de roles, debe usar otras herramientas, como Azure Portal, Azure PowerShell, la CLI de Azure o la API REST.

## <a name="next-steps"></a>Pasos siguientes

- [Enumeración de asignaciones de roles de Azure mediante Azure Portal](role-assignments-list-portal.md)
- [Enumeración de asignaciones de roles de Azure mediante Azure PowerShell](role-assignments-list-powershell.md)
- [Solución de problemas de Azure RBAC](troubleshooting.md)
