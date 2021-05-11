---
title: 'Montaje de un recurso compartido de archivos en una aplicación de funciones de Python: CLI de Azure'
description: Cree una aplicación de funciones de Python sin servidor y montaría un recurso compartido de archivos existente mediante la CLI de Azure.
ms.topic: sample
ms.date: 03/01/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: dd40b2c723e7aa6780db1c8ac96b2fc2bea20da4
ms.sourcegitcommit: 43be2ce9bf6d1186795609c99b6b8f6bb4676f47
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108277955"
---
# <a name="mount-a-file-share-to-a-python-function-app-using-azure-cli"></a>Montaje de un recurso compartido de archivos en una aplicación de funciones de Python mediante la CLI de Azure

Este script de ejemplo de Azure Functions crea una aplicación de funciones y crea un recurso compartido en Azure Files. Luego monta el recurso compartido, con el fin de que sus funciones puedan acceder a los datos.  

>[!NOTE]
>La aplicación de funciones creada se ejecuta en la versión 3.7 de Python. Azure Functions también [admite las versiones 3.6 y 3.8 de Python](../functions-reference-python.md#python-version).

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Este tutorial requiere la versión 2.0 de la CLI de Azure o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente. 

## <a name="sample-script"></a>Script de ejemplo

Este script crea una aplicación de funciones de Azure Functions con el [Plan de consumo](../consumption-plan.md).

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/functions-cli-mount-files-storage-linux/functions-cli-mount-files-storage-linux.sh "Create a function app on a Consumption plan")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Cada comando de la tabla crea un vínculo a documentación específica del comando. Este script usa los siguientes comandos:

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az storage account create](/cli/azure/storage/account#az_storage_account_create) | Crea una cuenta de Azure Storage. |
| [az functionapp create](/cli/azure/functionapp#az_functionapp_create) | Crea una aplicación de función. |
| [az storage share create](/cli/azure/storage/share#az_storage_share_create) | Crea un recurso compartido de archivos de Azure Files en una cuenta de almacenamiento. | 
| [az storage directory create](/cli/azure/storage/directory#az_storage_directory_create) | Crea un directorio en el recurso compartido de archivos. |
| [az webapp config storage-account add](/cli/azure/webapp/config/storage-account#az_webapp_config_storage_account_add) | Monta el recurso compartido en la aplicación de funciones. |
| [az webapp config storage-account list](/cli/azure/webapp/config/storage-account#az_webapp_config_storage_account_list) | Muestra los recursos compartidos de archivos montados en la aplicación de funciones. | 

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de CLI para Azure Functions en la [documentación de Azure Functions](../functions-cli-samples.md).
