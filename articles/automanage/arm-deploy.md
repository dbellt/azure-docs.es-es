---
title: Incorporación de una máquina a Azure Automanage con una plantilla de ARM
description: Aprenda a incorporar una máquina a Azure Automanage con una plantilla de Azure Resource Manager.
author: asinn826
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: how-to
ms.date: 04/09/2021
ms.author: alsin
ms.openlocfilehash: 78cf28903311c542a83c9ace4f794e1cdda9a61c
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107368537"
---
# <a name="onboard-a-machine-to-automanage-with-an-azure-resource-manager-arm-template"></a>Incorporación de una máquina a Automanage con una plantilla de Azure Resource Manager (ARM)


## <a name="overview"></a>Información general
Siga los pasos que se indican a continuación para incorporar una máquina a los procedimientos recomendados de Automanage. La plantilla de ARM siguiente creará un objeto `configurationProfileAssignment`, es decir, el recurso de Azure que representa a una máquina incorporada a Automanage.

## <a name="prerequisites"></a>Requisitos previos
* Debe haber creado una cuenta de Automanage. Consulte [este documento](./automanage-account.md) para obtener más información sobre las cuentas de Automanage y cómo crear una.
* Debe tener el rol **Colaborador** en el grupo de recursos que contiene las máquinas que quiere incorporar a Automanage.

## <a name="arm-template-overview"></a>Introducción a la plantilla de ARM
La plantilla de ARM siguiente incorporará la máquina especificada a los procedimientos recomendados de Azure Automanage. Los detalles sobre la plantilla de ARM y los pasos sobre cómo implementarla se encuentran en la sección de implementación de la plantilla de ARM que aparece [a continuación](#arm-template-deployment).
```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "machineName": {
            "type": "String"
        },
        "automanageAccountName": {
            "type": "String"
        },
        "configurationProfileAssignment": {
            "type": "String"
        }
    },
    "resources": [
        {
            "type": "Microsoft.Compute/virtualMachines/providers/configurationProfileAssignments",
            "apiVersion": "2020-06-30-preview",
            "name": "[concat(parameters('machineName'), '/Microsoft.Automanage/', 'default')]",
            "properties": {
                "configurationProfile": "[parameters('configurationProfileAssignment')]",
                "accountId": "[concat(resourceGroup().id, '/providers/Microsoft.Automanage/accounts/', parameters('automanageAccountName'))]"
            }
        }
    ]
}
```

## <a name="arm-template-deployment"></a>Implementación de la plantilla de ARM
La plantilla de ARM anterior creará una asignación del perfil de configuración para la máquina especificada mediante una cuenta de Automanage concreta. Si aún no ha creado una cuenta de Automanage, obtenga más información al respecto en [este documento.](./automanage-account.md)

El valor `configurationProfileAssignment` puede ser uno de los siguientes:
* "Production"
* "DevTest"

Siga estos pasos para implementar la plantilla de ARM:
1. Guarde la plantilla de ARM siguiente como `azuredeploy.json`.
1. Ejecute la implementación de la plantilla de ARM con `az deployment group create --resource-group myResourceGroup --template-file azuredeploy.json`.
1. Proporcione los valores para machineName, automanageAccountName y configurationProfileAssignment cuando se le solicite.
1. Ya está todo listo.

Al igual que con cualquier plantilla de ARM, se pueden simplificar los parámetros en un archivo `azuredeploy.parameters.json` independiente y usarlo como argumento al implementar.

## <a name="next-steps"></a>Pasos siguientes
Obtener más información acerca de Automanage para [Linux](./automanage-linux.md) y [Windows](./automanage-windows-server.md)