---
title: 'Virtual WAN: Creación de una tabla de rutas de concentrador virtual en NVA: Azure PowerShell'
description: Tabla de rutas de concentrador virtual de Virtual WAN para dirigir el tráfico a un dispositivo virtual de red.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: cherylmc
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 1f792f7a51cd6653885ed19fb8726b692a0523ba
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110703210"
---
# <a name="create-a-virtual-hub-route-table-to-steer-traffic-to-a-network-virtual-appliance"></a>Creación de una tabla de rutas de concentrador virtual para dirigir el tráfico a un dispositivo virtual de red

En este artículo se muestra cómo dirigir el tráfico de un concentrador virtual a un dispositivo virtual de red. 

![Diagrama de Virtual WAN](./media/virtual-wan-route-table-nva/vwanroute.png)

En este artículo, aprenderá a:

* Crear una red de área extensa (WAN)
* Crear un concentrador
* Crear conexiones de un concentrador a una red virtual
* Crear una ruta de concentrador
* Creación de una tabla de rutas
* Aplicar la tabla de rutas

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Compruebe que se cumplen los criterios siguientes:

* Tiene una aplicación virtual de red (NVA). Esta aplicación es un software de terceros de su elección que normalmente se aprovisiona en Azure Marketplace en una red virtual.
* Tiene una dirección IP privada asignada a la interfaz de red de NVA. 
* NVA no se puede implementar en el concentrador virtual. Se debe implementar en una red virtual independiente. En este artículo, se hace referencia a la red virtual de NVA como "red virtual DMZ".
* La red virtual perimetral puede tener una o varias redes virtuales conectados a ella. En este artículo, se hace referencia a la red virtual como "red virtual de radio indirecto'. Estas redes virtuales pueden conectarse a la red virtual perimetral con emparejamiento de redes virtuales.
* Compruebe haber creado 2 redes virtuales. Se usarán como redes virtuales de radio. En este artículo, los espacios de direcciones de red virtual de radio son 10.0.2.0/24 y 10.0.3.0/24. Si necesita más información sobre cómo crear una red virtual, consulte [Creación de una red virtual mediante PowerShell](../virtual-network/quick-create-powershell.md).
* Asegúrese de que no haya ninguna puerta de enlace de red virtual en las redes virtuales.

## <a name="1-sign-in"></a><a name="signin"></a>1. Iniciar sesión

Asegúrese de instalar la versión más reciente de los cmdlets de PowerShell de Resource Manager. Para más información sobre cómo instalar los cmdlets de PowerShell, consulte [Cómo instalar y configurar Azure PowerShell](/powershell/azure/install-az-ps). Esto es importante porque las versiones anteriores de los cmdlets no contienen los valores actuales que necesita para este ejercicio.

1. Abra la consola de PowerShell con privilegios elevados e inicie sesión en su cuenta de Azure. El cmdlet le pide las credenciales de inicio de sesión. Después de iniciar la sesión, se descarga la configuración de la cuenta a fin de que esté disponible para Azure PowerShell.

   ```powershell
   Connect-AzAccount
   ```
2. Obtenga una lista de las suscripciones de Azure.

   ```powershell
   Get-AzSubscription
   ```
3. Especifique la suscripción que desea usar.

   ```powershell
   Select-AzSubscription -SubscriptionName "Name of subscription"
   ```

## <a name="2-create-resources"></a><a name="rg"></a>2. Crear recursos

1. Cree un grupo de recursos.

   ```powershell
   New-AzResourceGroup -Location "West US" -Name "testRG"
   ```
2. Cree una WAN virtual.

   ```powershell
   $virtualWan = New-AzVirtualWan -ResourceGroupName "testRG" -Name "myVirtualWAN" -Location "West US"
   ```
3. Cree un concentrador virtual.

   ```powershell
   New-AzVirtualHub -VirtualWan $virtualWan -ResourceGroupName "testRG" -Name "westushub" -AddressPrefix "10.0.1.0/24" -Location "West US"
   ```

## <a name="3-create-connections"></a><a name="connections"></a>3. Crear conexiones

Cree conexiones de red virtual de concentrador desde la red virtual de radios indirectos y la red virtual perimetral al concentrador virtual.

  ```powershell
  $remoteVirtualNetwork1= Get-AzVirtualNetwork -Name "indirectspoke1" -ResourceGroupName "testRG"
  $remoteVirtualNetwork2= Get-AzVirtualNetwork -Name "indirectspoke2" -ResourceGroupName "testRG"
  $remoteVirtualNetwork3= Get-AzVirtualNetwork -Name "dmzvnet" -ResourceGroupName "testRG"

  New-AzVirtualHubVnetConnection -ResourceGroupName "testRG" -VirtualHubName "westushub" -Name  "testvnetconnection1" -RemoteVirtualNetwork $remoteVirtualNetwork1
  New-AzVirtualHubVnetConnection -ResourceGroupName "testRG" -VirtualHubName "westushub" -Name  "testvnetconnection2" -RemoteVirtualNetwork $remoteVirtualNetwork2
  New-AzVirtualHubVnetConnection -ResourceGroupName "testRG" -VirtualHubName "westushub" -Name  "testvnetconnection3" -RemoteVirtualNetwork $remoteVirtualNetwork3
  ```

## <a name="4-create-a-virtual-hub-route"></a><a name="route"></a>4. Crear una ruta de concentrador virtual

En este artículo, los espacios de direcciones de red virtual de radios indirectos son 10.0.2.0/24 y 10.0.3.0/24, y la dirección IP privada de la interfaz de red de la red virtual perimetral es 10.0.4.5.

```powershell
$route1 = New-AzVirtualHubRoute -AddressPrefix @("10.0.2.0/24", "10.0.3.0/24") -NextHopIpAddress "10.0.4.5"
```

## <a name="5-create-a-virtual-hub-route-table"></a><a name="applyroute"></a>5. Crear una tabla de rutas de concentrador virtual

Cree una tabla de rutas de concentrador virtual y, a continuación, aplíquele la ruta creada.
 
```powershell
$routeTable = New-AzVirtualHubRouteTable -Route @($route1)
```

## <a name="6-commit-the-changes"></a><a name="commit"></a>6. Confirmación de los cambios

Confirme los cambios en el concentrador virtual.

```powershell
Update-AzVirtualHub -ResourceGroupName "testRG" -Name "westushub" -RouteTable $routeTable
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Virtual WAN, consulte la página [Introducción a Virtual WAN](virtual-wan-about.md).
