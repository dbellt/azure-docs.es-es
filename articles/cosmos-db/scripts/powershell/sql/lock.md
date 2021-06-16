---
title: Script de PowerShell para crear un bloqueo de recursos para una base de datos y un contenedor de SQL API en Azure Cosmos
description: Creación de un bloqueo de recurso para una base de datos y un contenedor de SQL API en Azure Cosmos
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
ms.date: 06/12/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 1e89104e51d234a2a2a6420da44094de28688f66
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110679756"
---
# <a name="create-a-resource-lock-for-azure-cosmos-sql-api-database-and-container-using-azure-powershell"></a>Creación de un bloqueo de recursos para una base de datos y un contenedor de SQL API en Azure Cosmos con Azure PowerShell
[!INCLUDE[appliesto-sql-api](../../../includes/appliesto-sql-api.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

Este ejemplo requiere Azure PowerShell Az 5.4.0 o una versión posterior. Ejecute `Get-Module -ListAvailable Az` para ver qué versiones están instaladas.
Si necesita instalarlo, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps).

Ejecute [Connect AzAccount](/powershell/module/az.accounts/connect-azaccount) para iniciar sesión en Azure.

> [!IMPORTANT]
> Los bloqueos de recursos no funcionarán para los cambios que realicen los usuarios que se conecten mediante cualquier SDK de Cosmos DB, todas las herramientas que se conecten mediante claves de cuenta o Azure Portal, a menos que se bloquee en primer lugar la cuenta de Cosmos DB con la propiedad `disableKeyBasedMetadataWriteAccess` habilitada. Para más información sobre cómo habilitar esta propiedad, consulte [Evitar cambios de SDK](../../../role-based-access-control.md#prevent-sdk-changes).

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../../../powershell_scripts/cosmosdb/sql/ps-sql-lock.ps1 "Create, list, and remove resource locks")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Después de ejecutar el script de ejemplo, se puede usar el comando siguiente para quitar el grupo de recursos y todos los recursos asociados.

```powershell
Remove-AzResourceGroup -ResourceGroupName "myResourceGroup"
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
|**Recurso de Azure**| |
| [New-AzResourceLock](/powershell/module/az.resources/new-azresourcelock) | Crea un bloqueo de recursos. |
| [Get-AzResourceLock](/powershell/module/az.resources/get-azresourcelock) | Obtiene un bloqueo de recursos o enumera los bloqueos de recursos. |
| [Remove-AzResourceLock](/powershell/module/az.resources/remove-azresourcelock) | Elimina un bloqueo de recursos. |
|||

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/).
