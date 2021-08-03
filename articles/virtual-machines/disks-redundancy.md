---
title: Opciones de redundancia para Azure Managed Disks
description: Obtenga información sobre el almacenamiento con redundancia de zona y el almacenamiento con redundancia local para Azure Managed Disks.
author: roygara
ms.author: rogarana
ms.date: 05/26/2021
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: disks
ms.custom: references_regions, devx-track-azurepowershell
ms.openlocfilehash: 5bb6d0b66e365904e7f0fe4f523f3c19a48d5361
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110673437"
---
# <a name="redundancy-options-for-managed-disks"></a>Opciones de redundancia para Managed Disks

Azure Managed Disks ofrece dos opciones de redundancia de almacenamiento, el almacenamiento con redundancia de zona (ZRS) como versión preliminar y el almacenamiento con redundancia local. ZRS proporciona una mayor disponibilidad para los discos administrados que el almacenamiento con redundancia local (LRS). Sin embargo, la latencia de escritura de los discos LRS es mejor que la de los discos ZRS, ya que los discos LRS escriben datos de forma sincrónica en tres copias en un solo centro de datos.

## <a name="locally-redundant-storage-for-managed-disks"></a>Almacenamiento con redundancia local para Managed Disks

El almacenamiento con redundancia local (LRS) replica los datos tres veces dentro de un único centro de datos en la región seleccionada. LRS protege los datos frente a errores en la estantería de servidores y en la unidad. 

Mediante el uso de discos LRS, hay varias maneras de proteger la aplicación frente a un error de zona completo que pueda producirse debido a desastres naturales o problemas de hardware:
- Use una aplicación como SQL Server AlwaysOn, que puede escribir datos sincrónicamente en dos zonas y conmutar por error automáticamente a otra zona durante un desastre.
- Realice copias de seguridad frecuentes de los discos LRS con instantáneas de ZRS.
- Habilite la recuperación ante desastres entre zonas para los discos LRS mediante [Azure Site Recovery](../site-recovery/azure-to-azure-how-to-enable-zone-to-zone-disaster-recovery.md). Sin embargo, la recuperación ante desastres entre zonas no proporciona un objetivo de punto de recuperación (RPO) de cero.

Si el flujo de trabajo no admite escrituras sincrónicas en el nivel de aplicación entre zonas, o si la aplicación debe cumplir un RPO de cero, los discos ZRS serían ideales.

## <a name="zone-redundant-storage-for-managed-disks-preview"></a>Almacenamiento con redundancia de zona para Managed Disks

El almacenamiento con redundancia de zona (ZRS) replica los discos administrados de Azure de forma sincrónica en tres zonas de disponibilidad de Azure en la región seleccionada. Cada zona de disponibilidad es una ubicación física individual con alimentación, refrigeración y redes independientes. 

