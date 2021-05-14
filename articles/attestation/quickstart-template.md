---
title: Creación de un certificado de Azure Attestation mediante una plantilla de Azure Resource Manager
description: Aprenda a crear un certificado de Azure Attestation mediante una plantilla de Azure Resource Manager.
services: azure-resource-manager
author: msmbaldwin
ms.service: azure-resource-manager
ms.topic: quickstart
ms.custom: subject-armqs, devx-track-azurepowershell
ms.author: mbaldwin
ms.date: 05/20/2021
ms.openlocfilehash: 2aa4c1b95677dd2257751fd27270d927d700bdb6
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109750562"
---
# <a name="quickstart-create-an-azure-attestation-provider-with-an-arm-template"></a>Inicio rápido: Creación de un proveedor de Azure Attestation con una plantilla de Resource Manager

[Microsoft Azure Attestation](overview.md) es una solución para la atestación de entornos de ejecución de confianza (TEE). Este inicio rápido se centra en el proceso de implementación de una plantilla de Azure Resource Manager para crear una directiva de Microsoft Azure Attestation.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementar en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.attestation%2Fattestation-provider-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-attestation-provider-create).

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.attestation/attestation-provider-create/azuredeploy.json":::

Recursos de Azure definidos en la plantilla:

- Microsoft.Attestation/attestationProviders

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir la plantilla.

    [![Implementar en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.attestation%2Fattestation-provider-create%2Fazuredeploy.json)

1. Seleccione o escriba los siguientes valores.

    A menos que se especifique otra cosa, utilice el valor predeterminado para crear el proveedor de atestación.

    - **Nombre del proveedor de atestación**: seleccione un nombre para el proveedor de Azure Attestation.
    - **Ubicación**: Seleccione una ubicación. Por ejemplo, **Centro de EE. UU**.
    - **Etiquetas**: Seleccione una ubicación. Por ejemplo, **Centro de EE. UU**.

1. Seleccione **Comprar**. Cuando el recurso de atestación se haya implementado correctamente, recibirá una notificación.

Azure Portal se usa para implementar la plantilla. Además de Azure Portal, también puede usar Azure PowerShell, la CLI de Azure y API REST. Para obtener información sobre otros métodos de implementación, consulte [Implementación de plantillas](../azure-resource-manager/templates/deploy-powershell.md).

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Puede usar Azure Portal para comprobar el recurso de atestación.

## <a name="clean-up-resources"></a>Limpieza de recursos

Otros artículos de Azure Attestation se basan en este inicio rápido. Si tiene pensado seguir trabajando en otras guías de inicio rápido y tutoriales, considere la posibilidad de dejar estos recursos activos.

Cuando ya no lo necesite, elimine el grupo de recursos, que elimina el recurso de Azure Attestation. Para eliminar el grupo de recursos mediante la CLI de Azure o Azure PowerShell:

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

---

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido ha creado un recurso de atestación mediante una plantilla de Resource Manager y ha validado la implementación. Para más información sobre Azure Attestation, consulte [Introducción a Azure Attestation](overview.md).
