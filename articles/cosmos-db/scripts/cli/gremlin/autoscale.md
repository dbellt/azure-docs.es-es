---
title: Creación de una base de datos y un grafo con escalabilidad automática de Gremlin para Azure Cosmos DB
description: Creación de una base de datos y un grafo con escalabilidad automática de Gremlin para Azure Cosmos DB
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: sample
ms.date: 7/29/2020
ms.openlocfilehash: c1f7893cc77ee49f16565bf36da247903c3654b1
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107770856"
---
# <a name="create-an-azure-cosmos-gremlin-api-account-database-and-graph-with-autoscale-using-azure-cli"></a>Creación de una cuenta de Azure Cosmos y una base de datos y un grafo con escalabilidad automática de Gremlin API con la CLI de Azure
[!INCLUDE[appliesto-gremlin-api](../../../includes/appliesto-gremlin-api.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../../../includes/azure-cli-prepare-your-environment.md)]

- En este artículo se necesita la versión 2.9.1 de la CLI de Azure o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../../../cli_scripts/cosmosdb/gremlin/autoscale.sh "Create an Azure Cosmos DB Gremlin API account, database, and graph with autoscale.")]

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
| [az cosmosdb gremlin database create](/cli/azure/cosmosdb/gremlin/database#az_cosmosdb_gremlin_database_create) | Crea una base de datos de Gremlin para Azure Cosmos. |
| [az cosmosdb gremlin graph create](/cli/azure/cosmosdb/gremlin/graph#az_cosmosdb_gremlin_graph_create) | Crea un grafo de Gremlin para Azure Cosmos. |
| [az group delete](/cli/azure/resource#az_resource_delete) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte la [documentación de la CLI de Azure Cosmos DB](/cli/azure/cosmosdb).

Encontrará más ejemplos de scripts de la CLI de Azure Cosmos DB en el [repositorio de la CLI de Azure Cosmos DB en GitHub](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb).