Los discos ZRS permiten recuperarse de los errores en las zonas de disponibilidad. Si una zona completa deja de funcionar, se puede conectar un disco ZRS a una máquina virtual de otra zona. También puede usar discos ZRS como un disco compartido y proporcionar así una mejor disponibilidad para aplicaciones en clúster o distribuidas, como SQL FCI, ASCS/SCS de SAP o GFS2. Puede conectar un disco ZRS compartido a las máquinas virtuales principal y secundarias en distintas zonas para aprovechar las ventajas de ZRS y [Availability Zones](../availability-zones/az-overview.md). Si se produce un error en la zona principal, puede conmutar por error rápidamente a la máquina virtual secundaria mediante la [reserva persistente SCSI](disks-shared-enable.md#supported-scsi-pr-commands).

### <a name="billing-implications"></a>Implicaciones de facturación

Para obtener información detallada, consulte la [página de precios de Azure](https://azure.microsoft.com/pricing/details/managed-disks/).

### <a name="comparison-with-other-disk-types"></a>Comparación con otros tipos de disco

A excepción de tener más latencia de escritura, los discos que usan ZRS son idénticos a los discos que usan LRS. Tienen los mismos objetivos de rendimiento. Se recomienda realizar pruebas [comparativas de disco](disks-benchmarks.md) para simular la carga de trabajo de la aplicación y comparar la latencia entre los discos LRS y ZRS. 

### <a name="limitations"></a>Limitaciones

Durante la versión preliminar, ZRS para Managed Disks tiene las siguientes restricciones:

- Solo se admite con unidades de estado sólido (SSD) Premium y SSD Estándar.
- Actualmente, solo está disponible en las regiones Oeste de EE. UU. 2, Oeste de Europa, Norte de Europa y Centro de Francia.
- Los discos ZRS solo se pueden crear con uno de los métodos siguientes:
    -  Plantillas de Azure Resource Manager con la API `2020-12-01` en la versión preliminar pública.
    - La CLI de Azure más reciente.


### <a name="create-zrs-managed-disks"></a>Creación de discos administrados ZRS

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

#### <a name="prerequisites"></a>Requisitos previos

Debe habilitar la característica para su suscripción. Siga los pasos que se indican a continuación para habilitar la característica para su suscripción:

1.  Ejecute el siguiente comando para registrar la característica para su suscripción

    ```azurecli
    az feature register --namespace Microsoft.Compute --name SsdZrsManagedDisks
    ```
 
2.  Antes de probar la característica, compruebe que el estado de registro sea **Registrado** (puede tardar unos minutos) mediante el comando siguiente.

    ```azurecli
    az feature show --namespace Microsoft.Compute --name SsdZrsManagedDisks
    ```

#### <a name="create-a-vm-with-zrs-disks"></a>Creación de una máquina virtual con discos ZRS

```azurecli
rgName=yourRGName
vmName=yourVMName
location=westus2
vmSize=Standard_DS2_v2
image=UbuntuLTS 
osDiskSku=StandardSSD_ZRS
dataDiskSku=Premium_ZRS


az vm create -g $rgName \
-n $vmName \
-l $location \
--image $image \
--size $vmSize \
--generate-ssh-keys \
--data-disk-sizes-gb 128 \
--storage-sku os=$osDiskSku 0=$dataDiskSku
```
#### <a name="create-vms-with-a-shared-zrs-disk-attached-to-the-vms-in-different-zones"></a>Creación de máquinas virtuales con un disco ZRS compartido conectado a las máquinas virtuales de distintas zonas
```azurecli

location=westus2
rgName=yourRGName
vmNamePrefix=yourVMNamePrefix
vmSize=Standard_DS2_v2
image=UbuntuLTS
osDiskSku=StandardSSD_LRS
sharedDiskName=yourSharedDiskName
sharedDataDiskSku=Premium_ZRS


az disk create -g $rgName \
-n $sharedDiskName \
-l $location \
--size-gb 1024 \
--sku $sharedDataDiskSku \
--max-shares 2


sharedDiskId=$(az disk show -g $rgName -n $sharedDiskName --query 'id' -o tsv)

az vm create -g $rgName \
-n $vmNamePrefix"01" \
-l $location \
--image $image \
--size $vmSize \
--generate-ssh-keys \
--zone 1 \
--attach-data-disks $sharedDiskId \
--storage-sku os=$osDiskSku \
--vnet-name $vmNamePrefix"_vnet" \
--subnet $vmNamePrefix"_subnet"

az vm create -g $rgName \
-n $vmNamePrefix"02" \
-l $location \
--image $image \
--size $vmSize \
--generate-ssh-keys \
--zone 2 \
--attach-data-disks $sharedDiskId \
--storage-sku os=$osDiskSku \
--vnet-name $vmNamePrefix"_vnet" \
--subnet $vmNamePrefix"_subnet"

```
#### <a name="create-a-virtual-machine-scale-set-with-zrs-disks"></a>Creación de un conjunto de escalado de máquinas virtuales con discos ZRS
```azurecli
location=westus2
rgName=yourRGName
vmssName=yourVMSSName
vmSize=Standard_DS3_V2
image=UbuntuLTS 
osDiskSku=StandardSSD_ZRS
dataDiskSku=Premium_ZRS

az vmss create -g $rgName \
-n $vmssName \
--encryption-at-host \
--image UbuntuLTS \
--upgrade-policy automatic \
--generate-ssh-keys \
--data-disk-sizes-gb 128 \
--storage-sku os=$osDiskSku 0=$dataDiskSku
```

# <a name="resource-manager-template"></a>[Plantilla de Resource Manager](#tab/azure-resource-manager)

Utilice la API `2020-12-01` con la plantilla de Azure Resource Manager para crear un disco ZRS.

#### <a name="prerequisites"></a>Requisitos previos

Debe habilitar la característica para su suscripción. Siga los pasos que se indican a continuación para habilitar la característica para su suscripción:

1.  Ejecute el siguiente comando para registrar la característica para su suscripción

    ```powershell
     Register-AzProviderFeature -FeatureName "SsdZrsManagedDisks" -ProviderNamespace "Microsoft.Compute" 
    ```

2.  Antes de probar la característica, compruebe que el estado de registro sea **Registrado** (puede tardar unos minutos) mediante el comando siguiente.

    ```powershell
     Get-AzProviderFeature -FeatureName "SsdZrsManagedDisks" -ProviderNamespace "Microsoft.Compute"  
    ```
    
#### <a name="create-a-vm-with-zrs-disks"></a>Creación de una máquina virtual con discos ZRS

```
$vmName = "yourVMName" 
$adminUsername = "yourAdminUsername"
$adminPassword = ConvertTo-SecureString "yourAdminPassword" -AsPlainText -Force
$osDiskType = "StandardSSD_ZRS"
$dataDiskType = "Premium_ZRS"
$region = "eastus2euap"
$resourceGroupName = "yourResourceGroupName"

New-AzResourceGroup -Name $resourceGroupName -Location $region
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
-TemplateUri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/ZRSDisks/CreateVMWithZRSDataDisks.json" `
-resourceName $vmName `
-adminUsername $adminUsername `
-adminPassword $adminPassword `
-region $region `
-osDiskType $osDiskType `
-dataDiskType $dataDiskType
```

#### <a name="create-vms-with-a-shared-zrs-disk-attached-to-the-vms-in-different-zones"></a>Creación de máquinas virtuales con un disco ZRS compartido conectado a las máquinas virtuales de distintas zonas

```
$vmNamePrefix = "yourVMNamePrefix"
$adminUsername = "yourAdminUserName"
$adminPassword = ConvertTo-SecureString "yourAdminPassword" -AsPlainText -Force
$osDiskType = "StandardSSD_LRS"
$sharedDataDiskType = "Premium_ZRS"
$region = "eastus2euap"
$resourceGroupName = "zrstesting1"

New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
-TemplateUri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/ZRSDisks/CreateVMsWithASharedDisk.json" `
-vmNamePrefix $vmNamePrefix `
-adminUsername $adminUsername `
-adminPassword $adminPassword `
-region $region `
-osDiskType $osDiskType `
-dataDiskType $sharedDataDiskType
```

#### <a name="create-a-virtual-machine-scale-set-with-zrs-disks"></a>Creación de un conjunto de escalado de máquinas virtuales con discos ZRS

```
$vmssName="yourVMSSName"
$adminUsername="yourAdminName"
$adminPassword=ConvertTo-SecureString "yourAdminPassword" -AsPlainText -Force
$region="eastus2euap"
$osDiskType="StandardSSD_LRS"
$dataDiskType="Premium_ZRS"

New-AzResourceGroupDeployment -ResourceGroupName zrstesting `
-TemplateUri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/ZRSDisks/CreateVMSSWithZRSDisks.json" `
-vmssName "yourVMSSName" `
-adminUsername "yourAdminName" `
-adminPassword $password `
-region "eastus2euap" `
-osDiskType "StandardSSD_LRS" `
-dataDiskType "Premium_ZRS" `
```
---

## <a name="next-steps"></a>Pasos siguientes

- Vea más [plantillas de Azure Resource Manager para crear una máquina virtual con discos ZRS](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/ZRSDisks).
