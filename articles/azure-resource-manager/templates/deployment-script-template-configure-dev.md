---
title: Configuración del entorno de desarrollo para scripts de implementación en plantillas | Microsoft Docs
description: Configure el entorno de desarrollo para los scripts de implementación en las plantillas de Azure Resource Manager.
services: azure-resource-manager
author: mumian
ms.service: azure-resource-manager
ms.topic: conceptual
ms.date: 12/14/2020
ms.author: jgao
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 7a7c60ab9b3c9490c7731b3011ea9feb7ee0c02f
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108316468"
---
# <a name="configure-development-environment-for-deployment-scripts-in-arm-templates"></a>Configuración del entorno de desarrollo para scripts de implementación en plantillas de Resource Manager

Aprenda a crear un entorno de desarrollo para desarrollar y probar scripts de implementación de plantillas de ARM con una imagen de script de implementación. Puede crear una [instancia de contenedor de Azure](../../container-instances/container-instances-overview.md) o usar [Docker](https://docs.docker.com/get-docker/). En este artículo se explican ambas opciones.

## <a name="prerequisites"></a>Requisitos previos

### <a name="azure-powershell-container"></a>Contenedor de Azure PowerShell

Si no tiene un script de implementación de Azure PowerShell, puede crear un archivo *hello.ps1* mediante el siguiente contenido:

```powershell
param([string] $name)
$output = 'Hello {0}' -f $name
Write-Output $output
$DeploymentScriptOutputs = @{}
$DeploymentScriptOutputs['text'] = $output
```

### <a name="azure-cli-container"></a>Contenedor de la CLI de Azure

En el caso de una imagen de contenedor de la CLI de Azure, puede crear un archivo *hello.sh* con el siguiente contenido:

```bash
firstname=$1
lastname=$2
output="{\"name\":{\"displayName\":\"$firstname $lastname\",\"firstName\":\"$firstname\",\"lastName\":\"$lastname\"}}"
echo -n "Hello "
echo $output | jq -r '.name.displayName'
```

> [!NOTE]
> Al ejecutar un script de implementación de la CLI de Azure, una variable de entorno denominada `AZ_SCRIPTS_OUTPUT_PATH` almacena la ubicación del archivo de salida del script. La variable de entorno no está disponible en el contenedor del entorno de desarrollo. Para obtener más información acerca cómo trabajar con las salidas de la CLI de Azure, consulte [Tareas con las salidas del script de la CLI](deployment-script-template.md#work-with-outputs-from-cli-script).

## <a name="use-azure-powershell-container-instance"></a>Uso de la instancia de contenedor de Azure PowerShell

Para crear los scripts en el equipo, debe crear una cuenta de almacenamiento y montarla en la instancia de contenedor. Así, podrá cargar el script en la cuenta de almacenamiento y ejecutar el script en la instancia de contenedor.

> [!NOTE]
> La cuenta de almacenamiento que cree para probar el script no es la misma que usa el servicio de script de implementación para ejecutarlo. El servicio de implementación del script crea un nombre único como un recurso compartido de archivos en cada ejecución.

### <a name="create-an-azure-powershell-container-instance"></a>Creación de una instancia de contenedor de Azure PowerShell

La siguiente plantilla de Resource Manager crea una instancia de contenedor y un recurso compartido de archivos, y, luego, monta este en la imagen de contenedor.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "projectName": {
      "type": "string",
      "metadata": {
        "description": "Specify a project name that is used for generating resource names."
      }
    },
    "containerImage": {
      "type": "string",
      "defaultValue": "mcr.microsoft.com/azuredeploymentscripts-powershell:az5.2",
      "metadata": {
        "description": "Specify the container image."
      }
    },
    "mountPath": {
      "type": "string",
      "defaultValue": "/mnt/azscripts/azscriptinput",
      "metadata": {
        "description": "Specify the mount path."
      }
    }
  },
  "variables": {
    "storageAccountName": "[tolower(concat(parameters('projectName'), 'store'))]",
    "fileShareName": "[concat(parameters('projectName'), 'share')]",
    "containerGroupName": "[concat(parameters('projectName'), 'cg')]",
    "containerName": "[concat(parameters('projectName'), 'container')]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-06-01",
      "name": "[variables('storageAccountName')]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS",
        "tier": "Standard"
      },
      "kind": "StorageV2",
      "properties": {
        "accessTier": "Hot"
      }
    },
    {
      "type": "Microsoft.Storage/storageAccounts/fileServices/shares",
      "apiVersion": "2019-06-01",
      "name": "[concat(variables('storageAccountName'), '/default/', variables('fileShareName'))]",
      "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
      ]
    },
    {
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2019-12-01",
      "name": "[variables('containerGroupName')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
      ],
      "properties": {
        "containers": [
          {
            "name": "[variables('containerName')]",
            "properties": {
              "image": "[parameters('containerImage')]",
              "resources": {
                "requests": {
                  "cpu": 1,
                  "memoryInGb": 1.5
                }
              },
              "ports": [
                {
                  "protocol": "TCP",
                  "port": 80
                }
              ],
              "volumeMounts": [
                {
                  "name": "filesharevolume",
                  "mountPath": "[parameters('mountPath')]"
                }
              ],
              "command": [
                "/bin/sh",
                "-c",
                "pwsh -c 'Start-Sleep -Seconds 1800'"
              ]
            }
          }
        ],
        "osType": "Linux",
        "volumes": [
          {
            "name": "filesharevolume",
            "azureFile": {
              "readOnly": false,
              "shareName": "[variables('fileShareName')]",
              "storageAccountName": "[variables('storageAccountName')]",
              "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2019-06-01').keys[0].value]"
            }
          }
        ]
      }
    }
  ]
}
```

El valor predeterminado de la ruta de montaje es `/mnt/azscripts/azscriptinput`. Se trata de la ruta de acceso en la instancia de contenedor donde se monta en el recurso compartido de archivos.

La imagen de contenedor predeterminada especificada en la plantilla es **mcr.microsoft.com/azuredeploymentscripts-powershell:az5.2**. Consulte una lista de todas las [versiones de Azure PowerShell compatibles](https://mcr.microsoft.com/v2/azuredeploymentscripts-powershell/tags/list).

La plantilla suspende la instancia de contenedor después de 1800 segundos. Dispone de 30 minutos antes de que la instancia de contenedor pase al estado finalizado y la sesión finalice.

Para implementar la plantilla:

```azurepowershell
$projectName = Read-Host -Prompt "Enter a project name that is used to generate resource names"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$templateFile = Read-Host -Prompt "Enter the template file path and file name"
$resourceGroupName = "${projectName}rg"

