---
title: Implementación de archivos de Bicep con Cloud Shell
description: Use Azure Resource Manager y Azure Cloud Shell para implementar recursos en Azure. Los recursos se definen en un archivo de Bicep.
author: mumian
ms.author: jgao
ms.topic: conceptual
ms.date: 06/01/2021
ms.openlocfilehash: fa74250a290cc3e254e5c9f3757bb3e712c020da
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "111026845"
---
# <a name="deploy-bicep-files-from-azure-cloud-shell"></a>Implementación de archivos de Bicep desde Azure Cloud Shell

Puede usar [Azure Cloud Shell](../../cloud-shell/overview.md) para implementar un archivo de Bicep. Actualmente solo puede implementar un archivo de Bicep local desde Cloud Shell.

Asimismo, puede realizar la implementación en cualquier ámbito. En este artículo se muestra la implementación en un grupo de recursos.

## <a name="deploy-local-bicep-file"></a>Implementación de un archivo de Bicep local

Para implementar un archivo de Bicep local, primero debe cargarlo en la sesión de Cloud Shell local.

1. Inicie sesión en [Cloud Shell](https://shell.azure.com).
1. Seleccione **PowerShell** o **Bash**.

    :::image type="content" source="./media/deploy-cloud-shell/bicep-cloud-shell-bash-powershell.png" alt-text="Selección de Bash o PowerShell":::

1. Seleccione **Cargar/Descargar archivos** y, después, seleccione **Cargar**.

    :::image type="content" source="./media/deploy-cloud-shell/bicep-cloud-shell-upload.png" alt-text="Carga de un archivo":::.

1. Seleccione el archivo de Bicep que desea cargar y luego seleccione **Abrir**.
1. Para implementar el archivo de Bicep, use los comandos siguientes:

   # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

   ```azurecli-interactive
   az group create --name ExampleGroup --location "South Central US"
   az deployment group create \
     --resource-group ExampleGroup \
     --template-file azuredeploy.bicep \
     --parameters storageAccountType=Standard_GRS
   ```

   # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

   ```azurepowershell-interactive
   New-AzResourceGroup -Name ExampleGroup -Location "Central US"
   New-AzResourceGroupDeployment `
     -DeploymentName ExampleDeployment `
     -ResourceGroupName ExampleGroup `
     -TemplateFile azuredeploy.bicep `
     -storageAccountType Standard_GRS
   ```

   ---

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre los comandos de implementación, consulte [Implementación de recursos con Bicep y la CLI de Azure](deploy-cli.md) e [Implementación de recursos con Bicep y Azure PowerShell](deploy-powershell.md).
- Para obtener una vista previa de los cambios antes de implementar un archivo de Bicep, vea [Operación what-if de implementación de Bicep](./deploy-what-if.md).
