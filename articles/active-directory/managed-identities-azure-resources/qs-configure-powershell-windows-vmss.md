---
title: 'Configuración de identidades administradas en conjuntos de escalado de máquinas virtuales mediante PowerShell: Azure AD'
description: Instrucciones paso a paso para configurar identidades administradas asignadas por el sistema y por el usuario en un conjunto de escalado de máquinas virtuales de Azure mediante PowerShell.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/15/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: eb651df2fe3b9771154f9e188d41c660f7ffd028
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107774153"
---
# <a name="configure-managed-identities-for-azure-resources-on-virtual-machine-scale-sets-using-powershell"></a>Configuración de identidades administradas de recursos de Azure en conjuntos de escalado de máquinas virtuales mediante PowerShell

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Las identidades administradas para los recursos de Azure proporcionan a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory. Puede usar esta identidad para autenticar a cualquier servicio que admita la autenticación de Azure AD, sin necesidad de tener credenciales en el código. 

En este artículo, mediante PowerShell, aprenderá a realizar las operaciones de identidades administradas de recursos de Azure en un conjunto de escalado de máquinas virtuales:

- Habilitación y deshabilitación de la identidad administrada asignada por el sistema en un conjunto de escalado de máquinas virtuales
- Adición y eliminación de una identidad administrada asignada por el usuario en un conjunto de escalado de máquinas virtuales

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una asignada por el usuario](overview.md#managed-identity-types)** .

- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.

- Para realizar las operaciones de administración de este artículo, su cuenta debe tener las siguientes asignaciones de control de acceso basado en rol:

    > [!NOTE]
    > No se requieren asignaciones de roles del directorio de Azure AD.

    - [Colaborador de máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) para crear un conjunto de escalado de máquinas virtuales y habilitar y quitar la identidad administrada asignada por el usuario o por el sistema desde un conjunto de escalado de máquinas virtuales.
    - Rol [Colaborador de identidad administrada](../../role-based-access-control/built-in-roles.md#managed-identity-contributor) para crear una identidad administrada asignada por el usuario.
    - Rol [Operador de identidad administrada](../../role-based-access-control/built-in-roles.md#managed-identity-operator) para asignar y quitar una identidad administrada asignada por el usuario en un conjunto de escalado de máquinas virtuales.

- Para ejecutar los scripts de ejemplo, tiene dos opciones:
    - Use [Azure Cloud Shell](../../cloud-shell/overview.md), que puede abrir mediante el botón **Probar** en la esquina superior derecha de los bloques de código.
    - Ejecute scripts localmente instalando la versión más reciente de [Azure PowerShell](/powershell/azure/install-az-ps) y, a continuación, inicie sesión en Azure con `Connect-AzAccount`. 

## <a name="system-assigned-managed-identity"></a>Identidad administrada asignada por el sistema

En esta sección, aprenderá a habilitar y quitar una identidad administrada asignada por el sistema con Azure PowerShell.

### <a name="enable-system-assigned-managed-identity-during-the-creation-of-an-azure-virtual-machine-scale-set"></a>Habilitación de la identidad administrada asignada por el sistema durante la creación de un conjunto de escalado de máquinas virtuales de Azure

Para crear un conjunto de escalado de máquinas virtuales con la identidad administrada asignada por el sistema habilitada:

1. Consulte el *ejemplo 1* en el artículo de referencia del cmdlet [New-AzVmssConfig](/powershell/module/az.compute/new-azvmssconfig) para crear un conjunto de escalado de máquinas virtuales con una identidad administrada asignada por el sistema.  Agregue el parámetro `-IdentityType SystemAssigned` al cmdlet `New-AzVmssConfig`:

    ```azurepowershell-interactive
    $VMSS = New-AzVmssConfig -Location $Loc -SkuCapacity 2 -SkuName "Standard_A0" -UpgradePolicyMode "Automatic" -NetworkInterfaceConfiguration $NetCfg -IdentityType SystemAssigned`
    ```

### <a name="enable-system-assigned-managed-identity-on-an-existing-azure-virtual-machine-scale-set"></a>Habilitación de la identidad administrada asignada por el sistema en un conjunto de escalado de máquinas virtuales de Azure existente

Si necesita habilitar una identidad administrada asignada por el sistema en un conjunto de escalado de máquinas virtuales de Azure existente:

1. Asegúrese de que la cuenta que está usando pertenece a un rol que le conceda permisos de escritura en el conjunto de escalado de máquinas virtuales, como "Colaborador de la máquina virtual":
   
1. Recupere las propiedades del conjunto de escalado de máquinas virtuales mediante el cmdlet [`Get-AzVmss`](/powershell/module/az.compute/get-azvmss). Luego, para habilitar una identidad administrada asignada por el sistema, use el modificador `-IdentityType` en el cmdlet [Update-AzVmss](/powershell/module/az.compute/update-azvmss):

   ```azurepowershell-interactive
   Update-AzVmss -ResourceGroupName myResourceGroup -Name -myVmss -IdentityType "SystemAssigned"
   ```

### <a name="disable-the-system-assigned-managed-identity-from-an-azure-virtual-machine-scale-set"></a>Deshabilitación de la identidad administrada asignada por el sistema de un conjunto de escalado de máquinas virtuales de Azure

Si tiene un conjunto de escalado de máquinas virtuales que ya no necesita la identidad administrada asignada por el sistema, pero aún necesita identidades administradas asignadas por el usuario, use el siguiente cmdlet:

1. Asegúrese de que la cuenta pertenece a un rol que le conceda permisos de escritura en el conjunto de escalado de máquinas virtuales, como "Colaborador de la máquina virtual":

1. Ejecute el siguiente cmdlet:

   ```azurepowershell-interactive
   Update-AzVmss -ResourceGroupName myResourceGroup -Name myVmss -IdentityType "UserAssigned"
   ```

1. Si tiene un conjunto de escalado de máquinas virtuales que ya no necesita la identidad administrada asignada por el sistema y no tiene identidades administradas asignadas por el usuario, use el siguiente comando:

    ```azurepowershell-interactive
    Update-AzVmss -ResourceGroupName myResourceGroup -Name myVmss -IdentityType None
    ```
    
## <a name="user-assigned-managed-identity"></a>Identidad administrada asignada por el usuario

En esta sección, aprenderá a agregar y quitar una identidad administrada asignada por el usuario de un conjunto de escalado de máquinas virtuales mediante Azure PowerShell.

### <a name="assign-a-user-assigned-managed-identity-during-creation-of-an-azure-virtual-machine-scale-set"></a>Asignación de una identidad administrada asignada por el usuario durante la creación de un conjunto de escalado de máquinas virtuales de Azure

Actualmente, no se puede crear un conjunto de escalado de máquinas virtuales con una identidad administrada asignada por el usuario mediante PowerShell. Consulte la sección siguiente sobre cómo agregar una identidad administrada asignada por el usuario a un conjunto de escalado de máquinas virtuales existente. Compruebe si hay actualizaciones.

### <a name="assign-a-user-assigned-managed-identity-to-an-existing-azure-virtual-machine-scale-set"></a>Asignación de una identidad administrada asignada por el usuario a un conjunto de escalado de máquinas virtuales de Azure existente

Para asignar una identidad administrada asignada por el usuario a un conjunto de escalado de máquinas virtuales de Azure existente:

1. Asegúrese de que la cuenta pertenece a un rol que le conceda permisos de escritura en el conjunto de escalado de máquinas virtuales, como "Colaborador de la máquina virtual":

1. Recupere las propiedades del conjunto de escalado de máquinas virtuales mediante el cmdlet `Get-AzVM`. A continuación, para asignar una identidad administrada asignada por el usuario al conjunto de escalado de máquinas virtuales, use los modificadores `-IdentityType` y `-IdentityID` en el cmdlet [Update-AzVmss](/powershell/module/az.compute/update-azvmss). Reemplace `<VM NAME>`, `<SUBSCRIPTION ID>`, `<RESROURCE GROUP>`, `<USER ASSIGNED ID1>`, `USER ASSIGNED ID2` con sus propios valores.

   [!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

   ```azurepowershell-interactive
   Update-AzVmss -ResourceGroupName <RESOURCE GROUP> -Name <VMSS NAME> -IdentityType UserAssigned -IdentityID "<USER ASSIGNED ID1>","<USER ASSIGNED ID2>"
   ```

### <a name="remove-a-user-assigned-managed-identity-from-an-azure-virtual-machine-scale-set"></a>Eliminación de una identidad administrada asignada por el usuario de un conjunto de escalado de máquinas virtuales de Azure

Si el conjunto de escalado de máquinas virtuales tiene varias identidades administradas asignadas por el usuario, puede quitarlas todas, menos la última, utilizando los comandos siguientes. Asegúrese de reemplazar los valores de los parámetros `<RESOURCE GROUP>` y `<VIRTUAL MACHINE SCALE SET NAME>` con sus propios valores. `<USER ASSIGNED IDENTITY NAME>` es la propiedad de nombre de la identidad administrada asignada por el usuario, que debe permanecer en el conjunto de escalado de máquinas virtuales. Esta información se puede encontrar en la sección de identidad del conjunto de escalado de máquinas virtuales mediante `az vmss show`:

```azurepowershell-interactive
Update-AzVmss -ResourceGroupName myResourceGroup -Name myVmss -IdentityType UserAssigned -IdentityID "<USER ASSIGNED IDENTITY NAME>"
```
Si el conjunto de escalado de máquinas virtuales no tiene una identidad administrada asignada por el sistema y desea quitar de ella todas las identidades administradas asignadas por el usuario, utilice el siguiente comando:

```azurepowershell-interactive
Update-AzVmss -ResourceGroupName myResourceGroup -Name myVmss -IdentityType None
```
Si el conjunto de escalado de máquinas virtuales tiene identidades administradas asignadas tanto por el sistema como por el usuario, puede quitar todas las identidades administradas asignadas por el usuario si cambia para usar solo las identidades administradas asignadas por el sistema.

```azurepowershell-interactive
Update-AzVmss -ResourceGroupName myResourceGroup -Name myVmss -IdentityType "SystemAssigned"
```

## <a name="next-steps"></a>Pasos siguientes

- [Información general sobre las identidades administradas de recursos de Azure](overview.md)
- Para ver guías de inicio rápido completas acerca de la creación de máquinas virtuales de Azure, consulte:
  
  - [Creación de una máquina virtual Windows con PowerShell](../../virtual-machines/windows/quick-create-powershell.md) 
  - [Creación de una máquina virtual Linux con PowerShell](../../virtual-machines/linux/quick-create-powershell.md)
