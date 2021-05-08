---
title: Recursos secundarios en plantillas
description: Describe cómo establecer el nombre y el tipo de los recursos secundarios en una plantilla de Azure Resource Manager.
ms.topic: conceptual
ms.date: 04/23/2021
ms.openlocfilehash: 1928a94fbfefc694091a3f09ea577e8250540bf4
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107949531"
---
# <a name="set-name-and-type-for-child-resources"></a>Establecimiento del nombre y el tipo de los recursos secundarios

Los recursos secundarios son recursos que solo existen en el contexto de otro recurso. Por ejemplo, una [extensión de máquina virtual](/azure/templates/microsoft.compute/virtualmachines/extensions) no puede existir sin una [máquina virtual](/azure/templates/microsoft.compute/virtualmachines). El recurso de extensión es un elemento secundario de la máquina virtual.

Cada recurso primario solo acepta determinados tipos de recursos como recursos secundarios. El tipo de recurso para el recurso secundario incluye el tipo de recurso para el recurso primario. Por ejemplo, `Microsoft.Web/sites/config` y `Microsoft.Web/sites/extensions` son recursos secundarios de `Microsoft.Web/sites`. Los tipos de recursos que se aceptan se especifican en el [esquema de plantilla](https://github.com/Azure/azure-resource-manager-schemas) del recurso principal.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

En una plantilla de Azure Resource Manager (plantilla de ARM), puede especificar el recurso secundario dentro del recurso primario o fuera de él. Los valores proporcionados para el nombre y el tipo de recurso varían en función de si el recurso secundario se ha definido dentro del recurso primario o fuera de él.

## <a name="within-parent-resource"></a>Dentro del recurso primario

En el ejemplo siguiente se muestra el recurso secundario incluido en la propiedad resources del recurso primario.

# <a name="json"></a>[JSON](#tab/json)

```json
"resources": [
  {
    <parent-resource>
    "resources": [
      <child-resource>
    ]
  }
]
```

Los recursos secundarios solo se pueden definir en cinco niveles de profundidad.

Cuando el recurso secundario se define dentro del tipo de recurso primario, los valores type y name se indican como un solo segmento sin barras diagonales.

```json
"type": "{child-resource-type}",
"name": "{child-resource-name}",
```

En el ejemplo siguiente se muestra una red virtual con una subred. Observe que la subred se incluye en la matriz de recursos de la red virtual. El nombre se establece en **Subnet1** y el tipo se establece en **subnets**. El recurso secundario se marca como dependiente del recurso primario porque este debe existir antes de que se pueda implementar el recurso secundario.

```json
"resources": [
  {
    "type": "Microsoft.Network/virtualNetworks",
    "apiVersion": "2018-10-01",
    "name": "VNet1",
    "location": "[parameters('location')]",
    "properties": {
      "addressSpace": {
        "addressPrefixes": [
          "10.0.0.0/16"
        ]
      }
    },
    "resources": [
      {
        "type": "subnets",
        "apiVersion": "2018-10-01",
        "name": "Subnet1",
        "dependsOn": [
          "VNet1"
        ],
        "properties": {
          "addressPrefix": "10.0.0.0/24"
        }
      }
    ]
  }
]
```

El tipo de recurso completo sigue siendo `Microsoft.Network/virtualNetworks/subnets`. No se proporciona `Microsoft.Network/virtualNetworks/` porque se supone que viene del tipo de recurso primario.

El nombre del recurso secundario se establece en **Subnet1**, pero el nombre completo incluye el primario. No se proporciona **VNet1** porque se asume a partir del recurso primario.

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
resource <parent-resource-symbolic-name> '<resource-type>@<api-version>' = {
  <parent-resource-properties>

  resource <child-resource-symbolic-name> '<child-resource-type>' = {
    <child-resource-properties>
  }
}
```

Debe aparecer una declaración de recursos anidada en el nivel superior de sintaxis del recurso primario. Las declaraciones pueden anidarse a una profundidad arbitraria, siempre y cuando cada nivel sea un tipo secundario de su recurso primario.

Cuando el recurso secundario se define dentro del tipo de recurso primario, los valores type y name se indican como un solo segmento sin barras diagonales. En el ejemplo siguiente se muestra una red virtual con una subred. Observe que la subred se incluye en la matriz de recursos de la red virtual. El nombre se establece en **Subnet1** y el tipo se establece en **subnets**.

```bicep
param location string

resource VNet1 'Microsoft.Network/virtualNetworks@2018-10-01' = {
  name: 'VNet1'
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: [
        '10.0.0.0/16'
      ]
    }
  }

  resource VNet1_Subnet1 'subnets' = {
    name: 'Subnet1'
    properties: {
      addressPrefix: '10.0.0.0/24'
    }
  }
}
```

El tipo de recurso completo sigue siendo `Microsoft.Network/virtualNetworks/subnets`. No se proporciona `Microsoft.Network/virtualNetworks/` porque se supone que viene del tipo y versión del recurso primario. El recurso anidado puede declarar opcionalmente una versión de API mediante la sintaxis `<segment>@<version>`. Si el recurso anidado omite la versión de API, se usa la versión de API del recurso primario. Si el recurso anidado especifica una versión de API, se usa la versión de API especificada.

El nombre del recurso secundario se establece en **Subnet1**, pero el nombre completo incluye el primario. No se proporciona VNet1 porque se asume a partir del recurso primario.

Para acceder al nombre simbólico del recurso secundario, debe usar el operador `::`. Por ejemplo, para generar una propiedad desde un recurso secundario:

```bicep
output childAddressPrefix string = VNet1::VNet1_Subnet1.properties.addressPrefix
```

Un recurso anidado puede acceder a las propiedades de su recurso primario. Otros recursos declarados dentro del cuerpo del mismo recurso primario pueden hacerse referencia entre sí y se aplican las reglas típicas sobre las dependencias cíclicas. Es posible que un recurso primario no tenga acceso a las propiedades de los recursos que contiene, lo que provocaría una dependencia cíclica.

---

## <a name="outside-parent-resource"></a>Fuera del recurso primario

En el ejemplo siguiente se muestra el recurso secundario fuera del recurso primario. Este enfoque puede usarse si el recurso primario no está implementado en la misma plantilla o si quiere usar [copy](copy-resources.md) para crear más de un recurso secundario.

# <a name="json"></a>[JSON](#tab/json)

```json
"resources": [
  {
    <parent-resource>
  },
  {
    <child-resource>
  }
]
```

Cuando el recurso secundario se define fuera del recurso primario, los valores type y name se indican con barras diagonales para incluir el tipo y el nombre primarios.

```json
"type": "{resource-provider-namespace}/{parent-resource-type}/{child-resource-type}",
"name": "{parent-resource-name}/{child-resource-name}",
```

En el ejemplo siguiente se muestra una red virtual y una subred que se definen en el nivel raíz. Observe que la subred no está incluida dentro de la matriz de recursos de la red virtual. El nombre se establece en **VNet1/Subnet1** y el tipo se establece en `Microsoft.Network/virtualNetworks/subnets`. El recurso secundario se marca como dependiente del recurso primario porque este debe existir antes de que se pueda implementar el recurso secundario.

```json
"resources": [
  {
    "type": "Microsoft.Network/virtualNetworks",
    "apiVersion": "2018-10-01",
    "name": "VNet1",
    "location": "[parameters('location')]",
    "properties": {
      "addressSpace": {
        "addressPrefixes": [
          "10.0.0.0/16"
        ]
      }
    }
  },
  {
    "type": "Microsoft.Network/virtualNetworks/subnets",
    "apiVersion": "2018-10-01",
    "name": "VNet1/Subnet1",
    "dependsOn": [
      "VNet1"
    ],
    "properties": {
      "addressPrefix": "10.0.0.0/24"
    }
  }
]
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
resource <parent-resource-symbolic-name> '<resource-type>@<api-version>' = {
  <parent-resource-properties>
}

