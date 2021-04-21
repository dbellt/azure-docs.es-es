---
title: Bloqueo de imágenes
description: Establezca atributos para una imagen de contenedor o un repositorio a fin de que pueda eliminarse o sobrescribirse en una instancia de Azure Container Registry.
ms.topic: article
ms.date: 09/30/2019
ms.openlocfilehash: 340beb1bb6666ddf0de7de38adee6be71f5f52bd
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107772350"
---
# <a name="lock-a-container-image-in-an-azure-container-registry"></a>Bloqueo de una imagen de contenedor en una instancia de Azure Container Registry

En Azure Container Registry, puede bloquear una versión de la imagen o un repositorio para que no pueda eliminarse o actualizarse. Para bloquear una imagen o un repositorio, actualice sus atributos mediante el comando [az acr repository update][az-acr-repository-update] de la CLI de Azure. 

En este artículo se requiere que ejecute la CLI de Azure en Azure Cloud Shell o localmente (se recomienda la versión 2.0.55 o posterior). Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli].

> [!IMPORTANT]
> Este artículo no se aplica al bloqueo de un registro completo, por ejemplo, mediante el uso de **Configuración > Bloqueos** en Azure Portal o los comandos `az lock` de la CLI de Azure. El bloqueo de un recurso de registro no impide la creación, actualización o eliminación de datos en los repositorios. El bloqueo de un registro solo afecta a las operaciones de administración, como agregar o eliminar replicaciones o eliminar el propio registro. Para más información, consulte [Bloqueo de recursos para impedir cambios inesperados](../azure-resource-manager/management/lock-resources.md).

## <a name="scenarios"></a>Escenarios

De forma predeterminada, una imagen etiquetada en Azure Container Registry es *mutable*, por lo que con los permisos adecuados puede actualizar e insertar repetidamente una imagen con la misma etiqueta en un registro. Las imágenes de contenedor también se pueden [eliminar](container-registry-delete.md) según sea necesario. Este comportamiento es útil al desarrollar imágenes y es necesario mantener un tamaño para el registro.

Sin embargo, cuando implemente una imagen de contenedor en producción, puede que necesite una *inmutable*. Una imagen inmutable es aquella que no puede eliminar o sobrescribir por accidente.

Consulte [Recomendaciones para el etiquetado y el control de versiones de las imágenes de contenedor](container-registry-image-tag-version.md) para las estrategias de etiquetado y versión de las imágenes en el registro.

Use el comando [az acr repository update][az-acr-repository-update] para establecer los atributos del repositorio de forma que pueda:

* Bloquear una versión de la imagen o un repositorio entero

* Proteger una versión de la imagen o un repositorio de la eliminación, pero permitir actualizaciones

* Evitar operaciones de lectura (extracción) en una versión de la imagen o en la totalidad del repositorio

Consulte las secciones siguientes para ver ejemplos. 

## <a name="lock-an-image-or-repository"></a>Bloquear una imagen o un repositorio 

### <a name="show-the-current-repository-attributes"></a>Mostrar los atributos del repositorio actual
Para ver los atributos actuales de un repositorio, ejecute el siguiente comando [az acr repository show][az-acr-repository-show]:

```azurecli
az acr repository show \
    --name myregistry --repository myrepo \
    --output jsonc
```

### <a name="show-the-current-image-attributes"></a>Mostrar los atributos actuales de la imagen
Para ver los atributos actuales de una etiqueta, ejecute el siguiente comando [az acr repository show][az-acr-repository-show]:

```azurecli
az acr repository show \
    --name myregistry --image image:tag \
    --output jsonc
```

### <a name="lock-an-image-by-tag"></a>Bloquear una imagen por etiqueta

Para bloquear la imagen *myrepo/myimage:tag* en *myregistry*, ejecute el siguiente comando [az acr repository update][az-acr-repository-update]:

```azurecli
az acr repository update \
    --name myregistry --image myrepo/myimage:tag \
    --write-enabled false
```

