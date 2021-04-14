---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: msmbaldwin
ms.service: virtual-machines
ms.topic: include
ms.date: 10/06/2019
ms.author: mbaldwin
ms.custom: include file
ms.openlocfilehash: 3d8cd9891329e86ce47dac6d8d44af529c104b61
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107386873"
---
Azure Disk Encryption se puede habilitar y administrar mediante la [CLI de Azure](/cli/azure) y [Azure PowerShell](/powershell/azure/new-azureps-module-az). Para ello, es preciso instalar las herramientas localmente y conectarse a la suscripción de Azure.

### <a name="azure-cli"></a>Azure CLI

La [CLI de Azure 2.0](/cli/azure) es una herramienta de línea de comandos para administrar recursos de Azure. La CLI está diseñada para flexibilizar los datos de consulta, admitir operaciones de larga duración (como los procesos sin bloqueo) y facilitar la realización de scripts. Para instalarla localmente, siga los pasos que puede encontrar en [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Para [iniciar sesión en su cuenta de Azure con la CLI de Azure](/cli/azure/authenticate-azure-cli), use el comando [az login](/cli/azure/reference-index#az-login).

```azurecli
az login
```

Si quiere seleccionar un inquilino con el que iniciar sesión, use:
    
```azurecli
az login --tenant <tenant>
```

Si tiene varias suscripciones y quiere especificar una en concreto, use [az account list](/cli/azure/account#az-account-list) para obtener la lista de suscripciones y [az account set](/cli/azure/account#az-account-set) para especificar cuál.
     
```azurecli
az account list
az account set --subscription "<subscription name or ID>"
```

Para más información, consulte [Service Fabric y CLI de Azure 2.0](/cli/azure/get-started-with-azure-cli). 

### <a name="azure-powershell"></a>Azure PowerShell
El [módulo az de Azure PowerShell](/powershell/azure/new-azureps-module-az) ofrece un conjunto de cmdlets que usan el modelo de [Azure Resource Manager](../articles/azure-resource-manager/management/overview.md) para administrar los recursos de Azure. Se puede usar en el explorador con [Azure Cloud Shell](../articles/cloud-shell/overview.md), o bien se puede instalar en una máquina local, para lo que hay que seguir las instrucciones de [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). 

Si ya lo tiene instalado localmente, asegúrese de que usa la versión más reciente de la versión del SDK de Azure PowerShell para configurar Azure Disk Encryption. Descargue la versión más reciente de [Azure PowerShell](https://github.com/Azure/azure-powershell/releases).

Para [iniciar sesión en una cuenta de Azure con Azure PowerShell](/powershell/azure/authenticate-azureps), utilice el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

```powershell
Connect-AzAccount
```

Si tiene varias suscripciones y desea especificar una de ellas, use el cmdlet [Get-AzSubscription](/powershell/module/Az.Accounts/Get-AzSubscription) para enumerarlas y, después, el cmdlet [Set-AzContext](/powershell/module/az.accounts/set-azcontext):

```powershell
Set-AzContext -Subscription -Subscription <SubscriptionId>
```

La ejecución del cmdlet [Get-AzContext](/powershell/module/Az.Accounts/Get-AzContext) verificará que se ha seleccionado la suscripción correcta.

Para confirmar que están instalados los cmdlets de Azure Disk Encryption, use el cmdlet [Get-command](/powershell/module/microsoft.powershell.core/get-command):
     
```powershell
Get-command *diskencryption*
```
Para más información, consulte [Introducción a los cmdlets de Azure PowerShell](/powershell/azure/get-started-azureps).