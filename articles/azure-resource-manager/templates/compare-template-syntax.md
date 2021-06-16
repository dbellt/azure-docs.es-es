---
title: Comparación de la sintaxis de las plantillas de Azure Resource Manager en JSON y Bicep
description: Compara las plantillas de Azure Resource Manager desarrolladas con JSON y Bicep, y muestra cómo realizar la conversión entre los dos lenguajes.
ms.topic: conceptual
ms.date: 03/12/2021
ms.openlocfilehash: 225e52e9534a77a01502b762f043a4f34df19caa
ms.sourcegitcommit: 5c136a01bddfccb2cc9f7e7e7741e2cf2651ddbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111353512"
---
# <a name="comparing-json-and-bicep-for-templates"></a>Comparación de JSON y Bicep para plantillas

En este artículo se compara la sintaxis de Bicep con la sintaxis de JSON para las plantillas de Azure Resource Manager (plantillas de ARM). En la mayoría de los casos, Bicep proporciona una sintaxis menos detallada que el equivalente en JSON.

Si ya tiene experiencia con JSON para desarrollar plantillas de ARM, use los siguientes ejemplos para obtener información sobre los equivalentes de sintaxis para Bicep.

## <a name="expressions"></a>Expresiones

Para crear una expresión:

```bicep
func()
```

```json
"[func()]"
```

## <a name="parameters"></a>Parámetros

Para declarar un parámetro con un valor predeterminado:

```bicep
param demoParam string = 'Contoso'
```

```json
"parameters": {
  "demoParam": {
    "type": "string",
    "defaultValue": "Contoso"
  }
}
```

Para obtener el valor de un parámetro:

```bicep
demoParam
```

```json
[parameters('demoParam'))]
```

## <a name="variables"></a>Variables

Para declarar una variable:

```bicep
var demoVar = 'example value'
```

```json
"variables": {
  "demoVar": "example value"
},
```

Para obtener el valor de una variable:

```bicep
demoVar
```

```json
[variables('demoVar'))]
```

## <a name="strings"></a>Cadenas

Para concatenar cadenas:

```bicep
'${namePrefix}-vm'
```

```json
[concat(parameters('namePrefix'), '-vm')]
```

## <a name="logical-operators"></a>Operadores lógicos

Para devolver el operador **AND** lógico:

```bicep
isMonday && isNovember
```

```json
[and(parameter('isMonday'), parameter('isNovember'))]
```

Para establecer un valor de forma condicional:

```bicep
isMonday ? 'valueIfTrue' : 'valueIfFalse'
```

```json
[if(parameters('isMonday'), 'valueIfTrue', 'valueIfFalse')]
```

## <a name="deployment-scope"></a>Ámbito de la implementación

Para establecer el ámbito de destino de la implementación:

```bicep
targetScope = 'subscription'
```

```json
"$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#"
```

## <a name="resources"></a>Recursos

Para declarar un recurso:

```bicep
resource vm 'Microsoft.Compute/virtualMachines@2020-06-01' = {
  ...
}
```

```json
"resources": [ 
  { 
    "type": "Microsoft.Compute/virtualMachines", 
    "apiVersion": "2020-06-01", 
    ... 
  } 
]
```

Para implementar un recurso de forma condicional:

```bicep
resource vm 'Microsoft.Compute/virtualMachines@2020-06-01' = if(deployVM) {
  ...
}
```

```json
"resources": [ 
  {
    "condition": "[parameters('deployVM')]",
    "type": "Microsoft.Compute/virtualMachines", 
    "apiVersion": "2020-06-01", 
    ... 
  } 
]
```

Para establecer una propiedad de un recurso:

```bicep
sku: '2016-Datacenter'
```

```json
"sku": "2016-Datacenter",
```

Obtener el identificador de recurso en la plantilla:

```bicep
nic1.id
```

```json
[resourceId('Microsoft.Network/networkInterfaces', variables('nic1Name'))]
```

## <a name="loops"></a>Bucles

Para recorrer en iteración los elementos de una matriz o un recuento:

```bicep
[for storageName in storageAccounts: {
  ...
}]
```

```json
"copy": {
  "name": "storagecopy",
  "count": "[length(parameters('storageAccounts'))]"
},
...
```

## <a name="resource-dependencies"></a>Dependencias de recursos

Para establecer la dependencia entre los recursos:

En el caso de Bicep, puede utilizar la detección automática de las dependencias o establecerlas manualmente.

```bicep
dependsOn: [ stg ]
```

```json
"dependsOn": ["[resourceId('Microsoft.Storage/storageAccounts', 'parameters('storageAccountName'))]"]
```

## <a name="reference-resources"></a>Recursos de referencia

Para obtener una propiedad de un recurso de la plantilla:

```bicep
diagsAccount.properties.primaryEndpoints.blob
```

```json
[reference(resourceId('Microsoft.Storage/storageAccounts', variables('diagStorageAccountName'))).primaryEndpoints.blob]
```

Para obtener una propiedad de un recurso existente que no está implementado en la plantilla:

```bicep
resource stg 'Microsoft.Storage/storageAccounts@2019-06-01' existing = {
  name: storageAccountName
}

// use later in template as often as needed
stg.properties.primaryEndpoints.blob
```

```json
// required every time the property is needed
"[reference(resourceId('Microsoft.Storage/storageAccounts/', parameters('storageAccountName')), '2019-06-01').primaryEndpoints.blob]"
```

## <a name="outputs"></a>Salidas

Para enviar la salida de una propiedad de un recurso de la plantilla:

```bicep
output hostname string = publicIP.properties.dnsSettings.fqdn
```

```json
"outputs": {
  "hostname": {
    "type": "string",
    "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPAddressName'))).dnsSettings.fqdn]"
  },
}
```

## <a name="code-reuse"></a>Reutilización del código

Para dividir una solución en varios archivos:

* En el caso de Bicep, use [módulos](bicep-tutorial-add-modules.md).
* En el caso de JSON, use [plantillas vinculadas](linked-templates.md).

## <a name="recommendations"></a>Recomendaciones

* Cuando sea posible, evite usar las funciones [reference](template-functions-resource.md#reference) y [resourceId](template-functions-resource.md#resourceid) en el archivo Bicep. Cuando haga referencia a un recurso en la misma implementación de Bicep, use el id. de recursos en su lugar. Por ejemplo, si ha implementado un recurso en el archivo Bicep con `stg` como id. de recurso, use una sintaxis como `stg.id` o `stg.properties.primaryEndpoints.blob` para obtener los valores de las propiedades. Al usar el id. de recursos, se crea una dependencia implícita entre los recursos. No es necesario establecer explícitamente la dependencia con la propiedad dependsOn.
* Si el recurso no está implementado en el archivo de Bicep, todavía puede obtener una referencia simbólica al recurso mediante la palabra clave **existing**.
* Haga un uso coherente de mayúsculas y minúsculas para los identificadores. Si no está seguro de qué tipo de uso de mayúsculas aplicar, pruebe con la combinación de mayúsculas y minúsculas. Por ejemplo, `param myCamelCasedParameter string`.
* Agregue una descripción a un parámetro solo cuando la descripción proporcione información esencial a los usuarios. Puede usar comentarios de tipo `//` para incluir información.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información sobre Bicep, consulte el [Tutorial de Bicep](./bicep-tutorial-create-first-bicep.md).
* Para más información sobre cómo convertir plantillas de un lenguajes a otro, consulte [Conversión de plantillas de ARM entre JSON y Bicep](bicep-decompile.md).
