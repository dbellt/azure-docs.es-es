---
title: Secreto de Key Vault con Bicep
description: Se muestra cómo pasar un secreto de un almacén de claves como parámetro durante la implementación de Bicep.
author: mumian
ms.author: jgao
ms.topic: conceptual
ms.date: 06/01/2021
ms.openlocfilehash: f96b9228b6ebe6ab3ca6d48dc3403bbafa6e8d55
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112029682"
---
# <a name="use-azure-key-vault-to-pass-secure-parameter-value-during-bicep-deployment"></a>Uso de Azure Key Vault para pasar valores de parámetro seguros durante la implementación de Bicep

En lugar de colocar un valor seguro (como una contraseña) directamente en el archivo de Bicep o el archivo de parámetros, puede recuperar el valor de una instancia de [Azure Key Vault](../../key-vault/general/overview.md) durante una implementación. El valor se recupera haciendo referencia a Key Vault y al secreto del archivo de parámetros. Cuando un [módulo](./modules.md) espera un parámetro `string` con el modificador `secure:true`, puede usar la función `getSecret` para obtener un secreto del almacén de claves. El valor nunca se expone debido a que solo hace referencia a su identificador de almacén de claves. El almacén de claves puede existir en una suscripción distinta a la que usa para implementar el grupo de recursos.

