---
title: Funciones de plantilla en implementaciones con ámbito
description: Se describe cómo se resuelven las funciones de plantilla en implementaciones con ámbito. El ámbito puede ser un inquilino, grupos de administración, suscripciones y grupos de recursos.
ms.topic: conceptual
ms.date: 10/22/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: db2099c6c2405928a33f33bbba579c2280ad4877
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111959990"
---
# <a name="arm-template-functions-in-deployment-scopes"></a>Funciones de plantilla de Resource Manager en ámbitos de implementación

Con las plantillas de Azure Resource Manager, puede realizar implementaciones en grupos de recursos, suscripciones, grupos de administración o inquilinos. Por lo general, las [funciones de plantilla de Resource Manager](template-functions.md) funcionan igual en todos los ámbitos. En este artículo se describen las diferencias que existen en algunas funciones según el ámbito.

## <a name="supported-functions"></a>Funciones admitidas

Al realizar la implementación en distintos ámbitos, hay algunos aspectos importantes que se deben tener en cuenta:

* La función [resourceGroup()](template-functions-resource.md#resourcegroup) se **admite** con implementaciones de grupos de recursos.
* La función [subscription()](template-functions-resource.md#subscription) se **admite** con implementaciones de grupos de recursos y suscripciones.
* Las funciones [reference()](template-functions-resource.md#reference) y [list()](template-functions-resource.md#list) se **admiten** con todos los ámbitos.
* Use [resourceId()](template-functions-resource.md#resourceid) para obtener el identificador de un recurso implementado en el grupo de recursos.

  ```json
  "subnet": {
    "id": "[resourceId(parameters('virtualNetworkResourceGroup'), 'Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworkName'), parameters('subnet1Name'))]"
  }
  ```

* Use la función [subscriptionResourceId()](template-functions-resource.md#subscriptionresourceid) para obtener el identificador de un recurso implementado en la suscripción.

  Por ejemplo, para obtener el identificador de recurso de una definición de directiva que se implementa en una suscripción, use:

  ```json
  "roleDefinitionId": "[subscriptionResourceId('Microsoft.Authorization/roleDefinitions', 'acdd72a7-3385-48ef-bd42-f606fba81ae7')]"
  ```

* Use la función [extensionResourceId()](template-functions-resource.md#extensionresourceid) con los recursos que se implementan como extensiones del grupo de administración. Las definiciones de directivas personalizadas que se implementan en un grupo de administración son extensiones del grupo de administración.

  Para obtener el identificador de recurso de una definición de directiva personalizada en el nivel de grupo de administración, use:

  ```json
  "policyDefinitionId": "[extensionResourceId(variables('mgScope'), 'Microsoft.Authorization/policyDefinitions', parameters('policyDefinitionID'))]"
  ```

* Use la función [tenantResourceId()](template-functions-resource.md#tenantresourceid) para obtener el identificador de un recurso implementado en el inquilino. Las definiciones de directivas integradas son recursos del nivel de inquilino. Al asignar una directiva integrada en el nivel de grupo de administración, use la función tenantResourceId.

  Para obtener el identificador de recurso de una definición de directiva integrada, utilice:

  ```json
  "policyDefinitionId": "[tenantResourceId('Microsoft.Authorization/policyDefinitions', parameters('policyDefinitionID'))]"
  ```

## <a name="function-resolution-in-scopes"></a>Resolución de funciones en ámbitos

Al implementar en más de un ámbito, las funciones [resourceGroup()](template-functions-resource.md#resourcegroup) y [subscription()](template-functions-resource.md#subscription) se resuelven de manera diferente en función de cómo se especifique la plantilla. Cuando se vincula a una plantilla externa, las funciones siempre se resuelven en el ámbito de esa plantilla. Al anidar una plantilla dentro de una plantilla primaria, use la propiedad `expressionEvaluationOptions` para especificar si las funciones se resuelven en el grupo de recursos y en la suscripción de la plantilla primaria o en la plantilla anidada. Establezca la propiedad en `inner` para resolver el ámbito de la plantilla anidada. Establezca la propiedad en `outer` para resolver el ámbito de la plantilla primaria.

En la tabla siguiente se muestra si las funciones se resuelven en el grupo de recursos y la suscripción primarios o insertados.

| Tipo de plantilla | Ámbito | Solución |
| ------------- | ----- | ---------- |
| anidada        | exterior (valor predeterminado) | Grupo de recursos primario |
| anidada        | interna | Grupo de subrecursos |
| Vinculado        | N/D   | Grupo de subrecursos |

En la [plantilla de ejemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/crossresourcegroupproperties.json) se muestra:

* plantilla anidada con ámbito predeterminado (exterior)
* plantilla anidada con ámbito interno
* plantilla vinculada

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/crossresourcegroupproperties.json":::

Para probar la plantilla anterior y ver los resultados, use PowerShell o la CLI de Azure.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroup -Name parentGroup -Location southcentralus
New-AzResourceGroup -Name inlineGroup -Location southcentralus
New-AzResourceGroup -Name linkedGroup -Location southcentralus

New-AzResourceGroupDeployment `
  -ResourceGroupName parentGroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crossresourcegroupproperties.json
```

El resultado del ejemplo anterior es:

```output
 Name             Type                       Value
 ===============  =========================  ==========
 parentRG         String                     Parent resource group is parentGroup
 defaultScopeRG   String                     Default scope resource group is parentGroup
 innerScopeRG     String                     Inner scope resource group is inlineGroup
 linkedRG         String                     Linked resource group is linkedgroup
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az group create --name parentGroup --location southcentralus
az group create --name inlineGroup --location southcentralus
az group create --name linkedGroup --location southcentralus

az deployment group create \
  --name ExampleDeployment \
  --resource-group parentGroup \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crossresourcegroupproperties.json
```

El resultado del ejemplo anterior es:

```output
"outputs": {
  "defaultScopeRG": {
    "type": "String",
    "value": "Default scope resource group is parentGroup"
  },
  "innerScopeRG": {
    "type": "String",
    "value": "Inner scope resource group is inlineGroup"
  },
  "linkedRG": {
    "type": "String",
    "value": "Linked resource group is linkedGroup"
  },
  "parentRG": {
    "type": "String",
    "value": "Parent resource group is parentGroup"
  }
},
```

---

## <a name="next-steps"></a>Pasos siguientes

* Para entender cómo definir parámetros en la plantilla, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](./syntax.md).
* Para obtener sugerencias para resolver los errores de implementación más comunes, consulte [Solución de errores comunes de implementación de Azure con Azure Resource Manager](common-deployment-errors.md).
* Para más información sobre la implementación de una plantilla que requiere un token de SAS, vea [Implementación de una plantilla de Resource Manager privada con el token de SAS](secure-template-with-sas-token.md).