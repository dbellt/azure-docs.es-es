---
title: Variables en plantillas
description: Se describe cómo definir variables en una plantilla de Azure Resource Manager (plantilla de ARM) y un archivo de Bicep.
ms.topic: conceptual
ms.date: 02/19/2021
ms.openlocfilehash: 3ab14c9acfcc2d6c9edd23fb3bc4d876cd5ac756
ms.sourcegitcommit: 5c136a01bddfccb2cc9f7e7e7741e2cf2651ddbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111353608"
---
# <a name="variables-in-arm-templates"></a>Variables en plantillas de ARM

En este artículo se describe cómo definir y usar variables en una plantilla de Azure Resource Manager (plantilla de ARM) o archivo de Bicep. Las variables se utilizan para simplificar la plantilla. En lugar de repetir expresiones complicadas en toda la plantilla, defina una variable que contenga la expresión complicada. A continuación, use esa variable en toda la plantilla tanto como sea necesario.

Resource Manager resuelve las variables antes de iniciar las operaciones de implementación. Siempre que la variable se use en la plantilla, Resource Manager la reemplaza por el valor resuelto.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="define-variable"></a>Definición de la variable

Al definir una variable, no se especifica un [tipo de datos](data-types.md) para la variable. En su lugar, se especifica un valor o una expresión de plantilla. El tipo de variable se infiere a partir del valor resuelto. En el siguiente ejemplo se establece una variable en una cadena.

# <a name="json"></a>[JSON](#tab/json)

```json
"variables": {
  "stringVar": "example value"
},
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
var stringVar = 'example value'
```

---

Puede usar el valor de un parámetro u otra variable al construir la variable.

# <a name="json"></a>[JSON](#tab/json)

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

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param inputValue string = 'deployment parameter'

var stringVar = 'myVariable'
var concatToVar =  '${stringVar}-addtovar'
var concatToParam = '${inputValue}-addtoparam'
```

---

Puede usar las [funciones de plantilla](template-functions.md) para construir el valor de la variable.

En el ejemplo siguiente se crea un valor de cadena para un nombre de cuenta de almacenamiento. Usa varias funciones de plantilla para obtener un valor de parámetro y lo concatena a una cadena única.

# <a name="json"></a>[JSON](#tab/json)

```json
"variables": {
  "storageName": "[concat(toLower(parameters('storageNamePrefix')), uniqueString(resourceGroup().id))]"
},
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
var storageName = '${toLower(storageNamePrefix)}${uniqueString(resourceGroup().id)}'
```

---

En las plantillas JSON, puede usar la función [reference](template-functions-resource.md#reference) o cualquiera de las funciones [list](template-functions-resource.md#list) en la declaración de variables. Estas funciones obtienen el estado de tiempo de ejecución de un recurso y no se pueden ejecutar antes de la implementación cuando se resuelven variables.

En los archivos de Bicep, las funciones reference y list son válidas cuando se declara una variable.

## <a name="use-variable"></a>Uso de la variable

En el ejemplo siguiente se muestra cómo usar la variable para una propiedad de recurso.

# <a name="json"></a>[JSON](#tab/json)

En una plantilla JSON, debe hacer referencia al valor de la variable mediante la función [variables](template-functions-deployment.md#variables).

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

# <a name="bicep"></a>[Bicep](#tab/bicep)

En un archivo de Bicep, se hace referencia al valor de la variable al proporcionar el nombre de la variable.

```bicep
var storageName = '${toLower(storageNamePrefix)}${uniqueString(resourceGroup().id)}'

resource demoAccount 'Microsoft.Storage/storageAccounts@2019-06-01' = {
  name: storageName
```

---

## <a name="example-template"></a>Plantilla de ejemplo

La plantilla siguiente no implementa ningún recurso. Muestra algunas maneras de declarar variables de distintos tipos.

# <a name="json"></a>[JSON](#tab/json)

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/variables.json":::

# <a name="bicep"></a>[Bicep](#tab/bicep)

Actualmente, Bicep no admite bucles.

:::code language="bicep" source="~/resourcemanager-templates/azure-resource-manager/variables.bicep":::

---

## <a name="configuration-variables"></a>Variables de configuración

Puede definir variables que contengan valores relacionados para configurar un entorno. La variable se define como un objeto con los valores. En el ejemplo siguiente se muestra un objeto que contiene valores para dos entornos: **test** y **prod**. Pase uno de estos valores durante la implementación.

# <a name="json"></a>[JSON](#tab/json)

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/variablesconfigurations.json":::

# <a name="bicep"></a>[Bicep](#tab/bicep)

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/variablesconfigurations.bicep":::

---

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre las propiedades disponibles para variables, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](template-syntax.md).
* Para obtener recomendaciones sobre cómo crear variables, consulte [Procedimientos recomendados: variables](template-best-practices.md#variables).
* Para ver una plantilla de ejemplo que asigna reglas de seguridad a un grupo de seguridad de red, consulte las [reglas de seguridad de red](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/multiplesecurityrules.json) y el [archivo de parámetros](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/multiplesecurityrules.parameters.json).
