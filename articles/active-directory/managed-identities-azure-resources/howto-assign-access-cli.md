---
title: 'Asignación de un acceso de identidad administrada a un recurso mediante la CLI de Azure: Azure AD'
description: Instrucciones paso a paso para asignar una identidad administrada en un recurso y acceso a otro recurso usando la CLI de Azure.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 01/29/2021
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurecli
ms.openlocfilehash: 1e1fa22cc36df00b098274002b6bd444be4140ff
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107783294"
---
# <a name="assign-a-managed-identity-access-to-a-resource-using-azure-cli"></a>Asignación de un acceso de identidad administrada a un recurso mediante la CLI de Azure

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Una vez que haya configurado un recurso de Azure con una identidad administrada, puede dar acceso de identidad administrada a otro recurso, al igual que cualquier entidad de seguridad. En este ejemplo se muestra cómo otorgar acceso de identidad administrada a una máquina virtual o un conjunto de escalado de máquinas virtuales de Azure a una cuenta de almacenamiento de Azure mediante la CLI de Azure.

Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.

## <a name="prerequisites"></a>Prerrequisitos

- Si no está familiarizado con las identidades administradas para los recursos de Azure, vea [¿Qué son las identidades administradas para recursos de Azure?](overview.md). Para obtener información sobre los tipos de identidad administrada asignados por el sistema y asignados por el usuario, va [Tipos de identidad administrada](overview.md#managed-identity-types).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="use-azure-rbac-to-assign-a-managed-identity-access-to-another-resource"></a>Uso de RBAC de Azure para asignar el acceso de una identidad administrada a otro recurso

Después de habilitar la identidad administrada en un recurso de Azure, como una [máquina virtual de Azure](qs-configure-cli-windows-vm.md) o un [conjunto de escalado de máquinas virtuales de Azure](qs-configure-cli-windows-vmss.md): 

1. En este ejemplo, vamos a dar a una máquina virtual de Azure acceso a una cuenta de almacenamiento. En primer lugar se utiliza una [az resource list](/cli/azure/resource/#az_resource_list) para obtener la entidad de servicio para la máquina virtual denominada myVM:

   ```azurecli-interactive
   spID=$(az resource list -n myVM --query [*].identity.principalId --out tsv)
   ```
   En el caso de un conjunto de escalado de máquinas virtuales de Azure, el comando es el mismo excepto aquí, donde obtiene la entidad de servicio para el conjunto de escalado de máquinas virtuales denominado "DevTestVMSS":
   
   ```azurecli-interactive
   spID=$(az resource list -n DevTestVMSS --query [*].identity.principalId --out tsv)
   ```

1. Cuando ya tenga el identificador de la entidad de servicio, use [az role assignment create](/cli/azure/role/assignment#az_role_assignment_create) para conceder a la máquina virtual o el conjunto de escalado de máquinas virtuales acceso de "lector" a una cuenta de almacenamiento denominada "myStorageAcct":

   ```azurecli-interactive
   az role assignment create --assignee $spID --role 'Reader' --scope /subscriptions/<mySubscriptionID>/resourceGroups/<myResourceGroup>/providers/Microsoft.Storage/storageAccounts/myStorageAcct
   ```

## <a name="next-steps"></a>Pasos siguientes

- [Información general sobre las identidades administradas de recursos de Azure](overview.md)
- Para habilitar la identidad administrada de una máquina virtual de Azure, consulte [Configuración de identidades administradas de recursos de Azure en una VM de Azure mediante la CLI de Azure](qs-configure-cli-windows-vm.md).
- Para habilitar la identidad administrada en un conjunto de escalado de máquinas virtuales de Azure, consulte [Configuración de identidades administradas de recursos de Azure en un conjunto de escalado de máquinas virtuales mediante la CLI de Azure](qs-configure-cli-windows-vmss.md).
