---
title: 'Inicio rápido: Creación de una zona DNS privada de Azure mediante Azure PowerShell'
description: En este inicio rápido, aprenderá a crear y administrar la primera zona y el primer registro DNS privados con Azure PowerShell.
services: dns
author: rohinkoul
ms.author: rohink
ms.date: 10/20/2020
ms.topic: quickstart
ms.service: dns
ms.custom: devx-track-azurepowershell - mode-api
ms.openlocfilehash: 1a47ec718ef74a8a3df2ba6b87fb8d8955c64b7a
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110701179"
---
# <a name="quickstart-create-an-azure-private-dns-zone-using-azure-powershell"></a>Inicio rápido: Creación de una zona DNS privada de Azure mediante Azure PowerShell

Este artículo le guiará por los pasos necesarios para crear una zona y un registro DNS privados con Azure PowerShell.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Una zona DNS se usa para hospedar los registros DNS de un dominio concreto. Para iniciar el hospedaje de su dominio en DNS de Azure, debe crear una zona DNS para ese nombre de dominio. Cada registro DNS del dominio se crea luego en esta zona DNS. Para publicar una zona DNS privada en la red virtual, especifique la lista de redes virtuales que pueden resolver registros en ella.  Se denominan redes virtuales *vinculadas*. Cuando se habilita el registro automático, Azure DNS también actualiza los registros de zona cuando se crea una máquina virtual, se cambia su dirección IP o se elimina.

## <a name="prerequisites"></a>Prerrequisitos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

Si lo prefiere, puede completar este inicio rápido mediante la [CLI de Azure](private-dns-getstarted-cli.md).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-the-resource-group"></a>Crear el grupo de recursos

En primer lugar, cree un grupo de recursos que contenga la zona DNS: 

```azurepowershell
New-AzResourceGroup -name MyAzureResourceGroup -location "eastus"
```

## <a name="create-a-private-dns-zone"></a>Crear una zona DNS privada

Una zona DNS se crea mediante el cmdlet `New-AzPrivateDnsZone` .

En el ejemplo siguiente se crea una red virtual denominada **myAzureVNet**. Luego, se crea una zona DNS denominada **private.contoso.com** en el grupo de recursos **MyAzureResourceGroup**, se vincula esa zona DNS a la red virtual **MyAzureVnet** y se habilita el registro automático.

```azurepowershell
Install-Module -Name Az.PrivateDns -force

$backendSubnet = New-AzVirtualNetworkSubnetConfig -Name backendSubnet -AddressPrefix "10.2.0.0/24"
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName MyAzureResourceGroup `
  -Location eastus `
  -Name myAzureVNet `
  -AddressPrefix 10.2.0.0/16 `
  -Subnet $backendSubnet

$zone = New-AzPrivateDnsZone -Name private.contoso.com -ResourceGroupName MyAzureResourceGroup

$link = New-AzPrivateDnsVirtualNetworkLink -ZoneName private.contoso.com `
  -ResourceGroupName MyAzureResourceGroup -Name "mylink" `
  -VirtualNetworkId $vnet.id -EnableRegistration
```

Si quiere crear una zona únicamente para la resolución de nombres (sin registro de nombre de host automático), puede omitir el parámetro `-EnableRegistration`.

### <a name="list-dns-private-zones"></a>Listado de zonas privadas de DNS

Si se omite el nombre de la zona de `Get-AzPrivateDnsZone`, puede enumerar todas las zonas en un grupo de recursos. Esta operación devuelve una matriz de objetos de la zona.

```azurepowershell
$zones = Get-AzPrivateDnsZone -ResourceGroupName MyAzureResourceGroup
$zones
```

Si se omite tanto el nombre de zona como el nombre del grupo de recursos de `Get-AzPrivateDnsZone`, puede enumerar todas las zonas de la suscripción de Azure.

```azurepowershell
$zones = Get-AzPrivateDnsZone
$zones
```

## <a name="create-the-test-virtual-machines"></a>Creación de las máquinas virtuales de prueba

Ahora, cree dos máquinas virtuales para poder probar su zona DNS privada:

