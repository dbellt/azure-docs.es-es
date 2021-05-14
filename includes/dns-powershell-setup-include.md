---
title: archivo de inclusión de PowerShell para Azure DNS
description: archivo de inclusión de PowerShell para Azure DNS
services: dns
author: subsarma
ms.service: dns
ms.topic: include file for PowerShell for Azure DNS
ms.date: 04/28//2021
ms.author: subsarma
ms.custom: include file for PowerShell for Azure DNS
ms.openlocfilehash: 3634850a78db5ba03624a4363bbb72397460af38
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108203275"
---
## <a name="set-up-azure-powershell-for-azure-dns"></a>Configuración de Azure PowerShell para Azure DNS

### <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [requires-azurerm](requires-azurerm.md)]

Antes de comenzar con la configuración, compruebe que dispone de los elementos siguientes.

* Suscripción a Azure. Si todavía no la tiene, puede activar sus [ventajas como suscriptor de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/).
* Necesitará instalar la versión más reciente de los cmdlets de PowerShell de Azure Resource Manager. Para obtener más información, consulte [Instalación y configuración de Azure PowerShell](/powershell/azureps-cmdlets-docs).

### <a name="sign-in-to-your-azure-account"></a>Inicio de sesión en la cuenta de Azure.

Abre la consola de PowerShell y conéctate a tu cuenta. Para obtener más información, consulte [Inicio de sesión con Azure PowerShell](/powershell/azure/azurerm/authenticate-azureps).

```azurepowershell-interactive
Connect-AzAccount
```

### <a name="select-the-subscription"></a>Selección de la suscripción
 
Compruebe las suscripciones para la cuenta.

```azurepowershell-interactive
Get-AzSubscription
```

Elección de la suscripción de Azure que se va a usar.

```azurepowershell-interactive
Select-AzSubscription -SubscriptionName "your_subscription_name"
```

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Azure Resource Manager requiere que todos los grupos de recursos especifiquen una ubicación. Esta ubicación se utiliza como ubicación predeterminada para los recursos de ese grupo de recursos. Sin embargo, puesto que todos los recursos DNS son globales y no regionales, la elección de la ubicación del grupo de recursos no incide en DNS de Azure.

Puede omitir este paso si utiliza un grupo de recursos existente.

```azurepowershell-interactive
New-AzResourceGroup -Name MyDNSResourceGroup -location "West US"
```
