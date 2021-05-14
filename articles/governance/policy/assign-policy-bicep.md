---
title: 'Inicio rápido: nueva asignación de directiva con el archivo Bicep (versión preliminar)'
description: En este inicio rápido se usa un archivo Bicep (versión preliminar) para crear una asignación de directivas para identificar recursos no compatibles.
ms.date: 04/01/2021
ms.topic: quickstart
ms.custom: subject-bicepqs
ms.openlocfilehash: a1808114dd336d09af01beff6b8a47982444ce66
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108733543"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-by-using-a-bicep-file"></a>Creación de una asignación de directiva para identificar recursos no compatibles mediante un archivo Bicep

El primer paso para entender el cumplimiento en Azure es identificar el estado de sus recursos.
Este inicio rápido le guía por el proceso de usar un archivo [Bicep (versión preliminar)](https://github.com/Azure/bicep) creado en una plantilla de Resource Manager para crear una asignación de directivas con el objeto de identificar las máquinas virtuales que no usan discos administrados. Al finalizar este proceso, habrá identificado correctamente máquinas virtuales que no utilizan discos administrados. _No son compatibles_ con la asignación de directiva.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abre en Azure Portal.

:::image type="content" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Botón para implementar la plantilla de ARM para asignar una instancia de Azure Policy a Azure." border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azurepolicy-assign-builtinpolicy-resourcegroup%2Fazuredeploy.json":::

## <a name="prerequisites"></a>Prerrequisitos

- Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.
- Versión `0.3` de Bicep o posterior instalada. Si aún no tiene la CLI de Bicep o necesita actualizarla, consulte [Instalación de Bicep (versión preliminar)](../../azure-resource-manager/templates/bicep-install.md).

## <a name="review-the-bicep-file"></a>Revisión del archivo de Bicep

En este inicio rápido, creará una asignación de directiva y asignará una definición de directiva integrada denominada _Auditoría de máquinas virtuales que no usan discos administrados_ (`06a78e20-9358-41c9-923c-fb736d382a4d`). Para una lista parcial de las directivas integradas disponibles, consulte los [ejemplos de Azure Policy](./samples/index.md).

Cree el siguiente archivo Bicep como `assignment.bicep`:

```bicep
param policyAssignmentName string = 'audit-vm-manageddisks'
param policyDefinitionID string = '/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d'

resource assignment 'Microsoft.Authorization/policyAssignments@2019-09-01' = {
    name: policyAssignmentName
    properties: {
        scope: subscriptionResourceId('Microsoft.Resources/resourceGroups', resourceGroup().name)
        policyDefinitionId: policyDefinitionID
    }
}

output assignmentId string = assignment.id
```

El recurso definido en el archivo es el siguiente:

- [Microsoft.Authorization/policyAssignments](/azure/templates/microsoft.authorization/policyassignments)

## <a name="deploy-the-template"></a>Implementación de la plantilla

> [!NOTE]
> El servicio Azure Policy es gratuito. Para más información, consulte la [Introducción a Azure Policy](./overview.md).

Después de instalar la CLI de Bicep y crear el archivo, puede implementar el archivo Bicep con:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroupDeployment `
  -Name PolicyDeployment `
  -ResourceGroupName PolicyGroup `
  -TemplateFile assignment.bicep
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az deployment group create \
  --name PolicyDeployment \
  --resource-group PolicyGroup \
  --template-file assignment.bicep
```

---

Algunos recursos adicionales:

- Puede encontrar más plantillas de ejemplo en [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Authorization&pageNumber=1&sort=Popular).
- Para ver la referencia de plantilla, vaya a la [referencia de plantilla de Azure](/azure/templates/microsoft.authorization/allversions).
- Para aprender a desarrollar plantillas de Resource Manager, consulte la [documentación de Azure Resource Manager](../../azure-resource-manager/management/overview.md).
- Para información sobre la implementación de nivel de suscripción, consulte [Create resource groups and resources at the subscription level](../../azure-resource-manager/templates/deploy-to-subscription.md) (Creación de grupos de recursos y recursos en el nivel de suscripción).

## <a name="validate-the-deployment"></a>Validación de la implementación

Seleccione **Cumplimiento** en el panel izquierdo de la página. A continuación, busque la asignación de directiva _Auditoría de máquinas virtuales que no usan discos administrados_ que ha creado.

:::image type="content" source="./media/assign-policy-template/policy-compliance.png" alt-text="Captura de pantalla de los detalles de cumplimiento en la página Cumplimiento de directivas." border="false":::

Si hay algún recurso existente no compatible con esta nueva asignación, aparecerá en la pestaña **Recursos no compatibles**.

Para más información, consulte [How compliance works](./how-to/get-compliance-data.md#how-compliance-works) (Funcionamiento del cumplimiento).

## <a name="clean-up-resources"></a>Limpieza de recursos

Para quitar la asignación creada, siga estos pasos:

1. Seleccione **Cumplimiento** (o **Asignaciones**) en el lado izquierdo de página Azure Policy y busque la asignación de directiva _Auditoría de máquinas virtuales que no usan discos administrados_ asignación de directiva que ha creado.

1. Haga clic con el botón derecho en la asignación de directiva _Auditoría de máquinas virtuales que no usan discos administrados_ y seleccione **Eliminar asignación**.

   :::image type="content" source="./media/assign-policy-template/delete-assignment.png" alt-text="Captura de pantalla del uso del menú contextual para eliminar una asignación desde la página Cumplimiento." border="false":::

1. Elimine el archivo `assignment.bicep`.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido se asigna una definición de directiva integrada a un ámbito y se evalúa su informe de cumplimiento. La definición de la directiva confirma que todos los recursos del ámbito son compatibles y se identifican cuáles no lo son.

Para más información sobre la asignación de directivas para garantizar la compatibilidad de los nuevos recursos, continúe con el tutorial para:

> [!div class="nextstepaction"]
> [Crear y administrar directivas](./tutorials/create-and-manage.md)
