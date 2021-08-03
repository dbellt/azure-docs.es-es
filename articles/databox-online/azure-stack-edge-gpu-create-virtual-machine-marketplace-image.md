---
title: Uso de una imagen de Azure Marketplace para crear una imagen de máquina virtual para un dispositivo Azure Stack Edge Pro con GPU
description: En este artículo se describe cómo usar una imagen de Azure Marketplace para crear una imagen de máquina virtual y usarla en el dispositivo Azure Stack Edge Pro con GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 06/07/2021
ms.author: alkohli
ms.openlocfilehash: 8a4a9834b35055fbd5be4fd50f1d5413d5e54a35
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111758307"
---
# <a name="use-azure-marketplace-image-to-create-vm-image-for-your-azure-stack-edge-pro-gpu"></a>Uso de una imagen de Azure Marketplace para crear una imagen de máquina virtual para Azure Stack Edge Pro con GPU

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

Para implementar máquinas virtuales en el dispositivo Azure Stack Edge Pro con GPU, debe crear una imagen de máquina virtual que puede usar para crear máquinas virtuales. En este artículo se describen los pasos necesarios para crear una imagen de máquina virtual a partir de una imagen de Azure Marketplace. A continuación, puede usar esta imagen de máquina virtual para implementar máquinas virtuales en el dispositivo Azure Stack Edge Pro con GPU.

## <a name="vm-image-workflow"></a>Flujo de trabajo de la imagen de máquina virtual

En los pasos siguientes se describe el flujo de trabajo de la imagen de máquina virtual mediante un flujo de trabajo de Azure Marketplace:

1.  Conéctese a Azure Cloud Shell o a un cliente que tenga instalada la CLI de Azure.
2.  Busque Azure Marketplace e identifique la imagen que prefiera.
3.  Cree un disco administrado nuevo a partir de la imagen de Marketplace.
4.  Exporte un disco duro virtual del disco administrado a una cuenta de Azure Storage.
5.  Limpie el disco administrado.


Para más información, vaya a [Implementación de una máquina virtual en el dispositivo Azure Stack Edge Pro con Azure PowerShell](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md).

## <a name="prerequisites"></a>Prerrequisitos

Antes de poder usar imágenes de Azure Marketplace para Azure Stack Edge, asegúrese de que está conectado a Azure de cualquiera de las maneras siguientes.

[!INCLUDE [azure-cli-prepare-your-environment](../../includes/azure-cli-prepare-your-environment-no-header.md)]


## <a name="search-for-azure-marketplace-images"></a>Búsqueda de imágenes de Azure Marketplace

Ahora identificará una imagen de Azure Marketplace que desea usar. Azure Marketplace hospeda miles de imágenes de VM. 

Para buscar algunas de las imágenes de Marketplace más comunes que coincidan con los criterios de búsqueda, ejecute el comando siguiente.  

```azurecli
az vm image list --all [--publisher <Publisher>] [--offer <Offer>] [--sku <SKU>]
```
Las tres últimas marcas son opcionales pero, si las excluye, se devuelve una lista larga.

Estas son algunas consultas de ejemplo:

```azurecli
#Returns all images of type “Windows Server”
az vm image list --all --publisher "MicrosoftWindowsserver" --offer "WindowsServer" 

#Returns all Windows Server 2019 Datacenter images from West US published by Microsoft
az vm image list --all --location "westus" --publisher "MicrosoftWindowsserver" --offer "WindowsServer" --sku "2019-Datacenter"

#Returns all VM images from a publisher  
az vm image list --all --publisher "Canonical" 
```

Esta es una salida de ejemplo generada al consultar las imágenes de VM de determinado publicador, oferta y SKU.

