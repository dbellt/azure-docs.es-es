---
title: 'Script de PowerShell: adición de un conjunto de datos de blob a Azure Data Share'
description: Este script de PowerShell agrega un conjunto de datos de blob a una recurso compartido existente.
services: data-share
author: joannapea
ms.author: joanpo
ms.service: data-share
ms.topic: article
ms.date: 07/07/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 0135e0ace5c71d12afbe5af720a35cecac62fc92
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110697097"
---
# <a name="use-powershell-to-create-a-data-share-in-azure"></a>Uso de PowerShell para crear un recurso compartido de datos en Azure

Este script de PowerShell agrega un conjunto de datos de blob a una instancia de Data Share existente.

## <a name="sample-script"></a>Script de ejemplo

```powershell
# Set variables with your own values
$resourceGroupName = "<Resource group name>"
$dataShareAccountName = "<Data share account name>"
$dataShareName = "<Data share name>"
$blobDatasetName = "<Dataset name>"
$blobContainer = "<Blob container name>"
$blobFilePath = "<Blob file path>"
$storageAccountResourceId = "<Storage account resource id>"

#Add a blob dataset to the datashare
New-AzDataShareDataSet -ResourceGroupName $resourceGroupName -AccountName $dataShareAccountName -ShareName $dataShareName -Name $blobDataSetName -StorageAccountResourceId $storageAccountResourceId -FilePath $blobFilePath

```


## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos: 

| Get-Help | Notas |
|---|---|
| [New-AzDataShareDataSet](/powershell/module/az.datashare/new-azdatasharedataset) | Agrega un conjunto de datos a un recurso compartido de datos. |
|||

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/).

Encontrará más ejemplos de scripts de PowerShell para Azure Data Share en el artículo [Ejemplos de PowerShell para Azure Data Share](../../samples-powershell.md).
