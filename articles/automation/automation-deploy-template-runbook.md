---
title: Implementación de una plantilla de Azure Resource Manager en un runbook de Azure Automation PowerShell
description: En este artículo se describe cómo implementar una plantilla de Azure Resource Manager almacenada en Azure Storage desde un runbook de PowerShell.
services: automation
ms.subservice: process-automation
ms.date: 09/22/2020
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
keywords: powershell, runbook, json, azure automation
ms.openlocfilehash: 20dbf9f9bbf97ed0c24ea3a525c56c7cde2db428
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107834847"
---
# <a name="deploy-an-azure-resource-manager-template-in-a-powershell-runbook"></a>Implementación de una plantilla de Azure Resource Manager en un runbook de PowerShell

Puede escribir un [runbook de Azure Automation PowerShell](./learn/automation-tutorial-runbook-textual-powershell.md) que implemente un recurso de Azure mediante una [plantilla de Azure Resource Manager](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md). Las plantillas le permiten usar Azure Automation para automatizar la implementación de los recursos de Azure. Puede mantener las plantillas de Resource Manager en una ubicación central segura como Azure Storage.

En este artículo, crearemos un runbook de PowerShell que use una plantilla de Resource Manager almacenada en [Azure Storage](../storage/common/storage-introduction.md) para implementar una nueva cuenta de Azure Storage.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción de Azure. Si aún no tiene ninguna, puede [activar las ventajas de la suscripción a MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) o [suscribirse para obtener una cuenta gratuita](https://azure.microsoft.com/free/).
* [Cuenta de Automation](./automation-security-overview.md) para contener el Runbook y autenticarse en recursos de Azure. Esta cuenta debe tener permiso para iniciar y detener la máquina virtual.
* [Cuenta de Azure Storage](../storage/common/storage-account-create.md) donde se va a almacenar la plantilla de Resource Manager.
* Azure PowerShell instalado en una máquina local. Consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps) para información sobre cómo obtener Azure PowerShell.

## <a name="create-the-resource-manager-template"></a>Creación de la plantilla de Resource Manager

En este ejemplo, usamos una plantilla de Resource Manager que implementa una nueva cuenta de Azure Storage.

En un editor de texto, copie el texto siguiente:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
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
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    }
  },
  "variables": {
    "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('storageAccountName')]",
      "apiVersion": "2018-02-01",
      "location": "[parameters('location')]",
      "sku": {
          "name": "[parameters('storageAccountType')]"
      },
      "kind": "Storage", 
      "properties": {
      }
    }
  ],
  "outputs": {
      "storageAccountName": {
          "type": "string",
          "value": "[variables('storageAccountName')]"
      }
  }
}
```

Guarde el archivo localmente como **TemplateTest.json**.

## <a name="save-the-resource-manager-template-in-azure-storage"></a>Guarde la plantilla de Resource Manager en Azure Storage.

Ahora, usaremos PowerShell para crear un recurso compartido de archivos de Azure Storage y cargaremos el archivo **TemplateTest.json**. Para obtener instrucciones sobre cómo crear un recurso compartido de archivos y cargar un archivo en Azure Portal, consulte [Introducción a Azure File Storage en Windows](../storage/files/storage-dotnet-how-to-use-files.md).

Inicie PowerShell en la máquina local y ejecute los comandos siguientes para crear un recurso compartido de archivos y cargar la plantilla de Resource Manager en él.

```powershell
# Log into Azure
Connect-AzAccount

# Get the access key for your storage account
$key = Get-AzStorageAccountKey -ResourceGroupName 'MyAzureAccount' -Name 'MyStorageAccount'

# Create an Azure Storage context using the first access key
$context = New-AzStorageContext -StorageAccountName 'MyStorageAccount' -StorageAccountKey $key[0].value

# Create a file share named 'resource-templates' in your Azure Storage account
$fileShare = New-AzStorageShare -Name 'resource-templates' -Context $context

# Add the TemplateTest.json file to the new file share
# "TemplatePath" is the path where you saved the TemplateTest.json file
$templateFile = 'C:\TemplatePath'
Set-AzStorageFileContent -ShareName $fileShare.Name -Context $context -Source $templateFile
```

## <a name="create-the-powershell-runbook-script"></a>Creación del script del runbook de PowerShell

Ahora crearemos un script de PowerShell que obtenga el archivo **TemplateTest.json** de Azure Storage e implemente la plantilla para crear una nueva cuenta de Azure Storage.

En un editor de texto, pegue el texto siguiente:

```powershell
param (
    [Parameter(Mandatory=$true)]
    [string]
    $ResourceGroupName,

    [Parameter(Mandatory=$true)]
    [string]
    $StorageAccountName,

    [Parameter(Mandatory=$true)]
    [string]
    $StorageAccountKey,

    [Parameter(Mandatory=$true)]
    [string]
    $StorageFileName
)

