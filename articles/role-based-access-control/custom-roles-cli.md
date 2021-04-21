---
title: 'Creación o actualización de roles personalizados de Azure mediante la CLI de Azure: RBAC de Azure'
description: Aprenda a enumerar, crear, actualizar o eliminar roles personalizados de Azure mediante la CLI de Azure y el control de acceso basado en roles de Azure (RBAC de Azure).
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
ms.date: 06/17/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: d3d05ba65e0d3918f1651c36cd17700ebf74de76
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107778344"
---
# <a name="create-or-update-azure-custom-roles-using-azure-cli"></a>Creación o actualización de roles personalizados de Azure mediante la CLI de Azure

> [!IMPORTANT]
> La adición de un grupo de administración a `AssignableScopes` está actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Si los [roles integrados de Azure](built-in-roles.md) no cumplen las necesidades específicas de su organización, puede crear sus propios roles personalizados. En este artículo se describe cómo mostrar, crear, actualizar o eliminar roles personalizados con la CLI de Azure.

Para ver un tutorial paso a paso sobre cómo crear un rol personalizado, consulte [Tutorial: Creación de un rol personalizado para los recursos de Azure con la CLI de Azure](tutorial-custom-role-cli.md).

## <a name="prerequisites"></a>Requisitos previos

Para crear roles personalizados, necesita:

