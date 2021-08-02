---
title: Uso de Azure Image Builder y Shared Image Gallery para máquinas virtuales Linux
description: Aprenda a usar Azure Image Builder y la CLI de Azure para crear una versión de la imagen en el servicio Shared Image Gallery y después distribuirla globalmente.
author: cynthn
ms.author: cynthn
ms.date: 03/02/2021
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: image-builder
ms.collection: linux
ms.reviewer: danis
ms.openlocfilehash: 8ba997980de6e28fbe2b12fc9541a1d11667922d
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112028512"
---
# <a name="create-a-linux-image-and-distribute-it-to-a-shared-image-gallery"></a>Crear una imagen de Linux y distribuirla en una galería de imágenes compartidas 

En este artículo se muestra cómo puede usar Azure Image Builder y la CLI de Azure para crear una versión de la imagen en el servicio [Shared Image Gallery](../shared-image-galleries.md) y después distribuirla globalmente. También puede hacerlo mediante [Azure PowerShell](../windows/image-builder-gallery.md).


Se usará una plantilla .json de ejemplo para configurar la imagen. El archivo .json que se usa aquí es: [helloImageTemplateforSIG.json](https://github.com/azure/azvmimagebuilder/blob/master/quickquickstarts/1_Creating_a_Custom_Linux_Shared_Image_Gallery_Image/helloImageTemplateforSIG.json). 

Para distribuir la imagen en una galería de imágenes compartidas, en la plantilla se usa [sharedImage](image-builder-json.md#distribute-sharedimage) como valor de la sección `distribute` de la plantilla.


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

Usaremos algunos datos de forma repetida, por lo que crearemos diversas variables para almacenar esa información.

Image Builder solo admite la creación de imágenes personalizadas en el mismo grupo de recursos que la imagen administrada de origen. Actualice el nombre del grupo de recursos de este ejemplo para que coincida con el de la imagen administrada de origen.

```azurecli-interactive
# Resource group name - we are using ibLinuxGalleryRG in this example
sigResourceGroup=ibLinuxGalleryRG
# Datacenter location - we are using West US 2 in this example
location=westus2
# Additional region to replicate the image to - we are using East US in this example
additionalregion=eastus
# name of the shared image gallery - in this example we are using myGallery
sigName=myIbGallery
# name of the image definition to be created - in this example we are using myImageDef
imageDefName=myIbImageDef
# image distribution metadata reference name
runOutputName=aibLinuxSIG
```

Cree una variable para el id. de suscripción. Puede obtenerlo mediante `az account show | grep id`.

```azurecli-interactive
subscriptionID=<Subscription ID>
```

Cree el grupo de recursos.

```azurecli-interactive
az group create -n $sigResourceGroup -l $location
```

## <a name="create-a-user-assigned-identity-and-set-permissions-on-the-resource-group"></a>Creación de una identidad asignada por el usuario y establecimiento de los permisos en el grupo de recursos
Image Builder usará la [identidad de usuario](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md#user-assigned-managed-identity) proporcionada para insertar la imagen en la instancia de Azure Shared Image Gallery (SIG). En este ejemplo, se creará una definición de roles de Azure que tiene las acciones granulares necesarias para realizar la distribución de la imagen a la instancia de SIG. La definición de roles se asignará a la identidad del usuario.

```bash
# create user assigned identity for image builder to access the storage account where the script is located
idenityName=aibBuiUserId$(date +'%s')
az identity create -g $sigResourceGroup -n $idenityName

# get identity id
imgBuilderCliId=$(az identity show -g $sigResourceGroup -n $idenityName | grep "clientId" | cut -c16- | tr -d '",')

# get the user identity URI, needed for the template
imgBuilderId=/subscriptions/$subscriptionID/resourcegroups/$sigResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/$idenityName

# this command will download an Azure role definition template, and update the template with the parameters specified earlier.
curl https://raw.githubusercontent.com/azure/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json -o aibRoleImageCreation.json

imageRoleDefName="Azure Image Builder Image Def"$(date +'%s')

# update the definition
sed -i -e "s/<subscriptionID>/$subscriptionID/g" aibRoleImageCreation.json
sed -i -e "s/<rgName>/$sigResourceGroup/g" aibRoleImageCreation.json
sed -i -e "s/Azure Image Builder Service Image Creation Role/$imageRoleDefName/g" aibRoleImageCreation.json

# create role definitions
az role definition create --role-definition ./aibRoleImageCreation.json

# grant role definition to the user assigned identity
az role assignment create \
    --assignee $imgBuilderCliId \
    --role $imageRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$sigResourceGroup
```


## <a name="create-an-image-definition-and-gallery"></a>Creación de una definición de imagen y una galería

Para usar Image Builder con una galería de imágenes compartidas, debe tener ya una galería de imágenes y una definición de imagen. Image Builder no creará la galería de imágenes ni la definición de imagen automáticamente.

Si aún no tiene una galería y una definición de imagen para usar, empiece por crearlas. En primer lugar, cree una galería de imágenes.

```azurecli-interactive
az sig create \
    -g $sigResourceGroup \
    --gallery-name $sigName
```

Luego, cree una definición de imagen.

```azurecli-interactive
az sig image-definition create \
   -g $sigResourceGroup \
   --gallery-name $sigName \
   --gallery-image-definition $imageDefName \
   --publisher myIbPublisher \
   --offer myOffer \
   --sku 18.04-LTS \
   --os-type Linux
```


## <a name="download-and-configure-the-json"></a>Descarga y configuración del archivo .json

Descargue la plantilla .json y configúrela con las variables.

```azurecli-interactive
curl https://raw.githubusercontent.com/azure/azvmimagebuilder/master/quickquickstarts/1_Creating_a_Custom_Linux_Shared_Image_Gallery_Image/helloImageTemplateforSIG.json -o helloImageTemplateforSIG.json
sed -i -e "s/<subscriptionID>/$subscriptionID/g" helloImageTemplateforSIG.json
sed -i -e "s/<rgName>/$sigResourceGroup/g" helloImageTemplateforSIG.json
sed -i -e "s/<imageDefName>/$imageDefName/g" helloImageTemplateforSIG.json
sed -i -e "s/<sharedImageGalName>/$sigName/g" helloImageTemplateforSIG.json
sed -i -e "s/<region1>/$location/g" helloImageTemplateforSIG.json
sed -i -e "s/<region2>/$additionalregion/g" helloImageTemplateforSIG.json
sed -i -e "s/<runOutputName>/$runOutputName/g" helloImageTemplateforSIG.json
sed -i -e "s%<imgBuilderId>%$imgBuilderId%g" helloImageTemplateforSIG.json
```

## <a name="create-the-image-version"></a>Creación de la versión de la imagen

En la sección siguiente se creará la versión de la imagen en la galería. 

Envíe la configuración de la imagen al servicio del generador de imágenes de Azure.

```azurecli-interactive
az resource create \
    --resource-group $sigResourceGroup \
    --properties @helloImageTemplateforSIG.json \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateforSIG01
```

Inicie la generación de imágenes.

```azurecli-interactive
az resource invoke-action \
     --resource-group $sigResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n helloImageTemplateforSIG01 \
     --action Run 
```

La creación de la imagen y su replicación en las dos regiones puede llevar un tiempo. Espere a que termine esta parte antes de pasar a la creación de una máquina virtual.


## <a name="create-the-vm"></a>Creación de la máquina virtual

Cree una máquina virtual a partir la versión de la imagen creada por Azure Image Builder.

```azurecli-interactive
az vm create \
  --resource-group $sigResourceGroup \
  --name myAibGalleryVM \
  --admin-username aibuser \
  --location $location \
  --image "/subscriptions/$subscriptionID/resourceGroups/$sigResourceGroup/providers/Microsoft.Compute/galleries/$sigName/images/$imageDefName/versions/latest" \
  --generate-ssh-keys
```

Conéctese mediante SSH a la máquina virtual.

```azurecli-interactive
ssh aibuser@<publicIpAddress>
```

Debería ver que la imagen se ha personalizado con un *mensaje del día* en cuanto se establece la conexión SSH.

```console
*******************************************************
**            This VM was built from the:            **
**      !! AZURE VM IMAGE BUILDER Custom Image !!    **
**         You have just been Customized :-)         **
*******************************************************
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ahora quiere volver a personalizar la versión de la imagen para crear una nueva de la misma imagen, omita los siguientes pasos y vaya a [Uso de Azure Image Builder para crear otra versión de la imagen](image-builder-gallery-update-image-version.md).


Esta acción eliminará la imagen que se ha creado, junto con todos los demás archivos de recursos. Asegúrese de que haya terminado con esta implementación antes de eliminar los recursos.

Al eliminar los recursos de la galería de imágenes, tendrá que eliminar todas las versiones de la imagen antes de poder eliminar la definición de la imagen que se ha usado para crearlas. Para eliminar una galería, primero tiene que haber eliminado todas las definiciones de imagen de la galería.

Elimine la plantilla de Azure Image Builder.

```azurecli-interactive
az resource delete \
    --resource-group $sigResourceGroup \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateforSIG01
```

Elimine las asignaciones de permisos, los roles y la identidad.
```azurecli-interactive
az role assignment delete \
    --assignee $imgBuilderCliId \
    --role "$imageRoleDefName" \
    --scope /subscriptions/$subscriptionID/resourceGroups/$sigResourceGroup

az role definition delete --name "$imageRoleDefName"

az identity delete --ids $imgBuilderId
```

Obtenga la versión de la imagen creada por el generador de imágenes, que siempre empieza por `0.`, y después elimínela.

```azurecli-interactive
sigDefImgVersion=$(az sig image-version list \
   -g $sigResourceGroup \
   --gallery-name $sigName \
   --gallery-image-definition $imageDefName \
   --subscription $subscriptionID --query [].'name' -o json | grep 0. | tr -d '"')
az sig image-version delete \
   -g $sigResourceGroup \
   --gallery-image-version $sigDefImgVersion \
   --gallery-name $sigName \
   --gallery-image-definition $imageDefName \
   --subscription $subscriptionID
```   


Elimine la definición de la imagen.

```azurecli-interactive
az sig image-definition delete \
   -g $sigResourceGroup \
   --gallery-name $sigName \
   --gallery-image-definition $imageDefName \
   --subscription $subscriptionID
```

Elimine la galería.

```azurecli-interactive
az sig delete -r $sigName -g $sigResourceGroup
```

Elimine el grupo de recursos.

```azurecli-interactive
az group delete -n $sigResourceGroup -y
```

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre las [galerías de imágenes compartidas de Azure](../shared-image-galleries.md).
