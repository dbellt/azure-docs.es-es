---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 03/18/2020
ms.author: cynthn
ms.custom: include file, devx-track-azurepowershell
ms.openlocfilehash: 8e6a079aec7fd47e2909958d71ab45eb04e676da
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110720747"
---
## <a name="create-an-image-gallery"></a>Creación de una galería de imágenes 

Una galería de imágenes es el recurso principal que se usa para habilitar el uso compartido de imágenes. Los caracteres permitidos para el nombre de la Galería son letras mayúsculas o minúsculas, números y puntos. El nombre de la galería no puede contener guiones. Los nombres de las galerías deben ser únicos dentro de su suscripción. 

Cree una galería de imágenes con [New-AzGallery](/powershell/module/az.compute/new-azgallery). En el ejemplo siguiente se crea una galería denominada *myGallery* en el grupo de recursos *myGalleryRG*.

```azurepowershell-interactive
$resourceGroup = New-AzResourceGroup `
   -Name 'myGalleryRG' `
   -Location 'West Central US'  
$gallery = New-AzGallery `
   -GalleryName 'myGallery' `
   -ResourceGroupName $resourceGroup.ResourceGroupName `
   -Location $resourceGroup.Location `
   -Description 'Shared Image Gallery for my organization'  
```
   

## <a name="share-the-gallery"></a>Uso compartido de la galería

Se recomienda compartir el acceso en el nivel de la galería de imágenes. Use una dirección de correo electrónico y el cmdlet [Get-AzADUser](/powershell/module/az.resources/get-azaduser) para obtener el identificador de objeto del usuario y, después, utilice [New-AzRoleAssignment](/powershell/module/Az.Resources/New-AzRoleAssignment) para concederle acceso a la galería. Reemplace el correo electrónico alinne_montes@contoso.com de este ejemplo por su propia información.

```azurepowershell-interactive
# Get the object ID for the user
$user = Get-AzADUser -StartsWith alinne_montes@contoso.com
# Grant access to the user for our gallery
New-AzRoleAssignment `
   -ObjectId $user.Id `
   -RoleDefinitionName Reader `
   -ResourceName $gallery.Name `
   -ResourceType Microsoft.Compute/galleries `
   -ResourceGroupName $resourceGroup.ResourceGroupName

```
