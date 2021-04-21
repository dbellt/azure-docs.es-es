---
title: 'Ejemplo de script de la CLI de Azure: grupo de Linux en Batch'
description: Este script muestra algunos de los comandos disponibles en la CLI de Azure para crear y administrar un grupo de nodos de proceso de Linux en Azure Batch.
ms.topic: sample
ms.date: 01/29/2018
ms.custom: devx-track-azurecli
ms.openlocfilehash: b5e1bdccefffa7803fbe744e27c1b36ca719560d
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107768294"
---
# <a name="cli-example-create-and-manage-a-linux-pool-in-azure-batch"></a>Ejemplo de la CLI: creación y administración de un grupo de Linux en Azure Batch

Este script muestra algunos de los comandos disponibles en la CLI de Azure para crear y administrar un grupo de nodos de proceso de Linux en Azure Batch.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Este tutorial requiere la versión 2.0.20 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente. 

## <a name="example-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/batch/manage-pool/manage-pool-linux.sh "Manage Linux Virtual Machine Pool")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Ejecute el siguiente comando para quitar el grupo de recursos y todos los recursos asociados a él.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az batch account create](/cli/azure/batch/account#az_batch_account_create) | Crea la cuenta de Batch. |
| [az batch account login](/cli/azure/batch/account#az_batch_account_login) | Realiza la autenticación respecto de la cuenta de Batch especificada para una mayor interacción de la CLI.  |
| [az batch pool node-agent-skus list](../batch-linux-nodes.md#list-of-virtual-machine-images) | Enumera las SKU del agente del nodo disponibles y la información de la imagen.  |
| [az batch pool create](/cli/azure/batch/pool#az_batch_pool_create) | Crea un grupo de nodos de proceso.  |
| [az batch pool resize](/cli/azure/batch/pool#az_batch_pool_resize) | Cambia la cantidad de máquinas virtuales en ejecución en el grupo especificado.  |
| [az batch pool show](/cli/azure/batch/pool#az_batch_pool_show) | Muestra las propiedades de un grupo.  |
| [az batch node list](/cli/azure/batch/node#az_batch_node_list) | Enumera todos los nodos de proceso del grupo especificado.  |
| [az batch node reboot](/cli/azure/batch/node#az_batch_node_reboot) | Reinicia el nodo de proceso especificado.  |
| [az batch node delete](/cli/azure/batch/node#az_batch_node_delete) | Elimina los nodos enumerados del grupo especificado.  |
| [az group delete](/cli/azure/group#az_group_delete) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).