New-AzResourceGroup -Location $location -name $resourceGroupName
New-AzResourceGroupDeployment -resourceGroupName $resourceGroupName -TemplateFile $templatefile -projectName $projectName
```

### <a name="upload-the-deployment-script"></a>Descargue el script de implementación.

Cargue el script de implementación en la cuenta de almacenamiento. A continuación se muestra un ejemplo de script de PowerShell:

```azurepowershell
$projectName = Read-Host -Prompt "Enter the same project name that you used earlier"
$fileName = Read-Host -Prompt "Enter the deployment script file name with the path"

$resourceGroupName = "${projectName}rg"
$storageAccountName = "${projectName}store"
$fileShareName = "${projectName}share"

$context = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName).Context
Set-AzStorageFileContent -Context $context -ShareName $fileShareName -Source $fileName -Force
```

También puede cargar el archivo mediante Azure Portal o la CLI de Azure.

### <a name="test-the-deployment-script"></a>Prueba del script de implementación

1. En Azure Portal, abra el grupo de recursos en el que implementó la instancia de contenedor y la cuenta de almacenamiento.
2. Abra el grupo de contenedores. El nombre predeterminado del grupo de contenedores es el nombre del proyecto con *cg* anexado. La instancia de contenedor está en el estado **En ejecución**.
3. En el menú del recurso, seleccione **Contenedores**. El nombre de la instancia de contenedor es el del proyecto con la palabra *container* (contenedor) anexada.

    ![Captura de pantalla de la conexión de la instancia de contenedor del script de implementación en Azure Portal.](./media/deployment-script-template-configure-dev/deployment-script-container-instance-connect.png)

4. Seleccione **Conectar** y, después, **Conectar**. Si no puede conectarse a la instancia de contenedor, reinicie el grupo de contenedores e inténtelo de nuevo.
5. En el panel de la consola, ejecute el siguiente comando:

    ```console
    cd /mnt/azscripts/azscriptinput
    ls
    pwsh ./hello.ps1 "John Dole"
    ```

    La salida es **Hello John Dole**.

    ![Captura de pantalla de la salida de prueba de la conexión de la instancia de contenedor del script de implementación en la consola.](./media/deployment-script-template-configure-dev/deployment-script-container-instance-test.png)

## <a name="use-an-azure-cli-container-instance"></a>Uso de una instancia de contenedor de la CLI de Azure

Para crear los scripts en el equipo, cree una cuenta de almacenamiento y móntela en la instancia de contenedor. Después, podrá cargar el script en la cuenta de almacenamiento y ejecutar el script en la instancia de contenedor.

> [!NOTE]
> La cuenta de almacenamiento que cree para probar el script no es la misma que usa el servicio de script de implementación para ejecutar el script. El servicio de implementación del script crea un nombre único como un recurso compartido de archivos en cada ejecución.

### <a name="create-an-azure-cli-container-instance"></a>Creación de una instancia de contenedor de la CLI de Azure

La siguiente plantilla de ARM crea una instancia de contenedor y un recurso compartido de archivos y, a continuación, monta el recurso compartido de archivos en la imagen de contenedor:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "projectName": {
      "type": "string",
      "metadata": {
        "description": "Specify a project name that is used for generating resource names."
      }
    },
    "containerImage": {
      "type": "string",
      "defaultValue": "mcr.microsoft.com/azure-cli:2.9.1",
      "metadata": {
        "description": "Specify the container image."
      }
    },
    "mountPath": {
      "type": "string",
      "defaultValue": "/mnt/azscripts/azscriptinput",
      "metadata": {
        "description": "Specify the mount path."
      }
    }
  },
  "variables": {
    "storageAccountName": "[tolower(concat(parameters('projectName'), 'store'))]",
    "fileShareName": "[concat(parameters('projectName'), 'share')]",
    "containerGroupName": "[concat(parameters('projectName'), 'cg')]",
    "containerName": "[concat(parameters('projectName'), 'container')]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-06-01",
      "name": "[variables('storageAccountName')]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS",
        "tier": "Standard"
      },
      "kind": "StorageV2",
      "properties": {
        "accessTier": "Hot"
      }
    },
    {
      "type": "Microsoft.Storage/storageAccounts/fileServices/shares",
      "apiVersion": "2019-06-01",
      "name": "[concat(variables('storageAccountName'), '/default/', variables('fileShareName'))]",
      "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
      ]
    },
    {
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2019-12-01",
      "name": "[variables('containerGroupName')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
      ],
      "properties": {
        "containers": [
          {
            "name": "[variables('containerName')]",
            "properties": {
              "image": "[parameters('containerImage')]",
              "resources": {
                "requests": {
                  "cpu": 1,
                  "memoryInGb": 1.5
                }
              },
              "ports": [
                {
                  "protocol": "TCP",
                  "port": 80
                }
              ],
              "volumeMounts": [
                {
                  "name": "filesharevolume",
                  "mountPath": "[parameters('mountPath')]"
                }
              ],
              "command": [
                "/bin/bash",
                "-c",
                "echo hello; sleep 1800"
              ]
            }
          }
        ],
        "osType": "Linux",
        "volumes": [
          {
            "name": "filesharevolume",
            "azureFile": {
              "readOnly": false,
              "shareName": "[variables('fileShareName')]",
              "storageAccountName": "[variables('storageAccountName')]",
              "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2019-06-01').keys[0].value]"
            }
          }
        ]
      }
    }
  ]
}
```

