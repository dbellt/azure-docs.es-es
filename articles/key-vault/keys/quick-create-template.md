---
title: 'Inicio rápido de Azure: Creación de un almacén de claves de Azure y una clave mediante una plantilla de Azure Resource Manager | Microsoft Docs'
description: Inicio rápido que muestra cómo crear almacenes de claves de Azure y agregar claves a los almacenes mediante una plantilla de Azure Resource Manager.
services: key-vault
author: sebansal
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: keys
ms.topic: quickstart
ms.custom: mvc,subject-armqs
ms.date: 10/14/2020
ms.author: sebansal
ms.openlocfilehash: 261b939d62c4952b7dc6d1f1a4b0e91c2922b4ce
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108744936"
---
# <a name="quickstart-create-an-azure-key-vault-and-a-key-by-using-arm-template"></a>Inicio rápido: Creación de un almacén de claves de Azure y una clave mediante una plantilla de Resource Manager 

[Azure Key Vault](../general/overview.md) es un servicio en la nube que proporciona un almacén seguro para secretos, como claves, contraseñas y certificados, entre otros. Este inicio rápido se centra en el proceso de implementar una plantilla de Azure Resource Manager para crear un almacén de claves y una clave.

> [!NOTE]
> Esta característica no está disponible para Azure Government.

## <a name="prerequisites"></a>Requisitos previos

Para completar este artículo:

- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
- El usuario debería tener asignado un rol incorporado de Azure, por ejemplo, colaborador. [Obtén más información aquí](../../role-based-access-control/role-assignments-portal.md)
- La plantilla necesita el identificador de objeto de usuario de Azure AD para configurar los permisos. El procedimiento siguiente permite obtener el identificador de objeto (GUID).

    1. Ejecute el siguiente comando de Azure PowerShell o la CLI de Azure, para lo que debe seleccionar **Pruébelo** y, a continuación, pegar el script en el panel de shell. Para pegar el script, haga clic con el botón derecho en el shell y, a continuación, seleccione **Pegar**.

        # <a name="cli"></a>[CLI](#tab/CLI)
        ```azurecli-interactive
        echo "Enter your email address that is used to sign in to Azure:" &&
        read upn &&
        az ad user show --id $upn --query "objectId" &&
        echo "Press [ENTER] to continue ..."
        ```

        # <a name="powershell"></a>[PowerShell](#tab/PowerShell)
        ```azurepowershell-interactive
        $upn = Read-Host -Prompt "Enter your email address used to sign in to Azure"
        (Get-AzADUser -UserPrincipalName $upn).Id
        Write-Host "Press [ENTER] to continue..."
        ```

        ---

    1. Anote el identificador de objeto. Lo necesitará en la sección siguiente de este inicio rápido.

