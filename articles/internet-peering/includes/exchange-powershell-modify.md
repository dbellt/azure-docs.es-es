---
title: archivo de inclusión
titleSuffix: Azure
description: archivo de inclusión
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: include
ms.date: 11/27/2019
ms.author: prmitiki
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e1d5b85e722e8ec5624e4d6d9f546cbe401b4ea8
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110722109"
---
Las siguientes operaciones de modificación son compatibles con el emparejamiento de Exchange:
* Adición de conexiones de emparejamiento de Exchange
* Eliminación de conexiones de emparejamiento de Exchange
* Adición de una sesión IPv4 o IPv6 en conexiones activas
* Eliminación de una sesión IPv4 o IPv6 en conexiones activas


### <a name="add-exchange-peering-connections"></a>Adición de conexiones de emparejamiento de Exchange

En el ejemplo siguiente se describe cómo agregar conexiones a un emparejamiento de Exchange existente.

```powershell

$exchangePeering = Get-AzPeering -Name "SeattleExchangePeering" -ResourceGroupName "PeeringResourceGroup"

$connection = New-AzPeeringExchangeConnectionObject `
    -PeeringDBFacilityId $exchangeLocation[1].PeeringDBFacilityId `
    -PeerSessionIPv4Address 198.32.134.22 `
    -PeerSessionIPv6Address  2001:504:12::22 `
    -MaxPrefixesAdvertisedIPv4 2000 `
    -MaxPrefixesAdvertisedIPv6 2000 `

$exchangePeering.Connections.Add($connection)

$exchangePeering | Update-AzPeering

```

### <a name="remove-exchange-peering-connections"></a>Eliminación de conexiones de emparejamiento de Exchange

En el ejemplo siguiente se describe cómo eliminar conexiones de un emparejamiento de Exchange existente.

```powershell

$exchangePeering = Get-AzPeering -Name "SeattleExchangePeering" -ResourceGroupName "PeeringResourceGroup"

```

Vea todas las conexiones y seleccione la que quiera quitar. 

```powershell

$exchangePeering

Name              : SeattleExchangePeering
Sku.Name          : Basic_Exchange_Free
Kind              : Exchange
Connections       : {11}
PeerAsn.Id        : /subscriptions/{subscriptionId}/providers/Microsoft.Peering/peerAsns/{peerAsnName}
PeeringLocation   : Seattle
ProvisioningState : Succeeded
Location          : West US 2
Id                : /subscriptions/{subscriptionId}/resourceGroups/PeeringResourceGroup/providers/Microsoft.Peering/peerings/SeattleExchangePeering
Type              : Microsoft.Peering/peerings
Tags              : {}

```

En el siguiente comando, en lugar de 0, escriba el número de índice de la conexión que quiere quitar.

```powershell

$connection = $exchangePeering.Connections[0]

$exchangePeering.Connections.Remove($connection)

$exchangePeering | Update-AzPeering

```

### <a name="add-an-ipv4-or-ipv6-session-on-active-connections"></a>Adición de una sesión IPv4 o IPv6 en conexiones activas

En el ejemplo siguiente se describe cómo agregar una sesión IPv6 a una conexión de Exchange existente.

```powershell

$exchangePeering = Get-AzPeering -Name "SeattleExchangePeering" -ResourceGroupName "PeeringResourceGroup"

$exchangePeering.Connections[0].BgpSession.PeerSessionIPv6Address = "2001:504:12::34"

$exchangePeering | Update-AzPeering

```

### <a name="remove-an-ipv4-or-ipv6-session-on-active-connections"></a>Eliminación de una sesión IPv4 o IPv6 en conexiones activas

Eliminar una sesión IPv4 o IPv6 de una conexión existente no se admite actualmente en PowerShell. Para más información, póngase en contacto con el [servicio de emparejamiento de Microsoft](mailto:peeringexperience@microsoft.com).
