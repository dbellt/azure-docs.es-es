---
title: 'Inicio rápido: Creación de una instancia de Azure Database for PostgresSQL con la opción Servidor flexible mediante una plantilla de Resource Manager'
description: En este inicio rápido, aprenderá a crear una instancia de Azure Database for PostgresSQL con la opción Servidor flexible mediante la plantilla de Resource Manager.
author: mksuni
ms.service: postgresql
ms.topic: quickstart
ms.custom: subject-armqs, devx-track-azurepowershell
ms.author: sumuth
ms.date: 2/11/2021
ms.openlocfilehash: 35b64c3998180b6a051d5ed4dd8c27f36b213131
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110693649"
---
# <a name="quickstart-use-an-arm-template-to-create-an-azure-database-for-postgresql---flexible-server"></a>Inicio rápido: Uso de una plantilla de Resource Manager para crear una instancia de Azure Database for PostgreSQL con la opción Servidor flexible

> [!IMPORTANT]
> Azure Database for PostgreSQL con la opción Servidor flexible se encuentra en versión preliminar.

El Servidor flexible es un servicio administrado que usa para ejecutar, administrar y escalar bases de datos de PostgreSQL de alta disponibilidad en la nube. Puede usar una plantilla de Azure Resource Manager para aprovisionar un servidor flexible de PostgreSQL para implementar varios servidores o varias bases de datos en un servidor.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Azure Resource Manager es el servicio de implementación y administración para Azure. Proporciona una capa de administración que le permite crear, actualizar y eliminar recursos de la cuenta de Azure. Se usan las características de administración, como el control de acceso, la auditoría y las etiquetas, para proteger y organizar los recursos después de la implementación. Para información acerca de las plantillas de Azure Resource Manager, consulte [Información general de la implementación de plantillas](../../azure-resource-manager/templates/overview.md).

## <a name="prerequisites"></a>Requisitos previos

Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/).

## <a name="review-the-template"></a>Revisión de la plantilla

Un servidor de Azure Database for PostgresSQL es el recurso primario de una o varias bases de datos de una región. Proporciona el ámbito de las directivas de administración que se aplican a sus bases de datos: inicio de sesión, firewall, usuarios, roles y configuraciones.

Cree el archivo _postgres-flexible-server-template.json_ y copie el siguiente script JSON en él.

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "administratorLogin": {
      "type": "String"
    },
    "administratorLoginPassword": {
      "type": "SecureString"
    },
    "location": {
      "type": "String"
    },
    "serverName": {
      "type": "String"
    },
    "serverEdition": {
      "type": "String"
    },
    "storageSizeMB": {
      "type": "Int"
    },
    "haEnabled": {
      "type": "string"
    },
    "availabilityZone": {
      "type": "String"
    },
    "version": {
      "type": "String"
    },
    "tags": {
      "defaultValue": {},
      "type": "Object"
    },
    "firewallRules": {
      "defaultValue": {},
      "type": "Object"
    },
    "vnetData": {
      "defaultValue": {},
      "type": "Object"
    },
    "backupRetentionDays": {
      "type": "Int"
    }
  },
  "variables": {
    "api": "2020-02-14-privatepreview",
    "firewallRules": "[parameters('firewallRules').rules]",
    "publicNetworkAccess": "[if(empty(parameters('vnetData')), 'Enabled', 'Disabled')]",
    "vnetDataSet": "[if(empty(parameters('vnetData')), json('{ \"subnetArmResourceId\": \"\" }'), parameters('vnetData'))]",
    "finalVnetData": "[json(concat('{ \"subnetArmResourceId\": \"', variables('vnetDataSet').subnetArmResourceId, '\"}'))]"
  },
  "resources": [
    {
      "type": "Microsoft.DBforPostgreSQL/flexibleServers",
      "apiVersion": "[variables('api')]",
      "name": "[parameters('serverName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_D4ds_v4",
        "tier": "[parameters('serverEdition')]"
      },
      "tags": "[parameters('tags')]",
      "properties": {
        "version": "[parameters('version')]",
        "administratorLogin": "[parameters('administratorLogin')]",
        "administratorLoginPassword": "[parameters('administratorLoginPassword')]",
        "publicNetworkAccess": "[variables('publicNetworkAccess')]",
        "DelegatedSubnetArguments": "[if(empty(parameters('vnetData')), json('null'), variables('finalVnetData'))]",
        "haEnabled": "[parameters('haEnabled')]",
        "storageProfile": {
          "storageMB": "[parameters('storageSizeMB')]",
          "backupRetentionDays": "[parameters('backupRetentionDays')]"
        },
        "availabilityZone": "[parameters('availabilityZone')]"
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-08-01",
      "name": "[concat('firewallRules-', copyIndex())]",
      "dependsOn": [
        "[concat('Microsoft.DBforPostgreSQL/flexibleServers/', parameters('serverName'))]"
      ],
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "resources": [
            {
              "type": "Microsoft.DBforPostgreSQL/flexibleServers/firewallRules",
              "name": "[concat(parameters('serverName'),'/',variables('firewallRules')[copyIndex()].name)]",
              "apiVersion": "[variables('api')]",
              "properties": {
                "StartIpAddress": "[variables('firewallRules')[copyIndex()].startIPAddress]",
                "EndIpAddress": "[variables('firewallRules')[copyIndex()].endIPAddress]"
              }
            }
          ]
        }
      },
      "copy": {
        "name": "firewallRulesIterator",
        "count": "[if(greater(length(variables('firewallRules')), 0), length(variables('firewallRules')), 1)]",
        "mode": "Serial"
      },
      "condition": "[greater(length(variables('firewallRules')), 0)]"
    }
  ]
}
```

Estos recursos se definen en la plantilla:

- Microsoft.DBforPostgreSQL/flexibleServers

## <a name="deploy-the-template"></a>Implementación de la plantilla

Seleccione **Pruébelo** en el bloque de código de PowerShell siguiente para abrir Azure Cloud Shell.

```azurepowershell-interactive
$serverName = Read-Host -Prompt "Enter a name for the new Azure Database for PostgreSQL server"
$resourceGroupName = Read-Host -Prompt "Enter a name for the new resource group where the server will exist"
$location = Read-Host -Prompt "Enter an Azure region (for example, centralus) for the resource group"
$adminUser = Read-Host -Prompt "Enter the Azure Database for PostgreSQL server's administrator account name"
$adminPassword = Read-Host -Prompt "Enter the administrator password" -AsSecureString

