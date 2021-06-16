---
title: Implementación de plantillas con Cloud Shell
description: Use Azure Resource Manager y Azure Cloud Shell para implementar recursos en Azure. Los recursos se definen en una plantilla de Azure Resource Manager (plantilla de ARM).
ms.topic: conceptual
ms.date: 10/22/2020
ms.openlocfilehash: bd597c2cb29120e680493a99aac33f146940bc19
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111957881"
---
# <a name="deploy-arm-templates-from-azure-cloud-shell"></a>Implementación de plantillas de ARM desde Azure Cloud Shell

Puede usar [Azure Cloud Shell](../../cloud-shell/overview.md) para implementar una plantilla de Azure Resource Manager (plantilla de Resource Manager). Puede implementar una plantilla de Resource Manager que se almacena de forma remota o una plantilla de Resource Manager que se almacena en la cuenta de almacenamiento local de Cloud Shell.

Asimismo, puede realizar la implementación en cualquier ámbito. En este artículo se muestra la implementación en un grupo de recursos.

## <a name="deploy-remote-template"></a>Implementación de una plantilla remota

Para implementar una plantilla externa, proporcione el identificador URI de la plantilla exactamente como lo haría para cualquier implementación externa. La plantilla externa puede estar en un repositorio de GitHub o en una cuenta de almacenamiento externa.

1. Abra la instancia de Cloud Shell.

   :::image type="content" source="./media/deploy-cloud-shell/open-cloud-shell.png" alt-text="Abrir Cloud Shell":::.

1. Para implementar la plantilla, use los comandos siguientes:

   # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

   ```azurecli-interactive
   az group create --name ExampleGroup --location "Central US"
   az deployment group create \
     --name ExampleDeployment \
     --resource-group ExampleGroup \
     --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.storage/storage-account-create/azuredeploy.json" \
     --parameters storageAccountType=Standard_GRS
   ```

   # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

   ```azurepowershell-interactive
   New-AzResourceGroup -Name ExampleGroup -Location "Central US"
   New-AzResourceGroupDeployment `
     -DeploymentName ExampleDeployment `
     -ResourceGroupName ExampleGroup `
     -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.storage/storage-account-create/azuredeploy.json `
     -storageAccountType Standard_GRS
   ```

   ---

## <a name="deploy-local-template"></a>Implementar una plantilla local

Para implementar una plantilla local, primero debe cargarla en la cuenta de almacenamiento que está conectada a la sesión de Cloud Shell.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Seleccione el grupo de recursos de Cloud Shell. El patrón de nombre es `cloud-shell-storage-<region>`.

   ![Selección de un grupo de recursos](./media/deploy-cloud-shell/select-cloud-shell-resource-group.png)

1. Seleccione la cuenta de almacenamiento de Cloud Shell.

   :::image type="content" source="./media/deploy-cloud-shell/cloud-shell-storage.png" alt-text="Seleccionar cuenta de almacenamiento":::

1. Seleccione **Recursos compartidos de archivos**.

   :::image type="content" source="./media/deploy-cloud-shell/files-shares.png" alt-text="Seleccionar Recursos compartidos de archivos":::

1. Seleccione el recurso compartido de archivos predeterminado para Cloud Shell. El recurso compartido de archivos tiene el formato de nombre `cs-<user>-<domain>-com-<uniqueGuid>`.

   :::image type="content" source="./media/deploy-cloud-shell/select-file-share.png" alt-text="Recurso compartido de archivos predeterminado":::

1. Agregue un nuevo directorio para almacenar las plantillas. Seleccione ese directorio.

   :::image type="content" source="./media/deploy-cloud-shell/add-directory.png" alt-text="Agregar directorio":::

1. Seleccione **Cargar**.

   :::image type="content" source="./media/deploy-cloud-shell/upload-template.png" alt-text="Cargar plantilla":::

1. Busque y cargue la plantilla.

   :::image type="content" source="./media/deploy-cloud-shell/select-template.png" alt-text="Seleccionar plantilla":::

1. Abra la instancia de Cloud Shell.

   :::image type="content" source="./media/deploy-cloud-shell/open-cloud-shell.png" alt-text="Abrir Cloud Shell":::.

1. Vaya al directorio **clouddrive**. Vaya al directorio que agregó para guardar las plantillas.

1. Para implementar la plantilla, use los comandos siguientes:

   # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

   ```azurecli-interactive
   az group create --name ExampleGroup --location "South Central US"
   az deployment group create \
     --resource-group ExampleGroup \
     --template-file azuredeploy.json \
     --parameters storageAccountType=Standard_GRS
   ```

   # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

   ```azurepowershell-interactive
   New-AzResourceGroup -Name ExampleGroup -Location "Central US"
   New-AzResourceGroupDeployment `
     -DeploymentName ExampleDeployment `
     -ResourceGroupName ExampleGroup `
     -TemplateFile azuredeploy.json `
     -storageAccountType Standard_GRS
   ```

   ---

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre los comandos de implementación, consulte [Implementación de recursos con plantillas de Resource Manager y la CLI de Azure](deploy-cli.md) e [Implementación de recursos con plantillas de Resource Manager y Azure PowerShell](deploy-powershell.md).
- Para obtener una vista previa de los cambios antes de implementar una plantilla, consulte [Operación de hipotética de implementación de una plantilla de Resource Manager](./deploy-what-if.md).