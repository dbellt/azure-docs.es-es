---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 02/01/2019
ms.author: cherylmc
ms.custom: include file, devx-track-azurepowershell
ms.openlocfilehash: 9894787ed29226ad6e8d491b498b082cadd0626b
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110690791"
---
 Si está usando Azure Cloud Shell, iniciará sesión automáticamente en su cuenta de Azure después de hacer clic en "Probar". Para iniciar sesión de forma local, abra la consola de PowerShell con privilegios elevados y ejecute el cmdlet para conectarse.

```azurepowershell
Connect-AzAccount
```

Si tiene más de una suscripción, obtenga una lista de las suscripciones de Azure.

```azurepowershell-interactive
Get-AzSubscription
```

Especifique la suscripción que desea usar.

```azurepowershell-interactive
Select-AzSubscription -SubscriptionName "Name of subscription"
```
