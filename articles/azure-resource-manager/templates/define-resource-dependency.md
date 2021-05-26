---
title: Establecimiento del orden de implementación para recursos
description: En este artículo se describe cómo establecer un recurso de Azure como dependiente de otro recurso durante la implementación. Las dependencias garantizan que los recursos se implementan en el orden correcto.
ms.topic: conceptual
ms.date: 12/21/2020
ms.openlocfilehash: 723344a4619325eb29a481a42f7dd2de5094a7b0
ms.sourcegitcommit: 0ce834cd348bb8b28a5f7f612c2807084cde8e8f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109814071"
---
# <a name="define-the-order-for-deploying-resources-in-arm-templates"></a>Definición del orden de implementación de recursos en las plantillas de ARM

Al implementar recursos, es posible que tenga que asegurarse de que existen algunos recursos antes de crear otros. Por ejemplo, necesitará un servidor SQL lógico para poder implementar una base de datos. Establezca esta relación marcando un recurso como dependiente de otro recurso. Use el elemento `dependsOn` para definir una dependencia explícita. Utilice las funciones **reference** o **list** para definir una dependencia implícita.

Azure Resource Manager evalúa las dependencias entre recursos y los implementa en su orden dependiente. Cuando no hay recursos dependientes entre sí, Resource Manager los implementa en paralelo. Solo tiene que definir las dependencias de recursos que se implementan en la misma plantilla.

## <a name="dependson"></a>dependsOn

Dentro de la plantilla de Azure Resource Manager, el elemento `dependsOn` permite definir un recurso como dependiente de uno o más recursos. Su valor es una matriz de cadenas de notación de objetos JavaScript (JSON), cada una de las cuales es un nombre o identificador de recurso. La lista puede incluir recursos que [se implementan condicionalmente](conditional-resource-deployment.md). Cuando un recurso condicional no está implementado, Azure Resource Manager lo quita automáticamente de las dependencias necesarias.

En el ejemplo siguiente se muestra una interfaz de red que depende de una red virtual, un grupo de seguridad de red y una dirección IP pública. Para ver la plantilla completa, consulte la [plantilla de inicio rápido de una máquina virtual Linux](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-linux/azuredeploy.json).

```json
{
    "type": "Microsoft.Network/networkInterfaces",
    "apiVersion": "2020-06-01",
    "name": "[variables('networkInterfaceName')]",
    "location": "[parameters('location')]",
    "dependsOn": [
      "[resourceId('Microsoft.Network/networkSecurityGroups/', parameters('networkSecurityGroupName'))]",
      "[resourceId('Microsoft.Network/virtualNetworks/', parameters('virtualNetworkName'))]",
      "[resourceId('Microsoft.Network/publicIpAddresses/', variables('publicIpAddressName'))]"
    ],
    ...
}
```

Si bien puede que se sienta tentado a usar `dependsOn` para asignar relaciones entre los recursos, es importante comprender por qué lo hace. Por ejemplo, para documentar cómo están interconectados los recursos, `dependsOn` no es el enfoque correcto. Después de realizar la implementación, no se pueden consultar los recursos que se definieron en el elemento `dependsOn`. La configuración de dependencias innecesarias reduce el tiempo de implementación porque Resource Manager no puede implementar esos recursos en paralelo.

## <a name="child-resources"></a>Recursos secundarios

Una dependencia de implementación implícita no se crea automáticamente entre un [recurso secundario](child-resource-name-type.md) y el primario. Si necesita implementar el recurso secundario después del primario, establezca la propiedad `dependsOn`.

En el siguiente ejemplo se muestran un servidor SQL lógico y una base de datos. Observe que se ha definido una dependencia explícita entre la base de datos y el servidor, a pesar de que la base de datos es un elemento secundario del servidor.

```json
"resources": [
  {
    "type": "Microsoft.Sql/servers",
    "apiVersion": "2020-02-02-preview",
    "name": "[parameters('serverName')]",
    "location": "[parameters('location')]",
    "properties": {
      "administratorLogin": "[parameters('administratorLogin')]",
      "administratorLoginPassword": "[parameters('administratorLoginPassword')]"
    },
    "resources": [
      {
        "type": "databases",
        "apiVersion": "2020-08-01-preview",
        "name": "[parameters('sqlDBName')]",
        "location": "[parameters('location')]",
        "sku": {
          "name": "Standard",
          "tier": "Standard"
          },
        "dependsOn": [
          "[resourceId('Microsoft.Sql/servers', concat(parameters('serverName')))]"
        ]
      }
    ]
  }
]
```

Para ver la plantilla completa, consulte la [plantilla de inicio rápido para Azure SQL Database](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.sql/sql-database/azuredeploy.json).

## <a name="reference-and-list-functions"></a>Funciones de referencia y lista

