---
title: Configuración del cifrado con claves administradas por el cliente almacenadas en Azure Key Vault
titleSuffix: Azure Storage
description: Obtenga información sobre cómo configurar el cifrado de Azure Storage con claves administradas por el cliente almacenadas en Azure Key Vault mediante Azure Portal, PowerShell o la CLI de Azure.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 02/16/2021
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: 77a01a270f47ddacb71962188e7fedd0a0a9f6d0
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107790444"
---
# <a name="configure-encryption-with-customer-managed-keys-stored-in-azure-key-vault"></a>Configuración del cifrado con claves administradas por el cliente almacenadas en Azure Key Vault

Azure Storage cifra todos los datos de las cuentas de almacenamiento en reposo. De manera predeterminada, los datos se cifran con claves administradas por Microsoft. Para tener un mayor control sobre las claves de cifrado, puede administrar sus propias claves. Las claves administradas por el cliente deben estar almacenadas en Azure Key Vault o en el modelo de seguridad de hardware (HSM) administrado de Azure Key Vault (versión preliminar).

En este artículo se muestra cómo configurar el cifrado con claves administradas por el cliente almacenadas en un almacén de claves mediante Azure Portal, PowerShell o la CLI de Azure. Para obtener información sobre cómo configurar el cifrado con las claves administradas por el cliente almacenadas en un HSM administrado, consulte [Configuración del cifrado con claves administradas por el cliente almacenadas en HSM administrado de Azure Key Vault (versión preliminar)](customer-managed-keys-configure-key-vault-hsm.md).

> [!NOTE]
> Azure Key Vault y HSM administrado de Azure Key Vault admiten las mismas API e interfaces de administración para la configuración.

## <a name="configure-a-key-vault"></a>Configuración de un almacén de claves

Puede usar un almacén de claves nuevo o existente para almacenar las claves administradas por el cliente. La cuenta de almacenamiento y el almacén de claves deben estar en la misma región, pero pueden estar en distintas suscripciones.

El uso de claves administradas del cliente con el cifrado de Azure Storage requiere la eliminación temporal y la protección de purga estén habilitadas en el almacén de claves. La eliminación temporal está habilitada de manera predeterminada al crear un almacén de claves y no se puede deshabilitar. Puede habilitar la protección de purga cuando cree el almacén de claves o después de crearlo.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para obtener información sobre cómo crear un almacén de claves con Azure Portal, consulte [Inicio rápido: Creación de un almacén de claves mediante Azure Portal](../../key-vault/general/quick-create-portal.md). Al crear el almacén de claves, seleccione **Habilitar protección de purga**, como se muestra en la siguiente imagen.

:::image type="content" source="media/customer-managed-keys-configure-key-vault/configure-key-vault-portal.png" alt-text="Captura de pantalla que muestra cómo habilitar la protección de purga al crear un almacén de claves":::

Para habilitar la protección de purga en un almacén de claves existente, siga estos pasos:

1. Vaya al almacén de claves en Azure Portal.
1. En **Configuración**, elija **Propiedades**.
1. En la sección **Protección de purga**, elija **Habilitar protección de purga**.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para crear un nuevo almacén de claves con PowerShell, instale la versión 2.0.0 o posterior del módulo [Az.KeyVault](https://www.powershellgallery.com/packages/Az.KeyVault/2.0.0) de PowerShell. Luego llame a [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault) para crear un nuevo almacén de claves. Con la versión 2.0.0 y posteriores del módulo Az.KeyVault, la eliminación temporal está habilitada de forma predeterminada cuando se crea un nuevo almacén de claves.

En el ejemplo siguiente se crea un nuevo almacén de claves con la eliminación temporal y la protección de purga habilitadas. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores.

```powershell
$keyVault = New-AzKeyVault -Name <key-vault> `
    -ResourceGroupName <resource_group> `
    -Location <location> `
    -EnablePurgeProtection
```

Para obtener información sobre cómo habilitar la protección de purga en un almacén de claves existente con PowerShell, consulte [Uso de la eliminación temporal con PowerShell](../../key-vault/general/key-vault-recovery.md).

A continuación, asigne una identidad administrada asignada por el sistema a la cuenta de almacenamiento. Usará esta identidad administrada para conceder los permisos de la cuenta de almacenamiento para obtener acceso al almacén de claves. Para obtener más información sobre las identidades administradas asignadas por el sistema, consulte [¿Qué son las identidades administradas para los recursos de Azure?](../../active-directory/managed-identities-azure-resources/overview.md)

Para asignar una identidad administrada con PowerShell, llame a [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount):

```powershell
$storageAccount = Set-AzStorageAccount -ResourceGroupName <resource_group> `
    -Name <storage-account> `
    -AssignIdentity
```

Por último, configure la directiva de acceso del almacén de claves, para que la cuenta de almacenamiento tenga permiso para acceder a él. En este paso, usará la identidad administrada que asignó previamente a la cuenta de almacenamiento.

Para establecer la directiva de acceso del almacén de claves, llame a [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy):

```powershell
Set-AzKeyVaultAccessPolicy `
    -VaultName $keyVault.VaultName `
    -ObjectId $storageAccount.Identity.PrincipalId `
    -PermissionsToKeys wrapkey,unwrapkey,get
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para crear un nuevo almacén de claves con la CLI de Azure, llame al comando [az keyvault create](/cli/azure/keyvault#az_keyvault_create). No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```azurecli-interactive
az keyvault create \
    --name <key-vault> \
    --resource-group <resource_group> \
    --location <region> \
    --enable-purge-protection
```

Para obtener información sobre cómo habilitar la protección de purga en un almacén de claves existente con la CLI de Azure, consulte [Uso de la eliminación temporal con la CLI](../../key-vault/general/key-vault-recovery.md).

A continuación, asigne una identidad administrada asignada por el sistema a una cuenta de almacenamiento. Usará esta identidad administrada para conceder los permisos de la cuenta de almacenamiento para obtener acceso al almacén de claves. Para obtener más información sobre las identidades administradas asignadas por el sistema, consulte [¿Qué son las identidades administradas para los recursos de Azure?](../../active-directory/managed-identities-azure-resources/overview.md)

Para asignar una identidad administrada mediante la CLI de Azure, llame a [az storage account update](/cli/azure/storage/account#az_storage_account_update):

```azurecli-interactive
az storage account update \
    --name <storage-account> \
    --resource-group <resource_group> \
    --assign-identity
```

Por último, configure la directiva de acceso del almacén de claves, para que la cuenta de almacenamiento tenga permiso para acceder a él. En este paso, usará la identidad administrada que asignó previamente a la cuenta de almacenamiento.

Para establecer la directiva de acceso del almacén de claves, use [az keyvault set-policy](/cli/azure/keyvault#az_keyvault_set_policy):

```azurecli-interactive
storage_account_principal=$(az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query identity.principalId \
    --output tsv)
az keyvault set-policy \
    --name <key-vault> \
    --resource-group <resource_group>
    --object-id $storage_account_principal \
    --key-permissions get unwrapKey wrapKey
```

---

## <a name="add-a-key"></a>Adición de una clave

A continuación, agregue una clave al almacén de claves.

El cifrado de Azure Storage admite claves RSA y RSA-HSM de los tamaños 2048, 3072 y 4096. Para obtener más información sobre las claves, vea [Acerca de las claves](../../key-vault/keys/about-keys.md).

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para obtener información sobre cómo agregar una clave con Azure Portal, consulte [Inicio rápido: Establecimiento y recuperación de una clave de Azure Key Vault mediante Azure Portal](../../key-vault/keys/quick-create-portal.md).

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para agregar una clave con PowerShell, llame a [Add-AzKeyVaultKey](/powershell/module/az.keyvault/add-azkeyvaultkey). Recuerde reemplazar los valores de marcador de posición entre corchetes por sus propios valores y usar las variables definidas en los ejemplos anteriores.

```powershell
$key = Add-AzKeyVaultKey -VaultName $keyVault.VaultName `
    -Name <key> `
    -Destination 'Software'
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para agregar una clave con la CLI de Azure, llame a [az keyvault key create](/cli/azure/keyvault/key#az_keyvault_key_create). No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores.

```azurecli-interactive
az keyvault key create \
    --name <key> \
    --vault-name <key-vault>
```

---

## <a name="configure-encryption-with-customer-managed-keys"></a>Configuración del cifrado con claves que administra el cliente

A continuación, configurará su cuenta de Azure Storage para usar claves administradas por el cliente con Azure Key Vault y, después, especificará la clave que se va a asociar con la cuenta de almacenamiento.

Al configurar el cifrado con claves administradas por el cliente, puede optar por actualizar automáticamente la versión de la clave que se usa para el cifrado de Azure Storage cuando hay disponible una versión nueva en el almacén de claves asociado. Como alternativa, puede especificar explícitamente la versión de la clave que se usará para el cifrado hasta que la versión de la clave se actualice manualmente.

> [!NOTE]
> Para rotar una clave, cree una nueva versión de la misma en Azure Key Vault. Azure Storage no controla la rotación de la clave en Azure Key Vault, por lo que deberá rotar la clave manualmente o crear una función para hacerlo según una programación.

### <a name="configure-encryption-for-automatic-updating-of-key-versions"></a>Configuración del cifrado para la actualización automática de las versiones de clave

Azure Storage puede actualizar automáticamente la clave administrada por el cliente que se usa para el cifrado, de modo que se use la versión más reciente de esta. Cuando se rote la clave administrada por el cliente en Azure Key Vault, Azure Storage comenzará automáticamente a usar la versión más reciente de la clave para el cifrado.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para configurar las claves administradas por el cliente con la actualización automática de la versión de clave en Azure Portal, siga estos pasos:

1. Vaya a la cuenta de almacenamiento.
1. En la hoja **Configuración** de la cuenta de almacenamiento, haga clic en **Cifrado**. De forma predeterminada, la administración de claves está establecida en **Claves administradas de Microsoft**, como se muestra en la siguiente imagen.

    ![Captura de pantalla del portal que muestra la opción de cifrado](./media/customer-managed-keys-configure-key-vault/portal-configure-encryption-keys.png)

1. Seleccione la opción **Claves administradas de cliente**.
1. Elija la opción **Select from Key Vault** (Seleccionar desde almacén de claves).
1. Elija **Seleccione un almacén de claves y una clave**.
1. Seleccione el almacén de claves que contiene la clave que desea usar.
1. Seleccione la clave en el almacén de claves.

   ![Captura de pantalla que muestra cómo seleccionar el almacén de claves y la clave](./media/customer-managed-keys-configure-key-vault/portal-select-key-from-key-vault.png)

1. Guarde los cambios.

Después de especificar la clave, Azure Portal indica que está habilitada la actualización automática de la versión de la clave y muestra la versión de la clave actualmente en uso para el cifrado.

:::image type="content" source="media/customer-managed-keys-configure-key-vault/portal-auto-rotation-enabled.png" alt-text="Captura de pantalla que muestra habilita la opción de actualización automática de la versión de la clave":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para configurar las claves administradas por el cliente con la actualización automática de la versión de la clave con PowerShell, instale la versión 2.0.0 o posterior del módulo [Az.Storage](https://www.powershellgallery.com/packages/Az.Storage).

Para actualizar automáticamente la versión de una clave administrada por el cliente, omita la versión de la clave al configurar el cifrado con las claves administradas por el cliente para la cuenta de almacenamiento. Llame a [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) para actualizar la configuración de cifrado de la cuenta de almacenamiento, como se muestra en el ejemplo siguiente, e incluya la opción **-KeyvaultEncryption** para habilitar las claves administradas por el cliente para la cuenta de almacenamiento.

Recuerde reemplazar los valores de marcador de posición entre corchetes por sus propios valores y usar las variables definidas en los ejemplos anteriores.

```powershell
Set-AzStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName `
    -AccountName $storageAccount.StorageAccountName `
    -KeyvaultEncryption `
    -KeyName $key.Name `
    -KeyVaultUri $keyVault.VaultUri
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para configurar las claves administradas por el cliente con la actualización automática de la versión de la clave con la CLI de Azure, instale la [versión 2.4.0 de la CLI de Azure](/cli/azure/release-notes-azure-cli#april-21-2020), o cualquier versión posterior. Para más información, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Para actualizar automáticamente la versión de una clave administrada por el cliente, omita la versión de la clave al configurar el cifrado con las claves administradas por el cliente para la cuenta de almacenamiento. Llame a [az storage account update](/cli/azure/storage/account#az_storage_account_update) para actualizar la configuración de cifrado de la cuenta de almacenamiento, como se muestra en el ejemplo siguiente. Incluya el parámetro `--encryption-key-source` y establézcalo en `Microsoft.Keyvault` para habilitar las claves administradas por el cliente para la cuenta.

No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores.

```azurecli-interactive
key_vault_uri=$(az keyvault show \
    --name <key-vault> \
    --resource-group <resource_group> \
    --query properties.vaultUri \
    --output tsv)
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $key_vault_uri
```

---

### <a name="configure-encryption-for-manual-updating-of-key-versions"></a>Configuración del cifrado para la actualización manual de las versiones de la clave

Si prefiere actualizar manualmente la versión de la clave, especifique de manera explícita la versión en el momento en que configure el cifrado con las claves administradas por el cliente. En este caso, Azure Storage no actualizará automáticamente la versión de la clave cuando se cree una nueva versión en el almacén de claves. Para usar una nueva versión de la clave, debe actualizar manualmente la versión usada para el cifrado de Azure Storage.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para configurar las claves administradas por el cliente con la actualización manual de la versión de clave en Azure Portal, especifique el URI de la clave, incluida la versión. Para especificar una clave como URI, siga estos pasos:

1. Para buscar el URI de la clave en Azure Portal, vaya al almacén de claves y seleccione la opción de configuración **Claves**. Seleccione la clave que desee y luego haga clic en ella para ver sus versiones. Seleccione cualquiera de las versiones de clave para ver su configuración.
1. Copie el valor del campo **Identificador de clave**, que proporciona el URI.

    ![Captura de pantalla en que se muestra el URI de la clave del almacén de claves](media/customer-managed-keys-configure-key-vault/portal-copy-key-identifier.png)

1. En las opciones de **Clave de cifrado** de la cuenta de almacenamiento, elija la opción **Escribir el URI de la clave**.
1. Pegue el identificador URI que ha copiado en el campo **URI de clave**. Omita la versión de la clave en el URI para habilitar las actualizaciones automáticas de la versión de la clave.

   ![Captura de pantalla en que se muestra cómo introducir el URI de la clave](./media/customer-managed-keys-configure-key-vault/portal-specify-key-uri.png)

1. Especifique el identificador de la suscripción que contiene el almacén de claves.
1. Guarde los cambios.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para configurar las claves administradas por el cliente con la actualización manual de la versión de la clave, indique de manera explícita la versión de la clave al configurar el cifrado para la cuenta de almacenamiento. Llame a [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) para actualizar la configuración de cifrado de la cuenta de almacenamiento, como se muestra en el ejemplo siguiente, e incluya la opción **-KeyvaultEncryption** para habilitar las claves administradas por el cliente para la cuenta de almacenamiento.

Recuerde reemplazar los valores de marcador de posición entre corchetes por sus propios valores y usar las variables definidas en los ejemplos anteriores.

```powershell
Set-AzStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName `
    -AccountName $storageAccount.StorageAccountName `
    -KeyvaultEncryption `
    -KeyName $key.Name `
    -KeyVersion $key.Version `
    -KeyVaultUri $keyVault.VaultUri
```

Al actualizar manualmente la versión de la clave, deberá actualizar la configuración de cifrado de la cuenta de almacenamiento para que use la nueva versión. En primer lugar, llame a [Get-AzKeyVaultKey](/powershell/module/az.keyvault/get-azkeyvaultkey) para obtener la versión más reciente de la clave. A continuación, llame a [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) para actualizar la configuración de cifrado de la cuenta de almacenamiento y así poder usar la nueva versión de la clave, tal como se muestra en el ejemplo anterior.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para configurar las claves administradas por el cliente con la actualización manual de la versión de la clave, indique de manera explícita la versión de la clave al configurar el cifrado para la cuenta de almacenamiento. Llame a [az storage account update](/cli/azure/storage/account#az_storage_account_update) para actualizar la configuración de cifrado de la cuenta de almacenamiento, como se muestra en el ejemplo siguiente. Incluya el parámetro `--encryption-key-source` y establézcalo en `Microsoft.Keyvault` para habilitar las claves administradas por el cliente para la cuenta.

No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores.

```azurecli-interactive
key_vault_uri=$(az keyvault show \
    --name <key-vault> \
    --resource-group <resource_group> \
    --query properties.vaultUri \
    --output tsv)
key_version=$(az keyvault key list-versions \
    --name <key> \
    --vault-name <key-vault> \
    --query [-1].kid \
    --output tsv | cut -d '/' -f 6)
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-version $key_version \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $key_vault_uri
```

Al actualizar manualmente la versión de la clave, deberá actualizar la configuración de cifrado de la cuenta de almacenamiento para que use la nueva versión. En primer lugar, consulte el URI del almacén de claves mediante una llamada a [az keyvault show](/cli/azure/keyvault#az_keyvault_show) y, para la versión de la clave, llame a [az keyvault key list-versions](/cli/azure/keyvault/key#az_keyvault_key_list-versions). Luego, llame a [az storage account update](/cli/azure/storage/account#az_storage_account_update) para actualizar la configuración de cifrado de la cuenta de almacenamiento y así poder usar la nueva versión de la clave, tal como se muestra en el ejemplo anterior.

---

## <a name="change-the-key"></a>Cambio de la clave

Puede cambiar la clave que usa para el cifrado de Azure Storage en cualquier momento.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para cambiar la clave con Azure Portal, haga lo siguiente:

1. Vaya a la cuenta de almacenamiento y muestre las opciones de configuración de **cifrado**.
1. Seleccione el almacén de claves y elija una clave nueva.
1. Guarde los cambios.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para cambiar la clave con PowerShell, llame a [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) tal y como se muestra en [Configuración del cifrado con claves administradas por el cliente](#configure-encryption-with-customer-managed-keys) y proporcione el nombre y la versión de la nueva clave. Si la nueva clave se encuentra en otro almacén de claves, tiene que actualizar también el URI del almacén de claves.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para cambiar la clave con la CLI de Azure, llame a [az storage account update](/cli/azure/storage/account#az_storage_account_update) tal y como se muestra en [Configuración del cifrado con claves administradas por el cliente](#configure-encryption-with-customer-managed-keys) y proporcione el nombre y la versión de la nueva clave. Si la nueva clave se encuentra en otro almacén de claves, tiene que actualizar también el URI del almacén de claves.

---

## <a name="revoke-customer-managed-keys"></a>Revocación de claves administradas por el cliente

La revocación de una clave administrada por el cliente quita la asociación entre la cuenta de almacenamiento y el almacén de claves.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para revocar las claves administradas por el cliente con Azure Portal, deshabilite la clave tal y como se describe en [Deshabilitación de claves administradas por el cliente](#disable-customer-managed-keys).

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para revocar las claves administradas por el cliente, quite la directiva de acceso del almacén de claves. Para revocar una clave administrada por el cliente con PowerShell, llame al comando [Remove-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/remove-azkeyvaultaccesspolicy), como se muestra en el siguiente ejemplo. Recuerde reemplazar los valores de marcador de posición entre corchetes por sus propios valores y usar las variables definidas en los ejemplos anteriores.

```powershell
Remove-AzKeyVaultAccessPolicy -VaultName $keyVault.VaultName `
    -ObjectId $storageAccount.Identity.PrincipalId `
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para revocar las claves administradas por el cliente, quite la directiva de acceso del almacén de claves. Para revocar una clave administrada por el cliente con la CLI de Azure, llame al comando [az keyvault delete-policy](/cli/azure/keyvault#az_keyvault_delete_policy), como se muestra en el siguiente ejemplo. Recuerde reemplazar los valores de marcador de posición entre corchetes por sus propios valores y usar las variables definidas en los ejemplos anteriores.

```azurecli-interactive
az keyvault delete-policy \
    --name <key-vault> \
    --object-id $storage_account_principal
```

---

## <a name="disable-customer-managed-keys"></a>Deshabilitación de claves administradas por el cliente

Cuando las claves administradas por el cliente se deshabilitan, la cuenta de almacenamiento se vuelve a cifrar con claves administradas por Microsoft.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para deshabilitar claves administradas del cliente en Azure Portal, siga estos pasos:

1. Vaya a la cuenta de almacenamiento y muestre las opciones de configuración de **cifrado**.
1. Anule la selección de la casilla que se encuentra junto al valor **Usar su propia clave**.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para deshabilitar las claves administradas por el cliente con PowerShell, llame a [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) con la opción `-StorageEncryption` en, tal y como se muestra en el ejemplo siguiente. Recuerde reemplazar los valores de marcador de posición entre corchetes por sus propios valores y usar las variables definidas en los ejemplos anteriores.

```powershell
Set-AzStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName `
    -AccountName $storageAccount.StorageAccountName `
    -StorageEncryption  
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para deshabilitar las claves administradas por el cliente con la CLI de Azure, llame [az storage account update](/cli/azure/storage/account#az_storage_account_update) y establezca el `--encryption-key-source parameter` en `Microsoft.Storage`, tal y como se muestra en el ejemplo siguiente. Recuerde reemplazar los valores de marcador de posición entre corchetes por sus propios valores y usar las variables definidas en los ejemplos anteriores.

```azurecli-interactive
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-source Microsoft.Storage
```

---

## <a name="next-steps"></a>Pasos siguientes

- [Cifrado de Azure Storage para datos en reposo](storage-service-encryption.md)
- [Claves administradas por el cliente para el cifrado de Azure Storage](customer-managed-keys-overview.md)
- [Configuración del cifrado con claves administradas por el cliente almacenadas en HSM administrado de Azure Key Vault (versión preliminar)](customer-managed-keys-configure-key-vault-hsm.md)
