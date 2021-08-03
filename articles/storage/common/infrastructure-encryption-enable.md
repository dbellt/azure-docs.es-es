---
title: Habilitación del cifrado de infraestructura para el cifrado doble de datos
titleSuffix: Azure Storage
description: Los clientes que necesiten más seguridad de que sus datos estén seguros también pueden habilitar el cifrado AES de 256 bits en el nivel de infraestructura de Azure Storage. Cuando se habilita el cifrado de infraestructura, los datos de una cuenta de almacenamiento o ámbito de cifrado se cifran dos veces con dos algoritmos de cifrado y dos claves diferentes.
services: storage
author: tamram
ms.service: storage
ms.date: 06/01/2021
ms.topic: conceptual
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 5c87ed5222b5bb95660b6e63e259a2b5e14b7b47
ms.sourcegitcommit: eb20dcc97827ef255cb4ab2131a39b8cebe21258
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111372680"
---
# <a name="enable-infrastructure-encryption-for-double-encryption-of-data"></a>Habilitación del cifrado de infraestructura para el cifrado doble de datos

Azure Storage cifra de forma automática todos los datos de una cuenta de almacenamiento en el nivel de servicio mediante el cifrado AES de 256 bits, uno de los cifrados de bloques más sólidos que hay disponibles y compatible con FIPS 140-2. Los clientes que necesiten más seguridad para que sus datos estén seguros también pueden habilitar el cifrado AES de 256 bits en el nivel de infraestructura de Azure Storage para el cifrado doble. El doble cifrado de los datos de Azure Storage sirve de protección en caso de que uno de los algoritmos de cifrado o las claves puedan estar en peligro. En este escenario, la capa adicional de cifrado también protege los datos.

El cifrado de infraestructura se puede habilitar para toda la cuenta de almacenamiento o para un ámbito de cifrado dentro de una cuenta. Cuando se habilita el cifrado de infraestructura para una cuenta de almacenamiento o un ámbito de cifrado, los datos se cifran dos veces (una vez en el nivel de servicio y otra en el nivel de infraestructura) con dos algoritmos de cifrado y dos claves diferentes.

