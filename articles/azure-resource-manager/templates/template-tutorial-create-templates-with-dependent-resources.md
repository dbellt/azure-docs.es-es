---
title: Plantilla con recursos dependientes
description: Aprenda a crear una plantilla de Azure Resource Manager (ARM) con varios recursos y a implementarla mediante Azure Portal.
author: mumian
ms.date: 04/23/2020
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 152494ef7d24d3e39af63fcb549a26c78bb27bc6
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111959426"
---
# <a name="tutorial-create-arm-templates-with-dependent-resources"></a>Tutorial: Creación de plantillas de Resource Manager con recursos dependientes

Aprenda a crear una plantilla de Azure Resource Manager (ARM) para implementar varios recursos y configurar la orden de implementación. Después de crear la plantilla, puede implementarla mediante Cloud Shell en Azure Portal.

En este tutorial, creará una cuenta de almacenamiento, una máquina virtual, una red virtual y algunos otros recursos dependientes. Algunos de los recursos no se pueden implementar hasta que existe otro recurso. Por ejemplo, no se puede crear la máquina virtual hasta que existe su cuenta de almacenamiento y la interfaz de red. Esta relación se define al marcar un recurso como dependiente de los otros. Administrador de recursos evalúa las dependencias entre recursos y los implementa en su orden dependiente. Cuando no hay recursos dependientes entre sí, Resource Manager los implementa en paralelo. Para más información consulte [Definición del orden de implementación de recursos en plantillas de Azure Resource Manager](./resource-dependency.md).

![Diagrama de orden de implementación de recursos dependientes de la plantilla de Resource Manager](./media/template-tutorial-create-templates-with-dependent-resources/resource-manager-template-dependent-resources-diagram.png)

En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
> * Abra una plantilla de inicio rápido.
> * Exploración de la plantilla
> * Implementación de la plantilla

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

Para un módulo de Microsoft Learn que abarca las dependencias de los recursos, consulte [Administración de implementaciones complejas en la nube mediante características avanzadas de la plantilla de ARM](/learn/modules/manage-deployments-advanced-arm-template-features/).

## <a name="prerequisites"></a>Prerrequisitos

Para completar este artículo, necesitará lo siguiente:

* Visual Studio Code con la extensión Resource Manager Tools. Consulte [Quickstart: Creación de plantillas de ARM mediante Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).
* Para aumentar la seguridad, utilice una contraseña generada para la cuenta de administrador de máquina virtual. Este es un ejemplo para generar una contraseña:

    ```console
    openssl rand -base64 32
    ```

    Azure Key Vault está diseñado para proteger las claves criptográficas y otros secretos. Para más información, consulte el [Tutorial: Integración de Azure Key Vault en la implementación de la plantilla de Resource Manager](./template-tutorial-use-key-vault.md). También se recomienda actualizar la contraseña cada tres meses.

## <a name="open-a-quickstart-template"></a>Abra una plantilla de inicio rápido.

