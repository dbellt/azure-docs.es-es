---
title: 'Tutorial: Publicación de Azure Static Web Apps con una plantilla de ARM'
description: Creación e implementación de una plantilla de ARM para Static Web Apps
services: static-web-apps
author: petender
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 05/10/2021
ms.author: petender
ms.openlocfilehash: 4d79e417e637894f2a41bd55b52fbbe55300a694
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109685412"
---
# <a name="tutorial-publish-azure-static-web-apps-using-an-arm-template"></a>Tutorial: Publicación de Azure Static Web Apps con una plantilla de ARM

En este artículo se describe cómo implementar [Azure Static Web Apps](./overview.md) con una [plantilla de Azure Resource Manager](../azure-resource-manager/templates/overview.md) (plantilla de ARM).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> - Creación de una plantilla de ARM para Azure Static Web Apps
> - Implementación de la plantilla de ARM para crear una instancia de Azure Static Web App

## <a name="prerequisites"></a>Prerrequisitos

- Una **cuenta activa de Azure**: en caso de no tener una, puede [crear una cuenta gratuita](https://azure.microsoft.com/free/).
- Una **cuenta GitHub**: en caso de no tener una, puede [crear una cuenta de GitHub gratuita](https://github.com).
- Un **editor de plantillas de ARM**: la revisión y edición de plantillas requiere un editor JSON. Visual Studio Code con la [extensión de herramientas de Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools) es adecuado para editar plantillas de ARM. Para obtener instrucciones sobre cómo instalar y configurar Visual Studio Code, consulte [Inicio rápido: Creación de plantillas de ARM mediante Visual Studio Code](../azure-resource-manager/templates/quickstart-create-templates-use-visual-studio-code.md).

- **CLI de Azure o Azure PowerShell**: la implementación de plantillas de ARM requiere una herramienta de línea de comandos. Para obtener instrucciones de instalación, consulte:
  - [Instalación de la CLI de Azure en Windows](https://docs.microsoft.com/cli/azure/install-azure-cli-windows)
  - [Instalación de la CLI de Azure en Linux](https://docs.microsoft.com/cli/azure/install-azure-cli-linux)
  - [Instalación de la CLI de Azure en macOS](https://docs.microsoft.com/cli/azure/install-azure-cli-macos)
  - [Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps)

## <a name="create-a-github-personal-access-token"></a>Creación de un token de acceso personal de GitHub

Uno de los parámetros requeridos en la plantilla de ARM es `repositoryToken`, que permite que el proceso de implementación de ARM interactúe con el repositorio de GitHub que incluye el código fuente del sitio estático. 

1. En el perfil de la cuenta de GitHub (en la esquina superior derecha), seleccione **Configuración**.

1. Seleccione **Configuración del desarrollador**.

1. Seleccione **Tokens de acceso personal**.

1. Seleccione **Generar nuevo token**.

1. Proporcione un nombre para este token en el campo _Nota_, por ejemplo *myfirstswadeployment*.

1. Especifique los siguientes *ámbitos*: **repo, workflow, write:packages**

1. Seleccione **Generar token**.

1. Copie el valor del token y péguelo en un editor de texto para usarlo posteriormente.

> [!IMPORTANT]
> Asegúrese de copiar este token y guardarlo en un lugar seguro. Considere la posibilidad de almacenar este token en [Azure KeyVault](../azure-resource-manager/templates/template-tutorial-use-key-vault.md) y acceder a él en la plantilla de ARM.

## <a name="create-a-github-repo"></a>Creación de un repositorio de GitHub

En este artículo se usa un repositorio de plantillas de GitHub para facilitar los primeros pasos. La plantilla incluye una aplicación de inicio que se usa para realizar la implementación con Azure Static Web Apps.

1. Vaya a la siguiente ubicación para crear un repositorio:
    1. [https://github.com/staticwebdev/vanilla-basic/generate](https://github.com/login?return_to=/staticwebdev/vanilla-basic/generate)

1. Asigne al repositorio el nombre **myfirstswadeployment**.

    > [!NOTE]
    > Azure Static Web Apps requiere al menos un archivo HTML para crear una aplicación web. El repositorio que se crea en este paso incluye un solo archivo _index.html_.

1. Seleccione **Create repository from template** (Crear repositorio a partir de plantilla).

    :::image type="content" source="./media/getting-started/create-template.png" alt-text="Crear repositorio a partir de plantilla":::

## <a name="create-the-arm-template"></a>Creación de la plantilla de ARM

Una vez cumplidos los requisitos previos, el siguiente paso es definir el archivo de plantilla de implementación de ARM.

1. Cree una nueva carpeta para contener las plantillas de ARM.

1. Cree un nuevo archivo y asígnele el nombre **azuredeploy.json**.

1. Pegue el siguiente fragmento de plantilla de ARM en _azuredeploy.jsen_.

    ```json
        {
            "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
                "name": {
                    "type": "string"
                },
                "location": {
                    "type": "string"
                },
                "sku": {
                    "type": "string"
                },
                "skucode": {
                    "type": "string"
                },
                "repositoryUrl": {
                    "type": "string"
                },
                "branch": {
                    "type": "string"
                },
                "repositoryToken": {
                    "type": "securestring"
                },
                "appLocation": {
                    "type": "string"
                },
                "apiLocation": {
                    "type": "string"
                },
                "appArtifactLocation": {
                    "type": "string"
                },
                "resourceTags": {
                    "type": "object"
                }
            },
            "resources": [
                {
                    "apiVersion": "2019-12-01-preview",
                    "name": "[parameters('name')]",
                    "type": "Microsoft.Web/staticSites",
                    "location": "[parameters('location')]",
                    "tags": "[parameters('resourceTags')]",
                    "properties": {
                        "repositoryUrl": "[parameters('repositoryUrl')]",
                        "branch": "[parameters('branch')]",
                        "repositoryToken": "[parameters('repositoryToken')]",
                        "buildProperties": {
                            "appLocation": "[parameters('appLocation')]",
                            "apiLocation": "[parameters('apiLocation')]",
                            "appArtifactLocation": "[parameters('appArtifactLocation')]"
                        }
                    },
                    "sku": {
                        "Tier": "[parameters('sku')]",
                        "Name": "[parameters('skuCode')]"
                    }
                }
            ]
        }

    ```

1. Cree un nuevo archivo y asígnele el nombre **azuredeploy.parameters.json**.

1. Pegue el siguiente fragmento de plantilla de ARM en _azuredeploy.parameters.json_.

    ```json
        {
            "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
                "name": {
                    "value": "myfirstswadeployment"
                },
                "location": {
                "type": "string",
                "defaultValue": "Central US"
                },   
                "sku": {
                    "value": "Free"
                },
                "skucode": {
                    "value": "Free"
                },
                "repositoryUrl": {
                    "value": "https://github.com/<YOUR-GITHUB-USER-NAME>/<YOUR-GITHUB-REPOSITORY-NAME>"
                },
                "branch": {
                    "value": "main"
                },
                "repositoryToken": {
                    "value": "<YOUR-GITHUB-PAT>" 
                },
                "appLocation": {
                    "value": "/"
                },
                "apiLocation": {
                    "value": ""
                },
                "appArtifactLocation": {
                    "value": "public"
                },
                "resourceTags": {
                    "value": {
                        "Environment": "Development",
                        "Project": "Testing SWA with ARM",
                        "ApplicationName": "myfirstswadeployment"
                    }
                }
            }
        }
    ```

1. Actualice los siguientes parámetros.

    | Parámetro | Valor esperado |
    | --- | --- |
    | `repositoryUrl` | Proporcione la dirección URL al repositorio de GitHub de Static Web Apps. |
    | `repositoryToken` | Proporcione el token de acceso personal de GitHub. |

1. Guarde las actualizaciones antes de ejecutar la implementación en el paso siguiente.

## <a name="running-the-deployment"></a>Ejecución de la implementación

Necesita Azure PowerShell o la CLI de Azure para implementar la plantilla.

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Para implementar una plantilla, inicie sesión en la CLI de Azure o Azure PowerShell.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az login
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Connect-AzAccount
```

---

Si tiene varias suscripciones de Azure, seleccione la suscripción que desee usar. Reemplace `<SUBSCRIPTION-ID-OR-SUBSCRIPTION-NAME>` por la información de su suscripción:

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az account set --subscription <SUBSCRIPTION-ID-OR-SUBSCRIPTION-NAME>
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Set-AzContext <SUBSCRIPTION-ID-OR-SUBSCRIPTION-NAME>
```

---

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Al implementar una plantilla, especificará un grupo de recursos que contiene los recursos relacionados. Antes de ejecutar el comando de implementación, cree el grupo de recursos con la CLI de Azure o Azure PowerShell.

> [!NOTE]
> Los ejemplos de la CLI de este artículo están escritos para el shell de Bash.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
resourceGroupName="myfirstswadeployRG"

az group create \
  --name $resourceGroupName \
  --location "Central US"
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$resourceGroupName = "myfirstswadeployRG"

New-AzResourceGroup `
  -Name $resourceGroupName `
  -Location "Central US"
```

---

## <a name="deploy-template"></a>Implementar plantilla

Use una de estas opciones de implementación para implementar la plantilla.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli

az deployment group create \
  --name DeployLocalTemplate \
  --resource-group $resourceGroupName \
  --template-file <PATH-TO-AZUREDEPLOY.JSON> \
  --parameters <PATH-TO-AZUREDEPLOY.PARAMETERS.JSON> \
  --verbose
```

Para más información sobre la implementación de plantillas con la CLI de Azure, consulte [Implementación de recursos con plantillas de Resource Manager y la CLI de Azure](../azure-resource-manager/templates/deploy-cli.md).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$templateFile = Read-Host -Prompt "Enter the template file path and file name"
$templateparameterFile = Read-Host -Prompt "Enter the template parameter file path and file name"

New-AzResourceGroupDeployment `
  -Name DeployLocalTemplate `
  -ResourceGroupName $resourceGroupName `
  -TemplateFile $templateFile `
  -TemplateParameterFile $templateparameterfile `
  -verbose
```

Para más información sobre la implementación de plantillas con Azure PowerShell, consulte [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md).

---

[!INCLUDE [view website](../../includes/static-web-apps-get-started-view-website.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

Limpie los recursos que implementó eliminando el grupo de recursos.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Configuración de una aplicación web estática](./configuration.md)
