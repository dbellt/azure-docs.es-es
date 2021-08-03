---
title: Variables en plantillas
description: Se describe cómo definir variables en una plantilla de Azure Resource Manager (plantilla de ARM).
ms.topic: conceptual
ms.date: 05/14/2021
ms.openlocfilehash: 02b80e4ee9cc6392353d6229d6437feb5a1a41ec
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111959319"
---
# <a name="variables-in-arm-templates"></a>Variables en plantillas de ARM

En este artículo se describe cómo definir y usar variables en una plantilla de Azure Resource Manager (plantilla de ARM). Las variables se utilizan para simplificar la plantilla. En lugar de repetir expresiones complicadas en toda la plantilla, defina una variable que contenga la expresión complicada. A continuación, use esa variable en toda la plantilla tanto como sea necesario.

Resource Manager resuelve las variables antes de iniciar las operaciones de implementación. Siempre que la variable se use en la plantilla, Resource Manager la reemplaza por el valor resuelto.

## <a name="define-variable"></a>Definición de la variable

Al definir una variable, no se especifica un [tipo de datos](data-types.md) para la variable. En su lugar, se especifica un valor o una expresión de plantilla. El tipo de variable se infiere a partir del valor resuelto. En el siguiente ejemplo se establece una variable en una cadena.

```json
"variables": {
  "stringVar": "example value"
},
```

Para construir la variable, puede usar el valor de un parámetro u otra variable.

```json
"parameters": {
  "inputValue": {
    "defaultValue": "deployment parameter",
    "type": "string"
  }
},
"variables": {
  "stringVar": "myVariable",
  "concatToVar": "[concat(variables('stringVar'), '-addtovar') ]",
  "concatToParam": "[concat(parameters('inputValue'), '-addtoparam')]"
}
```

Puede usar las [funciones de plantilla](template-functions.md) para construir el valor de la variable.

En el ejemplo siguiente se crea un valor de cadena para un nombre de cuenta de almacenamiento. Usa varias funciones de plantilla para obtener un valor de parámetro y lo concatena a una cadena única.

```json
"variables": {
  "storageName": "[concat(toLower(parameters('storageNamePrefix')), uniqueString(resourceGroup().id))]"
},
```

No puede usar la función [reference](template-functions-resource.md#reference) ni ninguna de las funciones [list](template-functions-resource.md#list) de la declaración de variable. Estas funciones obtienen el estado de tiempo de ejecución de un recurso y no se pueden ejecutar antes de la implementación cuando se resuelven variables.

## <a name="use-variable"></a>Uso de la variable

En el ejemplo siguiente se muestra cómo usar la variable para una propiedad de recurso.

Para hacer referencia al valor de la variable, use la función [variables](template-functions-deployment.md#variables).

```json
"variables": {
  "storageName": "[concat(toLower(parameters('storageNamePrefix')), uniqueString(resourceGroup().id))]"
},
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[variables('storageName')]",
    ...
  }
]
```

## <a name="example-template"></a>Plantilla de ejemplo

La plantilla siguiente no implementa ningún recurso. Muestra algunas maneras de declarar variables de distintos tipos.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/variables.json":::

## <a name="configuration-variables"></a>Variables de configuración

Puede definir variables que contengan valores relacionados para configurar un entorno. La variable se define como un objeto con los valores. En el ejemplo siguiente se muestra un objeto que contiene valores para dos entornos: **test** y **prod**. Pase uno de estos valores durante la implementación.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/variablesconfigurations.json":::

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre las propiedades disponibles para variables, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](./syntax.md).
* Para obtener recomendaciones sobre cómo crear variables, consulte [Procedimientos recomendados: variables](./best-practices.md#variables).
* Para ver una plantilla de ejemplo que asigna reglas de seguridad a un grupo de seguridad de red, consulte las [reglas de seguridad de red](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/multiplesecurityrules.json) y el [archivo de parámetros](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/multiplesecurityrules.parameters.json).