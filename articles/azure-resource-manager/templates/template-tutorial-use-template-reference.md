---
title: Uso de referencia de plantilla
description: Use la referencia de plantilla de Azure Resource Manager (ARM) para crear una plantilla.
author: mumian
ms.date: 04/23/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: seodec18
ms.openlocfilehash: 9e5e64358a248f1796f30aafadc65a7d3b4bdbde
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112027108"
---
# <a name="tutorial-utilize-the-arm-template-reference"></a>Tutorial: Utilización de la referencia de la plantilla de Resource Manager

Aprenda cómo encontrar la información del esquema de la plantilla y a usarla para crear plantillas de Azure Resource Manager (ARM).

En este tutorial se usa una plantilla base de las plantillas de inicio rápido de Azure. Use la documentación de referencia de la plantilla para personalizar la plantilla.

![Cuenta de almacenamiento de la implementación de referencia de plantilla de Resource Manager](./media/template-tutorial-use-template-reference/resource-manager-template-tutorial-deploy-storage-account.png)

En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
> * Abra una plantilla de inicio rápido.
> * Descripción de la plantilla
> * Búsqueda de la referencia de la plantilla
> * Edición de la plantilla
> * Implementación de la plantilla

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar este artículo, necesitará lo siguiente:

* Visual Studio Code con la extensión Resource Manager Tools. Consulte [Quickstart: Creación de plantillas de ARM mediante Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).

## <a name="open-a-quickstart-template"></a>Abra una plantilla de inicio rápido.

[Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/) es un repositorio de plantillas de Azure Resource Manager. En lugar de crear una plantilla desde cero, puede buscar una plantilla de ejemplo y personalizarla. La plantilla usada en esta guía de inicio rápido se denomina [Crear una cuenta de almacenamiento estándar](https://azure.microsoft.com/resources/templates/storage-account-create/). La plantilla define un recurso de la cuenta de almacenamiento de Azure.

1. En Visual Studio Code, seleccione **Archivo** > **Abrir archivo**.
1. En **Nombre de archivo**, pegue el código URL siguiente:

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.storage/storage-account-create/azuredeploy.json
    ```

1. Seleccione **Abrir** para abrir el archivo.
1. Seleccione **Archivo** > **Guardar como** para guardar el archivo como _azuredeploy.json_ en el equipo local.

## <a name="understand-the-schema"></a>Información sobre el esquema

1. En Visual Studio Code, contraiga la plantilla al nivel raíz. Tiene la estructura más sencilla con los siguientes elementos:

    ![La estructura más sencilla de la plantilla de Resource Manager](./media/template-tutorial-use-template-reference/resource-manager-template-simplest-structure.png)

    * `$schema`: especifique la ubicación del archivo de esquema JSON que describe la versión del lenguaje de plantilla.
    * `contentVersion`: especifique cualquier valor para este elemento para documentar cambios importantes en la plantilla.
    * `parameters`: especifique los valores que se proporcionan cuando se ejecuta la implementación para personalizar la implementación de recursos.
    * `variables`: especifique los valores que se usan como fragmentos JSON en la plantilla para simplificar las expresiones de lenguaje de plantilla.
    * `resources`: especifique los tipos de recursos que se implementan o actualizan en un grupo de recursos.
    * `outputs`: especifique los valores que se devuelven después de la implementación.

1. Expanda `resources`. Hay un recurso `Microsoft.Storage/storageAccounts` definido. El nombre de SKU usa un valor de parámetro. El parámetro se denomina `storageAccountType`.

    ![Definición de cuenta de almacenamiento de la plantilla de Resource Manager](./media/template-tutorial-use-template-reference/resource-manager-template-storage-resource.png)

1. Expanda `parameters` para ver cómo se define `storageAccountType`. El parámetro tiene cuatro valores permitidos. Encontrará los demás valores permitidos. A continuación, revise la definición del parámetro.

    ![SKU de recursos la cuenta de almacenamiento de la plantilla de Resource Manager](./media/template-tutorial-use-template-reference/resource-manager-template-storage-resources-skus-old.png)

## <a name="find-the-template-reference"></a>Búsqueda de la referencia de la plantilla

1. Vaya a la [referencia sobre las plantillas de Azure](/azure/templates/).
1. En el cuadro **Filter by title** (Filtrar por título), escriba **storage accounts** (cuentas de almacenamiento) y seleccione la primera **cuenta de almacenamiento** en **Reference > Storage** (Referencia > Almacenamiento).

    ![Cuenta de almacenamiento de referencia de plantilla de Resource Manager](./media/template-tutorial-use-template-reference/resource-manager-template-resources-reference-storage-accounts.png)

    Lo habitual es que un proveedor de recursos tenga varias versiones de la API:

    ![Versiones de la cuenta de almacenamiento de la referencia de la plantilla de Resource Manager](./media/template-tutorial-use-template-reference/resource-manager-template-resources-reference-storage-accounts-versions.png)

1. Seleccione **Todos los recursos** en **Almacenamiento** en el panel izquierdo. En esta página se enumeran los tipos de recursos y las versiones del proveedor de recursos de almacenamiento. Se recomienda usar las versiones más recientes de las API para los tipos de recursos definidos en la plantilla.

    ![Versiones de los tipos de cuenta de almacenamiento de referencia de la plantilla de Resource Manager](./media/template-tutorial-use-template-reference/resource-manager-template-resources-reference-storage-accounts-types-versions.png)

1. Seleccione la versión más reciente del tipo de recurso `storageAccount`. En el momento en que se ha escrito el artículo, la versión más reciente es la **2019-06-01**. Asegúrese de que esta versión coincide con la versión usada para el recurso de la cuenta de almacenamiento de la plantilla. Si actualiza la versión de la API, compruebe que la definición de recursos coincide con la referencia de la plantilla.

1. En esta página se enumeran los detalles del tipo de recurso storageAccount. Por ejemplo, se enumeran los valores permitidos para el objeto **Sku**. Hay más SKU de las que aparecen enumeradas en la plantilla de inicio rápido que abrió antes. La plantilla de inicio rápido se puede personalizar para incluir todos los tipos de almacenamiento disponibles.

    ![SKU de la cuenta de almacenamiento de referencia de la plantilla de Resource Manager](./media/template-tutorial-use-template-reference/resource-manager-template-resources-reference-storage-accounts-skus.png)

## <a name="edit-the-template"></a>Edición de la plantilla

En Visual Studio Code, agregue los tipos de cuenta de almacenamiento adicionales como se muestra en la siguiente captura:

![Recursos de la cuenta de almacenamiento de la plantilla de Resource Manager](./media/template-tutorial-use-template-reference/resource-manager-template-storage-resources-skus.png)

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Inicio de sesión en [Azure Cloud Shell](https://shell.azure.com)

1. Elija el entorno que prefiera; para ello, seleccione **PowerShell** o **Bash** (para CLI) en la esquina superior izquierda.  Es necesario reiniciar el shell cuando realiza el cambio.

    ![Archivo de carga de Cloud Shell de Azure Portal](./media/template-tutorial-use-template-reference/azure-portal-cloud-shell-upload-file.png)

1. Seleccione **Cargar/descargar archivos** y, después, seleccione **Cargar**. Consulte la captura de pantalla anterior. Seleccione el archivo que guardó en la sección anterior. Después de cargar el archivo, puede usar el comando `ls` y el comando `cat` para comprobar que la operación de carga se ha realizado correctamente.

1. En Cloud Shell, ejecute los comandos siguientes. Seleccione la pestaña para mostrar el código de PowerShell o el código de la CLI.

   Al implementar la plantilla, especifique el parámetro `storageAccountType` con un valor recién agregado, por ejemplo, **Standard_RAGRS**. Si se usara la plantilla de inicio rápido original, la implementación no se realizaría, ya que **Standard_RAGRS** no era un valor permitido.

    # <a name="cli"></a>[CLI](#tab/CLI)

    ```azurecli
    echo "Enter a project name that is used to generate resource group name:" &&
    read projectName &&
    echo "Enter the location (i.e. centralus):" &&
    read location &&
    resourceGroupName="${projectName}rg" &&
    az group create --name $resourceGroupName --location "$location" &&
    az deployment group create --resource-group $resourceGroupName --template-file "$HOME/azuredeploy.json" --parameters storageAccountType='Standard_RAGRS'
    ```

    # <a name="powershell"></a>[PowerShell](#tab/PowerShell)

    ```azurepowershell
    $projectName = Read-Host -Prompt "Enter a project name that is used to generate resource group name"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateFile "$HOME/azuredeploy.json" -storageAccountType "Standard_RAGRS"
    ```

    ---

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando los recursos de Azure ya no sean necesarios, limpie los recursos que implementó eliminando el grupo de recursos.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
1. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
1. Seleccione el nombre del grupo de recursos.  Verá un total de seis recursos en el grupo de recursos.
1. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a usar la referencia de plantilla para personalizar una plantilla existente. Para información sobre cómo crear varias instancias de cuenta de almacenamiento, consulte:

> [!div class="nextstepaction"]
> [Creación de varias instancias](./template-tutorial-create-multiple-instances.md)
