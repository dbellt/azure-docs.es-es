---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 03/24/2020
ms.author: cynthn
ms.custom: include file, devx-track-azurecli
ms.openlocfilehash: fc35f277bfafa80f6239ef807f1a83591646a503
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107800104"
---
## <a name="create-an-image-gallery"></a>Creación de una galería de imágenes 

Una galería de imágenes es el recurso principal que se usa para habilitar el uso compartido de imágenes. 

Los caracteres permitidos para el nombre de la Galería son letras mayúsculas o minúsculas, números y puntos. El nombre de la galería no puede contener guiones.   Los nombres de las galerías deben ser únicos dentro de su suscripción. 

Cree una galería de imágenes mediante [az sig az create](/cli/azure/sig#az_sig_create). En el ejemplo siguiente se crea un grupo de recursos denominado *myGalleryRG* en la región *Este de EE. UU.* y una galería denominada *myGallery*.

```azurecli-interactive
az group create --name myGalleryRG --location eastus
az sig create --resource-group myGalleryRG --gallery-name myGallery
```

## <a name="share-the-gallery"></a>Uso compartido de la galería

Puede compartir imágenes entre suscripciones mediante el Control de acceso basado en roles (RBAC). Puede compartir imágenes en el nivel de la galería, la definición de imagen o la versión de imagen. Cualquier usuario que tenga permisos de lectura para una versión de la imagen, incluso entre suscripciones, podrá implementar una VM con la versión de la imagen.

Se recomienda que el uso compartido con otros usuarios se realice en el nivel de la galería. Para obtener el identificador de objeto de la galería, use [az sig show](/cli/azure/sig#az_sig_show).

```azurecli-interactive
az sig show \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --query id
```

Use el identificador de objeto como ámbito, junto con una dirección de correo electrónico y [az role assignment create](/cli/azure/role/assignment#az_role_assignment_create) para dar a un usuario acceso a la galería de imágenes compartidas. Reemplace `<email-address>` y `<gallery iD>` por su propia información.

```azurecli-interactive
az role assignment create \
   --role "Reader" \
   --assignee <email address> \
   --scope <gallery ID>
```

Para más información sobre cómo compartir recursos con RBAC, consulte [Administración del acceso mediante RBAC y la CLI de Azure](../articles/role-based-access-control/role-assignments-cli.md).