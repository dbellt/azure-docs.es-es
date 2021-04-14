---
title: archivo de inclusión de PowerShell para Azure DNS
description: archivo de inclusión de PowerShell para Azure DNS
services: dns
author: subsarma
ms.service: dns
ms.topic: include file for PowerShell for Azure DNS
ms.date: 03/21/2018
ms.author: subsarma
ms.custom: include file for PowerShell for Azure DNS
ms.openlocfilehash: 32c516ccee3a9f4f7604a3e330285703a776b47d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67133833"
---
## <a name="set-up-azure-powershell-for-azure-dns"></a>Configuración de Azure PowerShell para Azure DNS

### <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [requires-azurerm](requires-azurerm.md)]

Antes de comenzar con la configuración, compruebe que dispone de los elementos siguientes.

* Suscripción a Azure. Si todavía no la tiene, puede activar sus [ventajas como suscriptor de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/).
* Necesitará instalar la versión más reciente de los cmdlets de PowerShell de Azure Resource Manager. Para obtener más información, consulte [Instalación y configuración de Azure PowerShell](/powershell/azureps-cmdlets-docs).

Además, para usar las zonas privadas (versión preliminar pública), debe asegurarse de que tiene los siguientes módulos y versiones de PowerShell. 
* AzureRM.Dns: [versión 4.1.0](https://www.powershellgallery.com/packages/AzureRM.Dns/4.1.0) o superior
* AzureRM.Network: [versión 5.4.0](https://www.powershellgallery.com/packages/AzureRM.Network/5.4.0) o superior

```powershell 
Find-Module -Name AzureRM.Dns 
``` 
 
```powershell 
Find-Module -Name AzureRM.Network 
``` 
 
La salida de los comandos anteriores debe mostrar que la versión de AzureRM.Dns es 4.1.0 o una versión posterior y para AzureRM.Network es 5.4.0 o una versión posterior.  

En caso de que el sistema tenga versiones anteriores, puede instalar la última versión de Azure PowerShell, o descargar e instalar los módulos anteriores desde la galería de PowerShell, mediante los enlaces anteriores junto a las versiones de los módulos. Después, puede instalarlos con los comandos siguientes. Ambos módulos son necesarios y totalmente compatibles con versiones anteriores. 

```powershell
Install-Module -Name AzureRM.Dns -Force
```

```powershell
Install-Module -Name AzureRM.Network -Force
```

### <a name="sign-in-to-your-azure-account"></a>Inicio de sesión en la cuenta de Azure.

Abre la consola de PowerShell y conéctate a tu cuenta. Para más información, consulte [Inicio de sesión con AzureRM](/powershell/azure/azurerm/authenticate-azureps).

```powershell
Connect-AzureRmAccount
```

### <a name="select-the-subscription"></a>Selección de la suscripción
 
Compruebe las suscripciones para la cuenta.

```powershell
Get-AzureRmSubscription
```

Elección de la suscripción de Azure que se va a usar.

```powershell
Select-AzureRmSubscription -SubscriptionName "your_subscription_name"
```

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Azure Resource Manager requiere que todos los grupos de recursos especifiquen una ubicación. Esta ubicación se utiliza como ubicación predeterminada para los recursos de ese grupo de recursos. Sin embargo, puesto que todos los recursos DNS son globales y no regionales, la elección de la ubicación del grupo de recursos no incide en DNS de Azure.

Puede omitir este paso si utiliza un grupo de recursos existente.

```powershell
New-AzureRmResourceGroup -Name MyAzureResourceGroup -location "West US"
```

### <a name="register-resource-provider"></a>Registro del proveedor de recursos

El proveedor de recursos Microsoft.Network administra el servicio DNS de Azure. La suscripción a Azure debe estar registrada para usar este proveedor de recursos antes de utilizar Azure DNS. Se trata de una operación única para cada suscripción.

```powershell
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Network
```
