---
title: 'Creación de una aplicación de funciones en un plan de App Service: CLI de Azure'
description: 'Ejemplo de script de la CLI de Azure: creación de una instancia de Function App en un plan de App Service'
ms.assetid: 0e221db6-ee2d-4e16-9bf6-a456cd05b6e7
ms.topic: sample
ms.date: 07/03/2018
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: ea368714ded56d77a10e24b853144bdf1dd42dfe
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107786340"
---
# <a name="create-a-function-app-in-an-app-service-plan"></a>Creación de una instancia de Function App en un plan de App Service

Este script de ejemplo de Azure Functions crea una aplicación de función, que es un contenedor para las funciones. La aplicación de función que se crea utiliza un plan de App Service dedicado, lo que significa que los recursos del servidor están siempre disponibles.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Este tutorial requiere la versión 2.0 de la CLI de Azure o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente. 

## <a name="sample-script"></a>Script de ejemplo

Este script crea una aplicación de Azure Function App mediante un [plan de App Service](../dedicated-plan.md) dedicado.

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/create-function-app-app-service-plan/create-function-app-app-service-plan.sh "Create an Azure Function on an App Service plan")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Cada comando de la tabla crea un vínculo a documentación específica del comando. Este script usa los siguientes comandos:

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az storage account create](/cli/azure/storage/account#az_storage_account_create) | Crea una cuenta de Azure Storage. |
| [az functionapp plan create](/cli/azure/functionapp/plan#az_functionapp_plan_create) | Crea un plan Premium. |
| [az functionapp create](/cli/azure/functionapp#az_functionapp_create) | Crea una aplicación de función en el plan de App Service. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de CLI para Azure Functions en la [documentación de Azure Functions](../functions-cli-samples.md).
