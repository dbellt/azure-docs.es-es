---
title: 'Enumeración de las definiciones de roles de Azure: RBAC de Azure'
description: Aprenda a enumerar los roles integrados y personalizados de Azure mediante Azure Portal, Azure PowerShell, la CLI de Azure o la API REST.
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 05/06/2021
ms.author: rolyon
ms.openlocfilehash: 96eb88bf1e62753cff5606881af471d32bf1e85c
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109483714"
---
# <a name="list-azure-role-definitions"></a>Enumeración de las definiciones de roles de Azure

Una definición de roles es una colección de permisos que se pueden realizar, por ejemplo, de lectura, escritura y eliminación. Suele denominarse un rol. [El control de acceso basado en rol (RBAC) de Azure](overview.md) tiene más de 120 [roles integrados](built-in-roles.md) o puede crear sus propios roles personalizados. En este artículo se describe cómo mostrar en una lista los roles integrados y personalizados que puede usar para conceder acceso a los recursos de Azure.

Para ver la lista de los roles de administrador de Azure Active Directory, consulte [Permisos de roles de administrador en Azure Active Directory](../active-directory/roles/permissions-reference.md).

## <a name="azure-portal"></a>Azure Portal

### <a name="list-all-roles"></a>Lista de todos los roles

Siga estos pasos para enumerar todos los roles de Azure Portal.

La pestaña **Roles** se actualizó recientemente con algunas características adicionales. Si quiere ver la experiencia anterior, consulte la pestaña **Roles (Classic)** (Roles [clásico]). Puede usar cualquiera de las pestañas de roles para trabajar con los roles; sin embargo, si crea o elimina roles personalizados, es posible que tenga que actualizar manualmente la página para ver los últimos cambios.

#### <a name="roles"></a>[Roles](#tab/roles/)

1. En Azure Portal, haga clic en **Todos los servicios** y luego seleccione cualquier ámbito. Por ejemplo, puede seleccionar **Grupos de administración**, **Suscripciones**, **Grupos de recursos** o un recurso.

1. Haga clic en el recurso específico.

1. Haga clic en **Control de acceso (IAM).**

1. Haga clic en la pestaña **Roles** para ver una lista de todos los roles integrados y personalizados.

   ![Captura de pantalla que muestra la lista de roles con la nueva experiencia.](./media/role-definitions-list/roles-list.png)

1. Para ver los permisos de un rol determinado, en la columna **Detalles**, haga clic en el vínculo **Ver**.

    Aparece un panel de permisos.

1. Haga clic en la pestaña **Permisos** para ver y buscar los permisos del rol seleccionado.

   ![Captura de pantalla que muestra los permisos de rol con la nueva experiencia.](./media/role-definitions-list/role-permissions.png)

#### <a name="roles-classic"></a>[Roles (Classic)](#tab/roles-classic/) (Roles [clásico])

1. En Azure Portal, haga clic en **Todos los servicios** y luego seleccione cualquier ámbito. Por ejemplo, puede seleccionar **Grupos de administración**, **Suscripciones**, **Grupos de recursos** o un recurso.

1. Haga clic en el recurso específico.

1. Haga clic en **Control de acceso (IAM).**

1. Haga clic en la pestaña **Roles (Classic)** (Roles [clásico]) para ver una lista de todos los roles integrados y personalizados.

   Puede ver el número de usuarios y grupos asignados a cada rol en el ámbito actual.

   ![Lista Roles](./media/role-definitions-list/roles-list-classic.png)

---

## <a name="azure-powershell"></a>Azure PowerShell

### <a name="list-all-roles"></a>Lista de todos los roles

Para enumerar todos los roles en Azure PowerShell, use [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition).

```azurepowershell
Get-AzRoleDefinition | FT Name, Description
```

```Example
AcrImageSigner                                    acr image signer
AcrQuarantineReader                               acr quarantine data reader
AcrQuarantineWriter                               acr quarantine data writer
API Management Service Contributor                Can manage service and the APIs
API Management Service Operator Role              Can manage service but not the APIs
API Management Service Reader Role                Read-only access to service and APIs
Application Insights Component Contributor        Can manage Application Insights components
Application Insights Snapshot Debugger            Gives user permission to use Application Insights Snapshot Debugge...
Automation Job Operator                           Create and Manage Jobs using Automation Runbooks.
Automation Operator                               Automation Operators are able to start, stop, suspend, and resume ...
...
```

### <a name="list-a-role-definition"></a>Enumeración de una definición de roles

Para enumerar los detalles de un rol específico, use [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition).

