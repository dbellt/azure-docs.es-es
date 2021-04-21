---
title: 'Obtención de grupos de recursos administrados y cambio de tamaño de las máquinas virtuales: CLI de Azure'
description: Proporciona un script de ejemplo de la CLI de Azure que obtiene un grupo de recursos administrado en una aplicación administrada de Azure. El script cambia el tamaño de las máquinas virtuales.
author: tfitzmac
ms.devlang: azurecli
ms.topic: sample
ms.date: 10/25/2017
ms.author: tomfitz
ms.custom: devx-track-azurecli
ms.openlocfilehash: 179b1b64656d3f97778e183d57797e4b3660fece
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107775446"
---
# <a name="get-resources-in-a-managed-resource-group-and-resize-vms-with-azure-cli"></a>Obtención de recursos de un grupo de recursos administrados y cambio de tamaño de las máquinas virtuales con la CLI de Azure

Este script recupera los recursos de un grupo de recursos administrados y cambia el tamaño de las máquinas virtuales de ese grupo de recursos.


[!INCLUDE [sample-cli-install](../../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli[main](../../../../cli_scripts/managed-applications/get-application/get-application.sh "Get application")]


## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para implementar la aplicación administrada. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az managedapp list](/cli/azure/managedapp#az_managedapp_list) | Enumera las aplicaciones administradas. Proporciona valores de consulta que se centran en los resultados. |
| [az resource list](/cli/azure/resource#az_resource_list) | Enumera los recursos. Proporciona un grupo de recursos y valores de consulta que se centran en el resultado. |
| [az vm resize](/cli/azure/vm#az_vm_resize) | Actualiza el tamaño de una máquina virtual. |


## <a name="next-steps"></a>Pasos siguientes

* Para una introducción a las aplicaciones administradas, consulte la [introducción a las aplicaciones administradas de Azure](../overview.md).
* Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).