El valor predeterminado de la ruta de montaje es `/mnt/azscripts/azscriptinput`. Se trata de la ruta de acceso en la instancia de contenedor donde se monta en el recurso compartido de archivos.

La imagen de contenedor predeterminada especificada en la plantilla es **mcr.microsoft.com/azure-cli:2.9.1**. Consulte una lista de [versiones de la CLI de Azure compatibles](https://mcr.microsoft.com/v2/azure-cli/tags/list).

> [!IMPORTANT]
> El script de implementación usa las imágenes de la CLI disponibles de Microsoft Container Registry (MCR). Se necesita aproximadamente un mes para certificar una imagen de la CLI para el script de implementación. No utilice las versiones de la CLI que se publicaron en un plazo de 30 días. Para buscar las fechas de publicación de las imágenes, consulte las [notas de la versión de la CLI de Azure](/cli/azure/release-notes-azure-cli). Si usa una versión no compatible, el mensaje de error mostrará las versiones admitidas.

La plantilla suspende la instancia de contenedor después de 1800 segundos. Dispone de 30 minutos antes de que la instancia de contenedor pase al estado terminal y la sesión finalice.

Para implementar la plantilla:

```azurepowershell
$projectName = Read-Host -Prompt "Enter a project name that is used to generate resource names"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$templateFile = Read-Host -Prompt "Enter the template file path and file name"
$resourceGroupName = "${projectName}rg"

New-AzResourceGroup -Location $location -name $resourceGroupName
New-AzResourceGroupDeployment -resourceGroupName $resourceGroupName -TemplateFile $templatefile -projectName $projectName
```

### <a name="upload-the-deployment-script"></a>Descargue el script de implementación.

Cargue el script de implementación en la cuenta de almacenamiento. Este es un ejemplo de PowerShell:

```azurepowershell
$projectName = Read-Host -Prompt "Enter the same project name that you used earlier"
$fileName = Read-Host -Prompt "Enter the deployment script file name with the path"

$resourceGroupName = "${projectName}rg"
$storageAccountName = "${projectName}store"
$fileShareName = "${projectName}share"

$context = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName).Context
Set-AzStorageFileContent -Context $context -ShareName $fileShareName -Source $fileName -Force
```

También puede cargar el archivo mediante Azure Portal o la CLI de Azure.

### <a name="test-the-deployment-script"></a>Prueba del script de implementación

1. En Azure Portal, abra el grupo de recursos en el que implementó la instancia de contenedor y la cuenta de almacenamiento.
1. Abra el grupo de contenedores. El nombre predeterminado del grupo de contenedores es el nombre del proyecto con *cg* anexado. La instancia de contenedor se muestra en el estado **En ejecución**.
1. En el menú del recurso, seleccione **Contenedores**. El nombre de la instancia de contenedor es el del proyecto con la palabra *container* (contenedor) anexada.

    ![instancia de contenedor de conexión de script de implementación](./media/deployment-script-template-configure-dev/deployment-script-container-instance-connect.png)

1. Seleccione **Conectar** y, después, **Conectar**. Si no puede conectarse a la instancia de contenedor, reinicie el grupo de contenedores e inténtelo de nuevo.
1. En el panel de la consola, ejecute el siguiente comando:

    ```console
    cd /mnt/azscripts/azscriptinput
    ls
    ./hello.sh John Dole
    ```

    La salida es **Hello John Dole**.

    ![prueba de instancia de contenedor de script de implementación](./media/deployment-script-template-configure-dev/deployment-script-container-instance-test-cli.png)

## <a name="use-docker"></a>Uso de Docker

Puede usar una imagen de contenedor de Docker preconfigurada como entorno de desarrollo del script de implementación. Para instalar Docker, consulte [Obtener Docker](https://docs.docker.com/get-docker/).
También debe configurar el uso compartido de archivos para montar el directorio que contiene los scripts de implementación en el contenedor de Docker.

1. Extraiga la imagen del contenedor de scripts de implementación en el equipo local:

    ```command
    docker pull mcr.microsoft.com/azuredeploymentscripts-powershell:az4.3
    ```

    En este ejemplo se usa PowerShell versión 4.3.0.

    Para extraer una imagen de la CLI de un MCR, haga lo siguiente:

    ```command
    docker pull mcr.microsoft.com/azure-cli:2.0.80
    ```

    En este ejemplo se usa la CLI versión 2.0.80. El script de implementación usa las imágenes predeterminadas de contenedores de la CLI que se encuentran [aquí](https://hub.docker.com/_/microsoft-azure-cli).

1. Ejecute la imagen de Docker de manera local.

    ```command
    docker run -v <host drive letter>:/<host directory name>:/data -it mcr.microsoft.com/azuredeploymentscripts-powershell:az4.3
    ```

    Reemplace **&lt;letra de unidad del host >** y **&lt;nombre del directorio host >** por una carpeta existente en la unidad compartida. La carpeta se asigna a la carpeta _/data_ del contenedor. Por ejemplo, para asignar _D:\docker_:

    ```command
    docker run -v d:/docker:/data -it mcr.microsoft.com/azuredeploymentscripts-powershell:az4.3
    ```

    **-it** significa mantener activa la imagen del contenedor.

    Un ejemplo de la CLI:

    ```command
    docker run -v d:/docker:/data -it mcr.microsoft.com/azure-cli:2.0.80
    ```

1. En la siguiente captura de pantalla se muestra cómo ejecutar un script de PowerShell, dado que tiene un archivo *helloworld.ps1* en la unidad compartida.

    ![CMD de Docker del script de implementación de plantilla de Resource Manager](./media/deployment-script-template/resource-manager-deployment-script-docker-cmd.png)

Después de que el script se pruebe correctamente, puede usarlo como script de implementación en sus plantillas.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió a usar scripts de implementación. Para seguir un tutorial sobre scripts de implementación:

> [!div class="nextstepaction"]
> [Tutorial: Uso de scripts de implementación en plantillas de Resource Manager](./template-tutorial-deployment-script.md)
