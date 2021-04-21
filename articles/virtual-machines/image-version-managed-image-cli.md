---
title: Clonación de una imagen administrada en una versión de la imagen con la CLI de Azure
description: Aprenda a clonar una imagen administrada en una versión de la imagen de Shared Image Gallery mediante la CLI de Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: shared-image-gallery
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.custom: devx-track-azurecli
ms.openlocfilehash: 1d0644b9ec9009fe5d1db7701834cb9788f86ab0
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107790174"
---
# <a name="clone-a-managed-image-to-an-image-version-using-the-azure-cli"></a>Clonación de una imagen administrada en una versión de la imagen mediante la CLI de Azure
Si tiene una imagen administrada existente que desee clonar en Shared Image Gallery, puede crear una imagen de Shared Image Gallery directamente desde la imagen administrada. Una vez que haya probado la nueva imagen, puede eliminar la imagen administrada de origen. También puede migrar de una imagen administrada a Shared Image Gallery mediante [PowerShell](image-version-managed-image-powershell.md).

Las imágenes de una galería de imágenes tienen dos componentes, que se crearán durante este ejemplo:
- Una **definición de imagen**, que contiene información sobre la imagen y los requisitos para usarla. Esto incluye si la imagen es Windows o Linux, si es generalizada o especializada, las notas de la versión y los requisitos de memoria mínima y máxima. Es una definición de un tipo de imagen. 
- Una **versión de imagen** es lo que se usa para crear una máquina virtual cuando se usa Shared Image Gallery. Puede tener varias versiones de una imagen según sea necesario para su entorno. Cuando crea una máquina virtual, la versión de la imagen se usa para crear nuevos discos para dicha máquina virtual. Las versiones de las imágenes pueden usarse varias veces.


## <a name="before-you-begin"></a>Antes de empezar

Para completar este artículo, debe tener una instancia de [Shared Image Gallery](shared-images-cli.md). 

Para completar el ejemplo de este artículo, debe tener una imagen administrada existente de una VM generalizada. Para más información, consulte [Captura de una imagen administrada](./linux/capture-image.md). Si la imagen administrada contiene un disco de datos, el tamaño del disco de datos no puede ser mayor de 1 TB.

Al trabajar en este artículo, reemplace los nombres de grupo de recursos y máquina virtual cuando proceda.



## <a name="create-an-image-definition"></a>Creación de la definición de una imagen

Dado que las imágenes administradas son siempre imágenes generalizadas, creará una definición de imagen y usará `--os-state generalized` para lograr una imagen generalizada.

Los nombres de las definiciones de imagen pueden estar formados por letras mayúsculas o minúsculas, números, puntos y guiones. 

Para más información sobre los valores que se pueden especificar para una definición de imagen, consulte [Definiciones de imagen](./shared-image-galleries.md#image-definitions).

Cree una definición de imagen en la galería mediante [az sig image-definition create](/cli/azure/sig/image-definition#az_sig_image_definition_create).

En este ejemplo, la definición de la imagen se denomina *myImageDefinition* y es para una imagen del sistema operativo Linux [generalizada](./shared-image-galleries.md#generalized-and-specialized-images). Para crear una definición para las imágenes que usan un sistema operativo Windows, utilice `--os-type Windows`. 

```azurecli-interactive 
resourceGroup=myGalleryRG
gallery=myGallery
imageDef=myImageDefinition
az sig image-definition create \
   --resource-group $resourceGroup \
   --gallery-name $gallery \
   --gallery-image-definition $imageDef \
   --publisher myPublisher \
   --offer myOffer \
   --sku mySKU \
   --os-type Linux \
   --os-state generalized
```


## <a name="create-the-image-version"></a>Creación de la versión de la imagen

Cree versiones mediante [az image gallery create-image-version](/cli/azure/sig/image-version#az_sig_image_version_create). Deberá pasar el identificador de la imagen administrada que se usará como base para crear la versión de la imagen. Puede usar [az image list](/cli/azure/image?view#az_image_list) para obtener los identificadores de sus imágenes. 

```azurecli-interactive
az image list --query "[].[name, id]" -o tsv
```

Los caracteres permitidos para la versión de una imagen son números y puntos. Los números deben estar dentro del rango de un entero de 32 bits. Formato: *VersiónPrincipal*.*VersiónSecundaria*.*Revisión*.

En este ejemplo, la versión de la imagen es *1.0.0* y vamos a crear una réplica en la región *Centro-sur de EE. UU.* y otra en la región *Este de EE. UU. 2* mediante el almacenamiento con redundancia de zona. Al elegir las regiones de destino de la replicación, recuerde que también debe incluir la región de *origen* como destino de la replicación.

Use el identificador de la imagen administrada en el parámetro `--managed-image`.


```azurecli-interactive 
imageID="/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/images/myImage"
az sig image-version create \
   --resource-group $resourceGroup \
   --gallery-name $gallery \
   --gallery-image-definition $imageDef \
   --gallery-image-version 1.0.0 \
   --target-regions "southcentralus=1" "eastus2=1=standard_zrs" \
   --replica-count 2 \
   --managed-image $imageID
```

> [!NOTE]
> Deberá esperar a que la versión de la imagen termine de compilarse y replicarse por completo antes de poder usar la misma imagen administrada para crear otra versión de la imagen.
>
> También puede almacenar todas las réplicas de la versión de la imagen en [almacenamiento con redundancia de zona](../storage/common/storage-redundancy.md) si agrega `--storage-account-type standard_zrs` al crear la versión de la imagen.
>

## <a name="next-steps"></a>Pasos siguientes

Creación de una máquina virtual a partir de una [versión de imagen generalizada](vm-generalized-image-version-cli.md).

Para obtener información sobre cómo proporcionar información del plan de compra, consulte [Indicación de la información del plan de compra de Azure Marketplace al crear imágenes](marketplace-images.md).