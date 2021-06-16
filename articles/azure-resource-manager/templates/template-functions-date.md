---
title: 'Funciones de plantillas: fecha'
description: Aquí se describen las funciones que se usan en una plantilla de Azure Resource Manager (plantilla de ARM) para trabajar con fechas.
ms.topic: conceptual
ms.date: 05/11/2021
ms.openlocfilehash: c6bf3adca5dde4947e2c22dd8468f1b045f77120
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111959702"
---
# <a name="date-functions-for-arm-templates"></a>Funciones de cadena para plantillas de Resource Manager

Resource Manager ofrece las siguientes funciones para trabajar con fechas en la plantilla de Azure Resource Manager (plantilla de ARM):

* [dateTimeAdd](#datetimeadd)
* [utcNow](#utcnow)

## <a name="datetimeadd"></a>dateTimeAdd

`dateTimeAdd(base, duration, [format])`

Agrega una duración de tiempo a un valor base. Se esperaba el formato ISO 8601.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| base | Sí | string | Valor datetime inicial para la suma. Utilice el [formato de marca de tiempo ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). |
| duration | Sí | string | Valor de hora que se suma a la base. Puede ser un valor negativo. Utilice el [formato de duración ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Durations). |
| format | No | string | El formato de salida del resultado de fecha y hora. Si no se indica, se usa el formato del valor base. Use [cadenas de formato estándar](/dotnet/standard/base-types/standard-date-and-time-format-strings) o [cadenas de formato personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings). |

### <a name="return-value"></a>Valor devuelto

Valor datetime resultado de sumar el valor de duración al valor base.

### <a name="examples"></a>Ejemplos

En la plantilla de ejemplo siguiente se muestran distintas formas de sumar valores de hora.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "baseTime": {
      "type": "string",
      "defaultValue": "[utcNow('u')]"
    }
  },
  "variables": {
    "add3Years": "[dateTimeAdd(parameters('baseTime'), 'P3Y')]",
    "subtract9Days": "[dateTimeAdd(parameters('baseTime'), '-P9D')]",
    "add1Hour": "[dateTimeAdd(parameters('baseTime'), 'PT1H')]"
  },
  "resources": [],
  "outputs": {
    "add3YearsOutput": {
      "value": "[variables('add3Years')]",
      "type": "string"
    },
    "subtract9DaysOutput": {
      "value": "[variables('subtract9Days')]",
      "type": "string"
    },
    "add1HourOutput": {
      "value": "[variables('add1Hour')]",
      "type": "string"
    },
  }
}
```

Cuando la plantilla anterior se implementa con una hora base de `2020-04-07 14:53:14Z`, la salida es:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| add3YearsOutput | String | 4/7/2023 2:53:14 PM |
| subtract9DaysOutput | String | 3/29/2020 2:53:14 PM |
| add1HourOutput | String | 4/7/2020 3:53:14 PM |

En la plantilla de ejemplo siguiente se muestra cómo establecer la hora de inicio de una programación de Automation.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "omsAutomationAccountName": {
      "type": "string",
      "defaultValue": "demoAutomation",
      "metadata": {
        "description": "Use an existing Automation account."
      }
    },
    "scheduleName": {
      "type": "string",
      "defaultValue": "demoSchedule1",
      "metadata": {
        "description": "Name of the new schedule."
      }
    },
    "baseTime": {
      "type": "string",
      "defaultValue": "[utcNow('u')]",
      "metadata": {
        "description": "Schedule will start one hour from this time."
      }
    }
  },
  "variables": {
    "startTime": "[dateTimeAdd(parameters('baseTime'), 'PT1H')]"
  },
  "resources": [
    ...
    {
      "type": "Microsoft.Automation/automationAccounts/schedules",
      "apiVersion": "2015-10-31",
      "name": "[concat(parameters('omsAutomationAccountName'), '/', parameters('scheduleName'))]",

      "properties": {
        "description": "Demo Scheduler",
        "startTime": "[variables('startTime')]",
        "interval": 1,
        "frequency": "Hour"
      }
    }
  ],
  "outputs": {
  }
}
```

## <a name="utcnow"></a>utcNow

`utcNow(format)`

Devuelve el valor de fecha y hora (UTC) actual en el formato especificado. Si no se proporciona el formato, se usa ISO 8601 (`yyyyMMddTHHmmssZ`). **Esta función solo puede utilizarse en el valor predeterminado para un parámetro.**

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| format |No |string |El valor codificado por el identificador URI para convertir en una cadena. Use [cadenas de formato estándar](/dotnet/standard/base-types/standard-date-and-time-format-strings) o [cadenas de formato personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings). |

### <a name="remarks"></a>Observaciones

Solo puede usar esta función dentro de una expresión para el valor predeterminado de un parámetro. El uso de esta función en cualquier otro lugar de una plantilla genera un error. La función no se permite en otras partes de la plantilla porque devuelve un valor diferente cada vez que se le llama. La implementación de la misma plantilla con los mismos parámetros no produciría de forma confiable los mismos resultados.

Si usa la [opción de revertir en caso de error](rollback-on-error.md) a una implementación correcta anterior y dicha implementación incluye un parámetro que usa utcNow, el parámetro no se vuelve a evaluar. En su lugar, el valor del parámetro de la implementación anterior se reutiliza automáticamente en la implementación de reversión.

Tenga cuidado al volver a implementar una plantilla que se base en la función utcNow para un valor predeterminado. Si vuelve a implementar y no proporciona un valor para el parámetro, la función se vuelve a evaluar. Si desea actualizar un recurso existente en lugar de crear uno nuevo, pase el valor de parámetro de la implementación anterior.

### <a name="return-value"></a>Valor devuelto

El valor de fecha y hora UTC actual.

### <a name="examples"></a>Ejemplos

La plantilla de ejemplo siguiente muestra formatos diferentes para el valor de fecha y hora.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "utcValue": {
      "type": "string",
      "defaultValue": "[utcNow()]"
    },
    "utcShortValue": {
      "type": "string",
      "defaultValue": "[utcNow('d')]"
    },
    "utcCustomValue": {
      "type": "string",
      "defaultValue": "[utcNow('M d')]"
    }
  },
  "resources": [
  ],
  "outputs": {
    "utcOutput": {
      "type": "string",
      "value": "[parameters('utcValue')]"
    },
    "utcShortOutput": {
      "type": "string",
      "value": "[parameters('utcShortValue')]"
    },
    "utcCustomOutput": {
      "type": "string",
      "value": "[parameters('utcCustomValue')]"
    }
  }
}
```

El resultado del ejemplo anterior varía para cada implementación, pero será similar a:

| Nombre | Tipo | Value |
| ---- | ---- | ----- |
| utcOutput | string | 20190305T175318Z |
| utcShortOutput | string | 05/03/2019 |
| utcCustomOutput | string | 3 5 |

El ejemplo siguiente muestra cómo usar un valor de la función cuando se establece un valor de etiqueta.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "utcShort": {
      "type": "string",
      "defaultValue": "[utcNow('d')]"
    },
    "rgName": {
      "type": "string"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Resources/resourceGroups",
      "apiVersion": "2020-10-01",
      "name": "[parameters('rgName')]",
      "location": "westeurope",
      "tags": {
        "createdDate": "[parameters('utcShort')]"
      },
      "properties": {}
    }
  ],
  "outputs": {
    "utcShortOutput": {
      "type": "string",
      "value": "[parameters('utcShort')]"
    }
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

* Para obtener una descripción de las secciones de una plantilla de ARM, vea [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](./syntax.md).