```azurepowershell
Get-AzRoleDefinition <role_name>
```

```Example
PS C:\> Get-AzRoleDefinition "Contributor"

Name             : Contributor
Id               : b24988ac-6180-42a0-ab88-20f7382dd24c
IsCustom         : False
Description      : Lets you manage everything except access to resources.
Actions          : {*}
NotActions       : {Microsoft.Authorization/*/Delete, Microsoft.Authorization/*/Write,
                   Microsoft.Authorization/elevateAccess/Action}
DataActions      : {}
NotDataActions   : {}
AssignableScopes : {/}
```

### <a name="list-a-role-definition-in-json-format"></a>Enumeración de una definición de roles en formato JSON

Para mostrar un rol en formato JSON, use [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition).

```azurepowershell
Get-AzRoleDefinition <role_name> | ConvertTo-Json
```

```Example
PS C:\> Get-AzRoleDefinition "Contributor" | ConvertTo-Json

{
  "Name": "Contributor",
  "Id": "b24988ac-6180-42a0-ab88-20f7382dd24c",
  "IsCustom": false,
  "Description": "Lets you manage everything except access to resources.",
  "Actions": [
    "*"
  ],
  "NotActions": [
    "Microsoft.Authorization/*/Delete",
    "Microsoft.Authorization/*/Write",
    "Microsoft.Authorization/elevateAccess/Action",
    "Microsoft.Blueprint/blueprintAssignments/write",
    "Microsoft.Blueprint/blueprintAssignments/delete"
  ],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/"
  ]
}
```

### <a name="list-permissions-of-a-role-definition"></a>Lista de los permisos de una definición de roles

Para enumerar los permisos de un rol específico, use [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition).

```azurepowershell
Get-AzRoleDefinition <role_name> | FL Actions, NotActions
```

```Example
PS C:\> Get-AzRoleDefinition "Contributor" | FL Actions, NotActions

Actions    : {*}
NotActions : {Microsoft.Authorization/*/Delete, Microsoft.Authorization/*/Write,
             Microsoft.Authorization/elevateAccess/Action,
             Microsoft.Blueprint/blueprintAssignments/write...}
```

```azurepowershell
(Get-AzRoleDefinition <role_name>).Actions
```

```Example
PS C:\> (Get-AzRoleDefinition "Virtual Machine Contributor").Actions

Microsoft.Authorization/*/read
Microsoft.Compute/availabilitySets/*
Microsoft.Compute/locations/*
Microsoft.Compute/virtualMachines/*
Microsoft.Compute/virtualMachineScaleSets/*
Microsoft.DevTestLab/schedules/*
Microsoft.Insights/alertRules/*
Microsoft.Network/applicationGateways/backendAddressPools/join/action
Microsoft.Network/loadBalancers/backendAddressPools/join/action
...
```

## <a name="azure-cli"></a>Azure CLI

### <a name="list-all-roles"></a>Lista de todos los roles

