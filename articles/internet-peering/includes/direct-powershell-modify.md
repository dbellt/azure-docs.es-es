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
ms.openlocfilehash: ecc8396204138df05aa779e8bfa09bbfc6cd184d
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110719846"
---
En esta sección se describe cómo realizar las siguientes operaciones de modificación para el emparejamiento directo:

* Adición de conexiones de emparejamiento directo
* Eliminación de conexiones de emparejamiento directo
* Actualización o cambio a una versión anterior del ancho de banda en las conexiones activas
* Adición de sesiones IPv4 o IPv6 en conexiones activas
* Eliminación de sesiones IPv4 o IPv6 en conexiones activas

### <a name="add-direct-peering-connections"></a>Adición de conexiones de emparejamiento directo

En el ejemplo siguiente se describe cómo agregar conexiones a un emparejamiento directo existente.

```powershell

$directPeering = Get-AzPeering -Name "SeattleDirectPeering" -ResourceGroupName "PeeringResourceGroup"

$connection = New-AzPeeringDirectConnection `
    -PeeringDBFacilityId $peeringLocation.PeeringDBFacilityId `
    -SessionPrefixV4 "10.22.31.0/31" `
    -SessionPrefixV6 "fe02::3e:0/127" `
    -MaxPrefixesAdvertisedIPv4 1000 `
    -MaxPrefixesAdvertisedIPv6 100 `
    -BandwidthInMbps 10000

$directPeering.Connections.Add($connection)

$directPeering | Update-AzPeering
```

### <a name="remove-direct-peering-connections"></a>Eliminación de conexiones de emparejamiento directo

La eliminación de una conexión no se admite actualmente en PowerShell. Para más información, póngase en contacto con el [servicio de emparejamiento de Microsoft](mailto:peeringexperience@microsoft.com).

<!--
```powershell
$directPeering.Connections.Remove($directPeering.Connections[0])

$directPeering | Update-AzPeering
```
-->

### <a name="upgrade-or-downgrade-bandwidth-on-active-connections"></a>Actualización o cambio a una versión anterior del ancho de banda en las conexiones activas

En este ejemplo se describe cómo agregar 10 Gbps a una conexión directa existente.

```powershell

$directPeering = Get-AzPeering -Name "SeattleDirectPeering" -ResourceGroupName "PeeringResourceGroup"
$directPeering.Connections[0].BandwidthInMbps  = 20000
$directPeering | Update-AzPeering

```

### <a name="add-ipv4-or-ipv6-sessions-on-active-connections"></a>Adición de sesiones IPv4 o IPv6 en conexiones activas

En este ejemplo se describe cómo agregar una sesión IPv6 en una conexión directa existente solo con una sesión IPv4. 

```powershell

$directPeering = Get-AzPeering -Name "SeattleDirectPeering" -ResourceGroupName "PeeringResourceGroup"
$directPeering.Connections[0].BGPSession.SessionPrefixv6 = "fe01::3e:0/127"
$directPeering | Update-AzPeering

```

### <a name="remove-ipv4-or-ipv6-sessions-on-active-connections"></a>Eliminación de sesiones IPv4 o IPv6 en conexiones activas

Eliminar una sesión IPv4 o IPv6 de una conexión existente no se admite actualmente en PowerShell. Para más información, póngase en contacto con el [servicio de emparejamiento de Microsoft](mailto:peeringexperience@microsoft.com).
