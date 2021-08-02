---
title: 'Cambio del rendimiento de discos administrados de Azure: CLI/PowerShell'
description: Aprenda a cambiar los niveles de rendimiento de los discos administrados existentes mediante el módulo de Azure PowerShell o la CLI de Azure.
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 05/13/2021
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions, devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 32f589be39740d62ef77967867522688d29a3618
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110673505"
---
# <a name="change-your-performance-tier-using-the-azure-powershell-module-or-the-azure-cli"></a>El nivel de rendimiento se puede cambiar mediante el módulo Azure PowerShell o la CLI de Azure

[!INCLUDE [virtual-machines-disks-performance-tiers-intro](../../includes/virtual-machines-disks-performance-tiers-intro.md)]

## <a name="restrictions"></a>Restricciones

[!INCLUDE [virtual-machines-disks-performance-tiers-restrictions](../../includes/virtual-machines-disks-performance-tiers-restrictions.md)]

## <a name="create-an-empty-data-disk-with-a-tier-higher-than-the-baseline-tier"></a>Creación de un disco de datos vacío con un nivel superior al nivel de línea de base

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
subscriptionId=<yourSubscriptionIDHere>
resourceGroupName=<yourResourceGroupNameHere>
diskName=<yourDiskNameHere>
diskSize=<yourDiskSizeHere>
performanceTier=<yourDesiredPerformanceTier>
region=westcentralus

az login

az account set --subscription $subscriptionId

az disk create -n $diskName -g $resourceGroupName -l $region --sku Premium_LRS --size-gb $diskSize --tier $performanceTier
```
## <a name="create-an-os-disk-with-a-tier-higher-than-the-baseline-tier-from-an-azure-marketplace-image"></a>Creación de un disco del sistema operativo con un nivel superior al nivel de línea de base a partir de una imagen de Azure Marketplace

```azurecli
resourceGroupName=<yourResourceGroupNameHere>
diskName=<yourDiskNameHere>
performanceTier=<yourDesiredPerformanceTier>
region=westcentralus
image=Canonical:UbuntuServer:18.04-LTS:18.04.202002180

az disk create -n $diskName -g $resourceGroupName -l $region --image-reference $image --sku Premium_LRS --tier $performanceTier
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$subscriptionId='yourSubscriptionID'
$resourceGroupName='yourResourceGroupName'
$diskName='yourDiskName'
$diskSizeInGiB=4
$performanceTier='P50'
$sku='Premium_LRS'
$region='westcentralus'

Connect-AzAccount

Set-AzContext -Subscription $subscriptionId

$diskConfig = New-AzDiskConfig -SkuName $sku -Location $region -CreateOption Empty -DiskSizeGB $diskSizeInGiB -Tier $performanceTier
New-AzDisk -DiskName $diskName -Disk $diskConfig -ResourceGroupName $resourceGroupName
```
---

## <a name="update-the-tier-of-a-disk"></a>Actualización del nivel de un disco

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
resourceGroupName=<yourResourceGroupNameHere>
diskName=<yourDiskNameHere>
performanceTier=<yourDesiredPerformanceTier>

az disk update -n $diskName -g $resourceGroupName --set tier=$performanceTier
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$resourceGroupName='yourResourceGroupName'
$diskName='yourDiskName'
$performanceTier='P1'

$diskUpdateConfig = New-AzDiskUpdateConfig -Tier $performanceTier

Update-AzDisk -ResourceGroupName $resourceGroupName -DiskName $diskName -DiskUpdate $diskUpdateConfig
```
---

## <a name="show-the-tier-of-a-disk"></a>Consulta del nivel de un disco

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az disk show -n $diskName -g $resourceGroupName --query [tier] -o tsv
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$disk = Get-AzDisk -ResourceGroupName $resourceGroupName -DiskName $diskName

