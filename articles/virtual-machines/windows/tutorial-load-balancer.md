---
title: 'Tutorial: Equilibrio de carga de máquinas virtuales Windows en Azure'
description: En este tutorial, aprenderá a usar Azure PowerShell para crear un equilibrador de carga para una aplicación segura y de alta disponibilidad entre tres máquinas virtuales Windows.
author: cynthn
ms.service: virtual-machines
ms.subservice: networking
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 12/03/2018
ms.author: cynthn
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 891c114ea9d2272fd739bafdc16fd3ef2c85acbe
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110668047"
---
# <a name="tutorial-load-balance-windows-virtual-machines-in-azure-to-create-a-highly-available-application-with-azure-powershell"></a>Tutorial: Equilibrio de carga de máquinas virtuales Windows en Azure para crear una aplicación de alta disponibilidad con Azure PowerShell
El equilibrio de carga proporciona un mayor nivel de disponibilidad al distribuir las solicitudes entrantes entre varias máquinas virtuales. En este tutorial, aprenderá sobre los distintos componentes de Azure Load Balancer que distribuyen el tráfico y proporcionan una alta disponibilidad. Aprenderá a:

> [!div class="checklist"]
> * Creación de un equilibrador de carga de Azure
> * Creación del sondeo de estado de un equilibrador de carga
> * Crear reglas de tráfico del equilibrador de carga
> * Usar la extensión de script personalizada para crear un sitio IIS básico
> * Crear máquinas virtuales y conectarlas a un equilibrador de carga
> * Ver un equilibrador de carga en acción
> * Agregar y quitar las máquinas virtuales de un equilibrador de carga

## <a name="azure-load-balancer-overview"></a>Información general sobre Azure Load Balancer
Un equilibrador de carga de Azure es un equilibrador de carga de nivel 4 (TCP, UDP) que proporciona una alta disponibilidad mediante la distribución del tráfico entrante entre máquinas virtuales con un estado correcto. Un sondeo de estado de equilibrador de carga supervisa un puerto determinado en cada máquina virtual y solo distribuye tráfico a una máquina virtual operativa.

Se define una configuración de IP de front-end que contiene una o varias direcciones IP públicas. Esta configuración de IP de front-end permite que el equilibrador de carga y las aplicaciones sean accesibles a través de Internet. 

Las máquinas virtuales se conectarán a un equilibrador de carga mediante su tarjeta de interfaz de red (NIC) virtual. Para distribuir el tráfico a las máquinas virtuales, un grupo de direcciones de back-end contiene las direcciones IP de las tarjetas de interfaz de red (NIC) virtual conectadas al equilibrador de carga.

