---
title: 'Creación de una aplicación de funciones con almacenamiento conectado: CLI de Azure'
description: 'Ejemplo de script de la CLI de Azure: creación de una función de Azure que se conecta a una instancia de Azure Storage'
ms.topic: sample
ms.date: 04/20/2017
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 539c3a7dd95045b2e569dbb339be0e5a0c845902
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107786297"
---
# <a name="create-a-function-app-with-a-named-storage-account-connection"></a>Creación de una aplicación de funciones con una conexión de cuenta de almacenamiento con nombre 

Este script de ejemplo de Azure Functions crea una aplicación de función y conecta la función a una cuenta de Azure Storage. La configuración de la aplicación creada que contiene la conexión se puede utilizar con un [desencadenador o enlace de almacenamiento](../functions-bindings-storage-blob.md). 

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Este tutorial requiere la versión 2.0 de la CLI de Azure o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="sample-script"></a>Script de ejemplo

Este ejemplo crea una aplicación de Azure Function App y agrega la cadena de conexión de almacenamiento a una configuración de aplicación.

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/create-function-app-connect-to-storage/create-function-app-connect-to-storage-account.sh "Integrate Function App into Azure Storage Account")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Crea un grupo de recursos con una ubicación. |
| [az storage account create](/cli/azure/storage/account#az_storage_account_create) | Cree una cuenta de almacenamiento. |
| [az functionapp create](/cli/azure/functionapp#az_functionapp_create) | Crea una aplicación de función en el [Plan de consumo](../consumption-plan.md) sin servidor. |
| [az storage account show-connection-string](/cli/azure/storage/account#az_storage_account_show_connection_string) | Obtiene la cadena de conexión para la cuenta. |
| [az functionapp config appsettings set](/cli/azure/functionapp/config/appsettings#az_functionapp_config_appsettings_set) | Establece la cadena de conexión como una configuración de aplicación en la aplicación de funciones. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de CLI para Azure Functions en la [documentación de Azure Functions](../functions-cli-samples.md).
