---
title: 'Creación, enumeración y eliminación de una identidad administrada asignada el usuario mediante Azure PowerShell: Azure AD'
description: Instrucciones paso a paso sobre cómo usar Azure PowerShell para crear, enumerar y eliminar una identidad administrada asignada por el usuario.
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
ms.date: 12/02/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: af76affd9f4034401225e82de4e25e8b0a51125a
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107784846"
---
# <a name="create-list-or-delete-a-user-assigned-managed-identity-using-azure-powershell"></a>Creación, enumeración y eliminación de una identidad administrada asignada por el usuario con Azure PowerShell

Las identidades administradas de los recursos de Azure proporcionan a los servicios de Azure una identidad administrada en Azure Active Directory. Puede usar esta identidad para autenticar a cualquier servicio que admita la autenticación de Azure AD, sin necesidad de tener credenciales en el código. 

En este artículo, aprenderá a crear, enumerar y eliminar una identidad administrada asignada por el usuario con Azure PowerShell.

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario](overview.md#managed-identity-types)** .
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.
- Para ejecutar los scripts de ejemplo, tiene dos opciones:
    - Use [Azure Cloud Shell](../../cloud-shell/overview.md), que puede abrir mediante el botón **Probar** en la esquina superior derecha de los bloques de código.
    - Ejecute los scripts localmente con Azure PowerShell, tal como se describe en la sección siguiente.

### <a name="configure-azure-powershell-locally"></a>Configuración de Azure PowerShell de forma local

Para usar Azure PowerShell de forma local en este artículo (en lugar de usar Cloud Shell), complete los pasos siguientes:

1. Instale [la versión más reciente de Azure PowerShell](/powershell/azure/install-az-ps) si aún no lo ha hecho.

1. Inicie de sesión en Azure:

    ```azurepowershell
    Connect-AzAccount
    ```

1. Instalar la [versión más reciente de PowerShellGet](/powershell/scripting/gallery/installing-psget#for-systems-with-powershell-50-or-newer-you-can-install-the-latest-powershellget).

    ```azurepowershell
    Install-Module -Name PowerShellGet -AllowPrerelease
    ```

    Es posible que necesite `Exit` fuera de la sesión de PowerShell actual después de ejecutar este comando para el paso siguiente.

1. Instale la versión preliminar del módulo `Az.ManagedServiceIdentity` con el fin de realizar las operaciones de identidad administrada que haya asignado el usuario, y que se indican en este artículo.

    ```azurepowershell
    Install-Module -Name Az.ManagedServiceIdentity -AllowPrerelease
    ```

## <a name="create-a-user-assigned-managed-identity"></a>Crear una identidad administrada asignada por el usuario

Para crear una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

Para crear una identidad administrada asignada por el usuario, use el comando `New-AzUserAssignedIdentity`. El parámetro `ResourceGroupName` especifica el grupo de recursos donde se creará la identidad administrada asignada por el usuario, mientras que el parámetro `-Name` especifica su nombre. Reemplace los valores de los parámetros `<RESOURCE GROUP>` y `<USER ASSIGNED IDENTITY NAME>` con sus propios valores:

[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

 ```azurepowershell-interactive
New-AzUserAssignedIdentity -ResourceGroupName <RESOURCEGROUP> -Name <USER ASSIGNED IDENTITY NAME>
```
## <a name="list-user-assigned-managed-identities"></a>Enumerar identidades administradas asignadas por el usuario

Para crear o leer una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Operador de identidades administradas ](../../role-based-access-control/built-in-roles.md#managed-identity-operator) o [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

Para enumerar las identidades administradas asignadas por el usuario, use el comando [Get-AzUserAssigned].  El parámetro `-ResourceGroupName` especifica el grupo de recursos donde se creó la identidad administrada asignada por el usuario. Reemplace `<RESOURCE GROUP>` por su propio valor:

```azurepowershell-interactive
Get-AzUserAssignedIdentity -ResourceGroupName <RESOURCE GROUP>
```
En la respuesta, las identidades administradas asignadas por el usuario reciben el valor `"Microsoft.ManagedIdentity/userAssignedIdentities"` para la clave `Type`.

`Type :Microsoft.ManagedIdentity/userAssignedIdentities`

## <a name="delete-a-user-assigned-managed-identity"></a>Eliminar una identidad administrada asignada por el usuario

Para eliminar una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

Para eliminar una identidad administrada asignada por el usuario, use el comando `Remove-AzUserAssignedIdentity`.  El parámetro `-ResourceGroupName` especifica el grupo de recursos donde se creó la identidad asignada por el usuario, mientras que el parámetro `-Name` especifica su nombre. Reemplace los valores de los parámetros `<RESOURCE GROUP>` y `<USER ASSIGNED IDENTITY NAME>` con sus propios valores:

 ```azurepowershell-interactive
Remove-AzUserAssignedIdentity -ResourceGroupName <RESOURCE GROUP> -Name <USER ASSIGNED IDENTITY NAME>
```
> [!NOTE]
> El proceso de eliminación de una identidad administrada asignada por el usuario no quitará la referencia de ningún recurso al que se haya asignado. Las asignaciones de identidad deben quitarse por separado.

## <a name="next-steps"></a>Pasos siguientes

Para obtener una lista completa y más detalles sobre las identidades administradas de Azure PowerShell para los comandos de los recursos de Azure, consulte [Az.ManagedServiceIdentity](/powershell/module/az.managedserviceidentity#managed_service_identity).
