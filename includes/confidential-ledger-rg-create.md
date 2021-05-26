---
author: msmbaldwin
ms.service: confidential-ledger
ms.topic: include
ms.date: 05/05/2021
ms.author: msmbaldwin
ms.openlocfilehash: bebab5bbab271b83b6b7e039bdf68d6254bd5fe2
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388600"
---
Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Use el comando [az group create](/cli/azure/group#az_group_create) de la CLI de Azure o el cmdlet [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) de Azure PowerShell para crear un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az group create --name "myResourceGroup" -l "EastUS"
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
New-AzResourceGroup -Name myResourceGroup -Location eastus
```
---
