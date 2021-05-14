---
title: Uso de PowerShell para implementar máquinas virtuales de acceso puntual de Azure
description: Aprenda a usar Azure PowerShell para implementar máquinas virtuales de acceso puntual de Azure para ahorrar costos.
author: cynthn
ms.service: virtual-machines
ms.subservice: spot
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 03/22/2021
ms.author: cynthn
ms.reviewer: jagaveer
ms.openlocfilehash: bcf3cb5a020bb2efd5a2528c28421deba206ee31
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108125272"
---
# <a name="deploy-azure-spot-virtual-machines-using-azure-powershell"></a>Implementación de máquinas virtuales de acceso puntual de Azure con Azure PowerShell


El uso de [máquinas virtuales de acceso puntual de Azure](../spot-vms.md) permite aprovechar las ventajas de nuestra capacidad no utilizada con un importante ahorro en los costos. Siempre que Azure necesite recuperar la capacidad, su infraestructura expulsará las máquinas virtuales de acceso puntual de Azure. Por lo tanto, las máquinas virtuales de acceso puntual de Azure son excelentes para cargas de trabajo que puedan soportar interrupciones, como los trabajos de procesamiento por lotes, los entornos de desarrollo/pruebas, las grandes cargas de trabajo de proceso, etc.

Los precios de las máquinas virtuales de acceso puntual de Azure son variables, en función de la región y la SKU. Para más información, consulte precios de las máquinas virtuales para [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) y [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/). Para obtener más información sobre cómo se establece el precio máximo, consulte [Máquinas virtuales de acceso puntual de Azure: Precios](../spot-vms.md#pricing).

Puede establecer el precio máximo por hora que esté dispuesto por la máquina virtual. El precio máximo de una máquina virtual de acceso puntual de Azure se puede establecer en dólares estadounidenses (USD), con un máximo de 5 decimales. Por ejemplo, el valor `0.98765` correspondería a un precio máximo de 0,98765 USD por hora. Si establece el precio máximo en `-1`, la máquina virtual no se expulsará por precio. El precio de la máquina virtual será el actual de Spot o el de una máquina virtual estándar, el menor de los dos, siempre que haya capacidad y cuota disponibles.


## <a name="create-the-vm"></a>Creación de la máquina virtual

Cree una máquina virtual de Spot y sírvase de [New-AzVmConfig](/powershell/module/az.compute/new-azvmconfig) para crear la configuración. Incluya `-Priority Spot` y establezca `-MaxPrice` en:
- `-1` para que la máquina virtual no se expulse en función del precio
- o en una cantidad de dólares, de hasta 5 dígitos. Por ejemplo, `-MaxPrice .98765` significa que la máquina virtual se desasignará cuando el precio de una máquina virtual de Spot sea de aproximadamente 0,98765 USD por hora.


En este ejemplo se crea una máquina virtual de Spot que no se desasigna por precio (solo cuando Azure requiera la capacidad). La directiva de expulsión se establece para desasignar la máquina virtual, de modo que se pueda reiniciar en otro momento. Si quiere eliminar la máquina virtual y el disco subyacente cuando se expulsa la máquina virtual, establezca `-EvictionPolicy` en `Delete` en `New-AzVMConfig`.


```azurepowershell-interactive
$resourceGroup = "mySpotRG"
$location = "eastus"
$vmName = "mySpotVM"
$cred = Get-Credential -Message "Enter a username and password for the virtual machine."
New-AzResourceGroup -Name $resourceGroup -Location $location
$subnetConfig = New-AzVirtualNetworkSubnetConfig `
   -Name mySubnet -AddressPrefix 192.168.1.0/24
$vnet = New-AzVirtualNetwork -ResourceGroupName $resourceGroup `
   -Location $location -Name MYvNET -AddressPrefix 192.168.0.0/16 `
   -Subnet $subnetConfig
$pip = New-AzPublicIpAddress -ResourceGroupName $resourceGroup -Location $location `
  -Name "mypublicdns$(Get-Random)" -AllocationMethod Static -IdleTimeoutInMinutes 4
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleRDP  -Protocol Tcp `
  -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
  -DestinationPortRange 3389 -Access Allow
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $resourceGroup -Location $location `
  -Name myNetworkSecurityGroup -SecurityRules $nsgRuleRDP
$nic = New-AzNetworkInterface -Name myNic -ResourceGroupName $resourceGroup -Location $location `
  -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id

# Create a virtual machine configuration and set this to be an Azure Spot Virtual Machine

$vmConfig = New-AzVMConfig -VMName $vmName -VMSize Standard_D1 -Priority "Spot" -MaxPrice -1 -EvictionPolicy Deallocate | `
Set-AzVMOperatingSystem -Windows -ComputerName $vmName -Credential $cred | `
Set-AzVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2016-Datacenter -Version latest | `
Add-AzVMNetworkInterface -Id $nic.Id

New-AzVM -ResourceGroupName $resourceGroup -Location $location -VM $vmConfig
```

Una vez creada la máquina virtual, puede realizar una consulta para ver el precio máximo de todas las máquinas virtuales del grupo de recursos.

```azurepowershell-interactive
Get-AzVM -ResourceGroupName $resourceGroup | `
   Select-Object Name,@{Name="maxPrice"; Expression={$_.BillingProfile.MaxPrice}}
```

## <a name="simulate-an-eviction"></a>Simulación de una expulsión

Puede simular la expulsión de una máquina virtual de acceso puntual de Azure usando REST, PowerShell o la CLI para probar la capacidad de respuesta de una aplicación ante una expulsión repentina.

En la mayoría de los casos, querrá usar la API de REST [Máquinas virtuales: simulación de expulsión](/rest/api/compute/virtualmachines/simulateeviction) para facilitar las pruebas automatizadas de las aplicaciones. En REST un `Response Code: 204` significa que la expulsión simulada se ha realizado correctamente. Puede combinar expulsiones simuladas con el [servicio de eventos programados](scheduled-events.md) para automatizar el modo en que responderá la aplicación cuando se expulse la máquina virtual.

Para ver los eventos programados en acción, consulte [Azure Friday: uso de eventos programados de Azure para preparar el mantenimiento de la VM](https://channel9.msdn.com/Shows/Azure-Friday/Using-Azure-Scheduled-Events-to-Prepare-for-VM-Maintenance).


### <a name="quick-test"></a>Prueba rápida

Para realizar una prueba rápida para mostrar cómo funcionará una expulsión simulada, vamos a consultar el servicio de eventos programados para ver su aspecto cuando se simula una expulsión mediante PowerShell.

El servicio de eventos programados se habilita para su servicio la primera vez que se realiza una solicitud de eventos. 

Conéctese de forma remota a la VM y, a continuación, abra un símbolo del sistema. 

En el símbolo del sistema de la VM, escriba:

```
curl -H Metadata:true http://169.254.169.254/metadata/scheduledevents?api-version=2019-08-01
```

Esta primera respuesta podría tardar hasta 2 minutos. A partir ahora, se deben mostrar los resultados de salida casi de inmediato.

En un equipo que tenga instalado el módulo Az PowerShell (como la máquina local), simule una expulsión mediante [set-AzVM](/powershell/module/az.compute/set-azvm). Reemplace el nombre del grupo de recursos y el nombre de la VM con los suyos. 

```azurepowershell-interactive
Set-AzVM -ResourceGroupName "mySpotRG" -Name "mySpotVM" -SimulateEviction
```

La salida de respuesta tendrá el valor `Status: Succeeded` si la solicitud se realizó correctamente.

Vuelva rápidamente a la conexión remota de la máquina virtual de acceso puntual y vuelva a consultar el punto de conexión de Scheduled Events. Repita el siguiente comando hasta que obtenga un resultado que contenga más información:

```
curl -H Metadata:true http://169.254.169.254/metadata/scheduledevents?api-version=2019-08-01
```

Cuando el servicio de eventos programados obtiene la notificación de expulsión, recibirá una respuesta similar a la siguiente:

```output
{"DocumentIncarnation":1,"Events":[{"EventId":"A123BC45-1234-5678-AB90-ABCDEF123456","EventStatus":"Scheduled","EventType":"Preempt","ResourceType":"VirtualMachine","Resources":["myspotvm"],"NotBefore":"Tue, 16 Mar 2021 00:58:46 GMT","Description":"","EventSource":"Platform"}]}
```

Puede ver que `"EventType":"Preempt"` y el recurso es el recurso de la VM `"Resources":["myspotvm"]`. 

También puede ver cuándo se expulsará la VM; para ello, compruebe el valor `"NotBefore"`. La VM no se expulsará antes del tiempo establecido en `NotBefore`, por lo que la ventana de la aplicación se cerrará correctamente.


## <a name="next-steps"></a>Pasos siguientes

También puede crear una máquina virtual de acceso puntual de Azure mediante la [CLI de Azure](../linux/spot-cli.md), el [portal](../spot-portal.md) o una [plantilla](../linux/spot-template.md).

Consulte la información sobre precios con la [API de precios de venta directa de Azure](/rest/api/cost-management/retail-prices/azure-retail-prices) para conocer los precios de máquinas virtuales de acceso puntual de Azure. Tanto `meterName` como `skuName` contendrán `Spot`.

Si se produce un error, consulte [Códigos de error](../error-codes-spot.md).