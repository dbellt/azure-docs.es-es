---
title: Secreto de Key Vault con plantilla
description: Muestra cómo pasar un secreto de un almacén de claves como un parámetro durante la implementación.
ms.topic: conceptual
ms.date: 04/23/2021
ms.openlocfilehash: 232c73f1058ad3c5a931d02fa1a2184cf004263f
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107946086"
---
# <a name="use-azure-key-vault-to-pass-secure-parameter-value-during-deployment"></a>Uso de Azure Key Vault para pasar el valor de parámetro seguro durante la implementación

En lugar de pasar un valor seguro (como una contraseña) directamente en la plantilla o el archivo de parámetro, puede recuperar el valor de [Azure Key Vault](../../key-vault/general/overview.md) durante una implementación. El valor se recupera haciendo referencia a Key Vault y al secreto del archivo de parámetros. El valor nunca se expone debido a que solo hace referencia a su identificador de almacén de claves. El almacén de claves puede existir en una suscripción distinta a la que usa para implementar el grupo de recursos.

El enfoque de este artículo es el paso de un valor confidencial como parámetro de plantilla. En el artículo no se explica cómo establecer una propiedad de máquina virtual en la dirección URL de un certificado en un almacén de claves.
Para obtener una plantilla de inicio rápido de este escenario, consulte [Instalar un certificado de Azure Key Vault en una máquina virtual](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-winrm-keyvault-windows).

## <a name="deploy-key-vaults-and-secrets"></a>Implementación de almacenes de claves y secretos

Para acceder a un almacén de claves durante la implementación de plantilla, establezca `enabledForTemplateDeployment` en el almacén de claves en `true`.

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

