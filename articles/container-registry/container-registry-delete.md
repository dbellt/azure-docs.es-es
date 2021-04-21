---
title: Eliminar recursos de imagen
description: Detalles sobre cómo administrar de forma eficaz el tamaño del registro mediante la eliminación de datos de imagen de contenedor con comandos de la CLI de Azure.
ms.topic: article
ms.date: 07/31/2019
ms.openlocfilehash: af277d0c02960c989b4e9119f2ecbfd8f6d7ce07
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107783996"
---
# <a name="delete-container-images-in-azure-container-registry-using-the-azure-cli"></a>Eliminación de imágenes de contenedor en Azure Container Registry con la CLI de Azure

Para mantener el tamaño del registro de contenedor de Azure, debe eliminar periódicamente los datos de imagen obsoletos. Aunque algunas imágenes de contenedor implementadas en producción pueden requerir un almacenamiento a largo plazo, otras normalmente se pueden eliminar antes. Por ejemplo, en un escenario de compilación y prueba automatizado, el registro se puede rellenar rápidamente con imágenes que es posible que nunca se implementen y se pueden purgar poco después de completar el paso de compilación y prueba.

Dado que puede eliminar datos de la imagen de varias maneras diferentes, es importante comprender cómo afecta cada operación de eliminación al uso del almacenamiento. En este artículo se tratan varios métodos para eliminar los datos de imagen:

