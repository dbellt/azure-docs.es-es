---
title: Cómo etiquetar una máquina virtual de Azure mediante la CLI de Azure
description: Más información sobre cómo etiquetar una máquina virtual con la CLI de Azure
author: cynthn
ms.service: virtual-machines
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 11/11/2020
ms.author: cynthn
ms.custom: devx-track-azurecli
ms.openlocfilehash: 20bb4ab622a01646bcc61d0f691c514a25a06edc
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107502613"
---
# <a name="how-to-tag-a-vm-using-the-azure-cli"></a>Etiquetado de una máquina virtual mediante la CLI de Azure

En este artículo se describe cómo etiquetar una máquina virtual mediante la CLI de Azure. Las etiquetas son pares clave-valor definidos por el usuario que se pueden colocar directamente en un recurso o un grupo de recursos. Actualmente, Azure admite un máximo de 50 etiquetas por recurso y grupo de recursos. Las etiquetas se pueden colocar en un recurso en el momento de su creación, o bien se pueden agregar a un recurso existente. También puede etiquetar una máquina virtual mediante Azure [PowerShell](tag-powershell.md).


Mediante `az vm show` puede ver todas las propiedades de una máquina virtual dada, incluidas las etiquetas.

```azurecli-interactive
az vm show --resource-group myResourceGroup --name myVM --query tags
```

Para agregar una nueva etiqueta de máquina virtual a través de la CLI de Azure, puede usar el comando `azure vm update` junto con el parámetro de etiqueta `--set`:

```azurecli-interactive
az vm update \
    --resource-group myResourceGroup \
    --name myVM \
    --set tags.myNewTagName1=myNewTagValue1 tags.myNewTagName2=myNewTagValue2
```

Para quitar las etiquetas, puede usar el parámetro `--remove` en el comando `azure vm update`.

```azurecli-interactive
az vm update \
   --resource-group myResourceGroup \
   --name myVM \
   --remove tags.myNewTagName1
```

Ahora que hemos aplicado etiquetas a nuestros recursos a través de PowerShell, la CLI de Azure y el portal, analicemos los datos de uso para ver las etiquetas del portal de facturación.

### <a name="next-steps"></a>Pasos siguientes

- Para más información sobre el etiquetado de los recursos de Azure, consulte [Información general de Azure Resource Manager](../azure-resource-manager/management/overview.md) y [Uso de etiquetas para organizar los recursos de Azure](../azure-resource-manager/management/tag-resources.md).
- Para ver cómo las etiquetas pueden ayudarle a administrar el uso de recursos de Azure, consulte [Descripción de la factura de Microsoft Azure](../cost-management-billing/understand/review-individual-bill.md).
