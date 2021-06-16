---
title: Rotación del protector de TDE (PowerShell y CLI de Azure)
titleSuffix: Azure SQL Database & Azure Synapse Analytics
description: Obtenga información sobre cómo rotar el protector de Cifrado de datos transparente (TDE) para un servidor de Azure que usan Azure SQL Database y Azure Synapse Analytics mediante PowerShell y la CLI de Azure.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: seo-lt-2019 sqldbrb=1, devx-track-azurecli, devx-track-azurepowershell
ms.devlang: ''
ms.topic: how-to
author: shohamMSFT
ms.author: shohamd
ms.reviewer: vanto
ms.date: 03/12/2019
ms.openlocfilehash: 9c43cb7bb60b758230664571fd7d7e592009fff2
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110667197"
---
# <a name="rotate-the-transparent-data-encryption-tde-protector"></a>Rotación del protector de Cifrado de datos transparente (TDE)
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]


En este artículo se describe la rotación de claves para un [servidor](logical-servers.md) mediante un protector de TDE de Azure Key Vault. La rotación del protector de TDE lógico para un servidor significa cambiar a una nueva clave asimétrica que protege las bases de datos en el servidor. La rotación de claves es una operación en línea y solo debería tardar unos segundos en completarse, ya que únicamente descifra y vuelve a cifrar la clave de cifrado de los datos de la base de datos, no toda la base de datos.

En esta guía se describen dos opciones para rotar el protector de TDE en el servidor.

> [!NOTE]
> Un grupo de SQL dedicado en pausa en Azure Synapse Analytics se debe reanudar antes de la rotación de las claves.

> [!IMPORTANT]
> No elimine las versiones anteriores de la clave después de una sustitución. Cuando las claves se sustituyen, algunos datos siguen cifrados con las claves anteriores como, por ejemplo, las copias de seguridad de base de datos más antiguas.

## <a name="prerequisites"></a>Requisitos previos

- En esta guía de procedimientos se asume que ya utiliza una clave de Azure Key Vault como protector de TDE para Azure SQL Database o Azure Synapse Analytics. Consulte [Cifrado de datos transparente con compatibilidad con BYOK](transparent-data-encryption-byok-overview.md).
- Es preciso tener instalado y en ejecución Azure PowerShell.
- [Recomendado pero opcional] Cree primero el material de clave para el protector de TDE en un módulo de seguridad de hardware (HSM) o en un almacén local de claves e importe el material de clave a Azure Key Vault. Siga las [instrucciones para usar un módulo de seguridad de hardware (HSM) y Key Vault](../../key-vault/general/overview.md) para más información.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para obtener instrucciones sobre la instalación del módulo Az, consulte [Instalación de Azure PowerShell](/powershell/azure/install-az-ps). Para los cmdlets concretos, consulte [AzureRM.Sql](/powershell/module/AzureRM.Sql/).

> [!IMPORTANT]
> El módulo de Azure Resource Manager (RM) para PowerShell todavía es compatible, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. El módulo de AzureRM continuará recibiendo correcciones de errores hasta diciembre de 2020 como mínimo.  Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos. Para obtener más información sobre la compatibilidad, vea [Presentación del nuevo módulo Az de Azure PowerShell](/powershell/azure/new-azureps-module-az).

# <a name="the-azure-cli"></a>[La CLI de Azure](#tab/azure-cli)

Para la instalación, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

* * *

## <a name="manual-key-rotation"></a>Rotación manual de claves

La rotación de claves manual usa los siguientes comandos para agregar una clave completamente nueva, que podría estar en un nuevo nombre de clave o incluso en otro almacén de claves. Usar este enfoque permite agregar la misma clave a distintos almacenes de claves para admitir escenarios de alta disponibilidad y recuperación ante desastres geográfica.

> [!NOTE]
> La longitud combinada para el nombre del almacén de claves y el nombre de la clave no puede superar los 94 caracteres.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Use los cmdlets [Add-AzKeyVaultKey](/powershell/module/az.keyvault/Add-AzKeyVaultKey), [Add-AzSqlServerKeyVaultKey](/powershell/module/az.sql/add-azsqlserverkeyvaultkey) y [Set-AzSqlServerTransparentDataEncryptionProtector](/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector).

