---
title: Salidas en plantillas
description: Se describe cómo definir valores de salida en una plantilla de Azure Resource Manager (plantilla de ARM).
ms.topic: conceptual
ms.date: 05/18/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: d81499ca7a85f8652ee353cde2e8e1cee48d4bf4
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111960346"
---
# <a name="outputs-in-arm-templates"></a>Salidas en plantillas de ARM

En este artículo se describe cómo definir valores de salida en una plantilla de Azure Resource Manager (plantilla de ARM). Las salidas se usan cuando es necesario devolver valores de los recursos implementados.

El formato de cada valor de salida debe resolverse como uno de los [tipos de datos](data-types.md).

## <a name="define-output-values"></a>Definición de valores de salida

En el ejemplo siguiente se muestra cómo devolver una propiedad desde un recurso implementado.

Agregue la sección `outputs` a la plantilla. El valor de salida obtiene el nombre de dominio completo de una dirección IP pública.

```json
"outputs": {
  "hostname": {
    "type": "string",
    "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPAddressName'))).dnsSettings.fqdn]"
  },
}
```

Si necesita generar una propiedad que tenga un guion en el nombre, use corchetes alrededor del nombre en lugar de la notación de puntos. Por ejemplo, use `['property-name']` en lugar de `.property-name`.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "variables": {
    "user": {
      "user-name": "Test Person"
    }
  },
  "resources": [
  ],
  "outputs": {
    "nameResult": {
      "type": "string",
      "value": "[variables('user')['user-name']]"
    }
  }
}
```

## <a name="conditional-output"></a>Salida condicional

Puede usar el elemento `condition` para devolver condicionalmente un valor. Por norma general, se usa una salida condicional cuando se [implementa condicionalmente](conditional-resource-deployment.md) un recurso. En el ejemplo siguiente, se muestra cómo se devuelve condicionalmente el identificador de recurso de una dirección IP pública en función de si se ha implementado una nueva:

```json
"outputs": {
  "resourceID": {
    "condition": "[equals(parameters('publicIpNewOrExisting'), 'new')]",
    "type": "string",
    "value": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIPAddresses_name'))]"
  }
}
```

Para obtener un ejemplo sencillo de salida condicional, consulte la [plantilla de salida condicional](https://github.com/bmoore-msft/AzureRM-Samples/blob/master/conditional-output/azuredeploy.json).

## <a name="dynamic-number-of-outputs"></a>Número dinámico de salidas

En algunos escenarios, no se conoce el número de instancias de un valor que se debe devolver al crear la plantilla. Puede devolver un número variable de valores mediante la salida iterativa. Agregue el elemento `copy` para iterar una salida.

```json
"outputs": {
  "storageEndpoints": {
    "type": "array",
    "copy": {
      "count": "[parameters('storageCount')]",
      "input": "[reference(concat(copyIndex(), variables('baseName'))).primaryEndpoints.blob]"
    }
  }
}
```

Para más información, vea [Iteración de salida en plantillas de ARM](copy-outputs.md).

## <a name="linked-templates"></a>Plantillas vinculadas

Puede implementar plantillas relacionadas mediante [plantillas vinculadas](linked-templates.md). Para recuperar el valor de salida de una plantilla vinculada, use la función [reference](template-functions-resource.md#reference) en la plantilla principal. La sintaxis de la plantilla principal es la siguiente:

```json
"[reference('<deploymentName>').outputs.<propertyName>.value]"
```

En el ejemplo siguiente, se muestra cómo establecer la dirección IP en un equilibrador de carga mediante la recuperación de un valor desde una plantilla vinculada.

```json
"publicIPAddress": {
  "id": "[reference('linkedTemplate').outputs.resourceID.value]"
}
```

Si el nombre de la propiedad tiene un guion, use corchetes alrededor del nombre en lugar de la notación de puntos.

```json
"publicIPAddress": {
  "id": "[reference('linkedTemplate').outputs['resource-ID'].value]"
}
```

No se puede usar la función `reference` en la sección de salidas de una [plantilla anidada](linked-templates.md#nested-template). Para devolver los valores de un recurso implementado en una plantilla anidada, convierta la plantilla anidada en una plantilla vinculada.

La [Plantilla de dirección IP pública](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/public-ip.json) crea una dirección IP pública y genera el identificador del recurso. La [Plantilla de equilibrador de carga](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/public-ip-parentloadbalancer.json) se vincula a la plantilla anterior. Usa el identificador de recurso en la salida al crear el equilibrador de carga.

## <a name="example-template"></a>Plantilla de ejemplo

La plantilla siguiente no implementa ningún recurso. Muestra algunas formas de devolver salidas de tipos diferentes.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/outputs.json":::

## <a name="get-output-values"></a>Obtención de valores de salida

Cuando la implementación se realiza correctamente, los valores de salida se devuelven automáticamente en los resultados de la implementación.

Para obtener valores de salida del historial de implementación, puede usar un script.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
(Get-AzResourceGroupDeployment `
  -ResourceGroupName <resource-group-name> `
  -Name <deployment-name>).Outputs.resourceID.value
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az deployment group show \
  -g <resource-group-name> \
  -n <deployment-name> \
  --query properties.outputs.resourceID.value
```

---

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre las propiedades disponibles para salidas, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](./syntax.md).