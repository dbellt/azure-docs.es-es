---
title: 'Creación o actualización de roles personalizados de Azure mediante Azure PowerShell: RBAC de Azure'
description: Aprenda a enumerar, crear, actualizar o eliminar roles personalizados mediante Azure PowerShell y el control de acceso basado en roles de Azure (RBAC de Azure).
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: 9e225dba-9044-4b13-b573-2f30d77925a9
ms.service: role-based-access-control
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 03/18/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: a65ab5e32a3a11becff70fb2e563d6760fa95360
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110696651"
---
# <a name="create-or-update-azure-custom-roles-using-azure-powershell"></a>Creación o actualización de roles personalizados de Azure mediante Azure PowerShell

> [!IMPORTANT]
> La adición de un grupo de administración a `AssignableScopes` está actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Si los [roles integrados de Azure](built-in-roles.md) no cumplen las necesidades específicas de su organización, puede crear sus propios roles personalizados. En este artículo se describe cómo mostrar, crear, actualizar o eliminar roles personalizados con Azure PowerShell.

Para ver un tutorial paso a paso sobre cómo crear un rol personalizado, consulte [Tutorial: Creación de un rol personalizado para los recursos de Azure con Azure PowerShell](tutorial-custom-role-powershell.md).

[!INCLUDE [az-powershell-update](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para crear roles personalizados, necesita:

- Permisos para crear roles personalizados, como [Propietario](built-in-roles.md#owner) o [Administrador de acceso de usuarios](built-in-roles.md#user-access-administrator)
- [Azure Cloud Shell](../cloud-shell/overview.md) o [Azure PowerShell](/powershell/azure/install-az-ps)

## <a name="list-custom-roles"></a>Lista de roles personalizados

Para enumerar los roles que están disponibles para la asignación en un ámbito, use el comando [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition). En el ejemplo siguiente se enumeran todos los roles disponibles para la asignación en la suscripción seleccionada.

```azurepowershell
Get-AzRoleDefinition | FT Name, IsCustom
```

```Example
Name                                              IsCustom
----                                              --------
Virtual Machine Operator                              True
AcrImageSigner                                       False
AcrQuarantineReader                                  False
AcrQuarantineWriter                                  False
API Management Service Contributor                   False
...
```

En el ejemplo siguiente se enumeran solo los roles personalizados disponibles para la asignación en la suscripción seleccionada.

```azurepowershell
Get-AzRoleDefinition -Custom | FT Name, IsCustom
```

```Example
Name                     IsCustom
----                     --------
Virtual Machine Operator     True
```

Si la suscripción seleccionada no se encuentra en la `AssignableScopes` del rol, el rol personalizado no se incluye.

## <a name="list-a-custom-role-definition"></a>Enumeración de definiciones de roles personalizadas

Para enumerar una definición de rol personalizada, use [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition). Este es el mismo comando que se usa para un rol integrado.

```azurepowershell
Get-AzRoleDefinition <role_name> | ConvertTo-Json
```

```Example
PS C:\> Get-AzRoleDefinition "Virtual Machine Operator" | ConvertTo-Json

{
  "Name": "Virtual Machine Operator",
  "Id": "00000000-0000-0000-0000-000000000000",
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
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/11111111-1111-1111-1111-111111111111"
  ]
}
```

En el ejemplo siguiente se muestran solo las acciones del rol:

```azurepowershell
(Get-AzRoleDefinition <role_name>).Actions
```

```Example
PS C:\> (Get-AzRoleDefinition "Virtual Machine Operator").Actions

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
```

## <a name="create-a-custom-role"></a>Crear un rol personalizado

Para crear un rol personalizado, use el comando [New-AzRoleDefinition](/powershell/module/az.resources/new-azroledefinition). Existen dos métodos para estructurar el rol: por medio de `PSRoleDefinition` o una plantilla JSON. 

### <a name="get-operations-for-a-resource-provider"></a>Obtención de operaciones para un proveedor de recursos

A la hora de crear roles personalizados, es importante conocer todas las operaciones posibles de los proveedores de recursos.
Puede ver la lista de [operaciones de proveedor de recursos](resource-provider-operations.md) o puede usar el comando [Get AzProviderOperation](/powershell/module/az.resources/get-azprovideroperation) para obtener esta información.
Por ejemplo, si quiere comprobar todas las operaciones disponibles para las máquinas virtuales, use este comando:

```azurepowershell
Get-AzProviderOperation <operation> | FT OperationName, Operation, Description -AutoSize
```

```Example
PS C:\> Get-AzProviderOperation "Microsoft.Compute/virtualMachines/*" | FT OperationName, Operation, Description -AutoSize

OperationName                                  Operation                                                      Description
-------------                                  ---------                                                      -----------
Get Virtual Machine                            Microsoft.Compute/virtualMachines/read                         Get the propertie...
Create or Update Virtual Machine               Microsoft.Compute/virtualMachines/write                        Creates a new vir...
Delete Virtual Machine                         Microsoft.Compute/virtualMachines/delete                       Deletes the virtu...
Start Virtual Machine                          Microsoft.Compute/virtualMachines/start/action                 Starts the virtua...
...
```

### <a name="create-a-custom-role-with-the-psroledefinition-object"></a>Creación de un rol personalizado con el objeto PSRoleDefinition

Al usar PowerShell para crear un rol personalizado, puede usar uno de los [roles integrados](built-in-roles.md) como punto de partida o puede empezar desde cero. El primer ejemplo de esta sección empieza con un rol integrado y, luego, se personaliza con más permisos. Edite los atributos para agregar cualquier elemento `Actions`, `NotActions` o `AssignableScopes` que quiera y guarde los cambios como un nuevo rol.

El ejemplo siguiente se inicia con el rol integrado [Colaborador de la máquina virtual](built-in-roles.md#virtual-machine-contributor) para crear un rol personalizado denominado *Operador de máquina virtual*. El nuevo rol concede acceso a todas las operaciones de lectura de los proveedores de recursos *Microsoft.Compute*, *Microsoft.Storage* y *Microsoft.Network*, y concede acceso para iniciar, reiniciar y supervisar las máquinas virtuales. El rol personalizado se puede usar en dos suscripciones.

```azurepowershell
$role = Get-AzRoleDefinition "Virtual Machine Contributor"
$role.Id = $null
$role.Name = "Virtual Machine Operator"
$role.Description = "Can monitor and restart virtual machines."
$role.Actions.Clear()
$role.Actions.Add("Microsoft.Storage/*/read")
$role.Actions.Add("Microsoft.Network/*/read")
$role.Actions.Add("Microsoft.Compute/*/read")
$role.Actions.Add("Microsoft.Compute/virtualMachines/start/action")
$role.Actions.Add("Microsoft.Compute/virtualMachines/restart/action")
$role.Actions.Add("Microsoft.Authorization/*/read")
$role.Actions.Add("Microsoft.ResourceHealth/availabilityStatuses/read")
$role.Actions.Add("Microsoft.Resources/subscriptions/resourceGroups/read")
$role.Actions.Add("Microsoft.Insights/alertRules/*")
$role.Actions.Add("Microsoft.Support/*")
$role.AssignableScopes.Clear()
$role.AssignableScopes.Add("/subscriptions/00000000-0000-0000-0000-000000000000")
$role.AssignableScopes.Add("/subscriptions/11111111-1111-1111-1111-111111111111")
New-AzRoleDefinition -Role $role
```

En el ejemplo siguiente muestra otra forma de crear el rol personalizado de *Operador de máquina virtual*. Se inicia al crear un nuevo objeto `PSRoleDefinition`. Las operaciones de acción se especifican en la variable `perms` y se establecen en la propiedad `Actions`. La propiedad `NotActions` se establece al leer `NotActions` desde el rol integrado [Colaborador de la máquina virtual](built-in-roles.md#virtual-machine-contributor). Puesto que el [colaborador de la máquina Virtual](built-in-roles.md#virtual-machine-contributor) no tiene ningún `NotActions`, esta línea no es necesaria, pero muestra cómo se puede recuperar la información desde otro rol.

```azurepowershell
$role = [Microsoft.Azure.Commands.Resources.Models.Authorization.PSRoleDefinition]::new()
$role.Name = 'Virtual Machine Operator 2'
$role.Description = 'Can monitor and restart virtual machines.'
$role.IsCustom = $true
$perms = 'Microsoft.Storage/*/read','Microsoft.Network/*/read','Microsoft.Compute/*/read'
$perms += 'Microsoft.Compute/virtualMachines/start/action','Microsoft.Compute/virtualMachines/restart/action'
$perms += 'Microsoft.Authorization/*/read'
$perms += 'Microsoft.ResourceHealth/availabilityStatuses/read'
$perms += 'Microsoft.Resources/subscriptions/resourceGroups/read'
$perms += 'Microsoft.Insights/alertRules/*','Microsoft.Support/*'
$role.Actions = $perms
$role.NotActions = (Get-AzRoleDefinition -Name 'Virtual Machine Contributor').NotActions
$subs = '/subscriptions/00000000-0000-0000-0000-000000000000','/subscriptions/11111111-1111-1111-1111-111111111111'
$role.AssignableScopes = $subs
New-AzRoleDefinition -Role $role
```

### <a name="create-a-custom-role-with-json-template"></a>Creación de un rol personalizado con una plantilla JSON

Se puede usar una plantilla JSON como definición de origen para el rol personalizado. En el ejemplo siguiente se crea un rol personalizado que permite el acceso de lectura a recursos de almacenamiento y proceso y el acceso a la asistencia técnica, y además agrega ese rol a dos suscripciones. Cree un archivo `C:\CustomRoles\customrole1.json` con el siguiente ejemplo. El identificador se debe establecer en `null` durante la creación de rol inicial ya que se generará un nuevo identificador automáticamente. 

```json
{
  "Name": "Custom Role 1",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows for read access to Azure storage and compute resources and access to support",
  "Actions": [
    "Microsoft.Compute/*/read",
    "Microsoft.Storage/*/read",
    "Microsoft.Support/*"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/00000000-0000-0000-0000-000000000000",
    "/subscriptions/11111111-1111-1111-1111-111111111111"
  ]
}
```

Para agregar el rol a las suscripciones, ejecute el siguiente comando de PowerShell:

```azurepowershell
New-AzRoleDefinition -InputFile "C:\CustomRoles\customrole1.json"
```

## <a name="update-a-custom-role"></a>Actualización de un rol personalizado

De modo similar a la creación de un rol personalizado, puede modificar un rol personalizado existente por medio de `PSRoleDefinition` o una plantilla JSON.

### <a name="update-a-custom-role-with-the-psroledefinition-object"></a>Actualización de un rol personalizado con el objeto PSRoleDefinition

Para modificar un rol personalizado, use en primer lugar el comando [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition) para recuperar la definición de rol. Después, haga los cambios que desee en la definición de rol. Por último, use el comando [Set-AzRoleDefinition](/powershell/module/az.resources/set-azroledefinition) para guardar la definición de rol modificada.

En el ejemplo siguiente se agrega la operación `Microsoft.Insights/diagnosticSettings/*` al rol personalizado *Operador de máquina virtual* .

```azurepowershell
$role = Get-AzRoleDefinition "Virtual Machine Operator"
$role.Actions.Add("Microsoft.Insights/diagnosticSettings/*")
Set-AzRoleDefinition -Role $role
```

```Example
PS C:\> $role = Get-AzRoleDefinition "Virtual Machine Operator"
PS C:\> $role.Actions.Add("Microsoft.Insights/diagnosticSettings/*")
PS C:\> Set-AzRoleDefinition -Role $role

Name             : Virtual Machine Operator
Id               : 88888888-8888-8888-8888-888888888888
IsCustom         : True
Description      : Can monitor and restart virtual machines.
Actions          : {Microsoft.Storage/*/read, Microsoft.Network/*/read, Microsoft.Compute/*/read,
                   Microsoft.Compute/virtualMachines/start/action...}
NotActions       : {}
AssignableScopes : {/subscriptions/00000000-0000-0000-0000-000000000000,
                   /subscriptions/11111111-1111-1111-1111-111111111111}
```

En el ejemplo siguiente se agrega una suscripción de Azure a los ámbitos asignables del rol personalizado *Operador de máquina virtual* .

```azurepowershell
Get-AzSubscription -SubscriptionName Production3

$role = Get-AzRoleDefinition "Virtual Machine Operator"
$role.AssignableScopes.Add("/subscriptions/22222222-2222-2222-2222-222222222222")
Set-AzRoleDefinition -Role $role
```

```Example
PS C:\> Get-AzSubscription -SubscriptionName Production3

Name     : Production3
Id       : 22222222-2222-2222-2222-222222222222
TenantId : 99999999-9999-9999-9999-999999999999
State    : Enabled

PS C:\> $role = Get-AzRoleDefinition "Virtual Machine Operator"
PS C:\> $role.AssignableScopes.Add("/subscriptions/22222222-2222-2222-2222-222222222222")
PS C:\> Set-AzRoleDefinition -Role $role

Name             : Virtual Machine Operator
Id               : 88888888-8888-8888-8888-888888888888
IsCustom         : True
Description      : Can monitor and restart virtual machines.
Actions          : {Microsoft.Storage/*/read, Microsoft.Network/*/read, Microsoft.Compute/*/read,
                   Microsoft.Compute/virtualMachines/start/action...}
NotActions       : {}
AssignableScopes : {/subscriptions/00000000-0000-0000-0000-000000000000,
                   /subscriptions/11111111-1111-1111-1111-111111111111,
                   /subscriptions/22222222-2222-2222-2222-222222222222}
```

En el ejemplo siguiente se agrega un grupo de administración a `AssignableScopes` del rol personalizado *Operador de máquina virtual*. La adición de un grupo de administración a `AssignableScopes` está actualmente en versión preliminar.

```azurepowershell
Get-AzManagementGroup

$role = Get-AzRoleDefinition "Virtual Machine Operator"
$role.AssignableScopes.Add("/providers/Microsoft.Management/managementGroups/{groupId1}")
Set-AzRoleDefinition -Role $role
```

```Example
PS C:\> Get-AzManagementGroup

Id          : /providers/Microsoft.Management/managementGroups/marketing-group
Type        : /providers/Microsoft.Management/managementGroups
Name        : marketing-group
TenantId    : 99999999-9999-9999-9999-999999999999
DisplayName : Marketing group

PS C:\> $role = Get-AzRoleDefinition "Virtual Machine Operator"
PS C:\> $role.AssignableScopes.Add("/providers/Microsoft.Management/managementGroups/marketing-group")
PS C:\> Set-AzRoleDefinition -Role $role

Name             : Virtual Machine Operator
Id               : 88888888-8888-8888-8888-888888888888
IsCustom         : True
Description      : Can monitor and restart virtual machines.
Actions          : {Microsoft.Storage/*/read, Microsoft.Network/*/read, Microsoft.Compute/*/read,
                   Microsoft.Compute/virtualMachines/start/action...}
NotActions       : {}
AssignableScopes : {/subscriptions/00000000-0000-0000-0000-000000000000,
                   /subscriptions/11111111-1111-1111-1111-111111111111,
                   /subscriptions/22222222-2222-2222-2222-222222222222,
                   /providers/Microsoft.Management/managementGroups/marketing-group}
```

### <a name="update-a-custom-role-with-a-json-template"></a>Actualización de un rol personalizado con una plantilla JSON

Con la anterior plantilla JSON, puede modificar fácilmente un rol personalizado existente para agregar o quitar acciones. Actualice la plantilla JSON y agregue la acción de lectura para las redes tal y como se muestra en el siguiente ejemplo. Las definiciones que aparecen en la plantilla no se aplican de manera acumulativa a una definición existente, lo que significa que el rol aparecerá exactamente como lo especifique en la plantilla. También que necesita actualizar el identificador con el identificador del rol. Si no está seguro de cuál es este valor, puede usar el cmdlet [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition) para obtener esta información.

```json
{
  "Name": "Custom Role 1",
  "Id": "acce7ded-2559-449d-bcd5-e9604e50bad1",
  "IsCustom": true,
  "Description": "Allows for read access to Azure storage and compute resources and access to support",
  "Actions": [
    "Microsoft.Compute/*/read",
    "Microsoft.Storage/*/read",
    "Microsoft.Network/*/read",
    "Microsoft.Support/*"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/00000000-0000-0000-0000-000000000000",
    "/subscriptions/11111111-1111-1111-1111-111111111111"
  ]
}
```

Para actualizar el rol existente, ejecute el siguiente comando de PowerShell:

```azurepowershell
Set-AzRoleDefinition -InputFile "C:\CustomRoles\customrole1.json"
```

## <a name="delete-a-custom-role"></a>Eliminación de un rol personalizado

Para eliminar un rol personalizado, use el comando [Remove-AzRoleDefinition](/powershell/module/az.resources/remove-azroledefinition).

En el ejemplo siguiente se quita el rol personalizado *Operador de máquina virtual* .

```azurepowershell
Get-AzRoleDefinition "Virtual Machine Operator"
Get-AzRoleDefinition "Virtual Machine Operator" | Remove-AzRoleDefinition
```

```Example
PS C:\> Get-AzRoleDefinition "Virtual Machine Operator"

Name             : Virtual Machine Operator
Id               : 88888888-8888-8888-8888-888888888888
IsCustom         : True
Description      : Can monitor and restart virtual machines.
Actions          : {Microsoft.Storage/*/read, Microsoft.Network/*/read, Microsoft.Compute/*/read,
                   Microsoft.Compute/virtualMachines/start/action...}
NotActions       : {}
AssignableScopes : {/subscriptions/00000000-0000-0000-0000-000000000000,
                   /subscriptions/11111111-1111-1111-1111-111111111111}

PS C:\> Get-AzRoleDefinition "Virtual Machine Operator" | Remove-AzRoleDefinition

Confirm
Are you sure you want to remove role definition with name 'Virtual Machine Operator'.
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y
```

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Creación de un rol personalizado de Azure mediante Azure PowerShell](tutorial-custom-role-powershell.md)
- [Roles personalizados en los recursos de Azure](custom-roles.md)
- [Operaciones del proveedor de recursos de Azure](resource-provider-operations.md)
