---
title: Prevención de que se eliminen o cambien recursos de Azure Cosmos DB
description: Use los Bloqueos de recursos de Azure para impedir que se eliminen o cambien recursos de Azure Cosmos DB.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 05/13/2021
ms.author: mjbrown
ms.openlocfilehash: c6cf6d0f3bc7982f8ac41c12cfd6bf394f763348
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110094805"
---
# <a name="prevent-azure-cosmos-db-resources-from-being-deleted-or-changed"></a>Prevención de que se eliminen o cambien recursos de Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Como administrador, puede que tenga que bloquear una cuenta, base de datos o contenedor de Azure Cosmos para impedir que otros usuarios de su organización eliminen o modifiquen accidentalmente recursos críticos. Puede establecer el bloqueo de nivel en CanNotDelete o ReadOnly.

- **CanNotDelete** significa que los usuarios autorizados pueden leer y modificar recursos, pero no eliminarlos.
- **ReadOnly** significa que los usuarios autorizados solo pueden leer recursos, pero no actualizarlos ni eliminarlos. Aplicar este bloqueo es similar a restringir todos los usuarios autorizados a los permisos concedidos por el rol Lector.

## <a name="how-locks-are-applied"></a>Cómo se aplican los bloqueos

Cuando se aplica un bloqueo en un ámbito primario, todos los recursos heredan el mismo bloqueo. Incluso los recursos que agregue posteriormente heredan el bloqueo del elemento primario. El bloqueo más restrictivo de toda la herencia tiene prioridad.

A diferencia del control de acceso basado en rol de Azure, los bloqueos de administración se usan para aplicar una restricción a todos los usuarios y roles. Para obtener información acerca de Azure RBAC para Azure Cosmos DB, vea [Control de acceso basado en rol en Azure Cosmos DB](role-based-access-control.md).

Los bloqueos de Resource Manager solo se aplican a las operaciones que se producen en el plano de administración, que se compone de las operaciones enviadas a https://management.azure.com. Los bloqueos no restringen cómo los recursos realizan sus propias funciones. Los cambios de recursos están restringidos, pero las operaciones de recursos no lo están. Por ejemplo, un bloqueo ReadOnly en un contenedor de Azure Cosmos impide que lo elimine o modifique. Pero no le impide crear, actualizar o eliminar datos de este. Se permiten las transacciones de datos porque esas operaciones no se envían a https://management.azure.com.

## <a name="manage-locks"></a>Administración de bloqueos

> [!WARNING]
> Los bloqueos de recursos no funcionan para los cambios que realizan los usuarios que acceden a Azure Cosmos DB mediante claves de cuenta, a menos que la cuenta de Azure Cosmos se bloquee por primera vez habilitando la propiedad disableKeyBasedMetadataWriteAccess. Se debe tener cuidado antes de habilitar esta propiedad para asegurarse de que no interrumpe las aplicaciones existentes que realizan cambios en los recursos con cualquier SDK, Azure Portal o herramientas de terceros que se conectan a través de claves de cuenta y modifican recursos como, por ejemplo, el cambio de rendimiento, la actualización de directivas de índice, etc. Para obtener más información y consultar una lista de comprobación para asegurarse de que las aplicaciones continúan funcionando, vea [Bloqueo en los SDK de Azure Cosmos DB para evitar cambios](role-based-access-control.md#prevent-sdk-changes).

### <a name="powershell"></a>PowerShell

```powershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "my-cosmos-account"
$lockName = "$accountName-Lock"

# First, update the account to prevent changes by anything that connects via account keys
Update-AzCosmosDBAccount -ResourceGroupName $resourceGroupName -Name $accountName -DisableKeyBasedMetadataWriteAccess true

# Create a Delete Lock on an Azure Cosmos account resource and all child resources
New-AzResourceLock `
    -ApiVersion "2020-04-01" `
    -ResourceType "Microsoft.DocumentDB/databaseAccounts" `
    -ResourceGroupName $resourceGroupName `
    -ResourceName $accountName `
    -LockName $lockName `
    -LockLevel "CanNotDelete" # CanNotDelete or ReadOnly
```

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
resourceGroupName='myResourceGroup'
accountName='my-cosmos-account'
$lockName="$accountName-Lock"

# First, update the account to prevent changes by anything that connects via account keys
az cosmosdb update  --name $accountName --resource-group $resourceGroupName  --disable-key-based-metadata-write-access true

# Create a Delete Lock on an Azure Cosmos account resource
az lock create --name $lockName \
    --resource-group $resourceGroupName \
    --resource-type Microsoft.DocumentDB/databaseAccount \
    --lock-type 'CanNotDelete' # CanNotDelete or ReadOnly \
    --resource $accountName
```

### <a name="template"></a>Plantilla

Al aplicar un bloqueo a un recurso de Azure Cosmos DB, use los formatos siguientes:

- nombre: `{resourceName}/Microsoft.Authorization/{lockName}`
- tipo: `{resourceProviderNamespace}/{resourceType}/providers/locks`

> [!IMPORTANT]
> Al modificar una cuenta existente de Azure Cosmos, asegúrese de incluir las demás propiedades de la cuenta y los recursos secundarios al volver a implementar con esta propiedad. No implemente esta plantilla tal cual o se restablecerán todas las propiedades de la cuenta.

```json
"resources": [
    {
        "type": "Microsoft.DocumentDB/databaseAccounts",
        "name": "[variables('accountName')]",
        "apiVersion": "2020-04-01",
        "kind": "GlobalDocumentDB",
        "location": "[parameters('location')]",
        "properties": {
            "consistencyPolicy": "[variables('consistencyPolicy')[parameters('defaultConsistencyLevel')]]",
            "locations": "[variables('locations')]",
            "databaseAccountOfferType": "Standard",
            "enableAutomaticFailover": "[parameters('automaticFailover')]",
            "disableKeyBasedMetadataWriteAccess": true
        }
    },
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/providers/locks",
        "apiVersion": "2020-04-01",
        "name": "[concat(variables('accountName'), '/Microsoft.Authorization/siteLock')]",
        "dependsOn": [
        "[resourceId('Microsoft.DocumentDB/databaseAccounts', variables('accountName'))]"
        ],
        "properties": {
        "level": "CanNotDelete",
        "notes": "Cosmos account should not be deleted."
        }
    }
]
```

## <a name="next-steps"></a>Pasos siguientes

- [Información general de los bloqueos de Azure Resource Manager](../azure-resource-manager/management/lock-resources.md)