```output
PS /home/user> az vm image list --all --publisher "Canonical" --offer "UbuntuServer" --sku "12.04.4-LTS"
[
  {
    "offer": "UbuntuServer",
    "publisher": "Canonical",
    "sku": "12.04.4-LTS",
    "urn": "Canonical:UbuntuServer:12.04.4-LTS:12.04.201402270",
    "version": "12.04.201402270"
  },
  {
    "offer": "UbuntuServer",
    "publisher": "Canonical",
    "sku": "12.04.4-LTS",
    "urn": "Canonical:UbuntuServer:12.04.4-LTS:12.04.201404080",
    "version": "12.04.201404080"
  },
  {
    "offer": "UbuntuServer",
    "publisher": "Canonical",
    "sku": "12.04.4-LTS",
    "urn": "Canonical:UbuntuServer:12.04.4-LTS:12.04.201404280",
    "version": "12.04.201404280"
  },
  {
    "offer": "UbuntuServer",
    "publisher": "Canonical",
    "sku": "12.04.4-LTS",
    "urn": "Canonical:UbuntuServer:12.04.4-LTS:12.04.201405140",
    "version": "12.04.201405140"
  },
  {
    "offer": "UbuntuServer",
    "publisher": "Canonical",
    "sku": "12.04.4-LTS",
    "urn": "Canonical:UbuntuServer:12.04.4-LTS:12.04.201406060",
    "version": "12.04.201406060"
  },
  {
    "offer": "UbuntuServer",
    "publisher": "Canonical",
    "sku": "12.04.4-LTS",
    "urn": "Canonical:UbuntuServer:12.04.4-LTS:12.04.201406190",
    "version": "12.04.201406190"
  },
  {
    "offer": "UbuntuServer",
    "publisher": "Canonical",
    "sku": "12.04.4-LTS",
    "urn": "Canonical:UbuntuServer:12.04.4-LTS:12.04.201407020",
    "version": "12.04.201407020"
  },
  {
    "offer": "UbuntuServer",
    "publisher": "Canonical",
    "sku": "12.04.4-LTS",
    "urn": "Canonical:UbuntuServer:12.04.4-LTS:12.04.201407170",
    "version": "12.04.201407170"
  }
]
PS /home/user>
```

>[!IMPORTANT]
> Use solo imágenes de Gen 1. Las imágenes especificadas como Gen 2 (por lo general, la SKU tiene un sufijo "-g2") no funcionan en Azure Stack Edge.

En este ejemplo, seleccionaremos Windows Server 2019 Datacenter Core, versión 2019.0.20190410. Identificaremos esta imagen por su número de recurso universal ("URN"). 
 
:::image type="content" source="media/azure-stack-edge-create-virtual-machine-marketplace-image/marketplace-image-1.png" alt-text="Lista de imágenes de Marketplace":::

A continuación se muestra una lista de los URN de algunas de las imágenes más comunes. Si solo desea la versión más reciente de un SO determinado, puede reemplazar el número de versión por la palabra "latest" ("más reciente") en el URN. Por ejemplo, "MicrosoftWindowsServer:WindowsServer:2019-Datacenter:Latest". 


| SO              | SKU                                     | Versión               | URN                                                                                       |
|-----------------|-----------------------------------------|-----------------------|-------------------------------------------------------------------------------------------|
| Windows Server  | 2019 Datacenter                         | 17763.1879.2104091832 | MicrosoftWindowsServer:WindowsServer:2019-Datacenter:17763.1879.2104091832                |
| Windows Server  | 2019 Datacenter (disco pequeño de 30 GB)      | 17763.1879.2104091832 | MicrosoftWindowsServer:WindowsServer:2019-Datacenter-smalldisk:17763.1879.2104091832      |
| Windows Server  | 2019 Datacenter Core                    | 17763.1879.2104091832 | MicrosoftWindowsServer:WindowsServer:2019-Datacenter-Core:17763.1879.2104091832           |
| Windows Server  | 2019 Datacenter Core (disco pequeño de 30 GB) | 17763.1879.2104091832 | MicrosoftWindowsServer:WindowsServer:2019-Datacenter-Core-smalldisk:17763.1879.2104091832 |
| Escritorio de Windows | Windows 10 20H2 Pro                     | 19042.928.2104091209  | MicrosoftWindowsDesktop:Windows-10:20h2-pro:19042.928.2104091209                          |
| Ubuntu Server   | Canonical Ubuntu Server 18.04 LTS       | 18.04.202002180       | Canonical:UbuntuServer:18.04-LTS:18.04.202002180                                          |
| Ubuntu Server   | Canonical Ubuntu Server 16.04 LTS       | 16.04.202104160       | Canonical:UbuntuServer:16.04-LTS:16.04.202104160                                          |
| CentOS          | CentOS 8.1                              | 8.1.2020062400        | OpenLogic:CentOS:8_1:8.1.2020062400                                                       |
| CentOS          | CentOS 7.7                              | 7.7.2020062400        | OpenLogic:CentOS:7.7:7.7.2020062400                                                       |