$disk.Tier
```
---

## <a name="change-the-performance-tier-of-a-disk-without-downtime-preview"></a>Cambio del nivel de rendimiento de un disco sin que haya tiempo de inactividad (versión preliminar)

También puede cambiar el nivel de rendimiento sin tiempo de inactividad, por lo que no es preciso desasignar la máquina virtual ni desasociar el disco para cambiar el nivel.

### <a name="prerequisites"></a>Requisitos previos

El disco debe cumplir los requisitos establecidos en la sección [Cambio del nivel de rendimiento sin tiempo de inactividad (versión preliminar)](#change-performance-tier-without-downtime-preview); si no lo hace, el cambio del nivel de rendimiento incurrirá en tiempo de inactividad.

Debe habilitar la característica en su suscripción para poder cambiar el nivel de rendimiento de un disco sin tiempo de inactividad. Siga los pasos que se indican a continuación para habilitar la característica para su suscripción:

1.  Ejecute el siguiente comando para registrar la característica para su suscripción

    ```azurecli
    az feature register --namespace Microsoft.Compute --name LiveTierChange
    ```
 
1.  Compruebe que el estado de registro sea **Registrado** (puede tardar unos minutos) mediante el comando siguiente antes de probar la característica.

    ```azurecli
    az feature show --namespace Microsoft.Compute --name LiveTierChange
    ```

### <a name="update-the-performance-tier-of-a-disk-without-downtime-via-azure-cli"></a>Actualice el nivel de rendimiento de un disco sin tiempo de inactividad mediante la CLI de Azure

El siguiente script actualizará el nivel de un disco superior al nivel de línea de base. Reemplace `<yourResourceGroupNameHere>`, `<yourDiskNameHere>` y `<yourDesiredPerformanceTier>` y, luego, ejecute el script:

```azurecli
resourceGroupName=<yourResourceGroupNameHere>
diskName=<yourDiskNameHere>
performanceTier=<yourDesiredPerformanceTier>
 
az disk update -n $diskName -g $resourceGroupName --set tier=$performanceTier
```

### <a name="update-the-performance-tier-of-a-disk-without-downtime-via-arm-template"></a>Actualización del nivel de rendimiento de un disco sin tiempo de inactividad mediante una plantilla de ARM

El siguiente script actualizará el nivel de un disco a un nivel superior al de la linea base mediante la plantilla de ejemplo [CreateUpdateDataDiskWithTier.jsen](https://github.com/Azure/azure-managed-disks-performance-tiers/blob/main/CreateUpdateDataDiskWithTier.json). Reemplace `<yourSubScriptionID>`, `<yourResourceGroupName>`, `<yourDiskName>`, `<yourDiskSize>` y `<yourDesiredPerformanceTier>`, y, después, ejecute el script:

 ```cli
subscriptionId=<yourSubscriptionID>
resourceGroupName=<yourResourceGroupName>
diskName=<yourDiskName>
diskSize=<yourDiskSize>
performanceTier=<yourDesiredPerformanceTier>
region=EastUS2EUAP

 az login

 az account set --subscription $subscriptionId

 az group deployment create -g $resourceGroupName \
--template-uri "https://raw.githubusercontent.com/Azure/azure-managed-disks-performance-tiers/main/CreateUpdateDataDiskWithTier.json" \
--parameters "region=$region" "diskName=$diskName" "performanceTier=$performanceTier" "dataDiskSizeInGb=$diskSize"
```

## <a name="confirm-your-disk-has-changed-tiers"></a>Confirmación de que el disco ha cambiado de nivel

Los cambios del nivel de rendimiento pueden tardar hasta 15 minutos en finalizar. Para confirmar que el disco ha cambiado de nivel, use uno de los siguientes métodos:

### <a name="azure-resource-manager"></a>Azure Resource Manager

```cli
az resource show -n $diskName -g $resourceGroupName --namespace Microsoft.Compute --resource-type disks --api-version 2020-12-01 --query [properties.tier] -o tsv
```

### <a name="azure-cli"></a>Azure CLI

```azurecli
az disk show -n $diskName -g $resourceGroupName --query [tier] -o tsv
```

## <a name="next-steps"></a>Pasos siguientes

Si necesita cambiar el tamaño de un disco para aprovechar las ventajas de los niveles de rendimiento superiores, consulte estos artículos:

- [Expansión de discos duros virtuales en una VM Linux con la CLI de Azure](linux/expand-disks.md)
- [Expansión de un disco administrado asociado a una máquina virtual de Windows](windows/expand-os-disk.md)
