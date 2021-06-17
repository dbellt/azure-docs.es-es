---
title: 'Script de PowerShell: Aceptación de una invitación de una instancia de Azure Data Share'
description: Este script de PowerShell acepta invitaciones de un recurso compartido de datos existente.
services: data-share
author: joannapea
ms.author: joanpo
ms.service: data-share
ms.topic: article
ms.date: 07/07/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8ce2a315512af65147fa2354c5895bcff94710cf
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110697163"
---
# <a name="use-powershell-to-accept-a-data-share-invitation"></a>Uso de PowerShell para aceptar una invitación de un recurso compartido de datos

Este script de PowerShell acepta las invitaciones enviadas a un consumidor.

## <a name="sample-script"></a>Script de ejemplo
```powershell
#List invitations sent to a consumer
Get-AzDataShareInvitation

# Set variables with your own values
$resourceGroupName = "<Resource group name>"
$dataShareAccountName = "<Data share account name>"
$dataShareName = "<Data share name>"
$invitationId = "<Invitation id>"

#Accept a specific invitation by creating a share subscription
New-AzDataShareSubscription -ResourceGroupName $resourceGroupName -AccountName $dataShareAccountName -Name $dataShareName -InvitationId $invitationId

```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos: 

| Get-Help | Notas |
|---|---|
| [Get-AzDataShareInvitation](/powershell/module/az.datashare/get-azdatashareinvitation) | Obtiene y enumera las invitaciones de recurso compartido de datos enviadas. |
| [New-AzDataShareSubscription](/powershell/module/az.datashare/get-azdatasharesubscription) | Cree una suscripción de un recurso compartido de datos. |
|||

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/).

Encontrará más ejemplos de scripts de PowerShell para Azure Data Share en el artículo [Ejemplos de PowerShell para Azure Data Share](../../samples-powershell.md).
