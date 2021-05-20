---
title: Creación e implementación de una especificación de plantilla
description: Obtenga información sobre cómo crear una especificación de plantilla a partir de una plantilla de Resource Manager. A continuación, implemente la especificación de plantilla en un grupo de recursos de su suscripción.
author: tfitzmac
ms.date: 05/04/2021
ms.topic: quickstart
ms.author: tomfitz
ms.openlocfilehash: 0775d58bebaf487d5a536818aec788fabdace909
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108754188"
---
# <a name="quickstart-create-and-deploy-template-spec"></a>Inicio rápido: Creación e implementación de una especificación de plantilla

En este inicio rápido se muestra cómo empaquetar una plantilla de Azure Resource Manager como [especificación de plantilla](template-specs.md). y, posteriormente, implementarla. La especificación de plantilla contiene una plantilla de Resource Manager que implementa una cuenta de almacenamiento.

## <a name="prerequisites"></a>Requisitos previos

Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

> [!NOTE]
> Para usar especificaciones de plantilla con Azure PowerShell, debe instalar la [versión 5.0.0 o posterior](/powershell/azure/install-az-ps). Para usarlas con la CLI de Azure, use la [versión 2.14.2 o posterior](/cli/azure/install-azure-cli).

## <a name="create-template"></a>Creación de una plantilla

Las especificaciones de plantilla se crean a partir de una plantilla local. Copie la siguiente plantilla y guárdela en un archivo local llamado **azuredeploy.json**. En esta guía de inicio rápido se da por supuesto que se ha guardado en la ruta de acceso **c:\Templates\azuredeploy.json**, pero puede usar cualquier ruta de acceso.

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.storage/storage-account-create/azuredeploy.json":::

## <a name="create-template-spec"></a>Crear una especificación de plantilla

La especificación de plantilla es un tipo de recurso llamado `Microsoft.Resources/templateSpecs`. Para crear una especificación de plantilla, use PowerShell, la CLI de Azure, Azure Portal o una plantilla de Resource Manager.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Cree un nuevo grupo de recursos que contiene la especificación de plantilla.

    ```azurepowershell
    New-AzResourceGroup `
      -Name templateSpecRG `
      -Location westus2
    ```

