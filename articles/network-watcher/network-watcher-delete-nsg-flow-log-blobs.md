---
title: Eliminación de blobs de almacenamiento de registros de flujo de grupo de seguridad de red en Azure Network Watcher | Microsoft Docs
description: En este artículo se explica cómo eliminar los blobs de almacenamiento de registros de flujo de grupo de seguridad de red que están fuera del período de la directiva de retención en Azure Network Watcher.
services: network-watcher
documentationcenter: na
author: damendo
manager: ''
editor: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/07/2021
ms.author: damendo
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 9ef7160c667ed9fa5f932b49c295a1247ff94c30
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110665355"
---
# <a name="delete-network-security-group-flow-log-storage-blobs-in-network-watcher"></a>Eliminación de blobs de almacenamiento de registros de flujo de grupo de seguridad de red en Network Watcher

En caso de que alguna vez necesite eliminar manualmente los registros de flujo de la cuenta de almacenamiento, puede usar el siguiente script de PowerShell.
Este script solo elimina los blobs de almacenamiento que son más antiguos que la directiva de retención existente especificada por el usuario.

## <a name="run-powershell-script-to-delete-nsg-flow-logs"></a>Ejecución de un script de PowerShell para eliminar registros de flujo de NSG
 
Copie y guarde el siguiente script en una ubicación, como el directorio donde esté trabajando actualmente. 

```powershell
# This powershell script deletes all NSG flow log blobs that should not be retained anymore as per configured retention policy.
# While configuring NSG flow logs on Azure portal, the user configures the retention period of NSG flow log blobs in
# their storage account (in days).
# This script reads all blobs and deletes blobs that are not to be retained (outside retention window)
# if the retention days are zero; all blobs are retained forever and hence no blobs are deleted.
#
#

param (
        [string] [Parameter(Mandatory=$true)]  $SubscriptionId,
        [string] [Parameter(Mandatory=$true)]  $Location,
        [switch] [Parameter(Mandatory=$false)] $Confirm
    )

Login-AzAccount

$SubId = Get-AzSubscription| Where-Object {$_.Id.contains($SubscriptionId.ToLower())}

if ($SubId.Count -eq 0)
{
    Write-Error 'The SubscriptionId does not exist' -ErrorAction Stop
}

Set-AzContext -SubscriptionId $SubscriptionId

$NsgList = Get-AzNetworkSecurityGroup | Where-Object {$_.Location -eq $Location}
$NW = Get-AzNetworkWatcher | Where-Object {$_.Location -eq $Location}

$FlowLogsList = @()
foreach ($Nsg in $NsgList)
{
    # Query Flow Log Status which are enabled
    $NsgFlowLog = Get-AzNetworkWatcherFlowLogStatus -NetworkWatcher $NW -TargetResourceId $Nsg.Id | Where-Object {$_.Enabled -eq "True"}
    if ($NsgFlowLog.Count -gt 0)
    {
        $FlowLogsList +=  $NsgFlowLog
        Write-Output ('Enabled NSG found: ' +  $NsgFlowLog.TargetResourceId)
    }
}

foreach ($Psflowlog in $FlowLogsList)
{
    $RetentionDays = $Psflowlog.RetentionPolicy.Days
    if ($RetentionDays -le 0)
    {
        continue
    }

    $Strings = $Psflowlog.StorageId -split '/'
    $RGName = $Strings[4]
    $StorageAccountName = $Strings[-1]

    $Key = (Get-AzStorageAccountKey -ResourceGroupName $RGName -Name $StorageAccountName).Value[1]
    $StorageAccount = New-AzStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $Key

    $ContainerName = 'insights-logs-networksecuritygroupflowevent'  
    $BLobsList = Get-AzStorageBlob -Container $ContainerName -Context $StorageAccount.Context

    $TargetBLobsList = $BLobsList | Where-Object {$_.Name.contains($Psflowlog.TargetResourceId.ToUpper())}

    $RetentionDate = Get-Date
    $RetentionDate = $RetentionDate.AddDays(-1*$RetentionDays)
    $RetentionDateInUTC = $RetentionDate.ToUniversalTime()

    foreach ($Blob in $TargetBLobsList)
    {
        $BlobLastModifietedDTinUTC = [datetime]$Blob.LastModified.UtcDateTime

        if ($BlobLastModifietedDTinUTC -ge  $RetentionDateInUTC)
        {
            Write-Output ($Blob.Name + '===>' + $BlobLastModifietedDTinUTC  + ' ===> RETAINED')
            continue
        }

        if ($Confirm)
        {
            Write-Output (Blob to be deleted: $Blob.Name)
            $Confirmation = Read-Host "Are you sure you want to remove this blob (Y/N)?"
        }

        if ((-not $Confirm) -or ($Confirmation -eq 'Y'))
        {
            Write-Output ($Blob.Name + '===>' + $BlobLastModifietedDTinUTC  + ' ===> DELETED')
            Remove-AzStorageBlob -Container $ContainerName -Context $StorageAccount.Context -Blob $Blob.Name
        }
        else
        {
            Write-Output ($Blob.Name + '===>' + $BlobLastModifietedDTinUTC  + ' ===> RETAINED')
        }
    }
}

Write-Output ('Retention policy for all NSGs evaluated and completed successfully')
```

1. Escriba los siguientes parámetros en el script según sea necesario:
   - **SubscriptionId** [obligatorio]:identificador de la suscripción desde la que quiere eliminar los blobs de registro de flujo de NSG.
   - **Ubicación** [obligatorio]: La _cadena de ubicación_ de la región de los registros NSG donde quiere eliminar los blobs de registro de flujo de NSG. Esta información se puede encontrar en Azure Portal o en [GitHub](https://github.com/Azure/azure-extensions-cli/blob/beb3d3fe984cfa9c7798cb11a274c5337968cbc5/regions.go#L23).
   - **Confirmar** [opcional]: Pase la marca de confirmación en caso de que quiera confirmar manualmente la eliminación de cada blob de almacenamiento.

1. Ejecute el script guardado como se muestra en el siguiente ejemplo, donde el archivo de script se guardó como **Delete-NsgFlowLogsBlobs.ps1**:
   ```
   .\Delete-NsgFlowLogsBlobs.ps1 -SubscriptionId <subscriptionId> -Location  <location> -Confirm
   ```
    
## <a name="next-steps"></a>Pasos siguientes
- Los clientes pueden automatizar la ejecución del script mediante [Azure Logic Apps](../logic-apps/logic-apps-overview.md) o [Azure Automation](https://azure.microsoft.com/services/automation/).
- Para saber más sobre el registro de NSG, consulte [Registros de Azure Monitor para grupos de seguridad de red (NSG)](../virtual-network/virtual-network-nsg-manage-log.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json).

