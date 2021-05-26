---
author: msmbaldwin
ms.service: confidential-ledger
ms.topic: include
ms.date: 05/05/2021
ms.author: msmbaldwin
ms.openlocfilehash: 985c50f39476c3b1c52670f06d51fde5b3156978
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388598"
---
Inicie sesión en Azure mediante el comando [az login](/cli/azure/reference-index#az_login) de la CLI de Azure o el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) de Azure PowerShell.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli-interactive
az login
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell-interactive
Connect-AzAccount
```
---

Si la CLI o PowerShell pueden abrir el explorador predeterminado, lo harán y cargarán una página de inicio de sesión de Azure. En caso contrario, visite [https://aka.ms/devicelogin](https://aka.ms/devicelogin) y escriba el código de autorización que se muestra en el terminal.

Si se le pide, inicie sesión con las credenciales de su cuenta en el explorador.
