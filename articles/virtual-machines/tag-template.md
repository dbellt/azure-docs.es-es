---
title: Etiquetado de una máquina virtual mediante una plantilla
description: Información sobre cómo etiquetar una máquina virtual mediante una plantilla.
ms.service: virtual-machines
ms.topic: how-to
ms.workload: infrastructure-services
ms.author: cynthn
author: cynthn
ms.date: 10/26/2018
ms.openlocfilehash: 7a07c3b2c95367debb375785effc15b69dc8de0a
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110087587"
---
# <a name="tagging-a-vm-using-a-template"></a>Etiquetado de una máquina virtual mediante una plantilla

En este artículo se describe cómo etiquetar una máquina virtual en Azure mediante una plantilla de Resource Manager. Las etiquetas son pares clave-valor definidos por el usuario que se pueden colocar directamente en un recurso o un grupo de recursos. Actualmente, Azure admite un máximo de 50 etiquetas por recurso y grupo de recursos. Las etiquetas se pueden colocar en un recurso en el momento de su creación, o bien se pueden agregar a un recurso existente.

[Esta plantilla](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/vm-tags) coloca etiquetas en los siguientes recursos: proceso (máquina virtual), almacenamiento (cuenta de almacenamiento) y red (dirección IP pública, red virtual e interfaz de red). Esta plantilla es para una máquina virtual Windows, pero se puede adaptar a máquinas virtuales Linux.

Haga clic en el botón **Implementar en Azure** del [vínculo de la plantilla](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/vm-tags). Esto le remitirá al [Portal de Azure](https://portal.azure.com/) , donde puede implementar esta plantilla.

![Implementación sencilla con etiquetas](./media/tag/deploy-to-azure-tags.png)

Esta plantilla incluye las siguientes etiquetas: *Department*, *Application* y *Created By*. Estas etiquetas se pueden agregar o modificar directamente en la plantilla si se desea cambiar sus nombres.

![Etiquetas de Azure en una plantilla](./media/tag/azure-tags-in-a-template.png)

Como puede ver, las etiquetas se definen como pares de clave-valor, separados por dos puntos (:). Las etiquetas deben definirse con este formato:

```config
"tags": {
    "Key1" : "Value1",
    "Key2" : "Value2"
}
```

Guarde el archivo de plantilla cuando termine de modificarlo con las etiquetas que prefiera.

A continuación, en la sección **Editar parámetros** , puede rellenar los valores de las etiquetas.

![Editar etiquetas en el Portal de Azure](./media/tag/edit-tags-in-azure-portal.png)

Haga clic en **Crear** para implementar esta plantilla con sus valores de etiqueta.

### <a name="next-steps"></a>Pasos siguientes

- Para más información sobre el etiquetado de los recursos de Azure, consulte [Información general de Azure Resource Manager](../azure-resource-manager/management/overview.md) y [Uso de etiquetas para organizar los recursos de Azure](../azure-resource-manager/management/tag-resources.md).
- Para ver cómo las etiquetas pueden ayudarle a administrar el uso de recursos de Azure, consulte [Descripción de la factura de Microsoft Azure](../cost-management-billing/understand/review-individual-bill.md).
