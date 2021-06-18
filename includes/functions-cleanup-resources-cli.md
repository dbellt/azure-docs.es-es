---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 10/18/2020
ms.author: glenga
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 50d9e0e8ed9e6ac3d406570aa25d3a14cedab9b5
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110720763"
---
## <a name="clean-up-resources"></a>Limpieza de recursos

Si continúa con el [paso siguiente](#next-steps) y agrega un enlace de salida de la cola de Azure Storage, conserve todos los recursos intactos, ya que va a crear a partir de lo que ya ha hecho.

De lo contrario, use el siguiente comando para eliminar el grupo de recursos y todos los recursos que contiene para evitar incurrir en costos adicionales.

 # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az group delete --name AzureFunctionsQuickstart-rg
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell
Remove-AzResourceGroup -Name AzureFunctionsQuickstart-rg
```

---