1. Cree la especificación de plantilla en ese grupo de recursos. Asigne a la nueva especificación de plantilla el nombre **storageSpec**.

    ```azurepowershell
    New-AzTemplateSpec `
      -Name storageSpec `
      -Version "1.0" `
      -ResourceGroupName templateSpecRG `
      -Location westus2 `
      -TemplateFile "c:\Templates\azuredeploy.json"
    ```

# <a name="cli"></a>[CLI](#tab/azure-cli)

1. Cree un nuevo grupo de recursos que contiene la especificación de plantilla.

    ```azurecli
    az group create \
      --name templateSpecRG \
      --location westus2
    ```

1. Cree la especificación de plantilla en ese grupo de recursos. Asigne a la nueva especificación de plantilla el nombre **storageSpec**.

    ```azurecli
    az ts create \
      --name storageSpec \
      --version "1.0" \
      --resource-group templateSpecRG \
      --location "westus2" \
      --template-file "c:\Templates\azuredeploy.json"
    ```

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque **especificaciones de plantilla**. Seleccione **Especificaciones de plantilla** entre las opciones disponibles.

    :::image type="content" source="./media/quickstart-create-template-specs/search-template-spec.png" alt-text="Buscar especificaciones de plantilla":::

1. Seleccione **Importar plantilla**.

    :::image type="content" source="./media/quickstart-create-template-specs/import-template.png" alt-text="Importar plantilla":::

1. Seleccione el icono de la carpeta.

    :::image type="content" source="./media/quickstart-create-template-specs/open-folder.png" alt-text="Abrir carpeta":::

1. Vaya a la plantilla local que ha guardado y selecciónela. Seleccione **Open** (Abrir).
1. Seleccione **Import** (Importar).

    :::image type="content" source="./media/quickstart-create-template-specs/select-import.png" alt-text="Seleccione la opción de importación":::

1. Proporcione los valores siguientes:

    - **Nombre**: escriba un nombre para la especificación de plantilla.  Por ejemplo, **storageSpec**.
    - **Suscripción**: seleccione la suscripción de Azure usada para crear la especificación de plantilla.
    - **Grupo de recursos**: seleccione **Crear nuevo** y especifique un nuevo nombre de grupo de recursos.  Por ejemplo, **templateSpecRG**.
    - **Ubicación**: seleccione una ubicación para el grupo de recursos. Por ejemplo, **Oeste de EE. UU. 2**.
    - **Versión**: escriba una versión para la especificación de plantilla. Use **1.0**.

1. Seleccione **Revisar + crear**.
1. Seleccione **Crear**.

# <a name="arm-template"></a>[Plantilla de ARM](#tab/azure-resource-manager)

> [!NOTE]
> Es aconsejable que, en lugar de usar una plantilla de Resource Manager, se use PowerShell o la CLI para crear la especificación de plantilla. Dichas herramientas convierten automáticamente plantillas vinculadas en artefactos conectados a su plantilla principal. Si se usa una plantilla de Resource Manager para crear la especificación de plantilla, las plantillas vinculadas se deben agregar manualmente como artefactos, lo que puede resultar complicado.

1. Cuando se usa una plantilla de Resource Manager para crear la especificación de plantilla, la plantilla se incrusta en la definición de recurso. Es preciso realizar varios cambios en la plantilla local. Copie la siguiente plantilla y guárdela localmente como **azuredeploy.json**.

    > [!NOTE]
    > En la plantilla insertada, todas las [expresiones de plantilla](template-expressions.md) se deben escapar con un segundo corchete de apertura. Use `"[[` en lugar de `"[`. Las matrices JSON siguen usando un solo corchete de apertura.

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {},
      "functions": [],
      "variables": {},
      "resources": [
        {
          "type": "Microsoft.Resources/templateSpecs",
          "apiVersion": "2021-05-01",
          "name": "storageSpec",
          "location": "westus2",
          "properties": {
            "displayName": "Storage template spec"
          },
          "tags": {},
          "resources": [
            {
              "type": "versions",
              "apiVersion": "2021-05-01",
              "name": "1.0",
              "location": "westus2",
              "dependsOn": [ "storageSpec" ],
              "properties": {
                "mainTemplate": {
                  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
                  "contentVersion": "1.0.0.0",
                  "parameters": {
                    "storageAccountType": {
                      "type": "string",
                      "defaultValue": "Standard_LRS",
                      "allowedValues": [
                        "Standard_LRS",
                        "Standard_GRS",
                        "Standard_ZRS",
                        "Premium_LRS"
                      ],
                      "metadata": {
                        "description": "Storage Account type"
                      }
                    },
                    "location": {
                      "type": "string",
                      "defaultValue": "[[resourceGroup().location]",
                      "metadata": {
                        "description": "Location for all resources."
                      }
                    }
                  },
                  "variables": {
                    "storageAccountName": "[[concat('store', uniquestring(resourceGroup().id))]"
                  },
                  "resources": [
                    {
                      "type": "Microsoft.Storage/storageAccounts",
                      "apiVersion": "2019-04-01",
                      "name": "[[variables('storageAccountName')]",
                      "location": "[[parameters('location')]",
                      "sku": {
                        "name": "[[parameters('storageAccountType')]"
                      },
                      "kind": "StorageV2",
                      "properties": {}
                    }
                  ],
                  "outputs": {
                    "storageAccountName": {
                      "type": "string",
                      "value": "[[variables('storageAccountName')]"
                    }
                  }
                }
              },
              "tags": {}
            }
          ]
        }
      ],
      "outputs": {}
    }
    ```

1. Use la CLI de Azure o PowerShell para crear un nuevo grupo de recursos.

    ```azurepowershell
    New-AzResourceGroup `
      -Name templateSpecRG `
      -Location westus2
    ```

    ```azurecli
    az group create \
      --name templateSpecRG \
      --location westus2
    ```

1. Implemente la plantilla con la CLI de Azure o PowerShell.

    ```azurepowershell
    New-AzResourceGroupDeployment `
      -ResourceGroupName templateSpecRG `
      -TemplateFile "c:\Templates\azuredeploy.json"
    ```

    ```azurecli
    az deployment group create \
      --name templateSpecRG \
      --template-file "c:\Templates\azuredeploy.json"
    ```

---

## <a name="deploy-template-spec"></a>Implementación de la especificación de plantilla

Para implementar una especificación de plantilla, utilice los mismos comandos de implementación que para implementar una plantilla. Pase el id. de recurso de la especificación de plantilla para implementarla.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Cree un grupo de recursos que contenga la cuenta de almacenamiento nueva.

    ```azurepowershell
    New-AzResourceGroup `
      -Name storageRG `
      -Location westus2
    ```

1. Obtenga el id. de recurso de la especificación de plantilla.

    ```azurepowershell
    $id = (Get-AzTemplateSpec -ResourceGroupName templateSpecRG -Name storageSpec -Version "1.0").Versions.Id
    ```

1. Implemente la especificación de plantilla.

    ```azurepowershell
    New-AzResourceGroupDeployment `
      -TemplateSpecId $id `
      -ResourceGroupName storageRG
    ```

1. Proporcione los parámetros exactamente como lo haría para una plantilla de ARM. Vuelva a implementar la especificación de plantilla con un parámetro para el tipo de cuenta de almacenamiento.

    ```azurepowershell
    New-AzResourceGroupDeployment `
      -TemplateSpecId $id `
      -ResourceGroupName storageRG `
      -storageAccountType Standard_GRS
    ```

# <a name="cli"></a>[CLI](#tab/azure-cli)

1. Cree un grupo de recursos que contenga la cuenta de almacenamiento nueva.

    ```azurecli
    az group create \
      --name storageRG \
      --location westus2
    ```

1. Obtenga el id. de recurso de la especificación de plantilla.

    ```azurecli
    id=$(az ts show --name storageSpec --resource-group templateSpecRG --version "1.0" --query "id")
    ```

    > [!NOTE]
    > Existe un problema conocido al obtener el identificador de la especificación de plantilla y luego asignárselo a una variable en Windows PowerShell.

1. Implemente la especificación de plantilla.

    ```azurecli
    az deployment group create \
      --resource-group storageRG \
      --template-spec $id
    ```

1. Proporcione los parámetros exactamente como lo haría para una plantilla de ARM. Vuelva a implementar la especificación de plantilla con un parámetro para el tipo de cuenta de almacenamiento.

    ```azurecli
    az deployment group create \
      --resource-group storageRG \
      --template-spec $id \
      --parameters storageAccountType='Standard_GRS'
    ```

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. Seleccione la especificación de plantilla que creó.

    :::image type="content" source="./media/quickstart-create-template-specs/select-template-spec.png" alt-text="Seleccionar especificaciones de plantilla":::

1. Seleccione **Implementar**.

    :::image type="content" source="./media/quickstart-create-template-specs/deploy-template-spec.png" alt-text="Implementar especificaciones de plantilla":::

1. Proporcione los valores siguientes:

    - **Suscripción**: seleccione una suscripción de Azure para crear el recurso.
    - **Grupo de recursos**: seleccione **Crear nuevo** y, luego, escriba **storageRG**.
    - **Tipo de cuenta de almacenamiento**: seleccione **Standard_GRS**.

1. Seleccione **Revisar + crear**.
1. Seleccione **Crear**.

# <a name="arm-template"></a>[Plantilla de ARM](#tab/azure-resource-manager)

1. Copie la siguiente plantilla y guárdela en un archivo local llamado **storage.json**.

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {},
      "functions": [],
      "variables": {},
      "resources": [
        {
          "type": "Microsoft.Resources/deployments",
          "apiVersion": "2020-10-01",
          "name": "demo",
          "properties": {
            "templateLink": {
              "id": "[resourceId('templateSpecRG', 'Microsoft.Resources/templateSpecs/versions', 'storageSpec', '1.0')]"
            },
            "parameters": {
            },
            "mode": "Incremental"
          }
        }
      ],
      "outputs": {}
    }
    ```

1. Use la CLI de Azure o PowerShell para crear un nuevo grupo de recursos para la cuenta de almacenamiento.

    ```azurepowershell
    New-AzResourceGroup `
      -Name storageRG `
      -Location westus2
    ```

    ```azurecli
    az group create \
      --name storageRG \
      --location westus2
    ```

1. Implemente la plantilla con la CLI de Azure o PowerShell.

    ```azurepowershell
    New-AzResourceGroupDeployment `
      -ResourceGroupName storageRG `
      -TemplateFile "c:\Templates\storage.json"
    ```

    ```azurecli
    az deployment group create \
      --name storageRG \
      --template-file "c:\Templates\storage.json"
    ```

---

## <a name="grant-access"></a>Conceder acceso

Si quiere permitir que otros usuarios de su organización implementen la especificación de plantilla, debe concederles acceso de lectura. Puede asignar el rol Lector a un grupo de Azure AD para el grupo de recursos que contiene las especificaciones de plantilla que quiere compartir. Para más información, consulte el [Tutorial: Concesión de acceso de grupo a recursos de Azure mediante Azure PowerShell](../../role-based-access-control/tutorial-role-assignments-group-powershell.md).

## <a name="update-template"></a>Actualización de la plantilla

Supongamos que ha identificado un cambio que desea hacer en la plantilla en la especificación de plantilla. La siguiente plantilla es similar a la anterior, salvo que agrega un prefijo para el nombre de la cuenta de almacenamiento. Copie la siguiente plantilla y actualice el archivo azuredeploy.json.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/templatespecs/azuredeploy2.json":::

## <a name="update-template-spec-version"></a>Actualización de la versión de la especificación de plantilla

En lugar de crear una nueva especificación de plantilla para la plantilla revisada, agregue una nueva versión denominada `2.0` a la especificación de plantilla existente. Los usuarios pueden la versión que desean implementar.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Cree una versión de la especificación de plantilla.

   ```powershell
   New-AzTemplateSpec `
     -Name storageSpec `
     -Version "2.0" `
     -ResourceGroupName templateSpecRG `
     -Location westus2 `
     -TemplateFile "c:\Templates\azuredeploy.json"
   ```

1. Para implementar la nueva versión, obtenga el identificador de recurso de la versión `2.0`.

   ```powershell
   $id = (Get-AzTemplateSpec -ResourceGroupName templateSpecRG -Name storageSpec -Version "2.0").Versions.Id
   ```

1. Implemente esa versión. Proporcione un prefijo para el nombre de la cuenta de almacenamiento.

   ```powershell
   New-AzResourceGroupDeployment `
     -TemplateSpecId $id `
     -ResourceGroupName storageRG `
     -namePrefix "demoaccount"
   ```

# <a name="cli"></a>[CLI](#tab/azure-cli)

1. Cree una versión de la especificación de plantilla.

   ```azurecli
   az ts create \
     --name storageSpec \
     --version "2.0" \
     --resource-group templateSpecRG \
     --location "westus2" \
     --template-file "c:\Templates\azuredeploy.json"
   ```

1. Para implementar la nueva versión, obtenga el identificador de recurso de la versión `2.0`.

   ```azurecli
   id=$(az ts show --name storageSpec --resource-group templateSpecRG --version "2.0" --query "id")
   ```

1. Implemente esa versión. Proporcione un prefijo para el nombre de la cuenta de almacenamiento.

   ```azurecli
    az deployment group create \
      --resource-group storageRG \
      --template-spec $id \
      --parameters namePrefix='demoaccount'
    ```

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. En la especificación de plantilla, seleccione **Crear versión nueva**.

   :::image type="content" source="./media/quickstart-create-template-specs/select-versions.png" alt-text="Crear una versión":::

1. Asigne a la nueva versión el nombre `2.0` y, si lo desea, agregue notas. Seleccione **Editar plantilla**.

   :::image type="content" source="./media/quickstart-create-template-specs/add-version-name.png" alt-text="asignar un nombre a la versión nueva":::

1. Reemplace el contenido de la plantilla por la plantilla actualizada. Seleccione **Revisar y guardar**.
1. Seleccione **Save changes** (Guardar los cambios).

1. Para implementar la nueva versión, seleccione **Versiones**

   :::image type="content" source="./media/quickstart-create-template-specs/see-versions.png" alt-text="Enumerar versiones":::

1. Para la versión que desea implementar, seleccione los tres puntos e **Implementar**.

   :::image type="content" source="./media/quickstart-create-template-specs/deploy-version.png" alt-text="Seleccionar la versión que implementar":::

1. Rellene los campos como hizo al implementar la versión anterior.
1. Seleccione **Revisar + crear**.
1. Seleccione **Crear**.

# <a name="arm-template"></a>[Plantilla de ARM](#tab/azure-resource-manager)

1. Una vez más, debe realizar algunos campos en la plantilla local para que funcione con las especificaciones de plantilla. Copie la siguiente plantilla y guárdela localmente como azuredeploy.json.

   ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {},
      "functions": [],
      "variables": {},
      "resources": [
        {
          "type": "Microsoft.Resources/templateSpecs",
          "apiVersion": "2021-05-01",
          "name": "storageSpec",
          "location": "westus2",
          "properties": {
            "displayName": "Storage template spec"
          },
          "tags": {},
          "resources": [
            {
              "type": "versions",
              "apiVersion": "2021-05-01",
              "name": "2.0",
              "location": "westus2",
              "dependsOn": [ "storageSpec" ],
              "properties": {
                "mainTemplate": {
                  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
                  "contentVersion": "1.0.0.0",
                  "parameters": {
                    "storageAccountType": {
                      "type": "string",
                      "defaultValue": "Standard_LRS",
                      "allowedValues": [
                        "Standard_LRS",
                        "Standard_GRS",
                        "Standard_ZRS",
                        "Premium_LRS"
                      ],
                      "metadata": {
                        "description": "Storage Account type"
                      }
                    },
                    "location": {
                      "type": "string",
                      "defaultValue": "[[resourceGroup().location]",
                      "metadata": {
                        "description": "Location for all resources."
                      }
                    },
                    "namePrefix": {
                      "type": "string",
                      "maxLength": 11,
                      "defaultValue": "store",
                      "metadata": {
                        "description": "Prefix for storage account name"
                      }
                    }
                  },
                  "variables": {
                    "storageAccountName": "[[concat(parameters('namePrefix'), uniquestring(resourceGroup().id))]"
                  },
                  "resources": [
                    {
                      "type": "Microsoft.Storage/storageAccounts",
                      "apiVersion": "2019-04-01",
                      "name": "[[variables('storageAccountName')]",
                      "location": "[[parameters('location')]",
                      "sku": {
                        "name": "[[parameters('storageAccountType')]"
                      },
                      "kind": "StorageV2",
                      "properties": {}
                    }
                  ],
                  "outputs": {
                    "storageAccountName": {
                      "type": "string",
                      "value": "[[variables('storageAccountName')]"
                    }
                  }
                }
              },
              "tags": {}
            }
          ]
        }
      ],
      "outputs": {}
    }
    ```

1. Para agregar la nueva versión a la especificación de plantilla, implemente la plantilla con la CLI de Azure o PowerShell.

    ```azurepowershell
    New-AzResourceGroupDeployment `
      -ResourceGroupName templateSpecRG `
      -TemplateFile "c:\Templates\azuredeploy.json"
    ```

    ```azurecli
    az deployment group create \
      --name templateSpecRG \
      --template-file "c:\Templates\azuredeploy.json"
    ```

1. Copie la siguiente plantilla y guárdela en un archivo local llamado **storage.json**.

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {},
      "functions": [],
      "variables": {},
      "resources": [
        {
          "type": "Microsoft.Resources/deployments",
          "apiVersion": "2020-10-01",
          "name": "demo",
          "properties": {
            "templateLink": {
              "id": "[resourceId('templateSpecRG', 'Microsoft.Resources/templateSpecs/versions', 'storageSpec', '2.0')]"
            },
            "parameters": {
            },
            "mode": "Incremental"
          }
        }
      ],
      "outputs": {}
    }
    ```

1. Implemente la plantilla con la CLI de Azure o PowerShell.

    ```azurepowershell
    New-AzResourceGroupDeployment `
      -ResourceGroupName storageRG `
      -TemplateFile "c:\Templates\storage.json"
    ```

    ```azurecli
    az deployment group create \
      --name storageRG \
      --template-file "c:\Templates\storage.json"
    ```

---

## <a name="clean-up-resources"></a>Limpieza de recursos

Para limpiar el recurso que implementó en esta guía de inicio rápido, elimine los dos grupos de recursos que creó.

1. En Azure Portal, seleccione el grupo de recursos en el menú de la izquierda.

1. Escriba el nombre del grupo de recursos (templateSpecRG y storageRG) en el campo Filtrar por nombre.

1. Seleccione el nombre del grupo de recursos.

1. Seleccione Eliminar grupo de recursos en el menú superior.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo crear una especificación de plantilla que incluya plantillas vinculadas, consulte [Creación de una especificación de plantilla con plantillas vinculadas](template-specs-create-linked.md).