## <a name="create-a-new-managed-disk-from-the-marketplace-image"></a>Creación de un disco administrado a partir de la imagen de Marketplace

Cree un disco administrado de Azure a partir de la imagen de Marketplace que elija. 

1. Configure algunos parámetros.

    ```azurecli
    $urn = <URN of the Marketplace image> #Example: “MicrosoftWindowsServer:WindowsServer:2019-Datacenter:Latest”
    $diskName = <disk name> #Name for new disk to be created
    $diskRG = <resource group> #Resource group that contains the new disk
    ```


1. Cree el disco y genere una dirección URL de acceso de SAS.

    ```azurecli
    az disk create -g $diskRG -n $diskName --image-reference $urn
    $sas = az disk grant-access --duration-in-seconds 36000 --access-level Read --name $diskName --resource-group $diskRG
    $diskAccessSAS = ($sas | ConvertFrom-Json)[0].accessSas 
    ```

A continuación, se incluye una salida de ejemplo:

```output
PS /home/user> $urn = “MicrosoftWindowsServer:WindowsServer:2019-Datacenter:Latest”
PS /home/user> $diskName = "newmanageddisk1"
PS /home/user> $diskRG = "newrgmd1"
PS /home/user> az disk create -g $diskRG -n $diskName --image-reference $urn
{
  "burstingEnabled": null,
  "creationData": {
    "createOption": "FromImage",
    "galleryImageReference": null,
    "imageReference": {
      "id": "/Subscriptions/db4e2fdb-6d80-4e6e-b7cd-736098270664/Providers/Microsoft.Compute/Locations/eastus/Publishers/MicrosoftWindowsServer/ArtifactTypes/VMImage/Offers/WindowsServer/Skus/2019-Datacenter/Versions/17763.1935.2105080716",
      "lun": null
    },
    "logicalSectorSize": null,
    "sourceResourceId": null,
    "sourceUniqueId": null,
    "sourceUri": null,
    "storageAccountId": null,
    "uploadSizeBytes": null
  },
  "diskAccessId": null,
  "diskIopsReadOnly": null,
  "diskIopsReadWrite": 500,
  "diskMBpsReadOnly": null,
  "diskMBpsReadWrite": 100,
  "diskSizeBytes": 136367308800,
  "diskSizeGb": 127,
  "diskState": "Unattached",
  "encryption": {
    "diskEncryptionSetId": null,
    "type": "EncryptionAtRestWithPlatformKey"
  },
  "encryptionSettingsCollection": null,
  "extendedLocation": null,
  "hyperVGeneration": "V1",
  "id": "/subscriptions/db4e2fdb-6d80-4e6e-b7cd-736098270664/resourceGroups/newrgmd1/providers/Microsoft.Compute/disks/NewManagedDisk1",
  "location": "eastus",
  "managedBy": null,
  "managedByExtended": null,
  "maxShares": null,
  "name": "NewManagedDisk1",
  "networkAccessPolicy": "AllowAll",
  "osType": "Windows",
  "propertyUpdatesInProgress": null,
  "provisioningState": "Succeeded",
  "purchasePlan": null,
  "resourceGroup": "newrgmd1",
  "securityProfile": null,
  "shareInfo": null,
  "sku": {
    "name": "Premium_LRS",
    "tier": "Premium"
  },
  "supportsHibernation": null,
  "tags": {},
  "tier": "P10",
  "timeCreated": "2021-06-08T00:39:34.205982+00:00",
  "type": "Microsoft.Compute/disks",
  "uniqueId": "1a649ad4-3b95-471e-89ef-1d2ed1f51525",
  "zones": null
}

PS /home/user> $sas = az disk grant-access --duration-in-seconds 36000 --access-level Read --name $diskName --resource-group $diskRG
PS /home/user>  $diskAccessSAS = ($sas | ConvertFrom-Json)[0].accessSas
PS /home/user>
```

