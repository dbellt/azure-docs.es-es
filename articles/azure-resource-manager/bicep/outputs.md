---
title: Salidas en Bicep
description: Se describe cómo definir variables de salida en Bicep.
ms.topic: conceptual
ms.date: 06/01/2021
ms.openlocfilehash: 95366013ed972e0251c99732ea4b344b5b94e743
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "111027221"
---
# <a name="outputs-in-bicep"></a>Salidas en Bicep

En este artículo se describe cómo definir valores de salida en una plantilla de Azure Resource Manager (plantilla de ARM) y en un archivo Bicep. Las salidas se usan cuando es necesario devolver valores de los recursos implementados.

El formato de cada valor de salida debe resolverse como uno de los [tipos de datos](data-types.md).

## <a name="define-output-values"></a>Definición de valores de salida

En el ejemplo siguiente se muestra cómo usar la palabra clave `output` para devolver una propiedad desde un recurso implementado.

En el ejemplo siguiente, `publicIP` es el identificador (nombre simbólico) de una dirección IP pública implementada en el archivo de Bicep. El valor de salida obtiene el nombre de dominio completo de la dirección IP pública.

```bicep
output hostname string = publicIP.properties.dnsSettings.fqdn
```

Si necesita generar una propiedad que tenga un guion en el nombre, use corchetes alrededor del nombre en lugar de la notación de puntos. Por ejemplo, use `['property-name']` en lugar de `.property-name`.

```bicep
var user = {
  'user-name': 'Test Person'
}

output stringOutput string = user['user-name']
```

## <a name="conditional-output"></a>Salida condicional

Puede devolver un valor de forma condicional. Por norma general, se usa una salida condicional cuando se [implementa condicionalmente](conditional-resource-deployment.md) un recurso. En el ejemplo siguiente, se muestra cómo se devuelve condicionalmente el identificador de recurso de una dirección IP pública en función de si se ha implementado una nueva:

Para especificar una salida condicional en Bicep, use el operador `?`. En el ejemplo siguiente se devuelve una dirección URL de punto de conexión o una cadena vacía en función de una condición.

```bicep
param deployStorage bool = true
param storageName string
param location string = resourceGroup().location

resource myStorageAccount 'Microsoft.Storage/storageAccounts@2019-06-01' = if (deployStorage) {
  name: storageName
  location: location
  kind: 'StorageV2'
  sku:{
    name:'Standard_LRS'
    tier: 'Standard'
  }
  properties: {
    accessTier: 'Hot'
  }
}

output endpoint string = deployStorage ? myStorageAccount.properties.primaryEndpoints.blob : ''
```

## <a name="dynamic-number-of-outputs"></a>Número dinámico de salidas

En algunos escenarios, no se conoce el número de instancias de un valor que se debe devolver al crear la plantilla. Puede devolver un número variable de valores mediante la salida iterativa.

En Bicep, agregue una expresión `for` que defina las condiciones de la salida dinámica. En el ejemplo siguiente se recorre en iteración una matriz.

```bicep
param nsgLocation string = resourceGroup().location
param nsgNames array = [
  'nsg1'
  'nsg2'
  'nsg3'
]

resource nsg 'Microsoft.Network/networkSecurityGroups@2020-06-01' = [for name in nsgNames: {
  name: name
  location: nsgLocation
}]

output nsgs array = [for (name, i) in nsgNames: {
  name: nsg[i].name
  resourceId: nsg[i].id
}]
```

También puede recorrer en iteración un intervalo de enteros. Para más información, consulte [Iteración de salida en Bicep](loop-outputs.md).

## <a name="modules"></a>Módulos

Puede implementar plantillas relacionadas mediante módulos. Para recuperar un valor de salida de un módulo, use la siguiente sintaxis:

```bicep
<module-name>.outputs.<property-name>
```

En el ejemplo siguiente, se muestra cómo establecer la dirección IP en un equilibrador de carga mediante la recuperación de un valor desde un módulo. El nombre del módulo es `publicIP`.

```bicep
publicIPAddress: {
  id: publicIP.outputs.resourceID
}
```

## <a name="example-template"></a>Plantilla de ejemplo

La plantilla siguiente no implementa ningún recurso. Muestra algunas formas de devolver salidas de tipos diferentes.

Actualmente, Bicep no admite bucles.

:::code language="bicep" source="~/resourcemanager-templates/azure-resource-manager/outputs.bicep":::

## <a name="get-output-values"></a>Obtención de valores de salida

Cuando la implementación se realiza correctamente, los valores de salida se devuelven automáticamente en los resultados de la implementación.

Para obtener valores de salida del historial de implementación, puede usar la CLI de Azure o un script de Azure PowerShell.

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

* Para información sobre las propiedades disponibles para las salidas, consulte [Descripción de la estructura y la sintaxis de Bicep](./file.md).
