---
title: 'Asignación de un acceso de identidad administrada a un recurso mediante PowerShell: Azure AD'
description: Instrucciones paso a paso para asignar una identidad administrada en un recurso y el acceso a otro recurso usando PowerShell.
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
ms.date: 12/15/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: fc5df641f27f8d604f7b5647736128856a3ecd51
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107764376"
---
# <a name="assign-a-managed-identity-access-to-a-resource-using-powershell"></a>Asignación de un acceso de identidad administrada a un recurso mediante PowerShell

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Una vez que haya configurado un recurso de Azure con una identidad administrada, puede dar acceso de identidad administrada a otro recurso, al igual que cualquier entidad de seguridad. En este ejemplo se muestra cómo otorgar acceso de identidad administrada a una máquina virtual de Azure para una cuenta de almacenamiento de Azure, mediante PowerShell.

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario](overview.md#managed-identity-types)** .
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.
- Para ejecutar los scripts de ejemplo, tiene dos opciones:
    - Use [Azure Cloud Shell](../../cloud-shell/overview.md), que puede abrir mediante el botón **Probar** en la esquina superior derecha de los bloques de código.
    - Ejecute scripts localmente instalando la versión más reciente de [Azure PowerShell](/powershell/azure/install-az-ps) y, a continuación, inicie sesión en Azure con `Connect-AzAccount`. 

## <a name="use-azure-rbac-to-assign-a-managed-identity-access-to-another-resource"></a>Uso de RBAC de Azure para asignar el acceso de una identidad administrada a otro recurso

1. Habilite la identidad administrada en un recurso de Azure, [como una máquina virtual de Azure](qs-configure-powershell-windows-vm.md).

1. En este ejemplo, vamos a dar a una máquina virtual de Azure acceso a una cuenta de almacenamiento. En primer lugar, se usa [Get-AzVM](/powershell/module/az.compute/get-azvm) para obtener la entidad de servicio para la máquina virtual denominada `myVM`, que se creó cuando se habilitó la identidad administrada. A continuación, usamos [New-AzRoleAssignment](/powershell/module/Az.Resources/New-AzRoleAssignment) para conceder a la máquina virtual acceso de **lectura** a una cuenta de almacenamiento denominada `myStorageAcct`:

    ```azurepowershell-interactive
    $spID = (Get-AzVM -ResourceGroupName myRG -Name myVM).identity.principalid
    New-AzRoleAssignment -ObjectId $spID -RoleDefinitionName "Reader" -Scope "/subscriptions/<mySubscriptionID>/resourceGroups/<myResourceGroup>/providers/Microsoft.Storage/storageAccounts/<myStorageAcct>"
    ```

## <a name="next-steps"></a>Pasos siguientes

- [Identidades administradas de recursos de Azure](overview.md)
- Para habilitar la identidad administrada de una máquina virtual de Azure, consulte [Configuración de identidades administradas de recursos de Azure en una VM de Azure mediante PowerShell](qs-configure-powershell-windows-vm.md).
