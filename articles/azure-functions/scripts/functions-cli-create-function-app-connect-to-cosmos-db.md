---
title: 'Creación de una aplicación de funciones con Azure Cosmos DB: CLI de Azure'
description: 'Ejemplo de script de la CLI de Azure: creación de una función de Azure que se conecta a una instancia de Azure Cosmos DB'
ms.topic: sample
ms.date: 07/03/2018
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 261ede1493b93ccd3cfed19125bbabc3be871698
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107786304"
---
# <a name="create-an-azure-function-that-connects-to-an-azure-cosmos-db"></a>Creación de una función de Azure que se conecta a una instancia de Cosmos DB

Este script de ejemplo de Azure Functions crea una aplicación de función y conecta la función a una base de datos de Azure Cosmos DB. La configuración de la aplicación creada que contiene la conexión se puede utilizar con un [desencadenador o enlace de Azure Cosmos DB](../functions-bindings-cosmosdb.md).

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Este tutorial requiere la versión 2.0 de la CLI de Azure o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente. 

## <a name="sample-script"></a>Script de ejemplo

Este ejemplo crea una aplicación de Azure Function App y agrega una clave de acceso y un punto de conexión de Cosmos DB a configuración de la aplicación.

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/create-function-app-connect-to-cosmos-db/create-function-app-connect-to-cosmos-db.sh "Create an Azure Function that connects to an Azure Cosmos DB")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos: Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Crea un grupo de recursos con una ubicación. |
| [az storage accounts create](/cli/azure/storage/account#az_storage_account_create) | Crear una cuenta de almacenamiento |
| [az functionapp create](/cli/azure/functionapp#az_functionapp_create) | Crea una aplicación de función en el [Plan de consumo](../consumption-plan.md) sin servidor. |
| [az cosmosdb create](/cli/azure/cosmosdb#az_cosmosdb_create) | Crea una base de datos de Azure Cosmos DB. |
| [az cosmosdb show](/cli/azure/cosmosdb#az_cosmosdb_show)| Obtiene la conexión de la cuenta de base de datos. |
| [az cosmosdb list-keys](/cli/azure/cosmosdb#az_cosmosdb_list_keys)| Obtiene las claves para la base de datos. |
| [az functionapp config appsettings set](/cli/azure/functionapp/config/appsettings#az_functionapp_config_appsettings_set) | Establece la cadena de conexión como una configuración de aplicación en la aplicación de funciones. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de CLI para Azure Functions en la [documentación de Azure Functions](../functions-cli-samples.md).