- Permisos para crear roles personalizados, como [Propietario](built-in-roles.md#owner) o [Administrador de acceso de usuarios](built-in-roles.md#user-access-administrator)
- [Azure Cloud Shell](../cloud-shell/overview.md) o [CLI de Azure](/cli/azure/install-azure-cli)

## <a name="list-custom-roles"></a>Lista de roles personalizados

Para enumerar los roles personalizados que están disponibles para la asignación, use [az role definition list](/cli/azure/role/definition#az_role_definition_list). En el siguiente ejemplo se enumeran todos los roles personalizados de la suscripción actual.

```azurecli
az role definition list --custom-role-only true --output json --query '[].{roleName:roleName, roleType:roleType}'
```

```json
[
  {
    "roleName": "My Management Contributor",
    "type": "CustomRole"
  },
  {
    "roleName": "My Service Reader Role",
    "type": "CustomRole"
  },
  {
    "roleName": "Virtual Machine Operator",
    "type": "CustomRole"
  }
]
```

## <a name="list-a-custom-role-definition"></a>Enumeración de definiciones de roles personalizadas

Para enumerar una definición de roles personalizada, use [az role definition list](/cli/azure/role/definition#az_role_definition_list). Este es el mismo comando que se usaría para un rol integrado.

```azurecli
az role definition list --name {roleName}
```

En el ejemplo siguiente se muestra la definición de roles *Operador de máquina virtual*:

```azurecli
az role definition list --name "Virtual Machine Operator"
```

```json
[
  {
    "assignableScopes": [
      "/subscriptions/{subscriptionId}"
    ],
    "description": "Can monitor and restart virtual machines.",
    "id": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/00000000-0000-0000-0000-000000000000",
    "name": "00000000-0000-0000-0000-000000000000",
    "permissions": [
      {
        "actions": [
          "Microsoft.Storage/*/read",
          "Microsoft.Network/*/read",
          "Microsoft.Compute/*/read",
          "Microsoft.Compute/virtualMachines/start/action",
          "Microsoft.Compute/virtualMachines/restart/action",
          "Microsoft.Authorization/*/read",
          "Microsoft.ResourceHealth/availabilityStatuses/read",
          "Microsoft.Resources/subscriptions/resourceGroups/read",
          "Microsoft.Insights/alertRules/*",
          "Microsoft.Insights/diagnosticSettings/*",
          "Microsoft.Support/*"
        ],
        "dataActions": [],
        "notActions": [],
        "notDataActions": []
      }
    ],
    "roleName": "Virtual Machine Operator",
    "roleType": "CustomRole",
    "type": "Microsoft.Authorization/roleDefinitions"
  }
]
```

En el ejemplo siguiente se muestran solo las acciones del rol *Operador de máquina virtual*:

```azurecli
az role definition list --name "Virtual Machine Operator" --output json --query '[].permissions[0].actions'
```

```json
[
  [
    "Microsoft.Storage/*/read",
    "Microsoft.Network/*/read",
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Authorization/*/read",
    "Microsoft.ResourceHealth/availabilityStatuses/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read",
    "Microsoft.Insights/alertRules/*",
    "Microsoft.Insights/diagnosticSettings/*",
    "Microsoft.Support/*"
  ]
]
```

## <a name="create-a-custom-role"></a>Crear un rol personalizado

Para crear un rol personalizado, use [az role definition create](/cli/azure/role/definition#az_role_definition_create). La definición de roles puede ser una descripción JSON o una ruta de acceso a un archivo que contenga esta descripción.

```azurecli
az role definition create --role-definition {roleDefinition}
```

En el ejemplo siguiente, se crea un rol personalizado llamado *Operador de máquina virtual*. El rol personalizado asigna acceso a todas las operaciones de lectura de los proveedores de recursos *Microsoft.Compute*, *Microsoft.Storage* y *Microsoft.Network*, y asigna acceso para iniciar, reiniciar y supervisar las máquinas virtuales. El rol personalizado se puede usar en dos suscripciones. En este ejemplo, se usa un archivo JSON como entrada.

vmoperator.json

```json
{
  "Name": "Virtual Machine Operator",
  "IsCustom": true,
  "Description": "Can monitor and restart virtual machines.",
  "Actions": [
    "Microsoft.Storage/*/read",
    "Microsoft.Network/*/read",
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Authorization/*/read",
    "Microsoft.ResourceHealth/availabilityStatuses/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read",
    "Microsoft.Insights/alertRules/*",
    "Microsoft.Support/*"
  ],
  "NotActions": [

  ],
  "AssignableScopes": [
    "/subscriptions/{subscriptionId1}",
    "/subscriptions/{subscriptionId2}"
  ]
}
```

```azurecli
az role definition create --role-definition ~/roles/vmoperator.json
```

## <a name="update-a-custom-role"></a>Actualización de un rol personalizado

Para actualizar un rol personalizado, primero use [az role definition list](/cli/azure/role/definition#az_role_definition_list) para recuperar la definición de roles. Después, haga los cambios que desee en la definición de rol. Por último, use [az role definition update](/cli/azure/role/definition#az_role_definition_update) para guardar la definición de roles actualizada.

```azurecli
az role definition update --role-definition {roleDefinition}
```

En el siguiente ejemplo, se agrega la operación *Microsoft.Insights/diagnosticSettings/* a `Actions` y se agrega un grupo de administración a `AssignableScopes` del rol personalizado de *Operador de máquina virtual*. La adición de un grupo de administración a `AssignableScopes` está actualmente en versión preliminar.

vmoperator.json

```json
{
  "Name": "Virtual Machine Operator",
  "IsCustom": true,
  "Description": "Can monitor and restart virtual machines.",
  "Actions": [
    "Microsoft.Storage/*/read",
    "Microsoft.Network/*/read",
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Authorization/*/read",
    "Microsoft.ResourceHealth/availabilityStatuses/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read",
    "Microsoft.Insights/alertRules/*",
    "Microsoft.Insights/diagnosticSettings/*",
    "Microsoft.Support/*"
  ],
  "NotActions": [

  ],
  "AssignableScopes": [
    "/subscriptions/{subscriptionId1}",
    "/subscriptions/{subscriptionId2}",
    "/providers/Microsoft.Management/managementGroups/marketing-group"
  ]
}
```

```azurecli
az role definition update --role-definition ~/roles/vmoperator.json
```

## <a name="delete-a-custom-role"></a>Eliminación de un rol personalizado

Para eliminar un rol personalizado, use [az role definition delete](/cli/azure/role/definition#az_role_definition_delete). Para especificar el rol que desea eliminar, use el nombre del rol o el identificador. Para determinar el identificador del rol, use [az role definition list](/cli/azure/role/definition#az_role_definition_list).

```azurecli
az role definition delete --name {roleNameOrId}
```

En el ejemplo siguiente se elimina el rol personalizado *Operador de máquina virtual*.

```azurecli
az role definition delete --name "Virtual Machine Operator"
```

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Creación de un rol personalizado para los recursos de Azure con la CLI de Azure](tutorial-custom-role-cli.md)
- [Roles personalizados en los recursos de Azure](custom-roles.md)
- [Operaciones del proveedor de recursos de Azure](resource-provider-operations.md)