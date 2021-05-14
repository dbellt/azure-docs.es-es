---
title: Sintaxis y expresiones de plantillas
description: En este artículo se describe la sintaxis JSON declarativa de las plantillas de Azure Resource Manager (plantillas de ARM).
ms.topic: conceptual
ms.date: 03/17/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 0c010d71217dcae23348098355bf1752ffc43337
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108314902"
---
# <a name="syntax-and-expressions-in-arm-templates"></a>Sintaxis y expresiones en las plantillas de ARM

La sintaxis básica de la plantilla de Azure Resource Manager (plantilla de ARM) es la notación de objetos JavaScript (JSON). Sin embargo, puede utilizar expresiones para ampliar los valores JSON disponibles en la plantilla.  Las expresiones empiezan y terminan con corchetes: `[` y `]`, respectivamente. El valor de la expresión se evalúa cuando se implementa la plantilla. Una expresión puede devolver una cadena, un entero, un booleano, una matriz o un objeto.

Las expresiones de plantilla no pueden superar los 24 576 caracteres.

## <a name="use-functions"></a>Uso de las funciones

Azure Resource Manager cuenta con [funciones](template-functions.md) que se pueden usar en una plantilla. En el ejemplo siguiente, se muestra una expresión que utiliza una función en el valor predeterminado de un parámetro:

```json
"parameters": {
  "location": {
    "type": "string",
    "defaultValue": "[resourceGroup().location]"
  }
},
```

En de la expresión, la sintaxis `resourceGroup()` llama a una de las funciones que Resource Manager proporciona para utilizarla dentro de una plantilla. En este caso, se trata de la función [resourceGroup](template-functions-resource.md#resourcegroup). Al igual que en JavaScript, las llamadas de función tienen el formato `functionName(arg1,arg2,arg3)`. La sintaxis `.location` recupera una propiedad del objeto devuelto por esta función.

Las funciones de plantilla y sus parámetros no distinguen mayúsculas de minúsculas. Por ejemplo, Resource Manager resuelve `variables('var1')` y `VARIABLES('VAR1')` de la misma manera. Cuando se evalúa, a menos que la función modifique expresamente las mayúsculas y minúsculas (como `toUpper` o `toLower`), la función conserva lo que existe. Es posible que determinados tipos de recursos tengan requisitos de mayúsculas independientemente de cómo se evalúen las funciones.

Para pasar un valor de cadena como parámetro a una función, use comillas simples.

```json
"name": "[concat('storage', uniqueString(resourceGroup().id))]"
```

La mayoría de las funciones operan del mismo modo cuando se implementan en un grupo de recursos, en una suscripción, en un grupo de administración o en un inquilino. Las siguientes funciones tienen restricciones en función del ámbito:

* [resourceGroup](template-functions-resource.md#resourcegroup): solo puede utilizarse cuando la implementación se realiza en un grupo de recursos.
* [resourceId](template-functions-resource.md#resourceid): puede utilizarse con cualquier ámbito, pero los valores correctos cambiarán en función de este.
* [subscription](template-functions-resource.md#subscription): solo puede utilizarse cuando la implementación se realiza en un grupo de recursos o una suscripción.

## <a name="escape-characters"></a>Carácter de escape

Para que una cadena literal empiece con un corchete izquierdo `[` y termine con un corchete derecho `]`, pero no se interprete como una expresión, agregue otro corchete para que la cadena comience con `[[`. Por ejemplo, la variable:

```json
"demoVar1": "[[test value]"
```

Se resuelve como `[test value]`.

Sin embargo, si la cadena literal no termina con un corchete de cierre, no utilice un carácter de escape en el primer corchete. Por ejemplo, la variable:

```json
"demoVar2": "[test] value"
```

Se resuelve como `[test] value`.

Para utilizar un carácter de escape en las comillas dobles en una expresión, como, por ejemplo, al agregar un objeto JSON a la plantilla, use la barra diagonal inversa.

```json
"tags": {
    "CostCenter": "{\"Dept\":\"Finance\",\"Environment\":\"Production\"}"
},
```

Al pasar valores de parámetro, el uso de caracteres de escape depende de si se especifica el valor de parámetro. Si especifica un valor predeterminado en la plantilla, necesita un corchete de apertura adicional.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "demoParam1":{
            "type": "string",
            "defaultValue": "[[test value]"
        }
    },
    "resources": [],
    "outputs": {
        "exampleOutput": {
            "type": "string",
            "value": "[parameters('demoParam1')]"
        }
    }
}
```

Si usa el valor predeterminado, la plantilla devuelve `[test value]`.

Sin embargo, si se pasa un valor de parámetro mediante la línea de comandos, los caracteres se interpretan literalmente. Al implementar la plantilla anterior con:

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName demoGroup -TemplateFile azuredeploy.json -demoParam1 "[[test value]"
```

Devuelve `[[test value]`. En su lugar, use:

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName demoGroup -TemplateFile azuredeploy.json -demoParam1 "[test value]"
```

Se aplica el mismo formato al pasar valores de un archivo de parámetros. Los caracteres se interpretan literalmente. Cuando se usa con la plantilla anterior, el siguiente archivo de parámetros devuelve `[test value]`:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "demoParam1": {
            "value": "[test value]"
        }
   }
}
```

## <a name="null-values"></a>Valores NULL

Para establecer una propiedad en NULL, puede usar `null` o `[json('null')]`. La [función json](template-functions-object.md#json) devuelve un objeto vacío cuando se proporciona `null` como parámetro. En ambos casos, las plantillas de Resource Manager lo tratan como si la propiedad no estuviera presente.

```json
"stringValue": null,
"objectValue": "[json('null')]"
```

## <a name="next-steps"></a>Pasos siguientes

* Para la lista completa de las funciones de plantilla disponibles, consulte [Funciones de plantilla de ARM](template-functions.md).
* Para más información sobre los archivos de plantilla, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de ARM](template-syntax.md).
