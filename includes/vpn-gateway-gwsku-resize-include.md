---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 11/04/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: a3ae2a876d6a3772d941fec0b8a1ea3f537e60c3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96010862"
---
Puede usar el cmdlet de PowerShell `Resize-AzVirtualNetworkGateway` para actualizar o cambiar a una versión anterior una SKU de Generation1 o Generation2 (se puede cambiar el tamaño de todas las SKU de VpnGw excepto las SKU básicas). Si usa la SKU de puerta de enlace Basic, [siga estas instrucciones](../articles/vpn-gateway/vpn-gateway-about-skus-legacy.md#resize) para cambiar el tamaño de la puerta de enlace.

En el siguiente ejemplo de PowerShell se muestra una SKU de puerta de enlace cuyo tamaño se ha cambiado a VpnGw2.

```azurepowershell-interactive
$gw = Get-AzVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg
Resize-AzVirtualNetworkGateway -VirtualNetworkGateway $gw -GatewaySku VpnGw2
```