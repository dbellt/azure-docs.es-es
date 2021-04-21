---
title: 'Tutorial: Uso de identidades administradas para acceder a Azure Resource Manager (Windows): Azure AD'
description: En este tutorial, encontrará directrices para usar una identidad administrada asignada por el usuario de una máquina virtual Windows para acceder a Azure Resource Manager.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/02/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 37b7b6332bdd000968dc136b946d61ebe82d87ad
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107776382"
---
# <a name="tutorial-use-a-user-assigned-managed-identity-on-a-windows-vm-to-access-azure-resource-manager"></a>Tutorial: Uso de identidades administradas asignadas por el usuario en máquinas virtuales Windows para acceder a Azure Resource Manager

En este tutorial, se explican los pasos para crear una identidad asignada por el usuario, asignar esa identidad a una máquina virtual (VM) Windows y usarla después para acceder a la API de Azure Resource Manager. Azure administra automáticamente las identidades de servicio administradas. Estas identidades permiten autenticarse en servicios que admiten la autenticación de Azure AD, sin necesidad de incluir credenciales en el código. 

Aprenderá a:

> [!div class="checklist"]
> * Crear una identidad administrada asignada por el usuario
> * Asignar la identidad asignada por el usuario a la máquina virtual Windows.
> * Conceder a la identidad asignada por el usuario acceso a un grupo de recursos de Azure Resource Manager. 
> * Obtener un token de acceso utilizando la identidad asignada por el usuario y usarlo para llamar a Azure Resource Manager. 
> * Leer las propiedades de un grupo de recursos

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

