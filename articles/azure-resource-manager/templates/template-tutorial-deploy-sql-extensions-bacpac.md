---
title: Importación de archivos BACPAC de SQL con plantillas
description: Aprenda a usar las extensiones de Azure SQL Database para importar archivos BACPAC de SQL con plantillas de Azure Resource Manager (ARM).
author: mumian
ms.date: 12/09/2019
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: b19c636d7f0f85b2b70aa953ac6e734820f8e33d
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110702628"
---
# <a name="tutorial-import-sql-bacpac-files-with-arm-templates"></a>Tutorial: Importación de archivos BACPAC de SQL con plantillas de Resource Manager

Aprenda a usar las extensiones de Azure SQL Database para importar un archivo [BACPAC](/sql/relational-databases/data-tier-applications/data-tier-applications#bacpac) con las plantillas de Azure Resource Manager (ARM). Los artefactos de implementación son cualquier archivo, además de los archivos de la plantilla principal, que se necesitan para completar una implementación. El archivo BACPAC es un artefacto.

En este tutorial creará una plantilla para implementar un [servidor SQL lógico](../../azure-sql/database/logical-servers.md) y una base de datos única, e importar un archivo BACPAC. Para más información sobre la implementación de extensiones de máquina virtual de Azure mediante plantillas de Azure Resource Manager, consulte [Tutorial: Implementación de extensiones de máquina virtual con plantillas de Resource Manager](./template-tutorial-deploy-vm-extensions.md).

En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
>
> * Preparación de un archivo BACPAC.
> * Apertura de una plantilla de inicio rápido.
> * Edición de la plantilla.
> * Implemente la plantilla.
> * Comprobación de la implementación.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar este artículo, necesitará lo siguiente:

* Visual Studio Code con la extensión Herramientas de Resource Manager. Consulte [Quickstart: Creación de plantillas de ARM mediante Visual Studio Code](./quickstart-create-templates-use-visual-studio-code.md).
* Para aumentar la seguridad, utilice una contraseña generada para la cuenta de administrador del servidor. A continuación se muestra un ejemplo de generación de contraseña:

    ```console
    openssl rand -base64 32
    ```

    Azure Key Vault está diseñado para proteger las claves criptográficas y otros secretos. Para más información, consulte el [Tutorial: Integración de Azure Key Vault en la implementación de la plantilla de Resource Manager](./template-tutorial-use-key-vault.md). También se recomienda actualizar la contraseña cada tres meses.

## <a name="prepare-a-bacpac-file"></a>Preparación de un archivo BACPAC

En [GitHub](https://github.com/Azure/azure-docs-json-samples/raw/master/tutorial-sql-extension/SQLDatabaseExtension.bacpac) se comparte un archivo BACPAC. Para crear el suyo propio, consulte [Exportación de una base de datos de Azure SQL Database a un archivo BACPAC](../../azure-sql/database/database-export.md). Si elige publicar el archivo en su propia ubicación, tendrá que actualizar la plantilla más adelante en el tutorial.

El archivo BACPAC se debe almacenar en una cuenta de Azure Storage para poder importarlo con una plantilla de Resource Manager. El siguiente script de PowerShell prepara el archivo BACPAC con estos pasos:

* Descargue el archivo BACPAC.
* Cree una cuenta de Azure Storage.
* Cree un contenedor de blobs en una cuenta de almacenamiento.
* Cargue el archivo BACPAC en el contenedor.
* Mostrar la clave de la cuenta de almacenamiento y la dirección URL del blob.

1. Seleccione **Probar ahora** para abrir Azure Cloud Shell. Después, pegue el siguiente script de PowerShell en la ventana de Cloud Shell.

    ```azurepowershell-interactive
    $projectName = Read-Host -Prompt "Enter a project name that is used to generate Azure resource names"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"

    $resourceGroupName = "${projectName}rg"
    $storageAccountName = "${projectName}store"
    $containerName = "bacpacfiles"
    $bacpacFileName = "SQLDatabaseExtension.bacpac"
    $bacpacUrl = "https://github.com/Azure/azure-docs-json-samples/raw/master/tutorial-sql-extension/SQLDatabaseExtension.bacpac"

    # Download the bacpac file
    Invoke-WebRequest -Uri $bacpacUrl -OutFile "$HOME/$bacpacFileName"

    # Create a resource group
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create a storage account
    $storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroupName `
                                           -Name $storageAccountName `
                                           -SkuName Standard_LRS `
                                           -Location $location
    $storageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName `
                                                  -Name $storageAccountName).Value[0]

    # Create a container
    New-AzStorageContainer -Name $containerName -Context $storageAccount.Context

    # Upload the BACPAC file to the container
    Set-AzStorageBlobContent -File $HOME/$bacpacFileName `
                             -Container $containerName `
                             -Blob $bacpacFileName `
                             -Context $storageAccount.Context

    Write-Host "The storage account key is $storageAccountKey"
    Write-Host "The BACPAC file URL is https://$storageAccountName.blob.core.windows.net/$containerName/$bacpacFileName"
    Write-Host "Press [ENTER] to continue ..."
    ```

1. Anote la clave de la cuenta de almacenamiento y la dirección URL del archivo BACPAC. Necesitará estos valores al implementar la plantilla.

## <a name="open-a-quickstart-template"></a>Apertura de una plantilla de inicio rápido

La plantilla que se usa en este tutorial se almacena en [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorial-sql-extension/azuredeploy.json).

1. En Visual Studio Code, seleccione **Archivo** > **Abrir archivo**.
1. En **Nombre de archivo**, pegue el código URL siguiente:

    ```url
    https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorial-sql-extension/azuredeploy.json
    ```

1. Seleccione **Abrir** para abrir el archivo.

    En la plantilla hay dos recursos definidos:

   * `Microsoft.Sql/servers`. Consulte la [referencia de plantilla](/azure/templates/microsoft.sql/servers).
   * `Microsoft.SQL.servers/databases`. Consulte la [referencia de plantilla](/azure/templates/microsoft.sql/servers/databases).

        Resulta útil obtener cierta información básica de la plantilla antes de personalizarla.
1. Seleccione **Archivo** > **Guardar como** para guardar una copia del archivo en la máquina local con el nombre *azuredeploy.json*.

## <a name="edit-the-template"></a>Edición de la plantilla

1. Agregue dos parámetros más al final de la sección `parameters` para establecer la clave de la cuenta de almacenamiento y la dirección URL de BACPAC.

    ```json
        "storageAccountKey": {
          "type":"string",
          "metadata":{
            "description": "Specifies the key of the storage account where the BACPAC file is stored."
          }
        },
        "bacpacUrl": {
          "type":"string",
          "metadata":{
            "description": "Specifies the URL of the BACPAC file."
          }
        }
    ```

    Agregue una coma después de la llave de cierre (`}`) de la propiedad `adminPassword`. Para dar formato al archivo JSON desde Visual Studio Code, seleccione Mayús + Alt + F.

    Para obtener estos dos valores, consulte [Preparación de un archivo BACPAC](#prepare-a-bacpac-file).

1. Agregue dos recursos adicionales a la plantilla.

    * Para permitir que la extensión de SQL Database importe archivos BACPAC, necesita permitir el tráfico desde los servicios de Azure. Agregue la siguiente definición de regla de firewall en la definición del servidor:

        ```json
        "resources": [
          {
            "type": "firewallrules",
            "apiVersion": "2015-05-01-preview",
            "name": "AllowAllAzureIps",
            "location": "[parameters('location')]",
            "dependsOn": [
              "[parameters('databaseServerName')]"
            ],
            "properties": {
              "startIpAddress": "0.0.0.0",
              "endIpAddress": "0.0.0.0"
            }
          }
        ]
        ```

        Así es la plantilla:

        ![Plantilla con definición de regla de firewall](./media/template-tutorial-deploy-sql-extensions-bacpac/resource-manager-tutorial-deploy-sql-extensions-bacpac-firewall.png)

    * Agregue un recurso de extensión de SQL Database a la definición de la base de datos con el siguiente código JSON:

        ```json
        "resources": [
          {
            "type": "extensions",
            "apiVersion": "2014-04-01",
            "name": "Import",
            "dependsOn": [
              "[resourceId('Microsoft.Sql/servers/databases', parameters('databaseServerName'), parameters('databaseName'))]"
            ],
            "properties": {
              "storageKeyType": "StorageAccessKey",
              "storageKey": "[parameters('storageAccountKey')]",
              "storageUri": "[parameters('bacpacUrl')]",
              "administratorLogin": "[parameters('adminUser')]",
              "administratorLoginPassword": "[parameters('adminPassword')]",
              "operationMode": "Import"
            }
          }
        ]
        ```

        Así es la plantilla:

        ![Plantilla con extensión de SQL Database](./media/template-tutorial-deploy-sql-extensions-bacpac/resource-manager-tutorial-deploy-sql-extensions-bacpac.png)

        Para conocer la definición de recursos, consulte la [referencia de extensiones de SQL Database](/azure/templates/microsoft.sql/servers/databases/extensions). Estos son algunos elementos importantes:

        * `dependsOn`: el recurso de la extensión debe crearse después de que se haya creado la base de datos.
        * `storageKeyType`: especifique el tipo de clave de almacenamiento que se usa. El valor puede ser o `StorageAccessKey` o `SharedAccessKey`. Use `StorageAccessKey` en este tutorial.
        * `storageKey`: Especifique la clave de la cuenta de almacenamiento donde se almacena el archivo BACPAC. Si el tipo de clave de almacenamiento es `SharedAccessKey`, debe ir precedida de "?".
        * `storageUri`: Especifique la dirección URL del archivo BACPAC almacenado en una cuenta de almacenamiento.
        * `administratorLoginPassword`: la contraseña del administrador SQL. Use una contraseña generada. Consulte [Requisitos previos](#prerequisites).

La plantilla completada tiene este aspecto:

[!code-json[](~/resourcemanager-templates/tutorial-sql-extension/azuredeploy2.json?range=1-106&highlight=38-49,62-76,86-103)]

## <a name="deploy-the-template"></a>Implementación de la plantilla

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Consulte la sección [Implementación de la plantilla](./template-tutorial-create-templates-with-dependent-resources.md#deploy-the-template) para conocer el procedimiento de implementación. En su lugar, use el siguiente script de implementación de PowerShell.

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter the same project name that is used earlier"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$adminUsername = Read-Host -Prompt "Enter the SQL admin username"
$adminPassword = Read-Host -Prompt "Enter the admin password" -AsSecureString
$storageAccountKey = Read-Host -Prompt "Enter the storage account key"
$bacpacUrl = Read-Host -Prompt "Enter the URL of the BACPAC file"
$resourceGroupName = "${projectName}rg"

#New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -adminUser $adminUsername `
    -adminPassword $adminPassword `
    -TemplateFile "$HOME/azuredeploy.json" `
    -storageAccountKey $storageAccountKey `
    -bacpacUrl $bacpacUrl

Write-Host "Press [ENTER] to continue ..."
```

Considere la posibilidad de usar el mismo nombre de proyecto que usó al preparar el archivo BACPAC, de modo que todos los recursos se almacenen en el mismo grupo de recursos. De esta manera, resulta más fácil administrar las tareas de los recursos, como su limpieza. Si usa el mismo nombre de proyecto, puede eliminar el comando `New-AzResourceGroup` del script o responder Yes (y) o No (n) cuando se le pregunte si desea actualizar el grupo de recursos existente.

Use una contraseña generada. Consulte [Requisitos previos](#prerequisites).

## <a name="verify-the-deployment"></a>Comprobar la implementación

Para acceder al servidor desde el equipo cliente, debe agregar una regla de firewall adicional. Para más información, consulte [Creación y administración de reglas de firewall de IP](../../azure-sql/database/firewall-configure.md#create-and-manage-ip-firewall-rules).

En Azure Portal, seleccione la base de datos del grupo de recursos recién implementado. Seleccione **Editor de consultas (versión preliminar)** y, después, escriba las credenciales de administrador. Verá dos tablas importadas en la base de datos.

![Editor de consultas (versión preliminar)](./media/template-tutorial-deploy-sql-extensions-bacpac/resource-manager-tutorial-deploy-sql-extensions-bacpac-query-editor.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando los recursos de Azure ya no sean necesarios, limpie los recursos que implementó eliminando el grupo de recursos.

1. En Azure Portal, seleccione **Grupo de recursos** en el menú de la izquierda.
1. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
1. Seleccione el nombre del grupo de recursos. Verá un total de seis recursos en el grupo de recursos.
1. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha implementado un servidor y una base de datos, y ha importado un archivo BACPAC. Para obtener información sobre cómo solucionar problemas de implementación de plantillas, consulte:

> [!div class="nextstepaction"]
> [Solución de problemas de las implementaciones de plantillas de Resource Manager](./template-tutorial-troubleshoot.md)