# Authenticate to Azure if running from Azure Automation
$ServicePrincipalConnection = Get-AutomationConnection -Name "AzureRunAsConnection"
Connect-AzAccount `
    -ServicePrincipal `
    -Tenant $ServicePrincipalConnection.TenantId `
    -ApplicationId $ServicePrincipalConnection.ApplicationId `
    -CertificateThumbprint $ServicePrincipalConnection.CertificateThumbprint | Write-Verbose

#Set the parameter values for the Resource Manager template
$Parameters = @{
    "storageAccountType"="Standard_LRS"
    }

# Create a new context
$Context = New-AzStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

Get-AzStorageFileContent -ShareName 'resource-templates' -Context $Context -path 'TemplateTest.json' -Destination 'C:\Temp'

$TemplateFile = Join-Path -Path 'C:\Temp' -ChildPath $StorageFileName

# Deploy the storage account
New-AzResourceGroupDeployment -ResourceGroupName $ResourceGroupName -TemplateFile $TemplateFile -TemplateParameterObject $Parameters 
```

Guarde el archivo localmente como **DeployTemplate.ps1**.

## <a name="import-and-publish-the-runbook-into-your-azure-automation-account"></a>Importación y publicación del runbook en su cuenta de Azure Automation

Ahora usaremos PowerShell para importar el runbook en su cuenta de Azure Automation y, a continuación, publicaremos el runbook. Para información sobre cómo importar y publicar un runbook en Azure Portal, consulte [Administración de runbooks en Azure Automation](manage-runbooks.md).

Para importar **DeployTemplate.ps1** a su cuenta de Automation como un runbook de PowerShell, ejecute los siguientes comandos de PowerShell:

```powershell
# MyPath is the path where you saved DeployTemplate.ps1
# MyResourceGroup is the name of the Azure ResourceGroup that contains your Azure Automation account
# MyAutomationAccount is the name of your Automation account
$importParams = @{
    Path = 'C:\MyPath\DeployTemplate.ps1'
    ResourceGroupName = 'MyResourceGroup'
    AutomationAccountName = 'MyAutomationAccount'
    Type = 'PowerShell'
}
Import-AzAutomationRunbook @importParams

# Publish the runbook
$publishParams = @{
    ResourceGroupName = 'MyResourceGroup'
    AutomationAccountName = 'MyAutomationAccount'
    Name = 'DeployTemplate'
}
Publish-AzAutomationRunbook @publishParams
```

## <a name="start-the-runbook"></a>Inicio del runbook

Ahora iniciaremos el runbook llamando al cmdlet [Start-AzAutomationRunbook](/powershell/module/Az.Automation/Start-AzAutomationRunbook). Para obtener información acerca de cómo iniciar un runbook en Azure Portal, consulte [Inicio de un runbook en Azure Automation](./start-runbooks.md).

Ejecute los siguientes comandos en la consola de PowerShell:

```powershell
# Set up the parameters for the runbook
$runbookParams = @{
    ResourceGroupName = 'MyResourceGroup'
    StorageAccountName = 'MyStorageAccount'
    StorageAccountKey = $key[0].Value # We got this key earlier
    StorageFileName = 'TemplateTest.json'
}

# Set up parameters for the Start-AzAutomationRunbook cmdlet
$startParams = @{
    ResourceGroupName = 'MyResourceGroup'
    AutomationAccountName = 'MyAutomationAccount'
    Name = 'DeployTemplate'
    Parameters = $runbookParams
}

# Start the runbook
$job = Start-AzAutomationRunbook @startParams
```

Después de ejecutar el runbook, puede comprobar su estado recuperando el valor de propiedad del objeto de trabajo `$job.Status`.

El runbook obtiene la plantilla de Resource Manager y la utiliza para implementar una nueva cuenta de Azure Storage. Puede ver la nueva cuenta de almacenamiento que se ha creado ejecutando el siguiente comando:

```powershell
Get-AzStorageAccount
```

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre las plantillas de Resource Manager, consulte [Información general de Azure Resource Manager](../azure-resource-manager/management/overview.md).
* Para empezar a trabajar con Azure Storage, consulte [Introducción a Azure Storage](../storage/common/storage-introduction.md).
* Para encontrar otros runbooks útiles de Azure Automation, vea [Utilizar runbooks y módulos en Azure Automation](automation-runbook-gallery.md).
* Para ver una referencia de los cmdlets de PowerShell, consulte [Az.Automation](/powershell/module/az.automation#automation).
