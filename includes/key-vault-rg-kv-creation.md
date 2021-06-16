---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 07/20/2020
ms.author: msmbaldwin
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: fe21da6b8cd2a3f7291ae4a27ef7de8040a2acd1
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110720502"
---
En este inicio rápido se usa un almacén de claves de Azure creado previamente. Puede crear un almacén de claves siguiendo los pasos descritos en el [inicio rápido de CLI de Azure](../articles/key-vault/general/quick-create-cli.md), [inicio rápido de Azure PowerShell](../articles/key-vault/general/quick-create-powershell.md) o [inicio rápido de Azure Portal](../articles/key-vault/general/quick-create-portal.md). 

Como alternativa, puede ejecutar simplemente los siguientes comandos de la CLI de Azure o de Azure PowerShell.

> [!Important]
> Cada almacén de claves debe tener un nombre único. Reemplace <nombre-almacén de claves-único> por el nombre del almacén de claves en los ejemplos siguientes.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az group create --name "myResourceGroup" -l "EastUS"

az keyvault create --name "<your-unique-keyvault-name>" -g "myResourceGroup"
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
New-AzResourceGroup -Name myResourceGroup -Location eastus

New-AzKeyVault -Name <your-unique-keyvault-name> -ResourceGroupName myResourceGroup -Location eastus
```
---
