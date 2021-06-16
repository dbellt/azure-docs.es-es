---
title: 'Creación o actualización de roles personalizados de Azure mediante plantillas de Azure Resource Manager: Azure RBAC'
description: Obtenga información sobre cómo crear o actualizar roles personalizados de Azure mediante una plantilla de Azure Resource Manager (ARM) y el control de acceso basado en rol de Azure (Azure RBAC).
services: role-based-access-control,azure-resource-manager
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 12/16/2020
ms.author: rolyon
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 230a772ba0edb95c2616ede1c4ea9deb4a1b0943
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110696634"
---
# <a name="create-or-update-azure-custom-roles-using-an-arm-template"></a>Creación o actualización de roles personalizados de Azure mediante una plantilla de ARM

Si los [roles integrados de Azure](built-in-roles.md) no cumplen las necesidades específicas de su organización, puede crear los [suyos propios](custom-roles.md). En este artículo se describe cómo crear o actualizar un rol personalizado mediante una plantilla de Azure Resource Manager (ARM).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Para crear un rol personalizado, especifique un nombre de rol, los permisos y dónde se puede usar el rol. En este artículo, se creará un rol denominado _Rol personalizado: lector RG_ con permisos de recursos que se pueden asignar en un ámbito de suscripción o inferior.

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsubscription-deployments%2Fcreate-role-def%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Para crear un rol personalizado, debe tener:

- Permisos para crear roles personalizados, como [Propietario](built-in-roles.md#owner) o [Administrador de acceso de usuario](built-in-roles.md#user-access-administrator).

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este artículo forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/create-role-def). La plantilla tiene cuatro parámetros y una sección de recursos. Los cuatro parámetros son:

- Matriz de acciones con un valor predeterminado de `["Microsoft.Resources/subscriptions/resourceGroups/read"]`.
- Matriz de `notActions` con un valor predeterminado vacío.
- Nombre del rol con un valor predeterminado de `Custom Role - RG Reader`.
- Descripción del rol con un valor predeterminado de `Subscription Level Deployment of a Role Definition`.

El ámbito en el que se puede asignar este rol personalizado se establece en la suscripción actual.

:::code language="json" source="~/quickstart-templates/subscription-deployments/create-role-def/azuredeploy.json":::

El recurso definido en la plantilla es el siguiente:

- [Microsoft.Authorization/roleDefinitions](/azure/templates/Microsoft.Authorization/roleDefinitions)

## <a name="deploy-the-template"></a>Implementación de la plantilla

Siga estos pasos para implementar la plantilla anterior.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Abra Azure Cloud Shell para PowerShell.

1. Copie y pegue el siguiente script en Cloud Shell.

    ```azurepowershell-interactive
    $location = Read-Host -Prompt "Enter a location (i.e. centralus)"
    [string[]]$actions = Read-Host -Prompt "Enter actions as a comma-separated list (i.e. action1,action2)"
    $actions = $actions.Split(',')
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/subscription-deployments/create-role-def/azuredeploy.json"
    New-AzDeployment -Location $location -TemplateUri $templateUri -actions $actions
    ```

1. Escriba una ubicación para la implementación, como `centralus`.

1. Escriba una lista de acciones para el rol personalizado como una lista separada por comas, como `Microsoft.Resources/resources/read,Microsoft.Resources/subscriptions/resourceGroups/read`.

