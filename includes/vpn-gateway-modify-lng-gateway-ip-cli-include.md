---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 069d7af9cee0bee673c8f0b32659ce362fe4dd70
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96028543"
---
### <a name="to-modify-the-local-network-gateway-gatewayipaddress"></a>Para modificar la puerta de enlace de red local "gatewayIpAddress"

Si el dispositivo VPN al que desea conectarse ha cambiado su dirección IP pública, debe modificar la puerta de enlace de red local para reflejar ese cambio. La dirección IP de puerta de enlace se puede cambiar sin quitar una conexión de puerta de enlace VPN existente (si tiene una). Para modificar la dirección IP de puerta de enlace, reemplace los valores "Site2" y "TestRG1" por los suyos propios mediante el comando [az network local-gateway update](/cli/azure/network/local-gateway).

```azurecli
az network local-gateway update --gateway-ip-address 23.99.222.170 --name Site2 --resource-group TestRG1
```

Compruebe que la dirección IP sea correcta en la salida:

```
"gatewayIpAddress": "23.99.222.170",
```