* Eliminar un [repositorio](#delete-repository): elimina todas las imágenes y todas las capas únicas dentro del repositorio.
* Eliminar por [etiqueta](#delete-by-tag): elimina una imagen, la etiqueta, todas las capas únicas a las que hace referencia la imagen y todas las demás etiquetas asociadas con la imagen.
* Eliminar por [hash de manifiesto](#delete-by-manifest-digest): elimina una imagen, todas las capas únicas a las que hace referencia la imagen y todas las etiquetas asociadas con la imagen.

Los scripts de ejemplo se proporcionan para ayudarlo a automatizar las operaciones de eliminación.

Para una introducción a estos conceptos, consulte [About registries, repositories, and images](container-registry-concepts.md) (Acerca de los registros, los repositorios y las imágenes).

## <a name="delete-repository"></a>Eliminación de un repositorio

Al eliminar un repositorio se eliminan todas las imágenes del repositorio, incluidas todas las etiquetas, capas únicas y manifiestos. Cuando se elimina un repositorio, se recupera el espacio de almacenamiento utilizado por las imágenes que hacen referencia a capas únicas de ese repositorio.

El siguiente comando de la CLI de Azure elimina el repositorio "acr-helloworld" y todas las etiquetas y manifiestos dentro del repositorio. Si no se hace referencia a las capas a las que hacen referencia los manifiestos eliminados desde cualquier otra imagen del registro, también se eliminan los datos de la capa, recuperando el espacio de almacenamiento.

```azurecli
 az acr repository delete --name myregistry --repository acr-helloworld
```

## <a name="delete-by-tag"></a>Eliminación por etiqueta

Puede eliminar imágenes individuales de un repositorio especificando el nombre del repositorio y la etiqueta en la operación de eliminación. Cuando se elimina por etiqueta, se recupera el espacio de almacenamiento utilizado por todas las capas únicas de la imagen (las capas no compartidas por cualquier otra imagen del registro).

Para eliminar por etiqueta, use [az acr repository delete][az-acr-repository-delete] y especifique el nombre de la imagen en el parámetro `--image`. Se eliminan todas las capas únicas de la imagen y las demás etiquetas asociadas con la imagen.

Por ejemplo, para eliminar la imagen "acr-helloworld:latest" del registro "myregistry":

```azurecli
az acr repository delete --name myregistry --image acr-helloworld:latest
```

```output
This operation will delete the manifest 'sha256:0a2e01852872580b2c2fea9380ff8d7b637d3928783c55beb3f21a6e58d5d108' and all the following images: 'acr-helloworld:latest', 'acr-helloworld:v3'.
Are you sure you want to continue? (y/n):
```

> [!TIP]
> La eliminación *por etiqueta* no debe confundirse con la eliminación de una etiqueta (quitar etiqueta). Puede eliminar una etiqueta con el comando de la CLI de Azure [az acr repository untag][az-acr-repository-untag]. No se libera espacio al quitar la etiqueta de una imagen porque su [manifiesto](container-registry-concepts.md#manifest) y los datos de la capa permanecen en el registro. Solo se elimina la propia referencia de la etiqueta.

## <a name="delete-by-manifest-digest"></a>Eliminación por hash de manifiesto

Un [hash de manifiesto](container-registry-concepts.md#manifest-digest) se puede asociar con una, ninguna o varias etiquetas. Cuando se elimina por hash, se eliminan todas las etiquetas a las que hace referencia el manifiesto, así como los datos de capa de todas las capas únicas de la imagen. Los datos de las capas compartidas no se eliminan.

Para eliminar por hash, debe enumerar en primer lugar los hash de manifiesto del repositorio que contiene las imágenes que desea eliminar. Por ejemplo:

```azurecli
az acr repository show-manifests --name myregistry --repository acr-helloworld
```

```output
[
  {
    "digest": "sha256:0a2e01852872580b2c2fea9380ff8d7b637d3928783c55beb3f21a6e58d5d108",
    "tags": [
      "latest",
      "v3"
    ],
    "timestamp": "2018-07-12T15:52:00.2075864Z"
  },
  {
    "digest": "sha256:3168a21b98836dda7eb7a846b3d735286e09a32b0aa2401773da518e7eba3b57",
    "tags": [
      "v2"
    ],
    "timestamp": "2018-07-12T15:50:53.5372468Z"
  }
]
```

A continuación, especifique el hash que desea eliminar en el comando [az acr repository delete][az-acr-repository-delete]. El formato del comando es:

```azurecli
az acr repository delete --name <acrName> --image <repositoryName>@<digest>
```

Por ejemplo, para eliminar el último manifiesto enumerado en la salida anterior (con la etiqueta "v2"):

```azurecli
az acr repository delete --name myregistry --image acr-helloworld@sha256:3168a21b98836dda7eb7a846b3d735286e09a32b0aa2401773da518e7eba3b57
```

```output
This operation will delete the manifest 'sha256:3168a21b98836dda7eb7a846b3d735286e09a32b0aa2401773da518e7eba3b57' and all the following images: 'acr-helloworld:v2'.
Are you sure you want to continue? (y/n): 
```

La imagen `acr-helloworld:v2` se elimina del registro, así como los datos de la capa únicos de esa imagen. Si un manifiesto está asociado a varias etiquetas, también se eliminan todas las etiquetas asociadas.

## <a name="delete-digests-by-timestamp"></a>Eliminación de resúmenes por marca de tiempo

Para mantener el tamaño de un registro o un repositorio, puede que deba eliminar periódicamente los resúmenes de manifiesto anteriores a una fecha determinada.

El siguiente comando de la CLI de Azure enumera todos los resúmenes de manifiesto de un repositorio con una antigüedad superior a una marca de tiempo especificada, en orden ascendente. Reemplace `<acrName>` y `<repositoryName>` por los valores adecuados para su entorno. La marca de tiempo podría ser una expresión de fecha y hora completa o una fecha, como en este ejemplo.

```azurecli
az acr repository show-manifests --name <acrName> --repository <repositoryName> \
--orderby time_asc -o tsv --query "[?timestamp < '2019-04-05'].[digest, timestamp]"
```

Después de identificar los resúmenes de manifiesto obsoletos, puede ejecutar el siguiente script de Bash para eliminar los resúmenes de manifiesto anteriores a una marca de tiempo especificada. Requiere la CLI de Azure y **xargs**. De forma predeterminada, el script no realiza ninguna eliminación. Establezca el valor de `ENABLE_DELETE` en `true` para habilitar la eliminación de imágenes.

> [!WARNING]
> Use el siguiente script de ejemplo con precaución: los datos de las imágenes eliminadas son IRRECUPERABLES. Si tiene sistemas que extraen imágenes por resumen de manifiesto (en lugar de por el nombre de la imagen), no debe ejecutar estos scripts. La eliminación de los resúmenes de manifiesto impide que esos sistemas extraigan las imágenes del registro. En lugar de extraer por manifiesto, considere la posibilidad de adoptar un esquema de *etiquetado único*, un [procedimiento recomendado](container-registry-image-tag-version.md). 

```bash
#!/bin/bash

# WARNING! This script deletes data!
# Run only if you do not have systems
# that pull images via manifest digest.

# Change to 'true' to enable image delete
ENABLE_DELETE=false

# Modify for your environment
# TIMESTAMP can be a date-time string such as 2019-03-15T17:55:00.
REGISTRY=myregistry
REPOSITORY=myrepository
TIMESTAMP=2019-04-05  

# Delete all images older than specified timestamp.

if [ "$ENABLE_DELETE" = true ]
then
    az acr repository show-manifests --name $REGISTRY --repository $REPOSITORY \
    --orderby time_asc --query "[?timestamp < '$TIMESTAMP'].digest" -o tsv \
    | xargs -I% az acr repository delete --name $REGISTRY --image $REPOSITORY@% --yes
else
    echo "No data deleted."
    echo "Set ENABLE_DELETE=true to enable deletion of these images in $REPOSITORY:"
    az acr repository show-manifests --name $REGISTRY --repository $REPOSITORY \
   --orderby time_asc --query "[?timestamp < '$TIMESTAMP'].[digest, timestamp]" -o tsv
fi
```

## <a name="delete-untagged-images"></a>Eliminación de imágenes sin etiqueta

Como se mencionó en la sección [Hash de manifiesto](container-registry-concepts.md#manifest-digest), la inserción de una imagen modificada con una etiqueta existente **quita las etiquetas** de la imagen insertada previamente, lo que resulta en una imagen huérfana (o "pendiente"). El manifiesto de la imagen insertada previamente y los datos de la capa permanecen en el registro. Considere la siguiente secuencia de eventos:

1. Insertar la imagen *acr-helloworld* con la etiqueta **latest**: `docker push myregistry.azurecr.io/acr-helloworld:latest`
1. Comprobar los manifiestos del repositorio *acr-helloworld*:

   ```azurecli
   az acr repository show-manifests --name myregistry --repository acr-helloworld
   
   ```
   
   ```output
   [
     {
       "digest": "sha256:d2bdc0c22d78cde155f53b4092111d7e13fe28ebf87a945f94b19c248000ceec",
       "tags": [
         "latest"
       ],
       "timestamp": "2018-07-11T21:32:21.1400513Z"
     }
   ]
   ```

1. Modificar el archivo de Docker *acr-helloworld*
1. Insertar la imagen *acr-helloworld* con la etiqueta **latest**: `docker push myregistry.azurecr.io/acr-helloworld:latest`
1. Comprobar los manifiestos del repositorio *acr-helloworld*:

   ```azurecli
   az acr repository show-manifests --name myregistry --repository acr-helloworld
   ```
   
   ```output
   [
     {
       "digest": "sha256:7ca0e0ae50c95155dbb0e380f37d7471e98d2232ed9e31eece9f9fb9078f2728",
       "tags": [
         "latest"
       ],
       "timestamp": "2018-07-11T21:38:35.9170967Z"
     },
     {
       "digest": "sha256:d2bdc0c22d78cde155f53b4092111d7e13fe28ebf87a945f94b19c248000ceec",
       "tags": [],
       "timestamp": "2018-07-11T21:32:21.1400513Z"
     }
   ]
   ```

Como puede ver en la salida del último paso de la secuencia, ahora hay un manifiesto huérfano cuya propiedad `"tags"` es una lista vacía. Este manifiesto sigue existiendo en el registro, junto con los datos de cualquier capa única a la que haga referencia. **Para eliminar estas imágenes huérfanas y sus datos de capa, debe eliminar por hash del manifiesto**.

## <a name="delete-all-untagged-images"></a>Eliminación de todas las imágenes sin etiqueta

Puede enumerar todas las imágenes sin etiqueta del repositorio mediante el siguiente comando de la CLI de Azure. Reemplace `<acrName>` y `<repositoryName>` por los valores adecuados para su entorno.

```azurecli
az acr repository show-manifests --name <acrName> --repository <repositoryName> --query "[?tags[0]==null].digest"
```

Al usar este comando en un script, puede eliminar todas las imágenes sin etiquetas en un repositorio.

> [!WARNING]
> Utilice los siguientes scripts de ejemplo con precaución: los datos de las imágenes eliminadas son IRRECUPERABLES. Si tiene sistemas que extraen imágenes por resumen de manifiesto (en lugar de por el nombre de la imagen), no debe ejecutar estos scripts. La eliminación de imágenes sin etiqueta impedirá que esos sistemas extraigan las imágenes del registro. En lugar de extraer por manifiesto, considere la posibilidad de adoptar un esquema de *etiquetado único*, un [procedimiento recomendado](container-registry-image-tag-version.md).

**CLI de Azure en Bash**

El siguiente script de Bash elimina todas las imágenes no etiquetadas de un repositorio. Requiere la CLI de Azure y **xargs**. De forma predeterminada, el script no realiza ninguna eliminación. Establezca el valor de `ENABLE_DELETE` en `true` para habilitar la eliminación de imágenes.

```bash
#!/bin/bash

# WARNING! This script deletes data!
# Run only if you do not have systems
# that pull images via manifest digest.

# Change to 'true' to enable image delete
ENABLE_DELETE=false

# Modify for your environment
REGISTRY=myregistry
REPOSITORY=myrepository

# Delete all untagged (orphaned) images
if [ "$ENABLE_DELETE" = true ]
then
    az acr repository show-manifests --name $REGISTRY --repository $REPOSITORY  --query "[?tags[0]==null].digest" -o tsv \
    | xargs -I% az acr repository delete --name $REGISTRY --image $REPOSITORY@% --yes
else
    echo "No data deleted."
    echo "Set ENABLE_DELETE=true to enable image deletion of these images in $REPOSITORY:"
    az acr repository show-manifests --name $REGISTRY --repository $REPOSITORY --query "[?tags[0]==null]" -o tsv
fi
```

**CLI de Azure en PowerShell**

El siguiente script de PowerShell elimina todas las imágenes no etiquetadas de un repositorio. Requiere PowerShell y la CLI de Azure. De forma predeterminada, el script no realiza ninguna eliminación. Establezca el valor de `$enableDelete` en `$TRUE` para habilitar la eliminación de imágenes.

```powershell
# WARNING! This script deletes data!
# Run only if you do not have systems
# that pull images via manifest digest.

# Change to '$TRUE' to enable image delete
$enableDelete = $FALSE

# Modify for your environment
$registry = "myregistry"
$repository = "myrepository"

if ($enableDelete) {
    az acr repository show-manifests --name $registry --repository $repository --query "[?tags[0]==null].digest" -o tsv `
    | %{ az acr repository delete --name $registry --image $repository@$_ --yes }
} else {
    Write-Host "No data deleted."
    Write-Host "Set `$enableDelete = `$TRUE to enable image deletion."
    az acr repository show-manifests --name $registry --repository $repository --query "[?tags[0]==null]" -o tsv
}
```


## <a name="automatically-purge-tags-and-manifests-preview"></a>Purga automática de etiquetas y manifiestos (versión preliminar)

Como alternativa a los comandos de la CLI de Azure de scripting, ejecute una instancia de ACR Tasks a petición o programada para eliminar todas las etiquetas que tengan más de una duración determinada o que coincidan con un filtro de nombre especificado. Para más información, consulte [Automatically purge images from an Azure Container Registry](container-registry-auto-purge.md) (Purga automática de imágenes a partir de una instancia de Azure Container Registry).

Opcionalmente, establezca una [directiva de retención](container-registry-retention-policy.md) para cada registro con el fin de administrar los manifiestos sin etiquetar. Cuando se habilita una directiva de retención, los manifiestos de imagen del registro que no tienen etiquetas asociadas, y los datos de la capa subyacentes, se eliminan automáticamente después de un período establecido.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el almacenamiento de imágenes en Azure Container Registry, consulte [Almacenamiento de imágenes de contenedor en Azure Container Registry](container-registry-storage.md).

<!-- IMAGES -->
[manifest-digest]: ./media/container-registry-delete/01-manifest-digest.png

<!-- LINKS - External -->
[docker-manifest-inspect]: https://docs.docker.com/edge/engine/reference/commandline/manifest/#manifest-inspect
[portal]: https://portal.azure.com

<!-- LINKS - Internal -->
[az-acr-repository-delete]: /cli/azure/acr/repository#az_acr_repository_delete
[az-acr-repository-show-manifests]: /cli/azure/acr/repository#az_acr_repository_show_manifests
[az-acr-repository-untag]: /cli/azure/acr/repository#az_acr_repository_untag
