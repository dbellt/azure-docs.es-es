---
title: 'Versión preliminar: creación de una versión de imagen cifrada con claves propias'
description: Cree una versión de imagen en una galería de imágenes compartidas mediante claves de cifrado administradas por el cliente.
author: cynthn
ms.service: virtual-machines
ms.subservice: shared-image-gallery
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 11/3/2020
ms.author: cynthn
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 20e5d4f0d9d3f8f8ab168ca7699f99bc40919b32
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110669484"
---
# <a name="preview-use-customer-managed-keys-for-encrypting-images"></a>Vista previa: uso de claves administradas por el cliente para el cifrado de imágenes

Las imágenes de una galería de imágenes compartidas se almacenan como instantáneas, por lo que se cifran automáticamente a través del cifrado del lado servidor. El cifrado del lado servidor usa el [cifrado AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) de 256 bits, uno de los cifrados de bloques más seguros que existen. El cifrado del lado servidor también es compatible con FIPS 140-2. Para más información sobre de los módulos criptográficos subyacentes en los discos administrados de Azure, consulte [Cryptography API: última generación](/windows/desktop/seccng/cng-portal).

Puede confiar en las claves administradas por la plataforma para el cifrado de las imágenes, o bien usar claves propias. También puede usarlas conjuntamente para el cifrado doble. Si opta por administrar el cifrado con claves propias, puede especificar una *clave administrada por el cliente* que se usará para cifrar y descifrar todos los discos de las imágenes. 

El cifrado del lado servidor mediante claves administradas por el cliente usa Azure Key Vault. Puede importar [las claves RSA](../key-vault/keys/hsm-protected-keys.md) a su instancia de Key Vault o generar nuevas claves RSA en Azure Key Vault.

## <a name="prerequisites"></a>Requisitos previos

Para este artículo, es necesario que ya disponga de un conjunto de cifrado de disco en cada región donde quiere replicar la imagen:

- Para usar solo una clave administrada por el cliente, consulte los artículos sobre la habilitación de claves administradas por el cliente con el cifrado del lado servidor mediante [Azure Portal](./disks-enable-customer-managed-keys-portal.md) o [PowerShell](./windows/disks-enable-customer-managed-keys-powershell.md#set-up-an-azure-key-vault-and-diskencryptionset-without-automatic-key-rotation).

- Para usar las claves administradas por la plataforma y las administradas por el cliente (para el cifrado doble), consulte los artículos sobre la habilitación del cifrado doble en reposo mediante [Azure Portal](./disks-enable-double-encryption-at-rest-portal.md) o [PowerShell](./windows/disks-enable-double-encryption-at-rest-powershell.md).

   > [!IMPORTANT]
   > Debe usar el vínculo [https://aka.ms/diskencryptionupdates](https://aka.ms/diskencryptionupdates) para acceder a Azure Portal. El cifrado doble en reposo no está visible actualmente en Azure Portal público, salvo si usa el vínculo.

## <a name="limitations"></a>Limitaciones

Si usa claves administradas por el cliente para cifrar imágenes en una galería de imágenes compartidas, se aplican las limitaciones siguientes:   

- Los conjuntos de claves de cifrado deben estar en la misma suscripción que la imagen.

- Los conjuntos de claves de cifrado son recursos regionales, por lo que cada región requiere un conjunto de claves de cifrado diferente.

- No se pueden copiar ni compartir imágenes que usan claves administradas por el cliente. 

- Una vez que ha usado claves propias para cifrar un disco o una imagen, no puede volver a usar claves administradas por la plataforma para cifrar esos discos o imágenes.


> [!IMPORTANT]
> El cifrado mediante claves administradas por el cliente se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin un Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


## <a name="powershell"></a>PowerShell

En la versión preliminar pública, primero debe registrar la característica:

```azurepowershell-interactive
Register-AzProviderFeature -FeatureName SIGEncryption -ProviderNamespace Microsoft.Compute
```

El proceso de registro tarda unos minutos en completarse. Utilice `Get-AzProviderFeature` para comprobar el estado del registro de la característica:

```azurepowershell-interactive
Get-AzProviderFeature -FeatureName SIGEncryption -ProviderNamespace Microsoft.Compute
```

Cuando `RegistrationState` devuelva `Registered`, puede avanzar al paso siguiente.

Compruebe el registro del proveedor. Asegúrese de que devuelve `Registered`.

```azurepowershell-interactive
Get-AzResourceProvider -ProviderNamespace Microsoft.Compute | Format-table -Property ResourceTypes,RegistrationState
```

Si no devuelve `Registered`, use el código siguiente para registrar los proveedores:

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
```

Para especificar un conjunto de cifrado de disco para una versión de imagen, use [New-AzGalleryImageDefinition](/powershell/module/az.compute/new-azgalleryimageversion) con el parámetro `-TargetRegion`: 

```azurepowershell-interactive

$sourceId = <ID of the image version source>

$osDiskImageEncryption = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myDESet'}

$dataDiskImageEncryption1 = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myDESet1';Lun=1}

$dataDiskImageEncryption2 = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myDESet2';Lun=2}

$dataDiskImageEncryptions = @($dataDiskImageEncryption1,$dataDiskImageEncryption2)

$encryption1 = @{OSDiskImage=$osDiskImageEncryption;DataDiskImages=$dataDiskImageEncryptions}

$region1 = @{Name='West US';ReplicaCount=1;StorageAccountType=Standard_LRS;Encryption=$encryption1}

$eastUS2osDiskImageEncryption = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myEastUS2DESet'}

$eastUS2dataDiskImageEncryption1 = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myEastUS2DESet1';Lun=1}

$eastUS2dataDiskImageEncryption2 = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myEastUS2DESet2';Lun=2}

$eastUS2DataDiskImageEncryptions = @($eastUS2dataDiskImageEncryption1,$eastUS2dataDiskImageEncryption2)

$encryption2 = @{OSDiskImage=$eastUS2osDiskImageEncryption;DataDiskImages=$eastUS2DataDiskImageEncryptions}

$region2 = @{Name='East US 2';ReplicaCount=1;StorageAccountType=Standard_LRS;Encryption=$encryption2}

$targetRegion = @($region1, $region2)


# Create the image
New-AzGalleryImageVersion `
   -ResourceGroupName $rgname `
   -GalleryName $galleryName `
   -GalleryImageDefinitionName $imageDefinitionName `
   -Name $versionName -Location $location `
   -SourceImageId $sourceId `
   -ReplicaCount 2 `
   -StorageAccountType Standard_LRS `
   -PublishingProfileEndOfLifeDate '2020-12-01' `
   -TargetRegion $targetRegion
```

### <a name="create-a-vm"></a>Crear una VM

Puede crear una máquina virtual (VM) a partir de una galería de imágenes compartidas y usar claves administradas por el cliente para cifrar los discos. La sintaxis es la misma que para crear una VM [generalizada](vm-generalized-image-version-powershell.md) o [especializada](vm-specialized-image-version-powershell.md) a partir de una imagen. Use el conjunto de parámetros extendido y agregue `Set-AzVMOSDisk -Name $($vmName +"_OSDisk") -DiskEncryptionSetId $diskEncryptionSet.Id -CreateOption FromImage` a la configuración de la VM.

En el caso de los discos de datos, agregue el parámetro `-DiskEncryptionSetId $setID` al usar [Add-AzVMDataDisk](/powershell/module/az.compute/add-azvmdatadisk).


## <a name="cli"></a>CLI 

En la versión preliminar pública, primero debe registrar la característica. El registro tarda unos 30 minutos.

```azurecli-interactive
az feature register --namespace Microsoft.Compute --name SIGEncryption
```

Compruebe el estado del registro de la característica:

```azurecli-interactive
az feature show --namespace Microsoft.Compute --name SIGEncryption | grep state
```

Cuando este código devuelve `"state": "Registered"`, puede avanzar al paso siguiente.

Compruebe el registro:

```azurecli-interactive
az provider show -n Microsoft.Compute | grep registrationState
```

Si no se muestra como registrado, ejecute el comando siguiente:

```azurecli-interactive
az provider register -n Microsoft.Compute
```


Para especificar un conjunto de cifrado de disco para una versión de imagen, use [az image gallery create-image-version](/cli/azure/sig/image-version#az_sig_image_version_create) con el parámetro `--target-region-encryption`. El formato de `--target-region-encryption` es una lista de claves separadas por comas para cifrar los discos de sistema operativo y de datos. Debería ser parecido a este: `<encryption set for the OS disk>,<Lun number of the data disk>,<encryption set for the data disk>,<Lun number for the second data disk>,<encryption set for the second data disk>`. 

Si el origen del disco del sistema operativo es un disco administrado o una máquina virtual, use `--managed-image` para especificar el origen de la versión de la imagen. En este ejemplo, el origen es una imagen administrada que tiene un disco del sistema operativo y un disco de datos en LUN 0. El disco del sistema operativo se cifrará con DiskEncryptionSet1 y el disco de datos, con DiskEncryptionSet2.

```azurecli-interactive
az sig image-version create \
   -g MyResourceGroup \
   --gallery-image-version 1.0.0 \
   --location westus \
   --target-regions westus=2=standard_lrs eastus2 \
   --target-region-encryption WestUSDiskEncryptionSet1,0,WestUSDiskEncryptionSet2 EastUS2DiskEncryptionSet1,0,EastUS2DiskEncryptionSet2 \
   --gallery-name MyGallery \
   --gallery-image-definition MyImage \
   --managed-image "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/images/myImage"
```

Si el origen del disco del sistema operativo es una instantánea, use `--os-snapshot` para especificarlo. Si hay instantáneas de disco de datos que también deben formar parte de la versión de la imagen, agréguelas. Use `--data-snapshot-luns` para especificar el LUN y `--data-snapshots` para especificar las instantáneas.

En este ejemplo, los orígenes son instantáneas de disco. Hay un disco del sistema operativo y también uno de datos en LUN 0. El disco del sistema operativo se cifrará con DiskEncryptionSet1 y el disco de datos, con DiskEncryptionSet2.

```azurecli-interactive
az sig image-version create \
   -g MyResourceGroup \
   --gallery-image-version 1.0.0 \
   --location westus\
   --target-regions westus=2=standard_lrs eastus\
   --target-region-encryption WestUSDiskEncryptionSet1,0,WestUSDiskEncryptionSet2 EastUS2DiskEncryptionSet1,0,EastUS2DiskEncryptionSet2 \
   --os-snapshot "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/snapshots/myOSSnapshot" \
   --data-snapshot-luns 0 \
   --data-snapshots "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/snapshots/myDDSnapshot" \
   --gallery-name MyGallery \
   --gallery-image-definition MyImage 
   
```

### <a name="create-the-vm"></a>Creación de la máquina virtual

Puede crear una máquina virtual a partir de una galería de imágenes compartidas y usar claves administradas por el cliente para cifrar los discos. La sintaxis es la misma que para crear una VM [generalizada](vm-generalized-image-version-cli.md) o [especializada](vm-specialized-image-version-cli.md) a partir de una imagen. Solo tiene que agregar el parámetro `--os-disk-encryption-set` con el identificador del conjunto de cifrado. En el caso de los discos de datos, agregue `--data-disk-encryption-sets` con una lista delimitada por espacios de los conjuntos de cifrado de disco para los discos de datos.


## <a name="portal"></a>Portal

Al crear la versión de la imagen en el portal, puede usar la pestaña **Cifrado** para aplicar los conjuntos de cifrado de almacenamiento.

> [!IMPORTANT]
> Para usar el cifrado doble, debe usar el vínculo [https://aka.ms/diskencryptionupdates](https://aka.ms/diskencryptionupdates) para acceder a Azure Portal. El cifrado doble en reposo no está visible actualmente en Azure Portal público, salvo si usa el vínculo.


1. En la página **Crear una versión de imagen**, seleccione la pestaña **Cifrado**.
2. En **Tipo de cifrado**, seleccione **Cifrado en reposo con una clave administrada por el cliente** o **Cifrado doble con claves administradas por el cliente y la plataforma**. 
3. Para cada disco de la imagen, seleccione un conjunto de cifrado en la lista desplegable **Conjunto de cifrado de disco**. 

### <a name="create-the-vm"></a>Creación de la máquina virtual

Puede crear una máquina virtual a partir de una versión de imagen y usar claves administradas por el cliente para cifrar los discos. Al crear la VM en el portal, en la pestaña **Discos**, seleccione **Cifrado en reposo con claves administradas por el cliente** o **Cifrado doble con claves administradas por el cliente y la plataforma** para **Tipo de cifrado**. Después puede seleccionar el conjunto de cifrado en la lista desplegable.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre [cifrado de disco del lado servidor](./disk-encryption.md).

Para obtener información sobre cómo proporcionar información del plan de compra, consulte [Indicación de la información del plan de compra de Azure Marketplace al crear imágenes](marketplace-images.md).
