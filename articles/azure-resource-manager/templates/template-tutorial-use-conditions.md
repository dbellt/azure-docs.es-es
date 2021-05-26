---
title: Uso de condiciones en plantillas
description: Aprenda a implementar recursos de Azure según condiciones. Muestra cómo implementar un recurso nuevo o usar uno existente.
author: mumian
ms.date: 04/23/2020
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: bb5a6fd09fee0ef266df16055c46aac8e70ab1ec
ms.sourcegitcommit: 1ee13b62c094a550961498b7a52d0d9f0ae6d9c0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109838472"
---
# <a name="tutorial-use-condition-in-arm-templates"></a>Tutorial: Uso de condiciones en plantillas de Resource Manager

Aprenda a implementar recursos de Azure en función de las condiciones de una plantilla de Azure Resource Manager (ARM).

En el tutorial [Establecimiento del orden de implementación de los recursos](./template-tutorial-create-templates-with-dependent-resources.md), se crean una máquina virtual, una red virtual y algunos otros recursos dependientes incluidos en una cuenta de almacenamiento. En lugar de crear una nueva cuenta de almacenamiento, cada vez, dejará que la gente elija entre crear una nueva cuenta de almacenamiento y usar una existente. Para lograr este objetivo, definirá un parámetro adicional. Si el valor del parámetro es **new**, se crea una cuenta de almacenamiento. En caso contrario, se usa una cuenta de almacenamiento existente con el nombre proporcionado.

![Diagrama de las condiciones de uso de la plantilla de Resource Manager](./media/template-tutorial-use-conditions/resource-manager-template-use-condition-diagram.png)

En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
> * Abra una plantilla de inicio rápido.
> * Modificación de la plantilla
> * Implementación de la plantilla
> * Limpieza de recursos

Este tutorial solo trata de un escenario básico de condiciones de uso. Para más información, consulte:

* [Estructura de archivos de plantilla: Condición](conditional-resource-deployment.md).
* [Implementación de un recurso de forma condicional en una plantilla de Resource Manager](/azure/architecture/guide/azure-resource-manager/advanced-templates/conditional-deploy).
* [Función de plantilla: If](./template-functions-logical.md#if).
* [Funciones de comparación para plantillas de Resource Manager](./template-functions-comparison.md)

Para un módulo de Microsoft Learn que abarca las condiciones, consulte [Administración de implementaciones complejas en la nube mediante características avanzadas de la plantilla de ARM](/learn/modules/manage-deployments-advanced-arm-template-features/).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar este artículo, necesitará lo siguiente:

* Visual Studio Code con la extensión Resource Manager Tools. Consulte [Quickstart: Creación de plantillas de ARM mediante Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).
* Para aumentar la seguridad, utilice una contraseña generada para la cuenta de administrador de máquina virtual. Este es un ejemplo para generar una contraseña:

    ```console
    openssl rand -base64 32
    ```

    Azure Key Vault está diseñado para proteger las claves criptográficas y otros secretos. Para más información, consulte el [Tutorial: Integración de Azure Key Vault en la implementación de la plantilla de Resource Manager](./template-tutorial-use-key-vault.md). También se recomienda actualizar la contraseña cada tres meses.

## <a name="open-a-quickstart-template"></a>Abra una plantilla de inicio rápido.

Plantillas de inicio rápido de Azure es un repositorio de plantillas de Azure Resource Manager. En lugar de crear una plantilla desde cero, puede buscar una plantilla de ejemplo y personalizarla. La plantilla que se usa en este tutorial se denomina [Deploy a simple Windows VM](https://azure.microsoft.com/resources/templates/101-vm-simple-windows/).

1. En Visual Studio Code, seleccione **Archivo** > **Abrir archivo**.
1. En **Nombre de archivo**, pegue el código URL siguiente:

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.compute/vm-simple-windows/azuredeploy.json
    ```

1. Seleccione **Abrir** para abrir el archivo.
1. La plantilla define seis recursos:

   * [**Microsoft.Storage/storageAccounts**](/azure/templates/Microsoft.Storage/storageAccounts).
   * [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses).
   * [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups).
   * [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks).
   * [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces).
   * [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines).

    Puede resultar útil revisar la referencia de la plantilla antes de personalizar una plantilla.

1. Seleccione **Archivo** > **Guardar como** para guardar una copia del archivo en la máquina local con el nombre _azuredeploy.json_.

## <a name="modify-the-template"></a>Modificación de la plantilla

Realice dos cambios en la plantilla existente:

* Incorporación de un parámetro de nombre de cuenta de almacenamiento. Los usuarios pueden especificar un nuevo nombre de cuenta de almacenamiento o uno que ya exista.
* Agregue un nuevo parámetro llamado `newOrExisting`. En la implementación se usa este parámetro para determinar si crear una cuenta de almacenamiento o usar una cuenta de almacenamiento existente.

Éste es el procedimiento para realizar los cambios:

1. Abra _azuredeploy.json_ en Visual Studio Code.
1. Reemplace las tres apariciones de `variables('storageAccountName')` por `parameters('storageAccountName')` en toda la plantilla.
1. Quite la siguiente definición de variable:

    ![Captura de pantalla que resalta las definiciones de variables que se deben quitar.](./media/template-tutorial-use-conditions/resource-manager-tutorial-use-condition-template-remove-storageaccountname.png)

1. Agregue los dos parámetros siguientes al principio de la sección parameters:

    ```json
    "storageAccountName": {
      "type": "string"
    },
    "newOrExisting": {
      "type": "string",
      "allowedValues": [
        "new",
        "existing"
      ]
    },
    ```

    Presione Alt+Mayús+F para dar formato a la plantilla en Visual Studio Code.

    La definición de parámetros actualizada se parece a esta:

    ![Condición de uso de Resource Manager](./media/template-tutorial-use-conditions/resource-manager-tutorial-use-condition-template-parameters.png)

1. Agregue la siguiente línea al principio de la definición de la cuenta de almacenamiento.

    ```json
    "condition": "[equals(parameters('newOrExisting'),'new')]",
    ```

    La condición comprueba el valor del parámetro `newOrExisting`. Si el valor del parámetro es **new**, en la implementación se crea la cuenta de almacenamiento.

    La definición de la cuenta de almacenamiento actualizada se parece a esta:

    ![Captura de pantalla que muestra la definición de la cuenta de almacenamiento actualizada.](./media/template-tutorial-use-conditions/resource-manager-tutorial-use-condition-template.png)
1. Actualice la propiedad `storageUri` de la definición de recursos de la máquina virtual con el siguiente valor:

    ```json
    "storageUri": "[concat('https://', parameters('storageAccountName'), '.blob.core.windows.net')]"
    ```

    Este cambio es necesario cuando se usa una cuenta de almacenamiento existente en otro grupo de recursos.

1. Guarde los cambios.

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Inicio de sesión en [Azure Cloud Shell](https://shell.azure.com)

1. Elija el entorno que prefiera; para ello, seleccione **PowerShell** o **Bash** (para CLI) en la esquina superior izquierda. Es necesario reiniciar el shell cuando realiza el cambio.

    ![Archivo de carga de Cloud Shell de Azure Portal](./media/template-tutorial-use-template-reference/azure-portal-cloud-shell-upload-file.png)

1. Seleccione **Cargar/descargar archivos** y, después, seleccione **Cargar**. Consulte la captura de pantalla anterior. Seleccione el archivo que guardó en la sección anterior. Después de cargar el archivo, puede usar el comando `ls` y el comando `cat` para comprobar que la operación de carga se ha realizado correctamente.

1. Ejecute el siguiente script de PowerShell para implementar la plantilla.

    > [!IMPORTANT]
    > El nombre de la cuenta de almacenamiento debe ser único en Azure. El nombre debe tener solo letras minúsculas o números. No debe superar los 24 caracteres. El nombre de la cuenta de almacenamiento es el nombre del proyecto con **store** anexado. Asegúrese de que el nombre del proyecto y el nombre de la cuenta de almacenamiento generada cumplen los requisitos para el nombre de la cuenta de almacenamiento.

    ```azurepowershell
    $projectName = Read-Host -Prompt "Enter a project name that is used to generate resource group name and resource names"
    $newOrExisting = Read-Host -Prompt "Create new or use existing (Enter new or existing)"
    $location = Read-Host -Prompt "Enter the Azure location (i.e. centralus)"
    $vmAdmin = Read-Host -Prompt "Enter the admin username"
    $vmPassword = Read-Host -Prompt "Enter the admin password" -AsSecureString
    $dnsLabelPrefix = Read-Host -Prompt "Enter the DNS Label prefix"

    $resourceGroupName = "${projectName}rg"
    $storageAccountName = "${projectName}store"

    New-AzResourceGroup -Name $resourceGroupName -Location $location
    New-AzResourceGroupDeployment `
        -ResourceGroupName $resourceGroupName `
        -adminUsername $vmAdmin `
        -adminPassword $vmPassword `
        -dnsLabelPrefix $dnsLabelPrefix `
        -storageAccountName $storageAccountName `
        -newOrExisting $newOrExisting `
        -TemplateFile "$HOME/azuredeploy.json"

    Write-Host "Press [ENTER] to continue ..."
    ```

    > [!NOTE]
    > Se produce un error en la implementación si `newOrExisting` es **new**, pero la cuenta de almacenamiento con el nombre especificado ya existe.

Pruebe otra implementación con `newOrExisting` establecido en **existing** y especifique una cuenta de almacenamiento existente. Para crear una cuenta de almacenamiento con antelación, consulte [Creación de una cuenta de almacenamiento](../../storage/common/storage-account-create.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando los recursos de Azure ya no sean necesarios, limpie los recursos que implementó eliminando el grupo de recursos. Para eliminar el grupo de recursos, seleccione **Pruébelo** para abrir Cloud Shell. Para pegar el script de PowerShell, haga clic con el botón derecho en el panel de Shell y, a continuación, seleccione **Pegar**.

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter the same project name you used in the last procedure"
$resourceGroupName = "${projectName}rg"

Remove-AzResourceGroup -Name $resourceGroupName

Write-Host "Press [ENTER] to continue ..."
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha desarrollado una plantilla que permite a los usuarios elegir entre crear una cuenta de almacenamiento y usar una existente. Para aprender a recuperar secretos de Azure Key Vault y usar los secretos como contraseñas en la implementación de plantillas, consulte:

> [!div class="nextstepaction"]
> [Integrate Key Vault in template deployment](./template-tutorial-use-key-vault.md) (Integración de Key Vault en la implementación de plantillas)
