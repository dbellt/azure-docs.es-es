---
title: Creación de un Microsoft Azure Confidential Ledger mediante una plantilla de Azure Resource Manager
description: Aprenda a crear un Microsoft Azure Confidential Ledger mediante una plantilla de Azure Resource Manager.
services: azure-resource-manager
author: msmbaldwin
ms.service: azure-resource-manager
ms.topic: quickstart
ms.custom: subject-armqs, devx-track-azurepowershell
ms.author: mbaldwin
ms.date: 04/15/2021
ms.openlocfilehash: 1110d1441c63f3af1f7c31b9ac090c9693b36be1
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388595"
---
# <a name="quickstart-create-an-microsoft-azure-confidential-ledger-with-an-arm-template"></a>Inicio rápido: crear un Microsoft Azure Confidential Ledger con una plantilla de ARM

[Microsoft Azure Confidential Ledger](overview.md) es un servicio nuevo y muy seguro para administrar registros de datos confidenciales. Este inicio rápido se centra en el proceso de implementación de una plantilla de Azure Resource Manager para crear un libro de contabilidad nuevo.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementar en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-confidential-ledger-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://github.com/Azure/azure-quickstart-templates).

Recursos de Azure definidos en la plantilla:

- Microsoft.ConfidentialLedger/ledgers

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir la plantilla.

    [![Implementar en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-confidential-ledger-create%2Fazuredeploy.json)

1. Seleccione o escriba los siguientes valores.

    A menos que se especifique otra cosa, utilice el valor predeterminado para crear el Confidential Ledger.

    - **Nombre del libro de contabilidad**: seleccione un nombre para el libro de contabilidad. Los nombres de libros de contabilidad deben ser únicos globalmente.
    - **Ubicación**: Seleccione una ubicación. Por ejemplo **Este de EE. UU.**

1. Seleccione **Comprar**. Una vez que el recurso Confidential Ledger se haya implementado correctamente, recibirá una notificación.

Azure Portal se usa para implementar la plantilla. Además de Azure Portal, también puede usar Azure PowerShell, la CLI de Azure y API REST. Para obtener información sobre otros métodos de implementación, consulte [Implementación de plantillas](../azure-resource-manager/templates/deploy-powershell.md).

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Puede usar Azure Portal para comprobar el recurso del libro de contabilidad.

## <a name="clean-up-resources"></a>Limpieza de recursos

Otros artículos Microsoft Azure Confidential Ledger se pueden basar en esta guía de inicio rápido. Si tiene pensado seguir trabajando en otras guías de inicio rápido y tutoriales, considere la posibilidad de dejar estos recursos activos.

Cuando ya no lo necesite, elimine el grupo de recursos, que elimina el recurso del libro de contabilidad. Para eliminar el grupo de recursos mediante la CLI de Azure o Azure PowerShell:

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

En esta guía de inicio rápido, ha creado un recurso Confidential Ledger mediante una plantilla de ARM y ha validado la implementación. Para obtener más información sobre el servicio, consulte la [Información general de Microsoft Azure Confidential Ledger](overview.md).


