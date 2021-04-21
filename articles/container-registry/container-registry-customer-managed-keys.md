---
title: Cifrado del registro con una clave administrada por el cliente
description: Obtenga información sobre el cifrado en reposo de una instancia de Azure Container Registry y sobre cómo cifrar el registro Premium con una clave administrada por el cliente almacenada en Azure Key Vault
ms.topic: article
ms.date: 03/03/2021
ms.custom: ''
ms.openlocfilehash: 09eea79eb6fb9ad9e4526b1a0390664e5dd9d61e
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107784050"
---
# <a name="encrypt-registry-using-a-customer-managed-key"></a>Cifrado del registro con una clave administrada por el cliente

Cuando se almacenan imágenes y otros artefactos en una instancia de Azure Container Registry, Azure cifra automáticamente el contenido del registro en reposo con [claves administradas por el servicio](../security/fundamentals/encryption-models.md). Puede complementar el cifrado predeterminado con una capa de cifrado adicional mediante una clave que se crea y administra en Azure Key Vault (una clave administrada por el cliente). Este artículo le guía a través del proceso mediante la CLI de Azure, Azure Portal o una plantilla de Resource Manager.

Se admite el cifrado de lado servidor con claves administradas por el cliente por medio de la integración con [Azure Key Vault](../key-vault/general/overview.md): 

* Puede crear claves de cifrado propias y almacenarlas en un almacén de claves, o puede usar API de Azure Key Vault para generar claves. 
* Con Azure Key Vault, también puede auditar el uso de claves.
* Azure Container Registry admite la rotación automática de claves de cifrado del registro cuando hay disponible una nueva versión de clave en Azure Key Vault. También puede rotar manualmente las claves de cifrado del registro.

Esta característica está disponible en el nivel de servicio de un registro de contenedor **Premium**. Para obtener información sobre los límites y los niveles de servicio de los registros, vea [Niveles de servicio de Azure Container Registry](container-registry-skus.md).


## <a name="things-to-know"></a>Cosas que debe saber