Para obtener una lista de todos los roles de la CLI de Azure, use [az role definition list](/cli/azure/role/definition#az_role_definition_list).

```azurecli
az role definition list
```

En el ejemplo siguiente se muestra el nombre y la descripción de todas las definiciones de roles disponibles:

```azurecli
az role definition list --output json --query '[].{roleName:roleName, description:description}'
```

```json
[
  {
    "description": "Can manage service and the APIs",
    "roleName": "API Management Service Contributor"
  },
  {
    "description": "Can manage service but not the APIs",
    "roleName": "API Management Service Operator Role"
  },
  {
    "description": "Read-only access to service and APIs",
    "roleName": "API Management Service Reader Role"
  },

  ...

]
```

En el ejemplo siguiente se muestran todos los roles integrados.

```azurecli
az role definition list --custom-role-only false --output json --query '[].{roleName:roleName, description:description, roleType:roleType}'
```

```json
[
  {
    "description": "Can manage service and the APIs",
    "roleName": "API Management Service Contributor",
    "roleType": "BuiltInRole"
  },
  {
    "description": "Can manage service but not the APIs",
    "roleName": "API Management Service Operator Role",
    "roleType": "BuiltInRole"
  },
  {
    "description": "Read-only access to service and APIs",
    "roleName": "API Management Service Reader Role",
    "roleType": "BuiltInRole"
  },
  
  ...

]
```

### <a name="list-a-role-definition"></a>Enumeración de una definición de roles

Para mostrar los detalles de un rol, use [az role definition list](/cli/azure/role/definition#az_role_definition_list).

```azurecli
az role definition list --name {roleName}
```

En el ejemplo siguiente se muestra la definición de roles de *Colaborador*:

```azurecli
az role definition list --name "Contributor"
```

```json
[
  {
    "assignableScopes": [
      "/"
    ],
    "description": "Lets you manage everything except access to resources.",
    "id": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
    "name": "b24988ac-6180-42a0-ab88-20f7382dd24c",
    "permissions": [
      {
        "actions": [
          "*"
        ],
        "dataActions": [],
        "notActions": [
          "Microsoft.Authorization/*/Delete",
          "Microsoft.Authorization/*/Write",
          "Microsoft.Authorization/elevateAccess/Action",
          "Microsoft.Blueprint/blueprintAssignments/write",
          "Microsoft.Blueprint/blueprintAssignments/delete"
        ],
        "notDataActions": []
      }
    ],
    "roleName": "Contributor",
    "roleType": "BuiltInRole",
    "type": "Microsoft.Authorization/roleDefinitions"
  }
]
```

### <a name="list-permissions-of-a-role-definition"></a>Lista de los permisos de una definición de roles

En el ejemplo siguiente, se muestran solo los valores de *actions* y *notActions* del rol *Colaborador*.

```azurecli
az role definition list --name "Contributor" --output json --query '[].{actions:permissions[0].actions, notActions:permissions[0].notActions}'
```

```json
[
  {
    "actions": [
      "*"
    ],
    "notActions": [
      "Microsoft.Authorization/*/Delete",
      "Microsoft.Authorization/*/Write",
      "Microsoft.Authorization/elevateAccess/Action",
      "Microsoft.Blueprint/blueprintAssignments/write",
      "Microsoft.Blueprint/blueprintAssignments/delete"
    ]
  }
]
```

En el ejemplo siguiente se muestran solo los valores de actions del rol *Colaborador de máquina virtual*.

```azurecli
az role definition list --name "Virtual Machine Contributor" --output json --query '[].permissions[0].actions'
```

```json
[
  [
    "Microsoft.Authorization/*/read",
    "Microsoft.Compute/availabilitySets/*",
    "Microsoft.Compute/locations/*",
    "Microsoft.Compute/virtualMachines/*",
    "Microsoft.Compute/virtualMachineScaleSets/*",
    "Microsoft.Compute/disks/write",
    "Microsoft.Compute/disks/read",
    "Microsoft.Compute/disks/delete",
    "Microsoft.DevTestLab/schedules/*",
    "Microsoft.Insights/alertRules/*",
    "Microsoft.Network/applicationGateways/backendAddressPools/join/action",
    "Microsoft.Network/loadBalancers/backendAddressPools/join/action",

    ...

    "Microsoft.Storage/storageAccounts/listKeys/action",
    "Microsoft.Storage/storageAccounts/read",
    "Microsoft.Support/*"
  ]
]
```

## <a name="rest-api"></a>API DE REST

### <a name="list-role-definitions"></a>Lista de definiciones de roles

Para enumerar las definiciones de roles, use la API de REST [Definiciones de roles: lista](/rest/api/authorization/roledefinitions/list). Para mejorar los resultados, especifique un ámbito y un filtro opcional.

1. Empiece con la solicitud siguiente:

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions?$filter={$filter}&api-version=2015-07-01
    ```

1. En el identificador URI, reemplace *{scope}* por el ámbito cuya lista de definiciones de roles quiere obtener.

    > [!div class="mx-tableFixed"]
    > | Ámbito | Tipo |
    > | --- | --- |
    > | `providers/Microsoft.Management/managementGroups/{groupId1}` | Grupo de administración |
    > | `subscriptions/{subscriptionId1}` | Subscription |
    > | `subscriptions/{subscriptionId1}/resourceGroups/myresourcegroup1` | Resource group |
    > | `subscriptions/{subscriptionId1}/resourceGroups/myresourcegroup1/providers/Microsoft.Web/sites/mysite1` | Resource |

    En el ejemplo anterior, microsoft.web es un proveedor de recursos que hace referencia a una instancia de App Service. De forma similar, puede usar cualquier otro proveedor de recursos y especificar el ámbito. Para más información, consulte [Tipos y proveedores de recursos de Azure](../azure-resource-manager/management/resource-providers-and-types.md) y [Operaciones del proveedor de recursos de Azure](resource-provider-operations.md) compatibles.  
     
1. Reemplace *{filter}* por la condición que quiere aplicar para filtrar la lista de definiciones de roles.

    > [!div class="mx-tableFixed"]
    > | Filter | Descripción |
    > | --- | --- |
    > | `$filter=atScopeAndBelow()` | Muestra las definiciones de roles para el ámbito especificado y cualquier subámbito. |
    > | `$filter=type+eq+'{type}'` | Muestra las definiciones de roles del tipo especificado. El tipo de rol puede ser `CustomRole` o `BuiltInRole`. |

La siguiente solicitud enumera las definiciones de roles personalizadas en el ámbito de la suscripción:

```http
GET https://management.azure.com/subscriptions/{subscriptionId1}/providers/Microsoft.Authorization/roleDefinitions?api-version=2015-07-01&$filter=type+eq+'CustomRole'
```

El siguiente texto muestra un ejemplo de la salida:

```json
{
    "value": [
        {
            "properties": {
                "roleName": "Billing Reader Plus",
                "type": "CustomRole",
                "description": "Read billing data and download invoices",
                "assignableScopes": [
                    "/subscriptions/{subscriptionId1}"
                ],
                "permissions": [
                    {
                        "actions": [
                            "Microsoft.Authorization/*/read",
                            "Microsoft.Billing/*/read",
                            "Microsoft.Commerce/*/read",
                            "Microsoft.Consumption/*/read",
                            "Microsoft.Management/managementGroups/read",
                            "Microsoft.CostManagement/*/read",
                            "Microsoft.Billing/invoices/download/action",
                            "Microsoft.CostManagement/exports/*"
                        ],
                        "notActions": [
                            "Microsoft.CostManagement/exports/delete"
                        ]
                    }
                ],
                "createdOn": "2020-02-21T04:49:13.7679452Z",
                "updatedOn": "2020-02-21T04:49:13.7679452Z",
                "createdBy": "{createdByObjectId1}",
                "updatedBy": "{updatedByObjectId1}"
            },
            "id": "/subscriptions/{subscriptionId1}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId1}",
            "type": "Microsoft.Authorization/roleDefinitions",
            "name": "{roleDefinitionId1}"
        }
    ]
}
```

### <a name="list-a-role-definition"></a>Enumeración de una definición de roles

Para mostrar los detalles de un rol específico, use la API de REST [Definiciones de roles: obtención](/rest/api/authorization/roledefinitions/get) o [Definiciones de roles: obtención por id.](/rest/api/authorization/roledefinitions/getbyid)

1. Empiece con la solicitud siguiente:

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}?api-version=2015-07-01
    ```

    Para obtener una definición de roles de nivel de directorio, puede usar esta solicitud:

    ```http
    GET https://management.azure.com/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}?api-version=2015-07-01
    ```

1. En el identificador URI, reemplace *{scope}* por el ámbito cuya lista de definiciones de roles quiere obtener.

    > [!div class="mx-tableFixed"]
    > | Ámbito | Tipo |
    > | --- | --- |
    > | `providers/Microsoft.Management/managementGroups/{groupId1}` | Grupo de administración |
    > | `subscriptions/{subscriptionId1}` | Subscription |
    > | `subscriptions/{subscriptionId1}/resourceGroups/myresourcegroup1` | Resource group |
    > | `subscriptions/{subscriptionId1}/resourceGroups/myresourcegroup1/providers/Microsoft.Web/sites/mysite1` | Resource |
     
1. Reemplace *{roleDefinitionId}* por el identificador de la definición de roles.

La siguiente solicitud enumera la definición de roles de [lector](built-in-roles.md#reader):

```http
GET https://management.azure.com/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7?api-version=2015-07-01
```

El siguiente texto muestra un ejemplo de la salida:

```json
{
    "properties": {
        "roleName": "Reader",
        "type": "BuiltInRole",
        "description": "Lets you view everything, but not make any changes.",
        "assignableScopes": [
            "/"
        ],
        "permissions": [
            {
                "actions": [
                    "*/read"
                ],
                "notActions": []
            }
        ],
        "createdOn": "2015-02-02T21:55:09.8806423Z",
        "updatedOn": "2019-02-05T21:24:35.7424745Z",
        "createdBy": null,
        "updatedBy": null
    },
    "id": "/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7",
    "type": "Microsoft.Authorization/roleDefinitions",
    "name": "acdd72a7-3385-48ef-bd42-f606fba81ae7"
}
```

## <a name="next-steps"></a>Pasos siguientes

- [Roles integrados en los recursos de Azure](built-in-roles.md)
- [Roles personalizados en los recursos de Azure](custom-roles.md)
- [Enumeración de asignaciones de roles de Azure mediante Azure Portal](role-assignments-list-portal.md)
- [Asignación de roles de Azure mediante Azure Portal](role-assignments-portal.md)
