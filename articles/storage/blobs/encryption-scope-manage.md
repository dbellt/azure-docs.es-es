---
title: Creación y administración de ámbitos de cifrado
description: Aprenda a crear un ámbito de cifrado para aislar los datos de blobs en el nivel de contenedor o blob.
services: storage
author: tamram
ms.service: storage
ms.date: 03/26/2021
ms.topic: conceptual
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: 656443b0bc9d0e45f43634b1b4c21145de7a5bb5
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107792550"
---
# <a name="create-and-manage-encryption-scopes"></a>Creación y administración de ámbitos de cifrado

Los ámbitos de cifrado permiten administrar el cifrado en un blob o contenedor individual. Se pueden usar ámbitos de cifrado para crear límites seguros entre los datos que residen en la misma cuenta de almacenamiento, pero que pertenecen a clientes distintos. Para obtener más información sobre los ámbitos de cifrado, vea [Ámbitos de cifrado para Blob Storage](encryption-scope-overview.md).

En este artículo se muestra cómo crear un ámbito de cifrado. También se muestra cómo especificar un ámbito de cifrado al crear un blob o un contenedor.

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

## <a name="create-an-encryption-scope"></a>Creación de un ámbito de cifrado

Puede crear un ámbito de cifrado que esté protegido con una clave administrada por Microsoft o con una clave administrada por el cliente que se almacena en Azure Key Vault o en el modelo de seguridad de hardware administrado (HSM) de Azure Key Vault (versión preliminar). Para crear un ámbito de cifrado con una clave administrada por el cliente, en primer lugar, debe crear un almacén de claves o un HSM administrado y agregar la clave que quiere usar para el ámbito. El almacén de claves o el HSM administrado deben tener la protección de purga habilitada y deben estar en la misma región que la cuenta de almacenamiento.

Un ámbito de cifrado se habilita automáticamente al crearlo. Después de crearlo, puede especificarlo al crear un blob. También se puede especificar un ámbito de cifrado predeterminado al crear un contenedor, que se aplica automáticamente a todos los blobs del contenedor.

# <a name="portal"></a>[Portal](#tab/portal)

Para crear un ámbito de cifrado en Azure Portal, siga estos pasos:

1. Vaya a la cuenta de almacenamiento en Azure Portal.
1. Seleccione el valor **Cifrado**.
1. Seleccione la pestaña **Ámbitos de cifrado**.
1. Haga clic en el botón **Agregar** para agregar un ámbito de cifrado nuevo.
1. En el panel **Crear un ámbito de cifrado**, escriba un nombre para el ámbito nuevo.
1. Seleccione el tipo de compatibilidad de clave de cifrado deseado, ya sea **Claves administradas por Microsoft** o **Claves administradas por el cliente**.
    - Si ha seleccionado **Microsoft-managed keys** (Claves administradas por Microsoft), haga clic en **Crear** para crear el ámbito de cifrado.
    - Si ha seleccionado **Claves administradas por el cliente**, seleccione una suscripción y especifique un almacén de claves o un HSM administrado y una clave para usar en este ámbito de cifrado, tal como se muestra en la imagen siguiente.

    :::image type="content" source="media/encryption-scope-manage/create-encryption-scope-customer-managed-key-portal.png" alt-text="Captura de pantalla en la que se muestra cómo crear un ámbito de cifrado en Azure Portal":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para crear un ámbito de cifrado con PowerShell, instale la versión 3.4.0 del módulo [Az.Storage](https://www.powershellgallery.com/packages/Az.Storage) de PowerShell, o una posterior.

### <a name="create-an-encryption-scope-protected-by-microsoft-managed-keys"></a>Creación de un ámbito de cifrado protegido mediante claves administradas por Microsoft

Para crear un nuevo ámbito de cifrado protegido mediante claves administradas por Microsoft, llame al comando **New-AzStorageEncryptionScope** con el parámetro `-StorageEncryption`.

No olvide reemplazar los valores del marcador de posición en el ejemplo por los propios:

```powershell
$rgName = "<resource-group>"
$accountName = "<storage-account>"
$scopeName1 = "customer1scope"

New-AzStorageEncryptionScope -ResourceGroupName $rgName `
    -AccountName $accountName `
    -EncryptionScopeName $scopeName1 `
    -StorageEncryption
```

### <a name="create-an-encryption-scope-protected-by-customer-managed-keys"></a>Creación de un ámbito de cifrado protegido mediante claves administradas por el cliente

Para crear un ámbito de cifrado protegido mediante claves administradas por el cliente almacenadas en un almacén de claves o un HSM administrado, configure en primer lugar las claves administradas por el cliente para la cuenta de almacenamiento. Debe asignar una identidad administrada a la cuenta de almacenamiento y, después, usar esta identidad para configurar la directiva de acceso para el almacén de claves o el HSM administrado, de forma que la cuenta de almacenamiento tenga permisos de acceso.

Para configurar las claves administradas por el cliente para usarlas con un ámbito de cifrado, la protección de purga debe estar habilitada en el almacén de claves o en el HSM administrado. El almacén de claves o el HSM administrado deben estar en la misma región que la cuenta de almacenamiento.

No olvide reemplazar los valores del marcador de posición en el ejemplo por los propios:

```powershell
$rgName = "<resource-group>"
$accountName = "<storage-account>"
$keyVaultName = "<key-vault>"
$keyUri = "<key-uri>"
$scopeName2 = "customer2scope"

# Assign a system managed identity to the storage account.
$storageAccount = Set-AzStorageAccount -ResourceGroupName $rgName `
    -Name $accountName `
    -AssignIdentity

# Configure the access policy for the key vault.
Set-AzKeyVaultAccessPolicy `
    -VaultName $keyVaultName `
    -ObjectId $storageAccount.Identity.PrincipalId `
    -PermissionsToKeys wrapkey,unwrapkey,get
```

Después, llame al comando **New-AzStorageEncryptionScope** con el parámetro `-KeyvaultEncryption` y especifique el URI de la clave. La inclusión de la versión de la clave en el URI de clave es opcional. Si omite la versión de la clave, el ámbito de cifrado usará automáticamente la versión más reciente de la clave. Si incluye la versión de la clave, debe actualizarla manualmente para usar otra versión.

No olvide reemplazar los valores del marcador de posición en el ejemplo por los propios:

```powershell
New-AzStorageEncryptionScope -ResourceGroupName $rgName `
    -AccountName $accountName `
    -EncryptionScopeName $scopeName2 `
    -KeyUri $keyUri `
    -KeyvaultEncryption
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/cli)

Para crear un ámbito de cifrado con la CLI de Azure, instale primero la versión 2.20.0 de la CLI de Azure, o una posterior.

### <a name="create-an-encryption-scope-protected-by-microsoft-managed-keys"></a>Creación de un ámbito de cifrado protegido mediante claves administradas por Microsoft

Para crear un nuevo ámbito de cifrado protegido mediante claves administradas por Microsoft, llame al comando [az storage account encryption-scope create](/cli/azure/storage/account/encryption-scope#az_storage_account_encryption_scope_create) y especifique el parámetro `--key-source` como `Microsoft.Storage`. Recuerde reemplazar los valores de marcador de posición por los propios:

```azurecli-interactive
az storage account encryption-scope create \
    --resource-group <resource-group> \
    --account-name <storage-account> \
    --name <scope> \
    --key-source Microsoft.Storage
```

### <a name="create-an-encryption-scope-protected-by-customer-managed-keys"></a>Creación de un ámbito de cifrado protegido mediante claves administradas por el cliente

Para crear un nuevo ámbito de cifrado protegido mediante claves administradas por Microsoft, llame al comando [az storage account encryption-scope create](/cli/azure/storage/account/encryption-scope#az_storage_account_encryption_scope_create) y especifique el parámetro `--key-source` como `Microsoft.Storage`. Recuerde reemplazar los valores de marcador de posición por los propios:

Para crear un ámbito de cifrado protegido mediante claves administradas por el cliente en un almacén de claves o un HSM administrado, configure en primer lugar las claves administradas por el cliente para la cuenta de almacenamiento. Debe asignar una identidad administrada a la cuenta de almacenamiento y, después, usar esta identidad a fin de configurar la directiva de acceso para el almacén de claves, de forma que la cuenta de almacenamiento tenga permisos de acceso. Para más información, consulte [Claves administradas por el cliente para el cifrado de Azure Storage](../common/customer-managed-keys-overview.md).

Para configurar las claves administradas por el cliente para usarlas con un ámbito de cifrado, la protección de purga debe estar habilitada en el almacén de claves o en el HSM administrado. El almacén de claves o el HSM administrado deben estar en la misma región que la cuenta de almacenamiento.

No olvide reemplazar los valores del marcador de posición en el ejemplo por los propios:

```azurecli-interactive
az login
az account set --subscription <subscription-id>

az storage account update \
    --name <storage-account> \
    --resource-group <resource_group> \
    --assign-identity

storage_account_principal=$(az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query identity.principalId \
    --output tsv)

az keyvault set-policy \
    --name <key-vault> \
    --resource-group <resource_group> \
    --object-id $storage_account_principal \
    --key-permissions get unwrapKey wrapKey
```

Después, llame al comando **az storage account encryption-scope create** con el parámetro `--key-uri` y especifique el URI de la clave. La inclusión de la versión de la clave en el URI de clave es opcional. Si omite la versión de la clave, el ámbito de cifrado usará automáticamente la versión más reciente de la clave. Si incluye la versión de la clave, debe actualizarla manualmente para usar otra versión.

No olvide reemplazar los valores del marcador de posición en el ejemplo por los propios:

```azurecli-interactive
az storage account encryption-scope create \
    --resource-group <resource-group> \
    --account-name <storage-account> \
    --name <scope> \
    --key-source Microsoft.KeyVault \
    --key-uri <key-uri>
```

---

Para obtener información sobre cómo configurar el cifrado de Azure Storage con claves administradas por el cliente en un almacén de claves o HSM administrado, consulte los siguientes artículos:

- [Configuración del cifrado con claves administradas por el cliente almacenadas en Azure Key Vault](../common/customer-managed-keys-configure-key-vault.md)
- [Configuración del cifrado con claves administradas por el cliente almacenadas en HSM administrado de Azure Key Vault (versión preliminar)](../common/customer-managed-keys-configure-key-vault-hsm.md).

## <a name="list-encryption-scopes-for-storage-account"></a>Enumeración de los ámbitos de cifrado para la cuenta de almacenamiento

# <a name="portal"></a>[Portal](#tab/portal)

Para ver los ámbitos de cifrado de una cuenta de almacenamiento en Azure Portal, vaya a la opción **Ámbitos de cifrado** de la cuenta de almacenamiento. En este panel, se puede habilitar un ámbito de cifrado, deshabilitarlo o cambiarle la clave.

:::image type="content" source="media/encryption-scope-manage/list-encryption-scopes-portal.png" alt-text="Captura de pantalla en la que se muestra una lista de ámbitos de cifrado en Azure Portal":::

Para ver los detalles de una clave administrada por el cliente, incluidos el URI y la versión de la clave y si la versión de la clave se actualiza automáticamente, siga el vínculo de la columna de **Clave**.

:::image type="content" source="media/encryption-scope-manage/customer-managed-key-details-portal.png" alt-text="Captura de pantalla que muestra los detalles de una clave utilizada con un ámbito de cifrado":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para enumerar los ámbitos de cifrado disponibles para una cuenta de almacenamiento con PowerShell, llame al comando **Get-AzStorageEncryptionScope**. No olvide reemplazar los valores del marcador de posición en el ejemplo por los propios:

```powershell
Get-AzStorageEncryptionScope -ResourceGroupName $rgName `
    -StorageAccountName $accountName
```

Para enumerar todos los ámbitos de cifrado de un grupo de recursos por cuenta de almacenamiento, use la sintaxis de canalización:

```powershell
Get-AzStorageAccount -ResourceGroupName $rgName | Get-AzStorageEncryptionScope
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/cli)

Para enumerar los ámbitos de cifrado disponibles para una cuenta de almacenamiento con la CLI de Azure, llame al comando [az storage account encryption-scope list](/cli/azure/storage/account/encryption-scope#az_storage_account_encryption_scope_list). No olvide reemplazar los valores del marcador de posición en el ejemplo por los propios:

```azurecli-interactive
az storage account encryption-scope list \
    --account-name <storage-account> \
    --resource-group <resource-group>
```

---

## <a name="create-a-container-with-a-default-encryption-scope"></a>Creación de un contenedor con un ámbito de cifrado predeterminado

Al crear un contenedor, se puede especificar un ámbito de cifrado predeterminado. De forma predeterminada, los blobs de ese contenedor usarán ese ámbito.

Se puede crear un blob individual con su propio ámbito de cifrado, a menos que el contenedor esté configurado de tal forma que requiera que todos los blobs usen el ámbito predeterminado. Para obtener más información, consulte [Ámbitos de cifrado para contenedores y blobs](encryption-scope-overview.md#encryption-scopes-for-containers-and-blobs).

# <a name="portal"></a>[Portal](#tab/portal)

Para crear un contenedor con un ámbito de cifrado predeterminado en Azure Portal, primero se debe crear el ámbito de cifrado tal y como se describe en [Creación de un ámbito de cifrado](#create-an-encryption-scope). Después, siga estos pasos para crear el contenedor:

1. Vaya a la lista de contenedores de la cuenta de almacenamiento y seleccione el botón **Agregar** para crear un contenedor nuevo.
1. Expanda la configuración **Avanzado** en el panel **Nuevo contenedor**.
1. En la lista desplegable **Ámbito de cifrado**, seleccione el ámbito de cifrado predeterminado para el contenedor.
1. Para requerir que todos los blobs del contenedor usen el ámbito de cifrado predeterminado, seleccione la casilla para **Usar este ámbito de cifrado para todos los blobs del contenedor**. Si esta casilla está seleccionada, un blob individual del contenedor no podrá invalidar el ámbito de cifrado predeterminado.

    :::image type="content" source="media/encryption-scope-manage/create-container-default-encryption-scope.png" alt-text="Captura de pantalla en la que se muestra el contenedor con el ámbito de cifrado predeterminado":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para crear un contenedor con un ámbito de cifrado predeterminado mediante PowerShell, llame al comando [New-AzStorageContainer](/powershell/module/az.storage/new-azstoragecontainer) y especifique el ámbito del parámetro `-DefaultEncryptionScope`. Para forzar a todos los blobs de un contenedor a usar el ámbito predeterminado del contenedor, establezca el parámetro `-PreventEncryptionScopeOverride` en `true`.

```powershell
$containerName1 = "container1"
$ctx = New-AzStorageContext -StorageAccountName $accountName -UseConnectedAccount

# Create a container with a default encryption scope that cannot be overridden.
New-AzStorageContainer -Name $containerName1 `
    -Context $ctx `
    -DefaultEncryptionScope $scopeName1 `
    -PreventEncryptionScopeOverride $true
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/cli)

Para crear un contenedor con un ámbito de cifrado predeterminado mediante la CLI de Azure, llame al comando [az storage container create](/cli/azure/storage/container#az_storage_container_create) y especifique el ámbito del parámetro `--default-encryption-scope`. Para forzar a todos los blobs de un contenedor a usar el ámbito predeterminado del contenedor, establezca el parámetro `--prevent-encryption-scope-override` en `true`.

En el ejemplo siguiente se usa la cuenta de Azure AD para autorizar la operación de creación del contenedor. También se puede usar la clave de acceso de la cuenta. Para más información, consulte el artículo en el que se explica cómo [autorizar el acceso a los datos de blobs o colas con la CLI de Azure](./authorize-data-operations-cli.md).

```azurecli-interactive
az storage container create \
    --account-name <storage-account> \
    --resource-group <resource-group> \
    --name <container> \
    --default-encryption-scope <scope> \
    --prevent-encryption-scope-override true \
    --auth-mode login
```

---

Si un cliente intenta especificar un ámbito al cargar un blob en un contenedor que tiene un ámbito de cifrado predeterminado y el contenedor está configurado para evitar que los blobs invaliden el ámbito predeterminado, se produce un error en la operación con un mensaje que indica que la directiva de cifrado del contenedor prohíbe la solicitud.

## <a name="upload-a-blob-with-an-encryption-scope"></a>Carga de un blob con un ámbito de cifrado

Al cargar un blob, puede especificar un ámbito de cifrado para ese blob o usar el ámbito de cifrado predeterminado para el contenedor, si se ha especificado uno.

# <a name="portal"></a>[Portal](#tab/portal)

Para cargar un blob con un ámbito de cifrado a través de Azure Portal, primero se debe crear el ámbito de cifrado tal y como se describe en [Creación de un ámbito de cifrado](#create-an-encryption-scope). Después, siga estos pasos para crear el blob:

1. Vaya hasta el contenedor en el que quiere cargar el blob.
1. Seleccione el botón **Cargar** y busque el blob que se va a cargar.
1. Expanda la configuración **Avanzado** en el panel **Cargar blob**.
1. Busque la sección desplegable **Ámbito de cifrado**. De forma predeterminada, el blob se crea con el ámbito de cifrado predeterminado para el contenedor, si se ha especificado uno. Si el contenedor requiere que los blobs usen el ámbito de cifrado predeterminado, esta sección está deshabilitada.
1. A fin de especificar un ámbito diferente para el blob que se está cargando, seleccione **Elegir un ámbito existente** y, luego, el ámbito que quiera en la lista desplegable.

    :::image type="content" source="media/encryption-scope-manage/upload-blob-encryption-scope.png" alt-text="Captura de pantalla en la que se muestra cómo cargar un blob con un ámbito de cifrado":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para cargar un blob con un ámbito de cifrado a través de PowerShell, llame al comando [Set-AzStorageBlobContent](/powershell/module/az.storage/set-azstorageblobcontent) y proporcione el ámbito de cifrado para el blob.

```powershell
$containerName2 = "container2"
$localSrcFile = "C:\temp\helloworld.txt"
$ctx = New-AzStorageContext -StorageAccountName $accountName -UseConnectedAccount

# Create a new container with no default scope defined.
New-AzStorageContainer -Name $containerName2 -Context $ctx

# Upload a block upload with an encryption scope specified.
Set-AzStorageBlobContent -Context $ctx `
    -Container $containerName2 `
    -File $localSrcFile `
    -Blob "helloworld.txt" `
    -BlobType Block `
    -EncryptionScope $scopeName2
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/cli)

Para cargar un blob con un ámbito de cifrado a través de la CLI de Azure, llame al comando [az storage blob upload](/cli/azure/storage/blob#az_storage_blob_upload) y proporcione el ámbito de cifrado para el blob.

Si usa Azure Cloud Shell, siga los pasos descritos en [Cargar un blob](storage-quickstart-blobs-cli.md#upload-a-blob) para crear un archivo en el directorio raíz. Después, puede cargar este archivo en un blob mediante el ejemplo siguiente.

```azurecli-interactive
az storage blob upload \
    --account-name <storage-account> \
    --container-name <container> \
    --file <file> \
    --name <file> \
    --encryption-scope <scope>
```

---

## <a name="change-the-encryption-key-for-a-scope"></a>Cambio de la clave de cifrado de un ámbito

Para cambiar la clave que protege un ámbito de cifrado de una clave administrada por Microsoft a una clave administrada por el cliente, primero debe asegurarse de que ha habilitado las claves administradas por el cliente con Azure Key Vault o HSM de Key Vault para la cuenta de almacenamiento. Para más información, consulte [Configuración del cifrado con claves administradas por el cliente almacenadas en Azure Key Vault](../common/customer-managed-keys-configure-key-vault.md).

# <a name="portal"></a>[Portal](#tab/portal)

Para cambiar la clave que protege un ámbito en Azure Portal, siga estos pasos:

1. Vaya a la pestaña **Ámbitos de cifrado** para ver la lista de ámbitos de cifrado de la cuenta de almacenamiento.
1. Seleccione el botón **Más** situado junto al ámbito que quiera modificar.
1. En el panel **Edit encryption scope** (Editar ámbito de cifrado), se puede cambiar el tipo de cifrado de clave administrada por Microsoft a clave administrada por el cliente, o viceversa.
1. Para seleccionar una nueva clave administrada por el cliente, seleccione **Usar una nueva clave** y especifique el almacén de claves, la clave y la versión de la clave.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para cambiar la clave que protege un ámbito de cifrado de una clave administrada por el cliente a una clave administrada por Microsoft con PowerShell, llame al comando **Update-AzStorageEncryptionScope** y pase el parámetro `-StorageEncryption`:

```powershell
Update-AzStorageEncryptionScope -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -EncryptionScopeName $scopeName2 `
    -StorageEncryption
```

Después, llame al comando **Update-AzStorageEncryptionScope** y pase los parámetros `-KeyUri` y `-KeyvaultEncryption`:

```powershell
Update-AzStorageEncryptionScope -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -EncryptionScopeName $scopeName1 `
    -KeyUri $keyUri `
    -KeyvaultEncryption
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/cli)

Para cambiar la clave que protege un ámbito de cifrado de una clave administrada por el cliente a una clave administrada por Microsoft con la CLI de Azure, llame al comando [az storage account encryption-scope update](/cli/azure/storage/account/encryption-scope#az_storage_account_encryption_scope_update) y pase el parámetro `--key-source` con el valor `Microsoft.Storage`:

```azurecli-interactive
az storage account encryption-scope update \
    --account-name <storage-account> \
    --resource-group <resource-group>
    --name <scope> \
    --key-source Microsoft.Storage
```

Después, llame al comando **az storage account encryption-scope update**, pase el parámetro `--key-uri` y también el parámetro `--key-source` con el valor `Microsoft.KeyVault`:

```powershell
az storage account encryption-scope update \
    --resource-group <resource-group> \
    --account-name <storage-account> \
    --name <scope> \
    --key-source Microsoft.KeyVault \
    --key-uri <key-uri>
```

---

## <a name="disable-an-encryption-scope"></a>Deshabilitación de un ámbito de cifrado

Cuando se deshabilita un ámbito de cifrado, ya no se le facturará. Deshabilite los ámbitos de cifrado que no sean necesarios para evitar cargos innecesarios. Para más información, consulte [Cifrado de Azure Storage para datos en reposo](../common/storage-service-encryption.md).

# <a name="portal"></a>[Portal](#tab/portal)

Para deshabilitar un ámbito de cifrado en Azure Portal, vaya a la configuración **Ámbitos de cifrado** de la cuenta de almacenamiento, seleccione el ámbito de cifrado deseado y luego **Deshabilitar**.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para deshabilitar un ámbito de cifrado con PowerShell, llame al comando Update-AzStorageEncryptionScope e incluya el parámetro `-State` con un valor de `disabled`, tal como se muestra en el ejemplo siguiente. Para volver a habilitar un ámbito de cifrado, llame al mismo comando con el parámetro `-State` establecido en `enabled`. No olvide reemplazar los valores del marcador de posición en el ejemplo por los propios:

```powershell
Update-AzStorageEncryptionScope -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -EncryptionScopeName $scopeName1 `
    -State disabled
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/cli)

Para deshabilitar un ámbito de cifrado con la CLI de Azure, llame al comando [az storage account encryption-scope update](/cli/azure/storage/account/encryption-scope#az_storage_account_encryption_scope_update) e incluya el parámetro `--state` con un valor de `Disabled`, tal como se muestra en el ejemplo siguiente. Para volver a habilitar un ámbito de cifrado, llame al mismo comando con el parámetro `--state` establecido en `Enabled`. No olvide reemplazar los valores del marcador de posición en el ejemplo por los propios:

```azurecli-interactive
az storage account encryption-scope update \
    --account-name <storage-account> \
    --resource-group <resource-group> \
    --name <scope> \
    --state Disabled
```

> [!IMPORTANT]
> No es posible eliminar un ámbito de cifrado. Para evitar costos inesperados, asegúrese de deshabilitar los ámbitos de cifrado que no necesite actualmente.

---

## <a name="next-steps"></a>Pasos siguientes

- [Cifrado de Azure Storage para datos en reposo](../common/storage-service-encryption.md)
- [Ámbitos de cifrado para Blob Storage](encryption-scope-overview.md)
- [Claves administradas por el cliente para el cifrado de Azure Storage](../common/customer-managed-keys-overview.md)