### <a name="lock-an-image-by-manifest-digest"></a>Bloquear una imagen por la síntesis del manifiesto

Para bloquear una imagen *myrepo/myimage* identificada por la síntesis del manifiesto (hash de SHA-256, representado como `sha256:...`), ejecute el siguiente comando. (Para encontrar la síntesis del manifiesto asociada con una o varias etiquetas de imagen, ejecute el comando [az acr repository show-manifests][az-acr-repository-show-manifests]).

```azurecli
az acr repository update \
    --name myregistry --image myrepo/myimage@sha256:123456abcdefg \
    --write-enabled false
```

### <a name="lock-a-repository"></a>Bloqueo de un repositorio

Para bloquear el repositorio *myrepo/myimage* y todas las imágenes que contiene, ejecute el siguiente comando:

```azurecli
az acr repository update \
    --name myregistry --repository myrepo/myimage \
    --write-enabled false
```

## <a name="protect-an-image-or-repository-from-deletion"></a>Proteger una imagen o un repositorio de la eliminación

### <a name="protect-an-image-from-deletion"></a>Proteger una imagen de la eliminación

Para permitir que la imagen *myrepo/myimage:tag* se actualice pero no se elimine, ejecute el siguiente comando:

```azurecli
az acr repository update \
    --name myregistry --image myrepo/myimage:tag \
    --delete-enabled false --write-enabled true
```

### <a name="protect-a-repository-from-deletion"></a>Proteger un repositorio de la eliminación

El comando siguiente establece el repositorio *myrepo/myimage* para que no se pueda eliminar. Sin embargo, se pueden actualizar o eliminar imágenes individuales.

```azurecli
az acr repository update \
    --name myregistry --repository myrepo/myimage \
    --delete-enabled false --write-enabled true
```

## <a name="prevent-read-operations-on-an-image-or-repository"></a>Evitar operaciones de lectura en una imagen o un repositorio

Para evitar operaciones de lectura (extracción) en la imagen *myrepo/myimage:tag*, ejecute el siguiente comando:

```azurecli
az acr repository update \
    --name myregistry --image myrepo/myimage:tag \
    --read-enabled false
```

Para evitar operaciones de lectura en todas las imágenes del repositorio *myrepo/myimage*, ejecute el siguiente comando:

```azurecli
az acr repository update \
    --name myregistry --repository myrepo/myimage \
    --read-enabled false
```

## <a name="unlock-an-image-or-repository"></a>Desbloquear una imagen o un repositorio

Para restaurar el comportamiento predeterminado de la imagen *myrepo/myimage:tag* para que se pueda eliminar y actualizar, ejecute el siguiente comando:

```azurecli
az acr repository update \
    --name myregistry --image myrepo/myimage:tag \
    --delete-enabled true --write-enabled true
```

Para restaurar el comportamiento predeterminado del repositorio *myrepo/myimage* y de todas las imágenes para que se puedan eliminar y actualizar, ejecutan el siguiente comando:

```azurecli
az acr repository update \
    --name myregistry --repository myrepo/myimage \
    --delete-enabled true --write-enabled true
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido sobre el uso del comando [az acr repository update][az-acr-repository-update] para evitar la eliminación o la actualización de versiones de imágenes de un repositorio. Para establecer atributos adicionales, consulte la referencia del comando [az acr repository update][az-acr-repository-update].

Para ver los atributos establecidos para un repositorio o una versión de la imagen, use el comando [az acr repository show][az-acr-repository-show].

Para más información sobre las operaciones de eliminación, consulte [Eliminación de imágenes de contenedor en Azure Container Registry][container-registry-delete].

<!-- LINKS - Internal -->
[az-acr-repository-update]: /cli/azure/acr/repository#az_acr_repository_update
[az-acr-repository-show]: /cli/azure/acr/repository#az_acr_repository_show
[az-acr-repository-show-manifests]: /cli/azure/acr/repository#az_acr_repository_show_manifests
[azure-cli]: /cli/azure/install-azure-cli
[container-registry-delete]: container-registry-delete.md