Plantillas de inicio rápido de Azure es un repositorio de plantillas de Azure Resource Manager. En lugar de crear una plantilla desde cero, puede buscar una plantilla de ejemplo y personalizarla. La plantilla que se usa en este tutorial se denomina [Deploy a simple Windows VM](https://azure.microsoft.com/resources/templates/vm-simple-windows/).

1. En Visual Studio Code, seleccione **Archivo** > **Abrir archivo**.
2. En **Nombre de archivo**, pegue el código URL siguiente:

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.compute/vm-simple-windows/azuredeploy.json
    ```

3. Seleccione **Abrir** para abrir el archivo.
4. Seleccione **Archivo** > **Guardar como** para guardar una copia del archivo en la máquina local con el nombre _azuredeploy.json_.

## <a name="explore-the-template"></a>Exploración de la plantilla

Al explorar la plantilla de esta sección, intente responder estas preguntas:

* ¿Cuántos recursos de Azure se definen en esta plantilla?
* Uno de los recursos es una cuenta de Azure Storage. ¿Se parece la definición a la que se usó en el último tutorial?
* ¿Encuentra las referencias de la plantilla para los recursos definidos en esta plantilla?
* ¿Encuentra las dependencias de los recursos?

1. En Visual Studio Code, contraiga los elementos hasta que vea únicamente los elementos de primer nivel y los de segundo nivel dentro de `resources`:

    ![Plantillas de ARM de Visual Studio Code](./media/template-tutorial-create-templates-with-dependent-resources/resource-manager-template-visual-studio-code.png)

    La plantilla define seis recursos:

   * [**Microsoft.Storage/storageAccounts**](/azure/templates/Microsoft.Storage/storageAccounts).
   * [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses).
   * [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups).
   * [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks).
   * [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces).
   * [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines).

     Puede resultar útil revisar la referencia de la plantilla antes de personalizar una plantilla.

1. Expanda el primer recurso. Es una cuenta de almacenamiento. Compare la definición del recurso con la [referencia de la plantilla](/azure/templates/Microsoft.Storage/storageAccounts).

    ![Definición de la cuenta de almacenamiento de las plantilla de ARM de Visual Studio Code](./media/template-tutorial-create-templates-with-dependent-resources/resource-manager-template-storage-account-definition.png)

1. Expanda el segundo recurso. El tipo de recurso es `Microsoft.Network/publicIPAddresses`. Compare la definición del recurso con la [referencia de la plantilla](/azure/templates/microsoft.network/publicipaddresses).

    ![Definición de la dirección IP pública de las plantilla de ARM de Visual Studio Code](./media/template-tutorial-create-templates-with-dependent-resources/resource-manager-template-public-ip-address-definition.png)

1. Expanda el tercer recurso. El tipo de recurso es `Microsoft.Network/networkSecurityGroups`. Compare la definición del recurso con la [referencia de la plantilla](/azure/templates/microsoft.network/networksecuritygroups).

    ![Definición de grupo de seguridad de red de las plantilla de ARM de Visual Studio Code](./media/template-tutorial-create-templates-with-dependent-resources/resource-manager-template-network-security-group-definition.png)

1. Expanda el cuarto recurso. El tipo de recurso es `Microsoft.Network/virtualNetworks`:

    ![Elemento dependsOn de la red virtual de las plantillas de ARM de Visual Studio Code](./media/template-tutorial-create-templates-with-dependent-resources/resource-manager-template-virtual-network-definition.png)

    El elemento `dependsOn` le permite definir un recurso como dependiente de uno o varios recursos. Este recurso depende de otro recurso:

    * `Microsoft.Network/networkSecurityGroups`

1. Expanda el quinto recurso. El tipo de recurso es `Microsoft.Network/networkInterfaces`. El recurso depende de otros dos recursos:

    * `Microsoft.Network/publicIPAddresses`
    * `Microsoft.Network/virtualNetworks`

1. Expanda el sexto recurso. Este recurso es una máquina virtual. Depende de otros dos recursos:

    * `Microsoft.Storage/storageAccounts`
    * `Microsoft.Network/networkInterfaces`

El siguiente diagrama muestra los recursos y la información de dependencia de esta plantilla:

![Diagrama de dependencia de las plantilla de ARM de Visual Studio Code](./media/template-tutorial-create-templates-with-dependent-resources/resource-manager-template-visual-studio-code-dependency-diagram.png)

Mediante la especificación de las dependencias, Resource Manager implementa de forma eficaz la solución. Este implementa la cuenta de almacenamiento, la dirección IP pública y la red virtual en paralelo porque no tienen ninguna dependencia. Una vez implementadas la dirección IP y la red virtual, se crea la interfaz de red. Cuando se han implementado los recursos restantes, Resource Manager implementa la máquina virtual.

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Inicio de sesión en [Azure Cloud Shell](https://shell.azure.com)

1. Elija el entorno que prefiera; para ello, seleccione **PowerShell** o **Bash** (para CLI) en la esquina superior izquierda.  Es necesario reiniciar el shell cuando realiza el cambio.

    ![Archivo de carga de Cloud Shell de Azure Portal](./media/template-tutorial-use-template-reference/azure-portal-cloud-shell-upload-file.png)

1. Seleccione **Cargar/descargar archivos** y, después, seleccione **Cargar**. Consulte la captura de pantalla anterior. Seleccione el archivo que guardó anteriormente. Después de cargar el archivo, puede usar el comando `ls` y el comando `cat` para comprobar que la operación de carga se haya realizado correctamente.

1. Ejecute el siguiente script de PowerShell para implementar la plantilla.

    # <a name="cli"></a>[CLI](#tab/CLI)

    ```azurecli
    echo "Enter a project name that is used to generate resource group name:" &&
    read projectName &&
    echo "Enter the location (i.e. centralus):" &&
    read location &&
    echo "Enter the virtual machine admin username:" &&
    read adminUsername &&
    echo "Enter the DNS label prefix:" &&
    read dnsLabelPrefix &&
    resourceGroupName="${projectName}rg" &&
    az group create --name $resourceGroupName --location $location &&
    az deployment group create --resource-group $resourceGroupName --template-file "$HOME/azuredeploy.json" --parameters adminUsername=$adminUsername dnsLabelPrefix=$dnsLabelPrefix
    ```

    # <a name="powershell"></a>[PowerShell](#tab/PowerShell)

    ```azurepowershell
    $projectName = Read-Host -Prompt "Enter a project name that is used to generate resource group name"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $adminUsername = Read-Host -Prompt "Enter the virtual machine admin username"
    $adminPassword = Read-Host -Prompt "Enter the admin password" -AsSecureString
    $dnsLabelPrefix = Read-Host -Prompt "Enter the DNS label prefix"
    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment `
        -ResourceGroupName $resourceGroupName `
        -adminUsername $adminUsername `
        -adminPassword $adminPassword `
        -dnsLabelPrefix $dnsLabelPrefix `
        -TemplateFile "$HOME/azuredeploy.json"

    Write-Host "Press [ENTER] to continue ..."
    ```

    ---

1. RDP a la máquina virtual para comprobar que la máquina virtual se ha creado correctamente.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando los recursos de Azure ya no sean necesarios, limpie los recursos que implementó eliminando el grupo de recursos.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos. Verá un total de seis recursos en el grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha desarrollado e implementado una plantilla para crear una máquina virtual, una red virtual y los recursos dependientes. Para aprender a usar los scripts de implementación para realizar operaciones antes y después de esta, consulte:

> [!div class="nextstepaction"]
> [Uso de un script de implementación](./template-tutorial-deployment-script.md)