* De momento, solo se puede habilitar una clave administrada por el cliente al crear un registro. Al habilitar la clave, se configura una identidad administrada *asignada por el usuario* para acceder al almacén de claves.
* Después de habilitar el cifrado con una clave administrada por el cliente en un registro, no es posible deshabilitarlo.  
* Azure Container Registry solo admite claves RSA o RSA-HSM. Las claves de curva elíptica no se admiten en este momento.
* [Confianza de contenido](container-registry-content-trust.md) no se admite actualmente en un registro cifrado con una clave administrada por el cliente.
* En un registro cifrado con una clave administrada por el cliente, los registros de ejecución de [ACR Tasks](container-registry-tasks-overview.md) solo se conservan durante 24 horas. Si necesita conservar los registros durante un período más largo, vea la guía para [exportar y almacenar registros de ejecución de tareas](container-registry-tasks-logs.md#alternative-log-storage).


> [!IMPORTANT]
> Si planea almacenar la clave de cifrado del registro en una instancia de Azure Key Vault existente que deniegue el acceso público y solo permita el punto de conexión privado o las redes virtuales seleccionadas, se requieren algunos pasos de configuración adicionales. Consulte [Escenario avanzado: Firewall de Key Vault](#advanced-scenario-key-vault-firewall) en este artículo.

## <a name="automatic-or-manual-update-of-key-versions"></a>Actualización automática o manual de las versiones de claves

Una consideración importante para la seguridad de un registro cifrado con una clave administrada por el cliente es la frecuencia con que se actualiza (rota) la clave de cifrado. Su organización podría contar con directivas de cumplimiento que requieran la actualización periódica de las [versiones](../key-vault/general/about-keys-secrets-certificates.md#objects-identifiers-and-versioning) de las claves almacenadas en Azure Key Vault cuando se usan como claves administradas por el cliente. 

Al configurar el cifrado del registro con una clave administrada por el cliente, tiene dos opciones para actualizar la versión de la clave usada para el cifrado:

* **Actualizar automáticamente la versión de la clave**: Para actualizar automáticamente una clave administrada por el cliente a una nueva versión disponible en Azure Key Vault, omita la versión de la clave al habilitar el cifrado del registro con una clave administrada por el cliente. Cuando un registro se cifra con una clave sin versión, Azure Container Registry comprueba periódicamente el almacén de claves para obtener una nueva versión de clave y actualiza la clave administrada por el cliente en un plazo de 1 hora. Azure Container Registry usa automáticamente la versión más reciente de la clave.

* **Actualizar manualmente la versión de la clave**: Para usar una versión determinada de una clave para el cifrado del registro, especifique esa versión de la clave al habilitar el cifrado del registro con una clave administrada por el cliente. Cuando un registro se cifra con una versión de clave específica, Azure Container Registry usa esa versión para el cifrado hasta que se rota manualmente la clave administrada por el cliente.

Para obtener más información, consulte [Elección del identificador de clave con o sin versión de clave](#choose-key-id-with-or-without-key-version) y [Actualización de la versión de la clave](#update-key-version), más adelante en este artículo.

## <a name="prerequisites"></a>Prerrequisitos

Para aplicar los pasos de la CLI de Azure de este artículo, se necesita la versión 2.2.0 o posterior de la CLI de Azure, o Azure Cloud Shell. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="enable-customer-managed-key---cli"></a>Habilitación de una clave administrada por el cliente: CLI

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Si es necesario, ejecute el comando [az group create][az-group-create] a fin de crear un grupo de recursos para la creación del almacén de claves, el registro de contenedor y otros recursos necesarios.

```azurecli
az group create --name <resource-group-name> --location <location>
```

### <a name="create-a-user-assigned-managed-identity"></a>Crear una identidad administrada asignada por el usuario

Cree una [identidad administrada para recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md) asignada por el usuario con el comando [az identity create][az-identity-create]. El registro usa esta identidad para acceder al servicio Key Vault.

```azurecli
az identity create \
  --resource-group <resource-group-name> \
  --name <managed-identity-name>
```

En el resultado del comando, anote los siguientes valores: `id` y `principalId`. Los va a necesitar en pasos posteriores para configurar el acceso del registro al almacén de claves.

```JSON
{
  "clientId": "xxxx2bac-xxxx-xxxx-xxxx-192cxxxx6273",
  "clientSecretUrl": "https://control-eastus.identity.azure.net/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myresourcegroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myidentityname/credentials?tid=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&oid=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&aid=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myresourcegroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myresourcegroup",
  "location": "eastus",
  "name": "myidentityname",
  "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "resourceGroup": "myresourcegroup",
  "tags": {},
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "Microsoft.ManagedIdentity/userAssignedIdentities"
}
```

Por comodidad, almacene estos valores en variables de entorno:

```azurecli
identityID=$(az identity show --resource-group <resource-group-name> --name <managed-identity-name> --query 'id' --output tsv)

identityPrincipalID=$(az identity show --resource-group <resource-group-name> --name <managed-identity-name> --query 'principalId' --output tsv)
```

### <a name="create-a-key-vault"></a>Creación de un Almacén de claves

Cree un almacén de claves con [az keyvault create][az-keyvault-create] para almacenar una clave administrada por el cliente para el cifrado del registro. 

De manera predeterminada, la configuración de **eliminación temporal** está habilitada automáticamente en un nuevo almacén de claves. Para evitar la pérdida de datos causada por eliminaciones accidentales de la clave o del almacén de claves, habilite la opción de **protección de purga**.

```azurecli
az keyvault create --name <key-vault-name> \
  --resource-group <resource-group-name> \
  --enable-purge-protection
```

Para su uso en pasos posteriores, obtenga el identificador de recurso del almacén de claves:

```azurecli
keyvaultID=$(az keyvault show --resource-group <resource-group-name> --name <key-vault-name> --query 'id' --output tsv)
```

### <a name="enable-key-vault-access"></a>Habilitación del acceso al almacén de claves

Configure una directiva para el almacén de claves de modo que la identidad pueda acceder a él. En el siguiente comando [az keyvault set-policy][az-keyvault-set-policy], se pasa el identificador de la entidad de seguridad de la identidad administrada creada, almacenada previamente en una variable de entorno. Establezca los permisos de la clave en **get**, **unwrapKey** y **wrapKey**.  

```azurecli
az keyvault set-policy \
  --resource-group <resource-group-name> \
  --name <key-vault-name> \
  --object-id $identityPrincipalID \
  --key-permissions get unwrapKey wrapKey
```

También puede usar [Azure RBAC para Key Vault](../key-vault/general/rbac-guide.md) para asignar permisos a la identidad para acceder al almacén de claves. Por ejemplo, asigne el rol de cifrado de servicio criptográfico de Key Vault a la identidad mediante el comando [az role assignment create](/cli/azure/role/assignment#az_role_assignment_create):

```azurecli 
az role assignment create --assignee $identityPrincipalID \
  --role "Key Vault Crypto Service Encryption User" \
  --scope $keyvaultID
```

### <a name="create-key-and-get-key-id"></a>Creación de clave y obtención de su identificador

Ejecute el comando [az keyvault key create][az-keyvault-key-create] para crear una clave en el almacén de claves.

```azurecli
az keyvault key create \
  --name <key-name> \
  --vault-name <key-vault-name>
```

En el resultado del comando, anote el identificador de la clave, `kid`. Este identificador se usa en el paso siguiente:

```JSON
[...]
  "key": {
    "crv": null,
    "d": null,
    "dp": null,
    "dq": null,
    "e": "AQAB",
    "k": null,
    "keyOps": [
      "encrypt",
      "decrypt",
      "sign",
      "verify",
      "wrapKey",
      "unwrapKey"
    ],
    "kid": "https://mykeyvault.vault.azure.net/keys/mykey/<version>",
    "kty": "RSA",
[...]
```

### <a name="choose-key-id-with-or-without-key-version"></a>Elección del identificador de clave con o sin versión de clave

Para mayor comodidad, almacene el formato que elija para el identificador de clave en la variable de entorno $keyID. Puede usar un identificador de clave con versión o una clave sin versión.

#### <a name="manual-key-rotation---key-id-with-version"></a>Rotación manual de claves: id. de clave con versión

Cuando se usa para cifrar un registro con una clave administrada por el cliente, esta clave solo permite la rotación manual de claves en Azure Container Registry.

En este ejemplo se almacena la propiedad `kid` de la clave:

```azurecli
keyID=$(az keyvault key show \
  --name <keyname> \
  --vault-name <key-vault-name> \
  --query 'key.kid' --output tsv)
```

#### <a name="automatic-key-rotation---key-id-omitting-version"></a>Rotación automática de claves: id. de clave que omite la versión 

Cuando se usa para cifrar un registro con una clave administrada por el cliente, esta clave permite la rotación automática de claves cuando se detecta una nueva versión de la clave en Azure Key Vault.

En este ejemplo se quita la versión de la propiedad `kid` de la clave:

```azurecli
keyID=$(az keyvault key show \
  --name <keyname> \
  --vault-name <key-vault-name> \
  --query 'key.kid' --output tsv)

keyID=$(echo $keyID | sed -e "s/\/[^/]*$//")
```

### <a name="create-a-registry-with-customer-managed-key"></a>Creación de un registro con una clave administrada por el cliente

Ejecute el comando [az acr create][az-acr-create] para crear un registro en el nivel de servicio Premium y habilitar la clave administrada por el cliente. Pase el identificador de la identidad administrada y el identificador de clave almacenados previamente en variables de entorno:

```azurecli
az acr create \
  --resource-group <resource-group-name> \
  --name <container-registry-name> \
  --identity $identityID \
  --key-encryption-key $keyID \
  --sku Premium
```

### <a name="show-encryption-status"></a>Presentación del estado de cifrado

Para mostrar si está habilitado el cifrado del registro con una clave administrada por el cliente, ejecute el comando [az acr encryption show][az-acr-encryption-show]:

```azurecli
az acr encryption show --name <registry-name>
```

Según la clave usada para cifrar el registro, la salida es similar a la siguiente:

```console
{
  "keyVaultProperties": {
    "identity": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "keyIdentifier": "https://myvault.vault.azure.net/keys/myresourcegroup/abcdefg123456789...",
    "keyRotationEnabled": true,
    "lastKeyRotationTimestamp": xxxxxxxx
    "versionedKeyIdentifier": "https://myvault.vault.azure.net/keys/myresourcegroup/abcdefg123456789...",
  },
  "status&quot;: &quot;enabled"
}
```

## <a name="enable-customer-managed-key---portal"></a>Habilitación de una clave administrada por el cliente: portal

### <a name="create-a-managed-identity"></a>Creación de una entidad administrada

Cree una [identidad administrada para recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md) asignada por el usuario en Azure Portal. Para conocer los pasos, vea [Creación de una identidad asignada por el usuario](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md#create-a-user-assigned-managed-identity).

En los pasos posteriores, use el nombre de la identidad.

:::image type="content" source="media/container-registry-customer-managed-keys/create-managed-identity.png" alt-text="Crear una identidad asignada por el usuario en Azure Portal":::

### <a name="create-a-key-vault"></a>Creación de un Almacén de claves

Para conocer los pasos para crear un almacén de claves, vea [Inicio rápido: Creación de un almacén de claves mediante Azure Portal](../key-vault/general/quick-create-portal.md).

Al crear un almacén de claves para una clave administrada por el cliente, en la pestaña **Datos básicos**, habilite el valor **Protección de purga**. Este valor ayuda a evitar la pérdida de datos causada por eliminaciones accidentales de la clave o del almacén de claves.

:::image type="content" source="media/container-registry-customer-managed-keys/create-key-vault.png" alt-text="Creación de un almacén de claves en Azure Portal":::

### <a name="enable-key-vault-access"></a>Habilitación del acceso al almacén de claves

Configure una directiva para el almacén de claves de modo que la identidad pueda acceder a él.

1. Vaya al almacén de claves.
1. Seleccione **Configuración** > **Directivas de acceso > +Agregar directiva de acceso**.
1. Seleccione **Permisos de clave** y **Obtener**, **Desencapsular clave** y **Encapsular clave**.
1. En **Seleccionar la entidad de seguridad**, seleccione el nombre de recurso de la identidad administrada asignada por el usuario.  
1. Seleccione **Agregar** y después **Guardar**.

:::image type="content" source="media/container-registry-customer-managed-keys/add-key-vault-access-policy.png" alt-text="Creación de directiva de acceso del almacén de claves":::

También puede usar [Azure RBAC para Key Vault](../key-vault/general/rbac-guide.md) para asignar permisos a la identidad para acceder al almacén de claves. Por ejemplo, asigne el rol de cifrado de servicio criptográfico de Key Vault a la identidad.

1. Vaya al almacén de claves.
1. Seleccione **Control de acceso (IAM)**  >  **+Agregar** > **Agregar asignación de roles**.
1. En la ventana **Agregar asignación de roles**:
    1. Seleccione el rol **Usuario del cifrado del servicio de cifrado de Key Vault**. 
    1. Asigne acceso a la **identidad administrada asignada por el usuario**.
    1. Seleccione el nombre de recurso de la identidad administrada asignada por el usuario y luego **Guardar**.

### <a name="create-key-optional"></a>Creación de la clave (opcional)

Opcionalmente, puede crear una clave en el almacén de claves para su uso con el fin de cifrar el registro. Siga estos pasos si desea seleccionar una versión de clave específica como clave administrada por el cliente. 

1. Vaya al almacén de claves.
1. Seleccione **Configuración** > **Claves**.
1. Seleccione **+Generar o importar** y escriba un nombre único para la clave.
1. Acepte los restantes valores predeterminados y seleccione **Crear**.
1. Tras la creación, seleccione la clave y, después, la versión actual. Copie el **identificador de clave** de la versión de la clave.

### <a name="create-azure-container-registry"></a>Creación de una instancia de Azure Container Registry

1. Seleccione **Crear un recurso** > **Contenedores** > **Registro de contenedor**.
1. En la pestaña **Datos básicos**, seleccione o cree un grupo de recursos y escriba un nombre de registro. En **SKU**, seleccione **Premium**.
1. En la pestaña **Cifrado**, en **Clave administrada por el cliente**, seleccione **Habilitado**.
1. En **Identidad**, seleccione la identidad administrada que ha creado.
1. En **Cifrado**, realice alguna de la siguientes acciones:
    * Seleccione **Seleccionar de Key Vault** y seleccione un almacén de claves y una clave existentes, o bien **Crear nuevo**. La clave que seleccione no tiene versiones y permite la rotación automática de claves.
    * Seleccione **Escribir el URI de la clave** y proporcione un identificador de clave directamente. Puede proporcionar un URI de clave con versión (para una clave que se debe rotar manualmente) o un URI de clave sin versión (que habilita la rotación automática de claves). 
1. En la pestaña **Cifrado**, seleccione **Revisar y crear**.
1. Seleccione **Crear** para implementar la instancia del registro.

:::image type="content" source="media/container-registry-customer-managed-keys/create-encrypted-registry.png" alt-text="Creación de un registro cifrado en Azure Portal":::

Para ver el estado de cifrado del registro en el portal, vaya al registro. En **Configuración**, seleccione **Cifrado**.

## <a name="enable-customer-managed-key---template"></a>Habilitación de una clave administrada por el cliente: plantilla

También puede usar una plantilla de Resource Manager para crear un registro y habilitar el cifrado con una clave administrada por el cliente.

La siguiente plantilla crea un nuevo registro de contenedor y una identidad administrada asignada por el usuario. Copie el siguiente contenido en un archivo nuevo y guárdelo con un nombre de archivo como `CMKtemplate.json`.

```JSON
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vault_name": {
      "defaultValue": "",
      "type": "String"
    },
    "registry_name": {
      "defaultValue": "",
      "type": "String"
    },
    "identity_name": {
      "defaultValue": "",
      "type": "String"
    },
    "kek_id": {
      "type": "String"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.ContainerRegistry/registries",
      "apiVersion": "2019-12-01-preview",
      "name": "[parameters('registry_name')]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Premium",
        "tier": "Premium"
      },
      "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
          "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('identity_name'))]": {}
        }
      },
      "dependsOn": [
        "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('identity_name'))]"
      ],
      "properties": {
        "adminUserEnabled": false,
        "encryption": {
          "status": "enabled",
          "keyVaultProperties": {
            "identity": "[reference(resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('identity_name')), '2018-11-30').clientId]",
            "KeyIdentifier": "[parameters('kek_id')]"
          }
        },
        "networkRuleSet": {
          "defaultAction": "Allow",
          "virtualNetworkRules": [],
          "ipRules": []
        },
        "policies": {
          "quarantinePolicy": {
            "status": "disabled"
          },
          "trustPolicy": {
            "type": "Notary",
            "status": "disabled"
          },
          "retentionPolicy": {
            "days": 7,
            "status": "disabled"
          }
        }
      }
    },
    {
      "type": "Microsoft.KeyVault/vaults/accessPolicies",
      "apiVersion": "2018-02-14",
      "name": "[concat(parameters('vault_name'), '/add')]",
      "dependsOn": [
        "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('identity_name'))]"
      ],
      "properties": {
        "accessPolicies": [
          {
            "tenantId": "[subscription().tenantId]",
            "objectId": "[reference(resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('identity_name')), '2018-11-30').principalId]",
            "permissions": {
              "keys": [
                "get",
                "unwrapKey",
                "wrapKey"
              ]
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.ManagedIdentity/userAssignedIdentities",
      "apiVersion": "2018-11-30",
      "name": "[parameters('identity_name')]",
      "location": "[resourceGroup().location]"
    }
  ]
}
```

Siga los pasos de las secciones anteriores para crear los siguientes recursos:

* Almacén de claves, identificado por nombre
* Clave del almacén de claves, identificada por identificador de clave

Ejecute el siguiente comando [az deployment group create][az-deployment-group-create] para crear el registro con el archivo de plantilla anterior. Donde se indique, proporcione un nuevo nombre de registro y un nombre de identidad administrada, así como el nombre del almacén de claves y el identificador de clave que ha creado.

```bash
az deployment group create \
  --resource-group <resource-group-name> \
  --template-file CMKtemplate.json \
  --parameters \
    registry_name=<registry-name> \
    identity_name=<managed-identity> \
    vault_name=<key-vault-name> \
    kek_id=<key-vault-key-id>
```

### <a name="show-encryption-status"></a>Presentación del estado de cifrado

Para mostrar el estado de cifrado del registro, ejecute el comando [az acr encryption show][az-acr-encryption-show]:

```azurecli
az acr encryption show --name <registry-name>
```

## <a name="use-the-registry"></a>Uso del registro

Después de habilitar una clave administrada por el cliente en un registro, puede realizar las mismas operaciones de registro que en un registro que no está cifrado con una clave administrada por el cliente. Por ejemplo, puede autenticarse en el registro e insertar imágenes de Docker. Vea comandos de ejemplo en [Inserción y extracción de una imagen](container-registry-get-started-docker-cli.md).

## <a name="rotate-key"></a>Rotación de clave

Actualice la versión de la clave en Azure Key Vault o cree una nueva clave, y luego actualice el registro para cifrar los datos mediante la clave. Puede realizar estos pasos con la CLI de Azure o en el portal.

Al girar una clave, normalmente se especifica la misma identidad usada al crear el registro. Opcionalmente, configure una nueva identidad asignada por el usuario para el acceso a la clave o habilite y especifique la identidad asignada por el sistema del registro.

> [!NOTE]
> Establezca el [acceso al almacén de claves](#enable-key-vault-access) necesario para la identidad que configure para el acceso a claves.

### <a name="update-key-version"></a>Actualización de la versión de la clave

Un escenario común es actualizar la versión de la clave usada como clave administrada por el cliente. En función de cómo se configure el cifrado del registro, la clave administrada por el cliente en Azure Container Registry se actualiza automáticamente o se debe actualizar manualmente.

### <a name="azure-cli"></a>Azure CLI

Use comandos [az keyvault key][az-keyvault-key] para crear o administrar las claves del almacén de claves. Para crear una nueva versión de la clave, ejecute el comando [az keyvault key create][az-keyvault-key-create]:

```azurecli
# Create new version of existing key
az keyvault key create \
  –-name <key-name> \
  --vault-name <key-vault-name>
```

El paso siguiente depende de cómo esté configurado el cifrado del registro:

* Si el registro está configurado para detectar las actualizaciones de la versión de la clave, la clave administrada por el cliente se actualiza automáticamente en el plazo de 1 hora.

* Si el registro está configurado para requerir la actualización manual de una nueva versión de la clave, ejecute el comando [az acr encryption rotate-key][az-acr-encryption-rotate-key], pasando el nuevo identificador de clave y la identidad que quiere configurar:

Para actualizar manualmente la versión de la clave administrada por el cliente:

```azurecli
# Rotate key and use user-assigned identity
az acr encryption rotate-key \
  --name <registry-name> \
  --key-encryption-key <new-key-id> \
  --identity <principal-id-user-assigned-identity>

# Rotate key and use system-assigned identity
az acr encryption rotate-key \
  --name <registry-name> \
  --key-encryption-key <new-key-id> \
  --identity [system]
```

> [!TIP]
> Al ejecutar `az acr encryption rotate-key`, puede pasar un identificador de clave con versión o un identificador de clave sin versión. Si usa un identificador de clave sin versión, el registro se configura para detectar automáticamente las actualizaciones posteriores de la versión de la clave.

### <a name="portal"></a>Portal

Use el valor **Cifrado** del registro para actualizar el almacén de claves, la clave o la configuración de identidad usados para la clave administrada por el cliente.

Por ejemplo, para configurar una nueva clave:

1. En el portal, vaya al registro.
1. En **Configuración**, seleccione **Cifrado** > **Cambiar clave**.

    :::image type="content" source="media/container-registry-customer-managed-keys/rotate-key.png" alt-text="Giro de clave en Azure Portal":::
1. En **Cifrado**, realice alguna de la siguientes acciones:
    * Seleccione **Seleccionar de Key Vault** y seleccione un almacén de claves y una clave existentes, o bien **Crear nuevo**. La clave que seleccione no tiene versiones y permite la rotación automática de claves.
    * Seleccione **Escribir el URI de la clave** y proporcione un identificador de clave directamente. Puede proporcionar un URI de clave con versión (para una clave que se debe rotar manualmente) o un URI de clave sin versión (que habilita la rotación automática de claves).
1. Complete la selección de claves y seleccione **Guardar**.

## <a name="revoke-key"></a>Revocación de clave

Para revocar la clave de cifrado administrada por el cliente, cambie la directiva de acceso o los permisos en el almacén de claves o elimine la clave. Por ejemplo, use el comando [az keyvault delete-policy][az-keyvault-delete-policy] para cambiar la directiva de acceso de la identidad administrada usada por el registro:

```azurecli
az keyvault delete-policy \
  --resource-group <resource-group-name> \
  --name <key-vault-name> \
  --object-id $identityPrincipalID
```

Al revocar la clave realmente, se bloquea el acceso a todos los datos del registro, ya que este no puede acceder a la clave de cifrado. Si se habilita el acceso a la clave o se restaura la clave eliminada, el registro toma la clave para que se pueda acceder de nuevo a los datos cifrados del registro.

## <a name="advanced-scenario-key-vault-firewall"></a>Escenario avanzado: Firewall de Key Vault

Es posible que quiera almacenar la clave de cifrado mediante una instancia de Azure Key Vault existente configurada con un [firewall de Key Vault](../key-vault/general/network-security.md), que deniegue el acceso público y solo permita el punto de conexión privado o las redes virtuales seleccionadas. 

En este escenario, primero cree una nueva identidad asignada por el usuario, un almacén de claves y un registro de contenedor cifrados con una clave administrada por el cliente, mediante la [CLI de Azure](#enable-customer-managed-key---cli), el [portal](#enable-customer-managed-key---portal) o una [plantilla](#enable-customer-managed-key---template). En las secciones anteriores de este artículo se describen los pasos detallados.
   > [!NOTE]
   > El nuevo almacén de claves se implementa fuera del firewall. Solo se usa de forma temporal para almacenar la clave administrada por el cliente.

Después de crear el registro, continúe con los pasos siguientes. Consulte los detalles en las secciones siguientes.

1. Habilite la identidad asignada por el sistema del registro.
1. Conceda los permisos de identidad asignados por el sistema a las claves de acceso del almacén de claves restringido con el firewall de Key Vault.
1. Asegúrese de que el firewall de Key Vault permite ignorar servicios de confianza. Actualmente, Azure Container Registry solo puede omitir el firewall cuando usa su identidad administrada por el sistema. 
1. Gire la clave administrada por el cliente. Para ello, seleccione una en el almacén de claves que esté restringida con el firewall de Key Vault.
1. Cuando ya no lo necesite, puede eliminar el almacén de claves que se creó fuera del firewall.


### <a name="step-1---enable-registrys-system-assigned-identity"></a>Paso 1: Habilitar la identidad asignada por el sistema del registro

1. En el portal, vaya al registro.
1. Seleccione **Configuración** >  **Identidad**.
1. En **Asignado por el sistema**, establezca **Estado** en **Activado**. Seleccione **Guardar**.
1. Copie el **Identificador de objeto** de la identidad.

### <a name="step-2---grant-system-assigned-identity-access-to-your-key-vault"></a>Paso 2: Conceder acceso a la identidad asignada por el sistema al almacén de claves

1. En el portal, vaya al almacén de claves.
1. Seleccione **Configuración** > **Directivas de acceso > +Agregar directiva de acceso**.
1. Seleccione **Permisos de clave** y **Obtener**, **Desencapsular clave** y **Encapsular clave**.
1. Elija **Seleccionar la entidad de seguridad** y busque el identificador de objeto de la identidad administrada asignada por el sistema, o el nombre del registro.  
1. Seleccione **Agregar** y después **Guardar**.

### <a name="step-3---enable-key-vault-bypass"></a>Paso 3: Habilitar la omisión del almacén de claves

Para acceder a un almacén de claves configurado con un firewall de Key Vault, el registro debe omitir el firewall. Configure el almacén de claves para permitir el acceso de cualquier [servicio de confianza](../key-vault/general/overview-vnet-service-endpoints.md#trusted-services). Azure Container Registry es uno de los servicios de confianza.

1. En el portal, vaya al almacén de claves.
1. Seleccione **Configuración** > **Redes**.
1. Confirme, actualice o agregue la configuración de la red virtual. Para conocer los pasos detallados, vea [Configuración de firewalls y redes virtuales de Azure Key Vault](../key-vault/general/network-security.md).
1. En **¿Quiere permitir que los servicios de confianza de Microsoft puedan omitir este firewall?** , seleccione **Sí**. 

### <a name="step-4---rotate-the-customer-managed-key"></a>Paso 4: Girar la clave administrada por el cliente

Después de completar los pasos anteriores, gire a una nueva clave almacenada en el almacén de claves detrás de un firewall.

1. En el portal, vaya al registro.
1. En **Configuración**, seleccione **Cifrado** > **Cambiar clave**.
1. En **Identidad**, seleccione **Asignado por el sistema**.
1. Elija **Seleccionar de Key Vault** y seleccione el nombre del almacén de claves que se encuentra detrás de un firewall.
1. Seleccione una clave existente o la opción **Crear nuevo**. La clave que seleccione no tiene versiones y permite la rotación automática de claves.
1. Complete la selección de claves y seleccione **Guardar**.

## <a name="troubleshoot"></a>Solución de problemas

### <a name="removing-managed-identity"></a>Eliminación de una identidad administrada


Si intenta quitar una identidad asignada por el usuario o una asignada por el sistema de un registro que se usa para configurar el cifrado, es posible que vea un mensaje de error similar al siguiente:
 
```
Azure resource '/subscriptions/xxxx/resourcegroups/myGroup/providers/Microsoft.ContainerRegistry/registries/myRegistry' does not have access to identity 'xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx' Try forcibly adding the identity to the registry <registry name>. For more information on bring your own key, please visit 'https://aka.ms/acr/cmk'.
```
 
Tampoco podrá cambiar (girar) la clave de cifrado. Los pasos de resolución dependen del tipo de identidad que se usa para el cifrado.

**Identidad asignada por el usuario**

Si se produce este problema con una identidad asignada por el usuario, reasigne primero la identidad con el GUID que se muestra en el mensaje de error. Por ejemplo:

```azurecli
az acr identity assign -n myRegistry --identities xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx
```
        
Después, tras cambiar la clave y asignar otra identidad, puede quitar la identidad asignada por el usuario original.

**Identidad asignada por el sistema**

Si se produce este problema con una identidad asignada por el sistema, [cree una incidencia de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/) para solicitar ayuda para restaurar la identidad.


## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre el [cifrado en reposo en Azure](../security/fundamentals/encryption-atrest.md).
* Obtenga más información sobre las directivas de acceso y la [Protección del acceso a un almacén de claves](../key-vault/general/security-overview.md).


<!-- LINKS - external -->

<!-- LINKS - internal -->

[az-feature-register]: /cli/azure/feature#az_feature_register
[az-feature-show]: /cli/azure/feature#az_feature_show
[az-group-create]: /cli/azure/group#az_group_create
[az-identity-create]: /cli/azure/identity#az_identity_create
[az-feature-register]: /cli/azure/feature#az_feature_register
[az-deployment-group-create]: /cli/azure/deployment/group#az_deployment_group_create
[az-keyvault-create]: /cli/azure/keyvault#az_keyvault_create
[az-keyvault-key-create]: /cli/azure/keyvault/key#az_keyvault_key_create
[az-keyvault-key]: /cli/azure/keyvault/key
[az-keyvault-set-policy]: /cli/azure/keyvault#az_keyvault_set_policy
[az-keyvault-delete-policy]: /cli/azure/keyvault#az_keyvault_delete_policy
[az-resource-show]: /cli/azure/resource#az_resource_show
[az-acr-create]: /cli/azure/acr#az_acr_create
[az-acr-show]: /cli/azure/acr#az_acr_show
[az-acr-encryption-rotate-key]: /cli/azure/acr/encryption#az_acr_encryption_rotate_key
[az-acr-encryption-show]: /cli/azure/acr/encryption#az_acr_encryption_show
