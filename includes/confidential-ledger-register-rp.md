---
author: msmbaldwin
ms.service: confidential-ledger
ms.topic: include
ms.date: 05/05/2021
ms.author: msmbaldwin
ms.openlocfilehash: 5a94cdae7cc332b8e0df5b2ebdc6ae57b67c8a68
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388594"
---
Un proveedor de recursos es un servicio que proporciona recursos de Azure. Use el comando de la CLI de Azure [az provider register](/cli/azure/provider#az_provider_register) o el cmdlet de Azure PowerShell [Register-AzureRmResourceProvider](/powershell/module/azurerm.resources/register-azurermresourceprovider) para registrar el proveedor de recursos Confidential Ledger, "microsoft.ConfidentialLedger".

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az provider register --namespace "microsoft.ConfidentialLedger"
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Register-AzureRmResourceProvider -ProviderNamespace "microsoft.ConfidentialLedger"
```
---

Puede comprobar que el registro se ha completado con el comando de la CLI de Azure [az provider register](/cli/azure/provider#az_provider_show) o el cmdlet de Azure PowerShell [Get-AzureRmResourceProvider](/powershell/module/azurerm.resources/get-azurermresourceprovider).

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az provider show --namespace "microsoft.ConfidentialLedger"
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzureRmResourceProvider -ProviderNamespace "microsoft.ConfidentialLedger"
```
---
