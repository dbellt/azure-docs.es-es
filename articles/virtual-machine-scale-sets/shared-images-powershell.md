---
title: Uso de imágenes de máquinas virtuales compartidas para crear un conjunto de escalado en Azure PowerShell
description: Obtenga información sobre cómo usar Azure PowerShell para crear imágenes de máquinas virtuales compartidas que se utilizarán para implementar conjuntos de escalado de máquina virtual en Azure.
author: cynthn
ms.service: virtual-machine-scale-sets
ms.subservice: shared-image-gallery
ms.topic: how-to
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: mimckitt
ms.openlocfilehash: dbb7e07c9abc11e25e4fd46c185b413f51cec147
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112026244"
---
# <a name="create-and-use-shared-images-for-virtual-machine-scale-sets-with-the-azure-powershell"></a>Creación y uso de imágenes compartidas personalizada para conjuntos de escalado de máquinas virtuales con Azure PowerShell

Al crear el conjunto de escalado, se especifica la imagen que se usará cuando se implementen las instancias de máquina virtual. El servicio Galería de imágenes compartidas simplifica en gran medida compartir imágenes personalizadas en una organización. Las imágenes personalizadas son como las imágenes de Marketplace, pero las puede crear usted mismo. Las imágenes personalizadas pueden usarse para configuraciones de arranque como la carga previa de aplicaciones, configuraciones de aplicaciones y otras configuraciones del sistema operativo. 

La Galería de imágenes compartidas le permite compartir sus imágenes de máquina virtual personalizadas con otros usuarios de su organización, ya sea dentro o entre regiones, dentro de un inquilino de AAD. Elija las imágenes que desea compartir, qué regiones desea que estén disponibles en ellas y con quién desea compartirlas. Puede crear varias galerías que le permitirán agrupar lógicamente las imágenes compartidas. 

La galería es un recurso de nivel superior que proporciona control de acceso basado en roles de Azure (RBAC de Azure). Las imágenes pueden tener varias versiones y se puede optar por replicar cada versión de la imagen en un conjunto diferente de regiones de Azure. La galería solo funciona con imágenes administradas. 

La característica de galería de imágenes compartidas tiene varios tipos de recursos. 


[!INCLUDE [virtual-machines-shared-image-gallery-resources](../../includes/virtual-machines-shared-image-gallery-resources.md)]


## <a name="before-you-begin"></a>Antes de empezar

Los siguientes pasos explican cómo tomar una máquina virtual existente y convertirla en una imagen personalizada reutilizable que puede usar para crear nuevas instancias de máquinas virtuales.

Para completar el ejemplo de este artículo, debe tener una imagen administrada existente. Puede seguir [Tutorial: Creación y uso de una imagen personalizada para conjuntos de escalado de máquinas virtuales con Azure PowerShell](tutorial-use-custom-image-powershell.md) para crear una si es necesario. Si la imagen administrada contiene un disco de datos, el tamaño del disco de datos no puede ser mayor de 1 TB.

Al trabajar en el artículo, reemplace los nombres de grupo de recursos y máquina virtual cuando proceda.


[!INCLUDE [virtual-machines-common-shared-images-ps](../../includes/virtual-machines-common-shared-images-powershell.md)]




## <a name="next-steps"></a>Pasos siguientes

Puede crear también recursos de galería de imágenes compartidas con plantillas. Hay varias plantillas de Inicio rápido de Azure disponibles: 

- [Creación de una galería de imágenes compartidas](https://azure.microsoft.com/resources/templates/sig-create/)
- [Creación de una definición de imagen en una galería de imágenes compartidas](https://azure.microsoft.com/resources/templates/sig-image-definition-create/)
- [Creación de una versión de imagen en una galería de imágenes compartidas](https://azure.microsoft.com/resources/templates/sig-image-version-create/)
- [Creación de una máquina virtual a partir de la versión de la imagen](https://azure.microsoft.com/resources/templates/vm-from-sig/)

Para más información sobre las galerías de imágenes compartidas, consulte la [Introducción](../virtual-machines/shared-image-galleries.md). Si encuentra problemas, consulte [Solución de problemas de galerías de imágenes compartidas](../virtual-machines/troubleshooting-shared-images.md).
