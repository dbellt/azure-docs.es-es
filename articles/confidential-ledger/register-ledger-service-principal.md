---
title: Registro de una entidad de servicio de Ledger con Microsoft Azure Confidential Ledger
description: Registro de una entidad de servicio de Ledger con Microsoft Azure Confidential Ledger
services: confidential-ledger
author: msmbaldwin
ms.service: confidential-ledger
ms.topic: overview
ms.date: 04/15/2021
ms.author: mbaldwin
ms.openlocfilehash: a4f751bcb262011b073207751ac590edeb41d391
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388591"
---
# <a name="register-a-confidential-ledger-service-principal"></a>Registro de una entidad de servicio de Confidential Ledger

Para asociar una cuenta de almacenamiento a su instancia de Confidential Ledger, primero debe registrar la entidad de servicio de Confidential Ledger.

## <a name="create-a-service-principal"></a>Creación de una entidad de servicio

Para crear una entidad de servicio, ejecute el comando [az ad sp create](/cli/azure/ad/sp#az_ad_sp_create) de la CLI de Azure o los cmdlets [Connect-AzureAD](/powershell/module/azuread/connect-azuread) y [New-AzureADServicePrincipal](/powershell/module/azuread/new-azureadserviceprincipal) de Azure PowerShell.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli-interactive
az ad sp create --id 4353526e-1c33-4fcf-9e82-9683edf52848
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell-interactive
Connect-AzureAD -TenantId "<tenant-id-of-customer>"
New-AzureADServicePrincipal -AppId 4353526e-1c33-4fcf-9e82-9683edf52848 -DisplayName ConfidentialLedger
```
---

## <a name="assign-roles"></a>Asignación de roles

Establezca el "[Colaborador de datos de Blob Storage](../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)" de IAM para la entidad de servicio de Confidential Ledger de la cuenta de almacenamiento. Puede hacerlo con el comando [az role assignment create](/cli/azure/role/assignment) de la CLI de Azure o el cmdlet [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment) de Azure PowerShell.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli-interactive
az role assignment create --role "Storage Blob Data Contributor" --assignee "4353526e-1c33-4fcf-9e82-9683edf52848" --scope "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>"
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell-interactive
New-AzRoleAssignment -ApplicationId 4353526e-1c33-4fcf-9e82-9683edf52848 -RoleDefinitionName "Storage Blob Data Contributor" -Scope "/subscriptions/<subscription-id>/resourceGroups/sample-resource-group/providers/Microsoft.Storage/storageAccounts/<storage-account>"
```
---

## <a name="next-steps"></a>Pasos siguientes

- [Información general de Microsoft Azure Confidential Ledger](overview.md)
