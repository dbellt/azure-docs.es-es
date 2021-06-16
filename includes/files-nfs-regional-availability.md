---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 09/15/2020
ms.author: rogarana
ms.custom: include file, devx-track-azurepowershell
ms.openlocfilehash: 75eef9053defcf4e57639fa6dc6a9b327fa65287
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110720246"
---
NFS se admite en **TODAS** las regiones donde está disponible el almacenamiento de archivos prémium, que son más de 30.

Agregamos regiones continuamente. Para obtener la lista más actualizada, use el siguiente ejemplo para consultar la lista de regiones que admiten NFS. También puede comprobar la compatibilidad de las regiones en la [página Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=storage&regions=all), en **Almacenamiento de archivos prémium**.

```azurepowershell-interactive
# Log in first with Connect-AzAccount if not using Cloud Shell

$azContext = Get-AzContext
$azProfile = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile
$profileClient = New-Object -TypeName Microsoft.Azure.Commands.ResourceManager.Common.RMProfileClient -ArgumentList ($azProfile)
$token = $profileClient.AcquireAccessToken($azContext.Subscription.TenantId)
$authHeader = @{
    'Content-Type'='application/json'
    'Authorization'='Bearer ' + $token.AccessToken
}

# Provide specific subscription id if you want  list for a different subscription
$subscription = $azContext.Subscription.Id

# Invoke the REST API
$restUri = "https://management.azure.com/subscriptions/$subscription/providers/Microsoft.Storage/skus?api-version=2019-06-01"
$response = Invoke-RestMethod -Uri $restUri -Method Get -Headers $authHeader

# List of all regions that has NFS support.
$response.value| Where-Object -FilterScript {$_.capabilities| Where-Object { $_.name -eq 'supportsNfsShare' -and $_.value -eq 'true'}}| Select-Object locations, kind, name

# List of regions that support NFS Zonal redundancy.
$response.value| Where-Object -FilterScript {($_.name -EQ 'Premium_ZRS') -and ($_.capabilities| Where-Object { $_.name -eq 'supportsNfsShare' -and $_.value -eq 'true'})}| Select-Object locations
```

Respuesta de muestra
```
List of regions that support NFS Zonal redundancy
locations
---------
{eastus}
{eastus2}
{westeurope}
{southeastasia}
{japaneast}
{northeurope}
{australiaeast}
{westus2}
{uksouth}
{eastus2euap}
{francecentral}
```
