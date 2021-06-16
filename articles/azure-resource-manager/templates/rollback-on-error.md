---
title: Revertir en caso de error a una implementación correcta
description: Especifique que una implementación con errores debe revertir a una implementación correcta.
ms.topic: conceptual
ms.date: 02/02/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: d66df9cafdb64c216896a66b3283277930a63ed0
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111960074"
---
# <a name="rollback-on-error-to-successful-deployment"></a>Revertir en caso de error a una implementación correcta

Cuando se produce un error en una implementación, puede ejecutar automáticamente desde el historial de implementación una implementación anterior que sea correcta. Esta funcionalidad es útil si tiene un estado correcto conocido para la implementación de la infraestructura y quiere volver a ese estado. Puede especificar una implementación anterior concreta o la última implementación correcta.

> [!IMPORTANT]
> Esta característica revierte una implementación con errores mediante la reimplementación de una implementación anterior. Este resultado puede ser diferente de lo que cabría esperar al deshacer la implementación con errores. Asegúrese de que comprende cómo se reimplementa la implementación anterior.

## <a name="considerations-for-redeploying"></a>Consideraciones acerca de la reimplementación

Antes de usar esta característica, tenga en cuenta los detalles sobre cómo se controla la reimplementación:

- La implementación anterior se ejecuta mediante el [modo completo](./deployment-modes.md#complete-mode), incluso si usó el [modo incremental](./deployment-modes.md#incremental-mode) durante la implementación anterior. La reimplementación en el modo completo podría producir resultados inesperados si la implementación anterior usó el modo incremental. El modo completo significa que se eliminarán todos los recursos no incluidos en la implementación anterior. Especifique una implementación anterior que represente todos los recursos y estados que desee que existan en el grupo de recursos. Para más información, consulte [Modos de implementación](./deployment-modes.md).
- La reimplementación se ejecuta exactamente como se ejecutó previamente y con los mismos parámetros. Los parámetros no se pueden cambiar.
- La reimplementación solo afecta a los recursos; los cambios en los datos no se ven afectados.
- Esta característica solo se puede usar con implementaciones de grupos de recursos. No es compatible con implementaciones en el nivel de suscripción, grupo de administración o inquilino. Para más información sobre las implementaciones en el nivel de suscripción, consulte [Creación de grupos de recursos y otros recursos en el nivel de suscripción](./deploy-to-subscription.md).
- Solo se puede usar esta opción con las implementaciones de nivel de raíz. Las implementaciones de una plantilla anidada no están disponibles para volver a implementarse.

Para usar esta opción, las implementaciones deben tener nombres únicos en el historial de implementaciones. Solo se puede identificar una implementación concreta mediante estos nombres únicos. Si no hay nombres únicos, una implementación con errores podría sobrescribir una implementación anteriormente correcta en el historial.

Si especifica una implementación anterior que no existe en el historial de implementaciones, la reversión devuelve un error.

## <a name="powershell"></a>PowerShell

Para volver a implementar la última implementación correcta, agregue el parámetro `-RollbackToLastDeployment` como una marca.

```azurepowershell-interactive
New-AzResourceGroupDeployment -Name ExampleDeployment02 `
  -ResourceGroupName $resourceGroupName `
  -TemplateFile c:\MyTemplates\azuredeploy.json `
  -RollbackToLastDeployment
```

Para volver a implementar una implementación específica, use el parámetro `-RollBackDeploymentName` y proporcione el nombre de la implementación. La implementación especificada debe haberse realizado correctamente.

```azurepowershell-interactive
New-AzResourceGroupDeployment -Name ExampleDeployment02 `
  -ResourceGroupName $resourceGroupName `
  -TemplateFile c:\MyTemplates\azuredeploy.json `
  -RollBackDeploymentName ExampleDeployment01
```

## <a name="azure-cli"></a>Azure CLI

Para volver a implementar la última implementación correcta, agregue el parámetro `--rollback-on-error` como una marca.

```azurecli-interactive
az deployment group create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters storageAccountType=Standard_GRS \
  --rollback-on-error
```

Para volver a implementar una implementación específica, use el parámetro `--rollback-on-error` y proporcione el nombre de la implementación. La implementación especificada debe haberse realizado correctamente.

```azurecli-interactive
az deployment group create \
  --name ExampleDeployment02 \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters storageAccountType=Standard_GRS \
  --rollback-on-error ExampleDeployment01
```

## <a name="rest-api"></a>API DE REST

Para volver a implementar la última implementación correcta si se produce un error en la actual, use:

```json
{
  "properties": {
    "templateLink": {
      "uri": "http://mystorageaccount.blob.core.windows.net/templates/template.json",
      "contentVersion": "1.0.0.0"
    },
    "mode": "Incremental",
    "parametersLink": {
      "uri": "http://mystorageaccount.blob.core.windows.net/templates/parameters.json",
      "contentVersion": "1.0.0.0"
    },
    "onErrorDeployment": {
      "type": "LastSuccessful",
    }
  }
}
```

Para volver a implementar una implementación específica si se produce un error en la implementación actual, use:

```json
{
  "properties": {
    "templateLink": {
      "uri": "http://mystorageaccount.blob.core.windows.net/templates/template.json",
      "contentVersion": "1.0.0.0"
    },
    "mode": "Incremental",
    "parametersLink": {
      "uri": "http://mystorageaccount.blob.core.windows.net/templates/parameters.json",
      "contentVersion": "1.0.0.0"
    },
    "onErrorDeployment": {
      "type": "SpecificDeployment",
      "deploymentName": "<deploymentname>"
    }
  }
}
```

La implementación especificada debe haberse realizado correctamente.

## <a name="next-steps"></a>Pasos siguientes

- Para comprender los modos completo e incremental, consulte [Modos de implementación de Azure Resource Manager](deployment-modes.md).
- Para entender cómo definir parámetros en la plantilla, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](./syntax.md).