resource <child-resource-symbolic-name> '<child-resource-type>@<api-version>' = {
  <child-resource-properties>
}
```

Cuando el recurso secundario se define fuera del recurso primario, los valores type y name se indican con barras diagonales para incluir el tipo y el nombre primarios.

En el ejemplo siguiente se muestra una red virtual y una subred que se definen en el nivel raíz. Observe que la subred no está incluida dentro de la matriz de recursos de la red virtual. El nombre se establece en **VNet1/Subnet1** y el tipo se establece en `Microsoft.Network/virtualNetworks/subnets`. El recurso secundario se marca como dependiente del recurso primario porque este debe existir antes de que se pueda implementar el recurso secundario.

```bicep
param location string

resource VNet1 'Microsoft.Network/virtualNetworks@2018-10-01' = {
  name: 'VNet1'
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: [
        '10.0.0.0/16'
      ]
    }
  }
}

resource VNet1_Subnet1 'Microsoft.Network/virtualNetworks/subnets@2018-10-01' = {
  name: '${VNet1.name}/Subnet1'
  properties: {
    addressPrefix: '10.0.0.0/24'
  }
}
```

---

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información sobre cómo crear plantillas de Resource Manager, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](template-syntax.md).
* Para información sobre el formato del nombre del recurso al hacer referencia a él, consulte la [función reference](template-functions-resource.md#reference).
