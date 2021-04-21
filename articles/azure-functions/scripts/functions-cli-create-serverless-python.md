---
title: 'Creación de una aplicación de funciones de Python sin servidor: CLI de Azure'
description: Creación de una aplicación de funciones de Python sin servidor con la CLI de Azure
ms.assetid: 0e221db6-ee2d-4e16-9bf6-a456cd05b6e7
ms.topic: sample
ms.date: 11/23/2019
ms.custom: devx-track-python, devx-track-azurecli
ms.openlocfilehash: 9bd80d5644587c6dcc074a91bf91382d0692bc18
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107786204"
---
# <a name="create-a-serverless-python-function-app-using-azure-cli"></a>Creación de una aplicación de funciones de Python sin servidor con la CLI de Azure

Este script de ejemplo de Azure Functions crea una aplicación de función, que es un contenedor para las funciones. 

>[!NOTE]
>La aplicación de funciones creada se ejecuta en la versión 3.6 de Python. La versión 3.7 de Python también es compatible con Azure Functions.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Este tutorial requiere la versión 2.0 de la CLI de Azure o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente. 

## <a name="sample-script"></a>Script de ejemplo

Este script crea una aplicación de funciones de Azure con el [Plan de consumo](../consumption-plan.md).

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/create-function-app-consumption-python/create-function-app-consumption-python.sh "Create an Azure Function on a Consumption plan")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Cada comando de la tabla crea un vínculo a documentación específica del comando. Este script usa los siguientes comandos:

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az storage account create](/cli/azure/storage/account#az_storage_account_create) | Crea una cuenta de Azure Storage. |
| [az functionapp create](/cli/azure/functionapp#az_functionapp_create) | Crea una aplicación de función. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de CLI para Azure Functions en la [documentación de Azure Functions](../functions-cli-samples.md).
