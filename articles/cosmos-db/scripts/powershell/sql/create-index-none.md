---
title: Script de PowerShell para crear un contenedor con la indexación desactivada en una cuenta de Azure Cosmos DB
description: 'Script de ejemplo de Azure PowerShell: cree un contenedor con la indexación desactivada en una cuenta de Azure Cosmos DB'
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
ms.date: 05/13/2020
ms.author: mjbrown
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 93247ec99315c7026e11a49f7c4f1a0e45365e27
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110679936"
---
# <a name="create-a-container-with-indexing-turned-off-in-an-azure-cosmos-db-account-using-powershell"></a>Creación de un contenedor con la indexación desactivada en una cuenta de Azure Cosmos DB mediante PowerShell
[!INCLUDE[appliesto-sql-api](../../../includes/appliesto-sql-api.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

En este ejemplo, se necesita Azure PowerShell Az 5.4.0 o versiones posteriores. Ejecute `Get-Module -ListAvailable Az` para ver qué versiones están instaladas.
Si necesita instalarlo, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps).

Ejecute [Connect AzAccount](/powershell/module/az.accounts/connect-azaccount) para iniciar sesión en Azure.

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../../../powershell_scripts/cosmosdb/sql/ps-container-create-index-none.ps1 "Create a container with indexing turned off in an Azure Cosmos DB account")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Después de ejecutar el script de ejemplo, se puede usar el comando siguiente para quitar el grupo de recursos y todos los recursos asociados.

```powershell
Remove-AzResourceGroup -ResourceGroupName "myResourceGroup"
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
|**Azure Cosmos DB**| |
| [New-AzCosmosDBAccount](/powershell/module/az.cosmosdb/new-azcosmosdbaccount) | Crea una cuenta de Cosmos DB. |
| [New-AzCosmosDBSqlDatabase](/powershell/module/az.cosmosdb/new-azcosmosdbsqldatabase) | Crea una base de datos SQL de Cosmos DB. |
| [New-AzCosmosDBSqlIndexingPolicy](/powershell/module/az.cosmosdb/new-azcosmosdbsqlindexingpolicy) | Crea un objeto PSSqlIndexingPolicy que se usa como parámetro para New-AzCosmosDBSqlContainer. |
| [New-AzCosmosDBSqlContainer](/powershell/module/az.cosmosdb/new-azcosmosdbsqlcontainer) | Crea un contenedor SQL de Cosmos DB. |
|**Grupos de recursos de Azure**| |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |
|||

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/).
