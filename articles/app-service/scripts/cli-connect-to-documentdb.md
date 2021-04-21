---
title: 'CLI: Conexión de una aplicación a Cosmos DB'
description: Aprenda a usar la CLI de Azure para automatizar la implementación y administración de la aplicación App Service. En este ejemplo se indica cómo conectar una aplicación a MongoDB (Cosmos DB).
author: msangapu-msft
tags: azure-service-management
ms.assetid: bbbdbc42-efb5-4b4f-8ba6-c03c9d16a7ea
ms.devlang: azurecli
ms.topic: sample
ms.date: 12/11/2017
ms.author: msangapu
ms.custom: mvc, seodec18, devx-track-azurecli
ms.openlocfilehash: f76f036e2101fb1dbb4483ba417f5cf10f3e37f4
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107782422"
---
# <a name="connect-an-app-service-app-to-cosmos-db-using-cli"></a>Conexión de una aplicación de App Service a Cosmos DB mediante la CLI

Este script de ejemplo crea una cuenta de Azure Cosmos DB con la API de Azure Cosmos DB para MongoDB y una aplicación de App Service. A continuación, vincula una cadena de conexión de MongoDB a la aplicación web mediante la configuración de la aplicación.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, necesitará la CLI de Azure versión 2.0 o posterior. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/app-service/connect-to-documentdb/connect-to-documentdb.sh "Azure Cosmos DB")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos, una aplicación de App Service, Cosmos DB y todos los recursos relacionados. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [`az group create`](/cli/azure/group#az_group_create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [`az appservice plan create`](/cli/azure/appservice/plan#az_appservice_plan_create) | Crea un plan de App Service, |
| [`az webapp create`](/cli/azure/webapp#az_webapp_create) | Crea una aplicación de App Service. |
| [`az cosmosdb create`](/cli/azure/cosmosdb#az_cosmosdb_create) | Crea una cuenta de Cosmos DB. |
| [`az cosmosdb list-connection-strings`](/cli/azure/cosmosdb#az_cosmosdb_list_connection_strings) | Devuelve las cadenas de conexión de la cuenta de Cosmos DB especificada. |
| [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set) | Crea o actualiza una configuración de aplicación para una aplicación de App Service. La configuración de la aplicación se expone como variables de entorno para la aplicación. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Puede encontrar ejemplos de script adicionales de la CLI de App Service en la [documentación de Azure App Service](../samples-cli.md).
