---
title: Creación de un bloqueo de recurso para una base de datos y una colección de MongoDB API para Azure Cosmos DB
description: Creación de un bloqueo de recurso para una base de datos y una colección de MongoDB API para Azure Cosmos DB
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: sample
ms.date: 07/29/2020
ms.openlocfilehash: ee298b33736ae25b365cc54ee1bb5ec9f38bfb62
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107763620"
---
# <a name="create-a-resource-lock-for-azure-cosmos-dbs-api-for-mongodb-using-azure-cli"></a>Creación de un bloqueo de recurso para la API de Azure Cosmos DB para MongoDB mediante la CLI de Azure
[!INCLUDE[appliesto-mongodb-api](../../../includes/appliesto-mongodb-api.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../../../includes/azure-cli-prepare-your-environment.md)]

- En este artículo se necesita la versión 2.9.1 de la CLI de Azure o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

> [!IMPORTANT]
> Los bloqueos de recursos no funcionarán para los cambios que realicen los usuarios que se conectan mediante cualquier SDK de MongoDB, Mongoshell, cualquier otra herramienta o Azure Portal, a menos que se bloquee en primer lugar la cuenta de Cosmos DB con la propiedad `disableKeyBasedMetadataWriteAccess` habilitada. Para más información sobre cómo habilitar esta propiedad, consulte [Evitar cambios de SDK](../../../role-based-access-control.md#prevent-sdk-changes).

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../../../cli_scripts/cosmosdb/mongodb/lock.sh "Create a resource lock for an Azure Cosmos DB MongoDB API database and collection.")]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az lock create](/cli/azure/lock#az_lock_create) | Crea un bloqueo. |
| [az lock list](/cli/azure/lock#az_lock_list) | Muestra información del bloqueo. |
| [az lock show](/cli/azure/lock#az_lock_show) | Muestra las propiedades de un bloqueo. |
| [az lock delete](/cli/azure/lock#az_lock_delete) | Elimina un bloqueo. |

## <a name="next-steps"></a>Pasos siguientes

- [Bloqueo de recursos para impedir cambios inesperados](../../../../azure-resource-manager/management/lock-resources.md)

- [Documentación sobre la CLI de Azure Cosmos DB](/cli/azure/cosmosdb).

- [Repositorio de GitHub para la CLI de Azure Cosmos DB](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb).