```powershell
# add a new key to Key Vault
Add-AzKeyVaultKey -VaultName <keyVaultName> -Name <keyVaultKeyName> -Destination <hardwareOrSoftware>

# add the new key from Key Vault to the server
Add-AzSqlServerKeyVaultKey -KeyId <keyVaultKeyId> -ServerName <logicalServerName> -ResourceGroup <SQLDatabaseResourceGroupName>
  
# set the key as the TDE protector for all resources under the server
Set-AzSqlServerTransparentDataEncryptionProtector -Type AzureKeyVault -KeyId <keyVaultKeyId> `
   -ServerName <logicalServerName> -ResourceGroup <SQLDatabaseResourceGroupName>
```

# <a name="the-azure-cli"></a>[La CLI de Azure](#tab/azure-cli)

Use los comandos [az keyvault key create](/cli/azure/keyvault/key#az_keyvault_key_create), [az sql server key create](/cli/azure/sql/server/key#az_sql_server_key_create) y [az sql server tde-key set](/cli/azure/sql/server/tde-key#az_sql_server_tde_key_set).

```azurecli
# add a new key to Key Vault
az keyvault key create --name <keyVaultKeyName> --vault-name <keyVaultName> --protection <hsmOrSoftware>

# add the new key from Key Vault to the server
az sql server key create --kid <keyVaultKeyId> --resource-group <SQLDatabaseResourceGroupName> --server <logicalServerName>

# set the key as the TDE protector for all resources under the server
az sql server tde-key set --server-key-type AzureKeyVault --kid <keyVaultKeyId> --resource-group <SQLDatabaseResourceGroupName> --server <logicalServerName>
```

* * *

## <a name="switch-tde-protector-mode"></a>Cambio de modo del protector de TDE

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

- Para cambiar el protector de TDE del modo administrado por Microsoft a BYOK, use el cmdlet [Set-AzSqlServerTransparentDataEncryptionProtector](/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector).

   ```powershell
   Set-AzSqlServerTransparentDataEncryptionProtector -Type AzureKeyVault `
       -KeyId <keyVaultKeyId> -ServerName <logicalServerName> -ResourceGroup <SQLDatabaseResourceGroupName>
   ```

- Para cambiar el protector de TDE del modo BYOK al administrado por Microsoft, use el cmdlet [Set-AzSqlServerTransparentDataEncryptionProtector](/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector).

   ```powershell
   Set-AzSqlServerTransparentDataEncryptionProtector -Type ServiceManaged `
       -ServerName <logicalServerName> -ResourceGroup <SQLDatabaseResourceGroupName>
   ```

# <a name="the-azure-cli"></a>[La CLI de Azure](#tab/azure-cli)

En los ejemplos siguientes se usa [az sql server tde-key set](/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector).

- Para cambiar el protector de TDE del modo administrado de Microsoft a BYOK,

   ```azurecli
   az sql server tde-key set --server-key-type AzureKeyVault --kid <keyVaultKeyId> --resource-group <SQLDatabaseResourceGroupName> --server <logicalServerName>
   ```

- Para cambiar el protector de TDE del modo BYOK a administrado por Microsoft,

   ```azurecli
   az sql server tde-key set --server-key-type ServiceManaged --resource-group <SQLDatabaseResourceGroupName> --server <logicalServerName>
   ```

* * *

## <a name="next-steps"></a>Pasos siguientes

- En caso de riesgo de seguridad, aprenda a quitar un protector de TDE potencialmente comprometido: [Eliminación de una clave potencialmente comprometida](transparent-data-encryption-byok-remove-tde-protector.md).

- Introducción a la integración de Azure Key Vault y compatibilidad con Bring Your Own Key para TDE: [Habilitación de TDE con su propia clave de Key Vault mediante PowerShell](transparent-data-encryption-byok-configure.md).
