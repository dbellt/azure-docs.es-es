---
title: Configuración del cifrado con claves administradas por el cliente almacenadas en HSM administrado de Azure Key Vault (versión preliminar)
titleSuffix: Azure Storage
description: Obtenga información sobre cómo configurar el cifrado de Azure Storage con claves administradas por el cliente almacenadas en HSM administrado de Azure Key Vault (versión preliminar) mediante la CLI de Azure.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 03/30/2021
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: ea51c1f7fcfce5b795965eab2f9c03a820a6ab03
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106059367"
---
# <a name="configure-encryption-with-customer-managed-keys-stored-in-azure-key-vault-managed-hsm-preview"></a>Configuración del cifrado con claves administradas por el cliente almacenadas en HSM administrado de Azure Key Vault (versión preliminar)

Azure Storage cifra todos los datos de las cuentas de almacenamiento en reposo. De manera predeterminada, los datos se cifran con claves administradas por Microsoft. Para tener un mayor control sobre las claves de cifrado, puede administrar sus propias claves. Las claves administradas por el cliente deben estar almacenadas en Azure Key Vault o en el modelo de seguridad de hardware (HSM) administrado de Azure Key Vault (versión preliminar). Un HSM administrado de Azure Key Vault es un HSM validado de FIPS 140-2 de nivel 3.

En este artículo se muestra cómo configurar el cifrado con claves administradas por el cliente almacenadas en un HSM administrado mediante la CLI de Azure. Para obtener información sobre cómo configurar el cifrado con las claves administradas por el cliente almacenadas en un almacén de claves, consulte [Configuración del cifrado con claves administradas por el cliente almacenadas en Azure Key Vault](customer-managed-keys-configure-key-vault.md).

> [!IMPORTANT]
>
> El cifrado con claves administradas por el cliente almacenadas en HSM administrado de Azure Key Vault actualmente se encuentra en **VERSIÓN PRELIMINAR**. Consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.
>
> Azure Key Vault y HSM administrado de Azure Key Vault admiten las mismas API e interfaces de administración para la configuración.

## <a name="assign-an-identity-to-the-storage-account"></a>Asignación de una identidad a la cuenta de almacenamiento

En primer lugar, asigne una identidad administrada asignada por el sistema a una cuenta de almacenamiento. Usará esta identidad administrada para conceder los permisos de la cuenta de almacenamiento para acceder al HSM administrado. Para obtener más información sobre las identidades administradas asignadas por el sistema, consulte [¿Qué son las identidades administradas para los recursos de Azure?](../../active-directory/managed-identities-azure-resources/overview.md)

Para asignar una identidad administrada mediante la CLI de Azure, llame al comando [az storage account update](/cli/azure/storage/account#az-storage-account-update). No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```azurecli
az storage account update \
    --name <storage-account> \
    --resource-group <resource_group> \
    --assign-identity
```

## <a name="assign-a-role-to-the-storage-account-for-access-to-the-managed-hsm"></a>Asignación de un rol a la cuenta de almacenamiento para el acceso al HSM administrado

A continuación, asigne el rol **Managed HSM Crypto Service Encryption** a la identidad administrada de la cuenta de almacenamiento para que la cuenta de almacenamiento tenga permisos para el HSM administrado. Microsoft recomienda que limite la asignación del roles al nivel de la clave individual con el fin de conceder la menor cantidad de privilegios posible a la identidad administrada.

Para crear la asignación de roles para la cuenta de almacenamiento, llame a [az key vault role assignment create](/cli/azure/role/assignment#az_role_assignment_create). No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores.
  
```azurecli
storage_account_principal = $(az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query identity.principalId \
    --output tsv)

az keyvault role assignment create \
    --hsm-name <hsm-name> \
    --role "Managed HSM Crypto Service Encryption" \
    --assignee $storage_account_principal \
    --scope /keys/<key-name>
```

## <a name="configure-encryption-with-a-key-in-the-managed-hsm"></a>Configuración del cifrado con una clave en el HSM administrado

Por último, configure el cifrado de Azure Storage con claves administradas por el cliente para usar una clave almacenada en el HSM administrado. Entre los tipos de clave admitidos se incluyen las claves RSA-HSM de los tamaños 2048, 3072 y 4096. Para obtener información sobre cómo crear una clave en un HSM administrado, consulte [Creación de una clave HSM](../../key-vault/managed-hsm/key-management.md#create-an-hsm-key).

Instale la CLI de Azure 2.12.0 o posterior para configurar el cifrado con el fin de usar una clave administrada por el cliente en un HSM administrado. Para más información, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Para actualizar automáticamente la versión de una clave administrada por el cliente, omita la versión de la clave al configurar el cifrado con las claves administradas por el cliente para la cuenta de almacenamiento. Llame a [az storage account update](/cli/azure/storage/account#az_storage_account_update) para actualizar la configuración de cifrado de la cuenta de almacenamiento, como se muestra en el ejemplo siguiente. Incluya `--encryption-key-source parameter` y establézcalo en `Microsoft.Keyvault` para habilitar las claves administradas por el cliente para la cuenta. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores.

```azurecli
hsmurl = $(az keyvault show \
    --hsm-name <hsm-name> \
    --query properties.hsmUri \
    --output tsv)

az storage account update \
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $hsmurl
```

Para actualizar manualmente la versión de una clave administrada por el cliente, incluya la versión de la clave al configurar el cifrado para la cuenta de almacenamiento:

```azurecli-interactive
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-version $key_version \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $hsmurl
```

Al actualizar manualmente la versión de la clave, deberá actualizar la configuración de cifrado de la cuenta de almacenamiento para que use la nueva versión. En primer lugar, consulte el URI del almacén de claves mediante una llamada a [az keyvault show](/cli/azure/keyvault#az-keyvault-show) y, para la versión de la clave, llame a [az keyvault key list-versions](/cli/azure/keyvault/key#az-keyvault-key-list-versions). Luego, llame a [az storage account update](/cli/azure/storage/account#az-storage-account-update) para actualizar la configuración de cifrado de la cuenta de almacenamiento y así poder usar la nueva versión de la clave, tal como se muestra en el ejemplo anterior.

## <a name="next-steps"></a>Pasos siguientes

- [Cifrado de Azure Storage para datos en reposo](storage-service-encryption.md)
- [Claves administradas por el cliente para el cifrado de Azure Storage](customer-managed-keys-overview.md)