1. Si es necesario, presione Entrar para ejecutar el comando `New-AzDeployment`.

    El comando [New-AzDeployment](/powershell/module/az.resources/new-azdeployment) implementa la plantilla para crear el rol personalizado.

    Debería ver un resultado similar al siguiente:

    ```azurepowershell-interactive
    PS> New-AzDeployment -Location $location -TemplateUri $templateUri -actions $actions

    Id                      : /subscriptions/{subscriptionId}/providers/Microsoft.Resources/deployments/azuredeploy
    DeploymentName          : azuredeploy
    Location                : centralus
    ProvisioningState       : Succeeded
    Timestamp               : 6/25/2020 8:08:32 PM
    Mode                    : Incremental
    TemplateLink            :
                              Uri            : https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/subscription-deployments/create-role-def/azuredeploy.json
                              ContentVersion : 1.0.0.0

    Parameters              :
                              Name               Type                       Value
                              =================  =========================  ==========
                              actions            Array                      [
                                "Microsoft.Resources/resources/read",
                                "Microsoft.Resources/subscriptions/resourceGroups/read"
                              ]
                              notActions         Array                      []
                              roleName           String                     Custom Role - RG Reader
                              roleDescription    String                     Subscription Level Deployment of a Role Definition

    Outputs                 :
    DeploymentDebugLogLevel :
    ```

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Siga estos pasos para comprobar que se ha creado el rol personalizado.

1. Ejecute el comando [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition) para mostrar el rol personalizado.

    ```azurepowershell-interactive
    Get-AzRoleDefinition "Custom Role - RG Reader" | ConvertTo-Json
    ```

    Debería ver un resultado similar al siguiente:

    ```azurepowershell-interactive
    {
      "Name": "Custom Role - RG Reader",
      "Id": "11111111-1111-1111-1111-111111111111",
      "IsCustom": true,
      "Description": "Subscription Level Deployment of a Role Definition",
      "Actions": [
        "Microsoft.Resources/resources/read",
        "Microsoft.Resources/subscriptions/resourceGroups/read"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/{subscriptionId}"
      ]
    }
    ```

1. Abra la suscripción en Azure Portal.

1. En el menú izquierdo, seleccione **Control de acceso (IAM)** .

1. Seleccione la pestaña **Roles**.

1. Establezca la lista **Tipo** en **CustomRole**.

1. Compruebe que se muestre el rol **Rol personalizado: lector RG**.

   ![Nuevo rol personalizado en Azure Portal](./media/custom-roles-template/custom-role-template-portal.png)

## <a name="update-a-custom-role"></a>Actualización de un rol personalizado

De forma similar a la creación de un rol personalizado, puede actualizar un rol personalizado existente mediante una plantilla. Para actualizar un rol personalizado, debe especificar el rol que quiere actualizar.

Estos son los cambios que debe realizar en la plantilla de inicio rápido anterior para actualizar el rol personalizado.

- Incluya el id. de rol como parámetro.
    ```json
        ...
        "roleDefName": {
          "type": "string",
          "metadata": {
            "description": "ID of the role definition"
          }
        ...
    ```

- Incluya el parámetro de id. de rol en la definición de roles.

    ```json
      ...
      "resources": [
        {
          "type": "Microsoft.Authorization/roleDefinitions",
          "apiVersion": "2018-07-01",
          "name": "[parameters('roleDefName')]",
          "properties": {
            ...
    ```

A continuación proporcionamos un ejemplo de cómo implementar la plantilla.

```azurepowershell
$location = Read-Host -Prompt "Enter a location (i.e. centralus)"
[string[]]$actions = Read-Host -Prompt "Enter actions as a comma-separated list (i.e. action1,action2)"
$actions = $actions.Split(',')
$roleDefName = Read-Host -Prompt "Enter the role ID to update"
$templateFile = "rg-reader-update.json"
New-AzDeployment -Location $location -TemplateFile $templateFile -actions $actions -roleDefName $roleDefName
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Para quitar el rol personalizado, siga estos pasos.

1. Ejecute el siguiente comando para quitar el rol personalizado.

    ```azurepowershell-interactive
    Get-AzRoleDefinition -Name "Custom Role - RG Reader" | Remove-AzRoleDefinition
    ```

1. Escriba **Y** para confirmar que desea quitar el rol personalizado.

## <a name="next-steps"></a>Pasos siguientes

- [Descripción de las definiciones de roles de Azure](role-definitions.md)
- [Inicio rápido: Asignación de un role de Azure mediante una plantilla de Azure Resource Manager](quickstart-role-assignments-template.md)
- [Documentación de las plantillas de Resource Manager](../azure-resource-manager/templates/index.yml)