Para controlar el flujo de tráfico, se definen reglas de equilibrador de carga para determinados puertos y protocolos que se asignan a las máquinas virtuales.

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/powershell](https://shell.azure.com/powershell) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

## <a name="create-azure-load-balancer"></a>Creación del equilibrador de carga de Azure
En esta sección se detalla cómo se puede crear y configurar cada componente del equilibrador de carga. Para poder crear el equilibrador de carga, cree un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroupLoadBalancer* en la ubicación *EastUS*:

```azurepowershell-interactive
New-AzResourceGroup `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Location "EastUS"
```

### <a name="create-a-public-ip-address"></a>Crear una dirección IP pública
Para obtener acceso a la aplicación en Internet, necesita una dirección IP pública para el equilibrador de carga. Cree una dirección IP pública con [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress). En el ejemplo siguiente se crea una dirección IP pública denominada *myPublicIP* en el grupo de recursos *myResourceGroupLoadBalancer*:

```azurepowershell-interactive
$publicIP = New-AzPublicIpAddress `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Location "EastUS" `
  -AllocationMethod "Static" `
  -Name "myPublicIP"
```

### <a name="create-a-load-balancer"></a>Creación de un equilibrador de carga
Cree un grupo de direcciones IP de front-end con [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig). En el ejemplo siguiente se crea un grupo de direcciones IP de front-end llamado *myFrontEndPool* y se asocia la dirección *myPublicIP*: 

```azurepowershell-interactive
$frontendIP = New-AzLoadBalancerFrontendIpConfig `
  -Name "myFrontEndPool" `
  -PublicIpAddress $publicIP
```

Cree un grupo de direcciones de back-end con [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig). En el resto de los pasos las máquinas virtuales se conectan a este grupo back-end. En el siguiente ejemplo se crea un grupo de direcciones de back-end denominado *myBackEndPool*:

```azurepowershell-interactive
$backendPool = New-AzLoadBalancerBackendAddressPoolConfig `
  -Name "myBackEndPool"
```

Ahora, cree el equilibrador de carga con [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer). En el ejemplo siguiente se crea un equilibrador de carga llamado *myLoadBalancer* mediante los grupos de direcciones IP de front-end y back-end creados en los pasos anteriores:

```azurepowershell-interactive
$lb = New-AzLoadBalancer `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Name "myLoadBalancer" `
  -Location "EastUS" `
  -FrontendIpConfiguration $frontendIP `
  -BackendAddressPool $backendPool
```

### <a name="create-a-health-probe"></a>Creación de un sondeo de estado
Para permitir que el equilibrador de carga supervise el estado de la aplicación, utilice un sondeo de estado. El sondeo de estado agrega o quita de forma dinámica las máquinas virtuales de la rotación del equilibrador de carga en base a su respuesta a las comprobaciones de estado. De forma predeterminada, una máquina virtual se quita de la distribución del equilibrador de carga después de dos errores consecutivos en un intervalo de 15 segundos. Cree un sondeo de estado en función de un protocolo o una página de comprobación de mantenimiento específica para la aplicación. 

En el ejemplo siguiente se crea un sondeo de TCP. También se pueden crear sondeos HTTP personalizados para comprobaciones de estado más específicas. Al usar un sondeo HTTP personalizado, debe crear la página de comprobación de estado, por ejemplo *healthcheck.aspx*. El sondeo debe devolver una respuesta **HTTP 200 OK** para que el equilibrador de carga mantenga el host en rotación.

Para crear un sondeo de estado TCP, se usa [Add-AzLoadBalancerProbeConfig](/powershell/module/az.network/add-azloadbalancerprobeconfig). En el ejemplo siguiente se crea un sondeo de estado llamado *myHealthProbe* que supervisa cada máquina virtual en el puerto *TCP* *80*:

```azurepowershell-interactive
Add-AzLoadBalancerProbeConfig `
  -Name "myHealthProbe" `
  -LoadBalancer $lb `
  -Protocol tcp `
  -Port 80 `
  -IntervalInSeconds 15 `
  -ProbeCount 2
```

Para aplicar el sondeo de estado, actualice el equilibrador de carga con [Set-AzLoadBalancer](/powershell/module/az.network/set-azloadbalancer):

```azurepowershell-interactive
Set-AzLoadBalancer -LoadBalancer $lb
```

### <a name="create-a-load-balancer-rule"></a>Creación de una regla de equilibrador de carga
Las reglas de equilibrador de carga se utilizan para definir cómo se distribuye el tráfico a las máquinas virtuales. Se define la configuración de IP front-end para el tráfico entrante y el grupo IP de back-end para recibir el tráfico, junto con el puerto de origen y destino requeridos. Para asegurarse de que solo las máquinas virtuales correctas reciban tráfico, también hay que definir el sondeo de estado que se va usar.

Cree una regla del equilibrador de carga con [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/add-azloadbalancerruleconfig). En el ejemplo siguiente se crea una regla del equilibrador de carga llamada *myLoadBalancerRule* y se equilibra el tráfico en el puerto *TCP* *80*:

```azurepowershell-interactive
$probe = Get-AzLoadBalancerProbeConfig -LoadBalancer $lb -Name "myHealthProbe"

Add-AzLoadBalancerRuleConfig `
  -Name "myLoadBalancerRule" `
  -LoadBalancer $lb `
  -FrontendIpConfiguration $lb.FrontendIpConfigurations[0] `
  -BackendAddressPool $lb.BackendAddressPools[0] `
  -Protocol Tcp `
  -FrontendPort 80 `
  -BackendPort 80 `
  -Probe $probe
```

Actualice el equilibrador de carga con [Set-AzLoadBalancer](/powershell/module/az.network/set-azloadbalancer):

```azurepowershell-interactive
Set-AzLoadBalancer -LoadBalancer $lb
```

## <a name="configure-virtual-network"></a>Configurar la red virtual
Antes de implementar algunas máquinas virtuales y poder probar el equilibrador, cree los recursos de red virtual auxiliares. Para más información sobre las redes virtuales, consulte el tutorial [Administración de Azure Virtual Networks](tutorial-virtual-network.md).

### <a name="create-network-resources"></a>Crear recursos de red
Cree una red virtual con [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). En el ejemplo siguiente se crea una red virtual denominada con una subred llamada *myVnet* con *mySubnet*:

```azurepowershell-interactive
# Create subnet config
$subnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name "mySubnet" `
  -AddressPrefix 192.168.1.0/24

# Create the virtual network
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Location "EastUS" `
  -Name "myVnet" `
  -AddressPrefix 192.168.0.0/16 `
  -Subnet $subnetConfig
```

Las NIC virtuales se crean con [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface). En el ejemplo siguiente se crean tres NIC. (Una NIC virtual para cada máquina virtual que cree para la aplicación en los pasos siguientes). Puede crear NIC virtuales y máquinas virtuales adicionales en cualquier momento y agregarlas al equilibrador de carga:

```azurepowershell-interactive
for ($i=1; $i -le 3; $i++)
{
   New-AzNetworkInterface `
     -ResourceGroupName "myResourceGroupLoadBalancer" `
     -Name myVM$i `
     -Location "EastUS" `
     -Subnet $vnet.Subnets[0] `
     -LoadBalancerBackendAddressPool $lb.BackendAddressPools[0]
}
```


## <a name="create-virtual-machines"></a>Creación de máquinas virtuales
Para mejorar la alta disponibilidad de la aplicación, coloque las máquinas virtuales en un conjunto de disponibilidad.

Cree un conjunto de disponibilidad con [New-AzAvailabilitySet](/powershell/module/az.compute/new-azavailabilityset). En el ejemplo siguiente se crea un conjunto de disponibilidad denominado *myAvailabilitySet*:

```azurepowershell-interactive
$availabilitySet = New-AzAvailabilitySet `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Name "myAvailabilitySet" `
  -Location "EastUS" `
  -Sku aligned `
  -PlatformFaultDomainCount 2 `
  -PlatformUpdateDomainCount 2
```

Establezca un nombre de usuario de administrador y una contraseña para las máquinas virtuales con [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential):

```azurepowershell-interactive
$cred = Get-Credential
```

Ahora puede crear las máquinas virtuales con [New-AzVM](/powershell/module/az.compute/new-azvm). En el siguiente ejemplo se crean tres máquinas virtuales y los componentes de red virtual necesarios, si aún no existen:

```azurepowershell-interactive
for ($i=1; $i -le 3; $i++)
{
    New-AzVm `
        -ResourceGroupName "myResourceGroupLoadBalancer" `
        -Name "myVM$i" `
        -Location "East US" `
        -VirtualNetworkName "myVnet" `
        -SubnetName "mySubnet" `
        -SecurityGroupName "myNetworkSecurityGroup" `
        -OpenPorts 80 `
        -AvailabilitySetName "myAvailabilitySet" `
        -Credential $cred `
        -AsJob
}
```

El parámetro `-AsJob` crea la máquina virtual como tarea en segundo plano, por lo que PowerShell solicita la vuelta. Puede ver detalles de trabajos en segundo plano con el cmdlet `Job`. Se tarda unos minutos en crear y configurar las tres máquinas virtuales.


### <a name="install-iis-with-custom-script-extension"></a>Instalar IIS con la extensión de script personalizado
En un tutorial anterior sobre [Cómo personalizar una máquina virtual de Windows](tutorial-automate-vm-deployment.md), aprendió a automatizar la personalización de máquinas virtuales con la extensión de script personalizado para Windows. Puede usar el mismo enfoque para instalar y configurar IIS en las máquinas virtuales.

Use [Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension) para instalar la extensión de script personalizado. La extensión ejecuta `powershell Add-WindowsFeature Web-Server` para instalar el servidor web de IIS y después actualiza la página *Default.htm* para mostrar el nombre de host de la máquina virtual:

```azurepowershell-interactive
for ($i=1; $i -le 3; $i++)
{
   Set-AzVMExtension `
     -ResourceGroupName "myResourceGroupLoadBalancer" `
     -ExtensionName "IIS" `
     -VMName myVM$i `
     -Publisher Microsoft.Compute `
     -ExtensionType CustomScriptExtension `
     -TypeHandlerVersion 1.8 `
     -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}' `
     -Location EastUS
}
```

## <a name="test-load-balancer"></a>Prueba del equilibrador de carga
Obtenga la dirección IP pública del equilibrador de carga con [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress). En el ejemplo siguiente se obtiene la dirección IP de *myPublicIP* que se ha creado anteriormente:

```azurepowershell-interactive
Get-AzPublicIPAddress `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Name "myPublicIP" | select IpAddress
```

A continuación, puede escribir la dirección IP pública en un explorador web. Se muestra el sitio web, incluido el nombre de host de la máquina virtual a la que el equilibrador de carga distribuye el tráfico como en el ejemplo siguiente:

![Ejecución del sitio web de IIS](./media/tutorial-load-balancer/running-iis-website.png)

Para ver cómo el equilibrador de carga distribuye el tráfico entre las tres máquinas virtuales que ejecutan la aplicación, puede realizar una actualización forzada del explorador web.


## <a name="add-and-remove-vms"></a>Agregar y quitar máquinas virtuales
Puede que tenga que realizar labores de mantenimiento de las máquinas virtuales que ejecutan la aplicación, como la instalación de actualizaciones del sistema operativo. Para gestionar un aumento de tráfico a la aplicación, tiene que agregar más máquinas virtuales. Esta sección le muestra cómo quitar o agregar una máquina virtual desde el equilibrador de carga.

### <a name="remove-a-vm-from-the-load-balancer"></a>Eliminación de una máquina virtual del equilibrador de carga
Obtenga la tarjeta de interfaz de red con [Get-AzNetworkInterface](/powershell/module/az.network/get-aznetworkinterface) y después establezca la propiedad *LoadBalancerBackendAddressPools* de la NIC virtual en *$null*. Por último, actualice la NIC virtual:

```azurepowershell-interactive
$nic = Get-AzNetworkInterface `
    -ResourceGroupName "myResourceGroupLoadBalancer" `
    -Name "myVM2"
$nic.Ipconfigurations[0].LoadBalancerBackendAddressPools=$null
Set-AzNetworkInterface -NetworkInterface $nic
```

Para ver cómo el equilibrador de carga distribuye el tráfico entre las dos máquinas virtuales que quedan ejecutando la aplicación, puede realizar una actualización forzada del explorador web. Ahora puede realizar tareas de mantenimiento en la máquina virtual, como instalar actualizaciones del sistema operativo o realizar un reinicio de máquina virtual.

### <a name="add-a-vm-to-the-load-balancer"></a>Incorporación de una máquina virtual al equilibrador de carga
Después de realizar el mantenimiento de la máquina virtual o si necesita expandir la capacidad, establezca la propiedad *LoadBalancerBackendAddressPools* de la NIC virtual en el elemento *BackendAddressPool* de [Get-AzLoadBalancer](/powershell/module/az.network/get-azloadbalancer):

Obtención del equilibrador de carga:

```azurepowershell-interactive
$lb = Get-AzLoadBalancer `
    -ResourceGroupName myResourceGroupLoadBalancer `
    -Name myLoadBalancer 
$nic.IpConfigurations[0].LoadBalancerBackendAddressPools=$lb.BackendAddressPools[0]
Set-AzNetworkInterface -NetworkInterface $nic
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un equilibrador de carga y conectó máquinas virtuales a él. Ha aprendido a:

> [!div class="checklist"]
> * Creación de un equilibrador de carga de Azure
> * Creación del sondeo de estado de un equilibrador de carga
> * Crear reglas de tráfico del equilibrador de carga
> * Usar la extensión de script personalizada para crear un sitio IIS básico
> * Crear máquinas virtuales y conectarlas a un equilibrador de carga
> * Ver un equilibrador de carga en acción
> * Agregar y quitar las máquinas virtuales de un equilibrador de carga

Avance al siguiente tutorial para aprender a administrar redes de máquinas virtuales.

> [!div class="nextstepaction"]
> [Administración de máquinas y redes virtuales](./tutorial-virtual-network.md)
