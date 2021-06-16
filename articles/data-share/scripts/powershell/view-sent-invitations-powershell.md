---
title: 'Script de PowerShell: Enumeración de las invitaciones de Azure Data Share enviadas a un consumidor'
description: Obtenga información sobre cómo este script de PowerShell envía invitaciones a un consumidor y vea un ejemplo del script que puede usar.
author: joannapea
ms.author: joanpo
ms.service: data-share
ms.topic: article
ms.date: 07/07/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 0b39c31271a0382cce3388e9c8176ba943bc43e6
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110703878"
---
# <a name="use-powershell-to-get-a-data-share-invitation"></a>Uso de PowerShell para obtener una invitación de un recurso compartido de datos

Este script de PowerShell obtiene las invitaciones enviadas a un consumidor.

## <a name="sample-script"></a>Script de ejemplo
```powershell
# Set variables with your own values
$invitationId = "<invitationId>"
$location = "<location>"

#List invitations sent to a consumer
Get-AzDataShareInvitation

#Get a specific invitation sent to a consumer
Get-AzDataShareInvitation -location -invitationId 

```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos: 

| Get-Help | Notas |
|---|---|
| [Get-AzDataShareInvitation](/powershell/module/az.datashare/get-azdatashareinvitation) | Obtiene y enumera las invitaciones de recurso compartido de datos enviadas. |
|||

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/).

Encontrará más ejemplos de scripts de PowerShell para Azure Data Share en el artículo [Ejemplos de PowerShell para Azure Data Share](../../samples-powershell.md).
