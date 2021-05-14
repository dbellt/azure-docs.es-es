---
title: 'Tutorial: Implementación de una plantilla vinculada'
description: Aprenda a implementar una plantilla vinculada.
ms.date: 02/12/2021
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 6bfe1a2d7fb993e51c82f14b508d697900b7a333
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108319762"
---
# <a name="tutorial-deploy-a-linked-template"></a>Tutorial: Implementación de una plantilla vinculada

En los [tutoriales anteriores](./deployment-tutorial-local-template.md), aprendió a implementar una plantilla que está almacenada en la máquina local. Para implementar soluciones complejas, puede dividir una plantilla en muchas plantillas y, a continuación, implementar estas mediante una plantilla principal. En este tutorial, aprenderá a implementar una plantilla principal que contiene la referencia a una plantilla vinculada. Cuando se implementa la plantilla principal, se desencadena la implementación de la plantilla vinculada. También aprenderá a almacenar y proteger las plantillas mediante el token de SAS. Su tiempo de realización es de unos **12 minutos**.

## <a name="prerequisites"></a>Prerrequisitos

Aunque no es obligatorio, se recomienda realizar el tutorial anterior.

## <a name="review-template"></a>Revisión de la plantilla

En los tutoriales anteriores, implementó una plantilla que creaba una cuenta de almacenamiento, un plan de App Service y una aplicación web. La plantilla que se utilizó fue:

:::code language="json" source="~/resourcemanager-templates/get-started-deployment/local-template/azuredeploy.json":::

## <a name="create-a-linked-template"></a>Creación de una plantilla vinculada

Puede separar el recurso de la cuenta de almacenamiento en una plantilla vinculada:

:::code language="json" source="~/resourcemanager-templates/get-started-deployment/linked-template/linkedStorageAccount.json":::

La siguiente plantilla es la plantilla principal. El objeto resaltado `Microsoft.Resources/deployments` muestra cómo llamar a una plantilla vinculada. La plantilla vinculada no se puede almacenar como un archivo local o un archivo que solo está disponible en la red local. Puede proporcionar un valor de URI de la plantilla vinculada que incluya HTTP o HTTPS, o bien usar la propiedad _relativePath_ para implementar una plantilla vinculada remota en una ubicación relativa a la plantilla primaria. Una opción es colocar la plantilla principal y la plantilla vinculada en una cuenta de almacenamiento.

:::code language="json" source="~/resourcemanager-templates/get-started-deployment/linked-template/azuredeploy.json" highlight="34-52":::

## <a name="store-the-linked-template"></a>Almacenamiento de la plantilla vinculada

Tanto la plantilla principal como la vinculada se almacenan en GitHub:

El siguiente script de PowerShell crea una cuenta de almacenamiento, crea un contenedor y copia las dos plantillas de un repositorio de GitHub en el contenedor. Estas dos plantillas son:

- La plantilla principal: https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/get-started-deployment/linked-template/azuredeploy.json
- La plantilla vinculada: https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/get-started-deployment/linked-template/linkedStorageAccount.json

Seleccione **Probar** para abrir Cloud Shell, después, seleccione **Copiar** para copiar el script de PowerShell y haga clic con el botón derecho en el panel del shell para pegar el script:

> [!IMPORTANT]
> Los nombres de cuentas de almacenamiento deben tener entre 3 y 24 caracteres, y usar solo números y letras minúsculas. El nombre debe ser único. En la plantilla, el nombre de la cuenta de almacenamiento es el nombre del proyecto con **store** anexado, y el nombre del proyecto debe tener entre 3 y 11 caracteres. Por lo tanto, el nombre del proyecto debe cumplir los requisitos de nombre para la cuenta de almacenamiento y tener menos de 11 caracteres.

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter a project name:"   # This name is used to generate names for Azure resources, such as storage account name.
$location = Read-Host -Prompt "Enter a location (i.e. centralus)"

$resourceGroupName = $projectName + "rg"
$storageAccountName = $projectName + "store"
$containerName = "templates" # The name of the Blob container to be created.

$mainTemplateURL = "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/get-started-deployment/linked-template/azuredeploy.json"
$linkedTemplateURL = "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/get-started-deployment/linked-template/linkedStorageAccount.json"

$mainFileName = "azuredeploy.json" # A file name used for downloading and uploading the main template.Add-PSSnapin
$linkedFileName = "linkedStorageAccount.json" # A file name used for downloading and uploading the linked template.

