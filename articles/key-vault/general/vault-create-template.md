---
title: Creación de un almacén de claves de Azure y una directiva de acceso al almacén mediante una plantilla de Resource Manager
description: En este artículo se muestra cómo crear almacenes de claves de Azure y directivas de acceso a esos almacenes mediante una plantilla de Azure Resource Manager.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 3/14/2021
ms.author: mbaldwin
ms.openlocfilehash: e70906cbf26c899744bfbe137da4ce9cfa651b20
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107753153"
---
# <a name="how-to-create-an-azure-key-vault-and-vault-access-policy-by-using-a-resource-manager-template"></a>Creación de un almacén de claves de Azure y una directiva de acceso al almacén mediante una plantilla de Resource Manager

[Azure Key Vault](../general/overview.md) es un servicio en la nube que proporciona un lugar seguro para almacenar los secretos (claves, contraseñas y certificados). En este artículo se describe el proceso de implementación de una plantilla de Azure Resource Manager para crear un almacén de claves.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar los pasos de este artículo:

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.


## <a name="create-a-key-vault-resource-manager-template"></a>Creación de una plantilla de Resource Manager para Key Vault

En la plantilla siguiente se muestra una forma básica de crear un almacén de claves. Algunos valores se especifican en la plantilla.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "keyVaultName": {
      "type": "string",
      "metadata": {
        "description": "Specifies the name of the key vault."
      }
    },
    "skuName": {
      "type": "string",
      "defaultValue": "Standard",
      "allowedValues": [
        "Standard",
        "Premium"
      ],
      "metadata": {
        "description": "Specifies whether the key vault is a standard vault or a premium vault."
      }
    }
   },
  "resources": [
    {
      "type": "Microsoft.KeyVault/vaults",
      "apiVersion": "2019-09-01",
      "name": "[parameters('keyVaultName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "enabledForDeployment": "false",
        "enabledForDiskEncryption": "false",
        "enabledForTemplateDeployment": "false",
        "tenantId": "[subscription().tenantId]",
        "accessPolicies": [],
        "sku": {
          "name": "[parameters('skuName')]",
          "family": "A"
        },
        "networkAcls": {
          "defaultAction": "Allow",
          "bypass": "AzureServices"
        }
      }
    }
  ]
}

```

Para obtener más información sobre la configuración de la plantilla de Key Vault, consulte [Referencia de plantillas de ARM de Key Vault](/azure/templates/microsoft.keyvault/vaults).

> [!IMPORTANT]
> Si se vuelve a implementar una plantilla, se invalidarán las directivas de acceso existentes en el almacén de claves. Se recomienda rellenar la propiedad `accessPolicies` con las directivas de acceso existentes para evitar perder el acceso al almacén de claves. 

## <a name="add-an-access-policy-to-a-key-vault-resource-manager-template"></a>Adición de una directiva de acceso a una plantilla de Resource Manager para Key Vault

Puede implementar directivas de acceso en un almacén de claves existente sin tener que volver a implementar toda la plantilla del almacén de claves. En la plantilla siguiente se muestra una forma básica de crear directivas de acceso:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "keyVaultName": {
      "type": "string",
      "metadata": {
        "description": "Specifies the name of the key vault."
      }
    },
    "objectId": {
      "type": "string",
      "metadata": {
        "description": "Specifies the object ID of a user, service principal or security group in the Azure Active Directory tenant for the vault. The object ID must be unique for the list of access policies. Get it by using Get-AzADUser or Get-AzADServicePrincipal cmdlets."
      }
    },
    "keysPermissions": {
      "type": "array",
      "defaultValue": [
        "list"
      ],
      "metadata": {
        "description": "Specifies the permissions to keys in the vault. Valid values are: all, encrypt, decrypt, wrapKey, unwrapKey, sign, verify, get, list, create, update, import, delete, backup, restore, recover, and purge."
      }
    },
    "secretsPermissions": {
      "type": "array",
      "defaultValue": [
        "list"
      ],
      "metadata": {
        "description": "Specifies the permissions to secrets in the vault. Valid values are: all, get, list, set, delete, backup, restore, recover, and purge."
      }
    },
    "certificatePermissions": {
      "type": "array",
      "defaultValue": [
        "list"
      ],
      "metadata": {
        "description": "Specifies the permissions to certificates in the vault. Valid values are: all,  create, delete, update, deleteissuers, get, getissuers, import, list, listissuers, managecontacts, manageissuers,  recover, backup, restore, setissuers, and purge."
      }
    },
  "resources": [
     {
      "type": "Microsoft.KeyVault/vaults/accessPolicies",
      "name": "[concat(parameters('keyVaultName'), '/add')]",
      "apiVersion": "2019-09-01",
      "properties": {
        "accessPolicies": [
          {
            "tenantId": "[subscription().tenantId]",
            "objectId": "[parameters('objectId')]",
            "permissions": {
              "keys": "[parameters('keysPermissions')]",
              "secrets": "[parameters('secretsPermissions')]",
              "certificates": "[parameters('certificatePermissions')]"
            }
          }
        ]
      }
    }
  ]
}

```

Para más información sobre la configuración de la plantilla de Key Vault, consulte [Referencia de la plantilla de Resource Manager para Key Vault](/azure/templates/microsoft.keyvault/vaults/accesspolicies).

## <a name="more-key-vault-resource-manager-templates"></a>Más plantillas de Resource Manager para Key Vault

Existen otras plantillas de Resource Manager disponibles para los objetos de Key Vault:

| Secretos | Claves | Certificados |
|--|--|--|
|<ul><li>[Guía de inicio rápido](../secrets/quick-create-template.md)<li>[Referencia](/azure/templates/microsoft.keyvault/vaults/secrets)|N/D|N/D|

Puede encontrar más plantillas de Key Vault aquí: [Referencia de Resource Manager para Key Vault](/azure/templates/microsoft.keyvault/allversions).

## <a name="deploy-the-templates"></a>Implementación de las plantillas

Puede usar Azure Portal para implementar las plantillas anteriores con la opción **Build your own template in editor** (Crear su propia plantilla en el editor), como se describe aquí: [Implementación de recursos desde una plantilla personalizada](../../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template).

También puede guardar las plantillas anteriores en archivos y usar estos comandos:  [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) y [az deployment group create](/cli/azure/deployment/group#az_deployment_group_create):

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateFile key-vault-template.json
```

```azurecli
az deployment group create --resource-group ExampleGroup --template-file key-vault-template.json
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si tiene previsto seguir adelante con los inicios rápidos y tutoriales posteriores, puede conservar estos recursos. Cuando ya no los necesite, elimine el grupo de recursos. Si elimina el grupo, el almacén de claves y los recursos relacionados también se eliminan. Para eliminar el grupo de recursos mediante la CLI de Azure o Azure PowerShell, siga estos pasos:

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

---

## <a name="resources"></a>Recursos

- Lea una [introducción a Azure Key Vault](../general/overview.md).
- Obtenga más información sobre [Administrador de recursos de Azure](../../azure-resource-manager/management/overview.md).
- Consulte [Introducción a la seguridad de Azure Key Vault](security-overview.md)

## <a name="next-steps"></a>Pasos siguientes

- [Protección del acceso a un almacén de claves](security-overview.md)
- [Autenticación en un almacén de claves](authentication.md)
- [Guía del desarrollador de Azure Key Vault](developers-guide.md)
