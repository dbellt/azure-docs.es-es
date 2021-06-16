---
title: 'Enumeración de asignaciones de denegación de Azure mediante Azure PowerShell: RBAC de Azure'
description: Aprenda a enumerar los usuarios, grupos, entidades de servicio e identidades administradas a los que se les ha denegado el acceso a acciones específicas en recursos de Azure en un ámbito determinado mediante Azure PowerShell y el control de acceso basado en roles de Azure (RBAC de Azure).
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 06/12/2019
ms.author: rolyon
ms.reviewer: bagovind
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: ab180dcbf12b43a3bd60a88767169877be5e7e2f
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110696551"
---
# <a name="list-azure-deny-assignments-using-azure-powershell"></a>Enumeración de asignaciones de denegación de Azure mediante Azure PowerShell

Las [asignaciones de denegación de Azure](deny-assignments.md) impiden que los usuarios realicen acciones concretas en recursos de Azure, aunque una asignación de roles les conceda acceso. En este artículo se describe cómo enumerar las asignaciones de denegación mediante Azure PowerShell.

> [!NOTE]
> No se pueden crear directamente asignaciones de denegación propias. Para más información sobre cómo se crean las asignaciones de denegación, consulte [Descripción de las asignaciones de denegación para recursos de Azure](deny-assignments.md).

## <a name="prerequisites"></a>Prerrequisitos

Para obtener información sobre una asignación de denegación, debe tener lo siguiente:

- El permiso `Microsoft.Authorization/denyAssignments/read`, que está incluido en la mayoría de los [roles integrados de Azure](built-in-roles.md).
- [PowerShell en Azure Cloud Shell](../cloud-shell/overview.md) o [Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="list-deny-assignments"></a>Enumeración de asignaciones de denegación

### <a name="list-all-deny-assignments"></a>Enumeración de todas las asignaciones de denegación

Para enumerar todas las asignaciones de denegación para la suscripción actual, use [Get-AzDenyAssignment](/powershell/module/az.resources/get-azdenyassignment).

```azurepowershell
Get-AzDenyAssignment
```

```Example
PS C:\> Get-AzDenyAssignment

Id                      : 22222222-2222-2222-2222-222222222222
DenyAssignmentName      : Deny assignment '22222222-2222-2222-2222-222222222222' created by Blueprint Assignment
                          '/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Blueprint/blueprintAssignments/assignment-locked-storageaccount-TestingBPLocks'.
Description             : Created by Blueprint Assignment '/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Blueprint/blueprintAssignments/assignment-locked-storageaccount-TestingBPLocks'.
Actions                 : {*}
NotActions              : {*/read}
DataActions             : {}
NotDataActions          : {}
Scope                   : /subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/TestingBPLocks
DoNotApplyToChildScopes : True
Principals              : {
                          DisplayName:  All Principals
                          ObjectType:   SystemDefined
                          ObjectId:     00000000-0000-0000-0000-000000000000
                          }
ExcludePrincipals       : {
                          ObjectType:   ServicePrincipal
                          }
IsSystemProtected       : True

Id                      : 33333333-3333-3333-3333-333333333333
DenyAssignmentName      : Deny assignment '33333333-3333-3333-3333-333333333333' created by Blueprint Assignment
                          '/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Blueprint/blueprintAssignments/assignment-locked-storageaccount-TestingBPLocks'.
Description             : Created by Blueprint Assignment '/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Blueprint/blueprintAssignments/assignment-locked-storageaccount-TestingBPLocks'.
Actions                 : {*}
NotActions              : {*/read}
DataActions             : {}
NotDataActions          : {}
Scope                   : /subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/TestingBPLocks/providers/Microsoft.Storage/storageAccounts/storep6vkuxmu4m4pq
DoNotApplyToChildScopes : True
Principals              : {
                          DisplayName:  All Principals
                          ObjectType:   SystemDefined
                          ObjectId:     00000000-0000-0000-0000-000000000000
                          }
ExcludePrincipals       : {
                          DisplayName:  assignment-locked-storageaccount-TestingBPLocks
                          ObjectType:   ServicePrincipal
                          ObjectId:     2311a0b7-657a-4ca2-af6f-d1c33f6d2fff
                          }
IsSystemProtected       : True
```

### <a name="list-deny-assignments-at-a-resource-group-scope"></a>Enumeración de asignaciones de denegación en un ámbito de grupo de recursos

Para enumerar todas las asignaciones en un ámbito de grupo de recursos, use [Get-AzDenyAssignment](/powershell/module/az.resources/get-azdenyassignment).

```azurepowershell
Get-AzDenyAssignment -ResourceGroupName <resource_group_name>
```

```Example
PS C:\> Get-AzDenyAssignment -ResourceGroupName TestingBPLocks | FL DenyAssignmentName, Scope

DenyAssignmentName : Deny assignment '22222222-2222-2222-2222-222222222222' created by Blueprint Assignment
                     '/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Blueprint/blueprintAssignments/assignment-locked-storageaccount-TestingBPLocks'.
Scope              : /subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/TestingBPLocks
Principals         : {
                     DisplayName:       All Principals
                     ObjectType:        SystemDefined
                     ObjectId:  00000000-0000-0000-0000-000000000000
                     }
```

### <a name="list-deny-assignments-at-a-subscription-scope"></a>Enumeración de asignaciones de denegación en un ámbito de suscripción

Para enumerar todas las asignaciones en un ámbito de suscripción, use [Get-AzDenyAssignment](/powershell/module/az.resources/get-azdenyassignment). Para obtener el identificador de suscripción, lo puede encontrar en la hoja **Suscripciones** de Azure Portal o bien puede usar [Get-AzSubscription](/powershell/module/Az.Accounts/Get-AzSubscription).

```azurepowershell
Get-AzDenyAssignment -Scope /subscriptions/<subscription_id>
```

```Example
PS C:\> Get-AzDenyAssignment -Scope /subscriptions/11111111-1111-1111-1111-111111111111
```

## <a name="next-steps"></a>Pasos siguientes

- [Descripción de las asignaciones de denegación para recursos de Azure](deny-assignments.md)
- [Enumeración de asignaciones de denegación de Azure mediante Azure Portal](deny-assignments-portal.md)
- [Enumeración de asignaciones de denegación de Azure mediante la API REST](deny-assignments-rest.md)