Este artículo se centra en cómo pasar un valor confidencial como parámetro de Bicep. En el artículo no se explica cómo establecer una propiedad de máquina virtual en la dirección URL de un certificado en un almacén de claves.
Para obtener una plantilla de inicio rápido de este escenario, consulte [Instalar un certificado de Azure Key Vault en una máquina virtual](https://github.com/Azure/azure-quickstart-templates/tree/master/demos/vm-winrm-keyvault-windows).

## <a name="deploy-key-vaults-and-secrets"></a>Implementación de almacenes de claves y secretos

Para acceder a un almacén de claves durante la implementación de Bicep, establezca `enabledForTemplateDeployment` en el almacén de claves en `true`.

Si ya tiene un almacén de claves, asegúrese de que permite implementaciones de plantilla.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az keyvault update  --name ExampleVault --enabled-for-template-deployment true
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Set-AzKeyVaultAccessPolicy -VaultName ExampleVault -EnabledForTemplateDeployment
```

---

Para crear un nuevo almacén de claves y agregar un secreto, use:

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az group create --name ExampleGroup --location centralus
az keyvault create \
  --name ExampleVault \
  --resource-group ExampleGroup \
  --location centralus \
  --enabled-for-template-deployment true
az keyvault secret set --vault-name ExampleVault --name "ExamplePassword" --value "hVFkk965BuUv"
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroup -Name ExampleGroup -Location centralus
New-AzKeyVault `
  -VaultName ExampleVault `
  -resourceGroupName ExampleGroup `
  -Location centralus `
  -EnabledForTemplateDeployment
$secretvalue = ConvertTo-SecureString 'hVFkk965BuUv' -AsPlainText -Force
$secret = Set-AzKeyVaultSecret -VaultName ExampleVault -Name 'ExamplePassword' -SecretValue $secretvalue
```

---

Como propietario del almacén de claves, tiene acceso de forma automática a la creación de secretos. Si el usuario que trabaja con los secretos no es el propietario del almacén de claves, conceda acceso con:

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az keyvault set-policy \
  --upn <user-principal-name> \
  --name ExampleVault \
  --secret-permissions set delete get list
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$userPrincipalName = "<Email Address of the deployment operator>"

Set-AzKeyVaultAccessPolicy `
  -VaultName ExampleVault `
  -UserPrincipalName <user-principal-name> `
  -PermissionsToSecrets set,delete,get,list
```

---

Para más información sobre cómo crear almacenes de claves y agregar secretos, vea:

- [Establecimiento y recuperación de un secreto mediante la CLI](../../key-vault/secrets/quick-create-cli.md)
- [Establecimiento y recuperación de un secreto mediante PowerShell](../../key-vault/secrets/quick-create-powershell.md)
- [Establecimiento y recuperación de un secreto mediante Portal](../../key-vault/secrets/quick-create-portal.md)
- [Establecimiento y recuperación de un secreto mediante .NET](../../key-vault/secrets/quick-create-net.md)
- [Establecimiento y recuperación de un secreto mediante Node.js](../../key-vault/secrets/quick-create-node.md)

## <a name="grant-access-to-the-secrets"></a>Concesión de acceso a los secretos

El usuario que implementa el archivo de Bicep debe tener el permiso `Microsoft.KeyVault/vaults/deploy/action` para el ámbito del grupo de recursos y el almacén de claves. Los roles [Propietario](../../role-based-access-control/built-in-roles.md#owner) y [Colaborador](../../role-based-access-control/built-in-roles.md#contributor) conceden este acceso. Si creó el almacén de claves, es el propietario y tiene el permiso.

El siguiente procedimiento muestra cómo crear un rol con los permisos mínimos y cómo asignar el usuario.

1. Creación de un archivo JSON de definición de rol personalizado

    ```json
    {
      "Name": "Key Vault Bicep deployment operator",
      "IsCustom": true,
      "Description": "Lets you deploy a Bicep file with the access to the secrets in the Key Vault.",
      "Actions": [
        "Microsoft.KeyVault/vaults/deploy/action"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/00000000-0000-0000-0000-000000000000"
      ]
    }
    ```

    Reemplace "00000000-0000-0000-0000-000000000000" por el identificador de la suscripción.

2. Cree el nuevo rol con el archivo JSON:

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

    ```azurecli-interactive
    az role definition create --role-definition "<path-to-role-file>"
    az role assignment create \
      --role "Key Vault resource manager template deployment operator" \
      --assignee <user-principal-name> \
      --resource-group ExampleGroup
    ```

    # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

    ```azurepowershell-interactive
    New-AzRoleDefinition -InputFile "<path-to-role-file>"
    New-AzRoleAssignment `
      -ResourceGroupName ExampleGroup `
      -RoleDefinitionName "Key Vault resource manager template deployment operator" `
      -SignInName <user-principal-name>
    ```

    ---

    En los ejemplos se asigna el rol personalizado al usuario en el nivel de grupo de recursos.

Al usar un almacén de claves con el archivo de Bicep en una [aplicación administrada](../managed-applications/overview.md), debe conceder acceso a la entidad de servicio del **proveedor de recursos del dispositivo**. Para más información, consulte [Acceso al secreto de Key Vault al implementar Azure Managed Applications](../managed-applications/key-vault-access.md).

## <a name="use-getsecret-function"></a>Uso de la función getSecret

Puede usar la función [`getSecret` ](./bicep-functions-resource.md#getsecret) para obtener un secreto del almacén de claves y pasar el valor a un parámetro `string` de un módulo. Solo se puede llamar a la función `getSecret` en un recurso `Microsoft.KeyVault/vaults` y solo se puede usar con el parámetro con el decorador `@secure()`.

El siguiente archivo de Bicep crea un servidor de Azure SQL. El parámetro `adminPassword` tiene un decorador `@secure()`.

```bicep
param sqlServerName string
param adminLogin string

@secure()
param adminPassword string

resource sqlServer 'Microsoft.Sql/servers@2020-11-01-preview' = {
  name: sqlServerName
  location: resourceGroup().location
  properties: {
    administratorLogin: adminLogin
    administratorLoginPassword: adminPassword
    version: '12.0'
  }
}
```

Vamos a usar el archivo de Bicep anterior como módulo dado que el nombre de archivo es *sql.bicep* en el mismo directorio que el archivo de Bicep principal.

El siguiente archivo de Bicep consume sql.bicep como módulo.  El archivo de Bicep hace referencia a un almacén de claves existente y llama a la función `getSecret` para recuperar el secreto del almacén de claves; luego, pasa el valor como parámetro al módulo.

```bicep
param sqlServerName string
param adminLogin string

param subscriptionId string
param kvResourceGroup string
param kvName string

resource kv 'Microsoft.KeyVault/vaults@2019-09-01' existing = {
  name: kvName
  scope: resourceGroup(subscriptionId, kvResourceGroup )
}

module sql './sql.bicep' = {
  name: 'deploySQL'
  params: {
    sqlServerName: sqlServerName
    adminLogin: adminLogin
    adminPassword: kv.getSecret('vmAdminPassword')
  }
}
```

## <a name="reference-secrets-in-parameter-file"></a>Referencia a los secretos del archivo de parámetros

Con este enfoque, se hace referencia al almacén de claves del archivo de parámetros, no al archivo de Bicep. La siguiente imagen muestra que el archivo de parámetros hace referencia al secreto y pasa dicho valor al archivo de Bicep.

![Diagrama de la integración del almacén de claves de Resource Manager](./media/key-vault-parameter/statickeyvault.png)

El siguiente archivo de Bicep implementa un servidor SQL que incluye una contraseña de administrador. El parámetro de contraseña se establece en una cadena segura. No obstante, el archivo de Bicep no especifica de dónde procede ese valor.

```bicep
param adminLogin string

@secure()
param adminPassword string

param sqlServerName string

resource sqlServer 'Microsoft.Sql/servers@2020-11-01-preview' = {
  name: sqlServerName
  location: resourceGroup().location
  properties: {
    administratorLogin: adminLogin
    administratorLoginPassword: adminPassword
    version: '12.0'
  }
}
```

---

Ahora, cree un archivo de parámetros para el archivo de Bicep anterior. En el archivo de parámetros, especifique un parámetro que coincida con el nombre del parámetro del archivo de Bicep. Para el valor del parámetro, haga referencia al secreto del almacén de claves. Se hace referencia al secreto pasando el identificador de recurso de almacén de claves y el nombre del secreto:

En el siguiente archivo de parámetros, debe existir el secreto del almacén de claves y tendrá que proporcionar un valor estático para su identificador de recurso.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "adminLogin": {
      "value": "exampleadmin"
    },
    "adminPassword": {
      "reference": {
        "keyVault": {
          "id": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.KeyVault/vaults/<vault-name>"
        },
        "secretName": "ExamplePassword"
      }
    },
    "sqlServerName": {
      "value": "<your-server-name>"
    }
  }
}
```

Si necesita utilizar una versión del secreto distinta de la actual, incluya la propiedad `secretVersion`.

```json
"secretName": "ExamplePassword",
"secretVersion": "cd91b2b7e10e492ebb870a6ee0591b68"
```

Implemente la plantilla y pase el archivo de parámetros:

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az group create --name SqlGroup --location westus2
az deployment group create \
  --resource-group SqlGroup \
  --template-file <Bicep-file> \
  --parameters <parameter-file>
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment `
  -ResourceGroupName $resourceGroupName `
  -TemplateFile <Bicep-file> `
  -TemplateParameterFile <parameter-file>
```

---

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general sobre los almacenes de claves, consulte [¿Qué es Azure Key Vault?](../../key-vault/general/overview.md)
- Para obtener ejemplos completos de secretos de clave de referencia, consulte los [ejemplos de almacén de claves](https://github.com/rjmax/ArmExamples/tree/master/keyvaultexamples) en GitHub.
- Para un módulo de Microsoft Learn donde se usa un valor seguro de un almacén de claves, consulte [Administración de implementaciones complejas en la nube mediante características avanzadas de la plantilla de ARM](/learn/modules/manage-deployments-advanced-arm-template-features/).
