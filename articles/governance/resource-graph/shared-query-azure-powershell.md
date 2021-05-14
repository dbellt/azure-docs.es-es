---
title: 'Inicio rápido: Creación de una consulta compartida con Azure PowerShell'
description: En este inicio rápido puede seguir los pasos necesarios para crear una consulta compartida de Resource Graph mediante Azure PowerShell.
ms.date: 05/01/2021
ms.topic: quickstart
ms.custom:
- devx-track-azurepowershell
- mode-api
ms.openlocfilehash: 0bd223feca24a84903bd890121102ec79ac532df
ms.sourcegitcommit: f6b76df4c22f1c605682418f3f2385131512508d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108325848"
---
# <a name="quickstart-create-a-resource-graph-shared-query-using-azure-powershell"></a>Inicio rápido: Creación de una consulta compartida de Azure Resource Graph mediante Azure PowerShell

En este artículo se describe cómo crear una consulta compartida de Azure Resource Graph mediante el módulo de PowerShell [Az.ResourceGraph](/powershell/module/az.resourcegraph).

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

[!INCLUDE [azure-powershell-requirements-no-header.md](../../../includes/azure-powershell-requirements-no-header.md)]

  > [!IMPORTANT]
  > Mientras el módulo **Az.ResourceGraph** de PowerShell esté en versión preliminar, se debe instalar por separado mediante el cmdlet `Install-Module`.

  ```azurepowershell-interactive
  Install-Module -Name Az.ResourceGraph
  ```

- Si tiene varias suscripciones a Azure, elija la suscripción adecuada en la que se debe facturar el recurso. Seleccione una suscripción específica con el cmdlet [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

  ```azurepowershell-interactive
  Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
  ```

## <a name="create-a-resource-graph-shared-query"></a>Creación de una consulta compartida de Resource Graph

Con el módulo de PowerShell **Az.ResourceGraph** agregado al entorno que prefiera, es el momento de crear una consulta compartida de Resource Graph. Las consultas compartidas son objetos de Azure Resource Manager para los que se puede conceder permiso o que se pueden ejecutar en el Probador de Azure Resource Graph. La consulta resume el recuento de todos los recursos agrupados por _ubicación_.

1. Cree un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) para almacenar la consulta compartida de Azure Resource Graph. Este grupo de recursos se denomina `resource-graph-queries` y la ubicación es `westus2`.

   ```azurepowershell-interactive
   # Login first with `Connect-AzAccount` if not using Cloud Shell

   # Create the resource group
   New-AzResourceGroup -Name resource-graph-queries -Location westus2
   ```

1. Cree la consulta compartida de Azure Resource Graph mediante el módulo de PowerShell **Az.ResourceGraph** y el cmdlet [New-AzResourceGraphQuery](/powershell/module/az.resourcegraph/new-azresourcegraphquery):

   ```azurepowershell-interactive
   # Create the Azure Resource Graph shared query
   $Params = @{
     Name = 'Summarize resources by location'
     ResourceGroupName = 'resource-graph-queries'
     Location = 'westus2'
     Description = 'This shared query summarizes resources by location for a pinnable map graphic.'
     Query = 'Resources | summarize count() by location'
   }
   New-AzResourceGraphQuery @Params
   ```

1. Consulte una lista de las consultas compartidas del nuevo grupo de recursos. El cmdlet [Get-AzResourceGraphQuery](/powershell/module/az.resourcegraph/get-azresourcegraphquery) devuelve una matriz de valores.

   ```azurepowershell-interactive
   # List all the Azure Resource Graph shared queries in a resource group
   Get-AzResourceGraphQuery -ResourceGroupName resource-graph-queries
   ```

1. Para obtener un resultado de una única consulta compartida, utilice `Get-AzResourceGraphQuery` con su parámetro `Name`.

   ```azurepowershell-interactive
   # Show a specific Azure Resource Graph shared query
   Get-AzResourceGraphQuery -ResourceGroupName resource-graph-queries -Name 'Summarize resources by location'
   ```

## <a name="clean-up-resources"></a>Limpiar los recursos

Si quiere quitar la consulta compartida y el grupo de recursos de Resource Graph de su entorno de Azure, puede hacerlo mediante los siguientes comandos:

- [Remove-AzResourceGraphQuery](/powershell/module/az.resourcegraph/remove-azresourcegraphquery)
- [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup)

```azurepowershell-interactive
# Delete the Azure Resource Graph shared query
Remove-AzResourceGraphQuery -ResourceGroupName resource-graph-queries -Name 'Summarize resources by location'

# Remove the resource group
# WARNING: This command deletes ALL resources you've added to this resource group
Remove-AzResourceGroup -Name resource-graph-queries
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio ha creado una consulta compartida de Resource Graph mediante Azure PowerShell. Para más información sobre el lenguaje de Resource Graph, vaya a la página de detalles del lenguaje de consulta.

> [!div class="nextstepaction"]
> [Obtener más información sobre el lenguaje de consulta](./concepts/query-language.md)
