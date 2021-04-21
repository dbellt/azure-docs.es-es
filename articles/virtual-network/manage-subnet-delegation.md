---
title: Adición o eliminación de una delegación de subred en una red virtual de Azure
titlesuffix: Azure Virtual Network
description: Aprenda a agregar o quitar una subred delegada de un servicio en Azure.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/06/2019
ms.author: kumud
ms.openlocfilehash: 401124ed4b2794d891ca224ba3dc1c78edcae8d5
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107783420"
---
# <a name="add-or-remove-a-subnet-delegation"></a>Adición o eliminación de una delegación de subred

La delegación de subred proporciona permisos explícitos al servicio para crear los recursos específicos del servicio en la subred con un identificador único al implementar el servicio. En este artículo se describe cómo agregar o quitar una subred delegada en un servicio de Azure.

## <a name="portal"></a>Portal

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en https://portal.azure.com.

### <a name="create-the-virtual-network"></a>Crear la red virtual

En esta sección, creará una red virtual y la subred que posteriormente delegará en un servicio de Azure.

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso** > **Redes** > **Red virtual**.
1. En **Creación de una red virtual**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | Nombre | Escriba *MyVirtualNetwork*. |
    | Espacio de direcciones | Escriba *10.0.0.0/16*. |
    | Subscription | Seleccione su suscripción.|
    | Resource group | Seleccione **Crear nuevo**, escriba *myResourceGroup* y, después, seleccione **Aceptar**. |
    | Location | Seleccione **EastUS**.|
    | Subred: nombre | Escriba *mySubnet*. |
    | Subred: intervalo de direcciones | Escriba *10.0.0.0/24*. |
    |||
1. Deje el resto de opciones con el valor predeterminado y seleccione **Create** (Crear).

### <a name="permissions"></a>Permisos

Si no ha creado la subred que desea delegar en un servicio de Azure, necesita el siguiente permiso: `Microsoft.Network/virtualNetworks/subnets/write`.

El rol [Colaborador de la red ](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) integrado también contiene los permisos necesarios.

### <a name="delegate-a-subnet-to-an-azure-service"></a>Delegación de una subred a un servicio de Azure

En esta sección, delegará la subred que creó en la sección anterior en un servicio de Azure.

1. En la barra de búsqueda del portal, escriba *myVirtualNetwork*. Cuando aparezca la opción **myVirtualNetwork** en los resultados de la búsqueda, selecciónela.
2. En los resultados de la búsqueda, seleccione *myVirtualNetwork*.
3. Seleccione **Subredes** en **CONFIGURACIÓN** y, después, seleccione **mySubnet**.
4. En la página *mySubnet*, en la lista **Subnet delegation** (Delegación de subred), seleccione uno de los servicios enumerados en **Delegate subnet to a service** (Delegar subred en un servicio) (por ejemplo, **Microsoft. DBforPostgreSQL/serversv2**).  

### <a name="remove-subnet-delegation-from-an-azure-service"></a>Eliminación de la delegación de subred de un servicio de Azure

1. En la barra de búsqueda del portal, escriba *myVirtualNetwork*. Cuando aparezca la opción **myVirtualNetwork** en los resultados de la búsqueda, selecciónela.
2. En los resultados de la búsqueda, seleccione *myVirtualNetwork*.
3. Seleccione **Subredes** en **CONFIGURACIÓN** y, después, seleccione **mySubnet**.
4. En la página *mySubnet*, en la lista **Subnet delegation** (Delegación de subred), seleccione **None** (Ninguno) en los servicios enumerados en **Delegate subnet to a service** (Delegar subred en un servicio). 

## <a name="azure-cli"></a>Azure CLI

Prepare el entorno para la CLI de Azure.

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- En este artículo se necesita la versión 2.0.28 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