El cifrado de nivel de servicio permite usar claves administradas por Microsoft o claves administradas por el cliente con Azure Key Vault o el modelo de seguridad de hardware (HSM) administrado de Key Vault (versión preliminar). El cifrado en el nivel de infraestructura se basa en las claves administradas por Microsoft y siempre usa una clave independiente. Para más información sobre la administración de claves con el cifrado de Azure Storage, consulte la sección [Información sobre la administración de claves de cifrado](storage-service-encryption.md#about-encryption-key-management).

Para cifrar dos veces los datos, primero debe crear una cuenta de almacenamiento o un ámbito de cifrado que estén configurados para el cifrado de infraestructura. En este artículo se describe cómo habilitar el cifrado de infraestructura.

## <a name="create-an-account-with-infrastructure-encryption-enabled"></a>Creación de una cuenta con el cifrado de infraestructura habilitado

Para habilitar el cifrado de infraestructura para una cuenta de almacenamiento, debe configurar la cuenta de almacenamiento para que use el cifrado de infraestructura en el momento de crear la cuenta. Una vez que la cuenta se ha creado, no se puede habilitar o deshabilitar el cifrado de infraestructura. La cuenta de almacenamiento debe ser de tipo de uso general v2.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para usar PowerShell con el fin de crear una cuenta de almacenamiento con el cifrado de infraestructura habilitado, siga estos pasos:

1. En Azure Portal, vaya a la página **Cuentas de almacenamiento**.
1. Elija el botón **Agregar** para agregar una cuenta de almacenamiento v2 de uso general.
1. En la pestaña **Avanzado**, busque **Cifrado de infraestructura** y seleccione **Habilitado**.
1. Seleccione **Revisar y crear** para terminar de crear la cuenta de almacenamiento.

    :::image type="content" source="media/infrastructure-encryption-enable/create-account-infrastructure-encryption-portal.png" alt-text="Captura de pantalla que muestra cómo habilitar el cifrado de infraestructura al crear una la cuenta":::

Para comprobar que el cifrado de infraestructura está habilitado para una cuenta de almacenamiento con Azure Portal, siga estos pasos:

1. Vaya a la cuenta de almacenamiento en Azure Portal.
1. En **Configuración**, elija **Cifrado**.

    :::image type="content" source="media/infrastructure-encryption-enable/verify-infrastructure-encryption-portal.png" alt-text="Captura de pantalla que muestra cómo comprobar que el cifrado de infraestructura está habilitado para la cuenta":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para usar PowerShell para crear una cuenta de almacenamiento con el cifrado de infraestructura habilitado, asegúrese de haber instalado la versión 2.2.0 del módulo [Az.Storage PowerShell](https://www.powershellgallery.com/packages/Az.Storage), o cualquier versión posterior. Para más información, vea [Instalar Azure PowerShell](/powershell/azure/install-az-ps).

A continuación, cree una cuenta de almacenamiento v2 de uso general. Para ello, debe llamar al comando [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount). Incluya la opción `-RequireInfrastructureEncryption` para habilitar el cifrado de infraestructura.

En el ejemplo siguiente se muestra cómo crear una cuenta de almacenamiento v2 de uso general configurada para el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) y que tenga el cifrado de infraestructura habilitado para poder realizar un cifrado doble de los datos. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```powershell
New-AzStorageAccount -ResourceGroupName <resource_group> `
    -AccountName <storage-account> `
    -Location <location> `
    -SkuName "Standard_RAGRS" `
    -Kind StorageV2 `
    -RequireInfrastructureEncryption
```

Para comprobar que el cifrado de infraestructura está habilitado en una cuenta de almacenamiento, llame al comando [Get-AzStorageAccount](/powershell/module/az.storage/get-azstorageaccount). Este comando devuelve un conjunto de propiedades de la cuenta de almacenamiento y sus valores. Recupere el campo `RequireInfrastructureEncryption` dentro de la propiedad `Encryption` y compruebe que esté establecida en `True`.

En el ejemplo siguiente se recupera el valor de la propiedad `RequireInfrastructureEncryption`. No olvide reemplazar los valores del marcador de posición entre corchetes angulares por sus propios valores:

```powershell
$account = Get-AzStorageAccount -ResourceGroupName <resource-group> `
    -StorageAccountName <storage-account>
$account.Encryption.RequireInfrastructureEncryption
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para usar la CLI de Azure para crear una cuenta de almacenamiento que tenga el cifrado de infraestructura habilitado, asegúrese de que esté instalada la versión 2.8.0 de la CLI de Azure, o cualquier versión posterior. Para más información, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

A continuación, cree una cuenta de almacenamiento v2 de uso general. Para ello, llame al comando [az storage account create](/cli/azure/storage/account#az_storage_account_create) e incluya `--require-infrastructure-encryption option` para habilitar el cifrado de infraestructura.

En el ejemplo siguiente se muestra cómo crear una cuenta de almacenamiento v2 de uso general configurada para el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) y que tenga el cifrado de infraestructura habilitado para poder realizar un cifrado doble de los datos. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```azurecli-interactive
az storage account create \
    --name <storage-account> \
    --resource-group <resource-group> \
    --location <location> \
    --sku Standard_RAGRS \
    --kind StorageV2 \
    --require-infrastructure-encryption
```

Para comprobar que el cifrado de infraestructura está habilitado en una cuenta de almacenamiento, llame al comando [az storage account show](/cli/azure/storage/account#az-storage-account-show). Este comando devuelve un conjunto de propiedades de la cuenta de almacenamiento y sus valores. Busque el campo `requireInfrastructureEncryption` dentro de la propiedad `encryption` y compruebe que esté establecida en `true`.

En el ejemplo siguiente se recupera el valor de la propiedad `requireInfrastructureEncryption`. No olvide reemplazar los valores del marcador de posición entre corchetes angulares por sus propios valores:

```azurecli-interactive
az storage account show /
    --name <storage-account> /
    --resource-group <resource-group>
```

# <a name="template"></a>[Plantilla](#tab/template)

En el ejemplo siguiente de JSON se crea una cuenta de almacenamiento v2 de uso general configurada para el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) y que tenga el cifrado de infraestructura habilitado para poder realizar un cifrado doble de los datos. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```json
"resources": [
    {
        "type": "Microsoft.Storage/storageAccounts",
        "apiVersion": "2019-06-01",
        "name": "[parameters('<storage-account>')]",
        "location": "[parameters('<location>')]",
        "dependsOn": [],
        "tags": {},
        "sku": {
            "name": "[parameters('Standard_RAGRS')]"
        },
        "kind": "[parameters('StorageV2')]",
        "properties": {
            "accessTier": "[parameters('<accessTier>')]",
            "supportsHttpsTrafficOnly": "[parameters('supportsHttpsTrafficOnly')]",
            "largeFileSharesState": "[parameters('<largeFileSharesState>')]",
            "encryption": {
                "keySource": "Microsoft.Storage",
                "requireInfrastructureEncryption": true,
                "services": {
                    "blob": { "enabled": true },
                    "file": { "enabled": true }
              }
            }
        }
    }
],
```

---

Azure Policy proporciona una directiva integrada para requerir que el cifrado de infraestructura esté habilitado para una cuenta de almacenamiento. Para obtener más información, consulte la sección **Almacenamiento** de las [definiciones de directivas integradas de Azure Policy](../../governance/policy/samples/built-in-policies.md#storage).


## <a name="create-an-encryption-scope-with-infrastructure-encryption-enabled"></a>Creación de un ámbito de cifrado con el cifrado de infraestructura habilitado

Si el cifrado de infraestructura está habilitado para una cuenta, cualquier ámbito de cifrado creado en esa cuenta usa automáticamente el cifrado de infraestructura. Si el cifrado de infraestructura no está habilitado en el nivel de cuenta, tiene la opción de habilitarlo para un ámbito de cifrado en el momento de crear el ámbito. La configuración de cifrado de infraestructura para un ámbito de cifrado no se puede cambiar después de crear el ámbito. Para más información, consulte [Creación de un ámbito de cifrado](../blobs/encryption-scope-manage.md#create-an-encryption-scope).

## <a name="next-steps"></a>Pasos siguientes

- [Cifrado de Azure Storage para datos en reposo](storage-service-encryption.md)
- [Claves administradas por el cliente para el cifrado de Azure Storage](customer-managed-keys-overview.md)
- [Ámbitos de cifrado para Blob Storage](../blobs/encryption-scope-overview.md)
