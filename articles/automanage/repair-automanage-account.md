---
title: Reparación de una cuenta de Azure Automanage estropeada
description: Si ha movido recientemente una suscripción que contiene una cuenta de Automanage a un nuevo inquilino, debe volver a configurarla. En este artículo, aprenderá a hacerlo.
author: asinn826
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: conceptual
ms.date: 11/05/2020
ms.author: alsin
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: fdfeae56ba1ef46b58fdb8f5beb32f85a6e2fcb4
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109752326"
---
# <a name="repair-an-automanage-account"></a>Reparación de una cuenta de Automanage
La [cuenta de Azure Automanage](./automanage-virtual-machines.md#automanage-account) es el contexto de seguridad o la identidad bajo los que tienen lugar las operaciones automatizadas. Si ha movido recientemente una suscripción que contiene una cuenta de Automanage a un nuevo inquilino, debe volver a configurarla. Para volver a configurarla, debe restablecer el tipo de identidad y asignar los roles adecuados para la cuenta.

## <a name="step-1-reset-the-automanage-account-identity-type"></a>Paso 1: Restablecimiento del tipo de identidad de cuenta de Automanage
Restablezca el tipo de identidad de cuenta de Automanage mediante la siguiente plantilla de Azure Resource Manager (ARM). Guarde el archivo localmente como armdeploy.json o un nombre similar. Anote el nombre y la ubicación de la cuenta de Automanage porque son parámetros necesarios en la plantilla de Resource Manager.

1. Cree una implementación de Resource Manager mediante la siguiente plantilla. Use `identityType = None`.
    * Puede crear la implementación en la CLI de Azure mediante `az deployment sub create`. Para más información, consulte [az deployment sub](/cli/azure/deployment/sub).
    * Puede crear la implementación en PowerShell mediante el módulo `New-AzDeployment`. Para más información, consulte [New-AzDeployment](/powershell/module/az.resources/new-azdeployment).

1. Vuelva a ejecutar la misma plantilla de Resource Manager con `identityType = SystemAssigned`.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "accountName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "identityType": {
            "type": "string",
            "allowedValues": [ "None", "SystemAssigned" ]
        }
    },
    "resources": [
        {
            "apiVersion": "2020-06-30-preview",
            "name": "[parameters('accountName')]",
            "location": "[parameters('location')]",
            "type": "Microsoft.Automanage/accounts",
            "identity": {
                "type": "[parameters('identityType')]"
            }
        }
    ]
}

```

## <a name="step-2-assign-appropriate-roles-for-the-automanage-account"></a>Paso 2: Asignación de los roles adecuados para la cuenta de Automanage
La cuenta de Automanage requiere los roles Colaborador y Colaborador de la directiva de recursos en la suscripción que contiene las máquinas virtuales que Automanage administra. Puede asignar los roles mediante Azure Portal, plantillas de Resource Manager o la CLI de Azure.

Si usa una plantilla de Resource Manager o la CLI de Azure, necesitará el id. de entidad de seguridad (también conocido como id. de objeto) de la cuenta de Automanage. (El id. no es necesario si utiliza Azure Portal). Puede encontrar este id. mediante estos métodos:

- [CLI de Azure](/cli/azure/ad/sp): use el comando `az ad sp list --display-name <name of your Automanage Account>`.

- Azure Portal: vaya a **Azure Active Directory** y busque la cuenta de Automanage por nombre. En **Aplicaciones empresariales**, seleccione el nombre de la cuenta de Automanage cuando aparezca.

### <a name="azure-portal"></a>Azure portal
1. En **Suscripciones**, vaya a la suscripción que contiene las máquinas virtuales administradas automáticamente.
1. Vaya a **Control de acceso (IAM)** .
1. Seleccione **Agregar asignación de roles**.
1. Seleccione el rol **Colaborador** y escriba el nombre de la cuenta de Automanage.
1. Seleccione **Guardar**.
1. Repita los pasos del 3 al 5, esta vez con el rol **Colaborador de la directiva de recursos**.

### <a name="arm-template"></a>Plantilla ARM
Ejecute la siguiente plantilla de Resource Manager. Necesitará el id. de entidad de seguridad de la cuenta de Automanage. Los pasos para obtenerlo se encuentran al principio de esta sección. Escriba el id. cuando se le pida.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "principalId": {
            "type": "string",
            "metadata": {
                "description": "The principal to assign the role to"
            }
        }
    },
    "variables": {
        "Contributor": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', 'b24988ac-6180-42a0-ab88-20f7382dd24c')]",
        "Resource Policy Contributor": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', '36243c78-bf99-498c-9df9-86d9f8d28608')]"
    },
    "resources": [
        {
            "type": "Microsoft.Authorization/roleAssignments",
            "apiVersion": "2018-09-01-preview",
            "name": "[guid(uniqueString(variables('Contributor')))]",
            "properties": {
                "roleDefinitionId": "[variables('Contributor')]",
                "principalId": "[parameters('principalId')]"
            }
        },
        {
            "type": "Microsoft.Authorization/roleAssignments",
            "apiVersion": "2018-09-01-preview",
            "name": "[guid(uniqueString(variables('Resource Policy Contributor')))]",
            "properties": {
                "roleDefinitionId": "[variables('Resource Policy Contributor')]",
                "principalId": "[parameters('principalId')]"
            }
        }
    ]
}
```

### <a name="azure-cli"></a>CLI de Azure
Ejecute estos comandos:

```azurecli
az role assignment create --assignee-object-id <your Automanage Account Object ID> --role "Contributor" --scope /subscriptions/<your subscription ID>

az role assignment create --assignee-object-id <your Automanage Account Object ID> --role "Resource Policy Contributor" --scope /subscriptions/<your subscription ID>
```

## <a name="next-steps"></a>Pasos siguientes
[Más información sobre Azure Automanage](./automanage-virtual-machines.md)