### <a name="create-a-resource-group"></a>Crear un grupo de recursos
Cree un grupo de recursos con [az group create](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

En el ejemplo siguiente, se crea un grupo de recursos denominado **myResourceGroup** en la ubicación **eastus**:

```azurecli-interactive

  az group create \
    --name myResourceGroup \
    --location eastus

```

### <a name="create-a-virtual-network"></a>Creación de una red virtual
Cree una red virtual llamada **myVnet** con una subred llamada **mySubnet** en **myResourceGroup** con el comando [az network vnet create](/cli/azure/network/vnet).

```azurecli-interactive
  az network vnet create \
    --resource-group myResourceGroup \
    --location eastus \
    --name myVnet \
    --address-prefix 10.0.0.0/16 \
    --subnet-name mySubnet \
    --subnet-prefix 10.0.0.0/24
```
### <a name="permissions"></a>Permisos

Si no ha creado la subred que desea delegar en un servicio de Azure, necesita el siguiente permiso: `Microsoft.Network/virtualNetworks/subnets/write`.

El rol [Colaborador de la red ](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) integrado también contiene los permisos necesarios.

### <a name="delegate-a-subnet-to-an-azure-service"></a>Delegación de una subred a un servicio de Azure

En esta sección, delegará la subred que creó en la sección anterior en un servicio de Azure. 

Use [az network vnet subnet update](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_update) para actualizar la subred denominada **mySubnet** con una delegación a un servicio de Azure.  En este ejemplo , se usa **Microsoft.DBforPostgreSQL/serversv2** para la delegación de ejemplo:

```azurecli-interactive
  az network vnet subnet update \
  --resource-group myResourceGroup \
  --name mySubnet \
  --vnet-name myVnet \
  --delegations Microsoft.DBforPostgreSQL/serversv2
```

Para comprobar que la delegación se ha aplicado, use [az network vnet subnet show](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_show). Compruebe que el servicio esté delegado en la subred bajo la propiedad **serviceName**:

```azurecli-interactive
  az network vnet subnet show \
  --resource-group myResourceGroup \
  --name mySubnet \
  --vnet-name myVnet \
  --query delegations
```

```json
[
  {
    "actions": [
      "Microsoft.Network/virtualNetworks/subnets/join/action"
    ],
    "etag": "W/\"8a8bf16a-38cf-409f-9434-fe3b5ab9ae54\"",
    "id": "/subscriptions/3bf09329-ca61-4fee-88cb-7e30b9ee305b/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet/delegations/0",
    "name": "0",
    "provisioningState": "Succeeded",
    "resourceGroup": "myResourceGroup",
    "serviceName": "Microsoft.DBforPostgreSQL/serversv2",
    "type": "Microsoft.Network/virtualNetworks/subnets/delegations"
  }
]
```

### <a name="remove-subnet-delegation-from-an-azure-service"></a>Eliminación de la delegación de subred de un servicio de Azure

Use [az network vnet subnet update](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_update) para quitar la delegación de la subred denominada **mySubnet**:

```azurecli-interactive
  az network vnet subnet update \
  --resource-group myResourceGroup \
  --name mySubnet \
  --vnet-name myVnet \
  --remove delegations
```
Para comprobar que se ha quitado la delegación, use [az network vnet subnet show](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_show). Compruebe que el servicio se ha quitado de la subred en la propiedad **serviceName**:

```azurecli-interactive
  az network vnet subnet show \
  --resource-group myResourceGroup \
  --name mySubnet \
  --vnet-name myVnet \
  --query delegations
```
La salida del comando es un corchete nulo:
```json
[]
```

## <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="connect-to-azure"></a>Conexión con Azure

```azurepowershell-interactive
  Connect-AzAccount
```

### <a name="create-a-resource-group"></a>Crear un grupo de recursos
Cree un grupo de recursos con [New-AzResourceGroup](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurepowershell-interactive
  New-AzResourceGroup -Name myResourceGroup -Location eastus
```
### <a name="create-virtual-network"></a>Creación de una red virtual

Cree una red virtual denominada **myVnet** con una subred llamada **mySubnet** con [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) en **myResourceGroup** usando [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). El espacio de direcciones IP de la red virtual es **10.0.0.0/16**. La subred en la red virtual es **10.0.0.0/24**.  

```azurepowershell-interactive
  $subnet = New-AzVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix "10.0.0.0/24"

  New-AzVirtualNetwork -Name myVnet -ResourceGroupName myResourceGroup -Location eastus -AddressPrefix "10.0.0.0/16" -Subnet $subnet
```
### <a name="permissions"></a>Permisos

Si no ha creado la subred que desea delegar en un servicio de Azure, necesita el siguiente permiso: `Microsoft.Network/virtualNetworks/subnets/write`.

El rol [Colaborador de la red ](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) integrado también contiene los permisos necesarios.

### <a name="delegate-a-subnet-to-an-azure-service"></a>Delegación de una subred a un servicio de Azure

En esta sección, delegará la subred que creó en la sección anterior en un servicio de Azure. 

Use [Add-AzDelegation](/powershell/module/az.network/add-azdelegation) para actualizar la subred con el nombre **mySubnet** con una delegación denominada **myDelegation** en un servicio de Azure.  En este ejemplo , se usa **Microsoft.DBforPostgreSQL/serversv2** para la delegación de ejemplo:

```azurepowershell-interactive
  $vnet = Get-AzVirtualNetwork -Name "myVNet" -ResourceGroupName "myResourceGroup"
  $subnet = Get-AzVirtualNetworkSubnetConfig -Name "mySubnet" -VirtualNetwork $vnet
  $subnet = Add-AzDelegation -Name "myDelegation" -ServiceName "Microsoft.DBforPostgreSQL/serversv2" -Subnet $subnet
  Set-AzVirtualNetwork -VirtualNetwork $vnet
```
Use [Get-AzDelegation](/powershell/module/az.network/get-azdelegation) para comprobar la delegación:

```azurepowershell-interactive
  $subnet = Get-AzVirtualNetwork -Name "myVnet" -ResourceGroupName "myResourceGroup" | Get-AzVirtualNetworkSubnetConfig -Name "mySubnet"
  Get-AzDelegation -Name "myDelegation" -Subnet $subnet

  ProvisioningState : Succeeded
  ServiceName       : Microsoft.DBforPostgreSQL/serversv2
  Actions           : {Microsoft.Network/virtualNetworks/subnets/join/action}
  Name              : myDelegation
  Etag              : W/"9cba4b0e-2ceb-444b-b553-454f8da07d8a"
  Id                : /subscriptions/3bf09329-ca61-4fee-88cb-7e30b9ee305b/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet/delegations/myDelegation

```
### <a name="remove-subnet-delegation-from-an-azure-service"></a>Eliminación de la delegación de subred de un servicio de Azure

Use [Remove-AzDelegation](/powershell/module/az.network/remove-azdelegation) para quitar la delegación de la subred denominada **mySubnet**:

```azurepowershell-interactive
  $vnet = Get-AzVirtualNetwork -Name "myVnet" -ResourceGroupName "myResourceGroup"
  $subnet = Get-AzVirtualNetworkSubnetConfig -Name "mySubnet" -VirtualNetwork $vnet
  $subnet = Remove-AzDelegation -Name "myDelegation" -Subnet $subnet
  Set-AzVirtualNetwork -VirtualNetwork $vnet
```
Use [Get-AzDelegation](/powershell/module/az.network/get-azdelegation) para comprobar que se ha quitado la delegación:

```azurepowershell-interactive
  $subnet = Get-AzVirtualNetwork -Name "myVnet" -ResourceGroupName "myResourceGroup" | Get-AzVirtualNetworkSubnetConfig -Name "mySubnet"
  Get-AzDelegation -Name "myDelegation" -Subnet $subnet

  Get-AzDelegation: Sequence contains no matching element

```

## <a name="next-steps"></a>Pasos siguientes
- Aprenda a [administrar subredes en Azure](virtual-network-manage-subnet.md).
