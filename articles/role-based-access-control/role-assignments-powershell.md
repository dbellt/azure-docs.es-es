---
title: 'Asignación de roles de Azure mediante Azure PowerShell: RBAC de Azure'
description: Aprenda a conceder acceso a recursos de Azure para usuarios, grupos, entidades de servicio e identidades administradas mediante el control de acceso basado en rol (RBAC) de Azure y Azure PowerShell.
services: active-directory
author: rolyon
manager: daveba
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 02/15/2021
ms.author: rolyon
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: a67bb6a79e6ef3e51269090611d18c0711fb5bbd
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110691037"
---
# <a name="assign-azure-roles-using-azure-powershell"></a>Asignación de roles de Azure mediante Azure PowerShell

[!INCLUDE [Azure RBAC definition grant access](../../includes/role-based-access-control/definition-grant.md)] En este artículo se describe cómo asignar roles con Azure PowerShell.

[!INCLUDE [az-powershell-update](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para asignar roles, debe tener:

- Permisos `Microsoft.Authorization/roleAssignments/write`, como [Administrador de acceso de usuarios](built-in-roles.md#user-access-administrator) o [Propietario](built-in-roles.md#owner)
- [PowerShell en Azure Cloud Shell](../cloud-shell/overview.md) o [Azure PowerShell](/powershell/azure/install-az-ps).
- La cuenta que use para ejecutar el comando de PowerShell debe tener el permiso `Directory.Read.All` de Microsoft Graph.

## <a name="steps-to-assign-an-azure-role"></a>Pasos para asignar un rol de Azure

Para asignar un rol, se necesitan tres elementos: entidad de seguridad, definición de rol y ámbito.

### <a name="step-1-determine-who-needs-access"></a>Paso 1: determinar quién necesita acceso

Puede asignar un rol a un usuario, grupo, entidad de servicio o identidad administrada. Para asignar un rol, es posible que tenga que especificar el identificador único del objeto. El identificador tiene el formato: `11111111-1111-1111-1111-111111111111`. Puede obtener el identificador mediante Azure Portal o Azure PowerShell.

**User**

Para un usuario Azure AD, obtenga el nombre principal de usuario, como *patlong\@contoso.com* o el id. de objeto del usuario. Para obtener el identificador de objeto, puede usar [Get-AzADUser](/powershell/module/az.resources/get-azaduser).

```azurepowershell
Get-AzADUser -StartsWith <userName>
(Get-AzADUser -DisplayName <userName>).id
```

**Grupo**

Para un grupo de Azure AD, necesita el id. de objeto de grupo. Para obtener el identificador de objeto, puede usar [Get-AzADGroup](/powershell/module/az.resources/get-azadgroup).

```azurepowershell
Get-AzADGroup -SearchString <groupName>
(Get-AzADGroup -DisplayName <groupName>).id
```

**Entidad de servicio**

Para una entidad de servicio de Azure AD (identidad que usa una aplicación), necesita el id. de objeto de la entidad de servicio. Para obtener el identificador de objeto, puede usar [Get-AzADServicePrincipal](/powershell/module/az.resources/get-azadserviceprincipal). Para una entidad de servicio, use el id. de objeto y **no** el de aplicación.

```azurepowershell
Get-AzADServicePrincipal -SearchString <principalName>
(Get-AzADServicePrincipal -DisplayName <principalName>).id
```

**Identidad administrada**

Para una identidad administrada que haya asignado el sistema o el usuario, es necesario el id. de objeto. Para obtener el identificador de objeto, puede usar [Get-AzADServicePrincipal](/powershell/module/az.resources/get-azadserviceprincipal).

```azurepowershell
Get-AzADServicePrincipal -SearchString <principalName>
(Get-AzADServicePrincipal -DisplayName <principalName>).id
```
    
### <a name="step-2-select-the-appropriate-role"></a>Paso 2: seleccione el rol apropiado

Los permisos se agrupan en roles. Puede seleccionarlos de una lista de varios [roles integrados de Azure](built-in-roles.md) o puede utilizar sus propios roles personalizados. Es recomendable conceder acceso con el menor privilegio necesario, así que evite asignar un rol más amplio.

Para enumerar los roles y obtener el identificador de rol único, puede usar [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition).

```azurepowershell
Get-AzRoleDefinition | FT Name, IsCustom, Id
```

Aquí verá cómo mostrar los detalles de un rol determinado.

```azurepowershell
Get-AzRoleDefinition <roleName>
```

Para obtener más información, consulte la [Lista de definiciones de roles de Azure](role-definitions-list.md#azure-powershell).
 
### <a name="step-3-identify-the-needed-scope"></a>Paso 3: identificar el ámbito necesario

Azure proporciona cuatro niveles de ámbito: recursos, [grupo de recursos](../azure-resource-manager/management/overview.md#resource-groups), suscripción y [grupo de administración](../governance/management-groups/overview.md). Es recomendable conceder acceso con el menor privilegio necesario, así que evite asignar un rol a un ámbito más amplio. Para obtener más información sobre el ámbito, vea [Comprensión del ámbito](scope-overview.md).

**Ámbito de recursos**

Para el ámbito de recursos, necesitará el id. de recurso. Puede buscar el id. de recurso examinando las propiedades del recurso en Azure Portal. El id. de recurso tiene el formato siguiente.

```
/subscriptions/<subscriptionId>/resourcegroups/<resourceGroupName>/providers/<providerName>/<resourceType>/<resourceSubType>/<resourceName>
```

**Ámbito de grupo de recursos**

Para el ámbito del grupo de recursos, necesita el nombre del grupo de recursos. Puede encontrar el nombre en la página **Grupos de recursos** de Azure Portal o puede usar [Get-AzResourceGroup](/powershell/module/az.resources/get-azresourcegroup).

```azurepowershell
Get-AzResourceGroup
```

**Ámbito de suscripción** 

Para el ámbito de la suscripción, necesitará el id. de la suscripción. Puede encontrar el identificador en la página **Suscripciones** de Azure Portal o puede usar [Get-AzSubscription](/powershell/module/az.accounts/get-azsubscription).

```azurepowershell
Get-AzSubscription
```

**Ámbito del grupo de administración** 

Para el ámbito del grupo de administración, necesita el nombre del grupo de administración. Puede encontrar el nombre en la página **Grupos de administración** de Azure Portal o puede usar [Get-AzManagementGroup](/powershell/module/az.resources/get-azmanagementgroup).

```azurepowershell
Get-AzManagementGroup
```
    
### <a name="step-4-assign-role"></a>Paso 4: Asignación de un rol

Para asignar un rol, use el comando [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment). Dependiendo del ámbito, el comando normalmente tiene uno de los siguientes formatos.

**Ámbito de recursos**

```azurepowershell
New-AzRoleAssignment -ObjectId <objectId> `
-RoleDefinitionName <roleName> `
-Scope /subscriptions/<subscriptionId>/resourcegroups/<resourceGroupName>/providers/<providerName>/<resourceType>/<resourceSubType>/<resourceName>
```

```azurepowershell
New-AzRoleAssignment -ObjectId <objectId> `
-RoleDefinitionId <roleId> `
-ResourceName <resourceName> `
-ResourceType <resourceType> `
-ResourceGroupName <resourceGroupName>
```

**Ámbito de grupo de recursos**

```azurepowershell
New-AzRoleAssignment -SignInName <emailOrUserprincipalname> `
-RoleDefinitionName <roleName> `
-ResourceGroupName <resourceGroupName>
```

```azurepowershell
New-AzRoleAssignment -ObjectId <objectId> `
-RoleDefinitionName <roleName> `
-ResourceGroupName <resourceGroupName>
```

**Ámbito de suscripción** 

```azurepowershell
New-AzRoleAssignment -SignInName <emailOrUserprincipalname> `
-RoleDefinitionName <roleName> `
-Scope /subscriptions/<subscriptionId>
```

```azurepowershell
New-AzRoleAssignment -ObjectId <objectId> `
-RoleDefinitionName <roleName> `
-Scope /subscriptions/<subscriptionId>
```

**Ámbito del grupo de administración** 

```azurepowershell
New-AzRoleAssignment -SignInName <emailOrUserprincipalname> `
-RoleDefinitionName <roleName> `
-Scope /providers/Microsoft.Management/managementGroups/<groupName>
``` 

```azurepowershell
New-AzRoleAssignment -ObjectId <objectId> `
-RoleDefinitionName <roleName> `
-Scope /providers/Microsoft.Management/managementGroups/<groupName>
``` 
    
## <a name="assign-role-examples"></a>Ejemplos de asignación de roles

#### <a name="assign-a-role-for-all-blob-containers-in-a-storage-account-resource-scope"></a>Asignación de un rol a todos los contenedores de blobs en un ámbito de recursos de la cuenta de almacenamiento

Asigna el rol de [Colaborador de datos de Blob Storage](built-in-roles.md#storage-blob-data-contributor) a una entidad de servicio con el id. de objeto *55555555-5555-5555-5555-555555555555* en el ámbito de recursos de una cuenta de almacenamiento denominada *storage12345*.

```azurepowershell
PS C:\> New-AzRoleAssignment -ObjectId 55555555-5555-5555-5555-555555555555 `
-RoleDefinitionName "Storage Blob Data Contributor" `
-Scope "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/storage12345"

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/storage12345/providers/Microsoft.Authorization/roleAssignments/cccccccc-cccc-cccc-cccc-cccccccccccc
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/storage12345
DisplayName        : example-identity
SignInName         :
RoleDefinitionName : Storage Blob Data Contributor
RoleDefinitionId   : ba92f5b4-2d11-453d-a403-e96b0029c9fe
ObjectId           : 55555555-5555-5555-5555-555555555555
ObjectType         : ServicePrincipal
CanDelegate        : False
```

#### <a name="assign-a-role-for-a-specific-blob-container-resource-scope"></a>Asignación de un rol para un ámbito de recursos de un contenedor de blobs específico

Asigna el rol de [Colaborador de datos de Blob Storage](built-in-roles.md#storage-blob-data-contributor) a una entidad de servicio con el id. de objeto *55555555-5555-5555-5555-555555555555* en el ámbito de recursos de un contenedor de blobs denominado *blob-container-01*.

```azurepowershell
PS C:\> New-AzRoleAssignment -ObjectId 55555555-5555-5555-5555-555555555555 `
-RoleDefinitionName "Storage Blob Data Contributor" `
-Scope "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/storage12345/blobServices/default/containers/blob-container-01"

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/storage12345/blobServices/default/containers/blob-container-01/providers/Microsoft.Authorization/roleAssignm
                     ents/dddddddd-dddd-dddd-dddd-dddddddddddd
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/storage12345/blobServices/default/containers/blob-container-01
DisplayName        : example-identity
SignInName         :
RoleDefinitionName : Storage Blob Data Contributor
RoleDefinitionId   : ba92f5b4-2d11-453d-a403-e96b0029c9fe
ObjectId           : 55555555-5555-5555-5555-555555555555
ObjectType         : ServicePrincipal
CanDelegate        : False
```

#### <a name="assign-a-role-for-a-group-in-a-specific-virtual-network-resource-scope"></a>Asignación de un rol para un grupo en un ámbito de recursos de red virtual específico

Asigna el rol [Colaborador de la máquina virtual](built-in-roles.md#virtual-machine-contributor) al grupo *Pharma Sales Admins* con el identificador aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa en el ámbito del recurso de una red virtual denominada *pharma-sales-project-network*.

```azurepowershell
PS C:\> New-AzRoleAssignment -ObjectId aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa `
-RoleDefinitionName "Virtual Machine Contributor" `
-ResourceName pharma-sales-project-network `
-ResourceType Microsoft.Network/virtualNetworks `
-ResourceGroupName MyVirtualNetworkResourceGroup

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/MyVirtualNetworkResourceGroup
                     /providers/Microsoft.Network/virtualNetworks/pharma-sales-project-network/providers/Microsoft.Authorizat
                     ion/roleAssignments/bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/MyVirtualNetworkResourceGroup
                     /providers/Microsoft.Network/virtualNetworks/pharma-sales-project-network
DisplayName        : Pharma Sales Admins
SignInName         :
RoleDefinitionName : Virtual Machine Contributor
RoleDefinitionId   : 9980e02c-c2be-4d73-94e8-173b1dc7cf3c
ObjectId           : aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa
ObjectType         : Group
CanDelegate        : False
```

#### <a name="assign-a-role-for-a-user-at-a-resource-group-scope"></a>Asignación de un rol a un usuario en el ámbito de un grupo de recursos

Asigna el rol [Colaborador de la máquina virtual](built-in-roles.md#virtual-machine-contributor) al usuario *patlong\@contoso.com* en el ámbito del grupo de recursos *pharma-sales*.

```azurepowershell
PS C:\> New-AzRoleAssignment -SignInName patlong@contoso.com `
-RoleDefinitionName "Virtual Machine Contributor" `
-ResourceGroupName pharma-sales

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales/pr
                     oviders/Microsoft.Authorization/roleAssignments/55555555-5555-5555-5555-555555555555
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales
DisplayName        : Pat Long
SignInName         : patlong@contoso.com
RoleDefinitionName : Virtual Machine Contributor
RoleDefinitionId   : 9980e02c-c2be-4d73-94e8-173b1dc7cf3c
ObjectId           : 44444444-4444-4444-4444-444444444444
ObjectType         : User
CanDelegate        : False
```

Como alternativa, puede especificar el grupo de recursos completo con el parámetro `-Scope`:

```azurepowershell
PS C:\> New-AzRoleAssignment -SignInName patlong@contoso.com `
-RoleDefinitionName "Virtual Machine Contributor" `
-Scope "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales"

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales/providers/Microsoft.Authorization/roleAssignments/55555555-5555-5555-5555-555555555555
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales
DisplayName        : Pat Long
SignInName         : patlong@contoso.com
RoleDefinitionName : Virtual Machine Contributor
RoleDefinitionId   : 9980e02c-c2be-4d73-94e8-173b1dc7cf3c
ObjectId           : 44444444-4444-4444-4444-444444444444
ObjectType         : User
CanDelegate        : False
```

#### <a name="assign-a-role-for-a-user-using-the-unique-role-id-at-a-resource-group-scope"></a>Asignación de un rol a un usuario con el identificador de rol único en el ámbito de un grupo de recursos

Hay un par de veces en las que el nombre de un rol puede cambiar, por ejemplo:

- Utiliza su propio rol personalizado y decide cambiar el nombre.
- Utiliza una rol de versión preliminar que tiene **(Versión preliminar)** en el nombre. Cuando se libera el rol, se cambia su nombre.

Incluso si se cambia el nombre de un rol, su identificador no cambia. Si utiliza scripts o automatización para crear las asignaciones de roles, se recomienda utilizar el identificador de rol único en lugar del nombre del rol. Por tanto, si se cambia el nombre de un rol, es más probable que los scripts funcionen.

En el ejemplo siguiente, se asigna el rol [Colaborador de la máquina virtual](built-in-roles.md#virtual-machine-contributor) al usuario *patlong\@contoso.com* en el ámbito del grupo de recursos *pharma-sales*.

```azurepowershell
PS C:\> New-AzRoleAssignment -ObjectId 44444444-4444-4444-4444-444444444444 `
-RoleDefinitionId 9980e02c-c2be-4d73-94e8-173b1dc7cf3c `
-Scope "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales"

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales/providers/Microsoft.Authorization/roleAssignments/55555555-5555-5555-5555-555555555555
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales
DisplayName        : Pat Long
SignInName         : patlong@contoso.com
RoleDefinitionName : Virtual Machine Contributor
RoleDefinitionId   : 9980e02c-c2be-4d73-94e8-173b1dc7cf3c
ObjectId           : 44444444-4444-4444-4444-444444444444
ObjectType         : User
CanDelegate        : False
```

#### <a name="assign-a-role-for-an-application-at-a-resource-group-scope"></a>Asignación de un rol a una aplicación en el ámbito de un grupo de recursos

Asigna el rol [Colaborador de la máquina virtual](built-in-roles.md#virtual-machine-contributor) a una aplicación con el identificador de objeto de la entidad de servicio 77777777-7777-7777-7777-777777777777 en el ámbito del grupo de recursos *pharma-sales*.

```azurepowershell
PS C:\> New-AzRoleAssignment -ObjectId 77777777-7777-7777-7777-777777777777 `
-RoleDefinitionName "Virtual Machine Contributor" `
-ResourceGroupName pharma-sales

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/66666666-6666-6666-6666-666666666666
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales
DisplayName        : MyApp1
SignInName         :
RoleDefinitionName : Virtual Machine Contributor
RoleDefinitionId   : 9980e02c-c2be-4d73-94e8-173b1dc7cf3c
ObjectId           : 77777777-7777-7777-7777-777777777777
ObjectType         : ServicePrincipal
CanDelegate        : False
```

#### <a name="assign-a-role-for-a-user-at-a-subscription-scope"></a>Asignación de un rol a un usuario en el ámbito de una suscripción

Asigna el rol de [Lector](built-in-roles.md#reader) al usuario *annm\@example.com* en el ámbito de una suscripción.

```azurepowershell
PS C:\> New-AzRoleAssignment -SignInName annm@example.com `
-RoleDefinitionName "Reader" `
-Scope "/subscriptions/00000000-0000-0000-0000-000000000000"

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/66666666-6666-6666-6666-666666666666
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000
DisplayName        : Ann M
SignInName         : annm@example.com
RoleDefinitionName : Reader
RoleDefinitionId   : acdd72a7-3385-48ef-bd42-f606fba81ae7
ObjectId           : 77777777-7777-7777-7777-777777777777
ObjectType         : ServicePrincipal
CanDelegate        : False
```

#### <a name="assign-a-role-for-a-user-at-a-management-group-scope"></a>Asignación de un rol a un usuario en el ámbito de un grupo de administración

Asigna el rol [Lector de facturación](built-in-roles.md#billing-reader) al usuario *alain\@example.com* en un ámbito de grupo de administración.

```azurepowershell
PS C:\> New-AzRoleAssignment -SignInName alain@example.com `
-RoleDefinitionName "Billing Reader" `
-Scope "/providers/Microsoft.Management/managementGroups/marketing-group"

RoleAssignmentId   : /providers/Microsoft.Management/managementGroups/marketing-group/providers/Microsoft.Authorization/roleAssignments/22222222-2222-2222-2222-222222222222
Scope              : /providers/Microsoft.Management/managementGroups/marketing-group
DisplayName        : Alain Charon
SignInName         : alain@example.com
RoleDefinitionName : Billing Reader
RoleDefinitionId   : fa23ad8b-c56e-40d8-ac0c-ce449e1d2c64
ObjectId           : 44444444-4444-4444-4444-444444444444
ObjectType         : User
CanDelegate        : False
```

## <a name="next-steps"></a>Pasos siguientes

- [Enumeración de asignaciones de roles de Azure mediante Azure PowerShell](role-assignments-list-powershell.md)
- [Tutorial: Concesión de acceso de grupo a recursos de Azure mediante Azure PowerShell](tutorial-role-assignments-group-powershell.md)
- [Administración de recursos con Azure PowerShell](../azure-resource-manager/management/manage-resources-powershell.md)
