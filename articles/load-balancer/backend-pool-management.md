---
title: Administración de grupos de back-end
titleSuffix: Azure Load Balancer
description: Comience a aprender cómo configurar y administrar el grupo de back-end de Azure Load Balancer.
services: load-balancer
author: asudbring
ms.service: load-balancer
ms.topic: how-to
ms.date: 01/28/2021
ms.author: allensu
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8a0294e205dd8a22f9847140511cbce634322c4a
ms.sourcegitcommit: 91fdedcb190c0753180be8dc7db4b1d6da9854a1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2021
ms.locfileid: "112285236"
---
# <a name="backend-pool-management"></a>Administración de grupos de back-end
El grupo de back-end es un componente esencial del equilibrador de carga. Define el grupo de recursos que atenderán el tráfico de una regla de equilibrio de carga determinada.

Hay dos formas de configurar un grupo de back-end:
* Tarjeta de interfaz de red (NIC)
* Combinación de dirección IP e identificador de recurso de red virtual

Configure el grupo de back-end por la NIC si usa máquinas virtuales y conjuntos de escalado de máquinas virtuales existentes. Este método crea el vínculo más directo entre el recurso y el grupo de back-end.

Al asignar previamente el grupo de back-end con un intervalo de direcciones IP con el que planea crear posteriormente máquinas virtuales y conjuntos de escalado de máquinas virtuales, configure el grupo de back-end por la combinación de dirección IP e identificador de red virtual.

Aunque puede configurar grupos de back-end basados en NIC e IP para el mismo equilibrador de carga, no puede crear un único grupo de back-end que combine las direcciones guardadas que son el destino de las NIC y las direcciones IP dentro del mismo grupo.

Las secciones de configuración de este artículo se centrarán en los siguientes elementos:

* Azure PowerShell
* Azure CLI
* API DE REST
* Plantillas del Administrador de recursos de Azure

En estas secciones se proporciona información sobre cómo se estructuran los grupos de back-end para cada opción de configuración.

## <a name="configuring-backend-pool-by-nic"></a>Configuración del grupo de back-end mediante NIC
El grupo de back-end se crea como parte de la operación del equilibrador de carga. Se usa la propiedad de configuración de dirección IP de la NIC para agregar miembros al grupo de back-end.

Los siguientes ejemplos se centran en las operaciones de creación y rellenado del grupo de back-end para resaltar este flujo de trabajo y esta relación.

  >[!NOTE]
  >Es importante tener en cuenta que los grupos de back-end configurados mediante la interfaz de red no se pueden actualizar como parte de una operación en el grupo de back-end. Cualquier incorporación o eliminación de recursos de back-end debe realizarse en la interfaz de red del recurso.

### <a name="powershell"></a>PowerShell
Cree un nuevo grupo de back-end:
 
```azurepowershell-interactive
$resourceGroup = "myResourceGroup"
$loadBalancerName = "myLoadBalancer"
$backendPoolName = "myBackendPool"

$backendPool =
New-AzLoadBalancerBackendAddressPool -ResourceGroupName $resourceGroup -LoadBalancerName $loadBalancerName -BackendAddressPoolName $backendPoolName  
```

Cree una nueva interfaz de red y agréguela al grupo de back-end:

```azurepowershell-interactive
$resourceGroup = "myResourceGroup"
$loadBalancerName = "myLoadBalancer"
$backendPoolName = "myBackendPool"
$nicname = "myNic"
$location = "eastus"
$vnetname = <your-vnet-name>

$vnet =
Get-AzVirtualNetwork -Name $vnetname -ResourceGroupName $resourceGroup

$nic =
New-AzNetworkInterface -ResourceGroupName $resourceGroup -Location $location -Name $nicname -LoadBalancerBackendAddressPool $backendPoolName -Subnet $vnet.Subnets[0]
```

Recupere la información del grupo de back-end para que el equilibrador de carga confirme que esta interfaz de red se agrega al grupo de back-end:

```azurepowershell-interactive
$resourceGroup = "myResourceGroup"
$loadBalancerName = "myLoadBalancer"
$backendPoolName = "myBackendPool"

$lb =
Get-AzLoadBalancer -ResourceGroupName $res
Get-AzLoadBalancerBackendAddressPool -ResourceGroupName $resourceGroup -LoadBalancerName $loadBalancerName -BackendAddressPoolName $backendPoolName 
```

Cree una nueva máquina virtual y conecte la interfaz de red para colocarla en el grupo de back-end:

```azurepowershell-interactive
# Create a username and password for the virtual machine
$cred = Get-Credential

# Create a virtual machine configuration
$vmname = "myVM1"
$vmsize = "Standard_DS1_v2"
$pubname = "MicrosoftWindowsServer"
$nicname = "myNic"
$off = "WindowsServer"
$sku = "2019-Datacenter"
$resourceGroup = "myResourceGroup"
$location = "eastus"

$nic =
Get-AzNetworkInterface -Name $nicname -ResourceGroupName $resourceGroup

$vmConfig =
New-AzVMConfig -VMName $vmname -VMSize $vmsize | Set-AzVMOperatingSystem -Windows -ComputerName $vmname -Credential $cred | Set-AzVMSourceImage -PublisherName $pubname -Offer $off -Skus $sku -Version latest | Add-AzVMNetworkInterface -Id $nic.Id

# Create a virtual machine using the configuration
$vm1 = New-AzVM -ResourceGroupName $resourceGroup -Zone 1 -Location $location -VM $vmConfig
```

### <a name="cli"></a>CLI
Cree el grupo de back-end:

```azurecli-interactive
az network lb address-pool create \
--resource-group myResourceGroup \
--lb-name myLB \
--name myBackendPool
```

Cree una nueva interfaz de red y agréguela al grupo de back-end:

```azurecli-interactive
az network nic create \
--resource-group myResourceGroup \
--name myNic \
--vnet-name myVnet \
--subnet mySubnet \
--network-security-group myNetworkSecurityGroup \
--lb-name myLB \
--lb-address-pools myBackEndPool
```

Recupere el grupo de back-end para confirmar que la dirección IP se ha agregado correctamente:

```azurecli-interactive
az network lb address-pool show \
--resource-group myResourceGroup \
--lb-name myLb \
--name myBackendPool
```

Cree una nueva máquina virtual y conecte la interfaz de red para colocarla en el grupo de back-end:

```azurecli-interactive
az vm create \
--resource-group myResourceGroup \
--name myVM \
--nics myNic \
--image UbuntuLTS \
--admin-username azureuser \
--generate-ssh-keys
```

### <a name="resource-manager-template"></a>Plantilla de Resource Manager

Siga la [plantilla de Resource Manager de este inicio rápido](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.network/load-balancer-standard-create/) para implementar un equilibrador de carga y máquinas virtuales, y para agregar las máquinas virtuales al grupo de back-end a través de la interfaz de red.

Siga la [plantilla de Resource Manager de este inicio rápido](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.network/load-balancer-ip-configured-backend-pool) para implementar un equilibrador de carga y máquinas virtuales, y para agregar las máquinas virtuales al grupo de back-end a través de la dirección IP.


## <a name="configure-backend-pool-by-ip-address-and-virtual-network"></a>Configuración del grupo de back-end por dirección IP y red virtual
En escenarios con grupos de back-end rellenados previamente, use dirección IP y red virtual.

Toda la administración del grupo de back-end se realiza directamente en el objeto de grupo de back-end, como se resalta en los ejemplos siguientes.

### <a name="powershell"></a>PowerShell
Cree un nuevo grupo de back-end:

```azurepowershell-interactive
$resourceGroup = "myResourceGroup"
$loadBalancerName = "myLoadBalancer"
$backendPoolName = "myBackendPool"
$vnetName = "myVnet"
$location = "eastus"
$nicName = "myNic"

$backendPool = New-AzLoadBalancerBackendAddressPool -ResourceGroupName $resourceGroup -LoadBalancerName $loadBalancerName -Name $backendPoolName  
```

Actualice el grupo de back-end con una nueva dirección IP de la red virtual existente:
 
```azurepowershell-interactive
$virtualNetwork = 
Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroup 
 
$ip1 = New-AzLoadBalancerBackendAddressConfig -IpAddress "10.0.0.5" -Name "TestVNetRef" -VirtualNetwork $virtualNetwork  
 
$backendPool.LoadBalancerBackendAddresses.Add($ip1) 

Set-AzLoadBalancerBackendAddressPool -InputObject $backendPool
```

Recupere la información del grupo de back-end del equilibrador de carga para confirmar que las direcciones de back-end se agregan al grupo de back-end:

```azurepowershell-interactive
Get-AzLoadBalancerBackendAddressPool -ResourceGroupName $resourceGroup -LoadBalancerName $loadBalancerName -Name $backendPoolName 
```
Cree una interfaz de red y agréguela al grupo de back-end. Establezca la dirección IP en una de las direcciones de back-end:

```azurepowershell-interactive
$nic =
New-AzNetworkInterface -ResourceGroupName $resourceGroup -Location $location -Name $nicName -PrivateIpAddress 10.0.0.4 -Subnet $virtualNetwork.Subnets[0]
```

