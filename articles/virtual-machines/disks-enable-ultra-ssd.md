---
title: Discos Ultra para máquinas virtuales con Azure Managed Disks
description: Más información sobre discos Ultra para máquinas virtuales de Azure
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 04/21/2021
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: 5699cf29075bbc645ae656857b15dff8bab453f1
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107891505"
---
# <a name="using-azure-ultra-disks"></a>Uso de discos Ultra de Azure

En este artículo se explica cómo implementar y usar un disco Ultra; para obtener información conceptual sobre Ultra Disks, vea [¿Qué tipos de discos están disponibles en Azure?](disks-types.md#ultra-disk).

Los discos Ultra de Azure ofrecen un alto rendimiento, IOPS elevadas y un almacenamiento en disco coherente y de baja latencia para máquinas virtuales IaaS de Azure. En esta nueva oferta se proporciona un rendimiento exclusivo que se encuentra en los mismos niveles de disponibilidad que nuestras ofertas de discos existentes. Una ventaja importante de los discos Ultra es la posibilidad de cambiar dinámicamente el rendimiento del disco SSD junto con sus cargas de trabajo sin tener que reiniciar las máquinas virtuales. Además, los discos Ultra son adecuados para cargas de trabajo con grandes cantidades de datos, como SAP HANA, bases de datos de nivel superior y cargas de trabajo que admitan muchas transacciones.

## <a name="ga-scope-and-limitations"></a>Ámbito y limitaciones de la disponibilidad general

[!INCLUDE [managed-disks-ultra-disks-GA-scope-and-limitations](../../includes/managed-disks-ultra-disks-GA-scope-and-limitations.md)]

## <a name="determine-vm-size-and-region-availability"></a>Determinar el tamaño de la máquina virtual y la disponibilidad por región

### <a name="vms-using-availability-zones"></a>Máquinas virtuales que usan zonas de disponibilidad

Para aprovechar los discos Ultra, debe determinar en qué zona de disponibilidad se encuentra. No todas las regiones admiten todos los tamaños de máquina virtual con discos Ultra. Para determinar si la región, la zona y el tamaño de la máquina virtual admiten discos Ultra, ejecute cualquiera de los siguientes comandos y asegúrese de reemplazar primero los valores **region**, **vmSize** y **subscription**:

#### <a name="cli"></a>CLI

```azurecli
subscription="<yourSubID>"
# example value is southeastasia
region="<yourLocation>"
# example value is Standard_E64s_v3
vmSize="<yourVMSize>"

az vm list-skus --resource-type virtualMachines  --location $region --query "[?name=='$vmSize'].locationInfo[0].zoneDetails[0].Name" --subscription $subscription
```

#### <a name="powershell"></a>PowerShell

```powershell
$region = "southeastasia"
$vmSize = "Standard_E64s_v3"
$sku = (Get-AzComputeResourceSku | where {$_.Locations.Contains($region) -and ($_.Name -eq $vmSize) -and $_.LocationInfo[0].ZoneDetails.Count -gt 0})
if($sku){$sku[0].LocationInfo[0].ZoneDetails} Else {Write-host "$vmSize is not supported with Ultra Disk in $region region"}
```

La respuesta será similar al formulario siguiente, donde X es la zona que se utilizará para la implementación en la región elegida. X podría ser 1, 2 o 3.

Conserve el valor de **Zones**, ya que representa la zona de disponibilidad y la necesitará para implementar un disco Ultra.

|ResourceType  |Nombre  |Location  |Zones  |Restricción  |Capacidad  |Value  |
|---------|---------|---------|---------|---------|---------|---------|
|disks     |UltraSSD_LRS         |eastus2         |X         |         |         |         |

> [!NOTE]
> Si no ha habido respuesta del comando, el tamaño de máquina virtual seleccionado no es compatible con los discos Ultra en la región seleccionada.

Ahora que sabe en qué zona se va a realizar la implementación, siga los pasos de implementación de este artículo para implementar una máquina virtual con un disco Ultra conectado o conectar un disco Ultra a una máquina virtual existente.

### <a name="vms-with-no-redundancy-options"></a>Máquinas virtuales sin opciones de redundancia

Los discos Ultra implementados en regiones seleccionadas se deben implementar sin opciones de redundancia por el momento. Sin embargo, no todos los tamaños de disco que admiten discos Ultra pueden estar disponibles en estas regiones. Para determinar qué tamaños de disco admiten discos Ultra, puede usar cualquiera de los siguientes fragmentos de código. Asegúrese de reemplazar primero los valores `vmSize` y `subscription`:

```azurecli
subscription="<yourSubID>"
region="westus"
# example value is Standard_E64s_v3
vmSize="<yourVMSize>"

az vm list-skus --resource-type virtualMachines  --location $region --query "[?name=='$vmSize'].capabilities" --subscription $subscription
```

```azurepowershell
$region = "westus"
$vmSize = "Standard_E64s_v3"
(Get-AzComputeResourceSku | where {$_.Locations.Contains($region) -and ($_.Name -eq $vmSize) })[0].Capabilities
```

La respuesta tendrá una forma similar a la siguiente; `UltraSSDAvailable   True` indica si el tamaño de la máquina virtual admite Ultra Disks en esta región.

```
Name                                         Value
----                                         -----
MaxResourceVolumeMB                          884736
OSVhdSizeMB                                  1047552
vCPUs                                        64
HyperVGenerations                            V1,V2
MemoryGB                                     432
MaxDataDiskCount                             32
LowPriorityCapable                           True
PremiumIO                                    True
VMDeploymentTypes                            IaaS
vCPUsAvailable                               64
ACUs                                         160
vCPUsPerCore                                 2
CombinedTempDiskAndCachedIOPS                128000
CombinedTempDiskAndCachedReadBytesPerSecond  1073741824
CombinedTempDiskAndCachedWriteBytesPerSecond 1073741824
CachedDiskBytes                              1717986918400
UncachedDiskIOPS                             80000
UncachedDiskBytesPerSecond                   1258291200
EphemeralOSDiskSupported                     True
AcceleratedNetworkingEnabled                 True
RdmaEnabled                                  False
MaxNetworkInterfaces                         8
UltraSSDAvailable                            True
```

## <a name="deploy-an-ultra-disk-using-azure-resource-manager"></a>Implementación de un disco Ultra con Azure Resource Manager

En primer lugar, determine el tamaño de la máquina virtual que se va a implementar. Para obtener una lista de los tamaños de máquina virtual admitidos, vea [Ámbito y limitaciones de la disponibilidad general](#ga-scope-and-limitations).

Si quiere crear una máquina virtual con varios discos Ultra, vea el ejemplo [Creación de una máquina virtual con varios discos Ultra](https://aka.ms/ultradiskArmTemplate).

Si piensa usar su propia plantilla, asegúrese de que **apiVersion** para `Microsoft.Compute/virtualMachines` y `Microsoft.Compute/Disks` se establece como `2018-06-01` (o posterior).

Establezca la SKU de disco en **UltraSSD_LRS**, luego establezca la capacidad de disco, IOPS, zona de disponibilidad y rendimiento en MBps para crear un disco ultra.

Cuando se aprovisiona la máquina virtual, puede realizar una partición y dar formato a los discos de datos y configurarlos para las cargas de trabajo.


## <a name="deploy-an-ultra-disk"></a>Implementación de un disco Ultra

# <a name="portal"></a>[Portal](#tab/azure-portal)

En esta sección se describe la implementación de una máquina virtual equipada con un disco Ultra como disco de datos. Se supone que está familiarizado con la implementación de una máquina virtual; si no lo está, consulte [Inicio rápido: crear una máquina virtual de Windows en Azure Portal](./windows/quick-create-portal.md).

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) y navegue hasta Implementar una máquina virtual (VM).
1. Asegúrese de elegir un [tamaño y región compatibles de máquina virtual](#ga-scope-and-limitations).
1. Seleccione **Zona de disponibilidad** en **Opciones de disponibilidad**.
1. Rellene las entradas restantes con las selecciones de su elección.
1. Seleccione **Discos**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-vm-create.png" alt-text="Captura de pantalla del flujo de creación de la VM, hoja Básico." lightbox="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-vm-create.png":::

1. En la hoja Discos, seleccione **Sí** para **Habilitar compatibilidad con Ultra Disks**.
1. Seleccione **Crear y adjuntar un nuevo disco** para conectar un disco Ultra ahora.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-vm-disk-enable.png" alt-text="Captura de pantalla del flujo de creación de la máquina virtual, hoja Disco, ultra está habilitado y Crear y adjuntar un nuevo disco está resaltado." :::

1. En la hoja **Crear un disco**, escriba un nombre y, a continuación, seleccione **Cambiar tamaño**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-create-disk.png" alt-text="Captura de pantalla de la hoja Crear un disco nuevo, con el cambio de tamaño resaltado.":::


1. Cambie la **SKU del disco** a **Disco Ultra**.
1. Cambie los valores de **Tamaño de disco personalizado (GiB)** , **IOPS del disco** y **Rendimiento de disco** por los de su elección.
1. Seleccione **Aceptar** en ambas hojas.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-select-ultra-disk-size.png" alt-text="Captura de pantalla de la hoja Seleccionar un tamaño de disco, con Disco Ultra seleccionado para Tipo de almacenamiento y otros valores resaltados.":::

1. Continúe con la implementación de la máquina virtual, será igual a cómo implementaría cualquier otra máquina virtual.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

En primer lugar, determine el tamaño de la máquina virtual que se va a implementar. Vea la sección [Ámbito y limitaciones de la disponibilidad general](#ga-scope-and-limitations) para obtener una lista de los tamaños de máquina virtual admitidos.

Para conectar un disco Ultra, debe crear una máquina virtual que pueda usar discos Ultra.

Reemplace o establezca las variables **$vmname**, **$rgname**, **$diskname**, **$location**, **$password** y **$user** con sus propios valores. Establezca **$zone** en el valor de la zona de disponibilidad que ha obtenido al [inicio de este artículo](#determine-vm-size-and-region-availability). Después, ejecute el siguiente comando de la CLI para crear una máquina virtual habilitada para ultra:

```azurecli-interactive
az disk create --subscription $subscription -n $diskname -g $rgname --size-gb 1024 --location $location --sku UltraSSD_LRS --disk-iops-read-write 8192 --disk-mbps-read-write 400
az vm create --subscription $subscription -n $vmname -g $rgname --image Win2016Datacenter --ultra-ssd-enabled true --zone $zone --authentication-type password --admin-password $password --admin-username $user --size Standard_D4s_v3 --location $location --attach-data-disks $diskname
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

En primer lugar, determine el tamaño de la máquina virtual que se va a implementar. Vea la sección [Ámbito y limitaciones de la disponibilidad general](#ga-scope-and-limitations) para obtener una lista de los tamaños de máquina virtual admitidos.

Para usar discos Ultra, debe crear una máquina virtual que sea capaz de usar discos Ultra. Reemplace o establezca las variables **$resourcegroup** y **$vmName** con sus propios valores. Establezca **$zone** en el valor de la zona de disponibilidad que ha obtenido al [inicio de este artículo](#determine-vm-size-and-region-availability). Después, ejecute el siguiente comando de [New-AzVm](/powershell/module/az.compute/new-azvm) para crear una máquina virtual ultra habilitada:

```powershell
New-AzVm `
    -ResourceGroupName $resourcegroup `
    -Name $vmName `
    -Location "eastus2" `
    -Image "Win2016Datacenter" `
    -EnableUltraSSD `
    -size "Standard_D4s_v3" `
    -zone $zone
```

### <a name="create-and-attach-the-disk"></a>Creación y conexión del disco

Una vez implementada la máquina virtual, puede crear y conectar un disco Ultra a dicha máquina mediante el siguiente script:

```powershell
# Set parameters and select subscription
$subscription = "<yourSubscriptionID>"
$resourceGroup = "<yourResourceGroup>"
$vmName = "<yourVMName>"
$diskName = "<yourDiskName>"
$lun = 1
Connect-AzAccount -SubscriptionId $subscription

# Create the disk
$diskconfig = New-AzDiskConfig `
-Location 'EastUS2' `
-DiskSizeGB 8 `
-DiskIOPSReadWrite 1000 `
-DiskMBpsReadWrite 100 `
-AccountType UltraSSD_LRS `
-CreateOption Empty `
-zone $zone;

New-AzDisk `
-ResourceGroupName $resourceGroup `
-DiskName $diskName `
-Disk $diskconfig;

# add disk to VM
$vm = Get-AzVM -ResourceGroupName $resourceGroup -Name $vmName
$disk = Get-AzDisk -ResourceGroupName $resourceGroup -Name $diskName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
```

---

## <a name="deploy-an-ultra-disk---512-byte-sector-size"></a>Implementación de un disco Ultra: tamaño de sector de 512 bytes

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) y, a continuación, busque y seleccione **Discos**.
1. Seleccione **+ Nuevo** para crear un nuevo disco.
1. Seleccione una región que admita discos ultra y seleccione una zona de disponibilidad. Rellene el resto de los valores como desee.
1. Seleccione **Cambiar el tamaño**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/create-managed-disk-basics-workflow.png" alt-text="Captura de pantalla de la hoja de creación de disco, la región, la zona de disponibilidad y el cambio de tamaño resaltados.":::

1. En **SKU del disco**, seleccione **Disco Ultra**, rellene los valores para el rendimiento deseado y seleccione **Aceptar**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/select-disk-size-ultra.png" alt-text="Captura de pantalla de creación de un disco ultra.":::

1. En la hoja **Básico**, seleccione la pestaña **Opciones avanzadas**.
1. Seleccione **512** para el **tamaño del sector lógico** y, a continuación, seleccione **Revisar y crear**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/select-different-sector-size-ultra.png" alt-text="Captura de pantalla del selector para cambiar el tamaño del sector lógico del disco ultra a 512.":::

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

En primer lugar, determine el tamaño de la máquina virtual que se va a implementar. Vea la sección [Ámbito y limitaciones de la disponibilidad general](#ga-scope-and-limitations) para obtener una lista de los tamaños de máquina virtual admitidos.

Para conectar un disco Ultra, debe crear una máquina virtual que pueda usar discos Ultra.

Reemplace o establezca las variables **$vmname**, **$rgname**, **$diskname**, **$location**, **$password** y **$user** con sus propios valores. Establezca **$zone** en el valor de la zona de disponibilidad que ha obtenido al [inicio de este artículo](#determine-vm-size-and-region-availability). Después, ejecute el siguiente comando de la CLI para crear una máquina virtual con un disco Ultra con un tamaño de sector de 512 bytes:

```azurecli
#create an ultra disk with 512 sector size
az disk create --subscription $subscription -n $diskname -g $rgname --size-gb 1024 --location $location --sku UltraSSD_LRS --disk-iops-read-write 8192 --disk-mbps-read-write 400 --logical-sector-size 512
az vm create --subscription $subscription -n $vmname -g $rgname --image Win2016Datacenter --ultra-ssd-enabled true --zone $zone --authentication-type password --admin-password $password --admin-username $user --size Standard_D4s_v3 --location $location --attach-data-disks $diskname
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

En primer lugar, determine el tamaño de la máquina virtual que se va a implementar. Vea la sección [Ámbito y limitaciones de la disponibilidad general](#ga-scope-and-limitations) para obtener una lista de los tamaños de máquina virtual admitidos.

Para usar discos Ultra, debe crear una máquina virtual que sea capaz de usar discos Ultra. Reemplace o establezca las variables **$resourcegroup** y **$vmName** con sus propios valores. Establezca **$zone** en el valor de la zona de disponibilidad que ha obtenido al [inicio de este artículo](#determine-vm-size-and-region-availability). Después, ejecute el siguiente comando de [New-AzVm](/powershell/module/az.compute/new-azvm) para crear una máquina virtual ultra habilitada:

```powershell
New-AzVm `
    -ResourceGroupName $resourcegroup `
    -Name $vmName `
    -Location "eastus2" `
    -Image "Win2016Datacenter" `
    -EnableUltraSSD `
    -size "Standard_D4s_v3" `
    -zone $zone
```

Para crear y conectar un disco Ultra con un tamaño de sector de 512 bytes, puede usar el siguiente script:

```powershell
# Set parameters and select subscription
$subscription = "<yourSubscriptionID>"
$resourceGroup = "<yourResourceGroup>"
$vmName = "<yourVMName>"
$diskName = "<yourDiskName>"
$lun = 1
Connect-AzAccount -SubscriptionId $subscription

# Create the disk
$diskconfig = New-AzDiskConfig `
-Location 'EastUS2' `
-DiskSizeGB 8 `
-DiskIOPSReadWrite 1000 `
-DiskMBpsReadWrite 100 `
-LogicalSectorSize 512 `
-AccountType UltraSSD_LRS `
-CreateOption Empty `
-zone $zone;

New-AzDisk `
-ResourceGroupName $resourceGroup `
-DiskName $diskName `
-Disk $diskconfig;

# add disk to VM
$vm = Get-AzVM -ResourceGroupName $resourceGroup -Name $vmName
$disk = Get-AzDisk -ResourceGroupName $resourceGroup -Name $diskName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
```
---
## <a name="attach-an-ultra-disk"></a>Conexión de un disco Ultra

# <a name="portal"></a>[Portal](#tab/azure-portal)

Como alternativa, si la máquina virtual existente se encuentra en una región o zona de disponibilidad que puede usar discos Ultra, puede usar este tipo de discos sin tener que crear una nueva máquina virtual. Para ello, habilite los discos Ultra en la máquina virtual existente y, a continuación, asócielos como discos de datos. Para habilitar la compatibilidad con Ultra Disks, debe detener la máquina virtual. Después de detener la máquina virtual, puede habilitar la compatibilidad y reiniciar la máquina virtual. Una vez habilitada la compatibilidad, puede conectar un disco Ultra:

1. Vaya a la máquina virtual, deténgala y espere a que se desasigne.
1. Una vez desasignada la máquina virtual, seleccione **Discos**.
1. Seleccione **Configuración adicional**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-disk-additional-settings.png" alt-text="Captura de pantalla de la hoja disco, configuración adicional resaltada.":::

1. Seleccione **Sí** para **Habilitar compatibilidad con Ultra Disks**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/enable-ultra-disks-existing-vm.png" alt-text="Captura de pantalla de la habilitación de la compatibilidad con discos Ultra.":::

1. Seleccione **Guardar**.
1. Seleccione **Crear y adjuntar un nuevo disco** y escriba un nombre para el nuevo disco.
1. En **Tipo de almacenamiento**, seleccione **Disco Ultra**.
1. Cambie los valores de **Tamaño (GiB)** , **E/S máxima por segundo** y **Rendimiento máximo** por los de su elección.
1. Cuando vuelva a la hoja del disco, seleccione **Guardar**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-create-ultra-disk-existing-vm.png" alt-text="Captura de pantalla de la hoja del disco con la adición de un nuevo disco ultra.":::

1. Vuelva a iniciar la máquina virtual.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Como alternativa, si la máquina virtual existente se encuentra en una región o zona de disponibilidad que puede usar discos Ultra, puede usar este tipo de discos sin tener que crear una nueva máquina virtual.

### <a name="enable-ultra-disk-compatibility-on-an-existing-vm---cli"></a>Habilitación de la compatibilidad con discos Ultra en una máquina virtual existente: CLI

Si la máquina virtual cumple los requisitos descritos en [Ámbito y limitaciones de la disponibilidad general](#ga-scope-and-limitations) y se encuentra en la [zona adecuada de su cuenta](#determine-vm-size-and-region-availability), puede habilitar la compatibilidad con Ultra Disks en la máquina virtual.

Para habilitar la compatibilidad con Ultra Disks, debe detener la máquina virtual. Después de detener la máquina virtual, puede habilitar la compatibilidad y reiniciar la máquina virtual. Una vez habilitada la compatibilidad, puede conectar un disco Ultra:

```azurecli
az vm deallocate -n $vmName -g $rgName
az vm update -n $vmName -g $rgName --ultra-ssd-enabled true
az vm start -n $vmName -g $rgName
```

### <a name="create-an-ultra-disk---cli"></a>Creación de un disco Ultra: CLI

Ahora que ya tiene una máquina virtual en la que se pueden colocar discos Ultra, puede crear un disco Ultra y conectarlo a dicha máquina virtual.

```azurecli-interactive
location="eastus2"
subscription="xxx"
rgname="ultraRG"
diskname="ssd1"
vmname="ultravm1"
zone=123

#create an ultra disk
az disk create `
--subscription $subscription `
-n $diskname `
-g $rgname `
--size-gb 4 `
--location $location `
--zone $zone `
--sku UltraSSD_LRS `
--disk-iops-read-write 1000 `
--disk-mbps-read-write 50
```

### <a name="attach-the-disk---cli"></a>Conexión del disco: CLI

```azurecli
rgName="<yourResourceGroupName>"
vmName="<yourVMName>"
diskName="<yourDiskName>"
subscriptionId="<yourSubscriptionID>"

az vm disk attach -g $rgName --vm-name $vmName --disk $diskName --subscription $subscriptionId
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Como alternativa, si la máquina virtual existente se encuentra en una región o zona de disponibilidad que puede usar discos Ultra, puede usar este tipo de discos sin tener que crear una nueva máquina virtual.

### <a name="enable-ultra-disk-compatibility-on-an-existing-vm---powershell"></a>Habilitación de la compatibilidad con discos Ultra en una máquina virtual existente: PowerShell

Si la máquina virtual cumple los requisitos descritos en [Ámbito y limitaciones de la disponibilidad general](#ga-scope-and-limitations) y se encuentra en la [zona adecuada de su cuenta](#determine-vm-size-and-region-availability), puede habilitar la compatibilidad con Ultra Disks en la máquina virtual.

Para habilitar la compatibilidad con Ultra Disks, debe detener la máquina virtual. Después de detener la máquina virtual, puede habilitar la compatibilidad y reiniciar la máquina virtual. Una vez habilitada la compatibilidad, puede conectar un disco Ultra:

```azurepowershell
#Stop the VM
Stop-AzVM -Name $vmName -ResourceGroupName $rgName
#Enable ultra disk compatibility
$vm1 = Get-AzVM -name $vmName -ResourceGroupName $rgName
Update-AzVM -ResourceGroupName $rgName -VM $vm1 -UltraSSDEnabled $True
#Start the VM
Start-AzVM -Name $vmName -ResourceGroupName $rgName
```

### <a name="create-and-attach-an-ultra-disk---powershell"></a>Creación y conexión de un disco Ultra: PowerShell

Ahora que ya tiene una máquina virtual que puede usar discos Ultra, puede crear un disco Ultra y conectarlo a dicha máquina virtual:

```powershell
# Set parameters and select subscription
$subscription = "<yourSubscriptionID>"
$resourceGroup = "<yourResourceGroup>"
$vmName = "<yourVMName>"
$diskName = "<yourDiskName>"
$lun = 1
Connect-AzAccount -SubscriptionId $subscription

# Create the disk
$diskconfig = New-AzDiskConfig `
-Location 'EastUS2' `
-DiskSizeGB 8 `
-DiskIOPSReadWrite 1000 `
-DiskMBpsReadWrite 100 `
-AccountType UltraSSD_LRS `
-CreateOption Empty `
-zone $zone;

New-AzDisk `
-ResourceGroupName $resourceGroup `
-DiskName $diskName `
-Disk $diskconfig;

# add disk to VM
$vm = Get-AzVM -ResourceGroupName $resourceGroup -Name $vmName
$disk = Get-AzDisk -ResourceGroupName $resourceGroup -Name $diskName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
```

---
## <a name="adjust-the-performance-of-an-ultra-disk"></a>Ajuste del rendimiento de un disco Ultra

# <a name="portal"></a>[Portal](#tab/azure-portal)

Los discos Ultra ofrecen una capacidad única que permite ajustar su rendimiento. Puede realizar estos ajustes desde Azure Portal, en los propios discos.

1. Navegue hasta la máquina virtual y seleccione **Discos**.
1. Seleccione el disco Ultra en el que desea modificar el rendimiento.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/select-ultra-disk-to-modify.png" alt-text="Captura de pantalla de la hoja de discos de la máquina virtual con el disco Ultra resaltado.":::

1. Seleccione **Tamaño y rendimiento** y, a continuación, realice las modificaciones.
1. Seleccione **Guardar**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/modify-ultra-disk-performance.png" alt-text="Captura de pantalla de la hoja de configuración del disco Ultra, con el tamaño del disco, las IOPS, el rendimiento y la opción para guardar resaltados.":::

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Los discos Ultra ofrecen una capacidad única que permite ajustar su rendimiento. El comando siguiente describe cómo se usa esta característica:

```azurecli-interactive
az disk update `
--subscription $subscription `
--resource-group $rgname `
--name $diskName `
--set diskIopsReadWrite=80000 `
--set diskMbpsReadWrite=800
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

## <a name="adjust-the-performance-of-an-ultra-disk-using-powershell"></a>Ajuste del rendimiento de un disco Ultra mediante PowerShell

Los discos Ultra tienen una capacidad única que permite ajustar su rendimiento. El comando siguiente es un ejemplo que ajusta el rendimiento sin tener que desasociar el disco:

```powershell
$diskupdateconfig = New-AzDiskUpdateConfig -DiskMBpsReadWrite 2000
Update-AzDisk -ResourceGroupName $resourceGroup -DiskName $diskName -DiskUpdate $diskupdateconfig
```
---

## <a name="next-steps"></a>Pasos siguientes

- [Uso de discos Ultra de Azure en Azure Kubernetes Service (versión preliminar)](../aks/use-ultra-disks.md).
- [Migración del disco de registro al disco Ultra](../azure-sql/virtual-machines/windows/storage-migrate-to-ultradisk.md).
