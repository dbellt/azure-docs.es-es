---
title: 'Enumeración de asignaciones de roles de Azure mediante la CLI de Azure: RBAC de Azure'
description: Aprenda a determinar a qué recursos tienen acceso los usuarios, grupos, entidades de servicio e identidades administradas mediante la CLI de Azure y el control de acceso basado en roles de Azure (RBAC de Azure).
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: 3483ee01-8177-49e7-b337-4d5cb14f5e32
ms.service: role-based-access-control
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 10/30/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 35170cb047ae7b4cfd376f86a76a36c88bbbaaa2
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109787498"
---
# <a name="list-azure-role-assignments-using-azure-cli"></a>Enumeración de asignaciones de roles de Azure mediante la CLI de Azure

[!INCLUDE [Azure RBAC definition list access](../../includes/role-based-access-control/definition-list.md)] En este artículo se describe cómo enumerar las asignaciones de roles mediante la CLI de Azure.

> [!NOTE]
> Si su organización ha externalizado las funciones de administración a un proveedor de servicios que usa [Azure Lighthouse](../lighthouse/overview.md), las asignaciones de roles autorizadas por ese proveedor de servicios no se mostrarán aquí.

## <a name="prerequisites"></a>Prerrequisitos

- [Bash en Azure Cloud Shell](../cloud-shell/overview.md) o [CLI de Azure](/cli/azure)

## <a name="list-role-assignments-for-a-user"></a>Lista de las asignaciones de rol de un usuario

Para mostrar las asignaciones de roles de un usuario específico use [az role assignment list](/cli/azure/role/assignment#az_role_assignment_list):

```azurecli
az role assignment list --assignee {assignee}
```

De forma predeterminada, se mostrarán únicamente las asignaciones de roles para la suscripción actual. Para ver las asignaciones de roles de la suscripción actual y anteriores, agregue el parámetro `--all`. Para ver las asignaciones de roles heredadas, agregue el parámetro `--include-inherited`.

En el ejemplo siguiente, se muestran las asignaciones de roles asignadas directamente al usuario *patlong\@contoso.com*:

```azurecli
az role assignment list --all --assignee patlong@contoso.com --output json --query '[].{principalName:principalName, roleDefinitionName:roleDefinitionName, scope:scope}'
```

```json
[
  {
    "principalName": "patlong@contoso.com",
    "roleDefinitionName": "Backup Operator",
    "scope": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales"
  },
  {
    "principalName": "patlong@contoso.com",
    "roleDefinitionName": "Virtual Machine Contributor",
    "scope": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales"
  }
]
```

## <a name="list-role-assignments-for-a-resource-group"></a>Lista de las asignaciones de roles de un grupo de recursos

Para mostrar las asignaciones de roles de un ámbito de grupo de recursos, utilice [az role assignment list](/cli/azure/role/assignment#az_role_assignment_list):

```azurecli
az role assignment list --resource-group {resourceGroup}
```

En el ejemplo siguiente se muestran las asignaciones de roles del grupo de recursos *pharma-sales*:

```azurecli
az role assignment list --resource-group pharma-sales --output json --query '[].{principalName:principalName, roleDefinitionName:roleDefinitionName, scope:scope}'
```

```json
[
  {
    "principalName": "patlong@contoso.com",
    "roleDefinitionName": "Backup Operator",
    "scope": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales"
  },
  {
    "principalName": "patlong@contoso.com",
    "roleDefinitionName": "Virtual Machine Contributor",
    "scope": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales"
  },
  
  ...

]
```

## <a name="list-role-assignments-for-a-subscription"></a>Enumeración de asignaciones de roles de una subscripción

Para enumerar todas las asignaciones de roles en un ámbito de suscripción, use [az role assignment list](/cli/azure/role/assignment#az_role_assignment_list). El identificador de suscripción lo puede encontrar en la hoja **Suscripciones** de Azure Portal, o bien, puede usar [az account list](/cli/azure/account#az_account_list).

```azurecli
az role assignment list --subscription {subscriptionNameOrId}
```

Ejemplo:

```azurecli
az role assignment list --subscription 00000000-0000-0000-0000-000000000000 --output json --query '[].{principalName:principalName, roleDefinitionName:roleDefinitionName, scope:scope}'
```

```json
[
  {
    "principalName": "admin@contoso.com",
    "roleDefinitionName": "Owner",
    "scope": "/subscriptions/00000000-0000-0000-0000-000000000000"
  },
  {
    "principalName": "Subscription Admins",
    "roleDefinitionName": "Owner",
    "scope": "/subscriptions/00000000-0000-0000-0000-000000000000"
  },
  {
    "principalName": "alain@contoso.com",
    "roleDefinitionName": "Reader",
    "scope": "/subscriptions/00000000-0000-0000-0000-000000000000"
  },

  ...

]
```

## <a name="list-role-assignments-for-a-management-group"></a>Lista de asignaciones de rol para un grupo de administración

Para enumerar todas las asignaciones de roles de un ámbito de grupo de administración, use [az role assignment list](/cli/azure/role/assignment#az_role_assignment_list). El identificador del grupo de administración lo puede encontrar en la hoja **Grupos de administración** de Azure Portal, o bien, puede usar [az account management-group list](/cli/azure/account/management-group#az_account_management_group_list).

```azurecli
az role assignment list --scope /providers/Microsoft.Management/managementGroups/{groupId}
```

Ejemplo:

```azurecli
az role assignment list --scope /providers/Microsoft.Management/managementGroups/sales-group --output json --query '[].{principalName:principalName, roleDefinitionName:roleDefinitionName, scope:scope}'
```

```json
[
  {
    "principalName": "admin@contoso.com",
    "roleDefinitionName": "Owner",
    "scope": "/providers/Microsoft.Management/managementGroups/sales-group"
  },
  {
    "principalName": "alain@contoso.com",
    "roleDefinitionName": "Reader",
    "scope": "/providers/Microsoft.Management/managementGroups/sales-group"
  }
]
```

## <a name="list-role-assignments-for-a-managed-identity"></a>Lista de asignaciones de roles para una identidad administrada

1. Obtenga el identificador de entidad de seguridad de la identidad administrada asignada por el usuario o el sistema.

    Para obtener el identificador de entidad de seguridad de una identidad administrada asignada por el usuario, puede usar [az ad sp list](/cli/azure/ad/sp#az_ad_sp_list) o [az identity list](/cli/azure/identity#az_identity_list).

    ```azurecli
    az ad sp list --display-name "{name}" --query [].objectId --output tsv
    ```

    Para obtener el identificador de entidad de seguridad de una identidad administrada asignada por el sistema, puede usar [az ad sp list](/cli/azure/ad/sp#az_ad_sp_list).

    ```azurecli
    az ad sp list --display-name "{vmname}" --query [].objectId --output tsv
    ```

1. Para mostrar las asignaciones de roles use [az role assignment list](/cli/azure/role/assignment#az_role_assignment_list).

    De forma predeterminada, se mostrarán únicamente las asignaciones de roles para la suscripción actual. Para ver las asignaciones de roles de la suscripción actual y anteriores, agregue el parámetro `--all`. Para ver las asignaciones de roles heredadas, agregue el parámetro `--include-inherited`.

    ```azurecli
    az role assignment list --assignee {objectId}
    ```

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de roles de Azure mediante la CLI de Azure](role-assignments-cli.md)