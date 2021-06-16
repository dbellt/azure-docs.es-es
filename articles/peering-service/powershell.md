---
title: 'Registro de una conexión de Peering Service: Azure PowerShell '
description: En este tutorial aprenderá a registrar una conexión de Peering Service con PowerShell.
services: peering-service
author: derekolo
ms.service: peering-service
ms.topic: tutorial
ms.date: 05/18/2020
ms.author: derekol
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 1c13c19c388937a038e444c1559cd1ef3468ad98
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110688281"
---
# <a name="tutorial-register-a-peering-service-connection-using-azure-powershell"></a>Tutorial: Registro de una conexión de Peering Service mediante Azure PowerShell

En este tutorial, aprenderá a registrar Peering Service mediante Azure PowerShell.

Azure Peering Service es un servicio de red que mejora la conectividad de los clientes a los servicios en la nube de Microsoft, como Microsoft 365, Dynamics 365, servicios de software como servicio (SaaS), Azure o cualquier otro servicio de Microsoft accesible a través de la red pública de Internet. En este artículo, aprenderá a registrar una conexión de Peering Service mediante Azure PowerShell.

Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) ahora.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide en su lugar instalar y usar PowerShell de manera local, este inicio rápido necesita que use la versión 1.0.0. o posterior del módulo de Azure PowerShell. Ejecute `Get-Module -ListAvailable Az` para ver cuál es la versión instalada. Para información sobre la instalación y la actualización, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps).

Por último, si ejecuta PowerShell de manera local, también deberá ejecutar `Connect-AzAccount`. Ese comando crea una conexión con Azure.

Use el módulo de Azure PowerShell para registrar y administrar Peering Service. Se puede registrar o administrar Peering Service desde la línea de comandos de PowerShell o en scripts.


## <a name="prerequisites"></a>Requisitos previos  
Debe tener lo siguiente:

### <a name="azure-account"></a>Cuenta de Azure

Debe tener una cuenta de Microsoft Azure válida y activa. Esta cuenta es necesaria para configurar la conexión de Peering Service. Peering Service es un recurso dentro de las suscripciones de Azure.

### <a name="connectivity-provider"></a>Proveedor de conectividad

Puede trabajar con un proveedor de servicios Internet o con un asociado de intercambio de Internet para obtener Peering Service con el fin de conectar la red a la red de Microsoft.

Asegúrese de que los proveedores de conectividad se asocien con Microsoft.

### <a name="register-a-subscription-with-the-resource-provider-and-feature-flag"></a>Registro de una suscripción con el proveedor de recursos y la marca de característica

Antes de continuar con los pasos para registrar Peering Service, registre la suscripción con el proveedor de recursos y la marca de característica mediante Azure PowerShell. Los comandos de Azure PowerShell se especifican aquí:

```azurepowershell-interactive
Register-AzProviderFeature -FeatureName AllowPeeringService ProviderNamespace Microsoft.Peering 

Register-AzResourceProvider -ProviderNamespace Microsoft.Peering 

```

### <a name="fetch-the-location-and-service-provider"></a>Captura de la ubicación y del proveedor de servicios 

Ejecute los siguientes comandos en Azure PowerShell para adquirir la ubicación y el proveedor de servicios en el que debe habilitarse Peering Service. 

Obtención de las ubicaciones de Peering Service:

```azurepowershell-interactive
# Gets a list of available countries
Get-AzPeeringServiceCountry 
# Gets a list of metro locations serviced by country
Get-AzPeeringServiceLocation -Country "United States"
```

Obtención de los proveedores de Peering Service:
              
```azurepowershell-interactive
Get-AzPeeringServiceProvider
```

### <a name="register-the-peering-service-connection"></a>Registro de la conexión de Peering Service

Registre la conexión de Peering Service mediante el siguiente conjunto de comandos a través de Azure PowerShell. En este ejemplo se registra el servicio Peering Service denominado myPeeringService.

```azurepowershell-interactive
$loc = "Washington"
$provider = "TestPeer1"
$resourceGroup = "MyResourceGroup"
$name = “myPeeringService”
$peeringService = New-AzPeeringService -ResourceGroupName $resourceGroup -Name $name -PeeringLocation $loc -PeeringServiceProvider $provider
```

### <a name="register-the-peering-service-prefix"></a>Registro del prefijo de Peering Service

Registre el prefijo proporcionado por el proveedor de conectividad ejecutando los comandos siguientes a través de Azure PowerShell. En este ejemplo se registra el prefijo denominado myPrefix.

```azurepowershell-interactive
$loc = "Washington"
$provider = "TestPeer1"
$resourceGroup = "MyResourceGroup"
$name = “myPeeringService”
$peeringService = New-AzPeeringService -ResourceGroupName $resourceGroup -Name $name -PeeringLocation $loc -PeeringServiceProvider $provider
$prefixName = "myPrefix"
$prefix = “192.168.1.0/24”
$serviceKey = "6f48cdd6-2c2e-4722-af89-47e27b2513af"
$prefixService = $peeringService | New-AzPeeringServicePrefix -Name $prefixName -Prefix $prefix -ServiceKey $serviceKey
```

### <a name="list-all-the-peering-services-connections"></a>Enumeración de todas las conexiones de Peering Service

Para ver la lista de todas las instancias de Peering Service, ejecute el siguiente comando:

```azurepowershell-interactive
$peeringService = Get-AzPeeringService
```

### <a name="list-all-the-peering-service-prefixes"></a>Enumeración de todos los prefijos de Peering Service

Para ver la lista de todos los prefijos de Peering Service, ejecute el siguiente comando:

```azurepowershell-interactive
 $prefixName = "myPrefix"
```

```azurepowershell-interactive
$prefix = Get-AzPeeringServicePrefix -PeeringServiceName "myPeeringService" -ResourceGroupName "MyResourceGroup" -Name "myPrefix"
```

### <a name="remove-the-peering-service-prefix"></a>Eliminación del prefijo de Peering Service

Para quitar el prefijo de Peering Service, ejecute el siguiente comando:

```azurepowershell-interactive
Remove-AzPeeringServicePrefix -ResourceGroupName  "MyResourceGroup" -Name "myPrefix" -PeeringServiceName "myPeeringService"
```

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre la conexión de Peering Service, consulte [Conexión de Peering Service](connection.md).
- Para más información sobre la telemetría de la conexión de Peering Service, consulte [Telemetría de la conexión de Peering Service](connection-telemetry.md).
- Para registrar una conexión de Peering Service mediante Azure Portal, consulte [Registro de una conexión de Peering Service: Azure Portal](azure-portal.md).
- Para registrar una conexión de Peering Service mediante la CLI de Azure, consulte [Registro de una conexión de Peering Service: CLI de Azure](cli.md).
