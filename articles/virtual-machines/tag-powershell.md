---
title: Etiquetado de una máquina virtual con PowerShell
description: Más información sobre cómo etiquetar una máquina virtual con PowerShell
author: cynthn
ms.service: virtual-machines
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 11/11/2020
ms.author: cynthn
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: ed1201bf90a7178d59a6f7e287dd144c14790838
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110668082"
---
# <a name="how-to-tag-a-virtual-machine-in-azure-using-powershell"></a>Etiquetado de una máquina virtual en Azure con PowerShell

En este artículo se describe cómo etiquetar una máquina virtual en Azure mediante PowerShell. Las etiquetas son pares clave-valor definidos por el usuario que se pueden colocar directamente en un recurso o un grupo de recursos. Actualmente, Azure admite un máximo de 50 etiquetas por recurso y grupo de recursos. Las etiquetas se pueden colocar en un recurso en el momento de su creación, o bien se pueden agregar a un recurso existente. Si quiere etiquetar una máquina virtual con la CLI de Azure, consulte [Etiquetado de una máquina virtual Linux en Azure](tag-cli.md).

Use el cmdlet `Get-AzVM` para ver la lista actual de etiquetas para su máquina virtual.

```azurepowershell-interactive
Get-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM" | Format-List -Property Tags
```

Si la máquina virtual ya contiene etiquetas, las verá todas en formato de lista.

Para agregar etiquetas, use el comando `Set-AzResource`. Si actualiza las etiquetas mediante PowerShell, se actualizan todas ellas en conjunto. Si va a agregar una etiqueta a un recurso que ya tiene etiquetas, tendrá que incluir todas las etiquetas que desea colocar en el recurso. A continuación se muestra un ejemplo de cómo agregar etiquetas adicionales a un recurso mediante los cmdlets de PowerShell.

Asigne todas las etiquetas actuales de la máquina virtual a la variable `$tags`, mediante las propiedades `Get-AzResource` y `Tags`.

```azurepowershell-interactive
$tags = (Get-AzResource -ResourceGroupName myResourceGroup -Name myVM).Tags
```

Para ver las etiquetas actuales, escriba la variable.

```azurepowershell-interactive
$tags
```

Este es el aspecto que podría tener el resultado:

```output
Key           Value
----          -----
Department    MyDepartment
Application   MyApp1
Created By    MyName
Environment   Production
```

En el ejemplo siguiente, se agrega una etiqueta llamada `Location` con el valor `myLocation`. Use `+=` para anexar el nuevo par de clave-valor a la lista `$tags`.

```azurepowershell-interactive
$tags += @{Location="myLocation"}
```

Use `Set-AzResource` para establecer todas las etiquetas definidas en la variable *$tags* en la máquina virtual.

```azurepowershell-interactive
Set-AzResource -ResourceGroupName myResourceGroup -Name myVM -ResourceType "Microsoft.Compute/VirtualMachines" -Tag $tags
```

Use `Get-AzResource` para mostrar todas las etiquetas del recurso.

```azurepowershell-interactive
(Get-AzResource -ResourceGroupName myResourceGroup -Name myVM).Tags

```

La salida debe tener un aspecto similar al siguiente, que ahora incluye la nueva etiqueta:

```output

Key           Value
----          -----
Department    MyDepartment
Application   MyApp1
Created By    MyName
Environment   Production
Location      MyLocation
```

### <a name="next-steps"></a>Pasos siguientes

- Para más información sobre el etiquetado de los recursos de Azure, consulte [Información general de Azure Resource Manager](../azure-resource-manager/management/overview.md) y [Uso de etiquetas para organizar los recursos de Azure](../azure-resource-manager/management/tag-resources.md).
- Para ver cómo las etiquetas pueden ayudarle a administrar el uso de recursos de Azure, consulte [Descripción de la factura de Microsoft Azure](../cost-management-billing/understand/review-individual-bill.md).
