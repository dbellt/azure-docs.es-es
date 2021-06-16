---
title: 'Solución de problemas de conexiones y puerta de enlace de red virtual de Azure: Azure PowerShell'
titleSuffix: Azure Network Watcher
description: En esta página se explica cómo usar Azure Network Watcher para solucionar problemas con el cmdlet de PowerShell
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/07/2021
ms.author: damendo
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8db2f73a4b78479ed1a2c3b1ed03ae6a726fa5ab
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110663899"
---
# <a name="troubleshoot-virtual-network-gateway-and-connections-using-azure-network-watcher-powershell"></a>Solución de problemas de las conexiones y la puerta de enlace de Virtual Network mediante PowerShell de Azure Network Watcher

> [!div class="op_single_selector"]
> - [Portal](diagnose-communication-problem-between-networks.md)
> - [PowerShell](network-watcher-troubleshoot-manage-powershell.md)
> - [CLI de Azure](network-watcher-troubleshoot-manage-cli.md)
> - [REST API](network-watcher-troubleshoot-manage-rest.md)

Network Watcher proporciona numerosas funcionalidades con relación a los recursos de red de Azure. Una de estas funcionalidades es la solución de problemas de recursos. Se puede llamar a la solución de problemas de recursos mediante el portal, PowerShell, la CLI o la API de REST. Cuando se llama a Network Watcher, este inspecciona el estado de una puerta de enlace de Virtual Network o de una conexión y devuelve sus conclusiones.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="before-you-begin"></a>Antes de empezar

En este escenario, se da por hecho que ya ha seguido los pasos descritos en [Creación de una instancia de Network Watcher](network-watcher-create.md) para crear un monitor de red.

Para obtener una lista de los tipos de puerta de enlace compatibles, vea el artículo sobre los [tipos de puerta de enlace compatibles](network-watcher-troubleshoot-overview.md#supported-gateway-types).

## <a name="overview"></a>Información general

La solución de problemas de recursos permite solucionar los problemas que surgen con las puertas de enlace y las conexiones de Virtual Network. Cuando se envía una solicitud para solucionar problemas de recursos, se consultan y se inspeccionan los registros. Una vez finalizada la inspección, se devuelven los resultados. Las solicitudes para solucionar problemas de recursos son de larga ejecución, y podrían tardar varios minutos en devolver un resultado. Los registros de solución de problemas se almacenan en un contenedor en una cuenta de almacenamiento especificada.

## <a name="retrieve-network-watcher"></a>Recuperación de Network Watcher

El primer paso consiste en recuperar la instancia de Network Watcher. La variable `$networkWatcher` se pasa al cmdlet `Start-AzNetworkWatcherResourceTroubleshooting` en el paso 4.

```powershell
$networkWatcher = Get-AzNetworkWatcher -Location "WestCentralUS" 
```

## <a name="retrieve-a-virtual-network-gateway-connection"></a>Recuperación de una conexión de puerta de enlace de Virtual Network

En este ejemplo, la solución de problemas de recursos se va a ejecutar en una conexión. También puede pasarla una puerta de enlace de Virtual Network.

```powershell
$connection = Get-AzVirtualNetworkGatewayConnection -Name "2to3" -ResourceGroupName "testrg"
```

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

La solución de problemas de recursos devuelve datos sobre el estado de mantenimiento del recurso; también guarda registros en una cuenta de almacenamiento para su revisión. En este paso, se crea una cuenta de almacenamiento; si ya existe una cuenta de almacenamiento, puede usarla.

```powershell
$sa = New-AzStorageAccount -Name "contosoexamplesa" -SKU "Standard_LRS" -ResourceGroupName "testrg" -Location "WestCentralUS"
Set-AzCurrentStorageAccount -ResourceGroupName $sa.ResourceGroupName -Name $sa.StorageAccountName
$sc = New-AzStorageContainer -Name logs
```

## <a name="run-network-watcher-resource-troubleshooting"></a>Ejecución de la solución de problemas de recursos Network Watcher

Los problemas de recursos se solucionan con el cmdlet `Start-AzNetworkWatcherResourceTroubleshooting`. Pasamos el cmdlet, el objeto de Network Watcher, el identificador de la conexión o la puerta de enlace de Virtual Network, el identificador de la cuenta de almacenamiento y la ruta de acceso para almacenar los resultados.

> [!NOTE]
> El cmdlet `Start-AzNetworkWatcherResourceTroubleshooting` es de larga ejecución y puede tardar unos minutos en completarse.

```powershell
Start-AzNetworkWatcherResourceTroubleshooting -NetworkWatcher $networkWatcher -TargetResourceId $connection.Id -StorageId $sa.Id -StoragePath "$($sa.PrimaryEndpoints.Blob)$($sc.name)"
```

Después de ejecutar el cmdlet, Network Watcher revisa los recursos para comprobar el estado. Devuelve los resultados al shell y almacena los registros de los resultados en la cuenta de almacenamiento especificada.

## <a name="understanding-the-results"></a>Descripción de los resultados

El texto de la acción ofrece instrucciones generales sobre cómo resolver el problema. Si se puede realizar una acción para el problema, se proporciona un vínculo con instrucciones adicionales. Si no hay instrucciones adicionales, la respuesta proporciona la dirección URL para abrir un caso de soporte técnico.  Para más información acerca de las propiedades de la respuesta y lo que incluye, consulte la [introducción a la solución de problemas en Network Watcher](network-watcher-troubleshoot-overview.md)

Para más instrucciones acerca de cómo descargar archivos desde cuentas de Azure Storage, consulte [Introducción a Azure Blob Storage mediante .NET](../storage/blobs/storage-quickstart-blobs-dotnet.md). Otra herramienta que se puede utilizar es el Explorador de Storage. Encontrará más información acerca del Explorador de Storage en el siguiente vínculo: [Explorador de Storage](https://storageexplorer.com/)

## <a name="next-steps"></a>Pasos siguientes

Si se cambió la configuración y la conectividad de VPN se ha detenido, consulte [Administración de grupos de seguridad de red](../virtual-network/manage-network-security-group.md) para realizar un seguimiento de los grupos de seguridad de red y las reglas de seguridad que pueden estar afectados.
