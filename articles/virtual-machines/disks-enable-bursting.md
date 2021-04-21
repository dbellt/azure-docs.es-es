---
title: Habilitación de la expansión de disco a petición
description: Habilite la expansión de disco a petición en el disco administrado.
author: albecker1
ms.author: albecker
ms.date: 03/02/2021
ms.topic: conceptual
ms.service: virtual-machines
ms.subservice: disks
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: 5110e580bada7bb1090b17d6df22a9354622e8e4
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107483153"
---
# <a name="enable-on-demand-bursting"></a>Habilitación de la expansión de disco a petición

Las unidades de estado sólido (SSD) Premium tienen dos modelos de expansión de disco disponibles; expansión de disco basada en crédito y expansión de disco a petición. En este artículo se explica cómo cambiar a la expansión de disco a petición. Los discos que usan el modelo a petición se pueden expandir más allá de sus destinos aprovisionados originalmente. La expansión de disco a petición se produce con la frecuencia que necesite la carga de trabajo, hasta el destino de expansión máximo. La expansión de disco a petición conlleva cargos adicionales.

Para más información sobre la expansión de disco, consulte [Expansión de discos administrados](disk-bursting.md).

> [!IMPORTANT]
> No es necesario seguir los pasos descritos en este artículo para usar la expansión de disco basada en crédito. De manera predeterminada, la expansión de disco basada en crédito está habilitada en todos los discos elegibles.

Antes de habilitar la expansión de disco a petición, debe comprender lo siguiente:

[!INCLUDE [managed-disk-bursting-regions-limitations](../../includes/managed-disk-bursting-regions-limitations.md)]

### <a name="regional-availability"></a>Disponibilidad regional

[!INCLUDE [managed-disk-bursting-availability](../../includes/managed-disk-bursting-availability.md)]

## <a name="get-started"></a>Introducción

La expansión de disco a petición se puede habilitar con el módulo de Azure PowerShell, la CLI de Azure o con plantillas de Azure Resource Manager. En los ejemplos siguientes se describe cómo crear un nuevo disco con la expansión de disco a petición habilitada y cómo habilitar la expansión de disco a petición en discos existentes.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Los cmdlets de expansión de disco a petición están disponibles en la versión 5.5.0 y posteriores del módulo Az. Como alternativa, puede usar [Azure Cloud Shell](https://shell.azure.com/).
### <a name="create-an-empty-data-disk-with-on-demand-bursting"></a>Creación de un disco de datos vacío con expansión de disco a petición

Un disco administrado debe tener un tamaño mayor de 512 GiB para habilitar la expansión de disco a petición. Reemplace los parámetros `<myResourceGroupDisk>` y `<myDataDisk>` y, a continuación, ejecute el siguiente script para crear un disco SSD Premium con expansión de disco a petición:

```azurepowershell
Set-AzContext -SubscriptionName <yourSubscriptionName>

$diskConfig = New-AzDiskConfig -Location 'WestCentralUS' -CreateOption Empty -DiskSizeGB 1024 -SkuName Premium_LRS -BurstingEnabled $true

$dataDisk = New-AzDisk -ResourceGroupName <myResourceGroupDisk> -DiskName <myDataDisk> -Disk $diskConfig
```

### <a name="enable-on-demand-bursting-on-an-existing-disk"></a>Habilitación de la expansión de disco a petición en un disco existente

Un disco administrado debe tener un tamaño mayor de 512 GiB para habilitar la expansión de disco a petición. Reemplace los parámetros `<myResourceGroupDisk>` y `<myDataDisk>` y, a continuación, ejecute este comando para habilitar la expansión de disco a petición en un disco existente:

```azurepowershell
New-AzDiskUpdateConfig -BurstingEnabled $true | Update-AzDisk -ResourceGroupName <myResourceGroupDisk> -DiskName <myDataDisk> //Set the flag to $false to disable on-demand bursting
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Los cmdlets de expansión de disco a petición están disponibles en la versión 2.19.0 y posteriores del [módulo de la CLI de Azure](/cli/azure/install-azure-cli). Como alternativa, puede usar [Azure Cloud Shell](https://shell.azure.com/).

### <a name="create-and-attach-a-on-demand-bursting-data-disk"></a>Creación y conexión de un disco de datos con expansión de disco a petición

Un disco administrado debe tener un tamaño mayor de 512 GiB para habilitar la expansión de disco a petición. Reemplace los parámetros `<yourDiskName>`, `<yourResourceGroup>` y `<yourVMName>` y, a continuación, ejecute los siguientes comandos para crear un disco SSD Premium con expansión de disco a petición:

```azurecli
az disk create -g <yourResourceGroup> -n <yourDiskName> --size-gb 1024 --sku Premium_LRS -l westcentralus --enable-bursting true

az vm disk attach --vm-name <yourVMName> --name <yourDiskName> --resource-group <yourResourceGroup>
```

### <a name="enable-on-demand-bursting-on-an-existing-disk---cli"></a>Habilitación de la expansión de disco a petición en un disco existente: CLI

Un disco administrado debe tener un tamaño mayor de 512 GiB para habilitar la expansión de disco a petición. Reemplace los parámetros `<myResourceGroupDisk>` y `<yourDiskName>` y, a continuación, ejecute este comando para habilitar la expansión de disco a petición en un disco existente:

```azurecli
az disk update --name <yourDiskName> --resource-group <yourResourceGroup> --enable-bursting true //Set the flag to false to disable on-demand bursting
```

# <a name="azure-resource-manager"></a>[Azure Resource Manager](#tab/azure-resource-manager)

Con la API de disco `2020-09-30`, puede habilitar la expansión de disco a petición en los discos SSD Premium recién creados o existentes de más de 512 GiB. La API `2020-09-30` presentó una nueva propiedad, `burstingEnabled`. Esta propiedad está establecida en False de manera predeterminada. La siguiente plantilla de ejemplo crea un disco SSD Premium de 1 TiB en la región Centro-oeste de EE. UU. con la expansión de disco habilitada:

```
{
  "$schema": "http://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "diskSkuName": {
        "type": "string",
        "defaultValue": "Premium_LRS" //Supported on premium SSDs only
},
    "dataDiskSizeInGb": {
      "type": "string",
      "defaultValue": "1024" //Supported on disk size > 512 GiB
},
    "location": {
      "type": "string",
      "defaultValue": "westcentralus" //Preview regions: West Central US
},
"diskApiVersion": {
      "type": "string",
      "defaultValue": "2020-09-30" //Preview supported version: 2020-09-30 or above
    }
  },
  "resources": [
    {
      "apiVersion": "[parameters('diskApiVersion')]",
      "type": "Microsoft.Compute/disks",
      "name": "[parameters('diskName')]",
      "location": "[parameters(location)]",
      "properties": {
        "creationData": {
          "createOption": "Empty"
        },
        "diskSizeGB": "[parameters('dataDiskSizeInGb')]",
        "burstingEnabled": "true" //Feature flag to enable disk bursting on disks > 512 GiB
      },
      "sku": {
        "name": "[parameters('diskSkuName')]"
      }
    ]
}
```
---
 
## <a name="next-steps"></a>Pasos siguientes

Para obtener detalles sobre los recursos de expansión de disco, consulte [Métricas de expansión de disco](disks-metrics.md).
