---
title: Tipos de datos en plantillas
description: Describe los tipos de datos que están disponibles en las plantillas de Azure Resource Manager.
ms.topic: conceptual
ms.author: tomfitz
author: tfitzmac
ms.date: 03/04/2021
ms.openlocfilehash: 4d6c8306b3dbdfe895055dc008d81cc0d85d8d6c
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107538057"
---
# <a name="data-types-in-arm-templates"></a>Tipos de datos en plantillas de ARM

En este artículo se describen los tipos de datos admitidos en las plantillas de Azure Resource Manager (plantillas de ARM). Trata los tipos de datos JSON y Bicep.

## <a name="supported-types"></a>Tipos admitidos

Dentro de una plantilla de Resource Manager, puede usar estos tipos de datos:

* array
* bool
* int
* object
* secureObject: indicado por el modificador de Bicep.
* secureString: indicado por el modificador de Bicep.
* string

## <a name="arrays"></a>Matrices

Las matrices comienzan con un corchete de apertura (`[`) y terminan con un corchete de cierre (`]`).

En JSON, una matriz se puede declarar en una sola línea o en varias. Cada elemento está separado por una coma.

En Bicep, una matriz se debe declarar en varias líneas. No use comas entre valores.

# <a name="json"></a>[JSON](#tab/json)

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

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param exampleArray array = [
  1
  2
  3
]
```

---

Los elementos de una matriz pueden ser del mismo tipo o de tipos diferentes.

# <a name="json"></a>[JSON](#tab/json)

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

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
var mixedArray = [
  resourceGroup().name
  1
  true
  'example string'
]
```

---

## <a name="booleans"></a>Valores booleanos

Al especificar valores booleanos, use `true` o `false`. No incluya el valor entre comillas.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "exampleBool": {
    "type": "bool",
    "defaultValue": true
  }
},
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param exampleBool bool = true
```

---

## <a name="integers"></a>Enteros

Al especificar valores enteros, no use comillas.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "exampleInt": {
    "type": "int",
    "defaultValue": 1
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param exampleInt int = 1
```

---

En el caso de los enteros pasados como parámetros en línea, el intervalo de valores puede estar limitado por el SDK o la herramienta de línea de comandos que use para la implementación. Por ejemplo, al usar PowerShell para implementar una plantilla, los tipos de enteros pueden oscilar entre -2 147 483 648 y 2 147 483 647. Para evitar esta limitación, especifique valores enteros grandes en un [archivo de parámetros](parameter-files.md). Los tipos de recursos aplican sus propios límites para las propiedades de enteros.

## <a name="objects"></a>Objetos

Los objetos comienzan con una llave de apertura (`{`) y terminan con una llave de cierre (`}`). Cada propiedad de un objeto consta de una clave y un valor. La clave y el valor están separados por dos puntos (`:`).

# <a name="json"></a>[JSON](#tab/json)

En JSON, la clave va entre comillas dobles. Cada propiedad está separada por una coma.

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

# <a name="bicep"></a>[Bicep](#tab/bicep)

En Bicep, la clave no está entre comillas. No use comas entre las propiedades.

```bicep
param exampleObject object = {
  name: 'test name'
  id: '123-abc'
  isCurrent: true
  tier: 1
}
```

Los descriptores de acceso de propiedad se usan para acceder a las propiedades de un objeto. Se construyen mediante el operador `.`. Por ejemplo:

```bicep
var x = {
  y: {
    z: 'Hello`
    a: true
  }
  q: 42
}
```

Dada la declaración anterior, la expresión x.y.z se evalúa como la cadena literal "Hello". De igual modo, la expresión x.q se evalúa como el entero literal 42.

Los descriptores de acceso de propiedad se pueden usar con cualquier objeto. Esto incluye a los parámetros y variables de los tipos de objeto y los literales de objeto. Usar un descriptor de acceso de propiedad en una expresión que no sea de tipo objeto es un error.

---

## <a name="strings"></a>Cadenas

En JSON, las cadenas se marcan con comillas dobles. En JSON, las cadenas se marcan con comillas simples.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "exampleString": {
    "type": "string",
    "defaultValue": "test value"
  }
},
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param exampleString string = 'test value'
```
---

## <a name="secure-strings-and-objects"></a>Protección de cadenas y objetos

La cadena segura usa el mismo formato que la cadena y el objeto seguro usa el mismo formato que el objeto. Cuando se establece un parámetro en una cadena segura o un objeto seguro, el valor del parámetro no se guarda en el historial de implementaciones y no se registra. Sin embargo, si establece ese valor seguro en una propiedad que no espera un valor seguro, el valor no está protegido. Por ejemplo, si establece una cadena segura en una etiqueta, ese valor se almacena como texto sin formato. Use cadenas seguras para contraseñas y secretos.

Con Bicep, el modificador `@secure()` se agrega a una cadena u objeto.

En el ejemplo siguiente se muestran dos parámetros seguros:

# <a name="json"></a>[JSON](#tab/json)

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

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
@secure()
param password string

@secure()
param configValues object
```

---

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre la sintaxis de las plantillas, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de ARM](template-syntax.md).