El usuario que implementa la plantilla debe tener el permiso `Microsoft.KeyVault/vaults/deploy/action` en el ámbito del grupo de recursos y el almacén de claves. Los roles [Propietario](../../role-based-access-control/built-in-roles.md#owner) y [Colaborador](../../role-based-access-control/built-in-roles.md#contributor) conceden este acceso. Si creó el almacén de claves, es el propietario y tiene el permiso.

El siguiente procedimiento muestra cómo crear un rol con los permisos mínimos y cómo asignar el usuario.

1. Creación de un archivo JSON de definición de rol personalizado

    ```json
    {
      "Name": "Key Vault resource manager template deployment operator",
      "IsCustom": true,
      "Description": "Lets you deploy a resource manager template with the access to the secrets in the Key Vault.",
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

Si usa un almacén de claves con la plantilla para una [aplicación administrada](../managed-applications/overview.md), debe conceder acceso a la entidad de servicio del **proveedor de recursos de dispositivo**. Para más información, consulte [Acceso al secreto de Key Vault al implementar Azure Managed Applications](../managed-applications/key-vault-access.md).

## <a name="reference-secrets-with-static-id"></a>Referencia a secretos con identificador estático

Con este enfoque, se hace referencia al almacén de claves del archivo de parámetros, no de la plantilla. La siguiente imagen muestra que el archivo de parámetros hace referencia al secreto y pasa dicho valor a la plantilla.

![Diagrama de identificador estático de integración de almacén de claves de Resource Manager](./media/key-vault-parameter/statickeyvault.png)

[Tutorial: Integración de Azure Key Vault en Template Deployment de Resource Manager](./template-tutorial-use-key-vault.md) usa este método.

En la plantilla siguiente se implementa un servidor SQL que incluye una contraseña de administrador. El parámetro de contraseña se establece en una cadena segura. No obstante, en la plantilla no se especifica de dónde procede ese valor.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "adminLogin": {
      "type": "string"
    },
    "adminPassword": {
      "type": "securestring"
    },
    "sqlServerName": {
      "type": "string"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Sql/servers",
      "apiVersion": "2015-05-01-preview",
      "name": "[parameters('sqlServerName')]",
      "location": "[resourceGroup().location]",
      "tags": {},
      "properties": {
        "administratorLogin": "[parameters('adminLogin')]",
        "administratorLoginPassword": "[parameters('adminPassword')]",
        "version": "12.0"
      }
    }
  ],
  "outputs": {
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

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

Ahora, cree un archivo de parámetros para la plantilla anterior. En el archivo de parámetros, especifique un parámetro que coincida con el nombre del parámetro de la plantilla. Para el valor del parámetro, haga referencia al secreto del almacén de claves. Se hace referencia al secreto pasando el identificador de recurso de almacén de claves y el nombre del secreto:

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
  --template-uri <template-file-URI> \
  --parameters <parameter-file>
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment `
  -ResourceGroupName $resourceGroupName `
  -TemplateUri <template-file-URI> `
  -TemplateParameterFile <parameter-file>
```

---

## <a name="reference-secrets-with-dynamic-id"></a>Referencia a secretos con identificador dinámico

En la sección anterior se mostró cómo pasar un identificador de recurso estático para el secreto del almacén de claves con el parámetro. En algunos escenarios, debe hacer referencia a un secreto del almacén de claves, que varía en función de la implementación actual. O bien, puede pasar valores de parámetros a la plantilla en lugar de crear un parámetro de referencia en el archivo de parámetros. La solución es generar de forma dinámica el identificador de recurso de un secreto del almacén de claves mediante una plantilla vinculada.

El identificador del recurso no se puede generar dinámicamente en el archivo de parámetros, ya que no se permiten expresiones de plantilla en este tipo de archivos.

En la plantilla principal, agregue la plantilla anidada y pase un parámetro que contenga el identificador de recurso generado dinámicamente. La siguiente imagen muestra la forma en que un parámetro en la plantilla vinculada hace referencia el secreto.

![Identificador dinámico](./media/key-vault-parameter/dynamickeyvault.png)

La siguiente plantilla crea dinámicamente el identificador de almacén de claves y lo pasa como un parámetro.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "location": {
        "type": "string",
        "defaultValue": "[resourceGroup().location]",
        "metadata": {
          "description": "The location where the resources will be deployed."
        }
      },
      "vaultName": {
        "type": "string",
        "metadata": {
          "description": "The name of the keyvault that contains the secret."
        }
      },
      "secretName": {
        "type": "string",
        "metadata": {
          "description": "The name of the secret."
        }
      },
      "vaultResourceGroupName": {
        "type": "string",
        "metadata": {
          "description": "The name of the resource group that contains the keyvault."
        }
      },
      "vaultSubscription": {
        "type": "string",
        "defaultValue": "[subscription().subscriptionId]",
        "metadata": {
          "description": "The name of the subscription that contains the keyvault."
        }
      }
  },
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-10-01",
      "name": "dynamicSecret",
      "properties": {
        "mode": "Incremental",
        "expressionEvaluationOptions": {
          "scope": "inner"
        },
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "adminLogin": {
              "type": "string"
            },
            "adminPassword": {
              "type": "securestring"
            },
            "location": {
              "type": "string"
            }
          },
          "variables": {
            "sqlServerName": "[concat('sql-', uniqueString(resourceGroup().id, 'sql'))]"
          },
          "resources": [
            {
              "type": "Microsoft.Sql/servers",
              "apiVersion": "2018-06-01-preview",
              "name": "[variables('sqlServerName')]",
              "location": "[parameters('location')]",
              "properties": {
                "administratorLogin": "[parameters('adminLogin')]",
                "administratorLoginPassword": "[parameters('adminPassword')]"
              }
            }
          ],
          "outputs": {
            "sqlFQDN": {
              "type": "string",
              "value": "[reference(variables('sqlServerName')).fullyQualifiedDomainName]"
            }
          }
        },
        "parameters": {
          "location": {
            "value": "[parameters('location')]"
          },
          "adminLogin": {
            "value": "ghuser"
          },
          "adminPassword": {
            "reference": {
              "keyVault": {
                "id": "[resourceId(parameters('vaultSubscription'), parameters('vaultResourceGroupName'), 'Microsoft.KeyVault/vaults', parameters('vaultName'))]"
              },
              "secretName": "[parameters('secretName')]"
            }
          }
        }
      }
    }
  ],
  "outputs": {
  }
}
```

> [!NOTE]
> A partir de la versión 0.3.255 de Bicep, se necesita un archivo de parámetros para recuperar un secreto del almacén de claves porque no se admite la palabra clave `reference`. Estamos trabajando para agregar compatibilidad. Para más información, consulte el [problema de GitHub 1028](https://github.com/Azure/bicep/issues/1028).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general sobre los almacenes de claves, consulte [¿Qué es Azure Key Vault?](../../key-vault/general/overview.md)
- Para obtener ejemplos completos de secretos de clave de referencia, consulte los [ejemplos de almacén de claves](https://github.com/rjmax/ArmExamples/tree/master/keyvaultexamples) en GitHub.
- Para un módulo de Microsoft Learn donde se usa un valor seguro de un almacén de claves, consulte [Administración de implementaciones complejas en la nube mediante características avanzadas de la plantilla de ARM](/learn/modules/manage-deployments-advanced-arm-template-features/).