Cree una máquina virtual y conecte la NIC con una dirección IP en el grupo de back-end:
```azurepowershell-interactive
# Create a username and password for the virtual machine
$cred = Get-Credential

# Create a virtual machine configuration
$vmname = "myVM1"
$vmsize = "Standard_DS1_v2"
$pubname = "MicrosoftWindowsServer"
$nicname = "myNic"
$off = "WindowsServer"
$sku = "2019-Datacenter"
$resourceGroup = "myResourceGroup"
$location = "eastus"

$nic =
Get-AzNetworkInterface -Name $nicname -ResourceGroupName $resourceGroup

$vmConfig =
New-AzVMConfig -VMName $vmname -VMSize $vmsize | Set-AzVMOperatingSystem -Windows -ComputerName $vmname -Credential $cred | Set-AzVMSourceImage -PublisherName $pubname -Offer $off -Skus $sku -Version latest | Add-AzVMNetworkInterface -Id $nic.Id

# Create a virtual machine using the configuration
$vm1 = New-AzVM -ResourceGroupName $resourceGroup -Zone 1 -Location $location -VM $vmConfig
```

### <a name="cli"></a>CLI
Con la CLI puede rellenar el grupo de back-end mediante parámetros de la línea de comandos o mediante un archivo de configuración de JSON.

Cree y rellene el grupo de back-end mediante parámetros de la línea de comandos:

```azurecli-interactive
az network lb address-pool create \
--resource-group myResourceGroup \
--lb-name myLB \
--name myBackendPool \
--vnet {VNET resource ID} \
--backend-address name=addr1 ip-address=10.0.0.4 \
--backend-address name=addr2 ip-address=10.0.0.5
```

Cree y rellene el grupo de back-end mediante el archivo de configuración de JSON:

```azurecli-interactive
az network lb address-pool create \
--resource-group myResourceGroup \
--lb-name myLB \
--name myBackendPool \
--vnet {VNET resource ID} \
--backend-address-config-file @config_file.json
```

Archivo de configuración de JSON:
```JSON
        [
          {
            "name": "address1",
            "virtualNetwork": "/subscriptions/{subscriptionId}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/virtualNetworks/{vnet-name}",
            "ipAddress": "10.0.0.4"
          },
          {
            "name": "address2",
            "virtualNetwork": "/subscriptions/{subscriptionId}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/virtualNetworks/{vnet-name}",
            "ipAddress": "10.0.0.5"
          }
        ]
```

Recupere la información del grupo de back-end del equilibrador de carga para confirmar que las direcciones de back-end se agregan al grupo de back-end:

```azurecli-interactive
az network lb address-pool show \
--resource-group myResourceGroup \
--lb-name MyLb \
--name MyBackendPool
```

Cree una interfaz de red y agréguela al grupo de back-end. Establezca la dirección IP en una de las direcciones de back-end:

```azurecli-interactive
az network nic create \
  --resource-group myResourceGroup \
  --name myNic \
  --vnet-name myVnet \
  --subnet mySubnet \
  --network-security-group myNetworkSecurityGroup \
  --lb-name myLB \
  --private-ip-address 10.0.0.4
```

Cree una máquina virtual y conecte la NIC con una dirección IP en el grupo de back-end:

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --nics myNic \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

### <a name="limitations"></a>Limitaciones
Un grupo de back-end configurado por la dirección IP tiene las siguientes limitaciones:
  * Solo se puede usar para equilibradores de carga estándar
  * Límite de 100 direcciones IP en el grupo de back-end
  * Los recursos de back-end deben estar en la misma red virtual que el equilibrador de carga.
  * Un equilibrador de carga con un grupo de back-end basado en IP no puede funcionar como servicio de Private Link.
  * Los contenedores ACI no admiten esta característica actualmente
  * Los equilibradores de carga o servicios como Application Gateway no se pueden colocar en el grupo de back-end del equilibrador de carga.
  * No se pueden especificar reglas NAT de entrada mediante la dirección IP

>[!Important]
> Cuando un grupo de back-end se configure mediante una dirección IP, se comportará como un equilibrador de carga básico con la salida predeterminada habilitada. Para proteger de forma predeterminada la configuración y las aplicaciones con necesidades de salida exigentes, configure el grupo de back-end mediante NIC.

## <a name="next-steps"></a>Pasos siguientes
En este artículo, ha conocido la administración de grupos de back-end de Azure Load Balancer y ha aprendido a configurar un grupo de back-end mediante una combinación de dirección IP y red virtual.

Más información sobre [Azure Load Balancer](load-balancer-overview.md).

Consulte la [API REST](/rest/api/load-balancer/loadbalancerbackendaddresspools/createorupdate) para la administración de backendpool basada en IP.