- [Iniciar sesión en Azure Portal](https://portal.azure.com)

- [Crear una máquina virtual Windows](../../virtual-machines/windows/quick-create-portal.md)

- Para cumplir con los pasos necesarios de creación de recursos y administración de roles de este tutorial, la cuenta debe tener permisos de "Propietario" en el ámbito adecuado (en su suscripción o grupo de recursos). Si necesita ayuda con la asignación de roles, consulte [Asignación de roles de Azure para administrar el acceso a los recursos de una suscripción de Azure](../../role-based-access-control/role-assignments-portal.md).

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

## <a name="enable"></a>Habilitar

Para un escenario basado en una identidad asignada por el usuario, debe realizar los siguientes pasos:

- Creación de una identidad
- Asignación de la identidad recién creada

### <a name="create-identity"></a>Creación de la identidad

En esta sección se muestra cómo crear una identidad asignada por el usuario. Las identidades asignadas por el usuario se crean como recursos de Azure independientes. Mediante [New-AzUserAssignedIdentity](/powershell/module/az.managedserviceidentity/get-azuserassignedidentity), Azure crea una identidad en el inquilino de Azure AD que puede asignarse a una o varias instancias de servicio de Azure.

[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

```azurepowershell-interactive
New-AzUserAssignedIdentity -ResourceGroupName myResourceGroupVM -Name ID1
```

La respuesta contiene detalles sobre la identidad asignada por el usuario que se ha creado, tal y como se muestra en el ejemplo siguiente. Anote los valores `Id` y `ClientId` de la identidad asignada por el usuario, ya que los utilizará en los siguientes pasos:

```azurepowershell
{
Id: /subscriptions/<SUBSCRIPTIONID>/resourcegroups/myResourceGroupVM/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1
ResourceGroupName : myResourceGroupVM
Name: ID1
Location: westus
TenantId: 733a8f0e-ec41-4e69-8ad8-971fc4b533f8
PrincipalId: e591178e-b785-43c8-95d2-1397559b2fb9
ClientId: af825a31-b0e0-471f-baea-96de555632f9
ClientSecretUrl: https://control-westus.identity.azure.net/subscriptions/<SUBSCRIPTIONID>/resourcegroups/myResourceGroupVM/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1/credentials?tid=733a8f0e-ec41-4e69-8ad8-971fc4b533f8&oid=e591178e-b785-43c8-95d2-1397559b2fb9&aid=af825a31-b0e0-471f-baea-96de555632f9
Type: Microsoft.ManagedIdentity/userAssignedIdentities
}
```

### <a name="assign-identity"></a>Asignación de la identidad

En esta sección se muestra cómo asignar una identidad asignada por el usuario a una máquina virtual Windows. Los clientes pueden usar las identidades asignadas por el usuario en diversos recursos de Azure. Use los comandos siguientes para asociar la identidad asignada por el usuario con una única máquina virtual. Use la propiedad `Id` devuelta en el paso anterior para el parámetro `-IdentityID`.

```azurepowershell-interactive
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM
Update-AzVM -ResourceGroupName TestRG -VM $vm -IdentityType "UserAssigned" -IdentityID "/subscriptions/<SUBSCRIPTIONID>/resourcegroups/myResourceGroupVM/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"
```

## <a name="grant-access"></a>Conceder acceso 

En esta sección se muestra cómo conceder a la identidad asignada por el usuario el acceso a un grupo de recursos de Azure Resource Manager. Managed Identities for Azure Resources dispone de identidades que el código puede utilizar para solicitar tokens de acceso que le permitan autenticarse en las API de recursos compatibles con la autenticación de Azure AD. En este tutorial, el código accederá a la API de Azure Resource Manager. 

Antes de que el código pueda acceder a la API, debe conceder a la identidad acceso a un recurso en Azure Resource Manager. En este caso, se trata del grupo de recursos que contiene la máquina virtual. Actualice el valor de `<SUBSCRIPTION ID>` según corresponda en su entorno.

```azurepowershell-interactive
$spID = (Get-AzUserAssignedIdentity -ResourceGroupName myResourceGroupVM -Name ID1).principalid
New-AzRoleAssignment -ObjectId $spID -RoleDefinitionName "Reader" -Scope "/subscriptions/<SUBSCRIPTIONID>/resourcegroups/myResourceGroupVM/"
```

La respuesta contiene detalles de la asignación de roles que se ha creado, de forma similar al ejemplo siguiente:

```azurepowershell
RoleAssignmentId: /subscriptions/80c696ff-5efa-4909-a64d-f1b616f423ca/resourcegroups/myResourceGroupVM/providers/Microsoft.Authorization/roleAssignments/f9cc753d-265e-4434-ae19-0c3e2ead62ac
Scope: /subscriptions/80c696ff-5efa-4909-a64d-f1b616f423ca/resourcegroups/myResourceGroupVM
DisplayName: ID1
SignInName:
RoleDefinitionName: Reader
RoleDefinitionId: acdd72a7-3385-48ef-bd42-f606fba81ae7
ObjectId: e591178e-b785-43c8-95d2-1397559b2fb9
ObjectType: ServicePrincipal
CanDelegate: False
```

## <a name="access-data"></a>Acceso a los datos

### <a name="get-an-access-token"></a>Obtención de un token de acceso 

En el resto del tutorial, trabajará desde la máquina virtual que ha creado anteriormente.

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

2. En el portal, vaya a **Virtual Machines** y diríjase a la máquina virtual Windows. A continuación, en **Introducción**, haga clic en **Conectar**.

3. Escriba el **nombre de usuario** y la **contraseña** que usó al crear la máquina virtual Windows.

4. Ahora que ha creado una **conexión a Escritorio remoto** con la máquina virtual, abra **PowerShell** en la sesión remota.

5. Utilizando `Invoke-WebRequest` de PowerShell, realice una solicitud al punto de conexión local de Managed Identities for Azure Resources para obtener un token de acceso de Azure Resource Manager.  `client_id` es el valor que se devuelve al crear la identidad administrada asignada por el usuario.

    ```azurepowershell
    $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&client_id=af825a31-b0e0-471f-baea-96de555632f9&resource=https://management.azure.com/' -Method GET -Headers @{Metadata="true"}
    $content = $response.Content | ConvertFrom-Json
    $ArmToken = $content.access_token
    ```

### <a name="read-properties"></a>Lectura de las propiedades

Use el token de acceso que ha recuperado en el paso anterior para acceder a Azure Resource Manager y lea las propiedades del grupo de recursos para el que ha concedido acceso a la identidad asignada por el usuario. Reemplace `<SUBSCRIPTION ID>` con el identificador de la suscripción de su entorno.

```azurepowershell
(Invoke-WebRequest -Uri https://management.azure.com/subscriptions/80c696ff-5efa-4909-a64d-f1b616f423ca/resourceGroups/myResourceGroupVM?api-version=2016-06-01 -Method GET -ContentType "application/json" -Headers @{Authorization ="Bearer $ArmToken"}).content
```
La respuesta contiene la información específica del grupo de recursos, de forma similar al ejemplo siguiente:

```json
{"id":"/subscriptions/<SUBSCRIPTIONID>/resourceGroups/myResourceGroupVM","name":"myResourceGroupVM","location":"eastus","properties":{"provisioningState":"Succeeded"}}
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a crear una identidad asignada por el usuario y a asociarla a una instancia de Azure Virtual Machines para acceder a la API de Azure Resource Manager.  Para obtener más información sobre Azure Resource Manager, vea:

> [!div class="nextstepaction"]
>[Azure Resource Manager](../../azure-resource-manager/management/overview.md)
