---
title: 'Diagnosticar problemas de enrutamiento en una red de VM: Azure PowerShell'
titleSuffix: Azure Network Watcher
description: En este artículo, aprenderá a diagnosticar un problema de enrutamiento en una red de máquinas virtuales mediante la funcionalidad de Próximo salto de Azure Network Watcher.
services: network-watcher
documentationcenter: network-watcher
author: damendo
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: network-watcher
ms.workload: infrastructure
ms.date: 01/07/2021
ms.author: damendo
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 97871bb33ae1b52bd85b639295b9e232b8b466c4
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110662408"
---
# <a name="diagnose-a-virtual-machine-network-routing-problem---azure-powershell"></a>Diagnosticar problemas de enrutamiento en una red de máquinas virtuales: Azure PowerShell

En este artículo implementará una máquina virtual (VM) y comprobará las comunicaciones hacia una dirección IP y una dirección URL. Además, va a determinar la causa de un error de comunicación y cómo puede resolverlo.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de forma local, para realizar los pasos de este artículo necesita el módulo `Az` de Azure PowerShell. Ejecute `Get-Module -ListAvailable Az` para ver cuál es la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.



## <a name="create-a-vm"></a>Crear una VM

Para poder crear una máquina virtual, debe crear un grupo de recursos que contenga la máquina virtual. Cree un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.Resources/New-azResourceGroup). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*.

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location EastUS
```

Cree la máquina virtual con [New-AzVM](/powershell/module/az.compute/new-azvm). Cuando se realiza este paso, se le solicitará las credenciales. Los valores que especifique se configuran como el nombre de usuario y la contraseña de la máquina virtual.

```azurepowershell-interactive
$vM = New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVm" `
    -Location "East US"
```

La máquina virtual tarda en crearse unos minutos. No continúe con los pasos restantes hasta que la máquina virtual se haya creado y PowerShell devuelva la salida.

## <a name="test-network-communication"></a>Prueba de la comunicación de red

Para probar la comunicación de red con Network Watcher, primero debe habilitar un monitor de red en la región en la que quiere probar la máquina virtual y, a continuación, usar la funcionalidad de próximo salto de Network Watcher para probar la comunicación.

## <a name="enable-network-watcher"></a>Habilitación de Network Watcher

Si ya tiene un monitor de red habilitado en la región Este de EE. UU., use [Get-AzNetworkWatcher](/powershell/module/az.network/get-aznetworkwatcher) para recuperarlo. En el ejemplo siguiente, se recupera un monitor de red existente llamado *NetworkWatcher_eastus* que se encuentra en el grupo de recursos *NetworkWatcherRG*:

```azurepowershell-interactive
$networkWatcher = Get-AzNetworkWatcher `
  -Name NetworkWatcher_eastus `
  -ResourceGroupName NetworkWatcherRG
```

Si aún no tiene un monitor de red habilitado en la región Este de EE. UU., use [New-AzNetworkWatcher](/powershell/module/az.network/new-aznetworkwatcher) para crearlo:

```azurepowershell-interactive
$networkWatcher = New-AzNetworkWatcher `
  -Name "NetworkWatcher_eastus" `
  -ResourceGroupName "NetworkWatcherRG" `
  -Location "East US"
```

### <a name="use-next-hop"></a>Use el próximo salto

Azure crea rutas automáticamente a los destinos predeterminados. Es posible crear rutas personalizadas que reemplacen las rutas predeterminadas. En ocasiones, las rutas personalizadas pueden provocar errores de comunicación. Para probar el enrutamiento desde una máquina virtual, use el comando [Get-AzNetworkWatcherNextHop](/powershell/module/az.network/get-aznetworkwatchernexthop) para determinar el siguiente salto de enrutamiento cuando el tráfico esté destinado a una dirección específica.

Pruebe la comunicación de salida entre la máquina virtual y una de las direcciones IP de www.bing.com:

```azurepowershell-interactive
Get-AzNetworkWatcherNextHop `
  -NetworkWatcher $networkWatcher `
  -TargetVirtualMachineId $VM.Id `
  -SourceIPAddress 192.168.1.4 `
  -DestinationIPAddress 13.107.21.200
```

Después de unos segundos, el resultado le informa que el valor de **NextHopType** es **Internet** y que el valor de **RouteTableId** es **Ruta del sistema**. Este resultado le permite saber que hay una ruta válida para el destino.

Pruebe la comunicación de salida entre la máquina virtual y 172.31.0.100:

```azurepowershell-interactive
Get-AzNetworkWatcherNextHop `
  -NetworkWatcher $networkWatcher `
  -TargetVirtualMachineId $VM.Id `
  -SourceIPAddress 192.168.1.4 `
  -DestinationIPAddress 172.31.0.100
```

El resultado que se devuelve le informa que **Ninguno** es el valor de **NextHopType** y que el valor de **RouteTableId** también es **Ruta del sistema**. Este resultado le permite saber que, aunque no hay una ruta de sistema válida para el destino, no hay ningún próximo salto para enrutar el tráfico al destino.

## <a name="view-details-of-a-route"></a>Vista de los detalles de una ruta

Para analizar el enrutamiento detenidamente, revise las rutas efectivas para la interfaz de red con el comando [Get-AzEffectiveRouteTable](/powershell/module/az.network/get-azeffectiveroutetable):

```azurepowershell-interactive
Get-AzEffectiveRouteTable `
  -NetworkInterfaceName myVm `
  -ResourceGroupName myResourceGroup |
  Format-table
```

Se devuelve una salida que incluye el texto siguiente:

```powershell
Name State  Source  AddressPrefix           NextHopType NextHopIpAddress
---- -----  ------  -------------           ----------- ----------------
     Active Default {192.168.0.0/16}        VnetLocal   {}              
     Active Default {0.0.0.0/0}             Internet    {}              
     Active Default {10.0.0.0/8}            None        {}              
     Active Default {100.64.0.0/10}         None        {}              
     Active Default {172.16.0.0/12}         None        {}              
```

Como puede ver en la salida anterior, la ruta con **AddressPrefix** de **0.0.0.0/0** enruta todo el tráfico no destinado a direcciones dentro de otros prefijos de direcciones de ruta con un próximo salto de **Internet**. Como también puede ver en el resultado, aunque hay una ruta predeterminada hacia el prefijo 172.16.0.0/12 (que incluye la dirección 172.31.0.100), el valor de **nextHopType** es **Ninguno**. Azure crea una ruta predeterminada a 172.16.0.0/12, pero no especifica un tipo de próximo salto hasta que haya un motivo para hacerlo. Por ejemplo, si se agrega el intervalo de direcciones 172.16.0.0/12 al espacio de direcciones de la red virtual, Azure cambia el valor de **nextHopType** a **Red Virtual** para la ruta. A continuación, se muestra una marca de verificación de **Red virtual** como valor de **nextHopType**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, puede usar [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos y todos los recursos que contiene:

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo creó una máquina virtual y diagnosticó el enrutamiento de red a partir de esa máquina virtual. Se informó que Azure crear varias rutas predeterminadas y prueba el enrutamiento a dos destinos diferentes. Obtenga más información sobre [enrutamiento en Azure](../virtual-network/virtual-networks-udr-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) y cómo [crear rutas personalizadas](../virtual-network/manage-route-table.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#create-a-route).

Para las conexiones de máquinas virtuales salientes, también puede determinar la latencia y el tráfico de red permitido o denegado entre la máquina virtual y un punto de conexión mediante la funcionalidad de [resolución de problemas de conexión](network-watcher-connectivity-powershell.md) de Network Watcher. Puede supervisar la comunicación entre una máquina virtual y un punto de conexión, como una dirección IP o una URL, gracias a la funcionalidad de supervisión de conexiones de Network Watcher. Para obtener información sobre cómo puede hacer esto, consulte [Supervisar la comunicación de red ](connection-monitor.md).
