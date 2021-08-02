---
title: Creación de una versión de una imagen de una máquina virtual a partir de otra existente mediante Azure Image Builder
description: Creación de una nueva versión de una imagen de máquina virtual a partir de otra existente mediante Azure Image Builder en Linux.
author: cynthn
ms.author: cynthn
ms.date: 03/02/2020
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: image-builder
ms.collection: linux
ms.reviewer: danis
ms.openlocfilehash: e9508da2002035ee413d557c0cd48474b6e10e0d
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112071252"
---
# <a name="create-a-new-vm-image-version-from-an-existing-image-version-using-azure-image-builder-in-linux"></a>Creación de una versión de una imagen de una máquina virtual a partir de otra existente mediante Azure Image Builder en Linux

En este artículo se muestra cómo tomar una versión de imagen existente en una [galería de imágenes compartidas](../shared-image-galleries.md), actualizarla y publicarla como una nueva versión de imagen en la galería.

Se usará una plantilla .json de ejemplo para configurar la imagen. El archivo .json que se usa aquí es: [helloImageTemplateforSIGfromSIG.json](https://raw.githubusercontent.com/azure/azvmimagebuilder/master/quickquickstarts/2_Creating_a_Custom_Linux_Shared_Image_Gallery_Image_from_SIG/helloImageTemplateforSIGfromSIG.json). 


## <a name="register-the-features"></a>Registro de las características
Para usar Azure Image Builder, debe registrar la característica.

Compruebe el registro.

```azurecli-interactive
az provider show -n Microsoft.VirtualMachineImages | grep registrationState
az provider show -n Microsoft.KeyVault | grep registrationState
az provider show -n Microsoft.Compute | grep registrationState
az provider show -n Microsoft.Storage | grep registrationState
az provider show -n Microsoft.Network | grep registrationState
```

Si no se muestran como registradas, ejecute lo siguiente:

```azurecli-interactive
az provider register -n Microsoft.VirtualMachineImages
az provider register -n Microsoft.Compute
az provider register -n Microsoft.KeyVault
az provider register -n Microsoft.Storage
az provider register -n Microsoft.Network
```


## <a name="set-variables-and-permissions"></a>Establecimiento de variables y permisos

Si ha usado [Crear una imagen y distribuirla a una galería de imágenes compartidas](image-builder-gallery.md) para crear la galería de imágenes compartidas, ya se han creado algunas de las variables necesarias. Si no es así, configure algunas variables para usarlas en este ejemplo.


```console
# Resource group name 
sigResourceGroup=ibLinuxGalleryRG
# Gallery location 
location=westus2
# Additional region to replicate the image version to 
additionalregion=eastus
# Name of the shared image gallery 
sigName=myIbGallery
# Name of the image definition to use
imageDefName=myIbImageDef
# image distribution metadata reference name
runOutputName=aibSIGLinuxUpdate
```

Cree una variable para el id. de suscripción. Puede obtenerlo mediante `az account show | grep id`.

```console
subscriptionID=<Subscription ID>
```

Obtenga la versión de la imagen que quiera actualizar.

```azurecli
sigDefImgVersionId=$(az sig image-version list \
   -g $sigResourceGroup \
   --gallery-name $sigName \
   --gallery-image-definition $imageDefName \
   --subscription $subscriptionID --query [].'id' -o json | grep 0. | tr -d '"' | tr -d '[:space:]')
```

## <a name="create-a-user-assigned-identity-and-set-permissions-on-the-resource-group"></a>Creación de una identidad asignada por el usuario y establecimiento de los permisos en el grupo de recursos
Como ha establecido la identidad del usuario en el ejemplo anterior, solo tiene que obtener el identificador de recurso correspondiente, que se anexará a la plantilla.

```azurecli-interactive
#get identity used previously
imgBuilderId=$(az identity list -g $sigResourceGroup --query "[?contains(name, 'aibBuiUserId')].id" -o tsv)
```

Si ya tiene una galería de imágenes compartidas propia y no ha seguido el ejemplo anterior, tendrá que asignar permisos a Image Builder para acceder al grupo de recursos, para que pueda acceder a la galería. Revise los pasos del ejemplo [Crear una imagen y distribuirla en Shared Image Gallery](image-builder-gallery.md).


## <a name="modify-helloimage-example"></a>Modificación del ejemplo helloImage
Para revisar el ejemplo que se va a usar, abra el archivo .json aquí: [helloImageTemplateforSIGfromSIG.json](https://raw.githubusercontent.com/azure/azvmimagebuilder/master/quickquickstarts/2_Creating_a_Custom_Linux_Shared_Image_Gallery_Image_from_SIG/helloImageTemplateforSIGfromSIG.json) junto con la [referencia de plantillas de Image Builder](image-builder-json.md). 


Descargue el ejemplo de .json y configúrelo con las variables. 

```console
curl https://raw.githubusercontent.com/azure/azvmimagebuilder/master/quickquickstarts/8_Creating_a_Custom_Linux_Shared_Image_Gallery_Image_from_SIG/helloImageTemplateforSIGfromSIG.json -o helloImageTemplateforSIGfromSIG.json
sed -i -e "s/<subscriptionID>/$subscriptionID/g" helloImageTemplateforSIGfromSIG.json
sed -i -e "s/<rgName>/$sigResourceGroup/g" helloImageTemplateforSIGfromSIG.json
sed -i -e "s/<imageDefName>/$imageDefName/g" helloImageTemplateforSIGfromSIG.json
sed -i -e "s/<sharedImageGalName>/$sigName/g" helloImageTemplateforSIGfromSIG.json
sed -i -e "s%<sigDefImgVersionId>%$sigDefImgVersionId%g" helloImageTemplateforSIGfromSIG.json
sed -i -e "s/<region1>/$location/g" helloImageTemplateforSIGfromSIG.json
sed -i -e "s/<region2>/$additionalregion/g" helloImageTemplateforSIGfromSIG.json
sed -i -e "s/<runOutputName>/$runOutputName/g" helloImageTemplateforSIGfromSIG.json
sed -i -e "s%<imgBuilderId>%$imgBuilderId%g" helloImageTemplateforSIGfromSIG.json
```

## <a name="create-the-image"></a>Crear la imagen

Envíe la configuración de la imagen al servicio de generador de imágenes de máquina virtual.

```azurecli-interactive
az resource create \
    --resource-group $sigResourceGroup \
    --properties @helloImageTemplateforSIGfromSIG.json \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateforSIGfromSIG01
```

Inicie la generación de imágenes.

```azurecli-interactive
az resource invoke-action \
     --resource-group $sigResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n helloImageTemplateforSIGfromSIG01 \
     --action Run 
```

Espere hasta que se haya creado la imagen y la replicación antes de continuar con el paso siguiente.


## <a name="create-the-vm"></a>Creación de la máquina virtual

```azurecli-interactive
az vm create \
  --resource-group $sigResourceGroup \
  --name aibImgVm001 \
  --admin-username azureuser \
  --location $location \
  --image "/subscriptions/$subscriptionID/resourceGroups/$sigResourceGroup/providers/Microsoft.Compute/galleries/$sigName/images/$imageDefName/versions/latest" \
  --generate-ssh-keys
```

Cree una conexión SSH a la máquina virtual mediante la dirección IP pública de la misma.

```console
ssh azureuser@<pubIp>
```

Debería ver que la imagen se ha personalizado con un "Mensaje del día" en cuanto se establece la conexión SSH.

```output
*******************************************************
**            This VM was built from the:            **
**      !! AZURE VM IMAGE BUILDER Custom Image !!    **
**         You have just been Customized :-)         **
*******************************************************
```

Escriba `exit` para cerrar la conexión SSH.

También puede enumerar las versiones de la imagen que ahora están disponibles en la galería.

```azurecli-interactive
az sig image-version list -g $sigResourceGroup -r $sigName -i $imageDefName -o table
```


## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre los componentes del archivo .json que se usan en este artículo, vea [Referencia de la plantilla de generador de imágenes](../linux/image-builder-json.md).
