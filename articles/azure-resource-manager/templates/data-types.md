---
title: Tipos de datos en plantillas
description: Describe los tipos de datos que están disponibles en las plantillas de Azure Resource Manager.
ms.topic: conceptual
ms.author: tomfitz
author: tfitzmac
ms.date: 05/07/2021
ms.openlocfilehash: 4905dbe184301ea9bcf86d63d527d07276a95c9e
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111957932"
---
# <a name="data-types-in-arm-templates"></a>Tipos de datos en plantillas de ARM

En este artículo se describen los tipos de datos admitidos en las plantillas de Azure Resource Manager (plantillas de ARM).

## <a name="supported-types"></a>Tipos admitidos

Dentro de una plantilla de Resource Manager, puede usar estos tipos de datos:

* array
* bool
* int
* object
* secureObject
* secureString
* string

## <a name="arrays"></a>Matrices

Las matrices comienzan con un corchete de apertura (`[`) y terminan con un corchete de cierre (`]`). Una matriz se puede declarar en una sola línea o en varias. Cada elemento está separado por una coma.

```json
"parameters": {
  "exampleArray": {
    "type": "array",
    "defaultValue": [
      1,
      2,
      3
    ]
  }
},
```

Los elementos de una matriz pueden ser del mismo tipo o de tipos diferentes.

```json
"variables": {
  "mixedArray": [
    "[resourceGroup().name]",
    1,
    true,
    "example string"
  ]
}
```

## <a name="booleans"></a>Valores booleanos

Al especificar valores booleanos, use `true` o `false`. No incluya el valor entre comillas.

```json
"parameters": {
  "exampleBool": {
    "type": "bool",
    "defaultValue": true
  }
},
```

## <a name="integers"></a>Enteros

Al especificar valores enteros, no use comillas.

```json
"parameters": {
  "exampleInt": {
    "type": "int",
    "defaultValue": 1
  }
}
```

En el caso de los enteros pasados como parámetros en línea, el intervalo de valores puede estar limitado por el SDK o la herramienta de línea de comandos que use para la implementación. Por ejemplo, al usar PowerShell para implementar una plantilla, los tipos de enteros pueden oscilar entre -2 147 483 648 y 2 147 483 647. Para evitar esta limitación, especifique valores enteros grandes en un [archivo de parámetros](parameter-files.md). Los tipos de recursos aplican sus propios límites para las propiedades de enteros.

## <a name="objects"></a>Objetos

Los objetos comienzan con una llave de apertura (`{`) y terminan con una llave de cierre (`}`). Cada propiedad de un objeto consta de `key` y `value`. `key` y `value` se incluyen entre comillas dobles y se separan con dos puntos (`:`). Cada propiedad está separada por una coma.

```json
"parameters": {
  "exampleObject": {
    "type": "object",
    "defaultValue": {
      "name": "test name",
      "id": "123-abc",
      "isCurrent": true,
      "tier": 1
    }
  }
}
```

## <a name="strings"></a>Cadenas

Las cadenas se marcan con comillas dobles.

```json
"parameters": {
  "exampleString": {
    "type": "string",
    "defaultValue": "test value"
  }
},
```

## <a name="secure-strings-and-objects"></a>Protección de cadenas y objetos

La cadena segura usa el mismo formato que la cadena y el objeto seguro usa el mismo formato que el objeto. Cuando se establece un parámetro en una cadena segura o un objeto seguro, el valor del parámetro no se guarda en el historial de implementaciones y no se registra. Sin embargo, si establece ese valor seguro en una propiedad que no espera un valor seguro, el valor no está protegido. Por ejemplo, si establece una cadena segura en una etiqueta, ese valor se almacena como texto sin formato. Use cadenas seguras para contraseñas y secretos.

En el ejemplo siguiente se muestran dos parámetros seguros.

```json
"parameters": {
  "password": {
    "type": "secureString"
  },
  "configValues": {
    "type": "secureObject"
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre la sintaxis de las plantillas, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de ARM](./syntax.md).