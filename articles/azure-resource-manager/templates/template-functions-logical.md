---
title: 'Funciones de plantillas: lógicas'
description: Se describen las funciones que se pueden usar en una plantilla de Azure Resource Manager para determinar valores lógicos.
ms.topic: conceptual
ms.date: 05/13/2021
ms.openlocfilehash: c69e10b660d5b7cbf768ea31fda6678d07053224
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111959633"
---
# <a name="logical-functions-for-arm-templates"></a>Funciones lógicas para plantillas de ARM

Resource Manager ofrece varias funciones para realizar comparaciones en las plantillas de Azure Resource Manager:

* [and](#and)
* [bool](#bool)
* [false](#false)
* [if](#if)
* [not](#not)
* [or](#or)
* [true](#true)

## <a name="and"></a>y

`and(arg1, arg2, ...)`

Comprueba si todos los valores de parámetros son verdaderos.

La función `and` no se admite en Bicep; en su lugar, use el [operador &&](../bicep/operators-logical.md#and-).

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| arg1 |Sí |boolean |Primer valor cuya veracidad se comprueba. |
| arg2 |Sí |boolean |Segundo valor cuya veracidad se comprueba. |
| argumentos adicionales |No |boolean |Argumentos adicionales para comprobar si son verdaderos. |

### <a name="return-value"></a>Valor devuelto

Devuelve **True** si todos los valores son verdaderos; en caso contrario, devuelve **False**.

### <a name="examples"></a>Ejemplos

En la [plantilla de ejemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/andornot.json) siguiente se muestra cómo usar las funciones lógicas.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [],
  "outputs": {
    "andExampleOutput": {
      "type": "bool",
      "value": "[and(bool('true'), bool('false'))]"
    },
    "orExampleOutput": {
      "type": "bool",
      "value": "[or(bool('true'), bool('false'))]"
    },
    "notExampleOutput": {
      "type": "bool",
      "value": "[not(bool('true'))]"
    }
  }
}
```

El resultado del ejemplo anterior es:

| Nombre | Tipo | Valor |
| ---- | ---- | ----- |
| andExampleOutput | Bool | False |
| orExampleOutput | Bool | True |
| notExampleOutput | Bool | False |

## <a name="bool"></a>bool

`bool(arg1)`

Convierte el parámetro en un booleano.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| arg1 |Sí |cadena o entero |El valor para convertir en booleano. |

### <a name="return-value"></a>Valor devuelto

Valor booleano del valor convertido.

### <a name="remarks"></a>Observaciones

También puede usar [true()](#true) y [false()](#false) para obtener valores booleanos.

### <a name="examples"></a>Ejemplos

En la [plantilla de ejemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/bool.json) siguiente se muestra cómo usar bool con una cadena o un entero.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [],
  "outputs": {
    "trueString": {
      "type": "bool",
      "value": "[bool('true')]",
    },
    "falseString": {
      "type": "bool",
      "value": "[bool('false')]"
    },
    "trueInt": {
      "type": "bool",
      "value": "[bool(1)]"
    },
    "falseInt": {
      "type": "bool",
      "value": "[bool(0)]"
    }
  }
}
```

La salida del ejemplo anterior con el valor predeterminado es:

| Nombre | Tipo | Valor |
| ---- | ---- | ----- |
| trueString | Bool | True |
| falseString | Bool | False |
| trueInt | Bool | True |
| falseInt | Bool | False |

## <a name="false"></a>false

`false()`

Devuelve false.

La función `false` no está disponible en Bicep.  En su lugar, use la palabra clave `false`.

### <a name="parameters"></a>Parámetros

La función false no acepta ningún parámetro.

### <a name="return-value"></a>Valor devuelto

Valor booleano que siempre es false.

### <a name="example"></a>Ejemplo

El ejemplo siguiente devuelve un valor como de salida false.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [],
  "outputs": {
    "falseOutput": {
      "type": "bool",
      "value": "[false()]"
    }
  }
}
```

El resultado del ejemplo anterior es:

| Nombre | Tipo | Valor |
| ---- | ---- | ----- |
| falseOutput | Bool | False |

## <a name="if"></a>if

`if(condition, trueValue, falseValue)`

Devuelve un valor dependiendo de si una condición es verdadera o falsa.

La función `if` no se admite en Bicep. En su lugar, utilice el [operador ?:](../bicep/operators-logical.md#conditional-expression--).

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| condición |Sí |boolean |El valor que se va a comprobar si es true o false. |
| trueValue |Sí | cadena, int, objeto o matriz |Valor que se devuelve cuando la condición es verdadera. |
| falseValue |Sí | cadena, int, objeto o matriz |Valor que se devuelve cuando la condición es falsa. |

### <a name="return-value"></a>Valor devuelto

Devuelve el segundo parámetro si el primer parámetro es **True**; en caso contrario, devuelve el tercer parámetro.

### <a name="remarks"></a>Observaciones

Si la condición es **True**, solo se evalúa el valor true. Si la condición es **False**, solo se evalúa el valor false. Con la función `if`, puede incluir expresiones que solo son válidas con ciertas condiciones. Por ejemplo, puede hacer referencia a un recurso que existe bajo una condición, pero no bajo la otra. En la sección siguiente se muestra un ejemplo de las expresiones de evaluación con ciertas condiciones.

### <a name="examples"></a>Ejemplos

En la [plantilla de ejemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/if.json) siguiente se muestra cómo usar la función `if`.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
  ],
  "outputs": {
    "yesOutput": {
      "type": "string",
      "value": "[if(equals('a', 'a'), 'yes', 'no')]"
    },
    "noOutput": {
      "type": "string",
      "value": "[if(equals('a', 'b'), 'yes', 'no')]"
    },
    "objectOutput": {
      "type": "object",
      "value": "[if(equals('a', 'a'), json('{\"test\": \"value1\"}'), json('null'))]"
    }
  }
}
```

El resultado del ejemplo anterior es:

| Nombre | Tipo | Valor |
| ---- | ---- | ----- |
| yesOutput | String | sí |
| noOutput | String | no |
| objectOutput | Object | { "test": "value1" } |

En la [plantilla de ejemplo](https://github.com/krnese/AzureDeploy/blob/master/ARM/deployments/conditionWithReference.json) siguiente se muestra cómo usar esta función con expresiones que solo son válidas bajo ciertas condiciones.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string"
    },
    "location": {
      "type": "string"
    },
    "logAnalytics": {
      "type": "string",
      "defaultValue": ""
    }
  },
  "resources": [
    {
      "condition": "[not(empty(parameters('logAnalytics')))]",
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "apiVersion": "2017-03-30",
      "name": "[concat(parameters('vmName'),'/omsOnboarding')]",
      "location": "[parameters('location')]",
      "properties": {
        "publisher": "Microsoft.EnterpriseCloud.Monitoring",
        "type": "MicrosoftMonitoringAgent",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "workspaceId": "[if(not(empty(parameters('logAnalytics'))), reference(parameters('logAnalytics'), '2015-11-01-preview').customerId, json('null'))]"
        },
        "protectedSettings": {
          "workspaceKey": "[if(not(empty(parameters('logAnalytics'))), listKeys(parameters('logAnalytics'), '2015-11-01-preview').primarySharedKey, json('null'))]"
        }
      }
    }
  ],
  "outputs": {
    "mgmtStatus": {
      "type": "string",
      "value": "[if(not(empty(parameters('logAnalytics'))), 'Enabled monitoring for VM!', 'Nothing to enable')]"
    }
  }
}
```

## <a name="not"></a>not

`not(arg1)`

Convierte el valor booleano en su valor opuesto.

La función `not` no se admite en Bicep; en su lugar, use el [operador !](../bicep/operators-logical.md#not-).

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| arg1 |Sí |boolean |Valor que se va a convertir. |

### <a name="return-value"></a>Valor devuelto

Devuelve **True** si el parámetro es **False**. Devuelve **False** si el parámetro es **True**.

### <a name="examples"></a>Ejemplos

En la [plantilla de ejemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/andornot.json) siguiente se muestra cómo usar las funciones lógicas.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [],
  "outputs": {
    "andExampleOutput": {
      "type": "bool",
      "value": "[and(bool('true'), bool('false'))]",
    },
    "orExampleOutput": {
      "type": "bool",
      "value": "[or(bool('true'), bool('false'))]"
    },
    "notExampleOutput": {
      "type": "bool",
      "value": "[not(bool('true'))]"
    }
  }
}
```

El resultado del ejemplo anterior es:

| Nombre | Tipo | Valor |
| ---- | ---- | ----- |
| andExampleOutput | Bool | False |
| orExampleOutput | Bool | True |
| notExampleOutput | Bool | False |

En la [plantilla de ejemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/not-equals.json) siguiente se usa **not** con [equals](template-functions-comparison.md#equals).

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
  ],
  "outputs": {
    "checkNotEquals": {
      "type": "bool",
      "value": "[not(equals(1, 2))]"
    }
  }
}
```

El resultado del ejemplo anterior es:

| Nombre | Tipo | Valor |
| ---- | ---- | ----- |
| checkNotEquals | Bool | True |

## <a name="or"></a>or

`or(arg1, arg2, ...)`

Comprueba si algún valor de parámetro es verdadero.

La función `or` no se admite en Bicep; en su lugar, use el [operador ||](../bicep/operators-logical.md#or-).

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| arg1 |Sí |boolean |Primer valor cuya veracidad se comprueba. |
| arg2 |Sí |boolean |Segundo valor cuya veracidad se comprueba. |
| argumentos adicionales |No |boolean |Argumentos adicionales para comprobar si son verdaderos. |

### <a name="return-value"></a>Valor devuelto

Devuelve **True** si algún valor es verdadero; en caso contrario, devuelve **False**.

### <a name="examples"></a>Ejemplos

En la [plantilla de ejemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/andornot.json) siguiente se muestra cómo usar las funciones lógicas.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [],
  "outputs": {
    "andExampleOutput": {
      "value": "[and(bool('true'), bool('false'))]",
      "type": "bool"
    },
    "orExampleOutput": {
      "value": "[or(bool('true'), bool('false'))]",
      "type": "bool"
    },
    "notExampleOutput": {
      "value": "[not(bool('true'))]",
      "type": "bool"
    }
  }
}
```

El resultado del ejemplo anterior es:

| Nombre | Tipo | Valor |
| ---- | ---- | ----- |
| andExampleOutput | Bool | False |
| orExampleOutput | Bool | True |
| notExampleOutput | Bool | False |

## <a name="true"></a>true

`true()`

Devuelve true.

La función `true` no está disponible en Bicep.  En su lugar, use la palabra clave `true`.

### <a name="parameters"></a>Parámetros

La función true no acepta ningún parámetro.

### <a name="return-value"></a>Valor devuelto

Valor booleano que siempre es true.

### <a name="example"></a>Ejemplo

El ejemplo siguiente devuelve un valor como de salida true.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [],
  "outputs": {
    "trueOutput": {
      "type": "bool",
      "value": "[true()]"
    }
  }
}
```

El resultado del ejemplo anterior es:

| Nombre | Tipo | Valor |
| ---- | ---- | ----- |
| trueOutput | Bool | True |

## <a name="next-steps"></a>Pasos siguientes

* Para obtener una descripción de las secciones de una plantilla de ARM, vea [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](./syntax.md).