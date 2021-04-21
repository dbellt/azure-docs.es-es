---
title: 'Ejemplo de script de la CLI de Azure: creación de una cuenta en Batch - suscripción de usuario'
description: Este script crea una cuenta en Azure Batch en modo de suscripción de usuario. Esta cuenta asigna nodos de proceso a la suscripción.
ms.topic: sample
ms.date: 01/29/2018
ms.custom: devx-track-azurecli
ms.openlocfilehash: 9bd7b7ac3dbb52ebafa00499e64ec3cff0969a13
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107768366"
---
# <a name="cli-example-create-a-batch-account-in-user-subscription-mode"></a>Ejemplo de la CLI: creación de una cuenta en Batch en modo de suscripción de usuario

Este script crea una cuenta en Azure Batch en modo de suscripción de usuario. Una cuenta que asigna nodos de proceso a su suscripción debe estar autenticada mediante un token de Azure Active Directory. Los nodos de proceso asignaban un recuento con respecto a la cuota de vCPU (núcleos) de su suscripción. 

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Este tutorial requiere la versión 2.0.20 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.  

## <a name="example-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/batch/create-account/create-account-user-subscription.sh "Create Account using user subscription")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Ejecute el siguiente comando para quitar el grupo de recursos y todos los recursos asociados a él.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az role assignment create](/cli/azure/role) | Crea una nueva asignación de roles para un usuario, grupo o entidad de servicio. |
| [az group create](/cli/azure/group#az_group_create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az keyvault create](/cli/azure/keyvault#az_keyvault_create) | Crea un almacén de claves. |
| [az keyvault set-policy](/cli/azure/keyvault#az_keyvault_set_policy) | Actualiza la directiva de seguridad del almacén de Key Vault especificado. |
| [az batch account create](/cli/azure/batch/account#az_batch_account_create) | Crea la cuenta de Batch.  |
| [az batch account login](/cli/azure/batch/account#az_batch_account_login) | Realiza la autenticación respecto de la cuenta de Batch especificada para una mayor interacción de la CLI.  |
| [az group delete](/cli/azure/group#az_group_delete) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).
