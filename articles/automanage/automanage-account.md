---
title: Cuenta de Azure Automanage
description: Obtenga información sobre cómo funciona una cuenta de Automanage y cómo crear una.
author: asinn826
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: conceptual
ms.date: 04/07/2021
ms.author: alsin
ms.openlocfilehash: b79e061ae00c42ed2ec2ac39f5653a868f09a15f
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107368573"
---
# <a name="automanage-accounts"></a>Cuentas de Automanage

La cuenta de Automanage es la identidad que usa el servicio Automanage para realizar sus operaciones automatizadas.

En la experiencia de Azure Portal, al habilitar Automanage en las máquinas virtuales, hay una lista desplegable de opciones avanzadas en la hoja **Enable Azure VM best practice** (Habilitar procedimiento recomendado de máquina virtual de Azure) que le permite asignar o crear manualmente la cuenta de Automanage.

A la cuenta de Automanage se le concederán los roles **Colaborador** y **Colaborador de directivas de recursos** a las suscripciones que contienen las máquinas que se incorporan a Automanage. Puede usar la misma cuenta de Automanage en máquinas en varias suscripciones, lo que permitirá conceder a dicha cuenta de Automanage los permisos **Colaborador** y **Colaborador de directivas de recursos** en todas las suscripciones.

Si la máquina virtual está conectada a un área de trabajo de Log Analytics en otra suscripción, se le concederán a la cuenta de Automanage **Colaborador** y **Colaborador de directivas de recursos** también en esa otra suscripción.

Si va a habilitar Automanage con una cuenta nueva de Automanage, necesita los siguientes permisos en la suscripción: el rol **Propietario** o **Colaborador** junto con el rol **Administrador de acceso de usuario**.

Si habilita Automanage con una cuenta de Automanage existente, debe tener el rol **Colaborador** en el grupo de recursos que contiene las máquinas virtuales.

> [!NOTE]
> Al deshabilitar los procedimientos recomendados de Automanage, se conservarán los permisos de la cuenta de Automanage en las suscripciones asociadas. Para quitar manualmente los permisos, vaya a la página IAM de la suscripción o elimine la cuenta de Automanage. No se puede eliminar la cuenta de Automanage si sigue administrando las máquinas.

## <a name="create-an-automanage-account"></a>Creación de una cuenta de Automanage
Puede crear una cuenta de Automanage mediante el portal o mediante una plantilla de ARM.

### <a name="portal"></a>Portal
1. Vaya a la hoja **Automanage** del portal.
1. Haga clic en **Habilitar en la máquina existente**.
1. En **Avanzado**, haga clic en "Crear una nueva cuenta".
1. Rellene los campos obligatorios y haga clic en **Crear**.

### <a name="arm-template"></a>Plantilla ARM
La creación de una cuenta de Automanage mediante una plantilla de Resource Manager requiere 2 pasos:
1. Creación de la cuenta de Automanage
1. Concesión de los permisos suficientes a la cuenta para permitirle realizar operaciones
    1. Necesitará el identificador de objeto de la cuenta que creó para este paso.
        1. Los pasos para encontrar los detalles de la entidad de servicio de la cuenta (incluido el identificador de objeto) están disponibles [aquí](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/how-to-view-managed-identity-service-principal-portal#view-the-service-principal).
    1. Una vez que haya encontrado la entidad de servicio, copie el **identificador de objeto**. Guárdelo, ya que lo necesitará para delegar permisos a continuación.

#### <a name="1-create-automanage-account-does-not-grant-permissions-to-it"></a>1. Creación de una cuenta de Automanage (no le concede permisos a esta)
Para crear una cuenta de Automanage, guarde la siguiente plantilla de Resource Manager como `azuredeploy.json` y ejecute el siguiente comando de la CLI de Azure. Una vez que haya terminado, pase a la segunda plantilla siguiente para delegar permisos suficientes para la cuenta.

```azurecli-interactive
az deployment group create --resource-group <resource group name> --template-file azuredeploy.json
```

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "automanageAccountName": {
            "type": "String"
        },
        "location": {
            "type": "String"
        }
    },
    "resources": [
        {
            "apiVersion": "2020-06-30-preview",
            "type": "Microsoft.Automanage/accounts",
            "name": "[parameters('automanageAccountName')]",
            "location": "[parameters('location')]",
            "identity": {
                "type": "SystemAssigned"
            }
        }
    ]
}
```
#### <a name="2-grant-permissions-to-the-automanage-account"></a>2. Concesión de permisos a la cuenta de Automanage
Para conceder permisos suficientes a la cuenta de Automanage, deberá hacer lo siguiente:
1. Guarde la siguiente plantilla de Resource Manager como `azuredeploy2.json` y ejecute el siguiente comando de la CLI de Azure.
1. Cuando se le solicite, especifique el identificador de objeto de la cuenta de Automanage que creó y guardó.

```azurecli-interactive
az deployment group create --resource-group <resource group name> --template-file azuredeploy.json
```
```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
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
        "contributorRoleDefinitionID": "/providers/Microsoft.Authorization/roledefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
        "resourcePolicyContributorRoleDefinitionID": "/providers/Microsoft.Authorization/roledefinitions/36243c78-bf99-498c-9df9-86d9f8d28608"
    },
    "resources": [
        {
            "type": "Microsoft.Authorization/roleAssignments",
            "apiVersion": "2020-04-01-preview",
            "name": "[guid(variables('contributorRoleDefinitionID'))]",
            "properties": {
                "roleDefinitionId": "[variables('contributorRoleDefinitionID')]",
                "principalId": "[parameters('principalId')]"
            }
        },
        {
            "type": "Microsoft.Authorization/roleAssignments",
            "apiVersion": "2020-04-01-preview",
            "name": "[guid(variables('resourcePolicyContributorRoleDefinitionID'))]",
            "properties": {
                "roleDefinitionId": "[variables('resourcePolicyContributorRoleDefinitionID')]",
                "principalId": "[parameters('principalId')]"
            }
        }
    ]
}
```

## <a name="next-steps"></a>Pasos siguientes
* Más información sobre los servicios de Automanage para [Linux](./automanage-linux.md) y [Windows](./automanage-windows-server.md)