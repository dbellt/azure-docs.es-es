---
title: Asignación de una identidad administrada a un rol de aplicación mediante PowerShell en Azure AD
description: Instrucciones paso a paso para asignar acceso de identidad administrada al rol de otra aplicación mediante PowerShell.
services: active-directory
documentationcenter: ''
author: johndowns
manager: ''
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/10/2020
ms.author: jodowns
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 5150c26d67b77008c11764cc6e51ca7085ffcb31
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107784918"
---
# <a name="assign-a-managed-identity-access-to-an-application-role-using-powershell"></a>Asignación de acceso de identidad administrada a un rol de aplicación mediante PowerShell

Las identidades administradas para los recursos de Azure proporcionan a los servicios de Azure una identidad en Azure Active Directory. Funcionan sin necesidad de usar credenciales en el código. Los servicios de Azure usan esta identidad para autenticarse en servicios compatibles con la autenticación de Azure AD. Los roles de aplicación ofrecen una forma de control de acceso basado en roles y permiten que un servicio implemente reglas de autorización.

En este artículo, aprenderá a asignar una identidad administrada a un rol de aplicación mostrado por otra aplicación mediante Azure AD PowerShell.

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerrequisitos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario](overview.md#managed-identity-types)** .
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.
- Para ejecutar los scripts de ejemplo, tiene dos opciones:
    - Use el servicio [Azure Cloud Shell](../../cloud-shell/overview.md), que puede abrir mediante el botón **Probar**, ubicado en la esquina superior derecha de los bloques de código.
    - Ejecute scripts localmente mediante la instalación de la versión más reciente de [Azure AD PowerShell](/powershell/azure/active-directory/install-adv2).

## <a name="assign-a-managed-identity-access-to-another-applications-app-role"></a>Asignación de acceso de identidad administrada al rol de aplicación de otra aplicación

1. Habilite la identidad administrada en un recurso de Azure, [como una máquina virtual de Azure](qs-configure-powershell-windows-vm.md).

1. Busque el identificador de objeto de la entidad de servicio de la identidad administrada.

   **En el caso de una identidad administrada asignada por el sistema**, encontrará el identificador de objeto en la página de **identidad** del recurso en Azure Portal. También puede usar el siguiente script de PowerShell para buscar el identificador de objeto. Necesitará el identificador del recurso que creó en el paso 1, que está disponible en la página **Propiedades** del recurso en Azure Portal.

    ```powershell
    $resourceIdWithManagedIdentity = '/subscriptions/{my subscription ID}/resourceGroups/{my resource group name}/providers/Microsoft.Compute/virtualMachines/{my virtual machine name}'
    (Get-AzResource -ResourceId $resourceIdWithManagedIdentity).Identity.PrincipalId
    ```

    **En el caso de una identidad administrada asignada por el usuario**, encontrará el identificador de objeto de la identidad administrada en la página de **información general** del recurso en Azure Portal. También puede usar el siguiente script de PowerShell para buscar el identificador de objeto. Necesitará el identificador del recurso de la identidad administrada asignada por el usuario.

    ```powershell
    $userManagedIdentityResourceId = '/subscriptions/{my subscription ID}/resourceGroups/{my resource group name}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{my managed identity name}'
    (Get-AzResource -ResourceId $userManagedIdentityResourceId).Properties.PrincipalId
    ```

1. Cree un nuevo registro de aplicación que represente el servicio al que la identidad administrada enviará una solicitud. Omita este paso si la API o el servicio que muestran la concesión de rol de aplicación a la identidad administrada ya tiene una entidad de servicio en el inquilino de Azure AD. Por ejemplo, si desea conceder el acceso de identidad administrada a Microsoft Graph API, puede omitir el paso.

1. Busque el identificador de objeto de la entidad de servicio de la aplicación de servicio. Puede encontrarlo en Azure Portal. Vaya a Azure Active Directory y abra la página **Aplicaciones empresariales**. A continuación, busque la aplicación y el **identificador de objeto**. También puede buscar el identificador de objeto de la entidad de servicio por su nombre para mostrar. Para ello, use el siguiente script de PowerShell:

    ```powershell
    $serverServicePrincipalObjectId = (Get-AzureADServicePrincipal -Filter "DisplayName eq '$applicationName'").ObjectId
    ```

    > [!NOTE]
    > Los nombres para mostrar de las aplicaciones no son únicos, por lo que debe comprobar que obtiene la entidad de servicio de la aplicación correcta.

1. Agregue un [rol de aplicación](../develop/howto-add-app-roles-in-azure-ad-apps.md) a la aplicación que creó en el paso 3. Puede crear el rol mediante Azure Portal o Microsoft Graph. Por ejemplo, podría agregar un rol de aplicación como el siguiente:

    ```json
    {
        "allowedMemberTypes": [
            "Application"
        ],
        "displayName": "Read data from MyApi",
        "id": "0566419e-bb95-4d9d-a4f8-ed9a0f147fa6",
        "isEnabled": true,
        "description": "Allow the application to read data as itself.",
        "value": "MyApi.Read.All"
    }
    ```

1. Asigne el rol de aplicación a la identidad administrada. Para asignar el rol de aplicación, necesitará la siguiente información:
    * `managedIdentityObjectId`: el identificador de objeto de la entidad de servicio de la identidad administrada, que encontró en el paso 2.
    * `serverServicePrincipalObjectId`: el identificador de objeto de la entidad de servicio de la aplicación de servidor, que encontró en el paso 4.
    * `appRoleId`: el identificador del rol de aplicación mostrado por la aplicación de servidor, que se generó en el paso 5; en el ejemplo, el identificador del rol de aplicación es `0566419e-bb95-4d9d-a4f8-ed9a0f147fa6`.
   
   Ejecute el siguiente script de PowerShell para agregar la asignación de roles:

    ```powershell
    New-AzureADServiceAppRoleAssignment -ObjectId $managedIdentityObjectId -Id $appRoleId -PrincipalId $managedIdentityObjectId -ResourceId $serverServicePrincipalObjectId
    ```

## <a name="complete-script"></a>Completar script

Este script de ejemplo muestra cómo asignar una identidad administrada de una aplicación web de Azure a un rol de aplicación.

```powershell
# Install the module. (You need admin on the machine.)
# Install-Module AzureAD

# Your tenant ID (in the Azure portal, under Azure Active Directory > Overview).
$tenantID = '<tenant-id>'

# The name of your web app, which has a managed identity that should be assigned to the server app's app role.
$webAppName = '<web-app-name>'
$resourceGroupName = '<resource-group-name-containing-web-app>'

# The name of the server app that exposes the app role.
$serverApplicationName = '<server-application-name>' # For example, MyApi

# The name of the app role that the managed identity should be assigned to.
$appRoleName = '<app-role-name>' # For example, MyApi.Read.All

# Look up the web app's managed identity's object ID.
$managedIdentityObjectId = (Get-AzWebApp -ResourceGroupName $resourceGroupName -Name $webAppName).identity.principalid

Connect-AzureAD -TenantId $tenantID

# Look up the details about the server app's service principal and app role.
$serverServicePrincipal = (Get-AzureADServicePrincipal -Filter "DisplayName eq '$serverApplicationName'")
$serverServicePrincipalObjectId = $serverServicePrincipal.ObjectId
$appRoleId = ($serverServicePrincipal.AppRoles | Where-Object {$_.Value -eq $appRoleName }).Id

# Assign the managed identity access to the app role.
New-AzureADServiceAppRoleAssignment `
    -ObjectId $managedIdentityObjectId `
    -Id $appRoleId `
    -PrincipalId $managedIdentityObjectId `
    -ResourceId $serverServicePrincipalObjectId
```

## <a name="next-steps"></a>Pasos siguientes

- [Identidades administradas de recursos de Azure](overview.md)
- Para habilitar la identidad administrada de una máquina virtual de Azure, consulte [Configuración de identidades administradas de recursos de Azure en una VM de Azure mediante PowerShell](qs-configure-powershell-windows-vm.md).
