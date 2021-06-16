---
title: 'Script de PowerShell: creación de una instancia de Azure Data Share'
description: Este script de PowerShell crea un recurso compartido de datos en una cuenta existente de Data Share.
author: joannapea
ms.service: data-share
ms.topic: article
ms.date: 07/07/2019
ms.author: joanpo
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 3a3d4c85f5063623abd8d86304efcc3bcc375d22
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110704081"
---
# <a name="use-powershell-to-create-a-data-share-in-azure"></a>Uso de PowerShell para crear un recurso compartido de datos en Azure

Este script de PowerShell crea un recurso compartido de datos en una cuenta existente de Data Share.

## <a name="sample-script"></a>Script de ejemplo

```powershell

# Set variables with your own values
$resourceGroupName = "<Resource group name>"
$dataShareAccountName = "<Data share account name>"
$dataShareName = "<Data share name>"

# Create a new Azure Data Share
New-AzDataShare -ResourceGroupName $resourceGroupName -AccountName $dataShareAccountName -Name $dataShareName

```


## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos: 

| Get-Help | Notas |
|---|---|
| [New-AzDataShare](/powershell/module/az.datashare/new-azdatashare) | Crea un recurso compartido de datos. |
|||

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/).

Encontrará más ejemplos de scripts de PowerShell para Azure Data Share en el artículo [Ejemplos de PowerShell para Azure Data Share](../../samples-powershell.md).