## <a name="export-a-vhd-from-the-managed-disk-to-azure-storage"></a>Exportación de un disco duro virtual del disco administrado a Azure Storage 

En este paso se exportará un VHD del disco administrado a la cuenta de Azure Blob Storage que prefiera. Este VHD se puede usar para crear imágenes de VM en Azure Stack Edge.

1. Establezca la cuenta de almacenamiento de destino en la que se copiará el disco duro virtual.
    
    ```azurecli
    $storageAccountName = <destination storage account name>
    $containerName = <destination container name>
    $destBlobName = <blobname.vhd> #Blob that will be created, including .vhd extension
    $storageAccountKey = <storage account key>
    ```

1. Copie el VHD en la cuenta de almacenamiento de destino.

    ```azurecli
    $destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
    Start-AzureStorageBlobCopy -AbsoluteUri $diskAccessSAS -DestContainer $containerName -DestContext $destContext -DestBlob $destBlobName
    ```

    La copia del VHD tardará varios minutos en completarse. Asegúrese de que la copia se completó antes de seguir mediante la ejecución de este comando. El campo de estado mostrará "Correcto" una vez que se complete.
    
    ```azurecli
    Get-AzureStorageBlobCopyState –Container $containerName –Context $destContext -Blob $destBlobName 
    ```

A continuación, se incluye una salida de ejemplo:

```output
PS /home/user> $storageAccountName = "edgeazurevmeus"
PS /home/user> $containerName = "azurevmmp"
PS /home/user> $destBlobName = "newblobmp.vhd"
PS /home/user> $storageAccountKey = "n9sCytWLdTBz0F4Sco9SkPGWp6BJBtf7BJBk79msf1PfxJGQdqSfu6TboZWZ10xyZdc4y+Att08cC9B79jB0YA=="

PS /home/user> $destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
PS /home/user> Start-AzureStorageBlobCopy -AbsoluteUri $diskAccessSAS -DestContainer $containerName -DestContext $destContext -DestBlob $destBlobName

   AccountName: edgeazurevmeus, ContainerName: azurevmmp

Name                 BlobType  Length          ContentType                    LastModified         AccessTier SnapshotTime                 IsDeleted  VersionId
----                 --------  ------          -----------                    ------------         ---------- ------------                 ---------  ---------
newblobmp.vhd        PageBlob  -1                                             2021-06-08 00:50:10Z                                         False

PS /home/user> Get-AzureStorageBlobCopyState –Container $containerName –Context $destContext -Blob $destBlobName

CopyId                  : 24a1e3f5-886a-490d-9dd7-562bb4acff58
CompletionTime          :
Status                  : Pending
Source                  : https://md-lfn221fppr2c.blob.core.windows.net/d4tb2hp5ff2q/abcd?sv=2018-03-28&sr=b&si=4f588db1-9aac-44d9-9607-35497cc08a7f
BytesCopied             : 696254464
TotalBytes              : 136367309312
StatusDescription       :
DestinationSnapshotTime :

PS /home/user> Get-AzureStorageBlobCopyState –Container $containerName –Context $destContext -Blob $destBlobName

CopyId                  : 24a1e3f5-886a-490d-9dd7-562bb4acff58
CompletionTime          : 6/8/2021 12:57:26 AM +00:00
Status                  : Success
Source                  : https://md-lfn221fppr2c.blob.core.windows.net/d4tb2hp5ff2q/abcd?sv=2018-03-28&sr=b&si=4f588db1-9aac-44d9-9607-35497cc08a7f
BytesCopied             : 136367309312
TotalBytes              : 136367309312
StatusDescription       :
DestinationSnapshotTime :
```

## <a name="clean-up-the-managed-disk"></a>Limpieza del disco administrado

Para eliminar el disco administrado que creó, siga estos pasos:

```azurecli
az disk revoke-access --name $diskName --resource-group $diskRG 
az disk delete --name $diskName --resource-group $diskRG --yes 
```
La eliminación tarda unos minutos en completarse.

## <a name="next-steps"></a>Pasos siguientes

[Implementación de máquinas virtuales en el dispositivo Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-deploy-virtual-machine-portal.md).