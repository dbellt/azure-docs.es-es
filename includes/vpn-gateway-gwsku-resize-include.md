---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 11/04/2019
ms.author: cherylmc
ms.custom: include file, devx-track-azurepowershell
ms.openlocfilehash: 478856b6e62b06259ae37b049774c9e22c840d4a
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110720303"
---
Puede usar el cmdlet de PowerShell `Resize-AzVirtualNetworkGateway` para actualizar o cambiar a una versión anterior una SKU de Generation1 o Generation2 (se puede cambiar el tamaño de todas las SKU de VpnGw excepto las SKU básicas). Si usa la SKU de puerta de enlace Basic, [siga estas instrucciones](../articles/vpn-gateway/vpn-gateway-about-skus-legacy.md#resize) para cambiar el tamaño de la puerta de enlace.

En el siguiente ejemplo de PowerShell se muestra una SKU de puerta de enlace cuyo tamaño se ha cambiado a VpnGw2.

```azurepowershell-interactive
$gw = Get-AzVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg
Resize-AzVirtualNetworkGateway -VirtualNetworkGateway $gw -GatewaySku VpnGw2
```