La [función reference](template-functions-resource.md#reference) permite que una expresión derive su valor de otros pares de valor y nombre JSON o de recursos en tiempo de ejecución. Las [funciones list*](template-functions-resource.md#list) devuelven valores para un recurso de una operación de lista.

Las expresiones de referencia y lista declaran implícitamente que un recurso depende de otro. Siempre que sea posible, use una referencia implícita para evitar agregar una dependencia innecesaria.

Para aplicar una dependencia implícita, haga referencia al recurso por su nombre, y no por el identificador de recurso. Si se pasa el identificador de recurso a las funciones de referencia o lista, no se crea una referencia implícita.

El formato general de la función `reference` es:

```json
reference('resourceName').propertyPath
```

El formato general de la función `listKeys` es:

```json
listKeys('resourceName', 'yyyy-mm-dd')
```

En el ejemplo siguiente, un punto de conexión de CDN depende explícitamente del perfil de CDN e implícitamente de una aplicación web.

```json
{
    "name": "[variables('endpointName')]",
    "apiVersion": "2016-04-02",
    "type": "endpoints",
    "location": "[resourceGroup().location]",
    "dependsOn": [
      "[variables('profileName')]"
    ],
    "properties": {
      "originHostHeader": "[reference(variables('webAppName')).hostNames[0]]",
      ...
    }
```

Para más información, consulte [función reference](template-functions-resource.md#reference).

## <a name="depend-on-resources-in-a-loop"></a>Dependencia de los recursos de un bucle

Para implementar recursos que dependen de recursos en un [bucle de copia](copy-resources.md), tiene dos opciones. Puede establecer una dependencia de recursos individuales dentro del bucle o de todo el bucle.

> [!NOTE]
> En la mayoría de los escenarios, debe establecer la dependencia de recursos específicos dentro del bucle de copia. Dependa solo del bucle entero cuando necesite que todos los recursos del bucle existan antes de crear el recurso siguiente. Al establecer la dependencia de todo el bucle, el grafo de dependencias se expande de forma significativa, especialmente si esos recursos del bucle dependen de otros recursos. Las dependencias expandidas dificultan que la implementación se complete de forma eficaz.

En el ejemplo siguiente se muestra cómo implementar varias máquinas virtuales. La plantilla crea el mismo número de interfaces de red. Cada máquina virtual depende de una interfaz de red, en lugar de todo el bucle.

```json
{
  "type": "Microsoft.Network/networkInterfaces",
  "apiVersion": "2020-05-01",
  "name": "[concat(variables('nicPrefix'),'-',copyIndex())]",
  "location": "[parameters('location')]",
  "copy": {
    "name": "nicCopy",
    "count": "[parameters('vmCount')]"
  },
  ...
},
{
  "type": "Microsoft.Compute/virtualMachines",
  "apiVersion": "2020-06-01",
  "name": "[concat(variables('vmPrefix'),copyIndex())]",
  "location": "[parameters('location')]",
  "dependsOn": [
    "[resourceId('Microsoft.Network/networkInterfaces',concat(variables('nicPrefix'),'-',copyIndex()))]"
  ],
  "copy": {
    "name": "vmCopy",
    "count": "[parameters('vmCount')]"
  },
  "properties": {
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "[resourceId('Microsoft.Network/networkInterfaces',concat(variables('nicPrefix'),'-',copyIndex()))]",
          "properties": {
            "primary": "true"
          }
        }
      ]
    },
    ...
  }
}
```

En el ejemplo siguiente, se muestra cómo se implementan tres cuentas de almacenamiento antes de implementar la máquina virtual. Observe que el elemento `copy` tiene `name` establecido en `storagecopy` y el elemento `dependsOn` de la máquina virtual también está establecido en `storagecopy`.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[concat(copyIndex(),'storage', uniqueString(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "copy": {
        "name": "storagecopy",
        "count": 3
      },
      "properties": {}
    },
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2015-06-15",
      "name": "[concat('VM', uniqueString(resourceGroup().id))]",
      "dependsOn": ["storagecopy"],
      ...
    }
  ],
  "outputs": {}
}
```

## <a name="circular-dependencies"></a>Dependencias circulares

Resource Manager identifica dependencias circulares durante la validación de plantillas. Si recibe un error en una dependencia circular, evalúe la plantilla para ver si se puede quitar alguna dependencia. Si no es suficiente con quitar dependencias, puede evitar dependencias circulares moviendo algunas operaciones de implementación a recursos secundarios. Implemente los recursos secundarios después de los recursos que tienen la dependencia circular. Por ejemplo, suponga que va a implementar dos máquinas virtuales pero debe establecer propiedades en cada una que hagan referencia a la otra. Puede implementarlas en el orden siguiente:

1. vm1
2. vm2
3. La extensión en vm1 depende de vm1 y vm2. La extensión establece valores en vm1 que obtiene de vm2.
4. La extensión en vm2 depende de vm1 y vm2. La extensión establece valores en vm2 que obtiene de vm1.

Para información sobre cómo evaluar el orden de implementación y resolver errores de dependencia, consulte [Solución de errores comunes de implementación de Azure con Azure Resource Manager](common-deployment-errors.md).

## <a name="next-steps"></a>Pasos siguientes

* Para realizar un tutorial, consulte [Tutorial: Creación de plantillas de Resource Manager con recursos dependientes](template-tutorial-create-templates-with-dependent-resources.md).
* Para un módulo de Microsoft Learn que abarca las dependencias de los recursos, consulte [Administración de implementaciones complejas en la nube mediante características avanzadas de la plantilla de ARM](/learn/modules/manage-deployments-advanced-arm-template-features/).
* Para obtener recomendaciones sobre cómo establecer las dependencias, consulte [Procedimientos recomendados para plantillas de Resource Manager](template-best-practices.md).
* Para aprender a solucionar los problemas de dependencias durante la implementación, consulte [Solución de errores comunes de implementación de Azure con Azure Resource Manager](common-deployment-errors.md).
* Para aprender a crear plantillas de Azure Resource Manager, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](template-syntax.md).
* Para obtener una lista de las funciones disponibles en una plantilla, consulte [Funciones de plantilla de ARM](template-functions.md).
