---
title: Scripts de la CLI de Azure para operaciones de rendimiento (RU/s) para recursos de Core (SQL) API de Azure Cosmos DB
description: Scripts de la CLI de Azure para operaciones de rendimiento (RU/s) para recursos de Core (SQL) API de Azure Cosmos DB
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
ms.date: 10/07/2020
ms.openlocfilehash: 2df31e5903785b6e25ea79a107a53084849c66fe
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107789094"
---
# <a name="throughput-rus-operations-with-azure-cli-for-a-database-or-container-for-azure-cosmos-db-core-sql-api"></a>Operaciones de rendimiento (RU/s) con la CLI de Azure para una base de datos o un contenedor para Core (SQL) API de Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](../../../includes/appliesto-sql-api.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../../../includes/azure-cli-prepare-your-environment.md)]

- En este artículo se necesita la versión 2.12.1 de la CLI de Azure o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="sample-script"></a>Script de ejemplo

Este script crea una base de datos de Core (SQL) API con un rendimiento compartido y un contenedor de Core (SQL) API con un rendimiento dedicado y, a continuación, actualiza el rendimiento de la base de datos y el contenedor. A continuación, el script migra del rendimiento estándar a escalado automático y, a continuación, lee el valor del rendimiento de escalado automático una vez que se ha migrado.

[!code-azurecli-interactive[main](../../../../../cli_scripts/cosmosdb/sql/throughput.sh "Throughput operations for a SQL database and container.")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Después de ejecutar el script de ejemplo, se puede usar el comando siguiente para quitar el grupo de recursos y todos los recursos asociados.

```azurecli-interactive
az group delete --name $resourceGroupName
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az cosmosdb create](/cli/azure/cosmosdb#az_cosmosdb_create) | Crea una cuenta de Azure Cosmos DB. |
| [az cosmosdb sql database create](/cli/azure/cosmosdb/sql/database#az_cosmosdb_sql_database_create) | Crea una base de datos de Core (SQL) para Azure Cosmos. |
| [az cosmosdb sql container create](/cli/azure/cosmosdb/sql/container#az_cosmosdb_sql_container_create) | Crea un contenedor de Core (SQL) para Azure Cosmos. |
| [az cosmosdb sql database throughput update](/cli/azure/cosmosdb/sql/database/throughput#az_cosmosdb_sql_database_throughput_update) | Actualiza el rendimiento de una base de datos de Core (SQL) de Azure Cosmos. |
| [az cosmosdb sql container throughput update](/cli/azure/cosmosdb/sql/container/throughput#az_cosmosdb_sql_container_throughput_update) | Actualiza el rendimiento de un contenedor de Core (SQL) de Azure Cosmos. |
| [az cosmosdb sql database throughput migrate](/cli/azure/cosmosdb/sql/database/throughput#az_cosmosdb_sql_database_throughput_migrate) | Migra el rendimiento de una base de datos de Core (SQL) de Azure Cosmos. |
| [az cosmosdb sql container throughput migrate](/cli/azure/cosmosdb/sql/container/throughput#az_cosmosdb_sql_container_throughput_migrate) | Migra el rendimiento de un contenedor de Core (SQL) de Azure Cosmos. |
| [az group delete](/cli/azure/resource#az_resource_delete) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte la [documentación de la CLI de Azure Cosmos DB](/cli/azure/cosmosdb).

Encontrará más ejemplos de scripts de la CLI de Azure Cosmos DB en el [repositorio de la CLI de Azure Cosmos DB en GitHub](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb).