```azurepowershell
New-AzVm `
    -ResourceGroupName "myAzureResourceGroup" `
    -Name "myVM01" `
    -Location "East US" `
    -subnetname backendSubnet `
    -VirtualNetworkName "myAzureVnet" `
    -addressprefix 10.2.0.0/24 `
    -OpenPorts 3389

New-AzVm `
    -ResourceGroupName "myAzureResourceGroup" `
    -Name "myVM02" `
    -Location "East US" `
    -subnetname backendSubnet `
    -VirtualNetworkName "myAzureVnet" `
    -addressprefix 10.2.0.0/24 `
    -OpenPorts 3389
```

Esta operación tardará algunos minutos en completarse.

## <a name="create-an-additional-dns-record"></a>Creación de un registro de DNS adicional

Los conjuntos de registros se crean mediante el cmdlet `New-AzPrivateDnsRecordSet`. En el ejemplo siguiente se crea un registro con el nombre relativo **db** en la zona DNS **private.contoso.com** del grupo de recursos **MyAzureResourceGroup**. El nombre completo del conjunto de registros es **db.private.contoso.com**. El tipo de registro es "D", con la dirección IP 10.2.0.4 y el valor de TTL es de 3600 segundos.

```azurepowershell
New-AzPrivateDnsRecordSet -Name db -RecordType A -ZoneName private.contoso.com `
   -ResourceGroupName MyAzureResourceGroup -Ttl 3600 `
   -PrivateDnsRecords (New-AzPrivateDnsRecordConfig -IPv4Address "10.2.0.4")
```

### <a name="view-dns-records"></a>Visualización de registros DNS

Para enumerar los registros DNS de su zona, ejecute:

```azurepowershell
Get-AzPrivateDnsRecordSet -ZoneName private.contoso.com -ResourceGroupName MyAzureResourceGroup
```

## <a name="test-the-private-zone"></a>Prueba de la zona privada

Ya puede probar la resolución de nombres de la zona privada **private.contoso.com**.

### <a name="configure-vms-to-allow-inbound-icmp"></a>Configuración de máquinas virtuales para permitir ICMP de entrada

Puede usar el comando ping para probar la resolución de nombres. Por tanto, configure el firewall en ambas máquinas virtuales para permitir paquetes ICMP entrantes.

1. Conéctese a myVM01 y abra una ventana de Windows PowerShell con privilegios de administrador.
2. Ejecute el siguiente comando:

   ```powershell
   New-NetFirewallRule –DisplayName "Allow ICMPv4-In" –Protocol ICMPv4
   ```

Repita la operación con myVM02.

### <a name="ping-the-vms-by-name"></a>Realización de ping en las máquinas virtuales por nombre

1. En el símbolo del sistema de Windows PowerShell de myVM02, haga ping a myVM01 con el nombre de host registrado automáticamente:

   ```
   ping myVM01.private.contoso.com
   ```

   Debería ver una salida similar a esta:

   ```
   PS C:\> ping myvm01.private.contoso.com

   Pinging myvm01.private.contoso.com [10.2.0.4] with 32 bytes of data:
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time=1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128

   Ping statistics for 10.2.0.4:
       Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
   Approximate round trip times in milli-seconds:
       Minimum = 0ms, Maximum = 1ms, Average = 0ms
   PS C:\>
   ```

2. Ahora haga ping en el nombre de la **base de datos** que creó anteriormente:

   ```
   ping db.private.contoso.com
   ```

   Debería ver una salida similar a esta:

   ```
   PS C:\> ping db.private.contoso.com

   Pinging db.private.contoso.com [10.2.0.4] with 32 bytes of data:
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128

   Ping statistics for 10.2.0.4:
       Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
   Approximate round trip times in milliseconds:
       Minimum = 0ms, Maximum = 0ms, Average = 0ms
   PS C:\>
   ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando no lo necesite, elimine el grupo de recursos **MyAzureResourceGroup** para eliminar los recursos que ha creado en este artículo.

```azurepowershell
Remove-AzResourceGroup -Name MyAzureResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Escenarios de Azure DNS Private Zones](private-dns-scenarios.md)