New-AzResourceGroup -Name $resourceGroupName -Location $location # Use this command when you need to create a new resource group for your deployment
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
    -TemplateFile "D:\Azure\Templates\EngineeringSite.json
    -serverName $serverName `
    -administratorLogin $adminUser `
    -administratorLoginPassword $adminPassword

Read-Host -Prompt "Press [ENTER] to continue ..."
```
---

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Siga estos pasos para verificar si el servidor se creó en Azure.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

1. En [Azure Portal](https://portal.azure.com), busque y seleccione la opción **Azure Database for PostgreSQL Flexible Servers (Preview)** (Azure Database for PostgreSQL con la opción Servidor flexible [versión preliminar]).
1. En la lista base de datos, seleccione el nuevo servidor para ver la página **Información general** y administrar el servidor.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

Tendrá que escribir el nombre del nuevo servidor para ver los detalles del servidor flexible de Azure Database for PostgreSQL.

```azurepowershell-interactive
$serverName = Read-Host -Prompt "Enter the name of your Azure Database for PostgreSQL server"
Get-AzResource -ResourceType "Microsoft.DBforPostgreSQL/flexibleServers" -Name $serverName | ft
Write-Host "Press [ENTER] to continue..."
```

# <a name="cli"></a>[CLI](#tab/CLI)

Tendrá que escribir el nombre y el grupo de recursos del nuevo servidor para ver los detalles de la instancia de servidor flexible de Azure Database for PostgreSQL.

```azurecli-interactive
echo "Enter your Azure Database for PostgreSQL Flexible Server name:" &&
read serverName &&
echo "Enter the resource group where the Azure Database for PostgreSQL Flexible Server exists:" &&
read resourcegroupName &&
az resource show --resource-group $resourcegroupName --name $serverName --resource-type "Microsoft.DBforPostgreSQL/flexibleServers"
```

---


## <a name="clean-up-resources"></a>Limpieza de recursos

Mantenga este grupo de recursos, el servidor y la base de datos única si desea ir a los artículos de [Pasos siguientes](#next-steps). Los pasos siguientes muestran cómo conectarse a la base de datos y consultarla mediante distintos métodos.

Para eliminar el grupo de recursos:

# <a name="portal"></a>[Portal](#tab/azure-portal)

En el [portal](https://portal.azure.com), seleccione el grupo de recursos que quiere eliminar.

1. Seleccione **Eliminar grupo de recursos**.
1. Para confirmar la eliminación, escriba el nombre del grupo de recursos.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Remove-AzResourceGroup -Name ExampleResourceGroup
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az group delete --name ExampleResourceGroup
```
----

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Migración de una base de datos mediante volcado y restauración](../howto-migrate-using-dump-and-restore.md)