# Download the templates
Invoke-WebRequest -Uri $mainTemplateURL -OutFile "$home/$mainFileName"
Invoke-WebRequest -Uri $linkedTemplateURL -OutFile "$home/$linkedFileName"

# Create a resource group
New-AzResourceGroup -Name $resourceGroupName -Location $location

# Create a storage account
$storageAccount = New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName `
    -Location $location `
    -SkuName "Standard_LRS"

$context = $storageAccount.Context

# Create a container
New-AzStorageContainer -Name $containerName -Context $context -Permission Container

# Upload the templates
Set-AzStorageBlobContent `
    -Container $containerName `
    -File "$home/$mainFileName" `
    -Blob $mainFileName `
    -Context $context

Set-AzStorageBlobContent `
    -Container $containerName `
    -File "$home/$linkedFileName" `
    -Blob $linkedFileName `
    -Context $context

Write-Host "Press [ENTER] to continue ..."
```

## <a name="deploy-template"></a>Implementar plantilla

Para implementar plantillas en una cuenta de almacenamiento, genere un token de SAS y proporcione el parámetro _-QueryString_. Establezca el tiempo de expiración con un margen suficiente para completar la implementación. Los blobs que contienen las plantillas solo son accesibles para el propietario de la cuenta. Sin embargo, cuando se crea un token de SAS para el blob, este es accesible para cualquier persona con ese token. Si otro usuario intercepta el URI y el token de SAS, ese usuario podrá tener acceso a la plantilla. Un token de SAS es una buena forma de limitar el acceso a las plantillas, pero no debe incluir datos confidenciales, como contraseñas, directamente en la plantilla.

Si no ha creado el grupo de recursos, consulte [Creación del grupo de recursos](./deployment-tutorial-local-template.md#create-resource-group).

> [!NOTE]
> En el siguiente código de la CLI de Azure, el parámetro `-d` de `date` es un argumento no válido en macOS. Por tanto, para los usuarios de macOS, es necesario usar `-v+2H` para agregar 2 horas a la hora actual en el terminal de macOS.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive

$projectName = Read-Host -Prompt "Enter the same project name:"   # This name is used to generate names for Azure resources, such as storage account name.

$resourceGroupName="${projectName}rg"
$storageAccountName="${projectName}store"
$containerName = "templates"

$key = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName).Value[0]
$context = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $key

$mainTemplateUri = $context.BlobEndPoint + "$containerName/azuredeploy.json"
$sasToken = New-AzStorageContainerSASToken `
    -Context $context `
    -Container $containerName `
    -Permission r `
    -ExpiryTime (Get-Date).AddHours(2.0)
$newSas = $sasToken.substring(1)


New-AzResourceGroupDeployment `
  -Name DeployLinkedTemplate `
  -ResourceGroupName $resourceGroupName `
  -TemplateUri $mainTemplateUri `
  -QueryString $newSas `
  -projectName $projectName `
  -verbose

Write-Host "Press [ENTER] to continue ..."
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
echo "Enter a project name that is used to generate resource names:" &&
read projectName &&

resourceGroupName="${projectName}rg" &&
storageAccountName="${projectName}store" &&
containerName="templates" &&

key=$(az storage account keys list -g $resourceGroupName -n $storageAccountName --query [0].value -o tsv) &&

sasToken=$(az storage container generate-sas \
  --account-name $storageAccountName \
  --account-key $key \
  --name $containerName \
  --permissions r \
  --expiry `date -u -d "120 minutes" '+%Y-%m-%dT%H:%MZ'`) &&
sasToken=$(echo $sasToken | sed 's/"//g')&&

blobUri=$(az storage account show -n $storageAccountName -g $resourceGroupName -o tsv --query primaryEndpoints.blob) &&
templateUri="${blobUri}${containerName}/azuredeploy.json" &&

az deployment group create \
  --name DeployLinkedTemplate \
  --resource-group $resourceGroupName \
  --template-uri $templateUri \
  --parameters projectName=$projectName \
  --query-string $sasToken \
  --verbose
```

---

## <a name="clean-up-resources"></a>Limpieza de recursos

Limpie los recursos que implementó eliminando el grupo de recursos.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

Ha aprendido a implementar una plantilla vinculada. En el siguiente tutorial, aprenderá a crear una canalización de DevOps para implementar una plantilla.

> [!div class="nextstepaction"]
> [Crear una canalización](./deployment-tutorial-pipeline.md)
