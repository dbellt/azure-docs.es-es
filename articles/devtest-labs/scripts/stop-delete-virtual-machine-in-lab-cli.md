---
title: 'CLI de Azure: detención y eliminación una máquina virtual en un laboratorio'
description: En este artículo se proporciona un script de la CLI de Azure que detiene y elimina una máquina virtual de un laboratorio en Azure DevTest Labs.
ms.devlang: azurecli
ms.topic: sample
ms.date: 08/11/2020
ms.openlocfilehash: e30ee47c4f34fe6a71e8c934a4f36cb7edbbe20e
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107777361"
---
# <a name="use-azure-cli-to-stop-and-delete-a-virtual-machine-in-a-lab-in-azure-devtest-labs"></a>Uso de la CLI de Azure para detener y eliminar una máquina virtual en un laboratorio de Azure DevTest Labs

Este script de la CLI de Azure detiene y elimina una máquina virtual en un laboratorio. 

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/devtest-lab/stop-delete-virtual-machine-in-lab/stop-delete-virtual-machine-in-lab.sh "Stop and delete a VM in a lab")]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos:

| Get-Help | Notas |
|---|---|
| [az lab vm stop](/cli/azure/lab/vm#az_lab_vm_stop) | Detiene una máquina virtual en un laboratorio. Esta operación puede tardar varios minutos en completarse. |
| [az lab vm delete](/cli/azure/lab/vm#az_lab_vm_delete) | Elimina una máquina virtual en un laboratorio. Esta operación puede tardar varios minutos en completarse. |


## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de la CLI de Azure Lab Services en los [ejemplos de la CLI de Azure Lab Services](../samples-cli.md).