## <a name="review-the-template"></a>Revisión de la plantilla

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vaultName": {
      "type": "string",
      "metadata": {
        "description": "The name of the key vault to be created."
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
        "description": "The SKU of the vault to be created."
      }
    },
    "keyName": {
      "type": "string",
      "metadata": {
        "description": "The name of the key to be created."
      }
    },
    "keyType": {
      "type": "string",
      "metadata": {
        "description": "The JsonWebKeyType of the key to be created."
      }
    },
    "keyOps": {
      "type": "array",
      "defaultValue": [],
      "metadata": {
        "description": "The permitted JSON web key operations of the key to be created."
      }
    },
    "keySize": {
      "type": "int",
      "defaultValue": 2048,
      "metadata": {
        "description": "The size in bits of the key to be created."
      }
    },
    "curveName": {
      "type": "string",
      "defaultValue": "",
      "metadata": {
        "description": "The JsonWebKeyCurveName of the key to be created."
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.KeyVault/vaults",
      "apiVersion": "2019-09-01",
      "name": "[parameters('vaultName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "enableRbacAuthorization": false,
        "enableSoftDelete": false,
        "enabledForDeployment": false,
        "enabledForDiskEncryption": false,
        "enabledForTemplateDeployment": false,
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
    },
    {
      "type": "Microsoft.KeyVault/vaults/keys",
      "apiVersion": "2019-09-01",
      "name": "[concat(parameters('vaultName'), '/', parameters('keyName'))]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.KeyVault/vaults', parameters('vaultName'))]"
      ],
      "properties": {
        "kty": "[parameters('keyType')]",
        "keyOps": "[parameters('keyOps')]",
        "keySize": "[parameters('keySize')]",
        "curveName": "[parameters('curveName')]"
      }
    }
  ],
  "outputs": {
    "proxyKey": {
      "type": "object",
      "value": "[reference(resourceId('Microsoft.KeyVault/vaults/keys', parameters('vaultName'), parameters('keyName')))]"
    }
  }
}
```

En la plantilla se definen dos recursos:

- [Microsoft.KeyVault/vaults](/azure/templates/microsoft.keyvault/vaults)
- Microsoft.KeyVault/vaults/keys

Puede encontrar más ejemplos de plantillas de Azure Key Vault en [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Keyvault&pageNumber=1&sort=Popular).

## <a name="parameters-and-definitions"></a>Parámetros y definiciones

|Parámetro  |Definición  |
|---------|---------|
|**Keysize**  | Especifica las operaciones que se pueden realizar utilizando esta clave. Si no se especifica este parámetro, se pueden realizar todas las operaciones. Los valores aceptables para este parámetro son una lista separada por comas de las operaciones con clave, tal y como se define en la [especificación JSON Web Key (JWK):](https://tools.ietf.org/html/draft-ietf-jose-json-web-key-41) <br> `["sign", "verify", "encrypt", "decrypt", " wrapKey", "unwrapKey"]` |
|**CurveName**  |  Nombre de la curva elíptica para el tipo de clave EC. Consulte [JsonWebKeyCurveName](/rest/api/keyvault/createkey/createkey#jsonwebkeycurvename) |
|**Kty**  |  Tipo de clave que se va a crear. Para saber qué valores pueden usarse, consulte [JsonWebKeyType](/rest/api/keyvault/createkey/createkey#jsonwebkeytype) |
|**Etiquetas** | Metadatos específicos de la aplicación en forma de pares clave-valor.  |
|**nbf**  |  Especifica una hora como un objeto DateTime antes de la cual no se puede usar la clave. El formato sería una marca temporal de Unix (el número de segundos transcurridos en la época Unix, desde el 1 de enero de 1970 UTC).  |
|**exp**  |  Especifica la hora de vencimiento como un objeto DateTime. El formato sería una marca temporal de Unix (el número de segundos transcurridos en la época Unix, desde el 1 de enero de 1970 UTC). |

## <a name="deploy-the-template"></a>Implementación de la plantilla
Puede utilizar [Azure Portal](../../azure-resource-manager/templates/deploy-portal.md), Azure PowerShell, CLI de Azure o API REST. Para obtener información sobre otros métodos de implementación, consulte [Implementación de plantillas](../../azure-resource-manager/templates/deploy-powershell.md).

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Puede usar Azure Portal para comprobar el almacén de claves y la clave o utilizar el siguiente script de la CLI de Azure o de Azure PowerShell para enumerar la clave que ha creado.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
echo "Enter your key vault name:" &&
read keyVaultName &&
az keyvault key list --vault-name $keyVaultName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$keyVaultName = Read-Host -Prompt "Enter your key vault name"
Get-AzKeyVaultKey -vaultName $keyVaultName
Write-Host "Press [ENTER] to continue..."
```

---

## <a name="clean-up-resources"></a>Limpieza de recursos

Otras guías de inicio rápido y tutoriales de Key Vault se basan en esta. Si tiene pensado seguir trabajando en otras guías de inicio rápido y tutoriales, considere la posibilidad de dejar estos recursos activos.
Cuando ya no lo necesite, elimine el grupo de recursos; de este modo se eliminarán también Key Vault y los recursos relacionados. Para eliminar el grupo de recursos mediante la CLI de Azure o Azure PowerShell:

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

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un almacén de claves y una clave mediante una plantilla de Resource Manager y ha validado la implementación. Para más información sobre Key Vault y Azure Resource Manager, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](../general/overview.md).
- Obtenga más información sobre [Azure Resource Manager](../../azure-resource-manager/management/overview.md).
- Consulte [Introducción a la seguridad de Azure Key Vault](../general/security-features.md)
